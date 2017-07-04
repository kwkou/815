<properties
    pageTitle="Azure 虚拟机中的 SQL Server 概述 | Azure"
    description="了解如何在 Azure 虚拟机上运行完整的 SQL Server 版本。获取所有 SQL Server VM 映像和相关内容的直接链接。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="rothja"
    manager="jhubbard"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="c505089e-6bbf-4d14-af0e-dd39a1872767"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="vm-windows-sql-server"
    ms.workload="infrastructure-services"
    ms.date="01/09/2017"
    wacn.date="02/24/2017"
    ms.author="jroth" />  


# Azure 虚拟机中的 SQL Server 概述
本主题介绍了在 Azure 虚拟机 \(VM\) 上运行 SQL Server 的选项，提供了[门户映像链接](#option-1-create-a-sql-vm-with-per-minute-licensing)，同时概述了[常见任务](#manage-your-sql-vm)。

> [AZURE.NOTE]
如果用户已熟悉 SQL Server，只是想了解如何部署 SQL Server VM，则请参阅[在 Azure 门户中预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)。
> 
> 

## 概述
如果是数据库管理员或开发人员，Azure 虚拟机能够将本地 SQL Server 工作负荷和应用程序迁移到云。下面的视频是关于 SQL Server Azure VM 的技术概述。

## 方案
选择用在 Azure 中托管数据有诸多理由。将应用程序移到 Azure 后，移动数据的性能也有改善。而且还有其他方面的好处。会自动获取多个数据中心的访问权限，从而获得全局支持和灾难恢复能力。并且数据持久保存、高度安全。

可用使用“在 Azure VM 中运行 SQL Server”选项在 Azure 中存储关系数据。它对于几个方案来说是不错的选择。例如，你可能想要配置与本地 SQL Server 计算机高度相似的 Azure VM。或者可能想要在同一数据库服务器上运行其他应用程序和服务。有两个主要资源，可帮助用户仔细考虑更多方案和注意事项：

* [Azure 虚拟机上的 SQL Server](/home/features/virtual-machines/#home_vm_overview_info) 概述了在 Azure VM 上使用 SQL Server 的最佳方案。
* [选择云 SQL Server 选项：Azure SQL \(PaaS\) 数据库或 Azure VM 上的 SQL Server \(IaaS\)](/documentation/articles/sql-database-paas-vs-sql-server-iaas/) 介绍了 SQL 数据库与运行于 VM 上的 SQL Server 之间的详细比较。

## 创建新的 SQL VM
以下部分提供了直接链接，可链接到 Azure 门户的 SQL Server 虚拟机库映像。

有关教程中此过程的分步指导，请参阅[在 Azure 门户中预配 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-portal-sql-server-provision/)。另请查看 [Performance best practices for SQL Server VMs](/documentation/articles/virtual-machines-windows-sql-performance/)（SQL Server VM 的性能最佳实践），该文介绍了如何在预配期间选择适当的虚拟机大小和其他可用功能。

## <a name="option-1-create-a-sql-vm-with-per-minute-licensing"></a> 选项 1：使用每分钟许可创建 SQL VM
下表提供了虚拟机库中提供的最新 SQL Server 映像的矩阵。单击任何链接，即可开始创建具有指定版本和操作系统的新 SQL VM。

| 版本 | 操作系统 | 版本 |
| --- | --- | --- |
| **SQL Server 2016 SP1** |Windows Server 2016 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP1EnterpriseWindowsServer2016)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP1StandardWindowsServer2016)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP1WebWindowsServer2016)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP1ExpressWindowsServer2016)、[Developer](https://portal.azure.cn/#create/Microsoft.SQLServer2016SP1DeveloperWindowsServer2016) |
| **SQL Server 2014 SP2** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2EnterpriseWindowsServer2012R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2StandardWindowsServer2012R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2WebWindowsServer2012R2)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2014SP2ExpressWindowsServer2012R2) |
| **SQL Server 2012 SP3** |Windows Server 2012 R2 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3EnterpriseWindowsServer2012R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3StandardWindowsServer2012R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3WebWindowsServer2012R2)、[Express](https://portal.azure.cn/#create/Microsoft.SQLServer2012SP3ExpressWindowsServer2012R2) |
| **SQL Server 2008 R2 SP3** |Windows Server 2008 R2 |[Enterprise](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3EnterpriseWindowsServer2008R2)、[Standard](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3StandardWindowsServer2008R2)、[Web](https://portal.azure.cn/#create/Microsoft.SQLServer2008R2SP3WebWindowsServer2008R2) |

除了此列表，也可以使用 SQL Server 版本和操作系统的其他组合。在 Azure 门户中通过应用商店搜索查找其他映像。

## <a name="manage-your-sql-vm"></a> 管理 SQL VM
预配 SQL Server VM 之后，有几项可选的管理任务。在许多方面，完全可以像管理本地 SQL Server 实例一样配置和管理 SQL Server。但某些任务是特定于 Azure 的。下列各节重点介绍上述某些领域并提供详细信息链接。

### 连接到 VM
其中一个最基本的管理步骤是，通过工具连接到 SQL Server VM，如 SQL Server Management Studio \(SSMS\)。有关如何连接到新 SQL Server VM 的说明，请参阅[连接到 Azure 上的 SQL Server 虚拟机](/documentation/articles/virtual-machines-windows-sql-connect/)。

### 迁移数据
如果已有数据库，你会想要将该数据库移至新预配的 SQL VM。有关迁移选项的列表和指导，请参阅 [Migrating a Database to SQL Server on an Azure VM](/documentation/articles/virtual-machines-windows-migrate-sql/)（将数据库迁移到 Azure VM 上的 SQL Server）。

### 配置高可用性
如果你需要高可用性，请考虑配置 SQL Server 可用性组。这涉及虚拟网络中的多个 Azure VM。如果想要手动配置可用性组和关联的侦听器，请参阅[在 Azure VM 中配置 AlwaysOn 可用性组](/documentation/articles/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/)。

有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-windows-sql-high-availability-dr/)。

### 备份数据
Azure VM 可以利用[自动备份](/documentation/articles/virtual-machines-windows-sql-automated-backup/)，定期创建数据库到 Blob 存储的备份。你也可以手动使用此技术。有关详细信息，请参阅 [Use Azure Storage for SQL Server Backup and Restore](/documentation/articles/virtual-machines-windows-use-storage-sql-server-backup-restore/)（使用 Azure 存储进行 SQL Server 备份和还原）。有关所有备份和还原选项的概述，请参阅 [Backup and Restore for SQL Server in Azure Virtual Machines](/documentation/articles/virtual-machines-windows-sql-backup-recovery/)（Azure 虚拟机中 SQL Server 的备份和还原）。

### 自动更新
Azure VM 可以使用[自动修补](/documentation/articles/virtual-machines-windows-sql-automated-patching/)来安排维护时段，以便自动安装重要的 Windows 和 SQL Server 更新。

### 客户体验改善计划 \(CEIP\)
客户体验改善计划 \(CEIP\) 默认情况下已启用。它定期将报表发送给 Microsoft，以帮助改进 SQL Server。CEIP 不要求管理任务，除非想在预配后禁用它。你可以通过远程桌面连接到 VM，以自定义或禁用 CEIP。然后运行 **SQL Server 错误和使用情况报告**实用工具。请按照说明禁用报告功能。

有关详细信息，请参阅[接受许可条款](https://msdn.microsoft.com/zh-cn/library/ms143343.aspx)主题的 CEIP 部分。

## 后续步骤
有关定价问题，请参阅[定价](/pricing/details/virtual-machines/)。在**VM 类型**列表中选择 SQL Server 的目标版本。然后查看不同大小的虚拟机的价格。

其他问题？ 请先参阅 [Azure 虚拟机中的 SQL Server 常见问题解答](/documentation/articles/virtual-machines-windows-sql-server-iaas-faq/)。同时将问题或看法添加到任何 SQL VM 主题的底部，以便与 Azure.cn 和社区互动。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->