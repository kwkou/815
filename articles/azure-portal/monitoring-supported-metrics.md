<properties
	pageTitle="Azure 监视器指标 - 每种资源类型支持的指标 | Azure"
	description="可在 Azure 监视器中为每种资源类型使用的指标的列表。"
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  


<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/26/2016"
	wacn.date="11/14/2016"
	ms.author="johnkem"/>  


# Azure 监视器支持的指标

Azure 监视器提供多种方式来与指标交互，包括在门户中制作指标图表、通过 REST API 访问指标，或者使用 PowerShell 或 CLI 查询指标。下面是目前可在 Azure 监视器的指标管道中使用的完整指标列表。

> [AZURE.NOTE] 其他指标可在门户或旧版 API 中使用。此列表仅包含可以通过合并的 Azure 监视器指标管道公共预览版使用的公共预览版指标。

## Microsoft.Batch/batchAccounts

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CoreCount|核心计数|计数|总计|批处理帐户中的核心总数|
|TotalNodeCount|节点计数|计数|总计|批处理帐户中的节点总数|
|CreatingNodeCount|正在创建的节点计数|计数|总计|正在创建的节点数目|
|StartingNodeCount|正在启动的节点计数|计数|总计|正在启动的节点数目|
|WaitingForStartTaskNodeCount|正在等待启动任务的节点计数|计数|总计|正在等待启动任务完成的节点数目|
|StartTaskFailedNodeCount|启动任务失败的节点计数|计数|总计|启动任务失败的节点数目|
|IdleNodeCount|空闲节点计数|计数|总计|空闲节点数目|
|OfflineNodeCount|脱机节点计数|计数|总计|脱机节点数目|
|RebootingNodeCount|正在重新启动的节点计数|计数|总计|正在重新启动的节点数目|
|ReimagingNodeCount|正在重置映像的节点计数|计数|总计|正在重置映像的节点数目|
|RunningNodeCount|正在运行的节点计数|计数|总计|正在运行的节点数目|
|LeavingPoolNodeCount|正在退出池的节点计数|计数|总计|正在退出池的节点数目|
|UnusableNodeCount|不可用的节点计数|计数|总计|不可用的节点数目|
|TaskStartEvent|任务启动事件数|计数|总计|已启动的任务总数|
|TaskCompleteEvent|任务完成事件数|计数|总计|已完成的任务总数|
|TaskFailEvent|任务失败事件数|计数|总计|处于失败状态的已完成任务总数|
|PoolCreateEvent|池创建事件数|计数|总计|已创建的池总数|
|PoolResizeStartEvent|池调整大小启动事件数|计数|总计|已启动的池调整大小活动总数|
|PoolResizeCompleteEvent|池调整大小完成事件数|计数|总计|已完成的池调整大小活动总数|
|PoolDeleteStartEvent|池删除启动事件数|计数|总计|已启动的池删除活动总数|
|PoolDeleteCompleteEvent|池删除完成事件数|计数|总计|已完成的池删除活动总数|

## Microsoft.Cache/redis

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|connectedclients|连接的客户端数|计数|最大值||
|totalcommandsprocessed|总操作数|计数|总计||
|cachehits|缓存命中数|计数|总计||
|cachemisses|缓存未命中数|计数|总计||
|getcommands|获取数|计数|总计||
|setcommands|设置数|计数|总计||
|evictedkeys|逐出的密钥数|计数|总计||
|totalkeys|总密钥数|计数|最大值||
|expiredkeys|过期的密钥数|计数|总计||
|usedmemory|已用内存|字节|最大值||
|usedmemoryRss|已用内存 RSS|字节|最大值||
|serverLoad|服务器负载|百分比|最大值||
|cacheWrite|缓存写入量|每秒字节数|最大值||
|cacheRead|缓存读取量|每秒字节数|最大值||
|percentProcessorTime|CPU|百分比|最大值||
|connectedclients0|连接的客户端数(分片 0)|计数|最大值||
|totalcommandsprocessed0|总操作数(分片 0)|计数|总计||
|cachehits0|缓存命中数(分片 0)|计数|总计||
|cachemisses0|缓存未命中数(分片 0)|计数|总计||
|getcommands0|Get 数(分片 0)|计数|总计||
|setcommands0|Set 数(分片 0)|计数|总计||
|evictedkeys0|逐出的密钥数(分片 0)|计数|总计||
|totalkeys0|总密钥数(节点 0)|计数|最大值||
|expiredkeys0|过期的密钥数(分片 0)|计数|总计||
|usedmemory0|已用内存量(分片 0)|字节|最大值||
|usedmemoryRss0|已用内存 RSS (分片 0)|字节|最大值||
|serverLoad0|服务器负载(分片 0)|百分比|最大值||
|cacheWrite0|缓存写入量(分片 0)|每秒字节数|最大值||
|cacheRead0|缓存读取量(分片 0)|每秒字节数|最大值||
|percentProcessorTime0|CPU (分片 0)|百分比|最大值||
|connectedclients1|连接的客户端数(分片 1)|计数|最大值||
|totalcommandsprocessed1|总操作数(分片 1)|计数|总计||
|cachehits1|缓存命中数(分片 1)|计数|总计||
|cachemisses1|缓存未命中数(分片 1)|计数|总计||
|getcommands1|Get 数(分片 1)|计数|总计||
|setcommands1|Set 数(分片 1)|计数|总计||
|evictedkeys1|逐出的密钥数(分片 1)|计数|总计||
|totalkeys1|总密钥数(节点 1)|计数|最大值||
|expiredkeys1|过期的密钥数(分片 1)|计数|总计||
|usedmemory1|已用内存量(分片 1)|字节|最大值||
|usedmemoryRss1|已用内存 RSS (分片 1)|字节|最大值||
|serverLoad1|服务器负载(分片 1)|百分比|最大值||
|cacheWrite1|缓存写入量(分片 1)|每秒字节数|最大值||
|cacheRead1|缓存读取量(分片 1)|每秒字节数|最大值||
|percentProcessorTime1|CPU (分片 1)|百分比|最大值||
|connectedclients2|连接的客户端数(分片 2)|计数|最大值||
|totalcommandsprocessed2|总操作数(分片 2)|计数|总计||
|cachehits2|缓存命中数(分片 2)|计数|总计||
|cachemisses2|缓存未命中数(分片 2)|计数|总计||
|getcommands2|Get 数(分片 2)|计数|总计||
|setcommands2|Set 数(分片 2)|计数|总计||
|evictedkeys2|逐出的密钥数(分片 2)|计数|总计||
|totalkeys2|总密钥数(节点 2)|计数|最大值||
|expiredkeys2|过期的密钥数(分片 2)|计数|总计||
|usedmemory2|已用内存量(分片 2)|字节|最大值||
|usedmemoryRss2|已用内存 RSS (分片 2)|字节|最大值||
|serverLoad2|服务器负载(分片 2)|百分比|最大值||
|cacheWrite2|缓存写入量(分片 2)|每秒字节数|最大值||
|cacheRead2|缓存读取量(分片 2)|每秒字节数|最大值||
|percentProcessorTime2|CPU (分片 2)|百分比|最大值||
|connectedclients3|连接的客户端数(分片 3)|计数|最大值||
|totalcommandsprocessed3|总操作数(分片 3)|计数|总计||
|cachehits3|缓存命中数(分片 3)|计数|总计||
|cachemisses3|缓存未命中数(分片 3)|计数|总计||
|getcommands3|Get 数(分片 3)|计数|总计||
|setcommands3|Set 数(分片 3)|计数|总计||
|evictedkeys3|逐出的密钥数(分片 3)|计数|总计||
|totalkeys3|总密钥数(节点 3)|计数|最大值||
|expiredkeys3|过期的密钥数(分片 3)|计数|总计||
|usedmemory3|已用内存量(分片 3)|字节|最大值||
|usedmemoryRss3|已用内存 RSS (分片 3)|字节|最大值||
|serverLoad3|服务器负载(分片 3)|百分比|最大值||
|cacheWrite3|缓存写入量(分片 3)|每秒字节数|最大值||
|cacheRead3|缓存读取量(分片 3)|每秒字节数|最大值||
|percentProcessorTime3|CPU (分片 3)|百分比|最大值||
|connectedclients4|连接的客户端数(分片 4)|计数|最大值||
|totalcommandsprocessed4|总操作数(分片 4)|计数|总计||
|cachehits4|缓存命中数(分片 4)|计数|总计||
|cachemisses4|缓存未命中数(分片 4)|计数|总计||
|getcommands4|Get 数(分片 4)|计数|总计||
|setcommands4|Set 数(分片 4)|计数|总计||
|evictedkeys4|逐出的密钥数(分片 4)|计数|总计||
|totalkeys4|总密钥数(节点 4)|计数|最大值||
|expiredkeys4|过期的密钥数(分片 4)|计数|总计||
|usedmemory4|已用内存量(分片 4)|字节|最大值||
|usedmemoryRss4|已用内存 RSS (分片 4)|字节|最大值||
|serverLoad4|服务器负载(分片 4)|百分比|最大值||
|cacheWrite4|缓存写入量(分片 4)|每秒字节数|最大值||
|cacheRead4|缓存读取量(分片 4)|每秒字节数|最大值||
|percentProcessorTime4|CPU (分片 4)|百分比|最大值||
|connectedclients5|连接的客户端数(分片 5)|计数|最大值||
|totalcommandsprocessed5|总操作数(分片 5)|计数|总计||
|cachehits5|缓存命中数(分片 5)|计数|总计||
|cachemisses5|缓存未命中数(分片 5)|计数|总计||
|getcommands5|Get 数(分片 5)|计数|总计||
|setcommands5|Set 数(分片 5)|计数|总计||
|evictedkeys5|逐出的密钥数(分片 5)|计数|总计||
|totalkeys5|总密钥数(节点 5)|计数|最大值||
|expiredkeys5|过期的密钥数(分片 5)|计数|总计||
|usedmemory5|已用内存量(分片 5)|字节|最大值||
|usedmemoryRss5|已用内存 RSS (分片 5)|字节|最大值||
|serverLoad5|服务器负载(分片 5)|百分比|最大值||
|cacheWrite5|缓存写入量(分片 5)|每秒字节数|最大值||
|cacheRead5|缓存读取量(分片 5)|每秒字节数|最大值||
|percentProcessorTime5|CPU (分片 5)|百分比|最大值||
|connectedclients6|连接的客户端数(分片 6)|计数|最大值||
|totalcommandsprocessed6|总操作数(分片 6)|计数|总计||
|cachehits6|缓存命中数(分片 6)|计数|总计||
|cachemisses6|缓存未命中数(分片 6)|计数|总计||
|getcommands6|Get 数(分片 6)|计数|总计||
|setcommands6|Set 数(分片 6)|计数|总计||
|evictedkeys6|逐出的密钥数(分片 6)|计数|总计||
|totalkeys6|总密钥数(节点 6)|计数|最大值||
|expiredkeys6|过期的密钥数(分片 6)|计数|总计||
|usedmemory6|已用内存量(分片 6)|字节|最大值||
|usedmemoryRss6|已用内存 RSS (分片 6)|字节|最大值||
|serverLoad6|服务器负载(分片 6)|百分比|最大值||
|cacheWrite6|缓存写入量(分片 6)|每秒字节数|最大值||
|cacheRead6|缓存读取量(分片 6)|每秒字节数|最大值||
|percentProcessorTime6|CPU (分片 6)|百分比|最大值||
|connectedclients7|连接的客户端数(分片 7)|计数|最大值||
|totalcommandsprocessed7|总操作数(分片 7)|计数|总计||
|cachehits7|缓存命中数(分片 7)|计数|总计||
|cachemisses7|缓存未命中数(分片 7)|计数|总计||
|getcommands7|Get 数(分片 7)|计数|总计||
|setcommands7|Set 数(分片 7)|计数|总计||
|evictedkeys7|逐出的密钥数(分片 7)|计数|总计||
|totalkeys7|总密钥数(节点 7)|计数|最大值||
|expiredkeys7|过期的密钥数(分片 7)|计数|总计||
|usedmemory7|已用内存量(分片 7)|字节|最大值||
|usedmemoryRss7|已用内存 RSS (分片 7)|字节|最大值||
|serverLoad7|服务器负载(分片 7)|百分比|最大值||
|cacheWrite7|缓存写入量(分片 7)|每秒字节数|最大值||
|cacheRead7|缓存读取量(分片 7)|每秒字节数|最大值||
|percentProcessorTime7|CPU (分片 7)|百分比|最大值||
|connectedclients8|连接的客户端数(分片 8)|计数|最大值||
|totalcommandsprocessed8|总操作数(分片 8)|计数|总计||
|cachehits8|缓存命中数(分片 8)|计数|总计||
|cachemisses8|缓存未命中数(分片 8)|计数|总计||
|getcommands8|Get 数(分片 8)|计数|总计||
|setcommands8|Set 数(分片 8)|计数|总计||
|evictedkeys8|逐出的密钥数(分片 8)|计数|总计||
|totalkeys8|总密钥数(节点 8)|计数|最大值||
|expiredkeys8|过期的密钥数(分片 8)|计数|总计||
|usedmemory8|已用内存量(分片 8)|字节|最大值||
|usedmemoryRss8|已用内存 RSS (分片 8)|字节|最大值||
|serverLoad8|服务器负载(分片 8)|百分比|最大值||
|cacheWrite8|缓存写入量(分片 8)|每秒字节数|最大值||
|cacheRead8|缓存读取量(分片 8)|每秒字节数|最大值||
|percentProcessorTime8|CPU (分片 8)|百分比|最大值||
|connectedclients9|连接的客户端数(分片 9)|计数|最大值||
|totalcommandsprocessed9|总操作数(分片 9)|计数|总计||
|cachehits9|缓存命中数(分片 9)|计数|总计||
|cachemisses9|缓存未命中数(分片 9)|计数|总计||
|getcommands9|Get 数(分片 9)|计数|总计||
|setcommands9|Set 数(分片 9)|计数|总计||
|evictedkeys9|逐出的密钥数(分片 9)|计数|总计||
|totalkeys9|总密钥数(节点 9)|计数|最大值||
|expiredkeys9|过期的密钥数(分片 9)|计数|总计||
|usedmemory9|已用内存量(分片 9)|字节|最大值||
|usedmemoryRss9|已用内存 RSS (分片 9)|字节|最大值||
|serverLoad9|服务器负载(分片 9)|百分比|最大值||
|cacheWrite9|缓存写入量(分片 9)|每秒字节数|最大值||
|cacheRead9|缓存读取量(分片 9)|每秒字节数|最大值||
|percentProcessorTime9|CPU (分片 9)|百分比|最大值||

## Microsoft.CognitiveServices/accounts

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|NumberOfCalls|API 调用总数|计数|总计|API 调用总数。|
|NumberOfFailedCalls|失败的 API 调用总数|计数|总计|失败的 API 调用总数。|

## Microsoft.Compute/virtualMachines

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CPU 百分比|CPU 百分比|百分比|平均值|当前虚拟机正在使用的已分配计算单元百分比|
|网络传入|网络传入|字节|总计|虚拟机在所有网络接口上收到的字节数（传入流量）|
|网络传出|网络传出|字节|总计|虚拟机在所有网络接口上发出的字节数（传出流量）|
|磁盘读取字节数|磁盘读取字节数|字节|总计|监视期间从磁盘读取的字节总数|
|磁盘写入字节数|磁盘写入字节数|字节|总计|监视期间向磁盘写入的字节总数|
|磁盘读取操作次数/秒|磁盘读取操作次数/秒|每秒计数|平均值|磁盘读取 IOPS|
|磁盘写入操作次数/秒|磁盘写入操作次数/秒|每秒计数|平均值|磁盘写入 IOPS|

## Microsoft.Compute/virtualMachineScaleSets

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CPU 百分比|CPU 百分比|百分比|平均值|当前虚拟机正在使用的已分配计算单元百分比|
|网络传入|网络传入|字节|总计|虚拟机在所有网络接口上收到的字节数（传入流量）|
|网络传出|网络传出|字节|总计|虚拟机在所有网络接口上发出的字节数（传出流量）|
|磁盘读取字节数|磁盘读取字节数|字节|总计|监视期间从磁盘读取的字节总数|
|磁盘写入字节数|磁盘写入字节数|字节|总计|监视期间向磁盘写入的字节总数|
|磁盘读取操作次数/秒|磁盘读取操作次数/秒|每秒计数|平均值|磁盘读取 IOPS|
|磁盘写入操作次数/秒|磁盘写入操作次数/秒|每秒计数|平均值|磁盘写入 IOPS|

## Microsoft.Compute/virtualMachineScaleSets/virtualMachines

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CPU 百分比|CPU 百分比|百分比|平均值|当前虚拟机正在使用的已分配计算单元百分比|
|网络传入|网络传入|字节|总计|虚拟机在所有网络接口上收到的字节数（传入流量）|
|网络传出|网络传出|字节|总计|虚拟机在所有网络接口上发出的字节数（传出流量）|
|磁盘读取字节数|磁盘读取字节数|字节|总计|监视期间从磁盘读取的字节总数|
|磁盘写入字节数|磁盘写入字节数|字节|总计|监视期间向磁盘写入的字节总数|
|磁盘读取操作次数/秒|磁盘读取操作次数/秒|每秒计数|平均值|磁盘读取 IOPS|
|磁盘写入操作次数/秒|磁盘写入操作次数/秒|每秒计数|平均值|磁盘写入 IOPS|

## Microsoft.Devices/IotHubs

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|d2c.telemetry.ingress.allProtocol|遥测消息发送尝试次数|计数|总计|尝试发送到 IoT 中心的、设备到云的遥测消息数目|
|d2c.telemetry.ingress.success|发送的遥测消息数|计数|总计|成功发送到 IoT 中心的、设备到云的遥测消息数目|
|c2d.commands.egress.complete.success|完成的命令数|计数|总计|设备已成功完成的云到设备命令数目|
|c2d.commands.egress.abandon.success|放弃的命令数|计数|总计|设备放弃的云到设备命令数目|
|c2d.commands.egress.reject.success|拒绝的命令数|计数|总计|设备拒绝的云到设备命令数目|
|devices.totalDevices|设备总数|计数|总计|已注册到 IoT 中心的设备数目|
|devices.connectedDevices.allProtocol|连接的设备数|计数|总计|已连接到 IoT 中心的设备数目|

## Microsoft.EventHub/namespaces

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|INREQS|传入请求数|计数|总计|命名空间的事件中心传入消息吞吐量|
|SUCCREQ|成功的请求数|计数|总计|命名空间的成功请求总数|
|FAILREQ|失败的请求数|计数|总计|命名空间的失败请求总数|
|SVRBSY|服务器繁忙错误数|计数|总计|命名空间的服务器繁忙错误总数|
|INTERR|内部服务器错误数|计数|总计|命名空间的内部服务器错误总数|
|MISCERR|其他错误数|计数|总计|命名空间的失败请求总数|
|INMSGS|传入消息数|计数|总计|命名空间的传入消息总数|
|OUTMSGS|传出消息数|计数|总计|命名空间的传出消息总数|
|EHINMBS|每秒传入字节数|每秒字节数|总计|命名空间的事件中心传入消息吞吐量|
|EHOUTMBS|每秒传出字节数|每秒字节数|总计|命名空间的传出消息总数|
|EHABL|存档积压工作消息数|计数|总计|命名空间积压工作中的事件中心存档消息数|
|EHAMSGS|存档消息数|计数|总计|命名空间中的事件中心存档消息数|
|EHAMBS|存档消息吞吐量|每秒字节数|总计|命名空间中的事件中心存档消息吞吐量|

## Microsoft.Logic/workflows

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|RunsStarted|已启动的运行数|计数|总计|已启动的工作流运行数目。|
|RunsCompleted|已完成的运行数|计数|总计|已完成的工作流运行数目。|
|RunsSucceeded|成功的运行数|计数|总计|成功的工作流运行数目。|
|RunsFailed|失败的运行数|计数|总计|失败的工作流运行数目。|
|RunsCancelled|取消的运行数|计数|总计|已取消的工作流运行数目。|
|RunLatency|运行延迟|秒|平均值|已完成的工作流运行的延迟。|
|RunSuccessLatency|运行成功延迟|秒|平均值|已成功的工作流运行的延迟。|
|RunThrottledEvents|运行限制事件数|计数|总计|工作流操作或触发器限制事件数目。|
|RunFailurePercentage|运行失败百分比|百分比|总计|失败的工作流运行百分比。|
|ActionsStarted|启动的操作数 |计数|总计|已启动的工作流操作数目。|
|ActionsCompleted|完成的操作数 |计数|总计|已完成的工作流操作数目。|
|ActionsSucceeded|成功的操作数 |计数|总计|成功的工作流操作数目。|
|ActionsFailed|失败的操作数|计数|总计|失败的工作流操作数目。|
|ActionsSkipped|跳过的操作数 |计数|总计|已跳过的工作流操作数目。|
|ActionLatency|操作延迟 |秒|平均值|已完成的工作流操作的延迟。|
|ActionSuccessLatency|操作成功延迟 |秒|平均值|已成功的工作流操作的延迟。|
|ActionThrottledEvents|操作限制事件数|计数|总计|工作流操作限制事件数目。|
|TriggersStarted|启动的触发器数 |计数|总计|已启动的工作流触发器数目。|
|TriggersCompleted|完成的触发器数 |计数|总计|已完成的工作流触发器数目。|
|TriggersSucceeded|成功的触发器数 |计数|总计|成功的工作流触发器数目。|
|TriggersFailed|失败的触发器数 |计数|总计|失败的工作流触发器数目。|
|TriggersSkipped|跳过的触发器数|计数|总计|已跳过的工作流触发器数目。|
|TriggersFired|激发的触发器数 |计数|总计|已激发的工作流触发器数目。|
|TriggerLatency|触发器延迟 |秒|平均值|已完成的工作流触发器的延迟。|
|TriggerFireLatency|触发器激发延迟 |秒|平均值|已激发的工作流触发器的延迟。|
|TriggerSuccessLatency|触发器成功延迟 |秒|平均值|已成功的工作流触发器的延迟。|
|TriggerThrottledEvents|触发器限制事件数|计数|总计|工作流触发器限制事件数目。|
|BillableActionExecutions|计费的操作执行数|计数|总计|计费的工作流操作执行数目。|
|BillableTriggerExecutions|计费的触发器执行数|计数|总计|计费的工作流触发器执行数目。|
|TotalBillableExecutions|计费的执行总数|计数|总计|计费的工作流执行数目。|

## Microsoft.Network/applicationGateways

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|吞吐量|吞吐量|每秒字节数|平均值||

## Microsoft.Search/searchServices

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|SearchLatency|搜索延迟|秒|平均值|搜索服务的平均搜索延迟|
|SearchQueriesPerSecond|每秒搜索查询数|每秒计数|平均值|搜索服务的每秒搜索查询数|
|ThrottledSearchQueriesPercentage|限制的搜索查询百分比|百分比|平均值|为搜索服务限制的搜索查询百分比|

## Microsoft.ServiceBus/namespaces

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CPUXNS|每个命名空间的 CPU 使用率|百分比|最大值|服务总线高级命名空间 CPU 使用率指标|
|WSXNS|每个命名空间的内存使用量|百分比|最大值|服务总线高级命名空间内存使用率指标|

## Microsoft.Sql/servers/databases

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|cpu\_percent|CPU 百分比|百分比|平均值|CPU 百分比|
|physical\_data\_read\_percent|数据 IO 百分比|百分比|平均值|数据 IO 百分比|
|log\_write\_percent|日志 IO 百分比|百分比|平均值|日志 IO 百分比|
|dtu\_consumption\_percent|DTU 百分比|百分比|平均值|DTU 百分比|
|storage|数据库总大小|字节|最大值|数据库总大小|
|connection\_successful|成功的连接数|计数|总计|成功的连接数|
|connection\_failed|失败的连接数|计数|总计|失败的连接数|
|blocked\_by\_firewall|被防火墙阻止|计数|总计|被防火墙阻止|
|deadlock|死锁数|计数|总计|死锁数|
|storage\_percent|数据库大小百分比|百分比|最大值|数据库大小百分比|
|xtp\_storage\_percent|In-Memory OLTP 存储百分比（预览）|百分比|平均值|In-Memory OLTP 存储百分比（预览）|
|workers\_percent|辅助角色百分比|百分比|平均值|辅助角色百分比|
|sessions\_percent|会话百分比|百分比|平均值|会话百分比|
|dtu\_limit|DTU 限制|计数|平均值|DTU 限制|
|dtu\_used|已用的 DTU|计数|平均值|已用的 DTU|
|service\_level\_objective|数据库的服务级别目标|计数|总计|数据库的服务级别目标|
|dwu\_limit|dwu 限制|计数|最大值|dwu 限制|
|dwu\_consumption\_percent|DWU 百分比|百分比|平均值|DWU 百分比|
|dwu\_used|已用的 DWU|计数|平均值|已用的 DWU|

## Microsoft.Sql/servers/elasticPools

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|cpu\_percent|CPU 百分比|百分比|平均值|CPU 百分比|
|physical\_data\_read\_percent|数据 IO 百分比|百分比|平均值|数据 IO 百分比|
|log\_write\_percent|日志 IO 百分比|百分比|平均值|日志 IO 百分比|
|dtu\_consumption\_percent|DTU 百分比|百分比|平均值|DTU 百分比|
|storage\_percent|存储百分比|百分比|平均值|存储百分比|
|workers\_percent|辅助角色百分比|百分比|平均值|辅助角色百分比|
|sessions\_percent|会话百分比|百分比|平均值|会话百分比|
|eDTU\_limit|eDTU 限制|计数|平均值|eDTU 限制|
|storage\_limit|存储限制|字节|平均值|存储限制|
|eDTU\_used|已用的 eDTU|计数|平均值|已用的 eDTU|
|storage\_used|已用的存储量|字节|平均值|已用的存储量|

## Microsoft.StreamAnalytics/streamingjobs

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|ResourceUtilization|流单元利用率 %|百分比|最大值|流单元利用率 %|
|InputEvents|输入事件数|计数|总计|输入事件数|
|InputEventBytes|输入事件字节数|字节|总计|输入事件字节数|
|LateInputEvents|延迟输入事件数|计数|总计|延迟输入事件数|
|OutputEvents|输出事件数|计数|总计|输出事件数|
|ConversionErrors|数据转换错误数|计数|总计|数据转换错误数|
|错误|运行时错误|计数|总计|运行时错误|
|DroppedOrAdjustedEvents|失序事件数|计数|总计|失序事件数|
|AMLCalloutRequests|函数请求数|计数|总计|函数请求数|
|AMLCalloutFailedRequests|失败的函数请求数|计数|总计|失败的函数请求数|
|AMLCalloutInputEvents|函数事件数|计数|总计|函数事件数|

## Microsoft.Web/serverfarms

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CpuPercentage|CPU 百分比|百分比|平均值|CPU 百分比|
|MemoryPercentage|内存百分比|百分比|平均值|内存百分比|
|DiskQueueLength|磁盘队列长度|计数|总计|磁盘队列长度|
|HttpQueueLength|Http 队列长度|计数|总计|Http 队列长度|
|BytesReceived|数据输入|字节|总计|数据输入|
|BytesSent|数据输出|字节|总计|数据输出|

## Microsoft.Web/sites

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CpuTime|CPU 时间|秒|总计|CPU 时间|
|请求|请求|计数|总计|请求|
|BytesReceived|数据输入|字节|总计|数据输入|
|BytesSent|数据输出|字节|总计|数据输出|
|Http2xx|Http 2xx|计数|总计|Http 2xx|
|Http3xx|Http 3xx|计数|总计|Http 3xx|
|Http401|Http 401|计数|总计|Http 401|
|Http403|Http 403|计数|总计|Http 403|
|Http404|Http 404|计数|总计|Http 404|
|Http406|Http 406|计数|总计|Http 406|
|Http4xx|Http 4xx|计数|总计|Http 4xx|
|Http5xx|Http 服务器错误|计数|总计|Http 服务器错误|
|MemoryWorkingSet|内存工作集|字节|总计|内存工作集|
|AverageMemoryWorkingSet|平均内存工作集|字节|平均值|平均内存工作集|
|AverageResponseTime|平均响应时间|秒|平均值|平均响应时间|

## Microsoft.Web/sites/slots

|度量值|指标显示名称|计价单位|聚合类型|说明|
|---|---|---|---|---|
|CpuTime|CPU 时间|秒|总计|CPU 时间|
|请求|请求|计数|总计|请求|
|BytesReceived|数据输入|字节|总计|数据输入|
|BytesSent|数据输出|字节|总计|数据输出|
|Http2xx|Http 2xx|计数|总计|Http 2xx|
|Http3xx|Http 3xx|计数|总计|Http 3xx|
|Http401|Http 401|计数|总计|Http 401|
|Http403|Http 403|计数|总计|Http 403|
|Http404|Http 404|计数|总计|Http 404|
|Http406|Http 406|计数|总计|Http 406|
|Http4xx|Http 4xx|计数|总计|Http 4xx|
|Http5xx|Http 服务器错误|计数|总计|Http 服务器错误|
|MemoryWorkingSet|内存工作集|字节|总计|内存工作集|
|AverageMemoryWorkingSet|平均内存工作集|字节|平均值|平均内存工作集|
|AverageResponseTime|平均响应时间|秒|平均值|平均响应时间|

## 后续步骤

- [了解 Azure 监视器中的指标](/documentation/articles/monitoring-overview/#monitoring-sources)
- [针对指标创建警报](/documentation/articles/insights-receive-alert-notifications/)
- [将指标导出到存储、事件中心或 Log Analytics](/documentation/articles/monitoring-overview-of-diagnostic-logs/)

<!---HONumber=Mooncake_1107_2016-->