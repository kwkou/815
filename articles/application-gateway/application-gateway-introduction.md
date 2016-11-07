<properties
   pageTitle="应用程序网关简介 | Azure"
   description="此页提供第 7 层负载均衡的应用程序网关服务概述，包括网关的大小、HTTP 负载均衡、基于 cookie 的会话相关性和 SSL 卸载。"
   documentationCenter="na"
   services="application-gateway"
   authors="georgewallace"
   manager="carmonm"
   editor="tysonn"/>  

<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/26/2016"
   wacn.date="11/07/2016"
   ms.author="gwallace"/>

# 应用程序网关概述

## 什么是应用程序网关

Azure 应用程序网关以服务形式提供应用程序传送控制器 (ADC)，借此提供许多第 7 层负载均衡功能。简单来说，它的工作原理是，在接受流量后，根据所定义的规则，将流量路由到适当的后端实例。

应用程序负载均衡使 IT 管理员和开发人员能够基于 HTTP 协议为网络流量创建路由规则。应用程序网关服务高度可用，并附带计量功能。有关 SLA 和定价，请参阅 [SLA](/support/legal/sla/) 和[定价](/pricing/details/application-gateway/)页。

应用程序网关将路由规则应用到 HTTP 流量，以提供第 7 层 (HTTP) 负载均衡。创建应用程序网关时，将与一个终结点 (VIP) 相关联并将其用作流入网络流量的公共 IP。Azure 通过在传输层 (TCP/UDP) 运行 Azure 负载均衡器并让所有传入的网络流量负载均衡到应用程序网关服务，来提供第 4 层负载均衡。应用程序网关根据其配置（是虚拟机、云服务还是外部 IP 地址）来路由 HTTP 流量。

## 功能

应用程序网关当前支持具有以下功能的第 7 层应用程序传送：

- **[Web 应用程序防火墙（预览版）](/documentation/articles/application-gateway-webapplicationfirewall-overview/)**- Azure 应用程序网关中的 Web 应用程序防火墙 (WAF) 可保护 Web 应用程序，使其免受常见 Web 攻击的威胁，例如 SQL 注入、跨站点脚本攻击和会话劫持。
- **HTTP 负载均衡** - 应用程序网关提供轮循机制负载均衡。负载均衡在第 7 层完成，仅用于 HTTP(S) 流量。
- **基于 cookie 的会话相关性** - 想要在同一后端保留用户会话时，此功能十分有用。借助受网关管理的 cookie，应用程序网关能够将来自用户会话的后续流量转到同一后端进行处理。在会话状态是为用户会话而本地保存在后端服务器的情况下，此功能十分重要。
- **[安全套接字层 (SSL) 卸载](/documentation/articles/application-gateway-ssl-arm/)** - 此功能让 Web 服务器免于执行解密 HTTPS 流量的高成本任务。通过在应用程序网关终止 SSL 连接，并将请求转发到未加密的服务器，Web 服务器不用承担解密的负担。应用程序网关会重新加密响应，然后再将它发回客户端。在后端与 Azure 中的应用程序网关位于同一安全虚拟网络中的情况下，此功能十分有用。
- **[端到端 SSL](/documentation/articles/application-gateway-backend-ssl/)** - 应用程序网关支持对流量进行端到端加密。应用程序网关通过在应用程序网关上终止 SSL 连接来完成此任务。网关随后将路由规则应用于流量、重新加密数据包，并根据定义的路由规则将数据包转发到适当的后端。来自 Web 服务器的任何响应都会经历相同的过程返回最终用户。
- **[基于 URL 的内容路由](/documentation/articles/application-gateway-url-route-overview/)** - 此功能能够使用不同的后端服务器来处理不同的流量。可将 Web 服务器上的文件夹流量或 CDN 流量路由到不同的后端，让不提供特定内容的后端减少不必要的负载。
- **[多站点路由](/documentation/articles/application-gateway-multi-site-overview/)** - 应用程序网关允许在单个应用程序网关上合并最多 20 个网站。
- **[WebSocket 支持](/documentation/articles/application-gateway-websocket/)** - 应用程序网关的另一个重要功能是对 WebSocket 的本机支持。
- **[运行状况监视](/documentation/articles/application-gateway-probe-overview/)** - 应用程序网关提供默认的后端资源运行状况监视，以及用于监视更多特定方案的自定义探测。

## 优点

应用程序网关可用于：

- 需要使用来自同一用户/客户端会话的请求来访问相同后端虚拟机的应用程序。这些应用程序的示例包括购物车应用和 Web 邮件服务器。
- 希望 Web 服务器场不产生 SSL 终端开销的应用程序。
- 要求长时间运行的同一 TCP 连接上多个 HTTP 请求路由到或负载均衡到不同后端服务器的应用程序（例如内容交付网络）。
- 支持 WebSocket 流量的应用程序
- 保护 Web 应用程序，使其免受常见 Web 攻击的威胁，例如 SQL 注入、跨站点脚本攻击和会话劫持。

[AZURE.INCLUDE [load-balancer-compare-tm-ag-lb-include.md](../../includes/load-balancer-compare-tm-ag-lb-include.md)]

## 网关大小和实例

应用程序网关目前有三种大小：小型、中型和大型。小型实例大小适用于开发和测试方案。

目前，有两个应用程序网关 SKU：WAF 和标准。

最多可为每个订阅创建 50 个应用程序网关，每个应用程序网关最多可有 10 个实例。每个应用程序网关可以包含 20 个 http 侦听器。Azure 托管服务形式的应用程序网关负载均衡允许在 Azure 软件负载均衡器的后面预配第 7 层负载均衡器。

下表显示了每个应用程序网关实例的平均性能吞吐量：

| 后端页面响应 | 小型 | 中型 | 大型|
|---|---|---|---|
| 6K | 7\.5 Mbps | 13 Mbps | 50 Mbps |
|100K | 35 Mbps | 100 Mbps| 200 Mbps |

>[AZURE.NOTE] 这些值是应用程序网关吞吐量的大约值。实际吞吐量取决于平均页面大小、后端实例的位置、提供页面所需的处理时间等各种环境详细信息。如需确切的性能数据，应该运行自己的测试，此处提供的这些值仅用于容量计划指导目的。

## 运行状况监视

Azure 应用程序网关会通过基本或自定义运行状况探测，自动监视后端实例的运行状况。使用运行状况探测可确保只有正常的主机才会响应流量。有关详细信息，请参阅[应用程序网关运行状况监视概述](/documentation/articles/application-gateway-probe-overview/)。

## 配置和管理

对于其终结点，应用程序网关在配置时可以拥有公共 IP、专用 IP 或同时拥有两者。应用程序网关在其自己的子网中的虚拟网络内进行配置。为应用程序网关创建或使用的子网不能包含任何其他类型的资源，子网中唯一允许使用的资源是其他应用程序网关。若要保护后端资源，可以将后端服务器放在与应用程序网关位于同一虚拟网络中的其他子网内。后端应用程序不需要这个附加的子网，只要应用程序网关可以连接到 IP 地址，应用程序网关就能够为后端服务器提供 ADC 功能。

可以使用 REST API、PowerShell cmdlet、Azure CLI 或 [Azure 门户预览](https://portal.azure.cn/)来创建和管理应用程序网关。

## 后续步骤

了解应用程序网关后，可以[创建应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)，也可以[创建应用程序网关 SSL 卸载](/documentation/articles/application-gateway-ssl-arm/)，以便对 HTTPS 连接进行负载均衡。

若要详细了解如何使用基于 URL 的内容路由创建应用程序网关，请转到[使用基于 URL 的路由创建应用程序网关](/documentation/articles/application-gateway-create-url-route-arm-ps/)。

<!---HONumber=Mooncake_1024_2016-->