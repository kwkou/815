<properties
    pageTitle="创建、更改或删除 Azure 虚拟网络子网 | Azure"
    description="了解如何创建、更改或删除虚拟网络子网。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/10/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="fa17b03412492ffde2c5f9c092619451eca7b291"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="create-change-or-delete-virtual-network-subnets"></a>创建、更改或删除虚拟网络子网

了解如何创建、更改或删除虚拟网络 (VNet) 子网。 如果不熟悉 VNet，建议你在创建、更改或删除子网之前，阅读[虚拟网络概述](/documentation/articles/virtual-networks-overview/)和[创建、更改或删除虚拟网络](/documentation/articles/virtual-network-manage-network/)这两篇文章。 可以连接到 VNet 的 Azure 资源连接到 VNet 中的子网。 在 VNet 中创建多个子网通常是出于以下目的：
- 筛选子网之间的流量：可以对子网应用网络安全组 (NSG)，为所有连接到 VNet 的资源（例如虚拟机）筛选入站和出站网络流量。 若要详细了解如何创建 NSG，请阅读[创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-pportal/)一文。
- 控制子网之间的路由：Azure 创建默认路由，用于在子网之间自动路由流量。 可以通过创建用户定义的路由 (UDR) 来重写默认的 Azure 路由。 若要详细了解 UDR，请阅读[创建用户定义的路由](/documentation/articles/virtual-network-create-udr-arm-ps/)一文。 

本文介绍如何为通过 Azure Resource Manager 部署模型创建的 VNet 创建、更改和删除子网。

## <a name="before"></a>准备工作

在完成本文任何部分中的步骤之前，请完成以下任务：

- 如果不熟悉 Azure 中的 VNet 和子网，建议你在阅读本文之前，先完成[创建你的第一个 Azure 虚拟网络](/documentation/articles/virtual-network-get-started-vnet-subnet/)中的练习。 该练习可帮助你熟悉 VNet 和子网。
- 查看 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文，了解子网和 VNet 的限制。
- 使用 Azure 帐户登录到 Azure 门户预览、Azure 命令行界面 (CLI) 或 Azure PowerShell。 如果还没有 Azure 帐户，请注册[试用帐户](/pricing/1rmb-trial)。
- 如果使用 Azure PowerShell 命令来完成本文中的任务，首先必须[安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。 确保已安装最新版本的 Azure PowerShell cmdlet。 若要获取 PowerShell 命令的帮助和示例，请键入 `get-help <command> -full`。
- 如果使用 Azure 命令行界面 (CLI) 命令来完成本文中的任务，首先必须[安装和配置 Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 确保已安装最新版本的 Azure CLI。 若要获取 CLI 命令的帮助，请键入 `az <command> --help`。

## <a name="create-subnet"></a>创建子网

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到[门户](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户预览顶部包含“搜索资源”文本的框中，键入“虚拟网络”。 当“虚拟网络”出现在搜索结果中时，请单击它。
3. 在显示的“虚拟网络”边栏选项卡中，单击要向其添加子网的虚拟网络。
4. 在针对所选虚拟网络显示的窗格中，单击“子网”。
5. 单击“+ 子网”。
6. 在“添加子网”边栏选项卡中，输入以下参数的值：
    - 名称：名称在虚拟网络中必须唯一。
    - 地址范围：此范围在 VNet 的地址空间中必须唯一，不能与 VNet 中的其他子网地址范围重叠。 必须使用 CIDR 表示法指定地址空间。 例如，在地址空间为 10.0.0.0/16 的 VNet 中，可以将子网地址空间定义为 10.0.0.0/24。 可以指定的最小范围为 /29，为子网提供八个 IP 地址。 Azure 保留每个子网中的第一个地址和最后一个地址，以确保协议一致性。 此外还会保留三个地址供 Azure 服务使用。 因此，使用 /29 地址范围定义子网时，子网中会有三个可用 IP 地址。 如果计划将 VNet 连接到 VPN 网关，则必须创建网关子网。 详细了解[网关子网地址范围具体考虑事项](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwsub)。 在特定条件下，可以在子网创建后更改地址范围。 若要了解如何更改子网地址范围，请阅读本文的[更改子网](#change-subnet)部分。
    - 网络安全组 (NSG)：可以选择将现有 NSG 关联到子网，控制子网的入站和出站网络流量筛选。 NSG 必须与 VNet 存在于同一订阅和位置中，且必须通过 Resource Manager 部署模型创建。 若要详细了解如何创建 NSG，请阅读[网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-pportal/)一文。
    - 路由表：可以选择将现有的路由表关联到子网，控制目标为其他网络的网络流量路由。 路由表必须与 VNet 存在于同一订阅和位置中，且必须通过 Resource Manager 部署模型创建。 若要详细了解如何创建路由表，请阅读[用户定义的路由](/documentation/articles/virtual-network-create-udr-arm-ps/)一文。
    - 用户：可以使用内置角色或自己的自定义角色控制对子网的访问。 若要详细了解如何分配访问子网的角色和用户，请阅读[使用角色分配管理对 Azure 资源的访问权限](/documentation/articles/role-based-access-control-configure/#add-access)一文。
7. 单击“确定”按钮，将子网添加到所选 VNet。

命令

|工具|命令|
|---|---|
|CLI|[az network vnet subnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet/subnet#create)|
|PowerShell|[](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig?view=azurermps-3.8.0)New-AzureRmVirtualNetworkSubnetConfig、[](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/add-azurermvirtualnetworksubnetconfig?view=azurermps-3.8.0)Add-AzureRmVirtualNetworkSubnetConfig|

## <a name="change-subnet"></a>更改子网设置

可以在有资源连接到子网的情况下，更改子网的 NSG、路由表和用户访问权限。 若要了解这些设置，请阅读本文[创建子网](#create-subnet)部分的步骤 6。 若要更改子网的地址空间，则不能有资源连接到该子网。 如果有资源连接到子网，则必须先删除连接到子网的资源，然后才能更改地址范围。 资源删除说明因资源而异。 若要了解如何删除连接到子网的资源，请阅读要删除的每个资源类型的文档。 完成以下步骤，更改子网的地址范围：

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到[门户](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在门户顶部包含“搜索资源”文本的框中，键入“虚拟网络”。 当“虚拟网络”出现在搜索结果中时，请单击它。
3. 在显示的“虚拟网络”边栏选项卡中，单击要更改子网地址范围的 VNet。
4. 单击要更改地址范围的子网。
5. 在为所选子网显示的边栏选项卡的“地址范围”框中，输入新的地址范围。 此范围在 VNet 的地址空间中必须唯一，不能与 VNet 中的其他子网地址范围重叠。 必须使用 CIDR 表示法指定地址空间。 例如，在地址空间为 10.0.0.0/16 的 VNet 中，可以将子网地址空间定义为 10.0.0.0/24。 可以指定的最小范围为 /29，为子网提供八个 IP 地址。 Azure 保留每个子网中的第一个地址和最后一个地址，以确保协议一致性。 此外还会保留三个地址供 Azure 服务使用。 因此，地址范围为 /29 的子网有三个可用 IP 地址。 如果计划将 VNet 连接到 VPN 网关，则必须创建网关子网。 详细了解[网关子网地址范围具体考虑事项](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwsub)。 在特定条件下，可以在子网创建后更改地址范围。 若要了解如何更改子网地址范围，请阅读本文的[更改子网](#change-subnet)部分。
6. 单击“保存” 。

命令

|工具|命令|
|---|---|
|CLI|[az network vnet subnet update](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#update)|
|PowerShell|[Set-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig?view=azurermps-3.8.0)|

## <a name="delete-subnet"></a>删除子网

只有在没有资源连接到子网的情况下，才能删除该子网。 如果有资源连接到子网，则必须先删除连接到子网的资源。 资源删除说明因资源而异。 若要了解如何删除连接到子网的资源，请阅读要删除的每个资源类型的文档。

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到[门户](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户预览顶部包含“搜索资源”文本的框中，键入“虚拟网络”。 当“虚拟网络”出现在搜索结果中时，请单击它。
3. 在显示的“虚拟网络”边栏选项卡中，单击要从其删除子网的 VNet。
4. 在针对所选 VNet 显示的边栏选项卡的“设置”下，单击“子网”。
5. 在子网边栏选项卡的子网列表中，右键单击要删除的子网，然后依次单击“删除”、“是”删除该子网。

命令

|工具|命令|
|---|---|
|CLI|[az network vnet delete](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#delete)|
|PowerShell|[Remove-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.network/remove-azurermvirtualnetworksubnetconfig?view=azurermps-3.8.0)|

## <a name="next-steps"></a>后续步骤

若要创建 VM 并将其连接到子网，请阅读[创建 VNet 并连接 VM](/documentation/articles/virtual-network-get-started-vnet-subnet/#create-vms) 一文。