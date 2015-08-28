<properties 
	pageTitle="使用 Express Route 时的网络配置详细信息" 
	description="在已连接到 ExpressRoute 线路的虚拟网络中运行 App Service 环境时的网络配置详细信息。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="stefsch" 
	manager="nirma" 
	editor=""/>

<tags 
	ms.service="app-service-web"  
	ms.date="06/30/2015" 
	wacn.date=""/>	

# 使用 ExpressRoute 的 App Service 环境的网络配置详细信息 

## 概述 ##
客户可以将 [Azure ExpressRoute][ExpressRoute] 线路连接到虚拟网络基础结构，从而将其本地网络扩展到 Azure。可以在这个[虚拟网络][virtualnetwork]基础结构的子网中创建 App Service 环境。然后，在 App Service 环境上运行的应用程序可与后端资源建立安全连接，而后端资源只能通过 ExpressRoute 连接来访问。

## 所需的网络连接 ##
在已连接到 ExpressRoute 的虚拟网络中，可能一开始不符合 App Service 环境的网络连接要求。

App Service 环境需要以下所有项目才能正确工作：


-  与 App Service 环境位于相同区域的 Azure 存储空间和 SQL 数据库资源的出站网络连接。此网络路径不能遍历内部公司代理，因为这样做可能会更改出站网络流量的有效 NAT 地址。更改在 Azure 存储空间和 SQL 数据库上定向的 App Service 环境出站网络流量的 NAT 地址会导致连接失败。
-  虚拟网络的 DNS 配置必须可以解析以下 Azure 控制域中的终结点：**.file.core.windows.net*、**.blob.core.windows.net*、**.database.windows.net*。
-  每当创建 App Service 环境时，以及对 App Service 环境进行重新配置和缩放更改期间，虚拟网络的 DNS 配置必须保持稳定。   
-  必须根据此[文章][requiredports]中所述，允许对 App Service 环境所需的端口进行入站网络访问。

确保虚拟网络的 DNS 配置有效，即可满足 DNS 要求。

根据此[文章][requiredports]中所述，在 App Service 环境的子网上设置[网络安全组][NetworkSecurityGroups]以允许所需访问权限，即可满足入站网络访问的要求。

## 启用 App Service 环境的出站网络连接##
默认情况下，新创建的 ExpressRoute 线路将会通告允许出站 Internet 连接的默认路由。使用此配置，App Service 环境可以连接到其他 Azure 终结点。

但是，常见的客户配置是定义其自身的默认路由，以强制出站 Internet 流量通过本地流向客户的代理/防火墙基础结构。此流量一定会中断 App Service 环境，因为已在本地阻止出站流量，或者 NAT 到无法再使用各种 Azure 终结点的一组无法识别的地址。

解决方法是在子网上定义包含 App Service 环境的一个或多个用户定义路由 (UDR)。UDR 定义了要遵循的子网特定路由，而不是默认路由。

有关用户定义路由的背景信息，请参阅此[概述][UDROverview]。

有关创建和配置用户定义路由的详细信息，请参阅此[操作方法指南][UDRHowTo]。

## App Service 环境的示例 UDR 配置 ##

**先决条件**

1. 从 [Azure 下载页面][AzureDownloads]安装最新的 Azure Powershell（日期为 2015 年 6 月或更晚）。在“命令行工具”的“Windows Powershell”下，有一个“安装”链接可用于安装最新的 Powershell cmdlet。

2. 建议你创建唯一的子网，以专供 App Service 环境使用。这可确保应用到子网的 UDR 只会打开 App Service 环境的出站流量。
3. **重要说明**：只有在完成以下配置步骤**之后**，才可以部署 App Service 环境。这可确保在尝试部署 App Service 环境之前出站网络连接可用。

**步骤 1：创建命名路由表**

以下代码段将在 Azure 美国西部区域创建名为“DirectInternetRouteTable”的路由表：

    New-AzureRouteTable -Name 'DirectInternetRouteTable' -Location uswest

**步骤 2：在路由表中创建一个或多个路由**

需要将一个或多个路由添加到路由表中，才能启用出站 Internet 访问。以下示例添加了足够多的路由，以涵盖美国西部区域中使用的所有可能 Azure 地址。

    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 1' -AddressPrefix 23.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 2' -AddressPrefix 40.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 3' -AddressPrefix 65.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 4' -AddressPrefix 104.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 5' -AddressPrefix 137.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 6' -AddressPrefix 138.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 7' -AddressPrefix 157.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 8' -AddressPrefix 168.0.0.0/8 -NextHopType Internet
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 9' -AddressPrefix 191.0.0.0/8 -NextHopType Internet


有关 Azure 使用的 CIDR 范围的完整和已更新列表，可以从 [Microsoft 下载中心][DownloadCenterAddressRanges]下载包含所有范围的 XML 文件。

**注意：**在某个时间点，缩写的 CIDR 简称 0.0.0.0/0 将可用于 *AddressPrefix* 参数。这个简称等同于“所有 Internet 地址”。现在，开发人员需要改为使用一组足以涵盖所有可能 Azure 地址范围的更广 CIDR 范围，而 Azure 地址范围用于已部署 App Service 环境的区域中。

**步骤 3：将路由表关联到包含 App Service 环境的子网**

最后一个配置步骤是将路由表关联到部署 App Service 环境的子网。以下命令将“DirectInternetRouteTable”关联到最终会包含 App Service 环境的“ASESubnet”。

    Set-AzureSubnetRouteTable -VirtualNetworkName 'YourVirtualNetworkNameHere' -SubnetName 'ASESubnet' -RouteTableName 'DirectInternetRouteTable'


**步骤 4：最后的步骤**

将路由表绑定到子网后，建议先测试并确认预期效果。例如，将虚拟机部署到子网，并确认：


- 发往 Azure 终结点的出站流量不会沿着 ExpressRoute 线路流动。
- 已正确解析 Azure 终结点的 DNS 查找。 

确认上述步骤之后，即可继续创建 App Service 环境！

## 入门

若要开始使用 App Service 环境，请参阅 [App Service 环境简介][IntroToAppServiceEnvironment]

有关 Azure App Service 平台的详细信息，请参阅 [Azure App Service][AzureAppService]。

<!-- LINKS -->
[virtualnetwork]: /documentation/services/virtual-network
<!--[ExpressRoute]: http://azure.microsoft.com/services/expressroute/-->
[requiredports]: /documentation/articles/app-service-app-service-environment-control-inbound-traffic
[NetworkSecurityGroups]: /documentation/articles/virtual-networks-nsg
[UDROverview]: /documentation/articles/virtual-networks-udr-overview
[UDRHowTo]: /documentation/articles/virtual-networks-udr-how-to
[HowToCreateAnAppServiceEnvironment]: /documentation/articles/app-service-web-how-to-create-an-app-service-environment
[AzureDownloads]: /documentation/downloads/downloads
[DownloadCenterAddressRanges]: http://www.microsoft.com/zh-cn/download/details.aspx?id=41653
[NetworkSecurityGroups]: /documentation/articles/virtual-networks-nsg
[AzureAppService]: /documentation/articles/app-service-value-prop-what-is/
[IntroToAppServiceEnvironment]: /documentation/articles/app-service-app-service-environment-intro/
 

<!-- IMAGES -->

<!---HONumber=66-->