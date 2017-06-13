<properties
    pageTitle="为 Azure 网络接口配置 IP 地址 | Azure"
    description="了解如何为 NIC 添加、更改和删除公共和专用 IP 地址。"
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
    ms.date="05/04/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="08573697c737ebcc324396acb095910ec4301502"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="add-change-or-remove-ip-addresses-for-azure-network-interfaces"></a>为 Azure 网络接口添加、更改或删除 IP 地址

了解如何为网络接口 (NIC) 添加、更改和删除公共和专用 IP 地址。 通过分配给 NIC 的专用 IP 地址，VM 可以与 Internet 以及连接到 Azure 虚拟网络 (VNet) 的其他资源通信。 使用分配给 NIC 的公共 IP 地址，可以从 Internet 进行 VM 的入站通信。 

若需创建、更改或删除 NIC，请阅读 [NIC 设置和任务](/documentation/articles/virtual-network-network-interface/)一文。 如果需要在虚拟机中添加或删除 NIC，请阅读[添加或删除 NIC](/documentation/articles/virtual-network-network-interface-vm/) 一文。 

## <a name="before"></a>准备工作

请在完成本文任何部分中的任何步骤之前完成以下任务：

- 查看 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文，了解对公共和专用 IP 地址的限制。
- 使用 Azure 帐户登录到 Azure 门户预览、Azure 命令行界面 (CLI) 或 Azure PowerShell。 如果还没有 Azure 帐户，请注册一个[试用帐户](/pricing/1rmb-trial)。
- 如果使用 PowerShell 命令完成本文中的任务，请按[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 一文中的步骤安装和配置 Azure PowerShell。 确保已安装最新版本的 Azure PowerShell cmdlet。 若要获取 PowerShell 命令的帮助和示例，请键入 `get-help <command> -full`。
- 如果使用 Azure 命令行接口 (CLI) 命令完成本文中的任务，请按[如何安装和配置 Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli) 一文中的步骤安装和配置 Azure CLI。 确保已安装最新版本的 Azure CLI。 若要获取 CLI 命令的帮助，请键入 `az <command> --help`。

## <a name="about"></a>关于 NIC 和 IP 地址

可以为每个 NIC 分配多个专用和公共 IP 地址，只要符合 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)即可。 在以下场景中，有必要为 NIC 分配多个 IP 地址：
- 在一台服务器上托管具有不同 IP 地址和 SSL 证书的多个网站或服务。
- VM 用作网络虚拟设备，例如防火墙或负载均衡器。
- 能够将任何 NIC 的任意专用 IP 地址添加到 Azure 负载均衡器后端池。 过去，只有主 NIC 的主 IP 地址才能添加到后端池。 若要详细了解如何对多个 IP 配置进行负载均衡，请阅读[对多个 IP 配置进行负载均衡](/documentation/articles/load-balancer-multiple-ip/)一文。

IP 地址分配给 IP 配置。 始终为 NIC 分配一个**主** IP 配置，但还可为其分配多个**辅助**配置。 将为每个 IP 配置分配以下一种或两种地址类型：
- **专用：**VM 可以通过专用 IP 地址与 NIC 所在的 VNet 所连接的其他资源进行通信。 一个 IP 配置必须分配有一个专用 IP 地址。 Azure DHCP 服务器将 NIC 的主 IP 配置的专用 IP 地址分配到 VM 操作系统中的 NIC。 VM 也可通过专用 IP 地址与 Internet 进行出站通信。 出站连接是转换的源网络地址 (SNAT)。 若要了解 Azure 出站 Internet 连接的详细信息，请阅读 [Azure 出站 Internet 连接](/documentation/articles/load-balancer-outbound-connections/)一文。 不能从 Internet 与 VM 的专用 IP 地址进行入站通信。
- **公共：**可以从 Internet 通过公共 IP 地址与 VM 建立入站连接。 到 Internet 的出站连接未进行 SNAT 处理。 可以为 IP 配置分配公共 IP 地址，但这不是必需的。 若要详细了解公共 IP 地址，请阅读[公共 IP 地址](/documentation/articles/virtual-network-public-ip-address/)一文。

可分配到 NIC 的公共和专用 IP 地址数有限制。 有关详细信息，请参阅 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文。

可以使用以下分配方法来分配公共和专用 IP 地址：
- **动态：**默认情况下会分配动态专用和公共 IP 地址。 如果将 VM 置于“已停止”（“已解除分配”）状态，然后将其重启，动态地址可能会更改。 如果不希望 IP 地址在 VM 的生命周期内更改，请使用静态地址。
- **静态：**静态地址不会更改，除非将 VM 删除。 可以从 NIC 所附加到的地址空间为子网分配静态专用 IP 地址。 若要详细了解 Azure 如何分配静态公共 IP 地址，请阅读[公共 IP 地址](/documentation/articles/virtual-network-public-ip-address/)一文。

## <a name="create-ip-config"></a>添加 IP 地址

可将任意数量的 IP 地址添加到 NIC，只要不超过 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文中所列的限制即可。

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到 [Azure 门户预览](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户预览顶部包含“搜索资源”文本的框中，键入“网络接口”。 在搜索结果中出现“网络接口”  时，单击该接口。
3. 在显示的“网络接口”边栏选项卡中，单击要为其添加 IP 地址的 NIC。
4. 在所选 NIC 的边栏选项卡的“设置”部分，单击“IP 配置”。
5. 在打开的 IP 配置边栏选项卡中单击“+ 添加”。
6. 指定以下设置，然后单击“确定”关闭“添加 IP 配置”边栏选项卡：

    |设置|必需？|详细信息|
    |---|---|---|
    |名称|是|对此 NIC 而言必须唯一|
    |类型|是|由于要向现有 NIC 添加 IP 配置，且每个 NIC 均必须具有主 IP 配置，因此仅可选择“辅助” 。|
    |专用 IP 地址分配方法|是|如果在 VM 处于停止（解除分配）的状态下将它重新启动，**动态**地址可能会更改。 Azure 从 NIC 连接到的子网的地址空间分配可用地址。 只有在删除 NIC 之后，**静态**地址才会释放。 从子网地址空间范围中，指定当前未由其他 IP 配置使用的 IP 地址。|
    |公共 IP 地址|否|**禁用：** IP 配置目前未关联公共 IP 地址资源。 **启用：** 选择现有的公共 IP 地址或新建一个。 若要了解如何创建公共 IP 地址，请参阅[公共 IP 地址](/documentation/articles/virtual-network-public-ip-address/#create)一文。|
7. 可以遵循[将多个 IP 地址分配到虚拟机](/documentation/articles/virtual-network-multiple-ip-addresses-portal/#os-config)一文中的说明，手动将辅助专用 IP 地址添加到 VM 操作系统。 请勿向 VM 操作系统添加任何公共 IP 地址。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic ip-config create](https://docs.microsoft.com/zh-cn/cli/azure/network/nic/ip-config#create)|
|PowerShell|[Add-AzureRmNetworkInterfaceIpConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.4.0/add-azurermnetworkinterfaceipconfig)|

## <a name="change-ip-config"></a>更改 IP 地址设置

可能需要更改 IP 地址的分配方法、更改静态 IP 地址，或者更改分配到 NIC 的公共 IP 地址。 如果要更改与 VM 中的辅助 NIC关联的辅助 IP 配置的专用 IP 地址（有关详细信息，请参阅[主 NIC 和辅助 NIC](/documentation/articles/virtual-network-network-interface-vm/#about)），请将该 VM 置于“已停止”（“已解除分配”）状态，然后再完成以下步骤： 

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到 [Azure 门户预览](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户预览顶部包含“搜索资源”文本的框中，键入“网络接口”。 在搜索结果中出现“网络接口”  时，单击该接口。
3. 在显示的“网络接口”边栏选项卡中，单击想要查看或更改其 IP 地址设置的 NIC。
4. 在所选 NIC 的边栏选项卡的“设置”部分，单击“IP 配置”。
5. 在打开的 IP 配置边栏选项卡上显示的列表中，单击要修改的 IP 配置。
6. 参考本文[添加 IP 配置](#create-ip-config)部分步骤 6 中有关设置的信息，根据需要更改设置。 单击“保存”关闭所更改 IP 配置的边栏选项卡。

>[AZURE.NOTE]
>如果主 NIC 有多个 IP 配置，而你更改了主 IP 配置的专用 IP 地址，则必须手动将所有辅助 IP 地址重新分配到 Windows 中的 NIC（在 Linux 中不需要执行此操作）。 若要手动将 IP 地址分配到操作系统中的 NIC，请参阅[将多个 IP 地址分配到虚拟机](/documentation/articles/virtual-network-multiple-ip-addresses-portal/#os-config)一文。 请勿向 VM 操作系统添加任何公共 IP 地址。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic ip-config update](https://docs.microsoft.com/zh-cn/cli/azure/network/nic/ip-config#update)|
|PowerShell|[Set-AzureRMNetworkInterfaceIpConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.4.0/set-azurermnetworkinterfaceipconfig)|

## <a name="delete-ip-config"></a>删除 IP 地址

可以从 NIC 中删除专用和公共 IP 地址，但 NIC 必须始终有一个分配的专用 IP 地址。

1. 使用已分配订阅的“网络参与者”角色权限（最低权限）的帐户登录到 [Azure 门户预览](https://portal.azure.cn)。 请参阅[用于 Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文，详细了解如何将角色和权限分配给帐户。
2. 在 Azure 门户预览顶部包含“搜索资源”文本的框中，键入“网络接口”。 在搜索结果中出现“网络接口”  时，单击该接口。
3. 在显示的“网络接口”边栏选项卡中，单击要从其删除 IP 地址的 NIC。
4. 在所选 NIC 的边栏选项卡的“设置”部分，单击“IP 配置”。
5. 右键单击要删除的辅助 IP 配置（不能删除主配置），单击“删除”，然后单击“是”确认删除。 如果该配置关联了公共 IP 地址资源，则该资源将从 IP 配置中分离，但不会被删除。
6. 关闭“IP 配置”  边栏选项卡。

**命令**

|工具|命令|
|---|---|
|CLI|[az network nic ip-config delete](https://docs.microsoft.com/zh-cn/cli/azure/network/nic/ip-config#delete)|
|PowerShell|[Remove-AzureRmNetworkInterfaceIpConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.4.0/remove-azurermnetworkinterfaceipconfig)|

## <a name="next-steps"></a>后续步骤
若要创建具有多个 NIC 或 IP 地址的 VM，请阅读以下文章：

**命令**

|任务|工具|
|---|---|
|创建具有多个 NIC 的 VM|[CLI](/documentation/articles/virtual-machines-linux-multiple-nics/)|
||[PowerShell](/documentation/articles/virtual-machines-windows-multiple-nics/)|
|创建具有多个 IP 地址的单 NIC VM|[CLI](/documentation/articles/virtual-network-multiple-ip-addresses-cli/)|
||[PowerShell](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)|