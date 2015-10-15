<properties 
	pageTitle="Azure 虚拟机中的 SQL Server 数据仓库和事务工作负荷"
	description="介绍 Azure 中用于数据仓库和 OLTP 工作负荷的，经过优化的预配置 SQL Server 虚拟机映像。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />
<tags 
	ms.service="virtual-machines"
	ms.date="08/19/2015"
	wacn.date="09/18/2015" />

# Azure 虚拟机中的 SQL Server 数据仓库和事务工作负荷

若要在 Azure 虚拟机中使用 SQL Server 数据仓库或事务工作负荷，我们建议在 Azure 虚拟机库中使用一个预配置的虚拟机映像。这些映像已根据 [Azure 虚拟机中 SQL Server 的性能最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)中的建议进行优化。

本文重点介绍如何在 Azure 虚拟机上运行这些工作负荷（也称为基础结构即服务，或 IaaS）。你也可以在 Azure 中以服务的形式运行数据仓库和事务工作负荷。有关详细信息，请参阅 [Azure SQL 数据仓库预览版](http://azure.microsoft.com/documentation/services/sql-data-warehouse/)和 [Azure SQL 数据库](http://azure.microsoft.com/documentation/services/sql-database/)。

## 提供了哪些预配置的 VM 映像？
<!--
Azure VM 库中提供了以下预配置的 VM 映像：

- [针对 Windows Server 2012 R2 上的事务工作负荷优化的 SQL Server 2014 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2014fortransactionalworkloadswindowsserver2012r2/)

- [针对 Windows Server 2012 R2 上的数据仓库优化的 SQL Server 2014 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2014datawarehousingwindowsserver2012r2/)

- [针对 Windows Server 2012 R2 上的事务工作负荷优化的 SQL Server 2012 SP2 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2012sp2fortransactionalworkloadswindowsserver2012r2)

- [针对 Windows Server 2012 R2 上的数据仓库工作负荷优化的 SQL Server 2012 SP2 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2012sp2datawarehousingworkloadswindowsserver2012r2)

- [针对 Windows Server 2012 上的事务工作负荷优化的 SQL Server 2012 SP2 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2012sp2fortransactionalworkloadswindowsserver2012/)

- [针对 Windows Server 2012 上的数据仓库优化的 SQL Server 2012 SP2 Enterprise](http://azure.microsoft.com/marketplace/partners/microsoft/sqlserver2012sp2datawarehousingworkloadswindowsserver2012/)
-->
目前，我们在最多允许附加 16 个数据磁盘以提供最高吞吐量（或聚合带宽）的 VM 实例上支持这些映像。这些实例为标准层 A4、A7、A8、A9、D4、D13、D14、F3、G4 和 G5 及基本层 A4。有关大小和选项的更多详细信息，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-size-specs)。

>[AZURE.NOTE]在 2014 年 9 月以前的版本上，可以使用以前的事务和 DW 库映像。但是，这些映像需要你附加数据磁盘才可供使用。建议使用上面列出的较新映像，因为它们在预配后即可供使用。

## 使用事务/DW 映像从库预配 SQL VM

1. 登录到 [Azure 管理门户](http://manage.windowsazure.com/)。

1. 单击左窗格中 Azure 菜单项中的“虚拟机”。

1. 单击左下角的“新建”，然后依次单击“计算”、“虚拟机”、“从库中”。

1. 在“虚拟机映像选择”页上，为事务或数据仓库映像选择一个 SQL Server。

	![Azure VM 库](./media/virtual-machines-sql-server-dw-and-oltp-workloads/IC814362.png)

1. 在“虚拟机配置”页上的“大小”选项中，从支持的大小中进行选择。

	![Azure VM 库配置](./media/virtual-machines-sql-server-dw-and-oltp-workloads/IC814363.png)

	>[AZURE.NOTE]当前仅支持标准层 A4、A7、A8、A9、D4、D13、D14、G3、G4 和 G5 及基本层 A4。尝试预配不受支持的 VM 大小将会失败。

1. 等待预配完成。在等待过程中，可以在虚拟机页上查看预配状态（如下图中所示）。预配完成后，状态将为具有选中标记的“正在运行”。

	![Azure VM 库状态](./media/virtual-machines-sql-server-dw-and-oltp-workloads/IC814364.png)

## 使用 PowerShell 创建事务/DW 映像

也可以使用 PowerShell cmdlet **New-AzureQuickVM** 创建 VM。必须将云服务名称、VM 名称、映像名称、管理员用户名和密码及类似信息作为参数传递。获取映像名称的简单方法是使用 **Get-AzureVMImage** 列出所有可用的 VM 映像。

例如，以下 PowerShell 命令将返回与上一部分中列表内的映像标签“针对 Windows Server 2012 R2 上的数据仓库工作负荷优化的 SQL Server 2012 SP2 Enterprise”匹配的最新映像。

	(Get-AzureVMImage | where {$_.Label -like "SQL Server 2012 SP2 Enterprise Optimized for DataWarehousing Workloads on Windows Server 2012 R2"} | sort PublishedDate -Descending)[0].ImageName

有关使用 PowerShell 创建映像的详细信息，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)。

## 事务/DW 映像中包含的特定配置

对映像的优化是根据 [Azure 虚拟机中 SQL Server 的性能最佳实践](https://msdn.microsoft.com/zh-cn/library/azure/dn133149.aspx)做出的。具体而言，这些映像的配置已经过以下方面的优化。

>[AZURE.NOTE]如果你自带许可证并从头开始创建数据仓库或事务虚拟机，则你可以根据性能文章和以下预配置库映像中的优化示例做出你自己的优化。

### 磁盘配置


|---|---|
|附加的数据磁盘数|15|
|存储空间|两个存储池：<br/>-- 1 个数据池，其中包含 12 个数据磁盘，固定大小为 12 TB，柱数 = 12<br/>-- 1 个日志池，其中包含 3 个数据磁盘，固定大小为 3 TB，柱数 = 3<br/><br/>剩下一个数据磁盘可让用户附加并确定使用情况。<br/><br/>**DW**：条带大小 = 256 KB<br/>**事务**：条带大小 = 64 KB|
|磁盘大小、缓存、分配大小|每个为 1 TB，HostCache=无，NTFS 分配单元大小 = 64 KB|

### SQL Server 配置

|---|---|
|启动参数|-T1117，当数据库需要自动增长时帮助将数据文件保持在相同的大小<br/><br/>-T1118，帮助实现 tempdb 可伸缩性（有关详细信息，请参阅 [SQL Server（2005 和 2008）跟踪标志 1118 (-T1118) 的用法](http://blogs.msdn.com/b/psssql/archive/2008/12/17/sql-server-2005-and-2008-trace-flag-1118-t1118-usage.aspx?WT.mc_id=Blog_SQL_Announce_Announce)。）|
|恢复模式|**DW**：对于模型数据库，使用 ALTER DATABASE 设置为 SIMPLE<br/>**事务**：无变化|
|设置默认位置|将 SQL Server 错误日志和跟踪文件目录移至数据磁盘|
|数据库的默认位置|系统数据库已移至数据磁盘。<br/><br/>用于创建用户数据库的位置已更改为数据磁盘。|
|即时文件初始化|Enabled|
|在内存中锁定页面|已启用（有关详细信息，请参阅[启用在内存中锁定页面的选项 (Windows)）](https://msdn.microsoft.com/zh-cn/library/ms190730.aspx)。|

## 常见问题

- 优化的映像与未优化的映像之间有价格差异吗？

	没有。优化的映像遵循相同的定价模式（在[此处](/pricing/details/virtual-machines/)了解详细信息），不收取额外的费用。请注意，VM 实例大小越大，相关的费用就越高。

- 我是否应考虑任何其他性能修复程序？

	是的，请考虑为 SQL Server 应用相关的性能修复程序：

	- [修复：在 SQL Server 2012 中执行 select into temporary table 操作时出现的 I/O 性能不良](https://support.microsoft.com/kb/2958012)

	- [修复：在 SQL Server 2014 中移动 SQL Server 资源时出现“SQL Server 性能计数器已禁用”](http://support.microsoft.com/kb/2973444)

- 如何找到有关存储空间的详细信息？

	有关存储空间的更多详细信息，请参阅[存储空间常见问题 (FAQ)](http://social.technet.microsoft.com/wiki/contents/articles/11382.storage-spaces-frequently-asked-questions-faq.aspx)

- 新 DW 映像与以前的映像之间有什么差异？

	以前的 DW 映像需要客户执行其他步骤，例如，在创建 VM 后附加数据磁盘；而新的 DW 映像在创建后即可使用，因此可以更快地准备就绪，并且出错的几率更低。

- 我可以继续使用以前的 DW 映像吗？ 可以通过哪种方式访问它？

	以前的 VM 映像仍然可用，只不过无法从库直接访问，而是要使用 Powershell cmdlet 来访问。例如，你可以使用 **Get-AzureVMImage** 列出所有映像；如果你要根据描述和发布日期查找以前的 DW 映像，可以使用 **New-AzureVM** 进行预配。

## 后续步骤

在安装使用 SQL Server 的任何虚拟机后，你可能想要：

- [迁移数据](/documentation/articles/virtual-machines-migrate-onpremises-database)
- [设置连接](/documentation/articles/virtual-machines-sql-server-connectivity)

有关其他与在 Azure VM 中运行 SQL Server 相关的主题，请参阅 [Azure 虚拟机上的 SQL Server](/documentation/articles/virtual-machines-sql-server-infrastructure-services)。

<!---HONumber=70-->