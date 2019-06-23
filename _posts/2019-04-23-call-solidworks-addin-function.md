---
layout: post
title: How To Call Functions Of SOLIDWORKS add-in
description: Calling functions of SOLIDWORKS add-in from macros or stand-alone applications
image: /res/2019-04-23-call-solidworks-addin-function/solidworks-addin-api.png
labels: add-in api, add-in object, com, functions, solidworks api
redirect_from:
  - /2019/04/call-functions-of-solidworks-add-in.html
---
{% include img.html src="/res/2019-04-23-call-solidworks-addin-function/solidworks-addin-api.png" alt="SOLIDWORKS add-in API" align="center" %}

SOLIDWORKS add-ins are great options for automating SOLIDWORKS. But what if the add-in itself needs to be automated? This enables a whole new functionality as now add-in functions can be called from [macros](https://www.codestack.net/solidworks-api/getting-started/macros/), [add-ins](https://www.codestack.net/solidworks-api/getting-started/add-ins/), [stand-alone applications](https://www.codestack.net/solidworks-api/getting-started/stand-alone/) or even [scripts](https://www.codestack.net/solidworks-api/getting-started/scripts/). So effectively the add-in is exposing API itself.

There are multiple ways to add API to your add-in. In this blog post we will take a look in automating add-ins via COM interface and [ISldWorks::GetAddInObject](http://help.solidworks.com/2018/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.isldworks~getaddinobject.html) SOLIDWORKS API method.

When developing add-ins with .NET language (C# or VB.NET) it is required to enable COM communication via COM visible interfaces:

* Create public interface which exposes the API functions required to be accessed by external applications. Make this interface COM visible
* Implement this interface in your main add-in class
* Register assembly for COM interops

~~~ cs
[ComVisible(true)]
public interface IMyAddInApi
{
    double DoSomething(int someParam);
} 

[ComVisible(true), Guid("799A191E-A4CF-4622-9E77-EA1A9EF07621")]
public class MyAddIn : ISwAddIn
{
    ...
    public double DoSomething(int someParam)
    {
        //Implement
    }
}
~~~

As the result type library (*.tlb) file is generated. This now can be referenced in the macro or stand-alone application and add-in API can be called.

~~~ vb
Dim swApp As SldWorks.SldWorks

Sub main()

    Set swApp = Application.SldWorks
    
    Dim swMyAddIn As CodeStack.IMyAddInApi

    Set swMyAddIn = swApp.GetAddInObject("CodeStack.MyAddInApi")
    
    Dim res As Double

    res = swMyAddIn.DoSomething(10)
    
End Sub
~~~

Refer the [Call function of SOLIDWORKS add-in object from stand-alone application or macro](https://www.codestack.net/solidworks-api/getting-started/inter-process-communication/invoke-add-in-functions/via-add-in-object/) for detailed guide and code examples.