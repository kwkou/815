<properties
   pageTitle="配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接 | Azure"
   description="本文将指导你配置可在经典部署模型中并存的 ExpressRoute 连接和站点到站点 VPN 连接。"
   documentationCenter="na"
   services="expressroute"
   authors="charwen"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.date="04/06/2016"
   wacn.date="06/06/2015"/>

# 为经典部署模型配置 ExpressRoute 和站点到站点共存连接


> [AZURE.SELECTOR]
- [PowerShell - Resource Manager](/documentation/articles/expressroute-howto-coexist-resource-manager/)
- [PowerShell - 经典](/documentation/articles/expressroute-howto-coexist-classic/)

能够配置站点到站点 VPN 和 ExpressRoute 具有多项优势。你可以将站点到站点 VPN 配置为 ExressRoute 的安全故障转移路径，或者使用站点到站点 VPN 连接到不是通过 ExpressRoute 进行连接的站点。我们将在本文中介绍这两种方案的配置步骤。本文适用于经典部署模型。此配置在门户中不可用。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

>[AZURE.IMPORTANT] 按以下说明进行操作之前，必须预先配置ExpressRoute 线路。在按以下步骤操作之前，请务必遵循相关指南来[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)和[配置路由](/documentation/articles/expressroute-howto-routing-classic/)。

## 限制和局限性

- **不支持转换性路由：**你将不能（通过 Azure）在通过站点到站点 VPN 连接的本地网络与通过 ExpressRoute 连接的本地网络之间进行路由。
- **不支持点到站点连接：**你不能启用点到站点 VPN 连接来连接到已连接到 ExpressRoute 的同一 VNet。对于同一 VNet 而言，点到站点 VPN 和 ExpressRoute 不能共存。
- **无法在站点到站点 VPN 网关上启用强制隧道：**你只能通过 ExpressRoute 将所有 Internet 绑定流量“强制”返回到本地网络。 
- **仅限标准或高性能网关：**ExpressRoute 网关和站点到站点 VPN 网关必须使用标准或高性能网关。有关网关 SKU 的信息，请参阅[网关 SKU](/documentation/articles/vpn-gateway-about-vpngateways/)。
- **仅限基于路由的 VPN 网关：**必须使用基于路由的 VPN 网关。有关基于路由的 VPN 网关的信息，请参阅 [VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。
- **静态路由要求：**如果你的本地网络同时连接到 ExpressRoute 和站点到站点 VPN，则必须在本地网络中配置静态路由，以便将站点到站点 VPN 连接路由到公共 Internet。
- **必须先配置 ExpressRoute 网关：**必须先创建 ExpressRoute 网关，然后再添加站点到站点 VPN 网关。


## 配置设计

### 将站点到站点 VPN 配置为 ExpressRoute 的故障转移路径

你可以将站点到站点 VPN 连接配置为 ExpressRoute 的备份。这仅适用于链接到 Azure 专用对等路径的虚拟网络。对于可通过 Azure 公共线路访问的服务，没有基于 VPN 的故障转移解决方案。ExpressRoute 线路始终是主链接。仅当 ExpressRoute 线路失败时，数据才会流经站点到站点 VPN 路径。

![共存](./media/expressroute-howto-coexist-classic/scenario1.jpg)

### 配置站点到站点 VPN，以便连接到不通过 ExpressRoute 进行连接的站点

你可以对网络进行配置，使得部分站点通过站点到站点 VPN 直接连接到 Azure，部分站点通过 ExpressRoute 进行连接。

![共存](./media/expressroute-howto-coexist-classic/scenario2.jpg)

>[AZURE.NOTE]不能将虚拟网络配置为转换路由器。

## 选择要使用的步骤

若要配置能够共存的连接，有两组不同的过程可供选择。你选择的配置过程将取决于你有要连接到的现有虚拟网络，还是要创建新的虚拟网络。


- 我没有 VNet，需要创建一个。
	
	如果你还没有虚拟网络，此过程将指导你使用经典部署模型创建新的虚拟网络，然后创建新的 ExpressRoute 和站点到站点 VPN 连接。若要配置，请遵循本文中[创建新的虚拟网络和并存连接](#new)部分中的步骤。

- 我已有一个经典部署模型 VNet。

	你可能已在具有现有站点到站点 VPN 连接或 ExpressRoute 连接的位置拥有虚拟网络。[为现有的 VNet 配置并存连接](#add)部分将指导你删除网关，然后创建新的 ExpressRoute 连接和站点到站点 VPN 连接。请注意，在创建新连接时，必须按照非常特定的顺序完成步骤。不要按照其他文章中的说明来创建网关和连接。

	在此过程中，创建可以共存的连接将需要你删除网关，然后配置新网关。这意味着，在你删除并重新创建网关和连接时，跨界连接将会停止工作，但你无需将任何 VM 或服务迁移到新的虚拟网络。在你配置网关时，如果进行了相应配置，你的 VM 和服务仍可以通过负载平衡器与外界通信。


## <a name="new"></a>创建新的虚拟网络和并存连接

本过程将指导你创建 VNet，以及创建将共存的站点到站点连接和 ExpressRoute 连接。

1. 你需要安装最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。请注意，针对此配置使用的 cmdlet 可能与你熟悉的 cmdlet 稍有不同。请务必使用说明内容中指定的 cmdlet。 

2. 创建虚拟网络的架构。有关配置架构的详细信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)。

	在创建架构时，请确保使用以下值：

	- 虚拟网络的网关子网必须是 /27 或更短的前缀（例如 /26 或 /25）。
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

		Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'

4. <a name="gw"></a>创建 ExpressRoute 网关。请务必将 GatewaySKU 指定为 Standard 或 HighPerformance，并将 GatewayType 指定为 DynamicRouting。

	使用以下示例，将值替换为你自己的值。

		New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance

5. 将 ExpressRoute 网关连接到 ExpressRoute 线路。完成此步骤后，则已通过 ExpressRoute 建立本地网络与 Azure 之间的连接。

		New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET

6. <a name="vpngw"></a>接下来，创建站点到站点 VPN 网关。GatewaySKU 必须为 Standard 或 HighPerformance，GatewayType 必须为 DynamicRouting。

		New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance

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

	>[AZURE.IMPORTANT] netcfg 中未定义站点到站点 VPN 的本地站点。而是，必须使用此 cmdlet 指定本地站点参数。不能使用门户或 netcfg 文件定义它。

	使用下面的示例，并将值替换为你自己的值。

	`New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <MyLocalGatewayIp> -AddressSpace <MyLocalNetworkAddress>`

	> [AZURE.NOTE]如果你的本地网络具有多个路由，你可以通过数组的形式将其全部传入。$MyLocalNetworkAddress = @("10.1.2.0/24","10.1.3.0/24","10.2.1.0/24")


	若要检索虚拟网络网关设置（包括网关 ID 和公共 IP），请使用 `Get-AzureVirtualNetworkGateway` cmdlet。请参阅以下示例。

		Get-AzureLocalNetworkGateway

		GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
		GatewayName          : MyLocalNetwork
		IpAddress            : 23.39.x.y
		AddressSpace         : {10.1.2.0/24}
		OperationDescription : Get-AzureLocalNetworkGateway
		OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
		OperationStatus      : Succeeded


8. 配置本地 VPN 设备以连接到新网关。配置 VPN 设备时，使用在步骤 6 中检索到的信息。有关 VPN 设备配置的详细信息，请参阅 [VPN 设备配置](/documentation/articles/vpn-gateway-about-vpn-devices/)。

9. 将 Azure 上的站点到站点 VPN 网关连接到本地网关。

	在此示例中，connectedEntityId 是本地网关 ID，可以通过运行 `Get-AzureLocalNetworkGateway` 来查找它。可以通过使用 `Get-AzureVirtualNetworkGateway` cmdlet 查找 virtualNetworkGatewayId。完成此步骤后，已通过站点到站点 VPN 连接建立本地网络与 Azure 之间的连接。


	`New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>`

## <a name="add"></a>为现有的 VNet 配置并存连接

如果你已经有了一个虚拟网络，请检查网关子网大小。如果网关子网为 /28 或 /29，则必须先删除虚拟网络网关，然后增加网关子网大小。本部分的步骤将说明如何这样做。

如果网关子网为 /27 或更大，且虚拟网络是通过 ExpressRoute 连接的，则可跳过下面的步骤，转到前一部分的[“步骤 6 - 创建站点到站点 VPN 网关”](#vpngw)。

>[AZURE.NOTE] 如果你删除的是现有网关，则当你进行此配置时，本地系统将失去与虚拟网络建立的连接。

1. 你需要安装最新版本的 Azure Resource Manager PowerShell cmdlet。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。请注意，针对此配置使用的 cmdlet 可能与你熟悉的 cmdlet 稍有不同。请务必使用说明内容中指定的 cmdlet。 

2. 删除现有的 ExpressRoute 或站点到站点 VPN 网关。使用下面的 cmdlet，并将值替换为你自己的值。

	`Remove-AzureVNetGateway -VnetName MyAzureVNET`

3. 导出虚拟网络架构。使用下面的 PowerShell cmdlet，并将值替换为你自己的值。

	`Get-AzureVNetConfig -ExportToFile "C:\NetworkConfig.xml"`

4. 编辑网络配置文件架构，使网关子网为 /27 或更短的前缀（例如 /26 或 /25）。请参阅以下示例。
>[AZURE.NOTE] 如果你因为虚拟网络中没有剩余足够的 IP 地址而无法增加网关子网大小，则需增加 IP 地址空间。有关配置架构的详细信息，请参阅 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/library/azure/jj157100.aspx)。

          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.17.159.224/27</AddressPrefix>
          </Subnet>

5. 如果以前的网关是站点到站点 VPN，则还必须将连接类型更改为**“专用”**。

		         <Gateway>
		          <ConnectionsToLocalNetwork>
		            <LocalNetworkSiteRef name="MyLocalNetwork">
		              <Connection type="Dedicated" />
		            </LocalNetworkSiteRef>
		          </ConnectionsToLocalNetwork>
		        </Gateway>

6. 此时，你将拥有不带网关的虚拟网络。若要创建新网关并完成连接，可以转到[步骤 4 - 创建 ExpressRoute 网关](#gw)（可以在前一组步骤中找到）。

## 后续步骤

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。

<!---HONumber=82-->