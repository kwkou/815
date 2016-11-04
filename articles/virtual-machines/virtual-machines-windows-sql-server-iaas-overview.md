<properties
	pageTitle="Azure 虚拟机中的 SQL Server 概述 | Azure"
	description="了解如何在 Azure 虚拟机上运行完整的 SQL Server 版本。获取所有 SQL Server VM 映像和相关内容的直接链接。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="rothja"
	manager="jhubbard"
	editor=""
	tags="azure-service-management"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="08/29/2016"
	wacn.date="10/24/2016"
	ms.author="jroth"/>  


# Azure 虚拟机中的 SQL Server 概述

本主题介绍了在 Azure 虚拟机上运行 SQL Server 的选项，提供了[门户映像链接](#option-1-deploy-a-sql-vm-per-minute-licensing)，同时概述了[常见任务](#manage-your-sql-vm)。

>[AZURE.NOTE] 如果用户已熟悉 SQL Server，只是想了解如何部署 SQL Server VM，则请参阅 [Provision a SQL Server virtual machine in the Azure Portal Preview](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)（在 Azure 门户预览中预配 SQL Server 虚拟机）。

## 概述
你可能是想要将本地 SQL Server 工作负荷移至云的数据库管理员。或者，可能是针对 Azure 应用程序考虑 SQL Server 关系数据库功能的开发人员。在 Azure 虚拟机中运行 SQL Server 工作负荷有何优势？ 以下概述视频将讨论其优势并提供技术概述。

## 评估优势

开始之前，先评估在 Azure VM 上使用 SQL Server 的收获。

将其他工作负荷（例如企业应用程序）移至 Azure 时，最好也将任何依赖型 SQL Server 数据库移至 Azure，以改善性能。不过，在 Azure VM 中托管 SQL Server 可提供其他优势。例如，你会自动获取多个数据中心的访问权限，从而获得全局支持和灾难恢复能力。有关完整的方案和优势列表，请参阅 [Azure VM 产品页上的 SQL Server](/home/features/virtual-machines/#home_vm_overview_info)。

> [AZURE.NOTE] 评估 Azure VM 上的 SQL Server 时，也要查看 Azure 上的其他存储和 SQL 选项，例如 [SQL 数据库](/documentation/articles/sql-database-technical-overview/)、[SQL 数据仓库](/documentation/articles/sql-data-warehouse-overview-what-is/)和 [SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-overview/)。有关详细的比较，请参阅 [Choose a cloud SQL Server option: Azure SQL (PaaS) Database or SQL Server on Azure VMs (IaaS)](/documentation/articles/sql-database-paas-vs-sql-server-iaas/)（选择云 SQL Server 选项：Azure SQL (PaaS) 数据库或 Azure VM 上的 SQL Server (IaaS)）。

当你决定要在 Azure VM 上运行 SQL Server 之后，所做的第一个决策之一即是否使用包含 SQL Server 许可成本的 VM 映像。另一个选项是自带许可 (BYOL)，而只支付 VM 本身。下面两节会介绍这些选项。

## <a name="option-1-deploy-a-sql-vm-per-minute-licensing"></a> 选项 1：部署 SQL VM（每分钟许可）
下表提供了虚拟机库中提供的 SQL Server 映像的矩阵。单击任何链接，即可开始创建具有指定版本和操作系统的新 SQL VM。所有映像都包含 [SQL Server 许可成本](/pricing/details/virtual-machines/)。

[Provision a SQL Server virtual machine in the Azure PowerShell (Classic)](/documentation/articles/virtual-machines-windows-classic-ps-sql-create/)（在 Azure PowerShell（经典）中预配 SQL Server 虚拟机）教程提供了分步指导。另请查看 [Performance best practices for SQL Server VMs](/documentation/articles/virtual-machines-windows-sql-performance/)（SQL Server VM 的性能最佳实践），该文介绍了如何在预配期间选择适当的虚拟机大小和其他可用功能。

|版本|操作系统|版本|
|---|---|---|
|**SQL 2016**|Windows Server 2012 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2016RTMEnterpriseWindowsServer2012R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2016RTMStandardWindowsServer2012R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2016RTMWebWindowsServer2012R2), [Dev](https://portal.azure.cn/#create/Microsoft.SQLServer2016RTMDeveloperWindowsServer2012R2), [Express](https://portal.azure.cn/#create/Microsoft.SQLServer2016RTMExpressWindowsServer2012R2)|
|**SQL 2014 SP1**|Windows Server 2012 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP1EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP1StandardWindowsServer2012R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP1WebWindowsServer2012R2), [Express](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP1ExpressWindowsServer2012R2)|
|**SQL 2014**|Windows Server 2012 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2014EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2014StandardWindowsServer2012R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2014WebWindowsServer2012R2)|
|**SQL 2012 SP3**|Windows Server 2012 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3StandardWindowsServer2012R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3WebWindowsServer2012R2), [Express](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3ExpressWindowsServer2012R2)|
|**SQL 2012 SP2**|Windows Server 2012 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2EnterpriseWindowsServer2012R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2StandardWindowsServer2012R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2WebWindowsServer2012R2)|
|**SQL 2012 SP2**|Windows Server 2012|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2EnterpriseWindowsServer2012), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2StandardWindowsServer2012), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2WebWindowsServer2012), [Express](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP2ExpressWindowsServer2012)|
|**SQL 2008 R2 SP3**|Windows Server 2008 R2|[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3EnterpriseWindowsServer2008R2), [Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3StandardWindowsServer2008R2), [Web](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3WebWindowsServer2008R2)|
|**SQL 2008 R2 SP3**|Windows Server 2012|[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3ExpressWindowsServer2012)|

## 选项 2：部署 SQL VM (BYOL)
另一个选项是自带许可 (BYOL)。在此方案中，你只需支付 VM 费用，SQL Server 许可不需要任何额外的费用。若要使用自己的许可证，请参考下面的 SQL Server 版本和操作系统对照表。在门户中，映像名称带有 **{BYOL}** 前缀。

> [AZURE.IMPORTANT] 若要使用 BYOL VM 映像，必须具有包含 [Azure 上通过软件保障实现的许可移动性](/pricing/license-mobility/)的企业协议。此外，还需要有所要使用的 SQL Server 版本的有效许可证。必须在预配 VM 的 **10** 天内[向 Microsoft 提供必要的 BYOL 信息](http://d36cz9buwru1tt.cloudfront.net/License_Mobility_Customer_Verification_Guide.pdf)。

可以遵循[预配教程](/documentation/articles/virtual-machines-windows-classic-ps-sql-create/)中的指导，但必须使用以下 **BYOL** 映像选项之一。另请查看 [Performance best practices for SQL Server VMs](/documentation/articles/virtual-machines-windows-sql-performance/)（SQL Server VM 的性能最佳实践），该文介绍了如何在预配期间选择适当的虚拟机大小和其他可用功能。

|版本|操作系统|版本|
|---|---|---|
|**SQL Server 2016**|Windows Server 2012 R2|[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2016RTMStandardWindowsServer2012R2), [Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2016RTMStandardWindowsServer2012R2)|
|**SQL Server 2014 SP1**|Windows Server 2012 R2|[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2014SP1EnterpriseWindowsServer2012R2), [Standard BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2014SP1StandardWindowsServer2012R2)|
|**SQL Server 2012 SP2**|Windows Server 2012 R2|[Enterprise BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2012SP3EnterpriseWindowsServer2012R2), [Standard  BYOL](https://portal.azure.cn/#create/Microsoft.BYOLSQLServer2012SP3StandardWindowsServer2012R2)|

## <a name="manage-your-sql-vm"></a> 管理 SQL VM
预配 SQL Server VM 之后，有几项可选的管理任务。在某些方面，你完全可以像在本地一样配置和管理 SQL Server。但某些任务是 Azure 特有的。下列各节重点介绍上述某些领域并提供详细信息链接。

### 迁移数据

如果已有数据库，你会想要将该数据库移至新预配的 SQL VM。有关迁移选项的列表和指导，请参阅 [Migrating a Database to SQL Server on an Azure VM](/documentation/articles/virtual-machines-windows-migrate-sql/)（将数据库迁移到 Azure VM 上的 SQL Server）。

### 配置高可用性

如果你需要高可用性，请考虑配置 SQL Server 可用性组。这涉及虚拟网络中的多个 Azure VM。Azure 门户预览有一个模板，为用户设置了此配置。有关详细信息，请参阅 [Configure an AlwaysOn availability group in Classic virtual machines](/documentation/articles/virtual-machines-windows-classic-portal-sql-alwayson-availability-groups/)（在经典虚拟机中配置 AlwaysOn 可用性组）。

有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)。

### 备份数据
可以手动创建备份，将数据库备份到 blob 存储。你也可以手动使用此技术。有关详细信息，请参阅 [Use Azure Storage for SQL Server Backup and Restore](/documentation/articles/virtual-machines-windows-use-storage-sql-server-backup-restore/)（使用 Azure 存储进行 SQL Server 备份和还原）。有关所有备份和还原选项的概述，请参阅 [Backup and Restore for SQL Server in Azure Virtual Machines](/documentation/articles/virtual-machines-windows-sql-backup-recovery/)（Azure 虚拟机中 SQL Server 的备份和还原）。

### 自动更新
Azure VM 可以使用[自动修补](/documentation/articles/virtual-machines-windows-classic-sql-automated-patching/)来安排维护时段，以便自动安装重要的 Windows 和 SQL Server 更新。

### 客户体验改善计划 (CEIP)
客户体验改善计划 (CEIP) 默认情况下已启用。这不是一项管理任务，除非你想在预配之后禁用 CEIP。你可以通过远程桌面连接到 VM，以自定义或禁用 CEIP。然后运行 **SQL Server 错误和使用情况报告**实用工具。请按照说明禁用报告功能。

## 后续步骤

其他问题？ 请先参阅 [Azure 虚拟机中的 SQL Server 常见问题解答](/documentation/articles/virtual-machines-windows-sql-server-iaas-faq/)。同时将问题或看法添加到任何 SQL VM 主题的底部，以便与 Azure.cn 和社区互动。

<!---HONumber=Mooncake_1017_2016-->