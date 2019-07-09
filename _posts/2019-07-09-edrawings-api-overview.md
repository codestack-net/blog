---
layout: post
title: Overview of SOLIDWORKS eDrawings API
description: Detailed overview of eDrawings API, creating Windows Forms and WPF application, using markup API, creating the batch export to PDF without SOLIDWORKS using only eDrawings API
image: /res/2019-07-09-edrawings-api-overview/edrawings.png
labels: edrawings,markup,pdf
---
{% include img.html src="/res/2019-07-09-edrawings-api-overview/edrawings.svg" width=350 alt="SOLIDWORKS eDrawings" align="center" %}

eDrawings is a free SOLIDWORKS file viewer. Software can be downloaded from the [eDrawingsViewer](https://www.edrawingsviewer.com/) web-site. This application not only allows to view, markup, measure and print SOLIDWORKS, DXF and DWG files without having SOLIDWORKS installed, but also provides API for a programmatic access to its features.

In this blog article I will guide you though the basics of eDrawings API. I will show how to host the eDrawings control on Windows Forms or WPF application. Introduce to the eDrawings markup API to extract the measurements. And finally describe 3 most common use cases where eDrawings can be useful for, including the application to batch export SOLIDWORKS drawing to PDF format without having SOLIDWORKS installed or even without a need to own a SOLIDWORKS license.

## Creating Windows Forms application

{% include youtube.html id="-O1kLCsrFl0" width=560 height=315 %}

eDrawings interop provides signatures of API methods which can be used when programming in C# or VB.NET and located in the installation folder of eDrawings: *%commonprogramfiles%\eDrawings[Version]\eDrawings.Interop.EModelViewControl.dll*.

Simply create a wrapper around eDrawings ActiveX control and add it to your form. Now you can explore eDrawings API!

~~~ cs
public class EDrawingHost : AxHost
{
    public EDrawingHost() : base("22945A69-1191-4DCF-9E6F-409BDE94D101")
    {
        m_IsLoaded = false;
    }
}
~~~

Follow the [Hosting eDrawings control in Windows Forms](https://www.codestack.net/edrawings-api/gettings-started/winforms/) for detailed instruction and code examples.

## Creating WPF application

{% include img.html src="\res\2019-07-09-edrawings-api-overview\edrawings-wpf-window.png" width=350 alt="eDrawings control in WPF form" align="center" %}

If you are using the modern Windows Presentation Foundation (WPF) framework and want to host your eDrawings control in there you can do it by wrapping your control in the [WindowsFormsHost](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.integration.windowsformshost?view=netframework-4.8)

~~~ cs
public partial class eDrawingsHostControl : UserControl
{
    private EModelViewControl m_Ctrl;

    public eDrawingsHostControl()
    {
        InitializeComponent();

        var host = new WindowsFormsHost();
        var ctrl = new EDrawingHost();
        host.Child = ctrl;
        this.AddChild(host);
    }
}
~~~

Follow the [Hosting SOLIDWORKS eDrawings control in Windows Presentation Foundation (WPF)](https://www.codestack.net/edrawings-api/gettings-started/wpf/) for detailed instruction and code examples.

## Markup

One of the cool eDrawings features is an ability to add markups to the files. This includes stamps, comments, lines, arcs, clouds etc. It is also possible to perform measurements directly in the eDrawings viewer. Markup API is available in [IEModelMarkupControl](http://help.solidworks.com/2016/english/api/emodelapi/eDrawings.Interop.EModelMarkupControl~eDrawings.Interop.EModelMarkupControl.IEModelMarkupControl.html) interface.

Follow [Utilizing markup functionality using SOLIDWORKS eDrawings API](https://www.codestack.net/edrawings-api/markup/) tutorial for a guide and project example of using eDrawings markup API.

## Use Cases

Although eDrawings API is relatively small compared to SOLIDWORKS API, it is still very powerful and can be used in the number of applications. Below are the most common scenarios of utilizing eDrawings API

### Batch export to pdf

There is no way to export eDrawings files into foreign format, however the file can be printed and if PDF printer is setup, eDrawings can be used to effectively batch convert drawings into a PDF format. You can find the application which is doing this by following: [Batch export SOLIDWORKS files to PDF via eDrawings API (without SOLIDWORKS)](https://www.codestack.net/edrawings-api/output/print-to-pdf/) link.

### File viewer

This would be probably the most common use of eDrawings. If you are building Product Data Management or Product Catalogue applications and want to enable viewing of files eDrawings would provide simple and lightweight way to do this.

### Inspection tool

Using eDrawings markup capabilities it is possible to implement the inspection tool where users will be able to add markups, review comments and stamps to the files in the business workflow.

