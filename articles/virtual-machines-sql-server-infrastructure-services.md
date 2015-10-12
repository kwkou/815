<properties 
	pageTitle="Azure 虚拟机中的 SQL Server 概述" 
	description="本文概述了 Azure IaaS 虚拟机上托管的 SQL Server。这包括深度内容的链接。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="rothja" 
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.date="08/18/2015"
	wacn.date="09/18/2015"/>

# Azure 虚拟机中的 SQL Server 概述

## 概述
你可以使用各种配置（从单一数据库服务器到多计算机配置）通过 AlwaysOn 可用性组和 Azure 虚拟网络托管 [Azure 虚拟机中的 SQL Server](/documentation/services/virtual-machines/sql-server/)。本主题尝试指点一些可帮助你开始在 Azure 虚拟机中运行 SQL Server 的最佳资源。

> [AZURE.NOTE]在 Azure VM 中运行 SQL Server 是在 Azure 中存储关系数据的一个选项。此外，还可以使用 Azure SQL 数据库服务。
<!--For more information, see [Understanding Azure SQL Database and SQL Server in Azure VMs][sqldbcompared].-->
 
## 在单个 VM 上部署 SQL Server 实例

在 Azure 中部署 SQL Server 虚拟机的最简单方法是[在 Azure 管理门户中预配 SQL Server 计算机库映像](/documentation/articles/virtual-machines-provision-sql-server)。这些映像包括 VM 定价中的 SQL Server 许可。

但是，你也可以[创建不预装 SQL Server 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-tutorial)。可以安装你有许可证的 SQL Server 的任何实例。

在这些早期预配和配置阶段，常见的任务包括：

- [查看 Azure VM 中 SQL Server 的性能最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)
- [查看 Azure VM 中 SQL Server 的安全最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133147.aspx)
- [设置连接](/documentation/articles/virtual-machines-sql-server-connectivity)

## 使用多个 VM 部署高可用性配置

可以使用 SQL Server AlwaysOn 可用性组，实现 SQL Server 高可用性。这涉及虚拟网络中的多个 Azure VM。Azure 预览版门户有一个模板为你设置了此配置。有关详细信息，请参阅 [Microsoft Azure 门户库中的 SQL Server AlwaysOn 产品](http://blogs.technet.com/b/dataplatforminsider/archive/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery.aspx)。

如果你想要手动配置可用性组和关联的侦听器，请参阅以下文章：

- [在 Azure 中配置 AlwaysOn 可用性组 (GUI)](/documentation/articles/virtual-machines-sql-server-alwayson-availability-groups-gui)
- [在 Azure 中配置 AlwaysOn 可用性组的 ILB 侦听器](/documentation/articles/virtual-machines-sql-server-configure-ilb-alwayson-availability-group-listener)
- [将本地 AlwaysOn 可用性组扩展到 Azure](/documentation/articles/virtual-machines-sql-server-extend-on-premises-alwayson-availability-groups)

有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions)。

## 在 Azure 中运行商业智能、数据仓库和 OLTP 工作负荷   
你可以在 Azure 虚拟机上运行常见的 SQL Server 工作负荷。SQL Server 有多个经过优化的虚拟机映像在库中提供。这包括以下产品的映像：

- [商业智能](https://msdn.microsoft.com/zh-cn/library/azure/jj992719.aspx)
- [数据仓库](https://msdn.microsoft.com/zh-cn/library/azure/dn387396.aspx)
- [OLTP](https://msdn.microsoft.com/zh-cn/library/azure/dn387396.aspx)

## 迁移数据

在启动并运行 SQL Server 虚拟机后，你可能想要将现有数据库迁移到虚拟机。你可以采用多种方法，但 SQL Server Management Studio 中的部署向导适合大多数方案。有关这些方案的讨论以及向导教程，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-migrate-onpremises-database)。

## 备份和还原
对于本地数据库，Azure 可以充当用于存储 SQL Server 备份文件的辅助数据中心。有关备份与还原选项的概述，请参阅 [Azure 虚拟机中 SQL Server 的备份和还原](/documentation/articles/virtual-machines-sql-server-backup-and-restore)。

[SQL Server 备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx) 将 Azure 备份文件存储在 Azure Blob 存储中。[SQL Server 托管备份](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)允许你在 Azure 中计划备份和保留期。这些服务可用于本地 SQL Server 实例或 Azure VM 上运行的 SQL Server。Azure VM 还可以利用 SQL Server 的[自动备份](/documentation/articles/virtual-machines-sql-server-automated-backup)和[自动修补](/documentation/articles/virtual-machines-sql-server-automated-patching)。

<!---HONumber=70-->