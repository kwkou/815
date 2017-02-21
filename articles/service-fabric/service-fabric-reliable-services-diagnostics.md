<properties
    pageTitle="有状态 Reliable Services 诊断 | Azure"
    description="有状态 Reliable Services 的诊断功能"
    services="service-fabric"
    documentationcenter=".net"
    author="AlanWarwick"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ae0e8f99-69ab-4d45-896d-1fa80ed45659"
    ms.service="Service-Fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/04/2017"
    wacn.date="02/20/2017"
    ms.author="alanwar" />  


# 有状态 Reliable Services 的诊断功能
有状态 Reliable Services StatefulServiceBase 类会发出 [EventSource](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracing.eventsource.aspx) 事件，可将其用于调试服务、提供对运行时运行方式的深入了解以及帮助进行故障排除。

## EventSource 事件
有状态 Reliable Services StatefulServiceBase 类的 EventSource 名称是“Microsoft-ServiceFabric-Services”。当[在 Visual Studio 中调试](/documentation/articles/service-fabric-debugging-your-application/)服务时，来自此事件源的事件在[](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/#view-service-fabric-system-events-in-visual-studio)“诊断事件”窗口中显示。

可帮助收集和/或查看 EventSource 事件的工具和技术的示例包括 [PerfView](http://www.microsoft.com/download/details.aspx?id=28567)、[Microsoft Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)和 [Microsoft TraceEvent 库](http://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent)。

## 事件

|事件名称|事件 ID|级别|事件说明|
|----------|--------|-----|-----------------|
|StatefulRunAsyncInvocation|1|信息性|在服务 RunAsync 任务启动时发出|
|StatefulRunAsyncCancellation|2|信息性|在服务 RunAsync 任务取消时发出|
|StatefulRunAsyncCompletion|3|信息性|在服务 RunAsync 任务完成时发出|
|StatefulRunAsyncSlowCancellation|4|警告|在服务 RunAsync 任务完成取消所用时间过长时发出|
|StatefulRunAsyncFailure|5|错误|在服务 RunAsync 任务引发异常时发出|

## 解释事件

StatefulRunAsyncInvocation、StatefulRunAsyncCompletion 和 StatefulRunAsyncCancellation 事件可用于供服务编写者了解服务的生命周期以及服务启动、取消和完成时的计时。在调试服务问题或了解服务生命周期时，这可能会十分有用。

服务编写者应密切注意 StatefulRunAsyncSlowCancellation 和 StatefulRunAsyncFailure 事件，因为它们指示与服务有关的问题。

服务 RunAsync\(\) 任务引发异常时，都会发出 StatefulRunAsyncFailure。引发的异常通常指示服务中存在错误或 bug。此外，该异常会导致服务失败，从而将服务移到另一个节点。这可能是一种高开销的操作，并会在移动服务时延迟传入请求。服务编写者应确定异常的原因并在可能时缓解它。

每当 RunAsync 任务的取消请求花费的时间超过四秒时，都会发出 StatefulRunAsyncSlowCancellation。服务花费过长时间完成取消时，会影响在另一个节点快速重启服务的能力。这可能会影响服务的总体可用性。

## 后续步骤
* [PerfView 中的 EventSource 提供程序](https://blogs.msdn.microsoft.com/vancem/2012/07/09/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource/)

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: add "后续步骤" section-->