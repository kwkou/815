<properties
   pageTitle="使用 Resource Manager 和 PowerShell 将 VPN 网关添加到 ExpressRoute 的虚拟网络中 | Microsoft Azure"
   description="本文指导你将 VPN 网关添加到为 ExpressRoute 创建的 Resource Manager VNet 中"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>

<tags 
   ms.service="expressroute"
   ms.date="02/26/2016"
   wacn.date="04/05/2016"/>

# 将 VPN 网关添加到 ExpressRoute 的Resource Manager VNet 配置 

本文将逐步指你添加网关子网，以及为现有的 VNet 创建 VPN 网关。此配置的步骤是专用于使用 **Resource Manager 部署模型**创建的、将在 ExpressRoute 配置中使用的 VNet。如需有关使用经典部署模型创建适用于 VNet 的网关的信息，请参阅[使用经典门户配置 ExpressRoute 的虚拟网络](/documentation/articles/expressroute-howto-vnet-portal-classic)。

## 开始之前

确认已安装此配置所需的 Azure PowerShell cmdlet（1.0.2 或更高版本）。如果尚未安装 cmdlet，必须先安装，然后才能开始执行配置步骤。有关安装 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。


[AZURE.INCLUDE [expressroute-gateway-rm-ps](../includes/expressroute-gateway-rm-ps-include.md)]

## 验证是否已创建网关

使用以下命令来验证是否已创建网关。

	Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG
	
## 后续步骤

创建 VPN 网关之后，可以将 VNet 链接到 ExpressRoute 线路。请参阅[将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm)。

<!---HONumber=Mooncake_0328_2016-->