<properties
	pageTitle="Azure 中的混合云测试环境 | Azure"
	description="查找介绍如何构建适用于基于 Azure 的混合云的开发/测试或概念证明 IT 专业环境的文章。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/01/2016"
	wacn.date="06/07/2016"/>

# Azure 混合云测试环境

对于开发/测试或概念证明，混合云测试环境使用本地 Internet 连接和公共 IP 地址之一，逐步引导你设置可正常工作的跨界 Azure 虚拟网络 (VNet)。完成后，可以执行应用程序开发和测试、用简化的 IT 工作负荷进行试验，以及测量相对于你在 Internet 上的位置的站点到站点虚拟专用网络 (VPN) 连接的性能。

## 混合云基本配置

[混合云基本配置](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-base/)包括：

- 具有四个虚拟机（域控制器、应用程序服务器、客户端计算机以及运行 Windows server 及路由和远程访问的 VPN 设备）的简化本地网络
- 具有副本域控制器的 Azure 虚拟网络
- 站点到站点 VPN 连接。

## 混合云中的 SharePoint Intranet 场

[混合云测试环境中的 SharePoint Intranet 场](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-sp/)将 SQL Server 2014 服务器和 SharePoint Server 2013 服务器添加到混合云基本配置。这将创建两层 SharePoint 场，你可以从简化的本地网络中的客户端计算机进行访问。

## 混合云中基于 Web 的业务线 (LOB) 应用程序

[混合云测试环境中基于 Web 的 LOB 应用程序](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-lob/)将 SQL Server 2014 服务器和 Internet 信息服务 (IIS) 服务器添加到混合云基本配置。这将创建可在其中部署和测试基于 Web 的分层 LOB 应用程序的基础结构。

## 混合云中的 Office 365 目录同步 (DirSync) 服务器

[混合云测试环境中的 Office 365 DirSync 服务器](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-dirsync/)将 DirSync 服务器添加到混合云基本配置并通过密码同步到试用 Office 365 订阅演示 Office 365 DirSync。

## 模拟的混合云测试环境

对于直接 Internet 连接和公共 IP 地址不容易获得的组织和个人，[模拟的混合云测试环境](/documentation/articles/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/)在单独的 Azure 虚拟网络中构建出简化的本地网络，然后使用 VNet 到 VNet VPN 连接将两个虚拟网络连接在一起。


## 后续步骤

- 了解[实现准则](/documentation/articles/virtual-machines-linux-infrastructure-service-guidelines/)，以便在 Azure 基础结构服务中设计自定义开发/测试或生产部署。

<!---HONumber=Mooncake_0425_2016-->