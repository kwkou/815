<properties linkid="develop-media-services-how-to-guides-check-job-progress" urlDisplayName="Check Job Progress" pageTitle="How to Check Job Progress in Media Services - Azure" metaKeywords="" description="Learn how to use event handler code to track job progress and send status updates. Code samples are written in C# and use the Media Services SDK for .NET." metaCanonical="" services="media-services" documentationCenter="" title="How to: Check Job Progress" authors="migree" solutions="" manager="" editor="" />

如何：检查作业进度
==================

本文是介绍 Azure Media Services 编程的系列主题中的一篇。前一个主题是[如何：对资产进行编码](http://go.microsoft.com/fwlink/?LinkID=301753&clcid=0x409)。

当你运行作业时，通常需要采用某种方式来跟踪作业进度。以下代码示例定义了 StateChanged 事件处理程序。此事件处理程序将跟踪作业进度，并根据现状提供更新的状态。该代码还定义了 LogJobStop 方法。此帮助器方法将记录错误详细信息。

``` {}
private static void StateChanged(object sender, JobStateChangedEventArgs e)
{
Console.WriteLine("Job state changed event:");
Console.WriteLine("  Previous state:" + e.PreviousState);
Console.WriteLine("  Current state:" + e.CurrentState);

switch (e.CurrentState)
    {
case JobState.Finished:
Console.WriteLine();
Console.WriteLine("********************");
Console.WriteLine("Job is finished.");
Console.WriteLine("Please wait while local tasks or downloads complete...");
Console.WriteLine("********************");
Console.WriteLine();
Console.WriteLine();
break;
case JobState.Canceling:
case JobState.Queued:
case JobState.Scheduled:
case JobState.Processing:
Console.WriteLine("Please wait...\n");
break;
case JobState.Canceled:
case JobState.Error:
// Cast sender as a job.
IJob job = (IJob)sender;
// Display or log error details as needed.
LogJobStop(job.Id);
break;
default:
break;
    }
}

private static void LogJobStop(string jobId)
{
StringBuilder builder = new StringBuilder();
IJob job = GetJob(jobId);

builder.AppendLine("\nThe job stopped due to cancellation or an error.");
builder.AppendLine("***************************");
builder.AppendLine("Job ID:" + job.Id);
builder.AppendLine("Job Name:" + job.Name);
builder.AppendLine("Job State:" + job.State.ToString());
builder.AppendLine("Job started (server UTC time):" + job.StartTime.ToString());
builder.AppendLine("Media Services account name:" + _accountName);
builder.AppendLine("Media Services account location:" + _accountLocation);
// Log job errors if they exist.  
if (job.State == JobState.Error)
    {
builder.Append("Error Details:\n");
foreach (ITask task in job.Tasks)
        {
foreach (ErrorDetail detail in task.ErrorDetails)
            {
builder.AppendLine("  Task Id:" + task.Id);
builder.AppendLine("    Error Code:" + detail.Code);
builder.AppendLine("    Error Message:" + detail.Message + "\n");
            }
        }
    }
builder.AppendLine("***************************\n");
// Write the output to a local file and to the console.The template 
// for an error output file is:JobStop-{JobId}.txt
string outputFile = _outputFilesFolder + @"\JobStop-" + JobIdAsFileName(job.Id) + ".txt";
WriteToFile(outputFile, builder.ToString());
Console.Write(builder.ToString());
}

private static string JobIdAsFileName(string jobID)
{
return jobID.Replace(":", "_");
}
```

后续步骤
========

了解如何创建作业并跟踪其进度后，下一步就是保护资产。有关详细信息，请参阅[如何使用 Azure Media Services 保护资产](http://go.microsoft.com/fwlink/?LinkID=301813&clcid=0x409)。

