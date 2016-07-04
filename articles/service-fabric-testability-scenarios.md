<properties
   pageTitle="混沌和故障转移测试 | Azure"
   description="使用 Service Fabric 混沌测试和故障转移测试方案来引发故障，然后验证服务的可靠性。"
   services="service-fabric"
   documentationCenter=".net"
   authors="anmolah"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/25/2016"
   wacn.date="07/04/2016"/>

# 可测试性方案
大型分布式系统，例如云基础结构，在本质上都是不可靠的。Azure Service Fabric 使开发人员能够编写出可以在不可靠基础结构上运行的服务。若要编写高质量的服务，开发人员需要能够引入这种不可靠的基础结构来测试其服务的稳定性。

故障分析服务使开发人员能够引入故障操作，在存在故障的情况下测试服务。然而，定向模拟故障只能做到这种程度。若要进一步测试，可以使用 Service Fabric 中的测试方案：混沌测试和故障转移测试。这些方案在整个群集模拟连续交叉出现的故障，包括常规故障和非常规故障，时间跨度很长。为测试配置故障率和故障类型后，就可以通过 C# API 或 PowerShell 启动该测试，以在群集和服务中生成故障。

## 混沌测试
混沌方案跨整个 Service Fabric 群集生成故障。一般而言，该方案将几个月或几年经历的故障压缩到几小时。各种交叉故障的组合并具有高故障率，能够找出很有可能被忽视的极端状况。这会使服务的代码质量得到显著提高。

### 在混沌测试中模拟的故障
 - 重新启动节点
 - 重新启动已部署的代码包
 - 删除副本
 - 重新启动副本
 - 移动主副本（可选）
 - 移动辅助副本（可选）

混沌测试在指定时间段内运行故障和群集验证的多次循环。让群集达到稳定状态以及验证成功所用的时间也是可配置的。若你在群集验证中遇到一次故障，则方案失败。

例如，考虑一个设置为运行上小时且最多三个并发故障的测试。该测试将引入三个故障，然后验证群集运行状况。该测试将循环进行以上步骤，直到群集变得不正常或一小时的时间已过。如果在任何一次迭代中群集变得不正常，即在配置的时间内不稳定，则测试将引发异常而失败。此异常指出系统出现问题，并且需要进一步调查。

当前，混沌测试故障生成引擎只引入安全故障。这意味着在没有外部故障的情况下，绝对不会出现仲裁丢失或数据丢失。

### 重要的配置选项
 - **TimeToRun**：测试在成功完成之前将要运行的总时间。测试也可以提前完成来代替验证失败。
 - **MaxClusterStabilizationTimeout**：在测试失败之前，等待群集变得正常的最长时间。执行的检查包括群集运行状况是否正常、服务运行状况是否正常、对于服务分区而言目标副本集是否达到设定的大小以及是否不存在 InBuild 副本。
 - **MaxConcurrentFaults**：每次循环中并发故障的最大数量。数字越大，测试越激进，从而导致故障转移和转换组合更复杂。在没有外部故障的情况下，测试保证不管此配置有多大，都不会出现仲裁丢失或数据丢失。
 - **EnableMoveReplicaFaults**：启用或禁用导致主副本或辅助副本移动的故障。默认情况下，这些故障处于禁用状态。
 - **WaitTimeBetweenIterations**：循环之间（即在一轮故障和对应的验证之后）等待的时间量。

### 如何运行混沌测试
C# 示例

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunChaosTestScenarioAsync(clusterConnection).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunChaosTestScenarioAsync(string clusterConnection)
    {
        TimeSpan maxClusterStabilizationTimeout = TimeSpan.FromSeconds(180);
        uint maxConcurrentFaults = 3;
        bool enableMoveReplicaFaults = true;

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        ChaosTestScenarioParameters scenarioParameters = new ChaosTestScenarioParameters(
          maxClusterStabilizationTimeout,
          maxConcurrentFaults,
          enableMoveReplicaFaults,
          timeToRun);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        ChaosTestScenario chaosScenario = new ChaosTestScenario(fabricClient, scenarioParameters);

        try
        {
            await chaosScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```

PowerShell

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricChaosTestScenario -TimeToRunMinute $timeToRun -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -MaxConcurrentFaults $concurrentFaults -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec
```


## 故障转移测试

故障转移测试方案是混沌测试方案针对特定服务分区的一个版本。它在特定服务分区上测试故障转移的效果，同时让其他服务不受影响。一旦配置好目标分区信息和其他参数，它就可以作为一个客户端工具运行，使用 C# API 或 PowerShell 生成针对一个服务分区的故障。该方案遍历一系列的模块故障和服务验证，同时你的业务逻辑在一边继续运行以提供工作负荷。服务验证失败指出存在需要进一步调查的问题。

### 在故障转移测试中模拟的故障
- 重新启动分区所在的已部署代码包
- 删除主/辅助副本或无状态实例
- 重新启动主/辅助副本（如果是持久化服务）
- 移动主副本
- 移动辅助副本
- 重新启动分区

故障转移测试工作引入一个经过选择的故障，然后在服务上运行验证以确保其稳定性。与混沌测试中可能出现多个故障相比，故障转移测试一次仅引入一个故障。如果在每次故障之后，服务分区在配置的等待时间内没有达到稳定，则测试失败。测试只引入安全故障。这意味着在没有外部故障的情况下，将不会出现仲裁丢失或数据丢失。

### 重要的配置选项
 - **PartitionSelector**：选择器对象，指定需要作为目标的分区。
 - **TimeToRun**：测试在完成之前将要运行的总时间。
 - **MaxServiceStabilizationTimeout**：在测试失败之前，等待群集变得正常的最长时间。执行的检查包括服务运行状况是否正常、对于所有分区而言目标副本集是否达到设定的大小以及是否不存在 InBuild 副本。
 - **WaitTimeBetweenFaults**：每次故障和验证循环之间等待的时间量。

### 如何运行故障转移测试

**C#**

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunFailoverTestScenarioAsync(clusterConnection, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunFailoverTestScenarioAsync(string clusterConnection, Uri serviceName)
    {
        TimeSpan maxServiceStabilizationTimeout = TimeSpan.FromSeconds(180);
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        FailoverTestScenarioParameters scenarioParameters = new FailoverTestScenarioParameters(
          randomPartitionSelector,
          timeToRun,
          maxServiceStabilizationTimeout);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        FailoverTestScenario failoverScenario = new FailoverTestScenario(fabricClient, scenarioParameters);

        try
        {
            await failoverScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```


**PowerShell**

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$waitTimeBetweenFaultsSec = 10
$serviceName = "fabric:/SampleApp/SampleService"

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricFailoverTestScenario -TimeToRunMinute $timeToRun -MaxServiceStabilizationTimeoutSec $maxStabilizationTimeSecs -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ServiceName $serviceName -PartitionKindSingleton
```

<!---HONumber=Mooncake_0503_2016-->