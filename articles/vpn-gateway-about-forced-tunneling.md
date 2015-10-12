<properties 
	pageTitle="为 Microsoft Azure VPN 网关配置强制隧道 | Microsoft Azure" 
	description="如果你的虚拟网络具有跨界 VPN 网关，你可以将全部 Internet 绑定流量重定向或强制返回到本地位置。" 
	services="vpn-gateway" 
	documentationCenter="na" 
	authors="cherylmc" 
	manager="carolz" editor="" />
<tags  
   ms.service="vpn-gateway"
   ms.date="08/20/2015"
   wacn.date="10/08/2015" />

# 配置强制隧道

借助强制隧道，您可以通过站点到站点 VPN 隧道，将全部 Internet 绑定流量重定向或“强制”返回到本地位置，以进行检查和审核。这是很多企业 IT 策略的关键安全要求。没有强制隧道，来自 Azure 中虚拟机的 Internet 绑定流量会始终通过 Azure 网络基础设施直接连接到 Internet。没有该选项，您无法对流量进行检查或审核。未经授权的 Internet 访问可能会导致信息泄漏或其他类型的安全漏洞。

下图说明了强制隧道的工作方式。

![强制隧道](./media/vpn-gateway-about-forced-tunneling/forced-tunnel.png)

在上面的示例中，前端子网没有使用强制隧道。前端子网中的工作负载可以继续直接接受并响应来自 Internet 的客户请求。中间层和后端子网会使用强制隧道。任何从这两个子网到 Internet 的出站连接将通过一个 S2S VPN 隧道重定向或强制返回到本地站点。这样，在继续支持所需的多层服务体系结构的同时，您可以限制并检查来自虚拟机或 Azure 云服务的 Internet 访问。如果在虚拟网络中没有面向 Internet 的工作负载，您还可以选择在整个虚拟网络应用强制隧道连接。

## 要求和注意事项

在 Azure 中，通过虚拟网络用户定义路由配置强制隧道。将流量重定向到本地站点，这是 Azure VPN 网关的默认路由。以下部分列出了 Azure 虚拟网络路由和路由表的当前限制：



-  每个虚拟网络子网具有内置的系统路由表。系统路由表具有下面的 3 组路由：

	- **本地 VNet 路由：**直接路由到同一个虚拟网络中的目标虚拟机
	
	- **本地路由：**路由到 Azure VPN 网关
	
	- **默认路由：**直接路由到 Internet。请注意，如果要将数据包发送到不包含在前面两个路由中的专用 IP 地址，数据包将被删除。



-  在发布的用户定义路由中，您可以创建路由表来添加默认路由，然后将路由表关联到虚拟网络子网，在这些子网启用强制隧道。

- 强制隧道必须关联到具有动态路由 VPN 网关的 VNet，不能是静态网关。您需要在连接到虚拟网络的跨界本地站点中，设置一个“默认站点”。

- 请注意，ExpressRoute 强制隧道不是通过此机制配置的，而是通过 ExpressRoute BGP 对等会话播发默认路由来启用的。有关详细信息，请参阅 [ExpressRoute 文档](https://azure.microsoft.com/documentation/services/expressroute/)。

## 配置概述

以下过程将帮助你为虚拟网络指定强制隧道。配置步骤与下面的虚拟网络 netcfg 文件示例相对应。

在本示例中，虚拟网络“MultiTier-VNet”具有 3 个子网：*前端*、*中间层*和*后端*子网，具有 4 个跨界连接：一个 *DefaultSiteHQ* 和 3 个*分支*。以下过程步骤将 *DefaultSiteHQ* 设置为使用强制隧道的默认站点连接，并将*中间层*和*后端*子网配置为使用强制隧道。

	<VirtualNetworkSite name="MultiTier-VNet" Location="North Europe">
     <AddressSpace>
      <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Frontend">
            <AddressPrefix>10.1.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Midtier">
            <AddressPrefix>10.1.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="Backend">
            <AddressPrefix>10.1.2.0/23</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.1.200.0/28</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="DefaultSiteHQ">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch2">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
            <LocalNetworkSiteRef name="Branch3">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
        </Gateway>
      </VirtualNetworkSite>
	</VirtualNetworkSite>

### 先决条件

- Azure 订阅

- 已配置虚拟网络。

- 使用 Web 平台安装程序的 Azure PowerShell cmdlet 最新版本。可以从[下载页面](http://azure.microsoft.com/downloads/)的“Windows PowerShell”部分下载并安装最新版本。

## 配置强制隧道

使用以下过程配置强制隧道。

1. 创建一个路由表。使用以下 cmdlet 创建路由表。

		New-AzureRouteTable –Name "MyRouteTable" –Label "Routing Table for Forced Tunneling" –Location "North Europe"

1. 将默认路由添加到路由表中。

	下面的 cmdlet 示例将默认路由添加到在步骤 1 中创建的路由表。请注意，唯一支持的路由是“0.0.0.0/0”到“VPN 网关”下一跃点的目标前缀。
 
		Set-AzureRoute –RouteTableName "MyRouteTable" –RouteName "DefaultRoute" –AddressPrefix "0.0.0.0/0" –NextHopType VPNGateway

1. 将路由表关联到子网。

	创建路由表并添加路由后，可以使用以下 cmdlet 将路由表添加到 VNet 子网，或将路由表与 VNet 子网关联。下面的示例将“MyRouteTable”路由表添加到 VNet MultiTier-VNet 的中间层和后端子网。

		Set-AzureSubnetRouteTable -VNetName "MultiTier-VNet" -SubnetName "Midtier" -RouteTableName "MyRouteTable"

		Set-AzureSubnetRouteTable -VNetName "MultiTier-VNet" -SubnetName "Backend" -RouteTableName "MyRouteTable"

1. 为强制隧道指定默认站点。

	在前面的步骤中，示例 cmdlet 脚本创建了路由表，并将路由表关联到两个 VNet 子网。剩下的步骤是在虚拟网络的多站点连接中，选择一个本地站点作为默认站点或隧道。

		$DefaultSite = @("DefaultSiteHQ")
		Set-AzureVNetGatewayDefaultSite –VNetName "MultiTier-VNet" –DefaultSite "DefaultSiteHQ"

## 其他 PowerShell cmdlet

在使用强制隧道配置时，你可能会发现下面一些附加的 PowerShell cmdlet 很有用。

**删除路由表：**

	Remove-AzureRouteTable -RouteTableName <routeTableName>

**列出路由表：**

	Get-AzureRouteTable [-Name <routeTableName> [-DetailLevel <detailLevel>]]

**从路由表中删除路由：**

	Remove-AzureRouteTable –Name <routeTableName>

**从子网中删除路由：**

	Remove-AzureSubnetRouteTable –VNetName <virtualNetworkName> -SubnetName <subnetName>

**列出与子网关联的路由表：**
	
	Get-AzureSubnetRouteTable -VNetName <virtualNetworkName> -SubnetName <subnetName>

**从 VNet VPN 网关中删除默认站点：**

	Remove-AzureVnetGatewayDefaultSites -VNetName <virtualNetworkName>

## 后续步骤

有关保护网络流量的信息。参阅[什么是网络安全组](../virtual-network/virtual-networks-nsg.md)。

<!---HONumber=71-->