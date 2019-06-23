---
layout: post
title: SOLIDWORKS API Command Doesn't Exist? Need plan B
description: Alternative workarounds of calling SOLIDWORKS functions when SOLIDWORKS API method is not implemented. How to use Windows API and SOLIDWORKS commands to call functions of SOLIDWORKS
labels:
image: /res/2019-03-07-missing-solidworks-api-command/solidworks-api-method-unavailable.png
redirect_from:
  - /2019/03/solidworks-api-command-doesnt-exist.html
---
{% include img.html src="/res/2019-03-07-missing-solidworks-api-command/solidworks-api-method-unavailable.png" alt="Missing SOLIDWORKS API method" align="center" %}

SOLIDWORKS API is a powerful toolset covering majority of SOLIDWORKS functions. However in some cases certain APIs are not available. This could potentially put the whole development project at risk as workaround is not always possible.

In this post I will discuss alternative solutions of calling SOLIDWORKS commands when API cannot be used (either not available or faulty).

## Running commands from swCommands_e enumeration

SOLIDWORKS provides more than 3000 commands identifications via [swCommands_e](http://help.solidworks.com/2012/english/api/swcommands/solidworks.interop.swcommands~solidworks.interop.swcommands.swcommands_e.html) enumeration which can be invoked via [ISldWorks::RunCommand](http://help.solidworks.com/2012/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.isldworks~runcommand.html) method.

Follow the [Capture SOLIDWORKS Commands](https://www.codestack.net/solidworks-api/application/frame/capture-commands/) article for more information and macro to simplify capturing of the ids of invoked commands.

{% include img.html src="/res/2019-03-07-missing-solidworks-api-command/capturing-hide-command-id.png" alt="Capturing ID of a SOLIDWORKS hide command" align="center" %}

Refer the [Display Assembly Visualization Page](https://www.codestack.net/solidworks-api/document/assembly/display-assembly-visualization-page/) for a code example for invoking the command.

These commands are usually used to invoke dialog buttons (including Property Manager Page) or toolbar/menu buttons which cannot be called from the core SOLIDWORKS API.

By unknown reasons all the commands invoked via [ISldWorks::RunCommand](http://help.solidworks.com/2012/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.isldworks~runcommand.html) method introduce about 1 second delay. If multiple commands need to be invoked this may cause the performance issue. Refer the next section for an alternative solution.

## Calling Windows Command

Although swCommands_e enumeration described above is very powerful, there are still may be some commands unavailable.

I will describe another way of calling the commands using Windows API.

SOLIDWORKS is Win32 application which means that all the communication is routed via message map. Those commands can be called from the Windows API functions, such as [SendMessage](https://docs.microsoft.com/en-us/windows/desktop/api/winuser/nf-winuser-sendmessage).

The challenge is how to discover the id of the required command.

Fortunately, multiple tools available allowing to hook Win32 messages from Windows application. One of these tools is [Spy++](https://docs.microsoft.com/en-us/visualstudio/debugger/introducing-spy-increment?view=vs-2017) built into MS Visual Studio.

This tool is available from the TOOLS menu in MS Visual Studio IDE.

{% include img.html src="/res/2019-03-07-missing-solidworks-api-command/spy-plus-plus-utility.png" alt="Spy++ utility" align="center" %}

I would recommend to set the log options to only capture WM_COMMAND message.

{% include img.html src="/res/2019-03-07-missing-solidworks-api-command/log-options.png" alt="Spy++ utility log options" align="center" %}

Simply find the SOLIDWORKS Window, click required command and capture the wParam value

{% include img.html src="/res/2019-03-07-missing-solidworks-api-command/spy-plus-plus-capturing.gif" alt="Capturing Windows messages using Spy++" align="center" %}

The value recorded as wParam is a command id. Use the following code to invoke this command.

~~~ vb
#If VBA7 Then
     Private Declare PtrSafe Function SendMessage Lib "User32" Alias "SendMessageA" (ByVal hWnd As Long, ByVal wMsg As Long, ByVal wParam As Long, lParam As Any) As Long
#Else
     Private Declare Function SendMessage Lib "User32" Alias "SendMessageA" (ByVal hWnd As Long, ByVal wMsg As Long, ByVal wParam As Long, lParam As Any) As Long
#End If

Dim swApp As SldWorks.SldWorks
 
Sub main()
 
    Set swApp = Application.SldWorks
     
    RunWmCommand swApp, 33227

End Sub

Sub RunWmCommand(swApp As SldWorks.SldWorks, cmd As Long)
    
    Const WM_COMMAND As Long = &H111
        
    Dim swFrame As SldWorks.Frame
        
    Set swFrame = swApp.Frame
        
    SendMessage swFrame.GetHWnd(), WM_COMMAND, cmd, 0
     
End Sub
~~~

## Examples

* [Run Xpress Products](https://www.codestack.net/solidworks-api/application/frame/run-xpress-products/)
* [Show All Components (Show With Dependents)](https://www.codestack.net/solidworks-api/document/assembly/components/show-with-dependents/)
* [Move To Folder](https://www.codestack.net/solidworks-api/document/assembly/components/move-to-folder/)

> Sending message in unexpected way (e.g. calling the hide sketch where document is not opened and sketch is not selected) might result in unpredictable behavior (including crash). Always check the conditions before sending Win32 message.