<properties 
   pageTitle="使用 VPN 网关将多个本地站点连接到虚拟网络"
   description="本文指导你使用经典部署模型的 VPN 网关将多个本地站点连接到虚拟网络。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="yushwang"
   manager="rossort"
   editor=""
   tags="azure-service-management"/>

<tags 
   ms.service="vpn-gateway"
   ms.date="05/11/2016"
   wacn.date="06/24/2016" />

# 将多个本地站点连接到虚拟网络

本文适用于将多个本地站点连接到使用经典部署模型（也称为“服务管理”）创建的 VNet。当我们有使用“资源管理器”模型创建 VNet 的步骤的文章时，将在此页提供该文章的链接。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

**用于多站点连接的部署模型和工具**

[AZURE.INCLUDE [vpn-gateway-table-multi-site](../includes/vpn-gateway-table-multisite-include.md)]


## 关于连接

你可以将多个本地站点连接到单个虚拟网络。在构建混合云解决方案时，这种做法特别有用。与 Azure 虚拟网络网关建立多站点连接的操作与创建其他站点到站点连接的操作非常类似。事实上，你可以使用现有的 Azure VPN 网关，只要该网关是动态的（基于路由）即可。

如果已经有连接到虚拟网络的静态网关，可以将网关类型更改为动态，而不需要为了适应多站点而重建虚拟网络。在更改路由类型之前，请确保本地 VPN 网关支持基于路由的 VPN 配置。

![多站点示意图](./media/vpn-gateway-multi-site/multisite.png "多站点")

## 考虑的要点

**无法使用 Azure 经典管理门户对此虚拟网络进行更改。** 对于本发行版，你需要对网络配置文件进行更改，而不能使用 Azure 经典管理门户。如果在 Azure 经典管理门户中进行更改，这些更改将会覆盖此虚拟网络的多站点引用设置。

在完成多站点的相关过程后，你便可以轻松自如地使用网络配置文件。但是，如果有多个人在处理你的网络配置，你需要确保每个人都知道这个限制。这并不意味着你完全不能使用 Azure 经典管理门户。除了无法对此特定虚拟网络进行配置更改以外，你可以使用它来完成其他任何操作。

## 开始之前

在开始配置之前，请确认满足以下条件：

- Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。

- 每个本地位置都有兼容的 VPN 硬件。查看[关于用于虚拟网络连接的 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)，以确认你要使用的设备是否是已知兼容的设备。

- 每个 VPN 设备都有一个面向外部的公共 IPv4 IP 地址。该 IP 地址不能位于 NAT 后面，必须满足这一要求。

- 你需要安装 Azure PowerShell cmdlet 的最新版本。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

- 有人能够熟练地配置 VPN 硬件。无法在 Azure 经典管理门户中使用自动生成的 VPN 脚本来配置 VPN 设备。这意味着，你必须非常了解 VPN 设备的配置，或者与具有此能力的人员合作。

- 要用于虚拟网络（如果尚未创建）的 IP 地址范围。

- 要连接到的每个本地网络站点的 IP 地址范围。需确保要连接到的每个本地网络站点的 IP 地址范围不重叠。否则，Azure 经典管理门户或 REST API 将拒绝上载的配置。

	例如，如果两个本地网络站点都包含 IP 地址范围 10.2.3.0/24，并且某个包包含目标地址 10.2.3.3，则 Azure 将不知道你要将该包发送到哪个站点，因为地址范围是重叠的。为了防止路由问题，Azure 不允许你上载具有重叠范围的配置文件。



## 1\.创建站点到站点 VPN

如果你已有使用动态路由网关的站点到站点 VPN，那太好了！ 你可以转到[导出虚拟网络配置设置](#export)。否则，请执行以下操作：

### 如果已有一个站点到站点虚拟网络，但该虚拟网络使用静态（基于策略的）路由网关：

1. 将网关类型更改为动态路由。多站点 VPN 需要动态（也称为基于路由的）路由网关。若要更改网关类型，首先需要删除现有网关，然后创建新网关。有关说明，请参阅[更改 VPN 网关路由类型](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp/#how-to-change-your-vpn-gateway-type)。  

2. 配置新网关并创建 VPN 隧道。有关说明，请参阅[在 Azure 经典管理门户中配置 VPN 网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp/)。首先，将网关类型更改为动态路由。

### 如果你没有站点到站点虚拟网络：

1. 按照以下说明创建站点到站点虚拟网络：[在 Azure 经典管理门户中创建使用站点到站点 VPN 连接的虚拟网络](/documentation/articles/vpn-gateway-site-to-site-create/)。  

2. 按照以下说明配置动态路由网关：[配置 VPN 网关](/documentation/articles/vpn-gateway-configure-vpn-gateway-mp/)。请务必为网关类型选择“动态路由”。

## <a name="export"></a>2.导出网络配置文件 

导出网络配置文件。导出的文件将用于配置新的多站点设置。如果你需要有关如何导出文件的说明，请参阅以下文章中的相应部分：[如何在 Azure 经典管理门户中使用网络配置文件创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-portal/#how-to-create-a-vnet-using-a-network-config-file-in-the-azure-portal)。

## 3\.打开网络配置文件

打开你在执行上一步时下载的网络配置文件。使用你偏好的任何 xml 编辑器。该文件的内容类似于：

		<NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
		  <VirtualNetworkConfiguration>
		    <LocalNetworkSites>
		      <LocalNetworkSite name="Site1">
		        <AddressSpace>
		          <AddressPrefix>10.0.0.0/16</AddressPrefix>
		          <AddressPrefix>10.1.0.0/16</AddressPrefix>
		        </AddressSpace>
		        <VPNGatewayAddress>131.2.3.4</VPNGatewayAddress>
		      </LocalNetworkSite>
		      <LocalNetworkSite name="Site2">
		        <AddressSpace>
		          <AddressPrefix>10.2.0.0/16</AddressPrefix>
		          <AddressPrefix>10.3.0.0/16</AddressPrefix>
		        </AddressSpace>
		        <VPNGatewayAddress>131.4.5.6</VPNGatewayAddress>
		      </LocalNetworkSite>
		    </LocalNetworkSites>
		    <VirtualNetworkSites>
		      <VirtualNetworkSite name="VNet1" AffinityGroup="ChinaEast">
		        <AddressSpace>
		          <AddressPrefix>10.20.0.0/16</AddressPrefix>
		          <AddressPrefix>10.21.0.0/16</AddressPrefix>
		        </AddressSpace>
		        <Subnets>
		          <Subnet name="FE">
		            <AddressPrefix>10.20.0.0/24</AddressPrefix>
		          </Subnet>
		          <Subnet name="BE">
		            <AddressPrefix>10.20.1.0/24</AddressPrefix>
		          </Subnet>
		          <Subnet name="GatewaySubnet">
		            <AddressPrefix>10.20.2.0/29</AddressPrefix>
		          </Subnet>
		        </Subnets>
		        <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="Site1">
		              <Connection type="IPsec" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>
		      </VirtualNetworkSite>
		    </VirtualNetworkSites>
		  </VirtualNetworkConfiguration>
		</NetworkConfiguration>

## 4\.添加多个站点引用

在添加或删除站点引用信息时，将会对 ConnectionsToLocalNetwork/LocalNetworkSiteRef 进行配置更改。添加新的本地站点引用会触发 Azure 来创建新隧道。在以下示例中，网络配置适用于单站点连接。更改完后，请保存该文件。

		<Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

	To add additional site references (create a multi-site configuration), simply add additional "LocalNetworkSiteRef" lines, as shown in the example below: 

        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Site2"><Connection type="IPsec" /></LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

## 5\.导入网络配置文件

导入网络配置文件。在导入这个包含更改的文件时，将会添加新的隧道。这些隧道将使用你前面创建的动态网关。如果你需要有关如何导入文件的说明，请参阅以下文章中的相应部分：[如何在 Azure 经典管理门户中使用网络配置文件创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-portal/#how-to-create-a-vnet-using-a-network-config-file-in-the-azure-portal)。

## 6\.下载密钥

添加新的隧道后，使用 PowerShell cmdlet `Get-AzureVNetGatewayKey` 来获取每个隧道的 IPsec/IKE 预共享密钥。

例如：

	Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site1"

	Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site2"

如果需要，也可以使用获取虚拟网络网关共享密钥 REST API 来获取预共享密钥。

## 7\.验证连接

检查多站点隧道状态。下载每个隧道的密钥后，需要验证连接。请使用 `Get-AzureVnetConnection` 来获取虚拟网络隧道的列表，如以下示例所示。VNet1 是 VNet 的名称。

	Get-AzureVnetConnection -VNetName VNET1
		
	ConnectivityState         : Connected
	EgressBytesTransferred    : 661530
	IngressBytesTransferred   : 519207
	LastConnectionEstablished : 5/2/2014 2:51:40 PM
	LastEventID               : 23401
	LastEventMessage          : The connectivity state for the local network site 'Site1' changed from Not Connected to Connected.
	LastEventTimeStamp        : 5/2/2014 2:51:40 PM
	LocalNetworkSiteName      : Site1
	OperationDescription      : Get-AzureVNetConnection
	OperationId               : 7f68a8e6-51e9-9db4-88c2-16b8067fed7f
	OperationStatus           : Succeeded
		
	ConnectivityState         : Connected
	EgressBytesTransferred    : 789398
	IngressBytesTransferred   : 143908
	LastConnectionEstablished : 5/2/2014 3:20:40 PM
	LastEventID               : 23401
	LastEventMessage          : The connectivity state for the local network site 'Site2' changed from Not Connected to Connected.
	LastEventTimeStamp        : 5/2/2014 2:51:40 PM
	LocalNetworkSiteName      : Site2
	OperationDescription      : Get-AzureVNetConnection
	OperationId               : 7893b329-51e9-9db4-88c2-16b8067fed7f
	OperationStatus           : Succeeded

## 后续步骤

若要了解有关 VPN 网关的详细信息，请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。


<!---HONumber=Mooncake_0613_2016-->
