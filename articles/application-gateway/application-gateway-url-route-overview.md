<properties
   pageTitle="基于 URL 的内容路由概述 | Azure"
   description="本页提供基于应用程序网关 URL 的内容路由、UrlPathMap 配置和 PathBasedRouting 规则的概述。"
   documentationCenter="na"
   services="application-gateway"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.date="04/05/2016"
   wacn.date="06/30/2016"/>

# 基于 URL 路径的路由概述

基于 URL 路径的路由可让你根据请求的 URL 路径，将流量路由到后端服务器池。方案之一是将针对不同内容类型的请求路由到不同的后端服务器池。
在以下示例中，应用程序网关针对 contoso.com 从三个后端服务器池提供流量，例如：VideoServerPool、ImageServerPool 和 DefaultServerPool。

![imageURLroute](./media/application-gateway-url-route-overview/figure1.png)

对 http://contoso.com/video* 的请求将路由到 VideoServerPool，对 http://contoso.com/images* 的请求将路由到 ImageServerPool。如果没有任何路径模式匹配，则选择 DefaultServerPool。

## UrlPathMap 配置元素

UrlPathMap 元素是用于指定后端服务器池映射的路径模式。这是模板文件中 urlPathMap 元素的代码段。

	"urlPathMaps": [
	{
    "name": "<urlPathMapName>",
    "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/ urlPathMaps/<urlPathMapName>",
    "properties": {
        "defaultBackendAddressPool": {
            "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/backendAddressPools/<poolName>"
        },
        "defaultBackendHttpSettings": {
            "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/backendHttpSettingsList/<settingsName>"
        },
        "pathRules": [
            {
                "paths": [
                    <pathPattern>
                ],
                "backendAddressPool": {
                    "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/backendAddressPools/<poolName2>"
                },
                "backendHttpsettings": {
                    "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/backendHttpsettingsList/<settingsName2>"
                },

            },

        ],

    }
	}
	

>[AZURE.NOTE] PathPattern：这是要匹配的路径模式列表。每个模式必须以 / 开始，只允许在后接“/”的末尾处添加 *。发送到路径匹配器的字符串不会在第一个 ? 或 # 之后包含任何文本，这些字符在这里是不允许的。

有关详细信息，可以查看[使用基于 URL 的路由的 ARM 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-application-gateway-url-path-based-routing)。

## PathBasedRouting 规则

PathBasedRouting 类型的 RequestRoutingRule 可用于将侦听器绑定到 urlPathMap。针对此侦听器收到的所有请求将根据 urlPathMap 中指定的策略进行路由。PathBasedRouting 规则的代码段：

	"requestRoutingRules": [
  	{

    "name": "<ruleName>",
    "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/requestRoutingRules/<ruleName>",
    "properties": {
        "ruleType": "PathBasedRouting",
        "httpListener": {
            "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/httpListeners/<listenerName>"
        },
        "urlPathMap": {
            "id": "/subscriptions/<subscriptionId>/../microsoft.network/applicationGateways/<gatewayName>/ urlPathMaps/<urlPathMapName>"
        },

    }
	
## 后续步骤 

了解基于 URL 的内容路由之后，请转到[使用基于 URL 的路由创建应用程序网关](/documentation/articles/application-gateway-create-url-route-arm-ps/)，以使用 URL 路由规则创建应用程序网关。

<!---HONumber=Mooncake_0328_2016-->
