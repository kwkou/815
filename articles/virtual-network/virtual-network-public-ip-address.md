<properties
    pageTitle="创建、更改或删除 Azure 公共 IP 地址 | Azure"
    description="了解如何创建、更改或删除公共 IP 地址。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="bb71abaf-b2d9-4147-b607-38067a10caf6"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/05/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="fd5b08371259808ffa1f5bf5f55669e98282839a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

#  <a name="create-change-or-delete-public-ip-addresses"></a>创建、更改或删除公共 IP 地址

了解公共 IP 地址以及如何创建、更改和删除它们。 公共 IP 地址是一种自带可配置设置的资源。 通过将公共 IP 地址分配给其他 Azure 资源，可以：
- 与 Azure 虚拟机 (VM)、Azure 虚拟机规模集、Azure VPN 网关和面向 Internet 的 Azure 负载均衡器等资源建立入站 Internet 连接。
- 与未经过网络地址转换 (NAT) 的 Internet 建立出站连接。 例如，VM 可在未分配有公共 IP 地址的情况下出站连接到 Internet，但其地址是由 Azure 转换的网络地址。 若要详细了解从 Azure 资源的出站连接，请阅读[了解出站连接](/documentation/articles/load-balancer-outbound-connections/)一文。

本文介绍了如何处理通过 Azure Resource Manager 部署模型部署的公共 IP 地址。

## <a name="before"></a>准备工作

请在完成本文任何部分中的任何步骤之前完成以下任务：

- 查看 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文，了解对公共 IP 地址的限制。
- 使用 Azure 帐户登录到 Azure 门户、Azure 命令行界面 (CLI) 或 Azure PowerShell。 如果还没有 Azure 帐户，请注册一个[试用帐户](/pricing/1rmb-trial)。
- 如果使用 PowerShell 命令完成本文中的任务，请按[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 一文中的步骤安装和配置 Azure PowerShell。 确保已安装最新版本的 Azure PowerShell cmdlet。 若要获取 PowerShell 命令的帮助和示例，请键入 `get-help <command> -full`。
- 如果使用 Azure 命令行接口 (CLI) 命令完成本文中的任务，请按[如何安装和配置 Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli) 一文中的步骤安装和配置 Azure CLI。 确保已安装最新版本的 Azure CLI。 若要获取 CLI 命令的帮助，请键入 `az <command> --help`。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

公共 IP 地址会产生少许费用。 若要查看定价，请参阅 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。 

## <a name="create"></a>创建公共 IP 地址

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到 [Azure 门户](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户顶部带“搜索资源”文本的框中，键入*公共 IP 地址*。 在搜索结果中出现 **公共 IP 地址** 时，单击该地址。
3. 在出现的“公共 IP 地址”边栏选项卡中单击“+ 添加”。
4. 在出现的“创建公共 IP 地址”边栏选项卡中，为以下设置输入或选择值，然后单击“创建”：

    |设置|必需？|详细信息|
    |---|---|---|
    |名称|是|名称在所选资源组中必须唯一。|
    |IP 地址分配|是|**动态：** 仅在将公共 IP 地址与附加到 VM 的 NIC 进行关联，并且 VM 首次启动后，才会分配动态地址。 如果 NIC 附加到的 VM 已停止（解除分配），则动态地址可能会更改。 如果 VM 重启或停止（但未解除分配），该地址将保持不变。 **静态：** 创建公共 IP 地址时，将分配静态地址。 即使 VM 处于停止（解除分配）状态，静态地址也不会发生更改。 该地址仅在删除 NIC 后才会释放。 创建 NIC 后，即可更改分配方法。|
    |空闲超时(分钟)|否|在不依赖客户端发送保持连接消息的情况下，TCP 或 HTTP 连接持续打开的分钟数。|
    |DNS 名称标签|否|必须在创建名称的 Azure 位置中（在所有订阅和所有客户中）保持唯一。 Azure 公共 DNS 服务会自动注册名称和 IP 地址，便于你连接到具有名称的资源。 Azure 会将 *location.chinacloudapp.cn*（其中 location 是你选择的位置）追加到你提供的名称上，来创建完全限定的 DNS 名称。|
    |订阅|是|必须与要将公共 IP 地址关联到的资源位于同一[订阅](/documentation/articles/azure-glossary-cloud-terminology/#subscription)中。|
    |资源组|是|可与要将公共 IP 地址关联到的资源位于相同或不同的[资源组](/documentation/articles/azure-glossary-cloud-terminology/#resource-group)中。|
    |位置|是|必须与要将公共 IP 地址关联到的资源位于同一位置（也称为“区域”）。|

**命令**

|工具|命令|
|---|---|
|CLI|[az network public-ip-create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create)|
|PowerShell|[New-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/new-azurermpublicipaddress)|

## <a name="change"></a>更改公共 IP 地址的设置或删除公共 IP 地址

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到 [Azure 门户](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户顶部带“搜索资源”文本的框中，键入*公共 IP 地址*。 在搜索结果中出现 **公共 IP 地址** 时，单击该地址。
3. 在显示的“公共 IP 地址”  边栏选项卡中，单击要更改其设置或要删除的公共 IP 地址的名称。
4. 在显示的公共 IP 地址边栏选项卡中，根据是否要删除或更改公共 IP 地址完成以下相应选项。
    - **删除：**若要删除公共 IP 地址，请在该边栏选项卡的“概述”部分中单击“删除”。 如果该地址当前与 IP 配置关联，则无法删除。 如果地址当前与配置相关联，请单击“取消关联”  ，取消该地址与 IP 配置之间的关联。
    - **更改：**单击“配置”。 使用本文 [创建公共 IP 地址](#create) 部分的步骤 4 中的信息更改设置。 若要将分配从静态更改为动态，必须先从与其关联的 IP 配置中取消关联公共 IP 地址。 然后，可将分配方法更改为动态，然后单击“关联”  将 IP 地址关联到相同的 IP 配置或其他配置，还可将其保留为取消关联状态。 若要取消关联公共 IP 地址，请在“概述”部分中单击“取消关联”。

>[AZURE.WARNING]
>将分配方法从静态更改为动态时，将丢失分配给公共 IP 地址的 IP 地址。 尽管 Azure 公共 DNS 服务器会保留静态或动态地址与任何 DNS 名称标签（如果已定义）之间的映射，但如果 VM 在处于停止（解除分配）状态之后启动，动态 IP 地址可能会更改。 为防止地址变化，请分配静态 IP 地址。

**命令**

|工具|命令|
|---|---|
|CLI|[az network public-ip update](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#update) 用于更新；[az network public-ip delete](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#delete) 用于删除|
|PowerShell|[Set-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.4.0/set-azurermpublicipaddress) 用于更新；[Remove-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/remove-azurermpublicipaddress) 用于删除|

## <a name="next-steps"></a>后续步骤
创建以下 Azure 资源时，将分配公共 IP 地址：

- [Windows](/documentation/articles/virtual-machines-windows-hero-tutorial/) 或 [Linux](/documentation/articles/virtual-machines-linux-quick-create-portal/) 虚拟机
- [面向 Internet 的 Azure 负载均衡器](/documentation/articles/load-balancer-get-started-internet-portal/)
- [Azure 应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)
- [使用 Azure VPN 网关建立站点到站点连接](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [Azure 虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)

<!--Update_Description: wording update-->