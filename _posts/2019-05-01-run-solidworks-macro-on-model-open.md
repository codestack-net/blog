---
layout: post
title: Automatically Run Macro On Model Open
description: Options to automatically run VBA macro on document open (without add-in) using event notifications or macro feature
image: /res/2019-05-01-run-solidworks-macro-on-model-open/load-watcher-macro-feature.png
labels: document open, event, macro feature, run macro
redirect_from:
  - /2019/05/automatically-run-macro-on-model-open.html
---
Since I have published the guide to [Automatically Run Macro On Rebuild](https://www.codestack.net/solidworks-api/document/macro-feature/run-macro-on-rebuild/) I was asked several times of how to run the macro every time the document opens.

{% include img.html src="/res/2019-05-01-run-solidworks-macro-on-model-open/solidworks-file-open-dialog.png" alt="SOLIDWORKS file open dialog" align="center" %}

You might want to do this to update your SOLIDWORKS custom properties (e.g. from external text file), add some log entries, update tree etc.

Of course you can do this with [SOLIDWORKS add-ins](https://www.codestack.net/solidworks-api/getting-started/add-ins/), but what if it is required to easily share the model with colleagues or customers and add-in might not be an option? In this blog article I will describe two ways to do this directly from VBA macros without add-ins or any additional software.

## Run VBA Code Using SOLIDWORKS Document Load Notification

This is the video demonstration of this approach to automatically name files based on the shared counter value

{% include youtube.html id="tgRB8YtB4v4" width=560 height=315 %}

SOLIDWORKS API sends various notifications when state of the model or application is changed. One of those notifications is document load. I.e. it is possible to handle this event and run custom code.

The main challenge with this approach is to keep your VBA macro running all the time during the SOLIDWORKS session. This could be achieved by running an infinite loop with DoEvents call to process background messages.

~~~ vb
While True
    DoEvents
Wend
~~~

While performing tests I have noticed that macro stays loaded even if the above code not called (i.e. SOLIDWORKS seems to be not unloading the macro file immediately after run). But as this is not documented behaviour, I have decided is too risky to rely on this. There is a chance that SOLIDWORKS will unload macro automatically if not used and the monitoring will be stopped. Calling the infinite loop with DoEvents prevents macro from existing as it is constantly running in the background.

Follow [Run VBA macro automatically on document load using SOLIDWORKS API](https://www.codestack.net/solidworks-api/application/documents/handle-document-load/) tutorial for source code and detailed explanation of this approach.

## Run VBA Code Using Macro Feature

Below is a video demonstration of this approach to add log entry into a text file when document is opened

{% include youtube.html id="BTM5NZNdON8" width=560 height=315 %}

Previous approach is easy to setup and enables running of the macros for each opened documents. But what if model itself needs to know whether it should run any custom code (for example embedded macro)?

Fortunately SOLIDWORKS [Macro Feature](https://www.codestack.net/solidworks-api/document/macro-feature/) can do just that. It can be even added directly to the model and shared with anyone and the code will still run without any additional steps.

Developers, who are familiar with macro features would know that it provides the handles for 3 events: regeneration, editing definition and security event (which is sent on each state change).

~~~ vb
Function swmRebuild(varApp As Variant, varDoc As Variant, varFeat As Variant) As Variant
    swmRebuild = True
End Function

Function swmEditDefinition(varApp As Variant, varDoc As Variant, varFeat As Variant) As Variant
    swmEditDefinition = True
End Function

Function swmSecurity(varApp As Variant, varDoc As Variant, varFeat As Variant) As Variant
    swmSecurity = SwConst.swMacroFeatureSecurityOptions_e.swMacroFeatureSecurityByDefault
End Function
~~~

But there is no Load Event? So how can we run code only one time when feature is loaded.

For that we can use the Security notification as feature load is a state change. There is more tweaks required to distinguish loading event from any other security events as there will be hundreds of them sent during feature lifetime.

{% include img.html src="/res/2019-05-01-run-solidworks-macro-on-model-open/load-watcher-macro-feature.png" alt="Load Watcher macro feature in the feature manager tree" align="center" %}

Follow the [Run VBA macro on model load using macro feature and SOLIDWORKS API](https://www.codestack.net/solidworks-api/document/macro-feature/model-load-watcher/) for detailed instructions and source code.