<properties
   pageTitle="如何在 Service Fabric 服务中调用数据丢失 | Azure"
   description="介绍如何使用数据丢失 API"
   services="service-fabric"
   documentationCenter=".net"
   authors="LMWF"
   manager="rsinha"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/19/2016"
   wacn.date="08/08/2016"
   ms.author="lemai"/>
   
# 如何在服务中调用数据丢失

>[AZURE.WARNING] 本文档介绍如何导致服务数据丢失，应谨慎使用。

## 介绍
你可以通过调用 StartPartitionDataLossAsync() 在 Service Fabric 服务的分区调用数据丢失。此 API 使用错误注入和分析服务来执行导致数据丢失条件的工作。

## 使用错误注入和分析服务

错误注入和分析服务目前支持以下 API，如下面的图表所示。图表右侧显示相应的 PowerShell cmdlet。如需每个 API 的详细信息，请参阅相应的 MSDN 文档。

| C# API | PowerShell Cmdlet |
|-------------------------------------|-----------------------------------------------:|
|[StartPartitionDataLossAsync][dl] |[Start-ServiceFabricPartitionDataLoss][psdl] |
|[StartPartitionQuorumLossAsync][ql] |[Start-ServiceFabricPartitionQuorumLoss][psql] |
|[StartPartitionRestartAsync][rp] |[Start-ServiceFabricPartitionRestart][psrp] |

## 运行命令的概念性概述

错误注入和分析服务使用异步模型，即通过一个 API（在本文档中称为“启动”API）来启动命令，然后使用“GetProgress”API 来查看此命令的进度，直到此命令达到终止状态，或者你取消了此命令。若要启动命令，请调用相应 API 的“启动”API。当错误注入和分析服务接受请求后，将回到此 API。但是，它不会指示命令运行进度，甚至不会指示命令是否已启动。若要查看命令进度，可调用与此前调用的“启动”API 对应的“GetProgress”API。“GetProgress”API 将返回一个对象，在其“状态”属性中指示命令的当前状态。除非遇到以下条件，否则命令会无限期运行下去：

1.	命令已成功完成。如果你在这种情况下对其调用“GetProgress”，则进度对象的“状态”将为“已完成”。
2.	命令遇到致命错误。如果你在这种情况下对其调用“GetProgress”，则进度对象的“状态”将为“出错”
3.	可通过 [CancelTestCommandAsync][cancel] API 或 [Stop-ServiceFabricTestCommand][cancelps] PowerShell cmdlet 取消该命令。如果你在这种情况下对其调用“GetProgress”，则进度对象的“状态”将为“已取消”或“已强制取消”，具体取决于该 API 的参数。如需更多详细信息，请参阅 [CancelTestCommandAsync][cancel] 的文档。


## 运行命令的详细信息

若要启动命令，请使用预期的参数来调用启动 API。所有启动 API 都有名为 operationId 的 GUID 参数。应跟踪 operationId 参数，因为该参数用于跟踪此命令的进度。必须将该参数传入“GetProgress”API 中，然后才能跟踪命令的进度。OperationId 必须是唯一的。

在成功调用启动 API 以后，应采用循环方式调用 GetProgress API，直到返回的进度对象的“状态”属性为“已完成”。应重试所有 [FabricTransientException 类][fte]和 OperationCanceledException 类。当命令进入终止状态（“已完成”、“出错”或“已取消”）后，返回的进度对象的“结果”属性将包含其他信息。如果状态为“已完成”，Result.SelectedPartition.PartitionId 将包含所选分区 ID。Result.Exception 将为 Null。如果状态为“出错”，则 Result.Exception 会提供错误注入和分析服务无法执行命令的原因。Result.SelectedPartition.PartitionId 将提供所选分区 ID。某些情况下，该命令可能尚未执行到选择分区的进度。这种情况下，PartitionId 将为 0。如果状态为“已取消”，则 Result.Exception 将为 Null。与“出错”情况类型，Result.SelectedPartition.PartitionId 将会提供所选分区 ID，但如果命令尚未执行到相应进度，则 PartitionId 将为 0。另请参阅下面的示例。

下面的示例代码演示了如何先启动某个重启特定分区的命令，然后检查其进度。


	    static async Task PerformDataLossSample()
	    {
	        // Create a unique operation id for the command below
	        Guid operationId = Guid.NewGuid();

	        // Note: Use the appropriate overload for your configuration
	        FabricClient fabricClient = new FabricClient();

	        // The name of the target service
	        Uri targetServiceName = new Uri("fabric:/MyService");

	        // The id of the target partition inside the target service
	        Guid targetPartitionId = new Guid("00000000-0000-0000-0000-000002233445");

	        PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(targetServiceName, targetPartitionId);

	        // Start the command.  Retry OperationCanceledException and all FabricTransientException's.  Note when StartPartitionDataLossAsync completes
	        // successfully it only means the Fault Injection and Analysis Service has saved the intent to perform this work.  It does not say anything about the progress
	        // of the command.
	        while (true)
	        {
	            try
	            {
	                await fabricClient.TestManager.StartPartitionDataLossAsync(operationId, partitionSelector, DataLossMode.FullDataLoss).ConfigureAwait(false);
	                break;
	            }
	            catch (OperationCanceledException)
	            {
	            }
	            catch (FabricTransientException)
	            {
	            }

	            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
	        }

	        PartitionDataLossProgress progress = null;

	        // Poll the progress using GetPartitionDataLossProgressAsync until it is either Completed or Faulted.  In this example, we're assuming
	        // the command won't be cancelled.        

	        while (true)
	        {
	            try
	            {
	                progress = await fabricClient.TestManager.GetPartitionDataLossProgressAsync(operationId).ConfigureAwait(false);
	            }
	            catch (OperationCanceledException)
	            {
	                continue;
	            }
	            catch (FabricTransientException)
	            {
	                continue;
	            }

	            if (progress.State == TestCommandProgressState.Completed)
	            {
	                Console.WriteLine("Command '{0}' completed successfully", operationId);

	                // In a terminal state .Result.SelectedPartition.PartitionId will have the chosen partition
	                Console.WriteLine("  Printing selected partition='{0}'", progress.Result.SelectedPartition.PartitionId);
	                break;
	            }
	            else if (progress.State == TestCommandProgressState.Faulted)
	            {
	                // If State is Faulted, the progress object's Result property's Exception property will have the reason why.
	                Console.WriteLine("Command '{0}' failed with '{1}'", operationId, progress.Result.Exception);
	                break;
	            }
	            else
	            {
	                Console.WriteLine("Command '{0}' is currently Running", operationId);
	            }

	            await Task.Delay(TimeSpan.FromSeconds(5.0d)).ConfigureAwait(false);
	        }
	    }


下面的示例演示了如何使用 PartitionSelector 来选择指定服务的随机分区：


	    static async Task PerformDataLossUseSelectorSample()
	    {
	        // Create a unique operation id for the command below
	        Guid operationId = Guid.NewGuid();

	        // Note: Use the appropriate overload for your configuration
	        FabricClient fabricClient = new FabricClient();

	        // The name of the target service
	        Uri targetServiceName = new Uri("fabric:/SampleService ");

	        // Use a PartitionSelector that will have the Fault Injection and Analysis Service choose a random partition of “targetServiceName”
	        PartitionSelector partitionSelector = PartitionSelector.RandomOf(targetServiceName);

	        // Start the command.  Retry OperationCanceledException and all FabricTransientException's.  Note when StartPartitionDataLossAsync completes
	        // successfully it only means the Fault Injection and Analysis Service has saved the intent to perform this work.  It does not say anything about the progress
	        // of the command.
	        while (true)
	        {
	            try
	            {
	                await fabricClient.TestManager.StartPartitionDataLossAsync(operationId, partitionSelector, DataLossMode.FullDataLoss).ConfigureAwait(false);
	                break;
	            }
	            catch (OperationCanceledException)
	            {
	            }
	            catch (FabricTransientException)
	            {
	            }
	            catch (Exception e)
	            {
	                Console.WriteLine("Unexpected exception '{0}'", e);
	                throw;
	            }

	            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
	        }

	        PartitionDataLossProgress progress = null;

	        // Poll the progress using GetPartitionDataLossProgressAsync until it is either Completed or Faulted.  In this example, we're assuming
	        // the command won't be cancelled.

	        while (true)
	        {
	            try
	            {
	                progress = await fabricClient.TestManager.GetPartitionDataLossProgressAsync(operationId).ConfigureAwait(false);
	            }
	            catch (OperationCanceledException)
	            {
	                continue;
	            }
	            catch (FabricTransientException)
	            {
	                continue;
	            }

	            if (progress.State == TestCommandProgressState.Completed)
	            {
	                Console.WriteLine("Command '{0}' completed successfully", operationId);

	                Console.WriteLine("Printing progress.Result:");
	                Console.WriteLine("  Printing selected partition='{0}'", progress.Result.SelectedPartition.PartitionId);

	                break;
	            }
	            else if (progress.State == TestCommandProgressState.Faulted)
	            {
	                // If State is Faulted, the progress object's Result property's Exception property will have the reason why.
	                Console.WriteLine("Command '{0}' failed with '{1}', SelectedPartition {2}", operationId, progress.Result.Exception, progress.Result.SelectedPartition);
	                break;
	            }
	            else
	            {
	                Console.WriteLine("Command '{0}' is currently Running", operationId);
	            }

	            await Task.Delay(TimeSpan.FromSeconds(1.0d)).ConfigureAwait(false);
	        }
	    }


## 历史记录和截断

在命令进入终止状态以后，其元数据仍会在错误注入和分析服务中保留一定的时间，直至将其删除以节省空间。如果在删除命令后使用该命令的 operationId 来调用“GetProgress”，则会返回 ErrorCode 为 KeyNotFound 的 FabricException。

[dl]: https://msdn.microsoft.com/zh-cn/library/azure/mt693569.aspx
[ql]: https://msdn.microsoft.com/zh-cn/library/azure/mt693558.aspx
[rp]: https://msdn.microsoft.com/zh-cn/library/azure/mt645056.aspx
[psdl]: https://msdn.microsoft.com/zh-cn/library/mt697573.aspx
[psql]: https://msdn.microsoft.com/zh-cn/library/mt697557.aspx
[psrp]: https://msdn.microsoft.com/zh-cn/library/mt697560.aspx
[cancel]: https://msdn.microsoft.com/zh-cn/library/azure/mt668910.aspx
[cancelps]: https://msdn.microsoft.com/zh-cn/library/mt697566.aspx
[fte]: https://msdn.microsoft.com/zh-cn/library/azure/system.fabric.fabrictransientexception.aspx

<!---HONumber=Mooncake_0801_2016-->