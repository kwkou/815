<properties
	pageTitle="Azure 诊断概述"
	description="使用 Azure 诊断在云服务、虚拟机和 Service Fabric 中进行调试、性能度量、监视和流量分析"
	services="multiple"
	documentationCenter=".net"
	authors="rboucher"
	manager="jwhit"
	editor=""/>  


<tags
	ms.service="multiple"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="10/25/2016"
	wacn.date="12/23/2016"/>  




# 什么是 Azure 诊断
Azure 诊断是 Azure 中可对部署的应用程序启用诊断数据收集的功能。可以使用于自许多不同源的诊断扩展。目前支持的有 Azure 云服务 Web 和辅助角色、运行 Microsoft Windows 的 Azure 虚拟机，以及 Service Fabric。其他 Azure 服务都有自身不同的诊断扩展。

## 可以收集的数据
Azure 诊断可以收集以下类型的数据：

| 数据源 | 说明 |
| --- | --- |
| 性能计数器 |操作系统和自定义性能计数器 |
| 应用程序日志 |应用程序写入的跟踪消息 |
| Windows 事件日志 |发送到 Windows 事件日志记录系统的信息 |
| .NET 事件源 |使用 .NET EventSource 类的代码编写事件[](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracing.eventsource.aspx) |
| IIS Logs |有关 IIS 网站的信息 |
| 基于清单的 ETW |由任何进程生成的 Windows 事件的事件跟踪 |
| 故障转储 |有关应用程序崩溃时进程状态的信息 |
| 自定义错误日志 |应用程序或服务创建的日志 |
| Azure Diagnostics基础结构日志 |有关诊断自身的信息 |

Azure 诊断扩展可将此数据传输到 Azure 存储帐户。可以将这些数据用于调试和故障排除、度量性能、监视资源使用状况、进行流量分析和容量规划以及进行审核。

## 后续步骤
请选择要尝试在哪个服务上收集诊断数据，并使用以下文章来入门。有关具体任务的参考，请使用一般的 Azure 诊断链接。

## Web 应用
请注意，Web Apps 不使用 Azure 诊断。请在 [Web 应用](/documentation/articles/web-sites-enable-diagnostic-log/)中查找相应的信息

## 使用 Azure 诊断的云服务
请参阅
- [如何使用 Azure 诊断监视云服务](/documentation/articles/cloud-services-how-to-monitor/)
- [在云服务应用程序中设置 Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)

有关更高级的主题，请参阅

- [使用 Azure 诊断跟踪云服务应用程序的流](/documentation/articles/cloud-services-dotnet-diagnostics-trace-flow/)
- [使用 PowerShell 在云服务上设置诊断](/documentation/articles/virtual-machines-windows-ps-extensions-diagnostics/)

有关更高级的主题，请参阅

- [使用 PowerShell 在 Azure 虚拟机上设置诊断](/documentation/articles/virtual-machines-windows-ps-extensions-diagnostics/)
- [使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/)

## 使用 Azure 诊断的 Service Fabric
请参阅[监视 Service Fabric 应用程序](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/)来帮助入门。打开此文章后，可以使用左侧的导航树来查看其他许多 Service Fabric 诊断文章。

## 一般的 Azure 诊断文章
- [Azure 诊断架构配置](https://msdn.microsoft.com/zh-cn/library/azure/mt634524.aspx) — 了解如何更改架构文件以收集和路由诊断数据。请注意，也可以使用 Visual Studio 来更改架构文件。
- [Azure 诊断数据在 Azure 存储中的存储方式](/documentation/articles/cloud-services-dotnet-diagnostics-storage/) — 了解诊断数据写入到的表和 Blob 的名称。
- 了解如何[在 Azure 诊断中使用性能计数器](/documentation/articles/cloud-services-dotnet-diagnostics-performance-counters/)。
- 如果在开始诊断时或者在 Azure 存储表中查找数据时遇到问题，请参阅 [Azure 诊断故障排除](/documentation/articles/azure-diagnostics-troubleshooting/)

<!---HONumber=Mooncake_1212_2016-->
