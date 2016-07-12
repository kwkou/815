<properties
   pageTitle="使用事件跟踪进行故障排除 | Azure"
   description="在 Azure Service Fabric 上部署服务时遇到的最常见问题。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mattrowmsft"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="03/31/2016"
   wacn.date="07/04/2016"/>

# 排查在 Azure Service Fabric 上部署服务时遇到的常见问题

当你在开发人员计算机上运行服务时，可以轻松使用 [Visual Studio 的调试工具](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)。对于远程群集而言，[运行状况报告](/documentation/articles/service-fabric-view-entities-aggregated-health/)始终是一个不错的起点。访问这些报告的最简单方法是使用 PowerShell 或 [SFX](/documentation/articles/service-fabric-visualizing-your-cluster/)。本文假设你要调试远程群集，并且基本了解如何使用这些工具。

##应用程序崩溃
“分区低于目标副本或实例计数”报告很好地反映了你的服务即将崩溃。若要找出服务发生崩溃的位置，需要稍作一番调查。如果服务的运行强度很高，那么，你最好有一组详尽的跟踪数据。我们建议你尝试使用 [Azure 诊断](/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/)来收集这些跟踪，并[弹性搜索](/documentation/articles/service-fabric-diagnostic-how-to-use-elasticsearch/)等解决方案来查看和搜索跟踪。

![SFX 分区运行状况](./media/service-fabric-diagnostics-troubleshoot-common-scenarios/crashNewApp.png)

###在服务或执行组件初始化期间
在服务类型初始化之前发生的任何异常都将导致进程崩溃。对于这些类型的崩溃，应用程序事件日志将显示服务中的错误。
这是服务初始化之前遇到的最常见异常。

**System.IO.FileNotFoundException**

此错误通常是因为缺少程序集依赖项导致的。请检查 Visual Studio 中的 CopyLocal 属性，或节点的全局程序集缓存。

**System.Runtime.InteropServices.COMException** at System.Fabric.Interop.NativeRuntime+IFabricRuntime.RegisterStatefulServiceFactory(IntPtr, IFabricStatefulServiceFactory)
 
 这表明注册的服务类型名称与服务清单不匹配。

可将 [Azure 诊断](/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/)配置为自动上载所有节点的应用程序事件日志。

###RunAsync() 或 OnActivateAsync()
如果崩溃发生在初始化或运行已注册的服务类型或执行组件期间，Azure Service Fabric 将捕获该异常。可以通过“后续步骤”部分中所述的 EventSource 提供程序查看这些异常。

## 后续步骤

详细了解 Service Fabric 提供的现有诊断信息：

* [Reliable Actors 诊断](/documentation/articles/service-fabric-reliable-actors-diagnostics/)
* [Reliable Services 诊断](/documentation/articles/service-fabric-reliable-services-diagnostics/)

<!---HONumber=Mooncake_0425_2016-->