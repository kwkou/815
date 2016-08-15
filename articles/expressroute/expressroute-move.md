<properties
   pageTitle="将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型 | Azure"
   description="本页概述桥接经典部署模型与 Resource Manager 部署模型时所要了解的知识。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.date="04/01/2016"
   wacn.date="06/06/2016"/>

# 将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型

本文概述将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型的效果。

[AZURE.INCLUDE [vpn-gateway-sm-rm](../includes/vpn-gateway-classic-rm-include.md)]

可以使用一条 ExpressRoute 线路连接到在经典部署模型和 Resource Manager 部署模型中部署的虚拟网络。无论 ExpressRoute 线路的创建方式为何，现在都可以链接到这两种部署模型中的虚拟网络。

![跨两种部署模型链接到虚拟网络的 ExpressRoute 线路](./media/expressroute-move/expressroute-move-1.png)

## 在经典部署模型中创建的 ExpressRoute 线路

在经典部署模型中创建的 ExpressRoute 线路需先转移到 Resource Manager 部署模型，才能连接到经典部署模型和 Resource Manager 部署模型。转移连接时，不会发生连接断开的情况。经典部署模型中所有从线路到虚拟网络的链接（在同一订阅中的链接和跨订阅链接）将会保留。

成功完成转移后，ExpressRoute 线路的感观和执行方式与在 Resource Manager 部署模型中创建的 ExpressRoute 线路完全相同。现在，你可以在 Resource Manager 部署模型中建立与虚拟网络的连接。

将 ExpressRoute 线路转移到 Resource Manager 部署模型后，只能使用 Resource Manager 部署模型来管理 ExpressRoute 线路的生命周期。这意味着，某些操作（例如，添加/更新/删除对等互连，更新带宽、SKU 和计费类型等线路属性，以及删除线路）只能在 Resource Manager 部署模型中执行。请参阅下面关于在 Resource Manager 部署模型中创建的线路的部分，以便进一步详细了解如何管理对这两种部署模型的访问权限。

不需要与连接服务提供商合作即可执行转移。

## 在 Resource Manager 部署模型中创建的 ExpressRoute 线路

你可以允许从这两种部署模型访问在 Resource Manager 部署模型中创建的 ExpressRoute 线路。订阅中的任何 ExpressRoute 线路都可以从这两种部署模型访问。

- 默认情况下，在 Resource Manager 部署模型中创建的 ExpressRoute 线路无法访问经典部署模型。
- 默认情况下，可以从这两种部署模型访问从经典部署模型转移到 Resource Manager 部署模型的 ExpressRoute 线路。
- 无论是在 Resource Manager 部署模型还是经典部署模型中创建的，ExpressRoute 线路始终都可以访问 Resource Manager 部署模型。这意味着，你可以根据[如何链接虚拟网络](/documentation/articles/expressroute-howto-linkvnet-arm/)中的说明，与 Resource Manager 部署模型中创建的虚拟网络建立连接。
- 对经典部署模型的访问权限由 ExpressRoute 线路中的 **allowClassicOperations** 参数控制。

>[AZURE.IMPORTANT] 将应用[服务限制](/documentation/articles/azure-subscription-service-limits/)页中所述的所有配额。例如，标准线路最多可以有 10 个跨经典部署模型和 Resource Manager 部署模型的虚拟网络链接/连接。


## 控制对经典部署模型的访问权限

设置 ExpressRoute 线路的 **allowClassicOperations** 参数，即可让单个 ExpressRoute 线路链接到这两种部署模型中的虚拟网络。

将 **allowClassicOperations** 设置为 TRUE 即可从这两种部署模型中的虚拟网络链接到 ExpressRoute 线路。可以遵循有关[如何链接经典部署模型中的虚拟网络](/documentation/articles/expressroute-howto-linkvnet-classic/)的指导链接到经典部署模型中的虚拟网络。可以遵循有关[如何链接 Resource Manager 部署模型中的虚拟网络](/documentation/articles/expressroute-howto-linkvnet-arm/)的指导链接到 Resource Manager 部署模型中的虚拟网络。

将 **allowClassicOperations** 设置为 FALSE 会阻止从经典部署模型访问线路。但是，经典部署模型中的所有虚拟网络链接将会保留。在此情况下，ExpressRoute 线路不显示在经典部署模型中。

## 经典部署模型中支持的操作

将 **allowClassicOperations** 设置为 TRUE 时，ExpressRoute 线路支持以下经典操作。

 - 获取 ExpressRoute 线路信息
 - 创建/更新/获取/删除到经典虚拟网络的虚拟网络链接
 - 创建/更新/获取/删除跨订阅连接的虚拟网络链接授权

将 **allowClassicOperations** 设置为 TRUE 时，无法执行以下经典操作：

 - 创建/更新/获取/删除 Azure 专用对等互连、Azure 公共对等互连和 Microsoft 对等互连的边界网关协议 (BGP) 对等互连
 - 删除 ExpressRoute 线路

## 经典部署模型和 Resource Manager 部署模型之间的通信

ExpressRoute 线路相当于经典部署模型与 Resource Manager 部署模型之间的桥梁。经典部署模型的虚拟网络中的虚拟机与 Resource Manager 部署模型的虚拟网络中的虚拟机之间的流量将流经 ExpressRoute，前提是这两个虚拟网络链接到相同的 ExpressRoute 线路。

聚合吞吐量受限于虚拟网络网关的吞吐容量。在这种情况下，流量不进入连接服务提供商的网络或你的网络。虚拟网络之间的流量完全包含在 Microsoft 网络中。
## 对 Azure 公共互连资源的访问权限

你可以继续访问通常可通过 Azure 公共互连访问的资源，而不会出现任何中断。

## 支持的操作

本部分介绍可通过 ExpressRoute 线路执行的操作：

 - 可以使用一条 ExpressRoute 线路访问在经典部署模型和 Resource Manager 部署模型中部署的虚拟网络。
 - 可以将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型。转移后，ExpressRoute 线路的感观和执行方式与在 Resource Manager 部署模型中创建的任何其他 ExpressRoute 线路相似。
 - 只能转移 ExpressRoute 线路。无法通过此操作转移线路链接、虚拟网络和 VPN 网关。
 - 将 ExpressRoute 线路转移到 Resource Manager 部署模型后，只能使用 Resource Manager 部署模型来管理 ExpressRoute 线路的生命周期。这意味着，某些操作（例如，添加/更新/删除对等互连，更新带宽、SKU 和计费类型等线路属性，以及删除线路）只能在 Resource Manager 部署模型中执行。
 - ExpressRoute 线路相当于经典部署模型与 Resource Manager 部署模型之间的桥梁。经典部署模型的虚拟网络中的虚拟机与 Resource Manager 部署模型的虚拟网络中的虚拟机之间的流量将流经 ExpressRoute，前提是这两个虚拟网络链接到相同的 ExpressRoute 线路。
 - 经典部署模型和 Resource Manager 部署模型都支持跨订阅连接。

## 不支持的功能

本部分介绍不可通过 ExpressRoute 线路执行的操作：

 - 将线路链接、网关和虚拟网络从经典部署模型转移到 Resource Manager 部署模型。
 - 从经典部署模型管理 ExpressRoute 线路的生命周期。
 - 针对经典部署模型的基于角色的访问控制 (RBAC) 支持。无法对经典部署模型中的线路执行 RBAC 控制。订阅的任何管理员/共同管理员都可以将虚拟网络链接到线路，也都可以取消此类链接。

## 配置

遵循[将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型](/documentation/articles/expressroute-howto-move-arm/)中所述的说明。

## 后续步骤

- 有关工作流信息，请参阅 [ExpressRoute 线路预配工作流和线路状态](/documentation/articles/expressroute-workflows/)。
- 配置 ExpressRoute 连接的步骤：

	- [创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/)
	- [配置路由](/documentation/articles/expressroute-howto-routing-arm/)
	- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)

<!---HONumber=Mooncake_0509_2016-->