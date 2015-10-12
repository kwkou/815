<properties
	pageTitle="Azure 基础结构服务中托管的 SharePoint 场"
	description="查找介绍如何在 Windows Azure 基础结构服务中设置开发/测试或生产的 SharePoint 2013 场的关键文章。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="07/21/2015"
	wacn.date="09/18/2015"/>

# Azure 基础结构服务中托管的 SharePoint 场

在 Windows Azure 基础结构服务中设置你的第一个或下一个开发/测试或生产 SharePoint 场，在其中你可以利用配置的简便性和相关功能快速扩展场以包括关键功能的新增能力或优化。

## 基本 SharePoint 开发/测试场

对于服务管理中创建的虚拟机，可使用 Azure 预览门户的 [SharePoint 服务器场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)功能来为面向 Internet 的 SharePoint 网站创建基本开发/测试场。

自动创建的环境由仅限云的 Azure 虚拟网络中的三个服务器组成：分别用于域控制器、SQL Server 和 SharePoint 服务器。

若要创建与资源管理器中创建的虚拟机类似的配置，请使用模板。请参阅[部署包含 3 个服务器的 SharePoint 场](/documentation/articles/virtual-machines-workload-template-sharepoint#deploy-a-three-server-sharepoint-farm)。

## 高可用性 SharePoint 开发/测试场

对于服务管理中创建的虚拟机，可使用 Azure 预览门户的 [SharePoint 服务器场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)功能来为面向 Internet 的 SharePoint 网站创建高可用性 SharePoint 开发/测试场。

在仅限云的 Azure 虚拟网络中自动创建的环境由九个服务器组成：两个用于域控制器、三个用于 SQL Server 群集、两个应用程序层 SharePoint 服务器和两个 Web 层 SharePoint 服务器。

若要创建与资源管理器中创建的虚拟机类似的配置，请使用模板。请参阅[部署包含 9 个服务器的 SharePoint 场](/documentation/articles/virtual-machines-workload-template-sharepoint#deploy-a-nine-server-sharepoint-farm)。

## 混合云开发/测试场

使用[混合云开发/测试环境中的 SharePoint Intranet 场](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)，可以创建模拟的混合云配置，以托管简单的两层 SharePoint 场，你可以从 Internet 上你所在位置使用该场测试 Azure 中托管的 Intranet SharePoint 场。

此配置使用服务管理中创建的虚拟机。

## 高度可用的 Intranet SharePoint 生产场

借助于[使用 Azure 中的 SQL Server AlwaysOn 可用性组部署 SharePoint 2013](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)，你可以在 Azure 中构建出生产就绪的高可用性 Intranet SharePoint Server 2013 场。

此配置使用服务管理中创建的虚拟机。

## 其他资源

有关 Azure 中的 SharePoint 的其他信息和配置，请参阅以下资源：

- [适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)

- [在 Windows Azure 中使用 SharePoint Server 2013 的 Internet 站点](https://technet.microsoft.com/zh-cn/library/dn635307.aspx)

- [Windows Azure 中的 SharePoint Server 2013 灾难恢复](https://technet.microsoft.com/zh-cn/library/dn635313.aspx)

- [使用 Windows Azure Active Directory 进行 SharePoint 2013 身份验证](https://technet.microsoft.com/zh-cn/library/dn635311.aspx)

- [在 Windows Azure 中部署 Office 365 目录同步 (DirSync)](https://technet.microsoft.com/zh-cn/library/dn635310.aspx)

<!---HONumber=70-->