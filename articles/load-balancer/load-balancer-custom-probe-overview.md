<properties
   pageTitle="负载平衡器自定义探测和监视运行状况 | Azure"
   description="了解如何使用 Azure Load Balancer 的自定义探测来监视负载平衡器后面的实例"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags  
   ms.service="load-balancer"
   ms.date="04/05/2016"
   wacn.date="08/29/2016" />


# 负载平衡器探测

Azure Load Balancer 提供相应的功能让你使用探测来监视服务器实例的运行状况。当探测无法响应时，负载平衡器将会停止向状况不良的实例发送新连接。现有连接不受影响，新连接将发送到状况良好的实例。

当在负载平衡器后面使用虚拟机时，必须配置 TCP 或 HTTP 自定义探测。云服务角色（辅助角色和 Web 角色）是唯一支持来宾代理探测监视的服务器实例。

## 了解探测计数和超时

探测行为取决于：
- 允许实例标记为正在运行的成功探测数目。
- 导致实例标记为未运行的失败探测数目。

在 SuccessFailCount 中设置的超时和频率值决定了是要将实例判定为正在运行还是未运行。在 Azure 门户中，超时设置为频率值的两倍。

终结点（即负载平衡集）的所有负载平衡实例的探测配置必须相同。这意味着，对于同一托管服务中特定终结点组合的每个角色实例或虚拟机而言，不能使用不同的探测配置。例如，每个实例必须有相同的本地端口和超时。


>[AZURE.IMPORTANT] 负载平衡器探测使用 IP 地址 168.63.129.16。此公共 IP 地址有助于自带 IP Azure 虚拟网络方案中内部平台资源的通信。虚拟公共 IP 地址 168.63.129.16 用于所有区域，且不会更改。建议在所有本地防火墙策略中允许此 IP 地址。不应将它视为安全风险，因为只有内部 Azure 平台可以从该地址获取消息源。如果不这样做，各种不同的方案中将出现意外的行为，例如配置相同的 IP 地址范围 168.63.129.16 和出现重复的 IP 地址。


## 了解探测类型

### 来宾代理探测

此探测仅适用于 Azure 云服务。仅当实例处于就绪状态（即，不处于其他状态，例如“繁忙”、“正在回收”或“正在停止”）时，负载平衡器才利用虚拟机内部的来宾代理，然后侦听并以“HTTP 200 正常”作为响应。

有关详细信息，请参阅 [Configuring the service definition file (csdef) for health probes](https://msdn.microsoft.com/zh-cn/library/azure/jj151530.asp)（配置运行状况探测的服务定义文件 (csdef)）或 [Get started creating an Internet-facing load balancer for cloud services](/documentation/articles/load-balancer-get-started-internet-classic-cloud/#check-load-balancer-health-status-for-cloud-services)（开始为云服务创建面向 Internet 的负载平衡器）。

### 来宾代理探测将实例标记为状况不良的原因有哪些？

如果来宾代理无法响应“HTTP 200 正常”，则负载平衡器会将实例标记为无响应，并停止向该实例发送流量。负载平衡器将继续 ping 实例。如果来宾代理响应 HTTP 200，则负载平衡器再次向该实例发送流量。

使用 Web 角色时，网站代码通常在不受 Azure 结构或来宾代理监视的 w3wp.exe 中运行。这意味着，系统不会向来宾代理报告 w3wp.exe 中的失败（例如，HTTP 500 响应），并且负载平衡器不会将该实例退出轮转。


### HTTP 自定义探测

自定义 HTTP 负载平衡器探测会取代默认来宾代理探测，这意味着，你可以创建自己的自定义逻辑来确定角色实例的运行状况。默认情况下，负载平衡器每隔 15 秒探测你的终结点。如果实例在超时期限（默认为 31 秒）内以“HTTP 200 正常”响应，则认为实例在负载平衡器轮转阵容中。

如果想要实现自己的逻辑以便从负载平衡器轮转中删除实例，则这种方案可能很有用。例如，如果实例的 CPU 利用率超过 90% 并返回非 200 状态，则你可以决定删除该实例。如果 Web 角色使用了 w3wp.exe，则这也意味着你可以自动监视网站，因为网站代码中的错误会将非 200 状态返回给负载平衡器探测。

>[AZURE.NOTE] HTTP 自定义探测仅支持相对路径和 HTTP 协议。不支持 HTTPS。


### HTTP 自定义探测将实例标记为状况不良的原因有哪些？

- HTTP 应用程序返回非 200 的 HTTP 响应代码（例如，403、404 或 500）。这是应用程序实例应立即被带离服务的肯定回答。

-  HTTP 服务器在超时期限之后完全无响应。根据设置的超时值，这可能表示在探测标记为未运行之前（也就是说，在发送 SuccessFailCount 探测之前），多个探测请求并未获得响应。
- 	服务器通过 TCP 重置关闭连接。

### TCP 自定义探测

TCP 探测通过使用定义的端口执行三方握手来初始化连接。

### TCP 自定义探测将实例标记为状况不良的原因有哪些？

- TCP 服务器在超时期限之后完全无响应。当探测标记为未运行的时机取决于失败探测的数目，即，在将探测标记为未运行之前，这些请求未获得答复的次数。
- 	探测从角色实例接收 TCP 重置。

有关配置 HTTP 运行状况探测或 TCP 探测的详细信息，请参阅 [Get started creating an Internet-facing load balancer in Resource Manager using PowerShell](/documentation/articles/load-balancer-get-started-internet-arm-ps/#create-lb-rules-nat-rules-a-probe-and-a-load-balancer)（开始使用 PowerShell 在 Resource Manager 中创建面向 Internet 的负载平衡器）。

## 将状况良好的实例添加回到负载平衡器

在以下情况下，会将 TCP 和 HTTP 探测视为状况良好，并将角色实例标记为状况良好：

-  负载平衡器在 VM 首次启动时获得有效探测。
- SuccessFailCount 的数字（如前所述）定义了将角色实例标记为状况良好所需的成功探测值。如果已删除角色实例，成功且连续的探测数目必须大于或等于 SuccessFailCount 的值才能将角色实例标记为正在运行。

>[AZURE.NOTE] 如果角色实例的运行状况有波动，负载平衡器会等待更长时间，然后将角色实例恢复正常状态。这是通过使用策略保护用户和基础结构来实现的。

## 使用适用于负载平衡器的 Log Analytics

可以使用[适用于负载平衡器的 Log Analytics](/documentation/articles/load-balancer-monitor-log/) 来检查探测运行状况和探测计数。可以配合 Power BI 或 Azure Operation Insights 使用日志记录，以提供有关负载平衡器运行状况的统计信息。

<!---HONumber=Mooncake_0822_2016-->