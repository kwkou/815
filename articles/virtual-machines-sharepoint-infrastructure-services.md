<properties
	pageTitle="Azure 中的 SharePoint Server 2013 场 | Azure"
	description="查找说明如何在 Azure 中设置开发/测试环境或生产 SharePoint Server 2013 场的文章。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/21/2016"
	wacn.date=""/>

# Azure 基础结构服务中托管的 SharePoint 场

[AZURE.INCLUDE [learn-about-deployment-models-both-include](../includes/learn-about-deployment-models-both-include.md)]

在 Azure 基础结构服务中设置你的第一个或下一个开发/测试或生产 SharePoint Server 2013 场，在其中你可以利用配置的简便性和相关功能快速扩展场以包括关键功能的新增能力或优化。

> [AZURE.NOTE] Microsoft 已发布 SharePoint Server 2016 IT Preview。要使此预览版易于安装和测试，可以使用预安装了 SharePoint Server 2016 IT Preview 及其必备组件的 Azure 虚拟机库映像。有关详细信息，请参阅[在 Azure 中测试 SharePoint Server 2016 IT Preview](https://azure.microsoft.com/blog/test-sharepoint-server-2016-it-preview-4/)。

## 基本 SharePoint 开发/测试场

此自动创建的环境由仅限云的 Azure 虚拟网络中的三个服务器组成：分别是域控制器、SQL Server 和 SharePoint 服务器。

## 高可用性 SharePoint 开发/测试场

此自动创建的环境由仅限云的 Azure 虚拟网络中的九个服务器组成：两个用于域控制器、三个用于 SQL Server 群集、两个应用程序层 SharePoint 服务器和两个 Web 层 SharePoint 服务器。

## 混合云开发/测试场

使用[混合云开发/测试环境中的 SharePoint Intranet 场](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)，可以创建模拟的混合云配置，以托管简单的两层 SharePoint 场，你可以从 Internet 上你所在位置使用该场测试 Azure 中托管的 Intranet SharePoint 场。

## 后续步骤

- 发现 Azure 基础结构服务中的其他 [SharePoint 2013](https://technet.microsoft.com/zh-cn/library/dn635309.aspx) 配置。

<!---HONumber=Mooncake_0321_2016-->