<properties 
	pageTitle="从 App Service 环境安全连接到后端资源" 
	description="了解如何从 App Service 环境安全连接到后端资源。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.date="06/30/2015" 
	wacn.date=""/>	

# 从 App Service 环境安全连接到后端资源 #

## 概述 ##
由于 App Service 环境始终创建于区域[虚拟网络][virtualnetwork]的子网中，因此从 App Service 环境发往其他后端资源的出站连接可以独占方式通过虚拟网络发送。

例如，SQL Server 可能在已锁定端口 1433 的虚拟机群集上运行。此终结点可能已纳入 ACL，只允许从相同虚拟网络上的其他资源进行访问。

另举一例，机密终结点可能在内部运行并通过[站点到站点][SiteToSite]或 [Azure ExpressRoute][ExpressRoute] 连接来与 Azure 建立连接。因此，只有虚拟网络中已连接到站点到站点或 ExpressRoute 隧道的资源能够访问本地终结点。

在上述这些方案中，在 App Service 环境上运行的应用能够安全地连接到各种服务器和资源。从 App Service 环境中运行的应用程序发往相同虚拟网络中专用终结点（或连接到相同的虚拟网络）的出站流量，只通过虚拟网络发送。发往专用终结点的出站流量不通过公共 Internet 发送。

## 出站连接和 DNS 要求 ##
请注意，为了使让 App Service 环境正确工作，需要对 Azure 存储空间以及相同 Azure 区域中 SQL Database 的出站访问权限。如果虚拟网络中阻止了出站 Internet 访问，则 App Service 环境将无法访问这些 Azure 终结点。

客户可能还在虚拟网络中配置了自定义 DNS 服务器。App Service 环境需要能够解析 *.database.windows.net、*.file.core.windows.net 和 *.Blob.core.windows.net 下的 Azure 终结点。

此外，还建议事先在虚拟网络上设置任何自定义 DNS 服务器，然后创建 App Service 环境。如果在创建 App Service 环境时更改虚拟网络的 DNS 配置，将会导致 App Service 环境创建过程失败。

## 连接到 SQL Server
常见的 SQL Server 配置包含一个侦听端口 1433 的终结点：

![SQL Server 终结点][SqlServerEndpoint]

有两种方法可限制发往此终结点的流量：


- [网络访问控制列表][NetworkAccessControlLists]（网络 ACL）

- [网络安全组][NetworkSecurityGroups]


## 使用网络 ACL 限制访问

使用网络访问控制列表可以保护端口 1433。以下示例将源自虚拟网络内部的客户端地址列入允许列表，并阻止对所有其他客户端的访问。

[网络访问控制列表示例][NetworkAccessControlListExample]

在与 SQL Server 相同虚拟网络的 App Service 环境中运行的所有应用程序，都将能够使用 SQL Server 虚拟机的 **VNet 内部** IP 地址连接到 SQL Server 实例。

以下连接字符串示例使用其专用 IP 地址引用 SQL Server。

    Server=tcp:10.0.1.6;Database=MyDatabase;User ID=MyUser;Password=PasswordHere;provider=System.Data.SqlClient

尽管虚拟机还有公共终结点，但使用公共 IP 地址的连接尝试将由于网络 ACL 而遭到拒绝。

## 使用网络安全组限制访问
另一种保护访问安全的方法是使用网络安全组。网络安全组可以应用到单个虚拟机，或应用到包含虚拟机的子网。

首先需要创建网络安全组：

    New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"

使用网络安全组来限制仅允许访问 VNet 内部流量的操作十分简单。网络安全组中的默认规则只允许从相同虚拟网络中的其他网络客户端访问。

因此锁定 SQL Server 的访问权限，就如同将网络安全组及其默认规则应用到运行 SQL Server 的虚拟机或包含虚拟机的子网一样简单。

以下示例将网络安全组应用到包含的子网：

    Get-AzureNetworkSecurityGroup -Name "testNSGExample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-1'
    
最终结果是一组可阻止外部访问，同时允许 VNet 内部访问的安全规则：

![默认网络安全规则][DefaultNetworkSecurityRules]


## 入门

若要开始使用 App Service 环境，请参阅 [App Service 环境简介][IntroToAppServiceEnvironment]

有关控制发往 App Service 环境的入站流量的详细信息，请参阅[控制 App Service 环境的入站流量][ControlInboundASE]

有关 Azure App Service 平台的详细信息，请参阅 [Azure App Service][AzureAppService]。

[AZURE.INCLUDE [app-service-web-whats-changed](../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../includes/app-service-web-try-app-service.md)]
 

<!-- LINKS -->
[virtualnetwork]: https://msdn.microsoft.com/zh-cn/library/azure/dn133803.aspx
[ControlInboundTraffic]: /documentation/articles/app-service-app-service-environment-control-inbound-traffic
[SiteToSite]: /documentation/articles/vpn-gateway-site-to-site-create
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
<!--[NetworkAccessControlLists]: https://msdn.microsoft.com/library/azure/dn376541.aspx-->
[NetworkSecurityGroups]: https://msdn.microsoft.com/library/azure/dn848316.aspx
[IntroToAppServiceEnvironment]: /documentation/articles/app-service-app-service-environment-intro
[AzureAppService]: /documentation/articles/app-service-value-prop-what-is
[ControlInboundASE]: /documentation/articles/app-service-app-service-environment-control-inbound-traffic

<!-- IMAGES -->
[SqlServerEndpoint]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/SqlServerEndpoint01.png
[NetworkAccessControlListExample]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/NetworkAcl01.png
[DefaultNetworkSecurityRules]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/DefaultNetworkSecurityRules01.png

<!---HONumber=66-->