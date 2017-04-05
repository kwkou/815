<properties
    pageTitle="Azure 公共 IP 地址 | Azure"
    description="了解如何创建、修改和删除公共 IP 地址。"
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
    ms.date="03/14/2017"
    wacn.date="03/31/2017"
    ms.author="jdial" />  


# 公共 IP 地址

了解公共 IP 地址以及如何创建、更改和删除它们。公共 IP 地址是一种自带可配置设置的资源。通过将公共 IP 地址分配给其他 Azure 资源，可以：
- 通过 Internet 入站连接到 Azure 虚拟机 (VM)、Azure 虚拟机规模集、Azure VPN 网关以及面向 Internet 的 Azure 负载均衡器等资源。
- 出站连接到非网络地址转换 (NAT) 的 Internet。例如，VM 可在未分配有公共 IP 地址的情况下出站连接到 Internet，但其地址是由 Azure 转换的网络地址。若要详细了解从 Azure 资源的出站连接，请阅读[了解出站连接](/documentation/articles/load-balancer-outbound-connections/)一文。

本文介绍了如何使用公共 IP 地址。本文适用于通过 Azure Resource Manager 部署模型部署的资源。虽然可将公共 IP 地址分配给通过经典部署模型部署的资源，但这些地址的应用方式与通过 Resource Manager 部署的资源不同。如果不熟悉两种模型之间的区别，请阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

本文的剩余部分列出了相应步骤，引导你完成所有公共 IP 地址相关任务。每节分别列出了：
- 在 Azure 门户预览中完成任务的步骤。要完成这些步骤，必须登录 [Azure 门户预览](http://portal.azure.cn)。如果没有帐户，请注册[试用帐户](https://azure.microsoft.com/free)。
- 使用 Azure PowerShell 完成任务的命令，内附命令参考链接。请完成[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 文章中的步骤，安装和配置 PowerShell。若要获取 PowerShell 命令的帮助，请在示例中键入 `get-help <command> -full`。
- 使用 Azure 命令行接口 (CLI) 完成任务的命令，内附命令参考链接。请完成[如何安装和配置 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli) 文章中的步骤安装 Azure CLI。若要获取 CLI 命令的帮助，请键入 `az <command> -h`。

公共 IP 地址会产生少许费用。若要查看定价，请参阅 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。仅可在一个订阅中使用有限数量的公共 IP 地址。若要查看限制，请参阅 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文。

完成以下各节中的步骤，创建、更改或删除公共 IP 地址资源：

## <a name="create"></a>创建公共 IP 地址

若要创建公共 IP 地址，请完成以下步骤：
1. 通过分配有订阅中网络参与者权限（最低标准）的帐户，登录到 [Azure 门户预览](https://portal.azure.cn)。若要详细了解如何向帐户分配角色和权限，请参阅 [Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文。
2. 在 Azure 门户预览顶部带“搜索资源”文字的框中，键入“公共 IP 地址”。在搜索结果中出现**公共 IP 地址**时，单击该地址。
3. 在显示的“公共 IP 地址”边栏选项卡中，单击“+添加”。
4. 在显示的“创建公共 IP 地址”边栏选项卡中，输入或选择以下设置的值，然后单击“创建”：

	|**设置**|必需？|**详细信息**|
	|---|---|---|
	|**名称**|是|名称在所选资源组中必须唯一。|
	|**IP 地址分配**|是|**动态：**仅在将公共 IP 地址与附加到 VM 的 NIC 进行关联，并且 VM 首次启动后，才会分配动态地址。如果 NIC 附加到的 VM 已停止（已释放），则动态地址可能有变。如果 VM 重启或停止（但未释放），则该地址保持不变。**静态：**创建公共 IP 地址时，将分配静态地址。即使 VM 处于停止（已释放）状态，静态地址也不改变。该地址仅在删除 NIC 后才会释放。创建 NIC 后，即可更改分配方法。|
	|**空闲超时(分钟)**|否|在不依赖客户端发送保持连接消息的情况下，TCP 或 HTTP 连接持续打开的分钟数。|
	|**DNS 名称标签**|否|必须在创建名称的 Azure 位置中保持唯一（跨所有订阅和所有客户）。Azure 公共 DNS 服务会自动注册名称和 IP 地址，便于你连接到具有名称的资源。Azure 会将 *location.chinacloudapp.cn*（其中位置由你决定）追加到所提供的名称上，便于创建完全限定的 DNS 名称。 |
	|**订阅**|是|必须与要将公共 IP 地址关联到的资源同属一个订阅。|
	|**资源组**|是|可与要将公共 IP 地址关联到的资源同属一个资源组，也可分属不同资源组。|
	|**位置**|是|必须与要将公共 IP 地址关联到的资源同属一个位置。|

|**工具**|**命令**|
|---|---|
|**CLI**|[az network public-ip-create](https://docs.microsoft.com/cli/azure/network/public-ip#create)|
|**PowerShell**|[New-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/resourcemanager/azurerm.network/v3.4.0/new-azurermpublicipaddress)|

## <a name="change"></a>更改设置或删除公共 IP 地址

若要更改或删除公共 IP 地址，请完成以下步骤：

1. 通过分配有订阅中网络参与者权限（最低标准）的帐户，登录到 [Azure 门户预览](https://portal.azure.cn)。若要详细了解如何向帐户分配角色和权限，请参阅 [Azure 基于角色的访问控制的内置角色](/documentation/articles/role-based-access-built-in-roles/#network-contributor)一文。
2. 在 Azure 门户预览顶部带“搜索资源”文字的框中，键入“公共 IP 地址”。在搜索结果中出现**公共 IP 地址**时，单击该地址。
3. 在显示的“公共 IP 地址”边栏选项卡中，单击要更改其设置或要删除的公共 IP 地址的名称。
4. 在显示的公共 IP 地址边栏选项卡中，根据是否要删除或更改公共 IP 地址完成以下相应选项。
    - **删除：**若要删除公共 IP 地址，请在边栏选项卡的“概述”部分中单击“删除”。如果该地址当前与 IP 配置关联，则无法删除。如果地址当前与配置相关联，请单击“取消关联”，取消该地址与 IP 配置之间的关联。
    - **更改：**单击“配置”。使用本文[创建公共 IP 地址](#create)部分的步骤 4 中的信息更改设置。若要将分配从静态更改为动态，必须先从与其关联的 IP 配置中取消关联公共 IP 地址。然后，可将分配方法更改为动态，然后单击“关联”将 IP 地址关联到相同的 IP 配置或其他配置，还可将其保留为取消关联状态。若要取消关联公共 IP 地址，请在“概述”部分单击“取消关联”。

>[AZURE.WARNING]
将分配方法从静态更改为动态时，将丢失分配给公共 IP 地址的 IP 地址。虽然 Azure 公共 DNS 服务器会保持静态/动态地址与任何 DNS 名称标签（若已定义）之间的映射，但在 VM 处于停止（已释放）状态后，动态 IP 地址可能出现变化。为防止地址变化，请分配静态 IP 地址。

|**工具**|**命令**|
|---|---|
|**CLI**|[az network public-ip update](https://docs.microsoft.com/cli/azure/network/public-ip#update)（用于更新）；[az network public-ip delete](https://docs.microsoft.com/cli/azure/network/public-ip#delete)（用于删除）|
|**PowerShell**|[Set-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/resourcemanager/azurerm.network/v3.4.0/set-azurermpublicipaddress)（用于更新）；[Remove-AzureRmPublicIpAddress](https://docs.microsoft.com/powershell/resourcemanager/azurerm.network/v3.4.0/remove-azurermpublicipaddress)（用于删除）|

## <a name="next-steps"></a>后续步骤
创建以下 Azure 资源时，将分配公共 IP 地址：

- [Windows](/documentation/articles/virtual-machines-windows-hero-tutorial/) 或 [Linux](/documentation/articles/virtual-machines-linux-quick-create-portal/) 虚拟机
- [面向 Internet Azure 负载均衡器](/documentation/articles/load-balancer-get-started-internet-portal/)
- [Azure 应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)
- [使用 Azure VPN 网关的站点到站点连接](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [Azure 虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)

<!---HONumber=Mooncake_0327_2017-->