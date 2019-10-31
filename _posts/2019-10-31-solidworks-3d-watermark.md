---
layout: post
title: Creating 3D Watermark In SOLIDWORKS Model (Halloween Special)
description: Example of loading the body from the external file and displaying it as 3D watermark using SOLIDWORKS API
image: /res/2019-10-31-solidworks-3d-watermark/solidworks-halloween.png
labels: serialization,com stream
---
{% include img.html src="/res/2019-10-31-solidworks-3d-watermark/solidworks-halloween.svg" width=350 alt="SOLIDWORKS Halloween" align="center" %}

Below macro loads the Halloween pumpkin model from an external [geometry file](/res/2019-10-31-solidworks-3d-watermark/geometry.dat) and displays it in the graphics window only. The geometry displayed as 3D watermark and cannot be selected or edited. It is also not present in the Feature Manager Tree as it not generated using features.

{% include img.html src="/res/2019-10-31-solidworks-3d-watermark/temp-body.png" width=450 alt="Temp body displayed in the graphics view" align="center" %}

## How To Run Macro

* Create new macro and paste the code below

~~~vb
Private Declare PtrSafe Function CreateStreamOnHGlobal Lib "ole32" (ByVal hGlobal As LongPtr, ByVal fDeleteOnRelease As Long, ByRef ppstm As Any) As Long

Dim swApp As SldWorks.SldWorks
Dim swBody As SldWorks.Body2

Sub main()

    Set swApp = Application.SldWorks
    
    Dim swModel As SldWorks.ModelDoc2
    Set swModel = swApp.ActiveDoc
    
    If Not swModel Is Nothing Then
    
        Dim filePath As String
        filePath = swApp.GetCurrentMacroPathFolder() & "\geometry.dat"
        
        Set swBody = LoadBodyFromFile(filePath)
        swBody.Display3 swModel, RGB(255, 165, 0), swTempBodySelectOptions_e.swTempBodySelectOptionNone
        
    Else
        MsgBox "Please open the model"
    End If
    
End Sub

Function LoadBodyFromFile(filePath As String) As SldWorks.Body2

    Dim buff() As Byte
    buff = ReadByteArrFromFile(filePath)
    
    Dim comStream As IUnknown
    Set comStream = BytesArrToComStream(buff)
    
    Dim swModeler As SldWorks.Modeler
    Set swModeler = swApp.GetModeler
    
    Dim swBody As SldWorks.Body2
    Set swBody = swModeler.Restore(comStream)
    
    Set LoadBodyFromFile = swBody
        
End Function

Function ReadByteArrFromFile(filePath) As Byte()

    Dim buff() As Byte
    
    Dim fileNumb As Integer
    fileNumb = FreeFile
    
    Open filePath For Binary Access Read As fileNumb
    
    ReDim buff(0 To LOF(fileNumb) - 1)
    
    Get fileNumb, , buff
    
    Close fileNumb
    
    ReadByteArrFromFile = buff
    
End Function

Private Function BytesArrToComStream(ByRef buff() As Byte) As IUnknown
    
    Dim comStream As IUnknown
    
    If CreateStreamOnHGlobal(VarPtr(buff(LBound(buff))), 0, comStream) Then
        Err.Raise vbError, "", "Failed to create stream from byte array"
    End If
    
    Set BytesArrToComStream = comStream
    
End Function
~~~

* Download [geometry file](/res/2019-10-31-solidworks-3d-watermark/geometry.dat) and place in the same folder as macro
* Create new part file or open existing one
* Run the macro. 3D watermark is displayed. This is a copy of the model downloaded from [GrabCad](https://grabcad.com/library/pumpkin-5)
* Stop the macro or close the model to remove the watermark

## How It Works

This macro is utilizing the temp bodies functionality which allows to use low level geometry without SOLIDWORKS features.

Bodies can be serialized into the [COM Stream](https://docs.microsoft.com/en-us/windows/win32/api/objidl/nn-objidl-istream) via [IBody2::Save](http://help.solidworks.com/2016/english/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.IBody2~Save.html) method. Stream can then be saved into an external binary file as shown in [Save Body To File](https://www.codestack.net/solidworks-api/geometry/save-body-to-file/) example or can be used as a part of the [structured storage](https://www.codestack.net/solidworks-api/data-storage/third-party/tree-structure-serialization/).

In a similar way body can be restored via [IModeler::Restore](http://help.solidworks.com/2017/English/api/sldworksapi/SOLIDWORKS.Interop.sldworks~SOLIDWORKS.Interop.sldworks.IModeler~Restore.html) SOLIDWORKS API. Here is an example of [Reading The Body From File](https://www.codestack.net/solidworks-api/geometry/read-body-from-file/).

