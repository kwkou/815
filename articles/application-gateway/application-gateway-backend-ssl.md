<properties
    pageTitle="在应用程序网关上启用 SSL 策略和端到端 SSL | Azure"
    description="此页概述应用程序网关的端到端 SSL 支持。"
    documentationcenter="na"
    services="application-gateway"
    author="amsriva"
    manager="rossort"
    editor="amsriva" />
<tags
    ms.assetid="3976399b-25ad-45eb-8eb3-fdb736a598c5"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.custom="H1Hack27Feb2017"
    ms.workload="infrastructure-services"
    ms.date="04/04/2017"
    wacn.date="05/22/2017"
    ms.author="amsriva"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ae5b4f05b42a25482bc5d464a9ac77b9ebe97b4e"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="overview-of-end-to-end-ssl-and-ssl-policy-on-application-gateway"></a>应用程序网关的端到端 SSL 和 SSL 策略概述

应用程序网关支持在网关上终止 SSL，之后，流量通常会以未加密状态流到后端服务器。 此功能让 Web 服务器不用再负担昂贵的加密和解密开销。 但对于某些客户而言，与后端服务器的未加密通信不是可以接受的选项。 此通信未加密，可能是由于有安全要求、符合性要求，或应用程序可能仅接受安全连接。 对于此类应用程序，应用程序网关支持端到端 SSL 加密。

## <a name="overview"></a>概述

端到端 SSL 允许安全地将敏感数据以加密方式传输到后端，同时仍可利用应用程序网关提供的第 7 层负载均衡功能的好处。 部分功能包括：基于 Cookie 的会话相关性、基于 URL 的路由、基于站点的路由支持，或注入 X-Forwarded-* 标头。

如果配置为端到端 SSL 通信模式，应用程序网关会在网关上终止 SSL 会话，并解密用户流量。 然后，它会应用配置的规则，以选择要将流量路由到的适当后端池实例。 应用程序网关接下来会初始化到后端服务器的新 SSL 连接，并先使用后端服务器的公钥证书重新加密数据，然后再将请求传输到后端。 若要启用端到端 SSL，请将 BackendHTTPSetting 中的协议设置设为 HTTPS，然后再将其应用到后端池。 后端池中每个已启用端到端 SSL 的后端服务器都必须配置证书，以便能够进行安全的通信。

![端到端 ssl 方案][1]

此示例通过端到端 SSL 将使用 TLS1.2 的请求路由到 Pool1 中的后端服务器。

## <a name="end-to-end-ssl-and-whitelisting-of-certificates"></a>端到端 SSL 和证书允许列表

应用程序网关只会与已知的后端实例通信，这些实例已将其证书加入应用程序网关的允许列表。 若要启用证书允许列表，必须将后端服务器证书（不是根证书）的公钥上传到应用程序网关。 只允许连接到已知的和列入允许列表的后端。 其余后端会导致网关错误。 自签名证书仅用于测试目的，不建议用于生产工作负荷。 此类证书也必须按照上述步骤中的说明加入应用程序网关的允许列表，然后才能使用。

## <a name="application-gateway-ssl-policy"></a>应用程序网关 SSL 策略

应用程序网关支持用户可配置的 SSL 协商策略，这些策略允许客户更精细地控制应用程序网关上的 SSL 连接。

1. 默认情况下，所有应用程序网关都禁用 SSL 2.0 和 3.0。 无法对这些策略进行任何配置。
2. SSL 策略定义允许用户禁用以下 3 个协议中的任意协议：**TLSv1\_0**、**TLSv1\_1**、**TLSv1\_2**。
3. 如果没有定义任何 SSL 策略，这 3 个策略（TLSv1\_0、TLSv1\_1、TLSv1_2）会全部启用。

## <a name="next-steps"></a>后续步骤

了解端到端 SSL 及 SSL 策略后，可转到[在应用程序网关上启用端到端 SSL](/documentation/articles/application-gateway-end-to-end-ssl-powershell/)，使用端到端 SSL 创建应用程序网关。

<!--Image references-->

[1]: ./media/application-gateway-backend-ssl/scenario.png

<!--Update_Description: wording update-->