---
layout: post
title: Handling SOLIDWORKS events
description: Examples and guides on handling SOLIDWORKS events in VBA, C# and VB.NET using SOLIDWORKS API. Such notifications include, but not limited to file opening, selection, features creation, rebuild, etc.
image: /res/2019-10-09-handling-solidworks-events/solidworks-api-events.png
labels: events,notification,trigger
---
{% include img.html src="/res/2019-10-09-handling-solidworks-events/solidworks-api-events.png" alt="SOLIDWORKS API events" align="center" %}

Notification is a way of SOLIDWORKS to tell its listeners that something has happened. When developing applications or macros using SOLIDWORKS API it might be required to trigger certain functionality (i.e. run certain code) when some event happens. For example, you might want to update custom property when [new document is created](https://www.codestack.net/solidworks-api/application/documents/handle-new-document/) or write a log message on [file is closing](https://www.codestack.net/solidworks-api/document/file-close-event/).

There are much more events exposed by SOLIDWORKS, this includes but not limited to [feature creation](https://www.codestack.net/solidworks-api/document/features-manager/catch-new-feature-creation-event/), [document save](https://www.codestack.net/solidworks-api/application/documents/handle-document-save/), [objects selection](https://www.codestack.net/solidworks-api/document/selection/wait-for-selection#handling-the-selection-event) etc.

Below is a list of SOLIDWORKS interfaces which expose events and the corresponding links to the API help documentation:

* [ISldWorks](http://help.solidworks.com/SearchEx.aspx?q=DSldWorksEvents_&version=2019&lang=english&prod=api)
* [IPartDoc](http://help.solidworks.com/SearchEx.aspx?q=DPartDocEvents_&version=2019&lang=english&prod=api)
* [IAssemblyDoc](http://help.solidworks.com/SearchEx.aspx?q=DAssemblyDocEvents_&version=2019&lang=english&prod=api)
* [IDrawingDoc](http://help.solidworks.com/SearchEx.aspx?q=DDrawingDocEvents_&version=2019&lang=english&prod=api)
* [IModelView](http://help.solidworks.com/SearchEx.aspx?q=DModelViewEvents_&version=2019&lang=english&prod=api)
* [IFeatMgrView](http://help.solidworks.com/SearchEx.aspx?q=DFeatMgrViewEvents_&version=2019&lang=english&prod=api)
* [ITaskpaneView](http://help.solidworks.com/SearchEx.aspx?q=DTaskpaneViewEvents_&version=2019&lang=english&prod=api)
* [IMouse](http://help.solidworks.com/SearchEx.aspx?q=DMouseEvents_&version=2019&lang=english&prod=api)
* [IMotionStudy](http://help.solidworks.com/SearchEx.aspx?q=DMotionStudyEvents_&version=2019&lang=english&prod=api)
* [ISWPropertySheet](http://help.solidworks.com/SearchEx.aspx?q=DSWPropertySheetEvents_&version=2019&lang=english&prod=api)

In this blog article I will show how to handle [file open event](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.dsldworksevents_fileopennotify2eventhandler.html) in [VBA](#handling-events-in-vba-macros), [C#](#handling-events-in-c-applications) and [VB.NET](#handling-events-in-vbnet-applications)

We will display a simple message box with the full path to the opened document once the event is triggered.

{% include img.html src="/res/2019-10-09-handling-solidworks-events/doc-opened-message.png" alt="Document opened message box" align="center" %}

## Handling events in VBA Macros

To handle [events](https://www.codestack.net/visual-basic/events/) in VBA it is required to declare the object which exposes the events using the **WithEvents** keyword. This keyword can only be used in [class modules](https://www.codestack.net/visual-basic/classes/) and not available in [modules](https://www.codestack.net/visual-basic/modules/).

Once the object is declared with **WithEvents** keyword, its events can be selected from the drop-down in the VBA Editor.

{% include img.html src="/res/2019-10-09-handling-solidworks-events/events-list.png" alt="List of SOLIDWORKS events" align="center" %}

Select the required event and implement the code within the generated event handler function.

~~~vb
Dim WithEvents swApp As SldWorks.SldWorks

Private Sub Class_Initialize()
    Set swApp = Application.SldWorks
End Sub

Private Function swApp_FileOpenNotify2(ByVal FileName As String) As Long
    swApp.SendMsgToUser2 FileName & " opened", swMessageBoxIcon_e.swMbInformation, swMessageBoxBtn_e.swMbOk
    swApp_FileOpenNotify2 = 0
End Function
~~~

The class which handles the events could be instantiated in the main module once the macro runs.

{% include img.html src="/res/2019-10-09-handling-solidworks-events/vba-project-tree.png" width=250 alt="VBA project tree" align="center" %}

~~~vb
Dim swEventsHandler As EventsHandler

Sub main()
    
    Set swEventsHandler = New EventsHandler

End Sub
~~~

## Handling events in C# Applications

When developing C# applications events can be handled by using the **+=** operator and specifying the handler function. Handler function must match the signature of event's delegate.

> Use {Tab}{Tab} combination in Visual Studio to automatically generate event handler with the correct signature.

It is recommended to unsubscribe from the event on closing of the application. This can be done by calling the **-=** operator.

> Note it is required to use versions of the interfaces without I at the beginning as [I-methods](https://www.codestack.net/solidworks-api/getting-started/api-object-model/i-api-versions/) do not expose events (i.e. use SldWorks instead of ISldWorks).

~~~cs
using SolidWorks.Interop.sldworks;
using SolidWorks.Interop.swconst;
using System;

namespace CodeStack
{
    class Program
    {
        private static SldWorks m_App;

        static void Main(string[] args)
        {
            m_App = Activator.CreateInstance(Type.GetTypeFromProgID("SldWorks.Application")) as SldWorks;
            m_App.Visible = true;

            m_App.FileOpenNotify2 += OnFileOpenNotify2;

            Console.ReadLine();

            m_App.FileOpenNotify2 -= OnFileOpenNotify2;
        }

        private static int OnFileOpenNotify2(string fileName)
        {
            m_App.SendMsgToUser2($"{fileName} opened", (int)swMessageBoxIcon_e.swMbInformation, (int)swMessageBoxBtn_e.swMbOk);
            return 0;
        }
    }
}
~~~

## Handling events in VB.NET Applications

VB.NET allows to subscribe to events using the **AddHandler** keyword and providing the reference to the handler function using the **AddressOf** keyword. Event can be unsubscribed using the **RemoveHandler** keyword. 

~~~vb
Imports SolidWorks.Interop.sldworks
Imports SolidWorks.Interop.swconst

Module Module1

    Dim m_App As SldWorks

    Sub Main()

        m_App = TryCast(Activator.CreateInstance(Type.GetTypeFromProgID("SldWorks.Application")), SldWorks)
        m_App.Visible = True

        AddHandler m_App.FileOpenNotify2, AddressOf OnFileOpenNotify2

        Console.ReadLine()

        RemoveHandler m_App.FileOpenNotify2, AddressOf OnFileOpenNotify2

    End Sub

    Function OnFileOpenNotify2(ByVal fileName As String) As Integer
        m_App.SendMsgToUser2($"{fileName} opened", CInt(swMessageBoxIcon_e.swMbInformation), CInt(swMessageBoxBtn_e.swMbOk))
        Return 0
    End Function

End Module
~~~

## Handling events in SwEx.AddIn Framework

SOLIDWORKS doesn't group document events under the common [IModelDoc2](https://help.solidworks.com/2012/English/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IModelDoc2.html) interface, and in most cases provides similar or the same event for part, assembly and drawing. It is required to explicitly handle this case in your application.

To handle events of documents it required to catch the document loading events of **SldWorks** interface first and subscribe to target events within its handler.

[SwEx.AddIn](https://www.codestack.net/labs/solidworks/swex/add-in/) framework significantly simplifies managing of document events via its [document management](https://www.codestack.net/labs/solidworks/swex/add-in/documents-management/events/) mechanism.

For example the following code would handle rebuild operations (pre and post) for all types of documents currently loaded or newly created or opened in the SOLIDWORKS session.

~~~cs
private IDocumentsHandler<DocumentHandler> m_DocHandlerGeneric;

public override bool OnConnect()
{
	m_DocHandlerGeneric = CreateDocumentsHandler();
	m_DocHandlerGeneric.HandlerCreated += OnHandlerCreated;
	return true;
}

private void OnHandlerCreated(DocumentHandler doc)
{
	doc.Rebuild += OnRebuild;
}

private bool OnRebuild(DocumentHandler docHandler, RebuildState_e type)
{
    //TODO: implement handler for rebuilding
    return true;
}
~~~