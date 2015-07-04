<properties pageTitle="在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint" description="你在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint 可以分为五个阶段。" documentationCenter="" services="virtual-machines" authors="JoeDavies-MSFT" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.date="05/05/2015" wacn.date="06/26/2015"/>

# 在 Azure 中通过 SQL Server AlwaysOn 可用性组部署 SharePoint

本主题包含有关在 Azure 基础结构服务中通过 SQL Server AlwaysOn 可用性组部署仅限 Intranet 的 SharePoint 2013 场的分步说明的链接。该场包含以下计算机：

- 两个 SharePoint Web 服务器
- 两个 SharePoint 应用程序服务器
- 两个数据库服务器
- 一个群集多数节点服务器
- 两个域控制器

这是包含每个服务器的占位符名称的配置。

![](./media/virtual-machines-workload-intranet-sharepoint-overview/workload-spsqlao_05.png)
 
为每个角色配置两台计算机可确保高可用性。所有虚拟机都在单个区域中。用于特定角色的每个虚拟机组在其自己的可用性集中。

部署此配置分为以下几个阶段：

- [阶段 1：配置 Azure](virtual-machines-workload-intranet-sharepoint-phase1)。创建存储帐户、云服务和跨界虚拟网络。
- [阶段 2：配置域控制器](virtual-machines-workload-intranet-sharepoint-phase2)。创建和配置副本 Active Directory 域服务 (AD DS) 域控制器。
- [阶段 3：配置 SQL Server 基础结构](virtual-machines-workload-intranet-sharepoint-phase3)。创建并配置 SQL Server 虚拟机、使其准备好用于 SharePoint 并创建群集。
- [阶段 4：配置 SharePoint Server](virtual-machines-workload-intranet-sharepoint-phase4)。创建并配置四个 SharePoint 虚拟机。
- [阶段 5：创建可用性组并添加 SharePoint 数据库](virtual-machines-workload-intranet-sharepoint-phase5)。准备数据库并创建 SQL Server AlwaysOn 可用性组。

此配置是针对预定义的体系结构的规范性分阶段指南，用于在 Azure 基础结构服务中创建正常工作、高度可用的 Intranet SharePoint 场。有关在 Azure 中实现 SharePoint 2013 的其他体系结构指南，请参阅[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-CN/library/dn635309.aspx)。

请记住以下几点：

- 如果你是经验丰富的 SharePoint 实施者，请随意调整阶段 3 到阶段 5 的说明并生成最适合你的需求的场。 
- 如果你已有现有 Azure 混合云实现，随意调整或跳过阶段 1 和阶段 2 中的说明，以在相应的子网上托管新的 SharePoint 场。
- 所有服务器都位于 Azure 虚拟网络中的单个子网上。如果要提供等效于子网隔离的附加安全性，则可以使用[网络安全组](https://msdn.microsoft.com/zh-CN/library/azure/dn848316.aspx)。

若要构建此配置的开发/测试环境或概念证明，请参阅[在混合云中设置 SharePoint Intranet 场用于测试](virtual-networks-setup-sharepoint-hybrid-cloud-testing)。

有关 SharePoint 以及 SQL Server AlwaysOn 可用性组的其他信息，请参阅[针对 SharePoint 2013 配置 SQL Server 2012 AlwaysOn 可用性组](https://technet.microsoft.com/zh-CN/library/jj715261.aspx)。

## 后续步骤

若要开始配置此工作负荷，请转到[阶段 1：配置 Azure](virtual-machines-workload-intranet-sharepoint-phase1)。


## 其他资源
[适用于 SharePoint 2013 的 Windows Azure 体系结构](https://technet.microsoft.com/zh-CN/library/dn635309.aspx)

[Azure 基础结构服务中托管的 SharePoint 场](virtual-machines-sharepoint-infrastructure-services)

[Azure 基础结构服务实现准则](virtual-machines-infrastructure-services-implementation-guidelines)

<!---HONumber=61-->