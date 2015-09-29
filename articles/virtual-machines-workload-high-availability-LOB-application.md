<properties 
	pageTitle="Azure 基础结构服务工作负荷：高可用性业务线应用程序" 
	description="了解 Azure 中部署的基于 Web 的业务线应用程序的价值、如何设置开发/测试环境，以及如何部署高可用性生产配置。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines" 
	ms.date="08/11/2015" 
	wacn.date="09/15/2015"/>

# Azure 基础结构服务工作负荷：高可用性业务线应用程序

在 Windows Azure 中设置你的第一个或下一个仅限 Intranet 的业务线应用程序，并利用配置的简便性和相关功能快速扩展应用程序以包括新增能力。
 
借助 Azure 基础结构服务的虚拟机和虚拟网络功能，你可以快速部署并运行可供你组织的用户访问的业务线应用程序。例如，你可以进行此设置。

![](./media/virtual-machines-workload-high-availability-LOB-application/workload-lobapp-phase4.png)
 
由于 Azure 虚拟网络是你的本地网络的扩展（其中所有正确的命名和通信路由已到位），因此你的用户可以就像业务线应用程序的服务器位于本地数据中心内一样以相同的方式访问它。

此配置允许你通过添加新 Azure 虚拟机轻松地扩展应用程序的能力，其中硬件和维护的持续成本将低于在数据中心中运行等效的应用程序。

下一步是设置在 Azure 中托管的开发/测试业务线应用程序。

## 创建在 Azure 中托管的开发/测试业务线应用程序

跨界虚拟网络通过站点到站点 VPN 或 ExpressRoute 连接与本地网络连接。如果你想要创建开发/测试环境，以模拟通过 VPN 连接访问应用程序并执行远程管理的最终配置和试验，请参阅[在混合云中设置基于 Web 的 LOB 应用程序用于测试](/documentation/articles/virtual-networks-setup-lobapp-hybrid-cloud-testing)。

![](./media/virtual-machines-workload-high-availability-LOB-application/CreateLOBAppHybridCloud_3.png)
 
可以使用 [MSDN 订阅](http://azure.microsoft.com/pricing/member-offers/msdn-benefits/)或 [Azure 试用版订阅](http://azure.microsoft.com/pricing/free-trial/)免费创建此开发/测试环境。

下一步是在 Azure 中创建高可用性业务线应用程序。

## 部署在 Azure 中托管的业务线应用程序

Azure 中的高可用性业务线应用程序的基线代表配置如下所示。

![](./media/virtual-machines-workload-high-availability-LOB-application/workload-lobapp-phase4.png)
 
这包括：

- 在 Web 层和数据库层具有两个服务器的仅限 Intranet 业务线应用程序。
- 在群集中包含两个运行 SQL Server 的虚拟机和一个多数节点计算机的 SQL Server AlwaysOn 配置。
- 具有两个副本域控制器的虚拟网络中的 Active Directory 域服务。

有关业务线应用程序的概述，请参阅[业务线应用程序体系结构蓝图](http://msdn.microsoft.com/dn630664)。

### 材料清单

此基线配置需要以下一组 Azure 服务和组件：

- 七个虚拟机
- 四个用于域控制器和运行 SQL Server 的虚拟机的额外数据磁盘
- 三个可用性集
- 一个跨界虚拟网络
- 两个存储帐户

### 部署阶段

若要部署此配置，请使用以下过程：

- 阶段 1：配置 Azure 

	使用 Azure PowerShell 创建存储帐户、可用性集和跨界虚拟网络。有关详细的配置步骤，请参阅[阶段 1](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase1)。

- 阶段 2：配置域控制器

	为虚拟网络配置两个 Active Directory 副本域控制器和 DNS 设置。有关详细的配置步骤，请参阅[阶段 2](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase2)。

- 阶段 3：配置 SQL Server 基础结构。

	创建运行 SQL Server 和群集的虚拟机。有关详细的配置步骤，请参阅[阶段 3](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase3)。

- 阶段 4：配置 web 服务器。

	创建 web 服务器虚拟机并向其添加业务应用程序。有关详细的配置，请参阅[阶段 4](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase4)。

- 阶段 5：配置 SQL Server AlwaysOn 可用性组。

	准备应用程序数据库，创建 SQL Server AlwaysOn 可用性组，然后将应用程序数据库添加到该组中。有关详细的配置步骤，请参阅[阶段 5](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-phase5)。

在配置后，你可以通过向群集添加更多 web 服务器或运行 SQL Server 的虚拟机来轻松扩展此业务线应用程序。

## 其他资源

[在 Azure 中部署高可用性业务线应用程序](/documentation/articles/virtual-machines-workload-high-availability-LOB-application-overview)

[业务线应用程序体系结构蓝图](http://msdn.microsoft.com/dn630664)

[在混合云中设置用于测试且基于 Web 的 LOB 应用程序](/documentation/articles/virtual-networks-setup-lobapp-hybrid-cloud-testing)

[Azure 基础结构服务实施准则](/documentation/articles/virtual-machines-infrastructure-services-implementation-guidelines)

[Azure 基础结构服务工作负荷：SharePoint Server 2013 场](/documentation/articles/virtual-machines-workload-intranet-sharepoint-farm)

<!---HONumber=69-->