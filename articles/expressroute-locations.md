<properties
   pageTitle="ExpressRoute 位置"
   description="本页详细说明了服务的上市区域，以及如何连接到 Azure 区域。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="carolz"
   editor="tysonn" />
<tags 
   ms.service="expressroute"
   ms.date="08/06/2015"
   wacn.date="11/02/2015" />

# ExpressRoute 合作伙伴和对等位置
本页上的表格提供有关 ExpressRoute 连接提供商、ExpressRoute 地理覆盖范围、通过 ExpressRoute 支持的 Windows Azure 服务以及 ExpressRoute 系统集成商 (SI) 的信息。

## ExpressRoute 连接提供商
所有的 Azure 区域和位置都支持 ExpressRoute。以下地图提供了 Azure 区域和 ExpressRoute 位置的列表。ExpressRoute 位置是指 Azure 与连接提供商对等互连的位置。
 
![](./media/expressroute-locations/expressroute-locations-map.png)

下表提供了Azure区域内 ExpressRoute 位置与 Azure 区域的映射。

|**区域**|**Azure 区域**|**ExpressRoute 位置**|
|---|---|---|
|**中国**|中国北部|北京|
|**中国**|中国东部|上海|

不支持跨地缘政治区域的连接。你可以与连接提供商合作，使用其网络将连接扩展到不同的地缘政治区域。


## 连接提供商的位置

| **服务提供商** |**Windows Azure** | **Office 365** | **位置** |
|-----------------------|--------------------|----------------|---------------|
| **北京电信** | 支持 | 不支持 | 北京 |
| **上海电信** | 支持 | 不支持 | 上海 |

有关设置连接的步骤，请参阅[配置 Expressroute 连接](/documentation/articles/expressroute-configuring-exps)。

## 通过未列出的连接提供商建立连接 

如果上一部分中未列出你的连接提供商，你仍可以建立连接。

- 请咨询你的连接提供商，以了解他们是否连接到列出的连接服务商位置中节点。

## 后续步骤
- 验证你是否符合 [ExpressRoute 先决条件](/documentation/articles/expressroute-prerequisites)。
- 有关详细信息，请访问[常见问题](/documentation/articles/expressroute-faqs)。
- 如果要配置 ExpressRoute 连接，请参阅[配置 Expressroute 连接](/documentation/articles/expressroute-configuring-exps)。
- 如果要同时为同一虚拟网络配置站点到站点 VPN 连接和 ExpressRoute，请参阅[配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接](/documentation/articles/expressroute-coexist)。
 

<!---HONumber=69-->