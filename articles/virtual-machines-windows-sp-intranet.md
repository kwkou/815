<properties
	pageTitle="Azure 中的 SharePoint Server 2013 场 | Microsoft Azure"
	description="了解 Azure 中的 SharePoint Server 2013 场的价值、设置测试环境，以及部署高可用性配置。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/17/2015"
	wacn.date=""/>

# Azure 基础结构服务工作负荷：Intranet SharePoint 场

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典部署模型。

在 Microsoft Azure 中设置你的第一个或下一个 SharePoint 场，并利用配置的简便性和相关功能快速扩展场以包括关键功能的新增能力或优化。许多 SharePoint 场从标准的高度可用的三层配置扩展为可能具有十几个或更多服务器的场，该场针对性能或单独的角色进行了优化，如分布式缓存或搜索。

借助 Azure 基础结构服务的虚拟机和虚拟网络功能，你可以快速部署并运行以透明方式连接到你的本地网络的 SharePoint 场。例如，可以设置以下内容：

![](./media/virtual-machines-workload-intranet-sharepoint-farm/workload-spsqlao.png)

由于 Azure 虚拟网络是你的本地网络的扩展（其中所有正确的命名和通信路由已到位），因此你的用户可以就像它位于本地数据中心内一样以相同的方式访问它。

此配置允许你通过添加新 Azure 虚拟机轻松地扩展 SharePoint 场，其中硬件和维护的持续成本将低于在数据中心中运行等效的场。

在 Azure 基础结构服务中托管 Intranet SharePoint 场是业务线应用程序的一个示例。有关概述，请参阅[业务线应用程序体系结构蓝图](http://msdn.microsoft.com/dn630664)。

下一步是设置 Azure 中托管的开发/测试 Intranet SharePoint 场。

> [AZURE.NOTE]Microsoft 已发布 SharePoint Server 2016 IT Preview。要使此预览版易于安装和测试，可以使用预安装了 SharePoint Server 2016 IT Preview 及其必备组件的 Azure 虚拟机库映像。有关详细信息，请参阅[在 Azure 中测试 SharePoint Server 2016 IT Preview](http://azure.microsoft.com/blog/test-sharepoint-server-2016-it-preview-4/)。

## 创建 Azure 中托管的开发/测试 Intranet SharePoint 场

为 Azure 中托管的 SharePoint 场创建开发/测试环境有两个选择：

- 仅限云的虚拟网络
- 跨界虚拟网络

可以使用 [Visual Studio 订阅](http://azure.microsoft.com/pricing/member-offers/msdn-benefits/)或 [Azure 试用版订阅](/pricing/1rmb-trial/)免费创建这些开发/测试环境。

### 仅限云的虚拟网络

仅限云的虚拟网络未连接到本地网络。如果你只想快速创建基本或高可用性 SharePoint 场，请参阅[创建 SharePoint Server 场](/documentation/articles/virtual-machines-sharepoint-farm-azure-preview)。以下示例显示了基本 SharePoint 场配置。

![](./media/virtual-machines-workload-intranet-sharepoint-farm/Non-HAFarm.png)

### 跨界虚拟网络

跨界虚拟网络通过站点到站点 VPN 或 ExpressRoute 连接与本地网络连接。如果你想要创建开发/测试环境，以模拟通过 VPN 连接访问 SharePoint Server 并执行远程管理的最终配置和试验，请参阅[在混合云中设置 SharePoint Intranet 场用于测试](../virtual-network/virtual-networks-setup-sharepoint-hybrid-cloud-testing.md)。

![](./media/virtual-machines-workload-intranet-sharepoint-farm/CreateSPFarmHybridCloud.png)

下一步是在 Azure 中创建高可用性 Intranet SharePoint 场。

## 部署 Azure 中托管的 Intranet SharePoint 场

正常运行的高可用性 Intranet SharePoint 场的基线代表配置如下所示：

![](./media/virtual-machines-workload-intranet-sharepoint-farm/workload-spsqlao.png)

这包括：

- 在 Web 层、应用程序层和数据库层具有两个服务器的 Intranet SharePoint 场。
- 在群集中包含两个 SQL Server 和一个多数节点计算机的 SQL Server AlwaysOn 可用性组配置。
- 本地 Active Directory 域的两个副本域控制器。

若要以信息图形式查看此配置，请参阅[具有 SQL Server AlwaysOn 的 SharePoint](http://go.microsoft.com/fwlink/?LinkId=394788)。

若要部署此配置，请使用以下过程：

- 阶段 1：配置 Azure。

	使用 Azure PowerShell 创建存储帐户、可用性集和跨界虚拟网络。有关详细的配置步骤，请参阅[阶段 1](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase1)。

- 阶段 2：配置域控制器。

	为虚拟网络配置两个 Active Directory 副本域控制器和 DNS 设置。有关详细的配置步骤，请参阅[阶段 2](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase2)。

- 阶段 3：配置 SQL Server 基础结构。

	准备要用于 SharePoint 的 SQL Server 虚拟机并创建 SQL Server 群集。有关详细的配置步骤，请参阅[阶段 3](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase3)。

- 阶段 4：配置 SharePoint Server。

	为新的 SharePoint 场配置四个 SharePoint 虚拟机。有关详细的配置，请参阅[阶段 4](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase4)。

- 阶段 5：创建 AlwaysOn 可用性组。

	准备 SharePoint 数据库，创建 AlwaysOn 可用性组，然后将 SharePoint 数据库添加到该组中。有关详细的配置步骤，请参阅[阶段 5](/documentation/articles/virtual-machines-workload-intranet-sharepoint-phase5)。

配置后，你可以按照[适用于 SharePoint 2013 的 Microsoft Azure 体系结构](http://technet.microsoft.com/library/dn635309.aspx)中的指南扩展此 SharePoint 场。

## 后续步骤

- 在深入了解配置之前，获取生产工作负荷的[概述](/documentation/articles/virtual-machines-workload-intranet-sharepoint-overview)。

<!---HONumber=Mooncake_0104_2016-->