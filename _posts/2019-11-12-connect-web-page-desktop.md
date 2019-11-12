---
layout: post
title: Connecting Web Page To Desktop Application
description: Enabling the real-time connection between html web-page and desktop SOLIDWORKS application via cross-platform and cross-browser SignalR framework
image: /res/2019-11-12-connect-web-page-desktop/web-desktop-connection.png
labels: signalr,web,connection
---
{% include img.html src="/res/2019-11-12-connect-web-page-desktop/web-desktop-connection.svg" width=350 alt="Web Page and Desktop application connection" align="center" %}

## Intro

There are multiple scenarios where it is required to connect desktop application to the web page. This includes but not limited to

* Automation software to invoke the work on a remote server to be performed by desktop application
* Implementation of desktop agents for the cloud based application
* Dashboard to collect the application insights from the desktop application and report to centralized server

Usually the desktop application would act as a client which connects to the server. However in some cases desktop application could be considered as a server where web pages need to connect to.

## Task

In this blog post I will guide you through the steps of setting up a simple application which allows for a html web page client to connect to local SOLIDWORKS desktop application and perform the model generation job with a real time status updates.

One of the possible ways to approach this task is to use [ActiveX](https://en.wikipedia.org/wiki/ActiveX) technology. This way however has a lot of limitation due to the security concerns and will only work in Internet Explorer when specific settings enabled. Follow [Render Feature Tree In HTML Page](https://www.codestack.net/solidworks-api/getting-started/scripts/java-script/html-feature-tree/) for the example of utilizing this technology to connect to SOLIDWORKS application.

Another way to use [duplex services in Windows Communication Foundation (WCF)](https://docs.microsoft.com/en-us/dotnet/framework/wcf/feature-details/duplex-services). But this is Windows specific technology which most likely will not be supported in future version of .NET as it is currently not supported in .NET Core and presumably won't be ported to .NET 5.

The clear winner to solve this task is [SignalR](https://dotnet.microsoft.com/apps/aspnet/signalr) technology. This is free, open source, cross platform and cross browser framework for .NET allowing to enable real time communication between server (SignalR Hub) and clients in all modern browsers, operating systems and devices.

SignalR is using web sockets to enable real time communication, however will fallback on other options when web sockets are not available such as long polling, Server Sent Events (SSE) or Forever-frame. SignalR Software Development Kit (SDK) is available for .NET (C# and VB.NET), JavaScript and Java.

## Architecture

{% include img.html src="/res/2019-11-12-connect-web-page-desktop/architecture-schema.svg" alt="Application components schema" align="center" %}

SignalR hub is self-hosted in the SOLIDWORKS add-in on a localhost. Simple web page clients will connect to the hub via url. *Build* button in the html page would invoke the corresponding *Build* function in SignalR hub and pass the parameter input by the user.

SignalR hub invokes SOLIDWORKS API to generate a simple box geometry with specified parameters and save the generated model into the local folder.

Hub will broadcast the calculated mass of the generated body back to the caller client via *SendResult* method and will update all of the existing connected clients with the total number of models generated in the current session via *UpdateStatus* method.

Client will display a message box with the mass result and all clients will instantly update the label with total number of models (i.e. it won't be required to hit refresh (F5) button to update the page).

## Implementation

We will be using [Microsoft.AspNet.SignalR.SelfHost](https://www.nuget.org/packages/Microsoft.AspNet.SignalR.SelfHost/) and [Microsoft.Owin.Cors](https://www.nuget.org/packages/Microsoft.Owin.Cors/) nuget packages to create a self hosted SignalR hub

~~~cs
[HubName("ModelBuilder")]
public class ModelBuilderHub : Hub<IModelBuilderClient>
{
    ...

    public void Build(double width, double length, double height)
    {
        int totalModelsBuilt;
        var mass = m_ModelBuilder.Build(width, height, length, out totalModelsBuilt);
        Clients.All.UpdateStatus(totalModelsBuilt);
        Clients.Caller.SendResult(mass);
    }
}
~~~

Syntax of the SignalR clients can be defined via interface and set in the generic parameter of the hub

~~~cs
public interface IModelBuilderClient
{
    void SendResult(double mass);
    void UpdateStatus(int totalModelsBuilt);
}
~~~

We will be using [Open Web Interface for .NET (OWIN)](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/owin?view=aspnetcore-3.0) to host the SignalR hub. We will need to enable [Cross-Origin Resource Sharing (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) as our SignalR hub and web client will be running on different ports.

This can be done by providing the configuration startup class.

~~~cs
private class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseCors(CorsOptions.AllowAll);
        app.MapSignalR();
    }
}
...
m_SignalRHub = WebApp.Start<Startup>(URL);
~~~

In order to connect to the hub from the web page client we will be using SignalR JavaScript library based on jQuery.

~~~js
$.connection.hub.url = "http://localhost:8080/ModelBuilder/signalr";

var builder = $.connection.ModelBuilder;

builder.client.updateStatus = function (totalModelsBuilt) {
    $('#status').text("Total Models Built: " + totalModelsBuilt);
};

builder.client.SendResult = function (mass) {
    alert("Mass: " + mass);
};

$.connection.hub.start()
    .done(function () {
        $('#build').click(function () {
            var size = $('#size').val();
            builder.server.build(size, size, size);
        })
    });
~~~

Please see the detailed video tutorial below for developing this application from scratch:

{% include youtube.html id="4YxGjaKvNb0" width=560 height=315 %}

Source code is available on [GitHub](https://github.com/codestackdev/solidworks-api-examples/tree/master/swex/add-in/signalr-model-builder).

## Limitations

### Version Conflict

Nuget packages from the SDK might reference different versions of shared libraries. In this case Visual Studio will automatically generate the *app.config* file to resolve the conflict by redirecting the binding.

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Owin" publicKeyToken="31bf3856ad364e35" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-4.0.1.0" newVersion="4.0.1.0" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
~~~

This however will not work for the add-in application compiled to .dlls as the app.config will only be read from the hosting executable (in our example SLDWORKS.exe). Although it is possible to alter the *app.config* for application as well as provide system wide *app.config* it is not a recommended practice.

Instead we can resolve the references conflict at runtime as shown below:

~~~ cs
static ModelBuilderAddIn()
{
    AppDomain.CurrentDomain.AssemblyResolve += OnAssemblyResolve;
}

private static Assembly OnAssemblyResolve(object sender, ResolveEventArgs args)
{
    var dir = Path.GetDirectoryName(typeof(ModelBuilderAddIn).Assembly.Location);
    var fileName = $"{new AssemblyName(args.Name).Name}.dll";
    return Assembly.LoadFile(Path.Combine(dir, fileName));
}
~~~

### Licensing

Please refer the End User License Agreement (EULA) for the specific desktop application you are connecting to as there may be limitations regarding the use of the software via Local Access Network (LAN) or Internet (WAN). SOLIDWORKS Eula can be found at the [following link](https://www.solidworks.com/license-agreement), please refer the corresponding section which describes the limitation of usage over the internet.