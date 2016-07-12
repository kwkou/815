<properties
	pageTitle="虚拟机上的 SQL Server 概述 | Azure"
	description="开始在 Azure 虚拟机上运行云中的 SQL Server 数据库。基础结构即服务 (IaaS) 模型可让你在 Azure 中运行 SQL Server 工作负荷。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/10/2016"
	wacn.date="06/13/2016"/>

# Azure 虚拟机中的 SQL Server 概述

[Azure 虚拟机上运行的 SQL Server](/home/features/virtual-machines#virtual-machine-SQLserver) 可让你将 SQL Server 数据库托管在云中。例如，你可以将本地数据库迁移到预配置了 Windows Server 2012 R2 和 SQL Server 2014 Enterprise 版的 Azure VM。但是，也可以采用其他许多可能的方案，例如，支持高可用性或混合体系结构并连接到本地网络的多计算机配置。

有关详尽概述，请观看以下视频：[Azure VM is the best platform for SQL Server 2016（Azure VM 是 SQL Server 2016 的最佳平台）](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/Azure-VM-is-the-best-platform-for-SQL-Server-2016)。

## SQL 产品

在 Azure VM 中运行 SQL Server 是在 Azure 中存储关系数据的一个选项。下表汇总了不同的产品。

|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| SQL 产品 | 说明 |
|---:|---|---|
|![Azure 虚拟机中的 SQL Server](./media/virtual-machines-windows-sql-server-iaas-overview/sql-server-virtual-machine.png)|[Azure 虚拟机中的 SQL Server](/home/features/virtual-machines#virtual-machine-SQLserver)|运行 Azure 虚拟机中的 SQL Server。在零售版 SQL Server 上直接管理虚拟机并运行数据库。 |
|![SQL 数据库](./media/virtual-machines-windows-sql-server-iaas-overview/azure-sql-database.png)|[SQL 数据库](/home/features/sql-database/)|使用 SQL 数据库服务来访问和缩放数据库，而无需管理底层基础结构。|
|![SQL 数据仓库](./media/virtual-machines-windows-sql-server-iaas-overview/azure-sql-data-warehouse.png)|[SQL 数据仓库](/home/features/sql-data-warehouse)|使用 Azure SQL 数据仓库来处理大量的关系与非关系数据。以服务形式提供可缩放的数据仓库功能。|
|![SQL Server Stretch Database](./media/virtual-machines-windows-sql-server-iaas-overview/sql-server-stretch-database.png)|[SQL Server Stretch Database](/home/features/sql-server-stretch-database)|将本地事务数据从 Microsoft SQL Server 2016 动态延伸到 Azure。|

>[AZURE.NOTE] 有关 SQL VM 与 SQL 数据库之间的完整比较，请参阅 [Choose a cloud SQL Server option: Azure SQL (PaaS) Database or SQL Server on Azure VMs (IaaS)（选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)）](/documentation/articles/data-management-azure-sql-database-and-sql-server-iaas/)。

## 部署 SQL Server VM

若要在 Azure 中创建 SQL Server 虚拟机，你必须首先获取 Azure 平台订阅。你可以在[购买选项](/pricing/overview/)中购买 Azure 订阅。若要免费试用，请访问 [Azure 试用](/pricing/1rmb-trial/)。

注册订阅后，在 Azure 中部署 SQL Server 虚拟机的最简单方法是[使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。这些映像包括 VM 定价中的 SQL Server 许可。

请务必注意，用于创建和管理 Azure 虚拟机的模型有二种：经典模型和 Resource Manager 模型。Azure 建议大多数新部署使用资源管理器模型。有关详细信息，请参阅 [Understanding Resource Manager deployment and classic deployment（了解 Resource Manager 部署和经典部署）](/documentation/articles/resource-manager-deployment-model/)。每个主题应明确描述其目标模型，除非和本文一样同时适用于经典和 Resource Manager 模型。

## 选择 SQL VM 映像
下表提供了虚拟机库中提供的 SQL Server 映像的矩阵。单击表格中任一链接可创建该版本与操作系统的 VM。

|SQL Server 版本|操作系统|SQL Server 版本|
|---|---|---|
|**SQL Server 2016 RC**|Windows Server 2012 R2|计算|
|**SQL Server 2014 SP1**|Windows Server 2012 R2|Enterprise、Standard、Web、Express|
|**SQL Server 2014**|Windows Server 2012 R2|Enterprise、Standard、Web|
|**SQL Server 2012 SP2**|Windows Server 2012 R2|Enterprise、Standard、Web|
|**SQL Server 2012 SP2**|Windows Server 2012|Enterprise、Standard、Web、Express|
|**SQL Server 2008 R2 SP3**|Windows Server 2008 R2|Enterprise、Standard、Web|
|**SQL Server 2008 R2 SP3**|Windows Server 2012|Express|

>[AZURE.NOTE] 客户体验改善计划 (CEIP) 默认情况下已启用。如果需要，可以在预配虚拟机后自定义或禁用 CEIP。以远程桌面连接到 VM，并运行 **SQL Server 错误和使用报告**实用工具。

除了这些预配置的映像外，你还可以[创建不预装 SQL Server 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)。可以安装你有许可证的 SQL Server 的任何实例。可使用 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)将许可证迁移到 Azure，以便在 Azure 虚拟机中运行 SQL Server。在这种情况下，你只需为与虚拟机关联的 Azure 计算和存储[成本](/home/features/virtual-machines/pricing/)付费。

为了确定 SQL Server 映像的最佳虚拟机配置设置，请查看 [Azure 虚拟机中 SQL Server 的性能最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)。对于生产工作负荷，**DS3** 是 SQL Server Enterprise 版建议的最小虚拟机大小，**DS2** 是标准版建议的最小虚拟机大小。

## 迁移数据

在启动并运行 SQL Server 虚拟机后，你可能想要将现有数据库迁移到虚拟机。有关迁移选项的列表和指导，请参阅 [Migrating a Database to SQL Server on an Azure VM（将数据库迁移到 Azure VM 上的 SQL Server）](/documentation/articles/virtual-machines-windows-migrate-sql/)。

## 高可用性

如果你需要高可用性，请考虑配置 SQL Server 可用性组。这涉及虚拟网络中的多个 Azure VM。

有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)。

## 备份和还原
对于本地数据库，Azure 可以充当用于存储 SQL Server 备份文件的辅助数据中心。有关备份与还原选项的概述，请参阅 [Azure 虚拟机中 SQL Server 的备份和还原](/documentation/articles/virtual-machines-windows-sql-backup-recovery/)。

[SQL Server 备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx) 将 Azure 备份文件存储在 Azure Blob 存储中。[SQL Server 托管备份](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)允许你在 Azure 中计划备份和保留期。这些服务可用于本地 SQL Server 实例或 Azure VM 上运行的 SQL Server。Azure VM 还可以利用 SQL Server 的[自动备份](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup/)和[自动修补](/documentation/articles/virtual-machines-windows-classic-sql-automated-patching/)。

## 后续步骤

首先，[使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

然后，在考虑将 SQL Server 工作负荷移到 Azure VM 时，请参阅性能[最佳实践](/documentation/articles/virtual-machines-windows-sql-performance/)和[迁移方法](/documentation/articles/virtual-machines-windows-migrate-sql/)。

如果你有关于 Azure 虚拟机上 SQL Server 的问题，请参阅 [SQL Server on Azure Virtual Machines FAQ（Azure 虚拟机中的 SQL Server 常见问题）](/documentation/articles/virtual-machines-windows-sql-server-iaas-faq/)。或者将你的看法添加在任何 SQL VM 主题的底部，以便与 Microsoft 和社区互动。

<!---HONumber=Mooncake_0606_2016-->