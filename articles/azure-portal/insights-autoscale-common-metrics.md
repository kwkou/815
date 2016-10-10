<properties
	pageTitle="Azure Insights：Azure Insights 自动缩放常用指标。| Azure"
	description="了解自动缩放云服务、虚拟机和 Web Apps 时常用的指标。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="azure-portal"
	documentationCenter="na"/>

<tags
	ms.service="azure-portal"
	ms.date="08/02/2016"
	wacn.date="09/26/2016"/>

# Azure Insights 自动缩放常用指标

利用 Azure Insights 自动缩放，你可以根据遥测数据（指标）增加或减少正在运行的实例数。本文档介绍了你可能想要使用的常用指标。在云服务和服务器场的 Azure 门户预览中，你可以选择要作为缩放依据的资源指标。不过，你也可以选择其他资源的任何指标来作为缩放依据。

下面介绍了有关如何找到并列出要作为缩放依据的指标的详细信息。下列信息对于缩放虚拟机缩放集同样适用。

## 计算指标
默认情况下，Azure VM v2 附带已配置的诊断扩展，并已启用下列指标。

- [Windows VM v2 的来宾指标](#compute-metrics-for-windows-vm-v2-as-a-guest-os)
- [Linux VM v2 的来宾指标](#compute-metrics-for-linux-vm-v2-as-a-guest-os)

你可以使用 `Get MetricDefinitions` API/PoSH/CLI 来查看 VMSS 资源的可用指标。

如果你使用 VM 缩放集，而且发现特定指标未列出，可能是你的诊断扩展*已禁用*它。

如果特定指标未采样或以所需的频率传输，你可以更新诊断配置。



### Windows VM v2 作为来宾 OS 时的计算指标

当你在 Azure 中创建新的 VM (v2) 时，使用诊断扩展会启用诊断。

你可以在 PowerShell 中使用以下命令生成指标列表。


```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

你可以针对下列指标创建警报。

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



### Linux VM v2 作为来宾 OS 时的计算指标

当你在 Azure 中创建新的 VM (v2) 时，使用诊断扩展会默认启用诊断。

你可以在 PowerShell 中使用以下命令生成指标列表。

```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

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

```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

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
你可以将存储队列长度作为缩放依据，它是存储队列中的消息数目。存储队列长度是一个特殊指标，所应用的阈值将为每个实例的消息数。这意味着，如果有两个实例，且如果阈值设置为 100，则当队列中的消息总数为 200 时将进行缩放。例如，每个实例 100 条消息。

你可以在 Azure 门户预览的“设置”边栏选项卡中进行此配置。若使用 VM 缩放集，你可以将 ARM 模板中的“自动缩放”设置更新为使用 metricName 作为 ApproximateMessageCount，并传递存储队列的 ID 作为 metricResourceUri。


```
"metricName": "ApproximateMessageCount",
 "metricNamespace": "",
 "metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ClassicStorage/storageAccounts/mystorage/services/queue/queues/mystoragequeue"
 ```

## 常用的服务总线指标

你可以按服务总线队列的长度进行缩放，该长度是服务总线队列中的消息数量。服务总线队列长度是一项特殊指标，应用的指定阈值将是每个实例的消息数量。这意味着，如果有两个实例，且如果阈值设置为 100，则当队列中的消息总数为 200 时将进行缩放。 例如，每个实例 100 条消息。

若使用 VM 缩放集，你可以将 ARM 模板中的“自动缩放”设置更新为使用 metricName 作为 ApproximateMessageCount，并传递存储队列的 ID 作为 metricResourceUri。

```
"metricName": "MessageCount",
 "metricNamespace": "",
"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue"
```

>[AZURE.NOTE] 若使用服务总线，则不存在资源组这一概念，但 Azure Resource Manager 会为每个区域创建一个默认资源组。此资源组通常采用“Default-ServiceBus-[region]”的格式。

<!---HONumber=Mooncake_0503_2016-->