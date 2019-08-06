---
layout: post
title: Displaying Progress Of Long SOLIDWORKS API Operation
description: Overview of different methods of displaying the progress of long running operation in SOLIDWORKS API (User Progress Bar, Windows Forms, background processing, etc.) and performance comparison between methods
image: /res/2019-08-06-solidworks-api-operation-progress/progress.png
labels: progress bar,background
---
{% include img.html src="/res/2019-08-06-solidworks-api-operation-progress/progress.svg" width=350 alt="SOLIDWORKS operation progress bar" align="center" %}

If you are developing SOLIDWORKS macros or add-ins which are performing some long running operations (e.g. calculation, indexing, etc.) it is considered to be a good practice to let your user know that work is in progress and give an approximate estimation of how long the operation would take by rendering a responsive progress bar and updating its message and progress value.

In this blog article I will provide an overview of 5 different approaches of enabling the progress bar and compare the performance of each of those approaches, as at the end of the day it would not be acceptable to significantly increase the execution time of already long operation just to provide the progress.

## Implementation

For the experiments C# add-in is used which provides 5 buttons for each method as well as a toggle button to indicate if progress should be reported at each step or every 1% (so it got only reported 100 times).

{% include img.html src="/res/2019-08-06-solidworks-api-operation-progress/progress-handler-commands.png" alt="Menu and toolbar of the progress handler add-in" align="center" %}

Add-in traverses all the bodies and faces of the active SOLIDWORKS part document. It performs geometry information extraction for each face. In order to emulate long running operation add-in provides the constant which indicates how many times the processing of each face needs to be repeated:

~~~ cs
private const int ITERATIONS_COUNT = 1000;
~~~

The source code of the add-in used in the experiments which implement all the methods can be downloaded from [GitHub](https://github.com/codestackdev/solidworks-api-examples/tree/master/swex/add-in/progress-handler)

## Methods

### Baseline - No Progress Capturing

This is a method which doesn't report a progress and used as a baseline for the performance comparison

### Method A - User Form In Main Thread

This method displays the Windows User Form in the main thread and reports the progress. This approach is not satisfactory as form UI may be locked and artefacts can appear.

{% include img.html src="\res\2019-08-06-solidworks-api-operation-progress\progress-form-frozen.png" alt="Frozen windows form with progress bar when displayed in the main thread" align="center" %}

~~~ cs
private void DoWork()
{
    var form = new ProgressForm();

    form.Show();

    //call APIs and update progress bar in the form

    form.Close();
}
~~~

### Method B - User Progress Bar

This approach is based on progress bar provided by SOLIDWORKS API. This adds the standard progress bar in the bottom left corner of SOLIDWORKS application. And progress is also reflected in the SOLIDWORKS icon in the task bar. For usage example in VBA macro follow [User Progress Bar in VBA macro](https://www.codestack.net/solidworks-api/application/frame/user-progress-bar/) example.

{% include img.html src="\res\2019-08-06-solidworks-api-operation-progress\user-progress-bar.png" alt="SOLIDWORKS user progress bar" align="center" %}

~~~ cs
private void DoWork()
{
    UserProgressBar prgBar;
    App.GetUserProgressBar(out prgBar);

    //call APIs and update progress bar

    prgBar.End();
}
~~~

### Method C - Operation In Background Thread (Task)

This method displays the Windows User Form in the main thread but offloads the operation into a background thread via [System.Threading.Tasks.Task](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task?view=netframework-4.8). Similar functionality can be achieved by using [System.Threading.Thread](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread?view=netframework-4.8).

This approach allows to achieve the responsive form (as the heavy operation is performed in the background).

{% include img.html src="\res\2019-08-06-solidworks-api-operation-progress\progress-form.png" alt="Responsive progress bar in Windows Form in the background process" align="center" %}

As operation is performed in the background, SOLIDWORKS window is also responsive and user can manipulate the model or menu commands, unlike all other methods where the operation is performed in the main thread which locks the SOLIDWORKS window.

~~~ cs
private async void DoWork()
{
    var form = new ProgressForm();

    form.Show();

    await Task.Run(() => 
    {
        //call APIs and update progress bar in the form
    });

    form.Close();
}
~~~

### Method D - Do Events In Main Thread

This approach is based on repetitively calling the [System.Windows.Forms.Application.DoEvents](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.application.doevents?view=netframework-4.8) method to process all Windows messages in the message queue. This means that the UI will be refreshed every time when this method is called which will emulate the responsive User Interface.

~~~ cs
private void DoWork()
{
    var form = new ProgressForm();

    form.Show();

    //call APIs and update progress bar in the form
    //call Application.DoEvents(); after each update of the progress

    form.Close();
}
~~~

### Method E - User Form In Background Thread

In this method the operation is performed in the main thread, but the form is displayed in the background thread. The communication is performed via [System.Windows.Forms.Control.Invoke](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.control.invoke?view=netframework-4.8) method.

~~~ cs
private void DoWork()
{
    ProgressForm form = null;
    
    var th = new Thread(()=> 
    {
        form = new ProgressForm();
        form.ShowDialog();
    });

    th.SetApartmentState(ApartmentState.STA);

    th.Start();

    while (form == null || !form.IsHandleCreated)
    {
        Thread.Sleep(100);
    }

    //call APIs and update progress bar in the form
    //call form.Invoke to update progress from the main thread

    form.Invoke(new Action(() => form.Close()));
}
~~~

## Experiments

The above methods were tested in 4 different experiments based on 2 files

* Large part document with several bodies with over 2000 faces
* Small part with single body and 12 faces

* Experiment 1 - 10 iterations over 2000 faces (1 percent increment, i.e. 100 updates)
* Experiment 2 - 10 iterations over 2000 faces (each step increment, i.e. 20000 updates)
* Experiment 3 - 1000 iterations over 12 faces (1 percent increment, i.e. 100 updates)
* Experiment 4 - 1000 iterations over 12 faces (each step increment, i.e. 12000 updates)

## Results

Below are the table of execution times (in seconds) for each experiment against each method:

|          | Experiment 1 | Experiment 2 | Experiment 3 | Experiment 4 |
|----------|--------------|--------------|--------------|--------------|
| Baseline | 14.95        | 22.2         | 2.05         | 1.72         |
| Method A | 18.32        | 18.53        | 2.71         | 2.61         |
| Method B | 16.12        | 26.31        | 2.15         | 10.28        |
| Method C | 868.53       | 853.77       | 321.021      | 332.49       |
| Method D | 19.44        | 18.11        | 2.7          | 4.03         |
| Method E | 17.65        | 17.62        | 2.11         | 4.72         |

{% include img.html src="\res\2019-08-06-solidworks-api-operation-progress\chart.png" alt="Progress bar methods performance comparison chart" align="center" %}

## Summary

* As expected the best performance was demonstrates in [Method A](#method-a---user-form-in-main-thread) by running the progress form in the main thread. However this approach should not be used as the User Interface was frozen.

* [Method B](#method-b---user-progress-bar) which is utilizing SOLIDWORKS user progress bar is itself performance consuming so it is not recommended to use it to report each step. But if it is used to report every 1% (i.e. maximum of 100 calls) the performance is comparable with other methods.

* [Method D](#method-d---do-events-in-main-thread) and [Method E](#method-e---user-form-in-background-thread) shown similar results

* [Method C](#method-c---operation-in-background-thread-task) demonstrates the worst results with performance reduction in over 100 times in some experiments. This is expected result as call of SOLIDWORKS API from the background thread would be similar to performing of [out-of-process calls](/solidworks-stand-alone-performance).

* If it is required to call thousands of SOLIDWORKS APIs, do not use background process (Task or Thread), you can however use this approach if background operation doesn't communicate with SOLIDWORKS objects, but performing other activity, such as database connection, file creation, web services calls etc.

* Use build-in User Progress Bar to display a simple progress and enable native look and fill

* Use Windows Forms opened in background thread to provide more complex User Interface for the operations progress
