<properties
   pageTitle="在 Service Fabric 群集中引入混沌测试 | Azure"
   description="使用故障注入和群集分析服务 API 管理群集中的混沌测试。"
   services="service-fabric"
   documentationCenter=".net"
   authors="motanv"
   manager="rsinha"
   editor="toddabel"/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/19/2016"
   wacn.date="11/28/2016"
   ms.author="motanv"/>  


# 在 Service Fabric 群集中引入受控的混沌测试
大规模分布式系统，例如云基础结构，在本质上都是不可靠的。Azure Service Fabric 可让开发人员在不可靠的基础结构顶层编写可靠的服务。为了编写稳健的服务，开发人员需要能够针对这种不可靠的基础结构引入故障来测试其服务的稳定性。

故障注射和群集分析服务（也称为故障分析服务）使开发人员能够引入故障操作来测试服务。然而，定向模拟故障只能做到这种程度。若要进一步测试，可以使用混沌测试。

混沌测试在整个群集模拟连续交叉出现的故障，包括常规故障和非常规故障，时间跨度很长。使用故障率和故障类型配置混沌测试后，可以通过 C# API 或 PowerShell 启动或停止该测试，在群集和服务中生成故障。

混沌测试在运行时，将生成不同的事件来捕获当前的运行状态。例如，ExecutingFaultsEvent 包含该迭代中正在发生的所有故障。ValidationFailedEvent 包含群集验证期间发现的故障的详细信息。可以调用 GetChaosReportAsync API 获取混沌测试运行报告。

## 在混沌测试中引入的故障
混沌测试在整个 Service Fabric 群集中生成故障，将几个月或几年内出现的故障压缩成几小时。各种交叉故障的组合并具有高故障率，能够找出很有可能被忽视的极端状况。运用这种混沌测试会使服务的代码质量得到显著提高。

混沌测试引入以下类别的故障：

 - 重新启动节点
 - 重新启动已部署的代码包
 - 删除副本
 - 重新启动副本
 - 移动主副本（可配置）
 - 移动辅助副本（可配置）

混沌测试在多个迭代中运行。每个迭代包含指定时间段的故障和群集验证。可以配置使群集达到稳定状态以及成功完成验证所用的时间。如果在群集验证中发现故障，混沌测试将生成并保留一个包含 UTC 时间戳与故障详细信息的 ValidationFailedEvent。

例如，假设某个混沌测试实例设置为运行 1 小时，并且最多有 3 个并发故障。该混沌测试将引入三个故障，然后验证群集运行状况。它会循环访问上一步骤，直到通过 StopChaosAsync API 或者在一小时后将它显式停止。如果在任何一次迭代中群集变得不正常（即在配置的时间内不稳定），混沌测试将生成 ValidationFailedEvent。此事件指明系统出现问题，可能需要进一步调查。

从目前来看，混沌测试只会引入安全的故障。这意味着，在没有外部故障的情况下，绝对不会发生仲裁丢失或数据丢失。

## 重要的配置选项
 - **TimeToRun**：混沌测试在成功完成之前的总运行时间。在混沌测试运行了 TimeToRun 这段时间之前，可以通过 StopChaos API 将它停止。
 - **MaxClusterStabilizationTimeout**：在再次检查群集之前，等待群集变得正常的最长时间。等待这段时间的目的是在恢复时减少群集上的负载。执行的检查包括：
    - 群集运行状况是否正常
    - 服务运行状况是否正常
    - 服务分区是否达到目标副本集大小
    - 不存在 InBuild 副本
 - **MaxConcurrentFaults**：在每个迭代中引入的最大并发故障数。该数字越大，混沌测试就越激进。这会导致故障转移与转换的组合变得更复杂。无论此项配置的值有多大，混沌测试都能保证在没有外部故障的情况下，不会发生仲裁丢失或数据丢失。
 - **EnableMoveReplicaFaults**：启用或禁用导致主副本或辅助副本移动的故障。默认情况下，这些故障处于禁用状态。
 - **WaitTimeBetweenIterations**：每两次迭代（即在一轮故障和对应的验证之后）之间等待的时间。
 - **WaitTimeBetweenFaults**：迭代中每两次连续故障之间的等待时间。

## 如何运行混沌测试
**C#：**


	using System;
	using System.Collections.Generic;
	using System.Threading.Tasks;
	using System.Fabric;

	using System.Diagnostics;
	using System.Fabric.Chaos.DataStructures;

	class Program
	{
	    private class ChaosEventComparer : IEqualityComparer<ChaosEvent>
	    {
	        public bool Equals(ChaosEvent x, ChaosEvent y)
	        {
	            return x.TimeStampUtc.Equals(y.TimeStampUtc);
	        }

	        public int GetHashCode(ChaosEvent obj)
	        {
	            return obj.TimeStampUtc.GetHashCode();
	        }
	    }

	    static void Main(string[] args)
	    {
	        var clusterConnectionString = "localhost:19000";
	        using (var client = new FabricClient(clusterConnectionString))
	        {
	            var startTimeUtc = DateTime.UtcNow;
	            var stabilizationTimeout = TimeSpan.FromSeconds(30.0);
	            var timeToRun = TimeSpan.FromMinutes(60.0);
	            var maxConcurrentFaults = 3;

	            var parameters = new ChaosParameters(
	                stabilizationTimeout,
	                maxConcurrentFaults,
	                true, /* EnableMoveReplicaFault */
	                timeToRun);

	            try
	            {
	                client.TestManager.StartChaosAsync(parameters).GetAwaiter().GetResult();
	            }
	            catch (FabricChaosAlreadyRunningException)
	            {
	                Console.WriteLine("An instance of Chaos is already running in the cluster.");
	            }

	            var filter = new ChaosReportFilter(startTimeUtc, DateTime.MaxValue);

	            var eventSet = new HashSet<ChaosEvent>(new ChaosEventComparer());

	            while (true)
	            {
	                var report = client.TestManager.GetChaosReportAsync(filter).GetAwaiter().GetResult();

	                foreach (var chaosEvent in report.History)
	                {
	                    if (eventSet.add(chaosEvent))
	                    {
	                        Console.WriteLine(chaosEvent);
	                    }
	                }

	                // When Chaos stops, a StoppedEvent is created.
	                // If a StoppedEvent is found, exit the loop.
	                var lastEvent = report.History.LastOrDefault();

	                if (lastEvent is StoppedEvent)
	                {
	                    break;
	                }

	                Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
	            }
	        }
	    }
	}

**PowerShell：**


	$connection = "localhost:19000"
	$timeToRun = 60
	$maxStabilizationTimeSecs = 180
	$concurrentFaults = 3
	$waitTimeBetweenIterationsSec = 60

	Connect-ServiceFabricCluster $connection

	$events = @{}
	$now = [System.DateTime]::UtcNow

	Start-ServiceFabricChaos -TimeToRunMinute $timeToRun -MaxConcurrentFaults $concurrentFaults -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec

	while($true)
	{
	    $stopped = $false
	    $report = Get-ServiceFabricChaosReport -StartTimeUtc $now -EndTimeUtc ([System.DateTime]::MaxValue)

	    foreach ($e in $report.History) {

	        if(-Not ($events.Contains($e.TimeStampUtc.Ticks)))
	        {
	            $events.Add($e.TimeStampUtc.Ticks, $e)
	            if($e -is [System.Fabric.Chaos.DataStructures.ValidationFailedEvent])
	            {
	                Write-Host -BackgroundColor White -ForegroundColor Red $e
	            }
	            else
	            {
	                if($e -is [System.Fabric.Chaos.DataStructures.StoppedEvent])
	                {
	                    $stopped = $true
	                }

	                Write-Host $e
	            }
	        }
	    }

	    if($stopped -eq $true)
	    {
	        break
	    }

	    Start-Sleep -Seconds 1
	}

	Stop-ServiceFabricChaos

<!---HONumber=Mooncake_1121_2016-->