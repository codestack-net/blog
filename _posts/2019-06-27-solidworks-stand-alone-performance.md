---
layout: post
title: 'SOLIDWORKS Stand-Alone API: do not compromise on performance'
description: Comparison of various methods of improving the stand-alone invoking of SOLIDWORKS API. Explanation of the most efficient way of scheduling work to be invoked in-process
image: /res/2019-06-25-solidworks-stand-alone-performance/solidworks-apps-performance.png
labels: stand-alone,performance,command in progress
---
{% include img.html src="/res/2019-06-25-solidworks-stand-alone-performance/solidworks-apps-performance.png" width=250 alt="SOLIDWORKS applications performance" align="center" %}

## Preamble

If you have developed applications for SOLIDWORKS as [stand-alone (out-of-process)](https://www.codestack.net/solidworks-api/getting-started/stand-alone/) you might have noticed the significant execution performance reduction compared to [add-ins (in-process)](https://www.codestack.net/solidworks-api/getting-started/add-ins/). This is especially noticeable when developing C# or VB.NET applications. In some cases performance can be more than hundred times slower in stand-alone compared to add-ins.

In most cases stand-alone application is used as a part of a larger automation where certain parameters need to be passed to enable SOLIDWORKS API automation. Depending on the circumstances the performance drop can be simply not acceptable.

There are several tips & tricks can be used to optimize the performance, such as using of [ISldWorks::CommandInProgress](https://help.solidworks.com/2016/English/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ISldWorks~CommandInProgress.html), [IModelView::EnableGraphicsUpdate](http://help.solidworks.com/2019/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.imodelview~enablegraphicsupdate.html), etc. But none of the above would match the performance of in-process invoking.

## Solution

When trying to solve this problem you might think that the possible solution would be to create an add-in which will be used as a service which will expose public APIs. Fortunately add-in interface can be accessed with [ISldWorks::GetAddInObject](http://help.solidworks.com/2014/English/api/sldworksapi/SolidWorks.Interop.sldworks~SolidWorks.Interop.sldworks.ISldWorks~GetAddInObject.html) SOLIDWORKS API method so its methods could be called from the stand-alone application. That should work, right? At least I though so, but unfortunately this approach would still be executed out-of-process and would not have any performance benefits compared to a traditional stand-alone invocation.

So, is there a solution for this problem? Fortunately **Yes**, there is one. The job needs to be scheduled by out of process application and dispatched by add-in from the in-process. But how to do a scheduling? The best option which I have found is using the [OnIdle Notification](http://help.solidworks.com/2015/english/api/sldworksapi/solidworks.interop.sldworks~solidworks.interop.sldworks.dsldworksevents_onidlenotifyeventhandler.html).

Please find the detailed example for this approach: [In-Process invoking of SOLIDWORKS add-in API from out-of-process applications](https://www.codestack.net/solidworks-api/getting-started/inter-process-communication/invoke-add-in-functions/in-process-invoking/)

## Experiments

Now, let's conduct several experiments to measure the performance. Experiments will be executed for 3 data sets: small, large, extra large using 4 approaches: add-in (baseline performance), stand-alone, stand-alone with CommandInProgress option, stand-alone (scheduled) (solution described above). For the best results SOLIDWORKS is restarted after each experiment and operating system will be maintained in the same state (no new applications started etc.)

### Experiment 1: Filling general table

In first experiment we will be adding and filling the general table in the drawing document. We will run test for filling 100, 400 and 1600 cells.

{% include img.html src="res/2019-06-25-solidworks-stand-alone-performance/general-table.png" alt="General table with cell data" align="center" %}

The code used for this experiment is below.

~~~ cs
public void FillTable(IModelDoc2 model, int size)
{
    var start = DateTime.Now;
    {
        model.IActiveView.EnableGraphicsUpdate = false;
        
        var table = model.Extension.InsertGeneralTableAnnotation(false, 0, 0,
            (int)swBOMConfigurationAnchorType_e.swBOMConfigurationAnchor_BottomLeft, "", size, size);

        for (int i = 0; i < size; i++)
        {
            for (int j = 0; j < size; j++)
            {
                table.Text[i, j] = (i * size + j + 1).ToString();
            }
        }

        model.IActiveView.EnableGraphicsUpdate = true;
    }
    Trace.WriteLine($"Completed in {DateTime.Now.Subtract(start).TotalSeconds} seconds");
}
~~~

The results are displayed in the following chart:

{% include img.html src="res/2019-06-25-solidworks-stand-alone-performance/table-fill-performance-chart.png" alt="Performance comparison for filling general table cells task" align="center" %}

The execution time for in-process application is linear and proportional to the number of cells, however performance of the stand-alone applications is not linear and has a major reduction at the 400 cells.

Although the approach with scheduling the work in OnIdle notification demonstrated better result than traditional stand-alone application the difference is not significant. However the same functionality produced different results for another user as described in [this forum thread](https://forum.solidworks.com/message/966337). However I was not able to reproduce this in my experiments.

### Experiment 2: Indexing faces

Let's continue and run another experiment. We will use the following code which will traverse all the components, bodies and faces of the assembly and extract some geometrical information. We will run this on the models with 200, 600 and 17000 faces.

~~~ cs
public void IndexFaces(IAssemblyDoc assm)
{
    var start = DateTime.Now;
    {
        var comps = assm.GetComponents(false) as object[];
        
        if (comps != null)
        {
            foreach (IComponent2 comp in comps)
            {
                object bodyInfo;
                var bodies = comp.GetBodies3((int)swBodyType_e.swAllBodies, out bodyInfo) as object[];

                if (bodies != null)
                {
                    foreach (IBody2 body in bodies)
                    {
                        var faces = body.GetFaces() as object[];

                        if (faces != null)
                        {
                            foreach (IFace2 face in faces)
                            {
                                var surf = face.IGetSurface();
                                var type = (swSurfaceTypes_e)surf.Identity();
                                
                                Trace.WriteLine($"Area: {face.GetArea()}. Type: {type}");
                            }
                        }
                    }
                }
            }
        }
    }
    Trace.WriteLine($"Completed in {DateTime.Now.Subtract(start).TotalSeconds} seconds");
}
~~~

Indexing 200 faces in add-in took 0.12 seconds, while it took more than 24 seconds in the stand-alone and 1.56 seconds in stand-alone with CommandInProgress option enabled. The numbers are more shocking for the large assembly with over 17,000 faces as it took 3.43 seconds to index from the add-in, but almost 3 minutes (159.83 seconds) in stand-alone with CommandInProgress option enabled and more than 50 minutes in the stand-alone (3024.57 seconds).

To better emphasize the difference the following chart is created based on the performance drop ratio compared to a baseline (add-in). As you can see stand-alone application performance was 209-882 times slower than the add-in, while stand-alone with command in progress 14-47 times slower (which is also not acceptable). While scheduled execution shown only 2-4 times performance drop.

{% include img.html src="res/2019-06-25-solidworks-stand-alone-performance/faces-index-performance-chart.png" alt="Performance comparison for indexing faces task" align="center" %}

## Summary

* It was experimentally proven that the best performance benefits of execution of stand-alone application is achieved by [in-process invoking of SOLIDWORKS add-in API from out-of-process applications](https://www.codestack.net/solidworks-api/getting-started/inter-process-communication/invoke-add-in-functions/in-process-invoking/).
* For some experiments the above approach proven to be even faster than in-process invoking.
* The performance benefits of the in-process invoking from stand-alone application varies depending on the SOLIDWORKS API used. And can provide minimal benefits (for the task such as filling the table) to maximum benefits (for geometry traversing)