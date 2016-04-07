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
	ms.date="02/20/2016"
	wacn.date="04/05/2016"/>


# 什么是 Azure 诊断


Azure 诊断是 Azure 中可对部署的应用程序启用诊断数据收集的功能。可以使用于自许多不同源的诊断扩展。目前支持的有 Azure 云服务 Web 和辅助角色、运行 Microsoft Windows 的 Azure 虚拟机。其他 Azure 服务都有自身不同的诊断扩展。

## 可以收集的数据

Azure 诊断可以收集以下类型的数据：

数据源|说明
---|---
性能计数器 | 操作系统和自定义性能计数器
应用程序日志 | 应用程序写入的跟踪消息
Windows 事件日志 | 发送到 Windows 事件日志记录系统的信息
.NET 事件源 | 使用 .NET [EventSource](https://msdn.microsoft.com/zh-cn/library/system.diagnostics.tracing.eventsource.aspx) 类的代码编写事件
IIS Logs | 有关 IIS 网站的信息
基于清单的 ETW | 由任何进程生成的 Windows 事件的事件跟踪
故障转储 | 有关应用程序崩溃时进程状态的信息
自定义错误日志 | 应用程序或服务创建的日志
Azure Diagnostics基础结构日志|有关诊断自身的信息

Azure 诊断扩展可将此数据发送到 Azure 存储帐户，或者发送到 [Application Insights](/documentation/articles/app-insights-cloudservices) 等服务。可以将这些数据用于调试和故障排除、度量性能、监视资源使用状况、进行流量分析和容量规划以及进行审核。


## 版本控制
请参阅 [Azure 诊断版本控制历史记录](/documentation/articles/azure-diagnostics-versioning-history)。

## 后续步骤
请选择要尝试在哪个服务上收集诊断数据，并使用以下文章来入门。有关具体任务的参考，请使用一般的 Azure 诊断链接。

## Web Apps
请注意，Web Apps 不使用 Azure 诊断。请在 [Web Apps](/documentation/articles/web-sites-enable-diagnostic-log) 中查找相应的信息。

## 使用 Azure 诊断的云服务
- 如果使用 Visual Studio，请参阅[使用 Visual Studio 跟踪云服务应用程序](/documentation/articles/vs-azure-tools-debug-cloud-services-virtual-machines)来帮助入门。否则，请参阅
- [如何使用 Azure 诊断监视云服务](/documentation/articles/cloud-services-how-to-monitor)
- [在云服务应用程序中设置 Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics)

有关更高级的主题，请参阅
- [将 Azure 诊断与适用于云服务的 Application Insights 配合使用](/documentation/articles/app-insights-cloudservices)
- [使用 Azure 诊断跟踪云服务应用程序的流](/documentation/articles/cloud-services-dotnet-diagnostics-trace-flow)
- [使用 PowerShell 在云服务上设置诊断](/documentation/articles/virtual-machines-extensions-diagnostics-windows-powershell)


## 使用 Azure 诊断的虚拟机
- 如果使用 Visual Studio，请参阅[使用 Visual Studio 跟踪 Microsoft Azure 虚拟机](/documentation/articles/vs-azure-tools-debug-cloud-services-virtual-machines)来帮助入门。否则，请参阅
- [在 Azure 虚拟机上设置 Azure 诊断](./virtual-machines-dotnet-diagnostics.md)

有关更高级的主题，请参阅
- [使用 PowerShell 在 Azure 虚拟机上设置诊断](/documentation/articles/virtual-machines-extensions-diagnostics-windows-powershell)
- [使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机](/documentation/articles/virtual-machines-extensions-diagnostics-windows-template)


## 一般的 Azure 诊断文章
- [Azure 诊断架构配置](https://msdn.microsoft.com/zh-cn/library/azure/mt634524.aspx) - 了解如何更改架构文件以收集和路由诊断数据。请注意，也可以使用 Visual Studio 来更改架构文件。
- [Azure 诊断数据在 Azure 存储空间中的存储方式](/documentation/articles/cloud-services-dotnet-diagnostics-storage) - 了解诊断数据写入到的表和 Blob 的名称。
- 了解如何[在 Azure 诊断中使用性能计数器](/documentation/articles/cloud-services-dotnet-diagnostics-performance-counters)。
- 了解如何[将 Azure 诊断信息路由到 Application Insights](/documentation/articles/azure-diagnostics-configure-applicationinsights)。
- 如果你在开始诊断时或者在 Azure 存储空间表中查找数据时遇到问题，请参阅 [Azure 诊断故障排除](/documentation/articles/azure-diagnostics-troubleshooting)。

<!---HONumber=Mooncake_0328_2016-->