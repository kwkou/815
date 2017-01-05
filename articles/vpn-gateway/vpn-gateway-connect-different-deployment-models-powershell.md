<properties 
   pageTitle="如何使用 PowerShell 将经典虚拟网络连接到 Resource Manager 虚拟网络 | Azure"
   description="了解如何使用 VPN 网关和 PowerShell 在经典 VNet 与 Resource Manager VNet 之间创建 VPN 连接"
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
   ms.date="09/23/2016"
   wacn.date="11/07/2016"
   ms.author="cherylmc" />

# 使用 PowerShell 从不同的部署模型连接虚拟网络

> [AZURE.SELECTOR]
- [门户](/documentation/articles/vpn-gateway-connect-different-deployment-models-portal/)
- [PowerShell](/documentation/articles/vpn-gateway-connect-different-deployment-models-powershell/)

Azure 当前具有两个管理模型：经典模型和 Resource Manager (RM) 模型。如果 Azure 已经使用了一段时间，则您的 Azure VM 和实例角色可能是在经典 VNet 上运行。而较新的 VM 和角色实例可能是在 Resource Manager 中创建的 VNet 上运行。本文将指导您如何连接经典 VNet 和 Resource Manager Vnet，从而可以通过网关连接使不同部署模型中的资源能够相互通信。

可以在位于不同订阅、不同区域中的 VNet 之间创建连接。您还可以连接已连接到本地网络的 VNet，只要它们配置的网关是动态或基于路由的。有关 VNet 到 VNet 连接的详细信息，请参阅本文末尾的 [VNet 到 VNet 常见问题解答](#faq)。

### 用于连接不同部署模型中的 VNet 的部署模型和方法

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

当我们发布有关此配置的新文章和其他可用工具时，将会更新以下表格。有相关的文章发布时，我们会直接从该表格链接到该文章。<br><br>

[AZURE.INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

**VNet 对等互连**

[AZURE.INCLUDE [vpn-gateway-vnetpeeringlink](../../includes/vpn-gateway-vnetpeeringlink-include.md)]


## 开始之前

以下步骤将指导您完成为每个 VNet 配置动态或基于路由的网关以及在网关之间创建 VPN 连接所需的设置。此配置不支持静态或基于策略的网关。

### 先决条件

 - 已创建了两个 VNet。
 - 两个 VNet 的地址范围不相互重叠，也不与网关可能连接到的其他连接的任何范围重叠。
 - 已安装最新的 PowerShell cmdlet（1.0.2 或更高版本）。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。请确保安装服务管理 (SM) 和 Resource Manager (RM) cmdlet。

### <a name="exampleref"></a>示例设置

可以使用示例设置作为参考。

**经典 VNet 设置**

VNet 名称 = ClassicVNet <br>
位置 = 中国北部 <br>
虚拟网络地址空间 = 10.0.0.0/24 <br>
子网-1 = 10.0.0.0/27 <br>
网关子网 = 10.0.0.32/29 <br>
本地网络名称 = RMVNetLocal <br>
网关类型 = DynamicRouting

**Resource Manager VNet 设置**

VNet 名称 = RMVNet <br>
资源组 = RG1 <br>
虚拟网络 IP 地址空间 = 192.168.0.0/16 <br>
子网-1 = 192.168.1.0/24 <br>
网关子网 = 192.168.0.0/26 <br>
位置 = 中国东部 <br>
网关公共 IP 名称 = gwpip <br>
本地网络网关 = ClassicVNetLocal <br>
虚拟网络网关名称 = RMGateway <br>
网关 IP 寻址配置 = gwipconfig


## <a name="createsmgw"></a>第 1 节 — 配置经典 VNet

### 第 1 部分 — 下载网络配置文件。

1. 在 PowerShell 控制台中，使用提升的权限登录到 Azure 帐户。以下 cmdlet 将提示您提供 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。您将使用 SM PowerShell cmdlet 完成此部分配置。

		Add-AzureAccount -Environment AzureChinaCloud

2. 通过运行以下命令，导出 Azure 网络配置文件。如有必要，可以将文件的导出位置更改为其他位置。您将编辑文件，然后将其导入到 Azure。

		Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml

3. 打开下载的 .xml 文件进行编辑。有关网络配置文件的示例，请参阅[网络配置架构](https://msdn.microsoft.com/zh-cn/library/jj157100.aspx)。
 

### 第 2 部分 — 验证网关子网

在 **VirtualNetworkSites** 元素中，向 VNet 添加一个网关子网（如果尚未创建）。在使用网络配置文件时，网关子网必须命名为“GatewaySubnet”，否则 Azure 无法识别并将其用作网关子网。

[AZURE.INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**示例：**
	
    <VirtualNetworkSites>
      <VirtualNetworkSite name="ClassicVNet" Location="China North">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/24</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Subnet-1">
            <AddressPrefix>10.0.0.0/27</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.0.32/29</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
       
### 第 3 部分 — 添加本地网络站点

所添加的本地网络站点表示要连接到的 RM VNet。您可能需要向文件添加 **LocalNetworkSites** 元素（如果尚不存在）。此时，在配置中，VPNGatewayAddress 可以是任何有效的公共 IP 地址，因为我们尚未针对 Resource Manager VNet 创建网关。一旦创建网关，便会将此占位符 IP 地址替换为已分配给 RM 网关的正确公共 IP 地址。

    <LocalNetworkSites>
      <LocalNetworkSite name="RMVNetLocal">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### 第 4 部分 — 将 VNet 与本地网络站点关联

在此部分中，我们将指定您要将 VNet 连接到的本地网络站点。在本例中，该站点即前面提到的 Resource Manager VNet。请确保名称相匹配。此步骤不会创建网关。它指定网关将连接到的本地网络。

		<Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="RMVNetLocal">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

### 第 5 部分 — 保存文件并上传

保存文件，然后运行以下命令以将其导入到 Azure。确保根据环境需要更改文件路径。

		Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

应看到与此结果（显示导入成功）相似的内容。

		OperationDescription        OperationId                      OperationStatus                                                
		--------------------        -----------                      ---------------                                                
		Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 

### 第 6 部分 - 创建网关 

可以使用经典管理门户或 PowerShell 创建 VNet 网关。

使用以下示例创建动态路由网关：

	New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting

可以使用 `Get-AzureVNetGateway` cmdlet 检查网关状态。


## <a name="creatermgw"></a>第 2 节：配置 RM VNet 网关

若要为 RM VNet 创建 VPN 网关，请遵循以下说明。请务必在检索到经典 VNet 的网关的公共 IP 地址之后再开始执行以下步骤。

1. 在 PowerShell 控制台中，**登录到 Azure 帐户**。以下 cmdlet 将提示您提供 Azure 帐户的登录凭据。登录后将下载您的帐户设置，以便 Azure PowerShell 使用这些设置。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

 	如果您有多个订阅，则获取您的 Azure 订阅的列表。

		Get-AzureRmSubscription

	指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Name of subscription"


2.  **创建本地网络网关**。在虚拟网络中，局域网网关通常指你的本地位置。在本例中，本地网络网关是指经典 VNet。指定该网关的名称以供 Azure 引用，同时指定地址空间前缀。Azure 使用指定的 IP 地址前缀来识别要发送到本地位置的流量。如果稍后需要在创建网关之前调整此处的信息，可以修改这些值并再次运行该示例。<br><br>
	- `-Name` 是要分配来指代本地网络网关的名称。<br>
	- `-AddressPrefix` 是经典 VNet 的地址空间。<br>
	- `-GatewayIpAddress` 是经典 VNet 网关的公共 IP 地址。请务必更改下面的示例以反映正确的 IP 地址。
	
			New-AzureRmLocalNetworkGateway -Name ClassicVNetLocal `
			-Location "China North" -AddressPrefix "10.0.0.0/24" `
			-GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1

3. **请求一个公共 IP 地址**并将其分配到 Resource Manager VNet 的虚拟网络网关。无法指定要使用的 IP 地址。IP 地址动态分配到虚拟网络网关。但是，这并不意味着 IP 地址会更改。虚拟网络网关 IP 地址只在删除或重新创建网关时更改。它不会因为网关调整大小、重置或其他内部维护/升级而更改。<br>在此步骤中，我们还将设置一个在后续步骤中要使用的变量。


		$ipaddress = New-AzureRmPublicIpAddress -Name gwpip `
		-ResourceGroupName RG1 -Location 'ChinaEast' `
		-AllocationMethod Dynamic

4. **验证虚拟网络是否有网关子网**。如果不存在任何网关子网，则添加一个。请确保网关子网命名为 *GatewaySubnet*。

5. 通过运行以下命令，**检索用于网关的子网**。在此步骤中，我们还将设置一个要在下一步使用的变量。
	- `-Name` 是 Resource Manager VNet 的名称。
	- `-ResourceGroupName` 是 VNet 所关联的资源组。此 VNet 必须已经存在网关子网，并且该子网必须命名为 *GatewaySubnet* 才能正常工作。


			$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet `
			-VirtualNetwork (Get-AzureRmVirtualNetwork -Name RMVNet -ResourceGroupName RG1) 

6. **创建网关 IP 寻址配置**。网关配置定义要使用的子网和公共 IP 地址。使用下面的示例创建网关配置。<br>在此步骤中，`-SubnetId` 和 `-PublicIpAddressId` 参数必须分别从子网和 IP 地址对象传递 ID 属性。不能使用简单字符串。将在请求公共 IP 的步骤和检索子网的步骤中设置这些变量。

		$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig `
		-Name gwipconfig -SubnetId $subnet.id `
		-PublicIpAddressId $ipaddress.id
	
7. 运行以下命令**创建 Resource Manager 虚拟网络网关**。`-VpnType` 必须是 *RouteBased* 。可能需要 45 分钟或更长的时间才能完成此过程。

		New-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
		-Location "ChinaEast" -GatewaySKU Standard -GatewayType Vpn `
		-IpConfigurations $gwipconfig `
		-EnableBgp $false -VpnType RouteBased

8. 一旦创建 VPN 网关，便**复制公共 IP 地址**。为经典 VNet 配置本地网络设置时要使用该地址。可以使用以下 cmdlet 来检索公共 IP 地址。公共 IP 地址在返回结果中作为 *IpAddress* 列出。

		Get-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName RG1

## 第 3 节：修改经典 VNet 的本地网络

若要执行此步骤，可以导出网络配置文件、编辑它，然后保存并将文件导回到 Azure。也可以在经典管理门户中修改此设置。

### 在门户中修改

1. 打开[经典管理门户](https://manage.windowsazure.cn)。

2. 在经典管理门户的左侧向下滚动，然后单击“网络”。在“网络”页上，单击页面顶部的“本地网络”。

3. 在“本地网络”页上，单击以选择在第 1 部分中配置的本地网络，然后转到页面底部并单击“编辑”。

4. 在“指定本地网络详细信息”页上，将占位符 IP 地址替换为上一节中创建的 Resource Manager 网关的公共 IP 地址。单击箭头转到下一部分。确认“地址空间”是正确的，然后单击复选标记以接受更改。

### 使用网络配置文件进行修改

1. 导出网络配置文件。
2. 在文本编辑器中，修改 VPN 网关 IP 地址。

	    <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>

3. 保存所做更改并将编辑后的文件导回到 Azure。


## <a name="connect"></a>第 4 节：在网关之间创建连接

在网关之间创建连接需要用到 PowerShell。您可能需要添加 Azure 帐户才能使用经典 PowerShell cmdlet。若要执行此操作，可以使用以下 cmdlet：
		
	Add-AzureAccount -Environment AzureChinaCloud

1. 通过运行以下命令**设置共享密钥**。在本例中，`-VNetName` 是经典 VNet 的名称，`-LocalNetworkSiteName` 是在经典管理门户中配置本地网络时为其指定的名称。`-SharedKey` 是您可以生成并指定的值。此处指定的值必须与下一步中创建连接时指定的值相同。本示例的返回结果应显示“状态: 成功”。

		Set-AzureVNetGatewayKey -VNetName ClassicVNet `
		-LocalNetworkSiteName RMVNetLocal -SharedKey abc123

2. 通过运行以下命令创建 VPN 连接。

	**设置变量**

		$vnet01gateway = Get-AzureRMLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
		$vnet02gateway = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1

	**创建连接**<br> 请注意，`-ConnectionType` 为 IPsec，而不是 Vnet2Vnet。
		
		New-AzureRmVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
		-Location "China East" -VirtualNetworkGateway1 `
		$vnet02gateway -LocalNetworkGateway2 `
		$vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

3. 可以在门户中或使用 `Get-AzureRmVirtualNetworkGatewayConnection` cmdlet 查看连接。

		Get-AzureRmVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1

## <a name="faq"></a>VNet 到 VNet 常见问题解答

查看常见问题解答详细信息以获取有关 VNet 到 VNet 连接的其他信息。

[AZURE.INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]

<!---HONumber=Mooncake_1031_2016-->