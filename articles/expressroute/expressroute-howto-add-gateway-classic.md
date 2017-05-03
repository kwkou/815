<properties
    pageTitle="使用 PowerShell 为 ExpressRoute 配置 VNet 网关（经典）：Azure "
    description="在 ExpressRoute 配置中使用 PowerShell 配置经典部署模型 VNet 的 VNet 网关。"
    documentationcenter="na"
    services="expressroute"
    author="charwen"
    manager="carmonm"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="85ee0bc1-55be-4760-bfb4-34d9f2c96f30"
    ms.service="expressroute"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/21/2017"
    wacn.date="05/02/2017"
    ms.author="charwen"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="becb0ba865157970cc0c19c757d8e2f9d6a34794"
    ms.lasthandoff="04/22/2017" />

# <a name="configure-a-virtual-network-gateway-for-expressroute-using-powershell-classic"></a>使用 PowerShell 配置 ExpressRoute 的虚拟网络网关（经典）
> [AZURE.SELECTOR]
- [Resource Manager - PowerShell](/documentation/articles/expressroute-howto-add-gateway-resource-manager/)
- [经典 - PowerShell](/documentation/articles/expressroute-howto-add-gateway-classic/)

本文将指导你完成为预先存在的 VNet 添加、重设大小和删除虚拟网络 (VNet) 网关的步骤。此配置的步骤专用于使用**经典部署模型**创建的、将在 ExpressRoute 配置中使用的 VNet。

[AZURE.INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="before-beginning"></a>开始之前

确认已安装此配置所需的 Azure PowerShell cmdlet（1.0.2 或更高版本）。如果尚未安装 cmdlet，必须先安装，然后才能开始执行配置步骤。有关安装 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

[AZURE.INCLUDE [expressroute-gateway-classic-ps](../../includes/expressroute-gateway-classic-ps-include.md)]

## <a name="next-steps"></a>后续步骤

创建 VNet 网关之后，可以将 VNet 链接到 ExpressRoute 线路。请参阅[将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)。

