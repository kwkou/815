<!-- not suitable for Mooncake -->

<properties
	pageTitle="部署 SharePoint Server 2013 场 | Azure"
	description="在 Azure 中通过 SQL Server AlwaysOn 可用性组以五个阶段部署高可用性 SharePoint Server 2013 场。"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/17/2015"
	wacn.date=""/>

# 在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典部署模型。

本主题包含有关通过 SQL Server AlwaysOn 可用性组部署仅限 Intranet 的 SharePoint 2013 场的分步说明的链接。该场包含以下计算机：

- 两个 SharePoint Web 服务器
- 两个 SharePoint 应用程序服务器
- 两个数据库服务器
- 一个群集多数节点服务器
- 两个域控制器

这是包含每个服务器的占位符名称的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-overview/workload-spsqlao_05.png)

为每个角色配置两台计算机可确保高可用性。所有虚拟机都在单个区域中。用于特定角色的每个虚拟机组在其自己的可用性集中。

## 材料清单

此基线配置需要以下一组 Azure 服务和组件：

- 九个虚拟机。
- 四个用于域控制器和 SQL Server 的额外数据磁盘。
- 四个可用性集。
- 一个跨界虚拟网络。
- 一个存储帐户。
- 一个 Azure 订阅。

下面是此配置的虚拟机及其默认大小。

项目 | 虚拟机说明 | 库映像 | 默认大小
--- | --- | --- | ---
1\. | 第一个域控制器 | Windows Server 2012 R2 Datacenter | A2
2\. | 第二个域控制器 | Windows Server 2012 R2 Datacenter | A2
3\. | 第一个数据库服务器 | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | A5
4\. | 第二个数据库服务器 | Microsoft SQL Server 2014 Enterprise - Windows Server 2012 R2 | A5
5\. | 群集多数节点 | Windows Server 2012 R2 Datacenter | A1
6\. | 第一个 SharePoint 应用程序服务器 | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4
7\. | 第二个 SharePoint 应用程序服务器 | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4
8\. | 第一个 SharePoint Web 服务器 | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4
9\. | 第二个 SharePoint Web 服务器 | Microsoft SharePoint Server 2013 试用版 - Windows Server 2012 R2 | A4

若要计算此配置的估计成本，请参阅 [Azure 定价计算器](https://azure.microsoft.com/pricing/calculator/)。

1. 在“模块”中，单击“计算”，然后单击“虚拟机”相应次以创建包含九个虚拟机的列表。
2. 对于每个虚拟机，请选择：
	- 所需的区域
	- 对于类型，选择 **Windows**
	- 对于定价层，选择“标准”
	- 上一个表中的默认大小，或者在**实例大小**中选择所需的大小

> [AZURE.NOTE] Azure 定价计算器不包括运行 SQL Server 2014 Enterprise 的两个虚拟机的 SQL Server 许可证的额外成本。有关详细信息，请参阅[虚拟机定价-SQL](/home/features/virtual-machines/#price)。

## 部署阶段

部署此配置分为以下几个阶段：

- [阶段 1：配置 Azure](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1)。创建存储帐户、可用性集和跨界虚拟网络。
- [阶段 2：配置域控制器](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)。创建和配置副本 Active Directory 域服务 (AD DS) 域控制器。
- [阶段 3：配置 SQL Server 基础结构](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase3)。创建并配置 SQL Server 虚拟机、使其准备好用于 SharePoint 并创建群集。
- [阶段 4：配置 SharePoint Server](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4)。创建并配置四个 SharePoint 虚拟机。
- [阶段 5：创建可用性组并添加 SharePoint 数据库](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5)。准备数据库并创建 SQL Server AlwaysOn 可用性组。

这项具有 SQL Server AlwaysOn 可用性组的 SharePoint 部署旨在为[具有 SQL Server AlwaysOn 的 SharePoint 信息图](https://azure.microsoft.com/zh-cn/documentation/infographics/sharepoint-sqlserver-alwayson/)提供补充，并加入了最新的建议。

此配置是针对预定义的体系结构的规范性分阶段指南，用于在 Azure 基础结构服务中创建正常工作、高度可用的 Intranet SharePoint 场。有关在 Azure 中实现 SharePoint 2013 的其他体系结构指南，请参阅 [SharePoint 2013 的 Microsoft Azure 体系结构](https://technet.microsoft.com/zh-cn/library/dn635309.aspx)。

请记住以下几点：

- 如果你是经验丰富的 SharePoint 实施者，请随意调整阶段 3 到阶段 5 的说明并生成最适合你的需求的场。
- 如果你已有现有 Azure 混合云部署，可随意调整或跳过阶段 1 和阶段 2 中的说明，以在相应的子网上托管新的 SharePoint 场。
- 所有服务器都位于 Azure 虚拟网络中的单个子网上。如果要提供等效于子网隔离的附加安全性，则可以使用[网络安全组](/documentation/articles/virtual-networks-nsg)。

若要构建此配置的开发/测试环境或概念证明，请参阅[在混合云中设置 SharePoint Intranet 场用于测试](/documentation/articles/virtual-networks-setup-sharepoint-hybrid-cloud-testing)。

有关 SharePoint 以及 SQL Server AlwaysOn 可用性组的其他信息，请参阅[针对 SharePoint 2013 配置 SQL Server 2012 AlwaysOn 可用性组](https://technet.microsoft.com/zh-cn/library/jj715261.aspx)。

> [AZURE.NOTE] Microsoft 已发布 SharePoint Server 2016 IT Preview。要使此预览版易于安装和测试，可以使用预安装了 SharePoint Server 2016 IT Preview 及其必备组件的 Azure 虚拟机库映像。有关详细信息，请参阅[在 Azure 中测试 SharePoint Server 2016 IT Preview](https://azure.microsoft.com/blog/test-sharepoint-server-2016-it-preview-4/)。

## 后续步骤

- 开始使用[阶段 1](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1) 配置此工作负荷。


<!---HONumber=Mooncake_0411_2016-->