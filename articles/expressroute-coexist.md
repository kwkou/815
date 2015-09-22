<properties
   pageTitle="配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接"
   description="本教程将指导你完成配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor="tysonn" />
<tags
   ms.service="expressroute"
   ms.date="08/05/2015"
   wacn.date=""/>

# 配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接

现在可以将 ExpressRoute 和站点到站点 VPN 连接到同一个虚拟网络。有两种不同方案和两个不同的配置过程可供选择。

## 方案

### 方案 1

在此方案中，你拥有一个本地网络。ExpressRoute 是活动链路，而站点到站点 VPN 是备份。如果 ExpressRoute 连接失败，则站点到站点 VPN 连接就会处于活动状态。此方案最适合实现高可用性。

![共存](media/expressroute-coexist/scenario1.jpg)



### 方案 2

在此方案中，你拥有两个本地网络。可以将一个本地网络通过 ExpressRoute 连接到 Azure，并将另一个本地网络通过站点到站点 VPN 连接到 Azure。在这种情况下，这两种连接将同时处于活动状态。

![共存](media/expressroute-coexist/scenario2.jpg)


## 创建和配置

若要将连接配置为共存，有两组不同的过程可供选择。你选择的配置过程将取决于你有要连接到的现有虚拟网络，还是要创建新的虚拟网络。

- **创建新的虚拟网络和共存连接：** 
	
	如果你还没有虚拟网络，此过程将指导你完成创建新的虚拟网络并创建新的 ExpressRoute 和站点到站点 VPN 连接。若要配置，请按照这些步骤[创建新的虚拟网络和连接](#create-a-new-virtual-network-and-connections-that-coexist)。

- **配置现有虚拟网络实现共存连接：**
	
	你可能已在具有现有站点到站点 VPN 连接或 ExpressRoute 连接的位置拥有虚拟网络。[为现有虚拟网络配置共存连接](#configure-connections-that-coexist-for-your-existing-virtual-network)过程将指导你完成删除网关，然后创建新的 ExpressRoute 连接和站点到站点 VPN 连接。请注意，在创建新连接时，必须按照非常特定的顺序完成步骤。不要按照其他文章中的说明来创建网关和连接。

	在此过程中，创建可以共存的连接将需要你删除网关，然后配置新网关。这意味着，在你删除并重新创建网关和连接时，跨界连接将会停止工作，但你无需将任何 VM 或服务迁移到新的虚拟网络。在你配置网关时，如果进行了相应配置，你的 VM 和服务仍可以通过负载平衡器与外界通信。


## 说明和限制

- 你将不能（通过 Azure）在通过站点到站点 VPN 连接的本地网络与通过 ExpressRoute 连接的本地网络之间进行路由。
- 你不能启用与连接到 ExpressRoute 的同一虚拟网络的点到站点 VPN 连接。对于同一虚拟网络而言，点到站点 VPN 和 ExpressRoute 不能共存。
- ExpressRoute 网关和站点到站点 VPN 网关必须是标准网关 SKU 或高性能网关 SKU。有关网关 SKU 的信息，请参阅[网关 SKU](/documentation/articles/vpn-gateway/vpn-gateway-about-vpngateways)。
- 如果你的本地网络同时连接到 ExpressRoute 和站点到站点 VPN（方案 1），则你应在本地网络中配置静态路由，以便将站点到站点 VPN 连接路由到公共 Internet。 
- 必须先创建 ExpressRoute 网关，然后再添加站点到站点 VPN 网关。
- 这两个过程都假定你已配置有一条 ExpressRoute 线路。如果你未配置，请参阅以下文章： 

	- [通过网络服务提供商 (NSP) 配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps) 
	- [通过 Exchange 提供商 (EXP) 配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)


## 创建新的虚拟网络和共存连接

此过程将指导你完成创建虚拟网络，以及创建将共存的站点到站点连接和 ExpressRoute 连接。

1. 验证你是否有最新版本的 PowerShell cmdlet。可以从[下载页](/downloads/)的 PowerShell 部分下载和安装最新的 PowerShell cmdlet。
2. 创建虚拟网络的架构。有关使用网络配置文件的详细信息，请参阅[使用网络配置文件配置虚拟网络](../virtual-network/virtual-networks-using-network-configuration-file.md)。有关配置架构的详细信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)。

	在创建架构时，请确保使用以下值：

	- 虚拟网络的网关子网必须是 /27（或更短的前缀）。
	- 网关连接类型为“专用”。

		      <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
		        <AddressSpace>
		          <AddressPrefix>10.17.159.192/26</AddressPrefix>
		        </AddressSpace>
		        <Subnets>
		          <Subnet name="Subnet-1">
		            <AddressPrefix>10.17.159.192/27</AddressPrefix>
		          </Subnet>
		          <Subnet name="GatewaySubnet">
		            <AddressPrefix>10.17.159.224/27</AddressPrefix>
		          </Subnet>
		        </Subnets>
		        <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="MyLocalNetwork">
		              <Connection type="Dedicated" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>
		      </VirtualNetworkSite>



3. 在创建并配置 xml 架构文件之后，将文件上载。这将创建你的虚拟网络。

	使用以下 cmdlet 上载文件，并将值替换为你自己的值。

	`Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'`
4. 创建 ExpressRoute 网关。请务必将 GatewaySKU 指定为 *Standard* 或 *HighPerformance*，并将 GatewayType 指定为 *DynamicRouting*。 

	使用下面的示例，并将值替换为你自己的值。

	`New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance`

5. 将 ExpressRoute 网关连接到 ExpressRoute 线路。完成此步骤后，则已通过 ExpressRoute 建立本地网络与 Azure 之间的连接。

	`New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET`

6. 接下来，创建站点到站点 VPN 网关。GatewaySKU 必须为 *Standard* 或 *HighPerformance*，GatewayType 必须为 *DynamicRouting*。

	`New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance`

	若要检索虚拟网络网关设置（包括网关 ID 和公共 IP），请使用 `Get-AzureVirtualNetworkGateway` cmdlet。

		Get-AzureVirtualNetworkGateway
		
		GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
		GatewayName          : S2SVPN
		LastEventData        :
		GatewayType          : DynamicRouting
		LastEventTimeStamp   : 5/29/2015 4:41:41 PM
		LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
		LastEventID          : 23002
		State                : Provisioned
		VIPAddress           : 104.43.x.y
		DefaultSite          :
		GatewaySKU           : HighPerformance
		Location             :
		VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
		SubnetId             :
		EnableBgp            : False
		OperationDescription : Get-AzureVirtualNetworkGateway
		OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
		OperationStatus      : Succeeded

7. 创建一个本地站点 VPN 网关实体。此命令不会配置本地 VPN 网关，而是允许你提供本地网关设置（如公共 IP 和本地地址空间），以便 Azure VPN 网关可以连接到它。

	**重要提示：**netcfg 中未定义站点到站点 VPN 的本地站点。而是，必须使用此 cmdlet 指定本地站点参数。不能使用管理门户或 netcfg 文件定义它。

	使用下面的示例，并将值替换为你自己的值：

	`New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <local-network- gateway-public-IP> -AddressSpace <local-network-address-space>`

	若要检索虚拟网络网关设置（包括网关 ID 和公共 IP），请使用 `Get-AzureVirtualNetworkGateway` cmdlet。请参阅以下示例：

		Get-AzureLocalNetworkGateway
		
		GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
		GatewayName          : MyLocalNetwork
		IpAddress            : 23.39.x.y
		AddressSpace         : {10.1.2.0/24}
		OperationDescription : Get-AzureLocalNetworkGateway
		OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
		OperationStatus      : Succeeded

	
8. 配置本地 VPN 设备以连接到新网关。配置 VPN 设备时，使用在步骤 6 中检索到的信息。有关 VPN 设备配置的详细信息，请参阅 [VPN 设备配置](http://go.microsoft.com/fwlink/p/?linkid=615099)。
	

9. 将 Azure 上的站点到站点 VPN 网关连接到本地网关。

	在此示例中，connectedEntityId 是本地网关 ID，可以通过运行 `Get-AzureLocalNetworkGateway` 来查找它。可以通过使用 `Get-AzureVirtualNetworkGateway` cmdlet 查找 virtualNetworkGatewayId。完成此步骤后，已通过站点到站点 VPN 连接建立本地网络与 Azure 之间的连接。


	`New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>`


## 为现有虚拟网络配置共存的连接

如果有通过 ExpressRoute 连接或站点到站点 VPN 连接进行连接的现有虚拟网络，要使这两个连接都连接到现有虚拟网络，必须先删除现有网关。这意味着，当你进行此配置时，本地系统将丢失通过网关与虚拟网络建立的连接。

**在开始配置之前：**确认虚拟网络中剩下足够的 IP 地址，以便你可以增加网关子网大小。


1. 下载最新版本的 PowerShell cmdlet。可以从[下载页](/downloads/)的 PowerShell 部分下载和安装最新的 PowerShell cmdlet。
 
2. 删除现有的站点到站点 VPN 网关。使用下面的 cmdlet，并将值替换为你自己的值。

	`Remove-AzureVNetGateway –VnetName MyAzureVNET`

2. 导出虚拟网络架构。使用下面的 PowerShell cmdlet，并将值替换为你自己的值。

	`Get-AzureVNetConfig –ExportToFile “C:\NetworkConfig.xml”`
3. 编辑网络配置文件架构，使网关子网为 /27（或更短的前缀）。请参阅以下示例。有关使用网络配置文件的详细信息，请参阅[使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file)。有关配置架构的详细信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)。


          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.17.159.224/27</AddressPrefix>
          </Subnet>
4. 如果以前的网关是站点到站点 VPN，则还必须将连接类型更改为**“专用”**。

		         <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="MyLocalNetwork">
		              <Connection type="Dedicated" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>
5. 此时，你将拥有不带网关的虚拟网络。你可以继续执行本文中的**步骤 3** [创建新的虚拟网络和连接](#create-a-new-virtual-network-and-connections-that-coexist)，以创建新网关和完成连接。



## 后续步骤
	
了解有关 ExpressRoute 的详细信息。请参阅 [ExpressRoute 概述](/documentation/articles/expressroute-introduction)。

了解有关 VPN 网关的详细信息。请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways)。

<!---HONumber=69-->