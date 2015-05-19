<properties 
	pageTitle="Azure 基础结构服务中托管的 SharePoint 场" 
	description="访问介绍如何在 Azure 基础结构服务中设置开发/测试或生产的 SharePoint 2013 场的关键主题。" 
	documentationCenter="" 
	services="virtual-machines"
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""/>

<tags 
wacn.date="05/15/2015"
	ms.service="virtual-machines" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/06/2015" 
	ms.author="josephd"/>

# Azure 基础结构服务中托管的 SharePoint 场

在 Microsoft Azure 基础结构服务中设置你的第一个或下一个开发/测试或生产 SharePoint 场，在其中你可以利用配置的简便性和相关功能快速扩展场以包括关键功能的新增能力或优化。 

## 基本 SharePoint 开发/测试场 

你可以在 Azure 预览版门户中使用 [SharePoint 服务器场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)模板针对面向 Internet 的 SharePoint 网站创建基本开发/测试场。

自动创建的环境由仅限云的 Azure 虚拟网络中的三个服务器组成：分别用于域控制器、SQL Server 和 SharePoint 服务器。

## 高度可用的 SharePoint 开发/测试场

你还可以在 Azure 预览版门户中使用 [SharePoint 服务器场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)模板针对面向 Internet 的 SharePoint 网站创建高可用性 SharePoint 开发/测试场。

在仅限云的 Azure 虚拟网络中自动创建的环境由九个服务器组成：两个用于域控制器、三个用于 SQL Server 群集、两个应用程序层 SharePoint 服务器和两个 Web 层 SharePoint 服务器。

## 混合云开发/测试场

使用[混合云开发/测试环境中的 SharePoint Intranet 场](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)，可以创建模拟的混合云配置，以托管简单的两层 SharePoint 场，你可以从 Internet 上你所在位置使用该场测试 Azure 中托管的 Intranet SharePoint 场。

## 高度可用的 Intranet SharePoint 生产场

借助于[使用 Azure 中的 SQL Server AlwaysOn 可用性组部署 SharePoint 2013](https://msdn.microsoft.com/zh-CN/library/dn275959.aspx)，你可以在 Azure 中构建出生产就绪的高度可用 Intranet SharePoint Server 2013 场。

## 其他资源

[Azure 基础结构服务上的 SharePoint Server](https://msdn.microsoft.com/zh-CN/library/dn275955.aspx)

[在 Azure 基础结构服务上规划 SharePoint 2013](https://msdn.microsoft.com/zh-CN/library/dn275958.aspx)

[适用于 SharePoint 2013 的 Microsoft Azure 体系结构](https://technet.microsoft.com/zh-CN/library/dn635309.aspx)


<!--HONumber=53-->