<properties
    pageTitle="Azure 应用程序网关常见问题 | Azure"
    description="本页提供有关 Azure 应用程序网关常见问题的解答"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="d54ee7ec-4d6b-4db7-8a17-6513fda7e392"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/28/2017"
    wacn.date=""
    ms.author="gwallace"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="75890c3ffb1d1757de64a8b8344e9f2569f26273"
    ms.openlocfilehash="53cfe8ed2aef020aa6b6683b0c7305d2448eb140"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="04/25/2017" />

# <a name="frequently-asked-questions-for-application-gateway"></a>应用程序网关常见问题

## <a name="general"></a>常规

**问：什么是应用程序网关？**

Azure 应用程序网关是服务形式的应用程序传送控制器 (ADC)，借此为应用程序提供各种第 7 层负载均衡功能。 它提供完全由 Azure 管理的高度可用、可缩放的服务。

**问：应用程序网关支持哪些功能？**

应用程序网关支持 SSL 卸载和端到端 SSL、Web 应用程序防火墙（预览版）、基于 Cookie 的会话相关性、基于 URL 路径的路由、多站点托管，等等。 有关支持的功能的完整列表，请访问 [Introduction to Application Gateway](/documentation/articles/application-gateway-introduction/)（应用程序网关简介）

**问：应用程序网关与 Azure 负载均衡器之间有什么区别？**

应用程序网关是第 7 层负载均衡器。 这意味着，应用程序网关只处理 Web 流量 (HTTP/HTTPS/WebSocket)。 它支持应用程序负载均衡功能，例如 SSL 终止、基于 Cookie 的会话相关性，以及对流量进行负载均衡的轮循机制。 负载均衡器在第 4 层对流量进行负载均衡 (TCP/UDP)。

**问：应用程序网关支持哪些协议？**

应用程序网关支持 HTTP、HTTPS 和 WebSocket。

**问：目前支持在后端池中添加哪些资源？**

后端池可以包含 NIC、虚拟机规模集、公共 IP、内部 IP 和完全限定的域名 (FQDN)。 目前不提供 Azure Web 应用支持。 应用程序网关后端池成员不会绑定到可用性集。 后端池的成员可以跨群集、数据中心，或者在 Azure 外部，前提是它们建立了 IP 连接。

**问：应用程序网关是订阅专门的部署，还是在所有客户之间共享？**

应用程序网关是虚拟网络中的专用部署。

**问：是否支持 HTTP 到 HTTPS 的重定向？**

目前不支持。

**问：在何处可以找到应用程序网关的 IP 和 DNS？**

使用公共 IP 地址作为终结点时，可在公共 IP 地址资源中，或者在门户中应用程序网关的“概述”页上找到此信息。 对于内部 IP 地址，可在“概述”页上找到此信息。

**问：在应用程序网关的生存期内，其 IP 或 DNS 是否会变化？**

如果客户停止再启动网关，VIP 可能会变化。 与应用程序网关关联的 DNS 在网关的整个生命周期内不会变化。 出于此原因，建议使用 CNAME 别名并使其指向应用程序网关的 DNS 地址。

**问：应用程序网关是否支持静态 IP？**

应用程序网关不支持静态公共 IP 地址，但支持静态内部 IP。

**问：应用程序网关是否支持在网关上使用多个公共 IP？**

应用程序网关仅支持一个公共 IP 地址。

**问：应用程序网关是否支持 x-forwarded-for 标头？**

支持。应用程序网关会将 x-forwarded-for、x-forwarded-proto 和 x-forwarded-port 标头插入转发到后端的请求中。 x-forwarded-for 标头的格式是逗号分隔的“IP:端口”列表。 x-forwarded-proto 的有效值为 http 或 https。 x-forwarded-port 指定请求抵达应用程序网关时所在的端口。

## <a name="configuration"></a>配置

**问：是否始终要将应用程序网关部署在虚拟网络中？**

是的，始终要将应用程序网关部署在虚拟网络子网中。 此子网只能包含应用程序网关。

**问：应用程序网关是否能够与其虚拟网络外部的实例通信？**

应用程序网关可与其所在的虚拟网络外部的实例通信，前提是已建立 IP 连接。 如果打算使用内部 IP 作为后端池成员，则需要使用 [VNET 对等互连](/documentation/articles/virtual-network-peering-overview/)或 [VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。

**问：是否可以在应用程序网关子网中部署其他任何组件？**

不可以。但可以在子网中部署其他应用程序

**问：网关应用程序网关子网是否支持网络安全组？**

应用程序网关子网支持网络安全组，但必须为端口 65503-65534 添加例外，这样才能使后端正常运行。 不应阻止出站 Internet 连接。

**问：应用程序网关有哪些限制？是否可以提高这些限制？**

请访问[应用程序网关限制](/documentation/articles/azure-subscription-service-limits/#application-gateway-limits)查看限制。

**问：是否可以同时对外部和内部流量使用应用程序网关？**

可以。每个应用程序网关支持一个内部 IP 和一个外部 IP。

**问：是否支持 VNet 对等互连？**

是的，支持 VNet 对等互连，这有助于对其他虚拟网络中的流量进行负载均衡。

**问：如果通过 ExpressRoute 或 VPN 隧道连接本地服务器，是否可与这些服务器通信？**

可以，只要允许这种流量。

**问：是否可以使用一个后端池来为不同端口上的许多应用程序提供服务？**

支持微服务体系结构。 需要配置多个 http 设置才能探测不同的端口。

**问：自定义探测是否支持对响应数据使用通配符/正则表达式？**

自定义探测不支持对响应数据使用通配符或正则表达式。

**问：自定义探测的 Host 字段是什么意思？**

Host 字段指定要将探测数据发送到的名称。 仅在应用程序网关上配置了多站点的情况下适用，否则使用“127.0.0.1”。 此值不同于 VM 主机名，它采用 \<协议\>://\<主机\>:\<端口\>\<路径\> 格式。 

## <a name="performance"></a>性能

**问：应用程序网关如何支持高可用性和可伸缩性？**

如果已部署 2 个以上的实例，则应用程序网关支持高可用性方案。 Azure 将跨更新域和容错域分配这些实例，确保所有实例不会同时发生故障。 为了支持可伸缩性，应用程序网关将添加同一网关的多个实例来分担负载。

**问：如何使用应用程序网关实现跨数据中心的灾难恢复方案？**

客户可以使用流量管理器跨不同数据中心内的多个应用程序网关分配流量。

**问：是否支持自动缩放？**

不支持，但应用程序网关提供吞吐量指标，达到阈值时，可使用该指标发出警报。 手动添加实例或更改大小不会重新启动网关，且不会影响现有流量。

**问：手动扩展/缩减是否导致停机？**

不会出现停机，实例将跨升级域和容错域分布。

**问：是否可以在不造成中断的情况下，将实例大小从中型更改为大型？**

可以。Azure 将跨更新域和容错域分配实例，确保所有实例不会同时发生故障。 为了支持可伸缩性，应用程序网关将添加同一网关的多个实例来分担负载。

## <a name="ssl-configuration"></a>SSL 配置

**问：应用程序网关支持哪些证书？**

支持自签名证书、CA 证书和通配符证书。 不支持 EV 证书。

**问：应用程序网关支持哪些最新的加密套件？**

下面按优先顺序列出了支持的最新加密套件。

TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384_P384

TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256_P256

TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384_P256

TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256_P256

TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA_P256

TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA_P256

TLS_RSA_WITH_AES_256_GCM_SHA384

TLS_RSA_WITH_AES_128_GCM_SHA256

TLS_RSA_WITH_AES_256_CBC_SHA256

TLS_RSA_WITH_AES_128_CBC_SHA256

TLS_RSA_WITH_AES_256_CBC_SHA

TLS_RSA_WITH_AES_128_CBC_SHA

TLS_RSA_WITH_3DES_EDE_CBC_SHA

**问：应用程序网关是否也支持重新加密发往后端的流量？**

是的，应用程序网关支持 SSL 卸载和端到端 SSL，因此支持重新加密发往后端的流量。

**问：是否可以配置 SSL 策略来控制 SSL 协议版本？**

是的，可将应用程序网关配置为拒绝 TLS1.0、TLS1.1 和 TLS1.2。 SSL 2.0 和 3.0 默认已禁用，并且不可配置。

**问：是否可以配置 SSL 策略来控制加密套件？**

目前不可以。

**问：支持多少个 SSL 证书？**

最多支持 20 个 SSL 证书。

**问：支持使用多少个身份验证证书进行后端重新加密？**

最多支持 10 个身份验证证书，默认为 5 个。

**问：应用程序网关是否原生与 Azure Key Vault 集成？**

不是，它没有与 Azure Key Vault 集成。

## <a name="web-application-firewall-waf-configuration"></a>Web 应用程序防火墙 (WAF) 配置

**问：WAF SKU 是否提供标准 SKU 所提供的全部功能？**

是的，WAF 支持标准 SKU 中的所有功能。

**问：应用程序网关支持哪个 CRS 版本？**

应用程序网关支持 CRS [2.2.9](/documentation/articles/application-gateway-crs-rulegroups-rules/#owasp229) 和 CRS [3.0](/documentation/articles/application-gateway-crs-rulegroups-rules/#owasp30)。

**问：如何监视 WAF？**

可通过诊断日志记录监视 WAF。有关诊断日志记录的详细信息，请参阅 [Diagnostics Logging and Metrics for Application Gateway](/documentation/articles/application-gateway-diagnostics/)（应用程序网关的诊断日志记录和指标）

**问：检测模式是否会阻止流量？**

不会。检测模式仅记录触发了 WAF 规则的流量。

**问：如何自定义 WAF 规则？**

是的，WAF 规则可自定义，有关如何自定义这些规则的详细信息，请访问[自定义 WAF 规则组和规则](/documentation/articles/application-gateway-customize-waf-rules-portal/)

**问：目前支持哪些规则？**

WAF 目前支持 CRS [2.2.9](/documentation/articles/application-gateway-crs-rulegroups-rules/#owasp229) 和 CRS [3.0](/documentation/articles/application-gateway-crs-rulegroups-rules/#owasp30)，这些规则针对开放 Web 应用程序安全项目 (OWASP) 识别到的 10 大漏洞中的大多数漏洞提供基准安全要求，相关信息请参阅 [OWASP top 10 Vulnerabilities](https://www.owasp.org/index.php/Top10#OWASP_Top_10_for_2013)（OWASP 10 大漏洞）

* SQL 注入保护

* 跨站点脚本保护

* 常见 Web 攻击保护，例如命令注入、HTTP 请求走私、HTTP 响应拆分和远程文件包含攻击

* 防止 HTTP 协议违反行为

* 防止 HTTP 协议异常行为，例如缺少主机用户代理和接受标头

* 防止自动程序、爬网程序和扫描程序

* 检测常见应用程序错误配置（即 Apache、IIS 等）

**问：WAF 是否也支持 DDoS 防护？**

否，WAF 不提供 DDoS 防护。

## <a name="diagnostics-and-logging"></a>诊断和日志记录

**问：应用程序网关可以使用哪些类型的日志？**

应用程序网关可以使用三种日志。 有关这些日志和其他诊断功能的详细信息，请访问 [Backend health, diagnostics logging and metrics for Application Gateway](/documentation/articles/application-gateway-diagnostics/)（应用程序网关的后端运行状况、诊断日志记录和指标）。

- **ApplicationGatewayAccessLog** - 此日志包含提交到应用程序网关前端的每个请求。 数据包括调用方的 IP、请求的 URL、响应延迟、返回代码，以及传入和传出的字节数。 每隔 300 秒会收集一次访问日志。 此日志包含每个应用程序网关实例的一条记录。
- **ApplicationGatewayPerformanceLog** - 此日志捕获每个实例的性能信息，包括提供的请求总数、吞吐量（以字节为单位）、失败的请求计数、正常和不正常的后端实例计数。
- **ApplicationGatewayFirewallLog** - 此日志包含通过应用程序网关的检测或阻止模式（通过 Web 应用程序防火墙配置）记录的请求。

**问：如何知道后端池成员是否正常？**

可以使用 PowerShell cmdlet `Get-AzureRmApplicationGatewayBackendHealth`，或者在门户中访问[应用程序网关诊断](/documentation/articles/application-gateway-diagnostics/)来验证运行状况

**问：什么是诊断日志的保留策略？**

诊断日志将发往客户存储帐户，客户可以根据偏好设置保留策略。 此外，可将诊断日志发送到事件中心或 Log Analytics。 有关详细信息，请访问 [Application Gateway Diagnostics](/documentation/articles/application-gateway-diagnostics/)（应用程序网关诊断）。

**问：如何获取应用程序网关的审核日志？**

应用程序网关有相应的审核日志。 在门户上的应用程序网关菜单边栏选项卡中单击“活动日志”即可访问审核日志。 

**问：是否可以使用应用程序网关设置警报？**

可以，应用程序网关确实支持警报。可以基于指标设置警报。  应用程序网关目前提供“吞吐量”指标，可以使用它来配置警报。 若要了解有关警报的详细信息，请访问 [Receive alert notifications](/documentation/articles/insights-receive-alert-notifications/)（接收警报通知）。

**问：后端运行状况返回未知状态，原因是什么？**

最常见的原因是访问的后端被 NSG 或自定义 DNS 阻止。 有关详细信息，请访问 [Backend health, diagnostics logging, and metrics for Application Gateway](/documentation/articles/application-gateway-diagnostics/)（应用程序网关的后端运行状况、诊断日志记录和指标）。

## <a name="next-steps"></a>后续步骤

若要了解有关应用程序网关的详细信息，请访问 [Introduction to Application Gateway](/documentation/articles/application-gateway-introduction/)（应用程序网关简介）。

<!--Update_Description: adding WAF-->