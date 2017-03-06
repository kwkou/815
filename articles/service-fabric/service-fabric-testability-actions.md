<properties
    pageTitle="Azure 微服务中的模拟故障 | Azure"
    description="本文介绍了 Microsoft Azure Service Fabric 中的可测试性操作。"
    services="service-fabric"
    documentationcenter=".net"
    author="motanv"
    manager="timlt"
    editor="toddabel" />
<tags
    ms.assetid="ed53ca5c-4d5e-4b48-93c9-e386f32d8b7a"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/19/2017"
    wacn.date="03/03/2017"
    ms.author="motanv;heeldin" />  

# 可测试性操作
为了模拟一个不可靠的基础结构，Azure Service Fabric 向开发人员提供了众多方式来模拟各种现实世界故障和状态转换。这些方式被称为可测试性操作。这些操作属于低级别 API，会导致具体的故障注入、状态转换或验证。结合使用这些操作，可为服务编写全面的测试方案。

Service Fabric 提供某些由这些操作组成的常见测试方案。强烈建议使用这些内置方案，这些方案经过精心挑选，用于测试常见状态转换和故障案例。但是，希望添加尚未包含在内置方案中的方案时，或者需要为应用程序量身订做一个方案时，可以使用这些操作来创建自定义测试方案。

System.Fabric.dll 程序集中包含了这些操作的 C# 实现。Microsoft.ServiceFabric.Powershell.dll 程序集中包含了 System Fabric PowerShell 模块。运行时安装过程中，将安装 ServiceFabric PowerShell 模块以便易于使用。

##<a name="graceful-vs-ungraceful-fault-actions"></a>常规故障与非常规故障操作
可测试性操作分为两个主要的类型：

* 非常规故障：这些故障诸如计算机重启和进程崩溃等故障。在此类故障情形下，进程的执行上下文将突然停止。这意味着在该应用程序重新启动之前，无法运行任何状态清理。

* 常规错误：这些故障模拟由负载均衡触发的副本移动和删除等常规操作。在此类情形下，服务收到关闭通知并且可以在退出前清理状态。

为了实现更好的质量验证，在引入各种常规故障和非常规故障的情况下运行服务和业务工作负荷。非常规故障执行服务进程在某些工作流程中实然退出的方案。这会在 Service Fabric 还原服务副本时立即测试恢复路径。这将有助于测试数据一致性，以及是否在故障之后正确维护服务状态。另一组故障，即常规故障，测试服务是否对 Service Fabric 正在移动的副本做出正确的反应。这会测试 RunAsync 方法中的取消处理。服务需要检查是否正在设置取消标记、正确保存其状态及退出 RunAsync 方法。

## 可测试性操作列表

| 操作 | 说明 | 托管 API | PowerShell cmdlet | 常规/非常规故障 |
|---------|-------------|-------------|-------------------|------------------------------|
|CleanTestState| 在测试驱动器非正常关闭时从群集删除所有测试状态。 | CleanTestStateAsync | Remove-ServiceFabricTestState | 不适用 |
| InvokeDataLoss | 将数据丢失引入到服务分区。 | InvokeDataLossAsync | Invoke-ServiceFabricPartitionDataLoss | 常规 |
| InvokeQuorumLoss | 将给定有状态服务分区放入仲裁丢失中。 | InvokeQuorumLossAsync | Invoke-ServiceFabricQuorumLoss | 常规 |
| Move Primary | 将有状态服务的指定主副本移动到指定群集节点。 | MovePrimaryAsync | Move-ServiceFabricPrimaryReplica | 常规 |
| Move Secondary | 将有状态服务的当前辅助副本移动到另一个群集节点。 | MoveSecondaryAsync | Move-ServiceFabricSecondaryReplica | 常规 |
| RemoveReplica | 通过从群集删除副本来模拟副本故障。这将关闭该副本，将其转换为角色“None”，并从群集中删除其所有状态。 | RemoveReplicaAsync | Remove-ServiceFabricReplica | 常规 |
| RestartDeployedCodePackage | 通过重新启动部署在群集中某个节点上的代码包来模拟代码包进程故障。这将中止代码包进程，并会重新启动驻留在该进程中的所有用户服务副本。 | RestartDeployedCodePackageAsync | Restart-ServiceFabricDeployedCodePackage | 非常规 |
| RestartNode | 通过重新启动某个节点来模拟 Service Fabric 群集节点故障。 | RestartNodeAsync | Restart-ServiceFabricNode | 非常规 |
| RestartPartition | 通过重新启动分区的某些或全部副本来模拟数据中心中断或群集中断方案。 | RestartPartitionAsync | Restart-ServiceFabricPartition | 常规 |
| RestartReplica | 通过重新启动群集中的某个持久化副本、关闭副本然后重新打开该副本来模拟副本故障。 | RestartReplicaAsync | Restart-ServiceFabricReplica | 常规 |
| StartNode | 启动群集中已经停止的节点。 | StartNodeAsync | Start-ServiceFabricNode | 不适用 |
| StopNode | 通过停止群集中的某个节点来模拟节点故障。调用 StartNode 之前，该节点将一直处于关闭状态。 | StopNodeAsync | Stop-ServiceFabricNode | 非常规 |
| ValidateApplication | 验证某个应用程序内所有 Service Fabric 服务的可用性和运行状况，通常在将某些故障引入系统之后。 | ValidateApplicationAsync | Test-ServiceFabricApplication | 不适用 |
| ValidateService | 验证某个 Service Fabric 服务的可用性和运行状况，通常在将某些故障引入系统之后。 | ValidateServiceAsync | Test-ServiceFabricService | 不适用 |

## 使用 PowerShell 运行可测试性操作

本教程说明如何使用 PowerShell 运行可测试性操作。可了解如何针对本地（也称为“单机”）群集或 Azure 群集运行可测试性操作。安装 Microsoft Service Fabric MSI 时，会自动安装 Microsoft.Fabric.Powershell.dll（Service Fabric PowerShell 模块）。该模块在打开 PowerShell 提示符时自动加载。

教程章节：

- [针对单机群集运行操作](#run-an-action-against-a-one-box-cluster)
- [针对 Azure 群集运行操作](#run-an-action-against-an-azure-cluster)

###<a id="run-an-action-against-a-one-box-cluster"></a> 针对单机群集运行操作

若要针对本地群集运行可测试性操作，首先需要连接到群集并且应在管理员模式下打开 PowerShell 提示符。让我们看一下 **Restart-ServiceFabricNode** 操作。


Restart-ServiceFabricNode -NodeName Node1 -CompletionMode DoNotVerify


在这里，操作 **Restart-ServiceFabricNode** 在一个名为“Node1”的节点上运行。完成模式指定不应该验证重启节点操作是否实际成功。将完成模式指定为“Verify”会让其验证重启操作是否实际成功。除了按其名称直接指定节点以外，还可以通过分区键和副本类型指定节点，如下所示：


	Restart-ServiceFabricNode -ReplicaKindPrimary  -PartitionKindNamed -PartitionKey Partition3 -CompletionMode Verify




	$connection = "localhost:19000"
	$nodeName = "Node1"
	
	Connect-ServiceFabricCluster $connection
	Restart-ServiceFabricNode -NodeName $nodeName -CompletionMode DoNotVerify


应使用 **Restart-ServiceFabricNode** 来重新启动群集中的 Service Fabric 节点。这将停止 Fabric.exe 进程，该进程会重启驻留在该节点上的所有系统服务和用户服务副本。使用此 API 来测试服务有助于沿故障转移恢复路径发现 Bug。它帮助模拟群集中的节点故障。

以下屏幕截图显示操作中的 **Restart-ServiceFabricNode** 可测试性命令。

![](./media/service-fabric-testability-actions/Restart-ServiceFabricNode.png)

第一个 **Get-ServiceFabricNode**（来自 Service Fabric PowerShell 模块的一个 cmdlet）的输出显示本地群集有五个节点：Node.1 至 Node.5。在名为 Node.4 的节点上执行可测试性操作 (cmdlet) **Restart-ServiceFabricNode** 之后，我们看到该节点的正常运行时间已被重置。

###<a id="run-an-action-against-an-azure-cluster"></a> 针对 Azure 群集运行操作

针对 Azure 群集运行可测试性操作（使用 PowerShell）与针对本地群集运行操作类似。唯一的区别在于：能够运行操作之前，不是连接到本地群集，而是需要首先连接到 Azure 群集。

## 使用 C# 运行可测试性操作；

若要使用 C# 运行可测试性操作，首先需要使用 FabricClient 连接到群集。然后获取运行该操作所需的参数。可用不同的参数来运行相同的操作。请看一看 RestartServiceFabricNode 操作，运行该操作的方式之一是在群集中使用节点信息（节点名称和节点实例 ID）。


	RestartNodeAsync(nodeName, nodeInstanceId, completeMode, operationTimeout, CancellationToken.None)


参数说明：

- **CompleteMode** 指定该模式不应该验证重启操作是否实际成功。将完成模式指定为“Verify”会让其验证重启操作是否实际成功。
- **OperationTimeout** 设置在引发 TimeoutException 异常之前等待操作完成的时间量。
- **CancellationToken** 允许取消挂起调用。

除了按其名称直接指定节点以外，还可以通过分区键和副本类型指定节点。

有关更多信息，请参阅 [PartitionSelector 和 ReplicaSelector](#partition_replica_selector)。



	// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using System.Fabric.Testability;
	using System.Fabric;
	using System.Threading;
	using System.Numerics;
	
	class Test
	{
	    public static int Main(string[] args)
	    {
	        string clusterConnection = "localhost:19000";
	        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");
	        string nodeName = "N0040";
	        BigInteger nodeInstanceId = 130743013389060139;
	
	        Console.WriteLine("Starting RestartNode test");
	        try
	        {
	            //Restart the node by using ReplicaSelector
	            RestartNodeAsync(clusterConnection, serviceName).Wait();
	
	            //Another way to restart node is by using nodeName and nodeInstanceId
	            RestartNodeAsync(clusterConnection, nodeName, nodeInstanceId).Wait();
	        }
	        catch (AggregateException exAgg)
	        {
	            Console.WriteLine("RestartNode did not complete: ");
	            foreach (Exception ex in exAgg.InnerExceptions)
	            {
	                if (ex is FabricException)
	                {
	                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
	                }
	            }
	            return -1;
	        }
	
	        Console.WriteLine("RestartNode completed.");
	        return 0;
	    }
	
	    static async Task RestartNodeAsync(string clusterConnection, Uri serviceName)
	    {
	        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);
	        ReplicaSelector primaryofReplicaSelector = ReplicaSelector.PrimaryOf(randomPartitionSelector);
	
	        // Create FabricClient with connection and security information here
	        FabricClient fabricclient = new FabricClient(clusterConnection);
	        await fabricclient.FaultManager.RestartNodeAsync(primaryofReplicaSelector, CompletionMode.Verify);
	    }
	
	    static async Task RestartNodeAsync(string clusterConnection, string nodeName, BigInteger nodeInstanceId)
	    {
	        // Create FabricClient with connection and security information here
	        FabricClient fabricclient = new FabricClient(clusterConnection);
	        await fabricclient.FaultManager.RestartNodeAsync(nodeName, nodeInstanceId, CompletionMode.Verify);
	    }
	}


##<a name="partition_replica_selector"></a> PartitionSelector 和 ReplicaSelector

### PartitionSelector
PartitionSelector 是在可测试性中运用的一个帮助程序，用于选择在其上执行任何可测试性操作的具体分区。如果事先知道分区 ID，则它可用于选择具体分区。或者，可提供分区键，操作将在内部解析分区 ID。还可以选择一个随机分区。

若要使用此帮助器，请创建 PartitionSelector 对象，并使用 Select* 方法之一选择分区。然后在 PartitionSelector 对象中将其传递给需要它的 API。如果未选择任何选项，则默认为随机分区。


	Uri serviceName = new Uri("fabric:/samples/InMemoryToDoListApp/InMemoryToDoListService");
	Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
	string partitionName = "Partition1";
	Int64 partitionKeyUniformInt64 = 1;
	
	// Select a random partition
	PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);
	
	// Select a partition based on ID
	PartitionSelector partitionSelectorById = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);
	
	// Select a partition based on name
	PartitionSelector namedPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionName);
	
	// Select a partition based on partition key
	PartitionSelector uniformIntPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionKeyUniformInt64);


### ReplicaSelector
ReplicaSelector 是在可测试性中运用的一个帮助程序，用于帮助选择在其上执行任何可测试性操作的副本。如果事先知道副本 ID，则它可用于选择具体副本。此外，可以选择主副本，也可以选择随机辅助副本。ReplicaSelector 派生于 PartitionSelector，因此你需要同时选择要在其上执行可测试性操作的副本和分区。

若要使用此帮助器，请创建一个 ReplicaSelector 对象，并设置副本的分区的选择方式。然后，可将其传递给需要它的 API。如果未选择任何选项，则默认为随机副本和随机分区。


	Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
	PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);
	long replicaId = 130559876481875498;
	
	// Select a random replica
	ReplicaSelector randomReplicaSelector = ReplicaSelector.RandomOf(partitionSelector);
	
	// Select the primary replica
	ReplicaSelector primaryReplicaSelector = ReplicaSelector.PrimaryOf(partitionSelector);
	
	// Select the replica by ID
	ReplicaSelector replicaByIdSelector = ReplicaSelector.ReplicaIdOf(partitionSelector, replicaId);
	
	// Select a random secondary replica
	ReplicaSelector secondaryReplicaSelector = ReplicaSelector.RandomSecondaryOf(partitionSelector);


## 后续步骤

- [可测试性方案](/documentation/articles/service-fabric-testability-scenarios/)
- 如何测试服务
   - [在服务工作负荷期间模拟故障](/documentation/articles/service-fabric-testability-workload-tests/)
   - [服务到服务通信失败](/documentation/articles/service-fabric-testability-scenarios-service-communication/)
 

<!---HONumber=Mooncake_0227_2017-->