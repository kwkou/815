<properties
    pageTitle="测试流量管理器设置 | Azure"
    description="本文将帮助你测试流量管理器设置"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date="11/07/2016"
    ms.author="sewhee"
/>  


# 测试流量管理器设置

若要测试流量管理器设置，需要在不同的位置准备多个客户端，以便从中运行测试。然后，逐个关闭流量管理器配置文件中的终结点。

* 为 DNS TTL 设置一个较小的值（例如 30 秒），以便快速传播更改。
* 了解你要测试的配置文件中的 Azure 云服务和网站的 IP 地址。
* 使用工具将 DNS 名称解析为 IP 地址并显示该地址。

检查 DNS 名称是否可解析为配置文件中的终结点的 IP 地址。名称的解析方式应与流量管理器配置文件中定义的流量路由方法一致。可以使用 **nslookup** 或 **dig** 等工具来解析 DNS 名称。

以下示例可帮助测试流量管理器配置文件。

### 在 Windows 中使用 nslookup 和 ipconfig 检查流量管理器配置文件

1. 以管理员身份打开命令提示符或 Windows PowerShell 提示符。
2. 键入 `ipconfig /flushdns` 以刷新 DNS 解析程序缓存。
3. 键入 `nslookup <your Traffic Manager domain name>`。例如，以下命令将检查前缀为 *myapp.contoso* 的域名

        nslookup myapp.contoso.trafficmanager.cn

    典型的结果会显示以下信息：

    * 为解析此流量管理器域名而访问的 DNS 服务器的 DNS 名称和 IP 地址。
    * 在命令行中“nslookup”后键入的流量管理器域名以及该流量管理器域解析为的 IP 地址。需要重点检查第二个 IP 地址。它应当与所测试的流量管理器配置文件中某个云服务或网站的公用虚拟 IP (VIP) 地址匹配。

## 如何测试故障转移流量路由方法

1. 使所有终结点保持运行状态。
2. 在单个客户端中，使用 nslookup 或类似的实用工具请求对公司域名进行 DNS 解析。
3. 确保解析的 IP 地址与主终结点匹配。
4. 关闭主终结点或删除监视文件，使流量管理器认为应用程序已关闭。
5. 等待流量管理器配置文件的 DNS 生存时间 (TTL)，再额外等待两分钟。例如，如果 DNS TTL 为 300 秒（5 分钟），则必须等待 7 分钟。
6. 刷新 DNS 客户端缓存，然后使用 nslookup 请求 DNS 解析。在 Windows 中，可以使用 ipconfig /flushdns 命令刷新 DNS 缓存。
7. 确保解析的 IP 地址与辅助终结点匹配。
8. 重复该过程，依次关闭每个终结点。检查 DNS 是否返回列表中下一个终结点的 IP 地址。关闭所有终结点后，应该再次得到主终结点的 IP 地址。

## 如何测试加权流量路由方法

1. 使所有终结点保持运行状态。
2. 在单个客户端中，使用 nslookup 或类似的实用工具请求对公司域名进行 DNS 解析。
3. 确保解析的 IP 地址与某个终结点匹配。
4. 刷新 DNS 客户端缓存，针对每个终结点重复步骤 2 和 3。你应该会看到，为每个终结点返回的 IP 地址都不相同。

## 如何测试性能流量路由方法

若要有效地测试性能流量路由方法，你必须在世界各地拥有客户端。可以在不同的 Azure 区域中创建用于测试服务的客户端。如果你有全球网络，可以远程登录到位于世界另一地点的客户端，从中运行测试。

或者，可以使用基于 Web 的免费 DNS 查找和挖掘服务。使用其中的某些工具可以从全球的不同位置检查 DNS 名称解析。例如，针对“DNS 查找”执行搜索。可以使用 Gomez 或 Keynote 等第三方服务来确认配置文件是否按预期分布流量。

## 后续步骤

* [关于流量管理器流量路由方法](/documentation/articles/traffic-manager-routing-methods/)
* [流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations/)
* [流量管理器降级状态疑难解答](/documentation/articles/traffic-manager-troubleshooting-degraded/)

<!---HONumber=Mooncake_1031_2016-->