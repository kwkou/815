<properties
	pageTitle="Azure Monitor 自动缩放常用指标 | Azure"
	description="了解自动缩放云服务、虚拟机和 Web Apps 时常用的指标。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  


<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/06/2016"
	wacn.date="01/03/2017"
	ms.author="ashwink"/>

# Azure Monitor 自动缩放常用指标
利用 Azure Monitor 自动缩放，可根据遥测数据（指标）增加或减少正在运行的实例数。本文档介绍了你可能想要使用的常用指标。在云服务和服务器场的 Azure 门户预览中，可以选择要作为缩放依据的资源指标。不过，你也可以选择其他资源的任何指标来作为缩放依据。

下列信息对于缩放虚拟机规模集同样适用。

> [AZURE.NOTE]
此信息仅适用于基于 Resource Manager 的 VM 和 VM 规模集。
> 

## 基于 Resource Manager 的 VM 的计算指标
默认情况下，基于 Resource Manager 的虚拟机和虚拟机规模集发出基本（主机级）指标。此外，为 Azure VM 和 VMSS 配置诊断数据集合时，Azure 诊断扩展也会发出来宾 OS性能计数器（通常称为“来宾 OS 指标”）。可在自动缩放规则中使用所有这些指标。

可使用 `Get MetricDefinitions` API/PoSH/CLI 查看 VMSS 资源的可用指标。

如果使用 VM 规模集，而且发现特定指标未列出，则可能是诊断扩展已将其*禁用*。

如果特定指标未采样或以所需的频率传输，你可以更新诊断配置。

如果发生上述任一情况，请查看[使用 PowerShell 在运行 Windows 的虚拟机中启用 Azure 诊断](/documentation/articles/virtual-machines-windows-ps-extensions-diagnostics/)，将 Azure VM 诊断扩展配置和更新为启用该指标。这篇文章还包含一个诊断配置文件示例。

### <a name="compute-metrics-for-windows-vm-v2-as-a-guest-os"></a> Windows VM v2 作为来宾 OS 时的计算指标

当你在 Azure 中创建新的 VM (v2) 时，使用诊断扩展会启用诊断。

你可以在 PowerShell 中使用以下命令生成指标列表。

		Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit

可以针对下列指标创建警报：

|指标名称|	计价单位|
|---|---|
|\\Processor(\_Total)\\% Processor Time |百分比|
|\\Processor(\_Total)\\% Privileged Time |百分比|
|\\Processor(\_Total)\\% User Time |百分比|
|\\Processor Information(\_Total)\\Processor Frequency |计数|
|\\System\\Processes| 计数|
|\\Process(\_Total)\\Thread Count| 计数|
|\\Process(\_Total)\\Handle Count |计数|
|\\Memory\\% Committed Bytes In Use |百分比|
|\\Memory\\Available Bytes| 字节|
|\\Memory\\Committed Bytes |字节|
|\\Memory\\Commit Limit| 字节|
|\\Memory\\Pool Paged Bytes| 字节|
|\\Memory\\Pool Nonpaged Bytes| 字节|
|\\PhysicalDisk(\_Total)\\% Disk Time| 百分比|
|\\PhysicalDisk(\_Total)\\% Disk Read Time| 百分比|
|\\PhysicalDisk(\_Total)\\% Disk Write Time| 百分比|
|\\PhysicalDisk(\_Total)\\Disk Transfers/sec |每秒计数|
|\\PhysicalDisk(\_Total)\\Disk Reads/sec |每秒计数|
|\\PhysicalDisk(\_Total)\\Disk Writes/sec |每秒计数|
|\\PhysicalDisk(\_Total)\\Disk Bytes/sec |每秒字节数|
|\\PhysicalDisk(\_Total)\\Disk Read Bytes/sec| 每秒字节数|
|\\PhysicalDisk(\_Total)\\Disk Write Bytes/sec |每秒字节数|
|\\PhysicalDisk(\_Total)\\Avg.Disk Queue Length| 计数|
|\\PhysicalDisk(\_Total)\\Avg.Disk Read Queue Length| 计数|
|\\PhysicalDisk(\_Total)\\Avg.Disk Write Queue Length |计数|
|\\LogicalDisk(\_Total)\\% Free Space| 百分比|
|\\LogicalDisk(\_Total)\\Free Megabytes| 计数|



### <a name="compute-metrics-for-linux-vm-v2-as-a-guest-os"></a> Linux VM v2 作为来宾 OS 时的计算指标

在 Azure 中创建 VM 时，使用诊断扩展会默认启用诊断。

你可以在 PowerShell 中使用以下命令生成指标列表。

		Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit

 你可以针对下列指标创建警报。

|指标名称|	计价单位|
|---|---|
|\\Memory\\AvailableMemory |字节|
|\\Memory\\PercentAvailableMemory|	百分比|
|\\Memory\\UsedMemory|	字节|
|\\Memory\\PercentUsedMemory|	百分比|
|\\Memory\\PercentUsedByCache |百分比|
|\\Memory\\PagesPerSec|	每秒计数|
|\\Memory\\PagesReadPerSec|	每秒计数|
|\\Memory\\PagesWrittenPerSec |每秒计数|
|\\Memory\\AvailableSwap |字节|
|\\Memory\\PercentAvailableSwap|	百分比|
|\\Memory\\UsedSwap |字节|
|\\Memory\\PercentUsedSwap|	百分比|
|\\Processor\\PercentIdleTime|	百分比|
|\\Processor\\PercentUserTime|	百分比|
|\\Processor\\PercentNiceTime |百分比|
|\\Processor\\PercentPrivilegedTime |百分比|
|\\Processor\\PercentInterruptTime|	百分比|
|\\Processor\\PercentDPCTime|	百分比|
|\\Processor\\PercentProcessorTime |百分比|
|\\Processor\\PercentIOWaitTime |百分比|
|\\PhysicalDisk\\BytesPerSecond|	每秒字节数|
|\\PhysicalDisk\\ReadBytesPerSecond|	每秒字节数|
|\\PhysicalDisk\\WriteBytesPerSecond|	每秒字节数|
|\\PhysicalDisk\\TransfersPerSecond |每秒计数|
|\\PhysicalDisk\\ReadsPerSecond |每秒计数|
|\\PhysicalDisk\\WritesPerSecond |每秒计数|
|\\PhysicalDisk\\AverageReadTime|	秒|
|\\PhysicalDisk\\AverageWriteTime |秒|
|\\PhysicalDisk\\AverageTransferTime|	秒|
|\\PhysicalDisk\\AverageDiskQueueLength|	计数|
|\\NetworkInterface\\BytesTransmitted |字节|
|\\NetworkInterface\\BytesReceived |字节|
|\\NetworkInterface\\PacketsTransmitted |计数|
|\\NetworkInterface\\PacketsReceived |计数|
|\\NetworkInterface\\BytesTotal |字节|
|\\NetworkInterface\\TotalRxErrors|	计数|
|\\NetworkInterface\\TotalTxErrors|	计数|
|\\NetworkInterface\\TotalCollisions|	计数|




## 常用的 Web（服务器场）指标

你也可以根据常用的 Web 服务器指标（如 Http 队列长度）执行自动缩放。其指标名称为 **HttpQueueLength**。以下部分列出了可用的服务器场 (Web Apps) 指标。

### Web Apps 指标

你可以在 PowerShell 中使用以下命令生成 Web Apps 指标列表。

		Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit

你可以针对这些指标发出警报或以其为缩放依据。

|指标名称|	计价单位|
|---|---|
|CpuPercentage|	百分比|
|MemoryPercentage |百分比|
|DiskQueueLength|	计数|
|HttpQueueLength|	计数|
|BytesReceived|	字节|
|BytesSent|	字节|


## 常用的存储指标
你可以将存储队列长度作为缩放依据，它是存储队列中的消息数目。存储队列长度是一个特殊指标，阈值是每个实例的消息数。例如，如果有两个实例并且阈值设置为 100，则当队列中的消息总数为 200 时将进行缩放。这两个实例的消息数可能各为 100，或分别为 120 和 80，或者为其他相加大于等于 200 的数字组合。

在 Azure 门户预览的“设置”边栏选项卡中配置此配置。若使用 VM 规模集，可以将 Resource Manager 模板中的“自动缩放”设置更新为使用 *metricName* 作为 *ApproximateMessageCount*，并传递存储队列的 ID 作为 *metricResourceUri*。

例如，对于经典存储帐户，自动缩放设置 metricTrigger 将包括：

		"metricName": "ApproximateMessageCount",
		 "metricNamespace": "",
		 "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ClassicStorage/storageAccounts/mystorage/services/queue/queues/mystoragequeue"

对于（非经典）存储帐户，metricTrigger 将包括：

		"metricName": "ApproximateMessageCount",
		"metricNamespace": "",
		"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.Storage/storageAccounts/mystorage/services/queue/queues/mystoragequeue"

## 常用的服务总线指标
你可以按服务总线队列的长度进行缩放，该长度是服务总线队列中的消息数量。服务总线队列长度是一个特殊指标，阈值是每个实例的消息数。例如，如果有两个实例并且阈值设置为 100，则当队列中的消息总数为 200 时将进行缩放。这两个实例的消息数可能各为 100，或分别为 120 和 80，或者为其他相加大于等于 200 的数字组合。

若使用 VM 规模集，可以将 Resource Manager 模板中的“自动缩放”设置更新为使用 *metricName* 作为 *ApproximateMessageCount*，并传递存储队列的 ID 作为 *metricResourceUri*。

		"metricName": "MessageCount",
		 "metricNamespace": "",
		"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue"

>[AZURE.NOTE] 若使用服务总线，则不存在资源组这一概念，但 Azure Resource Manager 会为每个区域创建一个默认资源组。此资源组通常采用“Default-ServiceBus-[region]”的格式。

<!---HONumber=Mooncake_1226_2016-->