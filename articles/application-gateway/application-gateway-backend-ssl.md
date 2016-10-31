<properties
   pageTitle="在应用程序网关上启用 SSL 策略和端到端 SSL | Azure"
   description="此页概述应用程序网关的端到端 SSL 支持。"
   documentationCenter="na"
   services="application-gateway"
   authors="amsriva"
   manager="rossort"
   editor="amsriva"/>  

<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/26/2016"
   wacn.date="10/31/2016"
   ms.author="amsriva"/>  


# 在应用程序网关上启用 SSL 策略和端到端 SSL

## 概述

应用程序网关支持在网关上终止 SSL，之后，流量通常会以未加密状态流到后端服务器。这让 Web 服务器不用再负担昂贵的加密/解密开销。但对于某些客户而言，与后端服务器的未加密通信不是可以接受的选项。这可能是因为有安全/法规遵从性方面的要求，或应用程序可能仅接受安全连接。对于此类应用程序，应用程序网关现在可支持端到端 SSL 加密。

端到端 SSL 允许安全地将敏感数据以加密方式传输到后端，同时可利用应用程序网关提供的第 7 层负载均衡功能的好处，例如 cookie 相关性、基于 URL 的路由、基于站点的路由支持，或插入 X-Forwarded-* 标头的能力。

如果配置为端到端 SSL 通信模式，应用程序网关会在网关上终止用户的 SSL 会话，并解密用户流量。然后，它会应用配置的规则，以选择要将流量路由到的适当后端池实例。应用程序网关接下来会初始化到后端服务器的新 SSL 连接，并先使用后端服务器的公钥证书重新加密数据，然后再将请求传输到后端。若要启用端到端 SSL，请将 BackendHTTPSetting 中的协议设置设为 Https，然后再将其应用到后端池。后端池中每个已启用端到端 SSL 的后端服务器都必须配置证书，以便能够进行安全的通信。

![imageURLroute](./media/application-gateway-multi-site-overview/multisite.png)  


在此示例中，https://contoso.com 请求可通过 HTTP 路由到 ContosoServerPool，而 https://fabrikam.com 则会使用端到端 SSL 通过 HTTPS 路由到 FabrikamServerPool。

## 端到端 SSL 和证书白名单

应用程序网关只会与已知的后端实例通信，这些实例已将其证书加入应用程序网关的白名单。若要启用证书白名单，必须将后端服务器证书的公钥上传到应用程序网关。之后，只允许连接已经加入白名单的已知后端，其余连接会导致网关错误。自签名证书仅用于测试目的，不建议用于生产工作负荷。此类证书也必须如上所述加入应用程序网关的白名单，然后才可供使用。

## 应用程序网关 SSL 策略

应用程序网关还支持用户可配置的 SSL 协商策略，这些策略允许客户更精细地控制应用程序网关上的 SSL 连接。

1. 所有应用程序网关都强制禁用 SSL 2.0 和 3.0。无法对其进行任何配置。
2. SSL 策略定义提供可禁用以下 3 个协议中任意协议的选项 - TLSv1_0、TLSv1_1、TLSv1\_2。
3. 如果没有定义任何 SSL 策略，这 3 个策略（TLSv1_0、TLSv1_1、TLSv1\_2）会全部启用。

## 后续步骤

了解端到端 SSL 及 SSL 策略后，可转到[在应用程序网关上启用端到端 SSL](/documentation/articles/application-gateway-end-to-end-ssl-powershell/)，创建能够以加密形式将流量发送到后端的应用程序网关。

<!---HONumber=Mooncake_1024_2016-->