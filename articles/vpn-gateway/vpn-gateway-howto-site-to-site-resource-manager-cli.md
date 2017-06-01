<properties
    pageTitle="将本地网络连接到 Azure 虚拟网络：站点到站点 VPN：CLI | Azure"
    description="通过公共 Internet 创建从本地网络到 Azure 虚拟网络的 IPsec 连接的步骤。 这些步骤有助于使用 CLI 创建跨界站点到站点 VPN 网关连接。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/24/2017"
    wacn.date="05/31/2017"
    ms.author="cherylmc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="ae4fbaf2f708f202bc68be821e77a172477a4162"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="create-a-virtual-network-with-a-site-to-site-vpn-connection-using-cli"></a>使用 CLI 创建具有站点到站点 VPN 连接的虚拟网络

本文介绍如何使用 Azure CLI 创建站点到站点 VPN 网关连接，以便从本地网络连接到 VNet。 本文中的步骤适用于 Resource Manager 部署模型。 也可使用不同的部署工具或部署模型创建此配置，方法是从以下列表中选择另一选项：<br>
> [AZURE.SELECTOR]
- [Resource Manager - Azure 门户预览](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [Resource Manager - PowerShell](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)
- [Resource Manager - CLI](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-cli/)
- [经典 - Azure 门户预览](/documentation/articles/vpn-gateway-howto-site-to-site-classic-portal/)
- [经典 - 经典管理门户](/documentation/articles/vpn-gateway-site-to-site-create/)

![站点到站点 VPN 网关跨界连接示意图](./media/vpn-gateway-howto-site-to-site-resource-manager-cli/site-to-site-connection-diagram.png)

使用站点到站点 VPN 网关连接，通过 IPsec/IKE（IKEv1 或 IKEv2）VPN 隧道将本地网络连接到 Azure 虚拟网络。 此类型的连接要求位于本地的 VPN 设备分配有一个面向外部的公共 IP 地址。 有关 VPN 网关的详细信息，请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。

## <a name="before-you-begin"></a>开始之前

在开始配置之前，请验证是否符合以下条件：

* 验证是否需要使用 Resource Manager 部署模型。 [AZURE.INCLUDE [deployment models](../../includes/vpn-gateway-deployment-models-include.md)] 
* 一台兼容的 VPN 设备和能够对其进行配置的人员。 有关兼容的 VPN 设备和设备配置的详细信息，请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。
* 一个用于 VPN 设备的面向外部的公共 IPv4 地址。 此 IP 地址不得位于 NAT 之后。
* 如果熟悉本地网络配置中的 IP 地址范围，则需咨询能够提供此类详细信息的人员。 创建此配置时，必须指定 IP 地址范围前缀，Azure 会将该前缀路由到本地位置。 本地网络的任何子网都不得与要连接到的虚拟网络子网重叠。 
* 最新版本的 CLI 命令（2.0 或更高版本）。 有关安装 CLI 命令的信息，请参阅[安装 Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli) 和 [Azure CLI 2.0 入门](https://docs.microsoft.com/zh-cn/cli/azure/get-started-with-azure-cli)。

### <a name="example-values"></a>示例值

可使用以下值创建测试环境，或参考这些值以更好地理解本文中的示例：

    #Example values

    VnetName                = TestVNet1 
    ResourceGroup           = TestRG1 
    Location                = chinaeast 
    AddressSpace            = 10.12.0.0/16 
    SubnetName              = Subnet1 
    Subnet                  = 10.12.0.0/24 
    GatewaySubnet           = 10.12.255.0/27 
    LocalNetworkGatewayName = Site2 
    LNG Public IP           = <VPN device IP address>
    LocalAddrPrefix1        = 10.0.0.0/24 
    LocalAddrPrefix2        = 20.0.0.0/24   
    GatewayName             = VNet1GW 
    PublicIP                = VNet1GWIP 
    VPNType                 = RouteBased 
    GatewayType             = Vpn 
    ConnectionName          = VNet1toSite2

## <a name="Login"></a>1.连接到订阅

[AZURE.INCLUDE [CLI login](../../includes/vpn-gateway-cli-login-include.md)]

## <a name="2-create-a-resource-group"></a>2.创建资源组

以下示例在“chinaeast”位置创建名为“TestRG1”的资源组。 如果在需创建 VNet 的区域中已经有了一个资源组，则可改用该资源组。

    az group create --name TestRG1 --location chinaeast

## <a name="VNet"></a>3.创建虚拟网络

如果还没有虚拟网络，请使用 [az network vnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet#create) 命令创建一个。 创建虚拟网络时，请确保指定的地址空间不与本地网络的任一个地址空间重叠。 

以下示例创建一个名为“TestVNet1”的虚拟网络和一个名为“Subnet1”的子网。

    az network vnet create --name TestVNet1 --resource-group TestRG1 --address-prefix 10.12.0.0/16 --location chinaeast --subnet-name Subnet1 --subnet-prefix 10.12.0.0/24

## 4.<a name="gwsub"></a>创建网关子网

[AZURE.INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

对于此配置，还需要一个网关子网。 虚拟网关使用的网关子网包含 VPN 网关服务使用的 IP 地址。 创建网关子网时，必须将其命名为“GatewaySubnet”。 如果将其命名为其他名称，将创建子网，但 Azure 不将它视为网关子网。

指定的网关子网的大小取决于要创建的 VPN 网关配置。 尽管创建的网关子网最小可为 /29，但建议选择 /27 或 /28，创建包含更多地址的更大子网。 使用更大的网关子网可以有足够的 IP 地址来应对未来可能会有的配置。

使用 [az network vnet subnet create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet/subnet#create) 命令创建网关子网。

    az network vnet subnet create --address-prefix 10.12.255.0/27 --name GatewaySubnet --resource-group TestRG1 --vnet-name TestVNet1

## <a name="localnet"></a>5.创建本地网关

本地网络网关通常是指本地位置。 可以为站点提供一个名称供 Azure 引用，然后指定本地 VPN 设备的 IP 地址，以便创建一个连接来连接到该设备。 此外还可指定 IP 地址前缀，以便通过 VPN 网关将其路由到 VPN 设备。 指定的地址前缀是位于本地网络的前缀。 如果本地网络出现变化，可以轻松更新这些前缀。

使用以下值：

* *--gateway-ip-address* 是本地 VPN 设备的 IP 地址。 VPN 设备不能位于 NAT 之后。
* *--local-address-prefixes* 是本地地址空间。

使用 [az network local-gateway create](https://docs.microsoft.com/zh-cn/cli/azure/network/local-gateway#create) 命令添加具有多个地址前缀的本地网关：

    az network local-gateway create --gateway-ip-address 23.99.221.164 --name Site2 --resource-group TestRG1 --local-address-prefixes 10.0.0.0/24 20.0.0.0/24

## <a name="PublicIP"></a>6.请求公共 IP 地址

VPN 网关必须具有公共 IP 地址。 请先请求 IP 地址资源，然后在创建虚拟网关时参阅该资源。 创建 VPN 网关时，IP 地址是动态分配给资源的。 VPN 网关当前仅支持动态公共 IP 地址分配。 不能请求静态公共 IP 地址分配。 但这并不意味着 IP 地址在分配到 VPN 网关后会更改。 公共 IP 地址只在删除或重新创建网关时更改。 该地址不会因为 VPN 网关大小调整、重置或其他内部维护/升级而更改。

使用 [az network public-ip create](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#create) 命令请求动态公共 IP 地址。

    az network public-ip create --name VNet1GWIP --resource-group TestRG1 --allocation-method Dynamic

## <a name="CreateGateway"></a>7.创建 VPN 网关

创建虚拟网络 VPN 网关。 创建 VPN 网关可能需要长达 45 分钟或更长时间才能完成。

使用以下值：

* 站点到站点配置的 *--gateway-type* 为 *Vpn*。 网关类型永远是你要实现的配置的特定类型。 有关详细信息，请参阅[网关类型](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwtype)。
* *--vpn-type* 可以是 *RouteBased*（在某些文档中称为动态网关）或 *PolicyBased*（在某些文档中称为静态网关）。 具体设置取决于要连接到的设备的要求。 有关 VPN 网关类型的详细信息，请参阅[关于 VPN 网关配置设置](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#vpntype)。
* *--sku* 可以是 Basic、Standard 或 HighPerformance。 某些 SKU 存在配置限制。 有关详细信息，请参阅[网关 SKU](/documentation/articles/vpn-gateway-about-vpngateways/#gateway-skus)。

使用 [az network vnet-gateway create](https://docs.microsoft.com/zh-cn/cli/azure/network/vnet-gateway#create) 命令创建 VPN 网关。 如果使用“--no-wait”参数运行该命令，则不会显示任何反馈或输出。 此参数允许在后台创建网关。 创建网关大约需要 45 分钟时间。

    az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWIP --resource-group TestRG1 --vnet TestVNet1 --gateway-type Vpn --vpn-type RouteBased --sku Standard --no-wait 

## <a name="VPNDevice"></a>8.配置 VPN 设备

通过站点到站点连接连接到本地网络需要 VPN 设备。 在此步骤中，请配置 VPN 设备。 配置 VPN 设备时，需要以下项：

- 共享密钥。 此共享密钥就是在创建站点到站点 VPN 连接时指定的共享密钥。 在示例中，我们使用基本的共享密钥。 建议生成更复杂的可用密钥。
- 虚拟网关的“公共 IP 地址”。 可以通过 Azure 门户预览、PowerShell 或 CLI 查看公共 IP 地址。 若要查找虚拟网关的公共 IP 地址，请使用 [az network public-ip list](https://docs.microsoft.com/zh-cn/cli/azure/network/public-ip#list) 命令。 为了方便阅读，对输出进行了格式化，以表格式显示一系列公共 IP。

        az network public-ip list --resource-group TestRG1 --output table

[AZURE.INCLUDE [Configure VPN device](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

## <a name="CreateConnection"></a>9.创建 VPN 连接

在虚拟网关和本地 VPN 设备之间创建站点到站点 VPN 连接。 请特别注意共享密钥值，该值必须与为 VPN 设备配置的共享密钥值相符。

使用 [az network vpn-connection create](https://docs.microsoft.com/zh-cn/cli/azure/network/vpn-connection#create) 命令创建连接。

    az network vpn-connection create --name VNet1toSite2 -resource-group TestRG1 --vnet-gateway1 VNet1GW -l chinaeast --shared-key abc123 --local-gateway2 Site2

在一小段时间后，将建立该连接。

## <a name="toverify"></a>10.验证 VPN 连接

[AZURE.INCLUDE [verify connection](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)] 

若要使用其他方法来验证连接，请参阅[验证 VPN 网关连接](/documentation/articles/vpn-gateway-verify-connection-resource-manager/)。

## <a name="common-tasks"></a>常见任务

本部分包含各种常用命令，这些命令在进行站点到站点配置时很有用。 有关 CLI 网络命令的完整列表，请参阅 [Azure CLI - 网络](https://docs.microsoft.com/zh-cn/cli/azure/network)。

[AZURE.INCLUDE [local network gateway common tasks](../../includes/vpn-gateway-common-tasks-cli-include.md)] 

## <a name="next-steps"></a>后续步骤

*  连接完成后，即可将虚拟机添加到虚拟网络。 有关详细信息，请参阅[虚拟机](/documentation/services/virtual-machines/)。
* 有关 BGP 的信息，请参阅 [BGP 概述](/documentation/articles/vpn-gateway-bgp-overview/)和[如何配置 BGP](/documentation/articles/vpn-gateway-bgp-resource-manager-ps/)。
* 有关强制隧道的信息，请参阅[配置强制隧道](/documentation/articles/vpn-gateway-forced-tunneling-rm/)。
* 有关网络 Azure CLI 命令的列表，请参阅 [Azure CLI](https://docs.microsoft.com/zh-cn/cli/azure/network)。