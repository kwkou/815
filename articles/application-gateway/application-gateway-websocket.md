<properties
   pageTitle="应用程序网关 WebSocket 支持 | Azure"
   description="此页概述了应用程序网关的 WebSocket 支持。"
   documentationCenter="na"
   services="application-gateway"
   authors="amsriva"
   manager="rossort"
   editor="amsriva"/>  

<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/16/2016"
   wacn.date="12/30/2016"
   ms.author="amsriva"/>  


# 应用程序网关 WebSocket 支持

以 [RFC6455](https://tools.ietf.org/html/rfc6455) 进行标准化的 WebSocket 协议通过长时间运行的 TCP 连接，让服务器和客户端之间实现全双工通信。此功能让 Web 服务器和客户端之间能够进行交互性更强的通信。这种通信可以是双向的，而且不像基于 HTTP 的实现那样需要轮询。不同于 HTTP，WebSocket 的开销很低，并且可以对多个请求/响应重复使用同一 TCP 连接，进而提高资源利用率。WebSocket 协议设计为通过传统 HTTP 端口 80 和 443 运行。

应用程序网关跨所有网关大小为 WebSocket 提供本机支持。用户无法通过配置设置来选择性地启用或禁用 WebSocket 支持。可以在端口 80/443 上继续使用标准 HTTPListener 来接收 WebSocket 流量。随后会使用应用程序网关规则中指定的相应后端池，将 WebSocket 流量定向到已启用 WebSocket 的后端服务器。

后端服务器必须响应应用程序网关探测，如[运行状况探测概述](/documentation/articles/application-gateway-probe-overview/)部分中所述。应用程序网关运行状况探测仅限 HTTP/HTTPS，这意味着每个后端服务器都必须响应 HTTP 探测，应用程序网关才能将 WebSocket 流量路由到服务器。


## 侦听器配置元素

现有的 HTTPListener 可用于支持 WebSocket。以下是示例模板文件中 HttpListeners 元素的代码片段。需要同时拥有 HTTP 和 HTTPS 侦听器才能支持 WebSocket 并保护 WebSocket 流量。同样，可以使用[门户](/documentation/articles/application-gateway-create-gateway-portal/)或 [PowerShell](/documentation/articles/application-gateway-create-gateway-arm/) 在端口 80/443 上创建具有侦听器的应用程序网关，以支持 WebSocket 通信。


    "httpListeners": [
                {
                    "name": "appGatewayHttpsListener",
                    "properties": {
                        "FrontendIPConfiguration": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/DefaultFrontendPublicIP"
                        },
                        "FrontendPort": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort443'"
                        },
                        "Protocol": "Https",
                        "SslCertificate": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/sslCertificates/appGatewaySslCert1'"
                        },
                    }
                },
                {
                    "name": "appGatewayHttpListener",
                    "properties": {
                        "FrontendIPConfiguration": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendIPConfigurations/appGatewayFrontendIP'"
                        },
                        "FrontendPort": {
                            "Id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/frontendPorts/appGatewayFrontendPort80'"
                        },
                        "Protocol": "Http",
                    }
                }
            ],

## BackendAddressPool、BackendHttpSetting 和路由规则配置

如果后端池具有已启用 WebSocket 的服务器，那么应使用 BackendAddressPool 对其进行定义。只能使用后端端口 80/443 对 BackendHttpSetting 进行定义。基于 cookie 的相关性和 requestTimeouts 的属性与 WebSocket 流量不相关。不需更改路由规则。应继续使用“基本”路由规则，以便将适当的侦听器绑定到相应的后端地址池。

	"requestRoutingRules": [{
		"name": "<ruleName1>",
		"properties": {
			"RuleType": "Basic",
			"httpListener": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpsListener')]"
			},
			"backendAddressPool": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/ContosoServerPool')]"
			},
			"backendHttpSettings": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
			}
		}

	}, {
		"name": "<ruleName2>",
		"properties": {
			"RuleType": "Basic",
			"httpListener": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/httpListeners/appGatewayHttpListener')]"
			},
			"backendAddressPool": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/ContosoServerPool')]"
			},
			"backendHttpSettings": {
				"id": "/subscriptions/<subid>/resourceGroups/<rgName>/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings')]"
			}

		}
	}]

## 已启用 WebSocket 的后端

后端必须具有在已配置端口（通常为 80/443）上运行的 HTTP/HTTPS Web 服务器，WebSocket 才能运行。此要求是因为 WebSocket 协议要求初始握手是 HTTP，且标头字段为升级到 WebSocket 协议。

	GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Origin: http://example.com
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13

另一个原因是该应用程序网关后端运行状况探测仅支持 HTTP/HTTPS 协议。如果后端服务器没有响应 HTTP/HTTPS 探测，它将被移出后端池，且包括 WebSocket 请求在内的任何请求都无法到达此后端。

## 后续步骤

了解 WebSocket 支持后，请转到[创建应用程序网关](/documentation/articles/application-gateway-create-gateway/)，开始使用已启用 WebSocket 的 Web 应用程序。

<!---HONumber=Mooncake_1010_2016-->