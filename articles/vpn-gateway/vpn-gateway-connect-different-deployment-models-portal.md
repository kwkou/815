<properties 
   pageTitle="如何在门户中将经典虚拟网络连接到 Resource Manager 虚拟网络 | Azure"
   description="了解如何使用 VPN 网关和门户在经典 VNet 和 Resource Manager VNet 之间创建 VPN 连接"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-service-management,azure-resource-manager"/>  

<tags 
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/03/2016"
   wacn.date=""
   ms.author="cherylmc" />

# 从门户中的不同部署模型中连接虚拟网络

> [AZURE.SELECTOR]
- [门户](/documentation/articles/vpn-gateway-connect-different-deployment-models-portal/)
- [PowerShell](/documentation/articles/vpn-gateway-connect-different-deployment-models-powershell/)


Azure 当前具有两个管理模型：经典模型和 Resource Manager (RM) 模型。如果 Azure 已经使用了一段时间，则您的 Azure VM 和实例角色可能是在经典 VNet 上运行。而较新的 VM 和角色实例可能是在 Resource Manager 中创建的 VNet 上运行。本文将指导您如何连接经典 VNet 和 Resource Manager Vnet，从而可以通过网关连接使不同部署模型中的资源能够相互通信。

可以在位于不同订阅、不同区域中的 VNet 之间创建连接。您还可以连接已连接到本地网络的 VNet，只要它们配置的网关是动态或基于路由的。有关 VNet 到 VNet 连接的详细信息，请参阅本文末尾的 [VNet 到 VNet 常见问题解答](#faq)。

### VNet 到 VNet 连接的部署模型和方法

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

当我们发布有关此配置的新文章和其他可用工具时，将会更新以下表格。有相关的文章发布时，我们会直接从该表格链接到该文章。<br><br>

**VNet 到 VNet**

[AZURE.INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

#### VNet 对等互连

[AZURE.INCLUDE [vpn-gateway-vnetpeeringlink](../../includes/vpn-gateway-vnetpeeringlink-include.md)]

## 开始之前

以下步骤将指导您完成为每个 VNet 配置动态或基于路由的网关以及在网关之间创建 VPN 连接所需的设置。此配置不支持静态或基于策略的网关。

在本文中，我们将使用经典管理门户、Azure 门户预览和 PowerShell。目前，无法只使用 Azure 门户预览创建此配置。

### 先决条件

 - 已创建了两个 VNet。
 - 两个 VNet 的地址范围不相互重叠，也不与网关可能连接到的其他连接的任何范围重叠。
 - 已安装最新的 PowerShell cmdlet（1.0.2 或更高版本）。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。请确保安装服务管理 (SM) 和 Resource Manager (RM) cmdlet。

### <a name="values"></a>示例设置

可以使用示例设置作为参考。

**经典 VNet 设置**

VNet 名称 = ClassicVNet <br>
位置 = 中国北部 <br>
虚拟网络地址空间 = 10.0.0.0/24 <br>
子网-1 = 10.0.0.0/27 <br>
GatewaySubnet = 10.0.0.32/29 <br>
本地网络名称 = RMVNetLocal <br>

**Resource Manager VNet 设置**

VNet 名称 = RMVNet <br>
资源组 = RG1 <br>
虚拟网络 IP 地址空间 = 192.168.0.0/16 <br>
子网-1 = 192.168.1.0/24 <br>
GatewaySubnet = 192.168.0.0/26 <br>
位置 = 中国东部 <br>
虚拟网络网关名称 = RMGateway <br>
网关公共 IP 名称 = gwpip <br>
网关类型 = VPN <br>
VPN 类型 = 基于路由 <br>
本地网络网关 = ClassicVNetLocal <br>

## <a name="createsmgw"></a>第 1 部分：配置经典 VNet 设置

在本部分中，我们将为经典 VNet 创建本地网络和网关。本部分中的说明使用经典管理门户。目前，Azure 门户预览不提供与经典 VNet 相关的所有设置。

### 第 1 节 - 创建新的本地网络

打开[经典管理门户](https://manage.windowsazure.cn)，并使用您的 Azure 帐户登录。

1. 在屏幕的左下角，单击“新建”>“网络服务”>“虚拟网络”>“添加本地网络”。

2. 在“指定本地网络详细信息”窗口中，键入想要连接到的 RM VNet 的名称。在“VPN 设备 IP 地址 (可选)”框中，键入任何有效的公共 IP 地址。这只是一个临时占位符。您可以稍后更改此 IP 地址。单击窗口右下角的箭头按钮。
 
3. 在“指定地址空间”页的“起始 IP”文本框中，键入想要连接到的 Resource Manager VNet 的网络前缀和 CIDR 块。此设置用于指定要路由到 RM VNet 的地址空间。

### 第 2 节 - 将本地网络关联到 VNet

1. 单击页面顶部的“虚拟网络”以切换到“虚拟网络”屏幕，然后单击以选择经典 VNet。在 VNet 的页面上，单击“配置”以导航到配置页。

2. 在“站点到站点连接”连接部分下面，选中“连接到本地网络”复选框。然后选择您创建的本地网络。如果您创建了多个本地网络，请务必从下拉列表中选择创建的用于代表 Resource Manager VNet 的本地网络。

3. 单击页面底部的“保存”。

### 第 3 节 - 创建网关

1. 保存设置后，单击页面顶部的“仪表板”以切换到“仪表板”页面。在“仪表板”页的底部，单击“创建网关”，然后单击“动态路由”。单击“是”即可开始创建网关。该配置需要动态路由网关。

2. 等待网关创建完毕。有时可能需要 45 分钟或更长时间才能完成。

### <a name="ip"></a>第 4 节 - 查看网关的公共 IP 地址

创建网关后，可以在“仪表板”页面中查看网关 IP 地址。这是网关的公共 IP 地址。记下或复制此公共 IP 地址。将在后面为 Resource Manager VNet 配置创建本地网络的步骤中使用此地址。


## <a name="creatermgw"></a>第 2 部分：配置 Resource Manager VNet 设置

在本部分中，我们将为 Resource Manager VNet 创建虚拟网络网关和本地网络。请务必在检索到经典 VNet 的网关的公共 IP 地址之后再开始执行以下步骤。

这些屏幕截图作为示例提供。请务必替换为你自己的值。如果您在练习创建此配置，请引用这些[值](#values)。


### 第 1 节 - 创建网关子网

将虚拟网络连接到网关之前，必须先创建要连接的虚拟网络的网关子网。创建 CIDR 计数为 /28 或更大（/27、/26 等）的网关子网

[AZURE.INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

从浏览器导航到 [Azure 门户预览](http://portal.azure.cn)并使用 Azure 帐户登录。

[AZURE.INCLUDE [vpn-gateway-add-gwsubnet-rm-portal](../../includes/vpn-gateway-add-gwsubnet-rm-portal-include.md)]

### 第 2 节 - 创建虚拟网络网关


[AZURE.INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]


### 第 3 节 - 创建本地网络网关

“本地网络网关”通常是指您的本地位置。它向 Azure 指明了路由到该位置的 IP 地址范围和该位置的设备的公共 IP 地址。但是，在此处它指的是与经典 VNet 和虚拟网络网关相关联的地址范围和公共 IP 地址。

为本地网络网关命名，Azure 可通过该名称引用该网关。可以在创建虚拟网络网关时创建本地网络网关。对于此配置，请使用在[上一部分](#ip)中已分配给经典 VNet 网关的公共 IP 地址。

[AZURE.INCLUDE [vpn-gateway-add-lng-rm-portal](../../includes/vpn-gateway-add-lng-rm-portal-include.md)]


### 第 4 节 - 复制公共 IP 地址

创建虚拟网络网关后，复制与该网关相关联的公共 IP 地址。为经典 VNet 配置本地网络设置时要使用该地址。


## 第 3 节：修改经典 VNet 的本地网络

打开[经典管理门户](https://manage.windowsazure.cn)。

2. 在经典管理门户的左侧向下滚动，然后单击“网络”。在“网络”页上，单击页面顶部的“本地网络”。

3. 单击以选择在第 1 节配置的本地网络。在页面底部单击“编辑”。

4. 在“指定本地网络详细信息”页上，将占位符 IP 地址替换为上一节中创建的 Resource Manager 网关的公共 IP 地址。单击箭头转到下一部分。确认“地址空间”是正确的，然后单击复选标记以接受更改。

## <a name="connect"></a>第 4 部分：创建连接

在本部分中，我们将创建 VNet 之间的连接。本部分的步骤需要使用 PowerShell。无法在任一门户中创建此连接。请确保您已下载并安装了经典 (SM) 和 Resource Manager (RM) PowerShell cmdlet。


1. 在 PowerShell 控制台中，登录到您的 Azure 帐户。以下 cmdlet 将提示您提供 Azure 帐户的登录凭据。登录后将下载您的帐户设置，以便 Azure PowerShell 使用这些设置。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

 	如果您有多个订阅，则获取您的 Azure 订阅的列表。

		Get-AzureRmSubscription

	指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Name of subscription"


2. 添加要使用经典 PowerShell cmdlet 的 Azure 帐户。若要执行此操作，请使用以下命令：

		Add-AzureAccount -Environment AzureChinaCloud

3. 通过运行下面的示例设置共享密钥。在此示例中，`-VNetName` 是经典 VNet 的名称，`-LocalNetworkSiteName` 是在经典管理门户中配置本地网络时为其指定的名称。`-SharedKey` 是您可以生成并指定的值。此处指定的值必须与下一步中创建连接时指定的值相同。

		Set-AzureVNetGatewayKey -VNetName ClassicVNet `
		-LocalNetworkSiteName RMVNetLocal -SharedKey abc123

4. 运行以下命令创建 VPN 连接：
	
	**设置变量**

		$vnet01gateway = Get-AzureRMLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
		$vnet02gateway = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1

	**创建连接**<br>请注意，`-ConnectionType` 为 IPsec，而不是 Vnet2Vnet。在此示例中，`-Name` 是想要调用连接的名称。下面的示例创建了名为 *rm-to-classic-connection* 的连接。
		
		New-AzureRmVirtualNetworkGatewayConnection -Name rm-to-classic-connection -ResourceGroupName RG1 `
		-Location "China East" -VirtualNetworkGateway1 `
		$vnet02gateway -LocalNetworkGateway2 `
		$vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

## 验证连接

可以使用经典管理门户、Azure 门户预览或 PowerShell 来验证连接。可使用以下步骤来验证连接。将值替换为你自己的值。

[AZURE.INCLUDE [vpn-gateway-verify connection](../../includes/vpn-gateway-verify-connection-rm-include.md)]

## <a name="faq"></a>VNet 到 VNet 常见问题解答

查看常见问题解答详细信息以获取有关 VNet 到 VNet 连接的其他信息。

[AZURE.INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]

<!---HONumber=Mooncake_1031_2016-->