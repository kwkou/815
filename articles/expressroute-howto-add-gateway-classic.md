<properties
   pageTitle="使用 PowerShell 为 ExpressRoute 配置 VNet 网关 | Azure"
   description="在 ExpressRoute 配置中使用 PowerShell 配置经典部署模型 VNet 的 VNet 网关。"
   documentationCenter="na"
   services="expressroute"
   authors="charwen"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.date="04/06/2016"
   wacn.date="05/16/2016"/>

# 使用经典部署模型和 PowerShell 为 ExpressRoute 配置虚拟网络网关

> [AZURE.SELECTOR]
- [PowerShell - Resource Manager](/documentation/articles/expressroute-howto-add-gateway-resource-manager/)
- [PowerShell - Classic](/documentation/articles/expressroute-howto-add-gateway-classic/)

本文将指导你完成为预先存在的 VNet 添加、重设大小和删除虚拟网络 (VNet) 网关的步骤。此配置的步骤专用于使用**经典部署模型**创建的、将在 ExpressRoute 配置中使用的 VNet。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

## 开始之前

确认已安装此配置所需的 Azure PowerShell cmdlet（1.0.2 或更高版本）。如果尚未安装 cmdlet，必须先安装，然后才能开始执行配置步骤。有关安装 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


[AZURE.INCLUDE [expressroute-gateway-classic-ps](../includes/expressroute-gateway-classic-ps-include.md)]

	
## 后续步骤

创建 VNet 网关之后，可以将 VNet 链接到 ExpressRoute 线路。请参阅[将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)。

<!---HONumber=Mooncake_0509_2016-->