<properties
   pageTitle="应用程序网关简介 | Azure"
   description="此页提供第 7 层负载平衡的应用程序网关服务概述，包括网关的大小、HTTP 负载平衡、基于 Cookie 的会话相关性和 SSL 卸载。"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.date="02/18/2016"
   wacn.date="04/05/2016"/>

# 应用程序网关概述

Azure 应用程序网关提供基于第 7 层负载平衡的 Azure 托管 HTTP 负载平衡解决方案。

应用程序负载平衡使 IT 管理员和开发人员能够基于 HTTP 为网络流量创建路由规则。应用程序网关服务高度可用，并附带计量功能。有关 SLA 和定价，请参阅 [SLA](/support/legal/sla) 和[定价](/home/features/application-gateway/pricing/)页。

应用程序网关当前支持以下第 7 层应用程序传送功能：

- HTTP 负载平衡
- 基于 Cookie 的会话相关性
- [安全套接字层 (SSL) 卸载](/documentation/articles/application-gateway-ssl-arm/)
- [基于 URL 的内容路由](/documentation/articles/application-gateway-url-route-overview/) 

## HTTP 第 7 层负载平衡

Azure 通过在传输层 (TCP/UDP) 运行 Azure 负载平衡器并让所有传入的网络流量负载平衡到应用程序网关服务，来提供第 4 层负载平衡。然后，应用程序网关将路由规则应用到 HTTP 流量，以提供第 7 层 (HTTP) 负载平衡。当你创建应用程序网关时，将与一个终结点 (VIP) 相关联并将其用作输入网络流量的公共 IP。

应用程序网关将会根据其配置（是虚拟机、云服务、Web 应用还是外部 IP 地址）来路由 HTTP 流量。

HTTP 第 7 层负载平衡适合用于：

- 需要使用来自同一用户/客户端会话的请求来访问相同后端虚拟机的应用程序。此类应用程序的示例包括购物车应用程序和 Web 邮件服务器。
- 希望 Web 服务器场不产生 SSL 终端开销的应用程序。
- 要求长时间运行的同一 TCP 连接上多个 HTTP 请求路由到或负载平衡到不同后端服务器的应用程序（例如内容传送网络）。

[AZURE.INCLUDE [load-balancer-compare-tm-ag-lb-include.md](../includes/load-balancer-compare-tm-ag-lb-include.md)]

## 网关大小和实例

应用程序网关目前有三种大小：小型、中型和大型。小型实例大小适用于开发和测试方案。

最多可为每个订阅创建 50 个应用程序网关，每个应用程序网关最多可有 10 个实例。Azure 托管服务形式的应用程序网关负载平衡允许在 Azure 软件负载平衡器的后面预配第 7 层负载平衡器。

下表显示了每个应用程序网关实例的平均性能吞吐量：


| 后端页面响应 | 小型 | 中型 | 大型|
|---|---|---|---|
| 6K | 7\.5 Mbps | 13 Mbps | 50 Mbps |
|100K | 35 Mbps | 100 Mbps| 200 Mbps |


>[AZURE.NOTE] 这是有关应用程序网关吞吐量的大约指导数据。实际吞吐量取决于平均页面大小、后端实例的位置、提供页面所需的处理时间等各种环境详细信息。

## 运行状况监视

Azure 应用程序网关自动监视后端实例的运行状况。有关详细信息，请参阅[应用程序网关运行状况监视概述](/documentation/articles/application-gateway-probe-overview/)。

## 配置和管理

可以使用 REST API 和 PowerShell cmdlet 来创建及管理应用程序网关。


## 后续步骤

了解应用程序网关以后，你可以[创建应用程序网关](/documentation/articles/application-gateway-create-gateway/)，也可以[创建应用程序网关 SSL 卸载](/documentation/articles/application-gateway-ssl/)，以便对 HTTPS 连接进行负载平衡。

若要详细了解如何使用基于 URL 的内容路由创建应用程序网关，请转到[使用基于 URL 的路由创建应用程序网关](/documentation/articles/application-gateway-create-url-route-arm-ps/)。


<!---HONumber=Mooncake_0328_2016-->
