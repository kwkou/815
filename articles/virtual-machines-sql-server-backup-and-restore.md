<properties
	pageTitle="SQL Server 的备份和恢复 | Azure"
	description="介绍 Azure 虚拟机上运行的 SQL Server 数据库的备份和还原注意事项。"
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar"
	tags="azure-resource-management" />

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/06/2016"
	wacn.date="06/13/2016"/>

# Azure 虚拟机中 SQL Server 的备份和还原

## 概述

备份 SQL Server 数据库中的数据是防止应用程序或用户错误导致数据丢失而采取的策略的重要组成部分。对于 Azure 虚拟机 (VM) 上运行的 SQL Server 同样如此。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

对于 Azure VM 中运行的 SQL Server，可以使用附加的磁盘作为备份文件目标，通过本机备份和还原技术实现此目的。不过，你只能[根据虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)，将有限数量的磁盘附加到 Azure 虚拟机。磁盘管理开销也是一个考虑因素。

从 SQL Server 2014 开始，你可以备份和还原到 Azure Blob 存储。SQL Server 2016 进一步增强了此选项。此外，对于 Azure Blob 存储中存储的数据库文件，SQL Server 2016 提供了一个选项，让你使用 Azure 快照以接近实时的效率进行备份和快速还原。本文将概述这些选项，你可以 [SQL Server Backup and Restore with Azure Blob Storage Service（使用 Azure Blob 存储服务执行 SQL Server 备份和还原）](https://msdn.microsoft.com/zh-cn/library/jj919148.aspx)中找到更多信息。

>[AZURE.NOTE] 有关用于备份极大型数据库的选项的介绍，请参阅[适用于 Azure 虚拟机的多 TB SQL Server 数据库备份策略](http://blogs.msdn.com/b/igorpag/archive/2015/07/28/multi-terabyte-sql-server-database-backup-strategies-for-azure-virtual-machines.aspx)。

下列部分介绍特定于 Azure 虚拟机中支持的不同 SQL Server 版本的信息。

## SQL Server 虚拟机

如果你的 SQL Server 实例在 Azure 虚拟机上运行，数据库文件已驻留在 Azure 中的数据磁盘上。这些磁盘驻留在 Azure Blob 存储中。因此，备份数据库的原因和捕获更改的方式略有不同。请考虑以下代码。

- 你不再需要执行数据库备份以针对硬件或介质故障提供保护，因为 Azure 在 Azure 服务中提供了此保护。

- 你仍需要执行数据库备份以针对用户错误提供保护，或者满足存档目的、法规原因或管理目的。

- 可以直接在 Azure 中存储备份文件。有关详细信息，请参阅以下部分，其中提供了适用于不同版本的 SQL Server 的指导。

## SQL Server 2016 Release Candidate

与 SQL Server 2014 一样，Microsoft SQL Server 2016 Release Candidate (RC3) 也支持[使用 Azure Blob 进行备份和还原](https://msdn.microsoft.com/zh-cn/library/jj919148.aspx)的功能。不过，它还包括以下增强功能：

| 2016 增强功能 | 详细信息 |
|---------------------|-------------------------------|
| **条带化** | 当备份到 Azure Blob 存储时，SQL Server 2016 支持备份到多个 Blob，以便能够备份高达 12.8 TB 的大型数据库。 |
| **快照备份** | 通过使用 Azure 快照，SQL Server 快照备份为使用 Azure Blob 存储服务中存储的数据库文件提供接近实时的备份和更快速的还原。使用此功能可简化备份和还原策略。文件快照备份还支持时间点还原。有关详细信息，请参阅 [Azure 中针对数据库文件的快照备份](https://msdn.microsoft.com/zh-cn/library/mt169363%28v=sql.130%29.aspx)。 |
| **受管理的备份计划** | SQL Server 托管备份到 Azure 现在支持自定义计划。有关详细信息，请参阅 [SQL Server Managed Backup to Azure（SQL Server 托管备份到 Azure）](https://msdn.microsoft.com/zh-cn/library/dn449496.aspx)。 |

有关在使用 Azure Blob 存储时使用 SQL Server 2016 功能的教程，请参阅 [Tutorial: Using the Azure Blob storage service with SQL Server 2016 databases（教程：将 Azure Blob 存储服务用于 SQL Server 2016 数据库）](https://msdn.microsoft.com/zh-cn/library/dn466438.aspx)。

## SQL Server 2014


SQL Server 2014 包括以下增强功能：

1. **备份和还原到 Azure**：

 - 现在，SQL Server Management Studio 支持 SQL Server 备份到 URL。当使用“备份”或“还原”任务或在 SQL Server Management Studio 中使用维护计划向导时，现在可以使用备份到 Azure 的选项。有关详细信息，请参阅 [SQL Server 备份到 URL](https://msdn.microsoft.com/zh-cn/library/jj919148%28v=sql.120%29.aspx)。
 - SQL Server 托管备份到 Azure：允许自动执行备份管理的新功能。这对于针对在 Azure 计算机上运行的 SQL Server 2014 实例自动执行备份管理特别有用。有关详细信息，请参阅 [SQL Server Managed Backup to Azure（SQL Server 托管备份到 Azure）](https://msdn.microsoft.com/zh-cn/library/dn449496%28v=sql.120%29.aspx)。
 - 自动备份：在 Azure 中 SQL Server VM 的所有现有和新数据库上，提供额外的自动化功能来自动启用 SQL Server 托管备份到 Azure。有关详细信息，请参阅[针对 Azure 虚拟机中 SQL Server 的自动备份](/documentation/articles/virtual-machines-windows-classic-sql-automated-backup/)。
 - 有关 SQL Server 2014 备份到 Azure 的所有选项概述，请参阅 [SQL Server Backup and Restore with Azure Blob Storage Service（使用 Azure Blob 存储服务执行 SQL Server 备份和还原）](https://msdn.microsoft.com/zh-cn/library/jj919148%28v=sql.120%29.aspx)。

1. **加密**：SQL Server 2014 支持在创建备份时加密数据。它支持几种加密算法和使用证书或非对称密钥。有关详细信息，请参阅[备份加密](https://msdn.microsoft.com/zh-cn/library/dn449489%28v=sql.120%29.aspx)。

## SQL Server 2012

有关 SQL Server 2012 中 SQL Server 备份和还原的详细信息，请参阅 [SQL Server 数据库的备份和还原 (SQL Server 2012)](https://msdn.microsoft.com/zh-cn/library/ms187048%28v=sql.110%29.aspx)。

从 SQL Server 2012 SP1 累积更新 2 起，你可以备份到 Azure Blob 存储服务以及从该服务中还原。通过此增强功能，可以对在 Azure 虚拟机或本地实例上运行的 SQL Server 备份 SQL Server 数据库。有关详细信息，请参阅[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](https://msdn.microsoft.com/zh-cn/library/jj919148%28v=sql.110%29.aspx)。

使用 Azure Blob 存储服务的一些好处包括能够避开 16 个附加磁盘的限制、易于管理、备份文件直接可用于在 Azure 虚拟机上运行的其他 SQL Server 实例，或者用于本地实例以进行迁移或灾难恢复。有关使用 Azure Blob 存储服务进行 SQL Server 备份的所有好处，请参阅[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](https://msdn.microsoft.com/zh-cn/library/jj919148%28v=sql.110%29.aspx)中的*好处*部分。

有关最佳实践建议和故障排除信息，请参阅[备份和还原最佳实践（Azure Blob 存储服务）](https://msdn.microsoft.com/zh-cn/library/jj919149%28v=sql.110%29.aspx)。

## SQL Server 2008

有关 SQL Server 2008 R2 中的 SQL Server 备份和还原的信息，请参阅[在 SQL Server 中备份和还原数据库 (SQL Server 2008 R2)](https://msdn.microsoft.com/zh-cn/library/ms187048%28v=sql.105%29.aspx)。

有关 SQL Server 2008 中的 SQL Server 备份和还原的信息，请参阅[在 SQL Server 中备份和还原数据库 (SQL Server 2008)](https://msdn.microsoft.com/zh-cn/library/ms187048%28v=sql.100%29.aspx)。

## 后续步骤

如果你在规划 Azure VM 中的 SQL Server 部署，可在以下教程中找到预配指导：[使用 Azure PowerShell 预配 SQL Server 虚拟机 (Resource Manager)](/documentation/articles/virtual-machines-windows-ps-sql-create/)。

尽管备份和还原可用于迁移数据，但是，Azure VM 上的 SQL Server 可能还存在更便捷的数据迁移路径。有关迁移选项和建议的完整讨论，请参阅[将数据库迁移到 Azure VM 上的 SQL Server](/documentation/articles/virtual-machines-windows-migrate-sql/)。

请查看其他[有关在 Azure 虚拟机中运行 SQL Server 的资源](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

<!---HONumber=Mooncake_0606_2016-->