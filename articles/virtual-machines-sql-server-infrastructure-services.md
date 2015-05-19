<properties 
	pageTitle="Azure 虚拟机中的 SQL Server" 
	description="本文概述了在 Azure 基础结构服务 (IaaS) 中托管的 SQL Server。这包括深度内容的链接。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="rothja" 
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services" 
	ms.date="04/17/2015"
  wacn.date="05/15/2015"
	ms.author="jroth"/>

# Azure 虚拟机中的 SQL Server

## 概述
你可以使用各种配置（从单个数据库服务器到多计算机配置）通过 AlwaysOn 可用性组和 Azure 虚拟网络托管 [Azure 虚拟机中的 SQL Server][sqlvmlanding]。

> [AZURE.NOTE] 在 Azure VM 中运行 SQL Server 是在 Azure 中存储关系数据的一个选项。此外，还可以使用 Azure SQL Database 服务。 <!--For more information, see [Understanding Azure SQL Database and SQL Server in Azure VMs][sqldbcompared].-->
 
## 在单个 VM 上部署 SQL Server 实例
[使用 Azure 门户创建 Azure 虚拟机][createvmportal] 或自动安装后，可以安装你有其许可证的 SQL Server 的任何实例。但是，你必须采取其他步骤在 SQL Server 计算机和其他客户端计算机之间[设置连接][setupconnectivity]。
 
你还可以从库中安装众多不同的 SQL Server 虚拟机映像之一。这些映像包括 VM 定价中的 SQL Server 许可。有关详细信息，请参阅[在 Azure 上设置 SQL Server 虚拟机][provisionsqlvm]。

## 使用多个 VM 部署高可用性配置
可以使用 SQL Server AlwaysOn 可用性组，实现 SQL Server 高可用性。这涉及虚拟网络中的多个 Azure VM。Azure 预览版门户有一个模板为你设置了此配置。有关详细信息，请参阅 [Microsoft Azure 门户库中的 SQL Server AlwaysOn 产品][sqlalwaysonportal]。或者，你可以[手动配置 AlwaysOn 可用性组][sqlalwaysonmanual]。有关其他高可用性注意事项，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复][sqlhadr]。

## 在 Azure 中运行商业智能、数据仓库和 OLTP 工作负荷   
你可以在 Azure 虚拟机上运行常见的 SQL Server 工作负荷。SQL Server 有多个经过优化的虚拟机映像在库中提供。其中包括[商业智能][sqlbi]、[数据仓库][sqldw] 和 [OLTP][sqloltp] 的映像。

## 迁移数据
有几种可行方法可将你的数据迁移到运行 SQL Server 的 Azure VM 中。使用 Azure 门户、PowerShell 自动化或 SQL Server Management Studio 中的部署向导首次设置 SQL Server 虚拟机。经过优化的 SQL Server 映像包括其定价模型中的许可，但你还可以使用你自己的许可证安装 SQL Server。若要迁移你的数据，有几个选项，例如使用部署向导或将数据磁盘迁移到目标虚拟机。有关详细信息，请参阅[准备迁移到 Azure 虚拟机中的 SQL Server][migratesql]。

## 备份和还原
对于本地数据库，Azure 可以充当用于存储 SQL Server 备份文件的辅助数据中心。[SQL Server 备份到 URL][backupurl] 将 Azure 备份文件存储在 Azure Blob 存储中。[SQL Server 托管备份][managedbackup] 允许你在 Azure 中计划备份和保留期。这些服务可用于本地 SQL Server 实例或 Azure VM 上运行的 SQL Server。Azure VM 还可以利用 SQL Server 的[自动备份][autobackup] 和[自动修补][autopatching]。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的管理任务][managementtasks]。

## 其他资源：
[Azure VM 中的 SQL Server][sqlmsdnlanding]

[Azure 虚拟机中的 SQL Server 入门][sqlvmgetstarted]

[Azure 虚拟机中 SQL Server 的性能最佳实践][sqlperf]

[Azure 虚拟机中 SQL Server 的安全注意事项][sqlsecurity]

[Azure 虚拟机中 SQL Server 的技术文章][technicalarticles]

  [sqlvmlanding]: /home/features/virtual-machines/
<!--[sqldbcompared]: /documentation/articles/data-management-azure-sql-database-and-sql-server-iaas-->
  [createvmportal]: /documentation/articles/virtual-machines-windows-tutorial/
  [setupconnectivity]: https://msdn.microsoft.com/zh-CN/library/azure/dn133152.aspx
  [provisionsqlvm]: /documentation/articles/virtual-machines-provision-sql-server/
  [sqlalwaysonportal]: http://go.microsoft.com/fwlink/?LinkId=526941
  [sqlalwaysonmanual]: https://msdn.microsoft.com/zh-CN/library/azure/dn249504.aspx
  [sqlhadr]: https://msdn.microsoft.com/zh-CN/library/azure/jj870962.aspx
  [sqlbi]: https://msdn.microsoft.com/zh-CN/library/azure/jj992719.aspx
  [sqldw]: https://msdn.microsoft.com/zh-CN/library/azure/dn387396.aspx
  [sqloltp]: https://msdn.microsoft.com/zh-CN/library/azure/eb0188e2-5569-48ff-b92c-1f6c0bf79620#about
  [migratesql]: https://msdn.microsoft.com/zh-CN/library/azure/dn133142.aspx
  [backupurl]: https://msdn.microsoft.com/zh-CN/library/dn435916(v=sql.120).aspx
  [managedbackup]: https://msdn.microsoft.com/zh-CN/library/dn449496.aspx
  [autobackup]: https://msdn.microsoft.com/zh-CN/library/azure/dn906091.aspx
  [autopatching]: https://msdn.microsoft.com/zh-CN/library/azure/dn961166.aspx
  [managementtasks]: https://msdn.microsoft.com/zh-CN/library/azure/dn906886.aspx
  [sqlmsdnlanding]: https://msdn.microsoft.com/zh-CN/library/azure/jj823132.aspx
  [sqlvmgetstarted]: https://msdn.microsoft.com/zh-CN/library/azure/dn133151.aspx
  [sqlperf]: https://msdn.microsoft.com/zh-CN/library/azure/dn133149.aspx
  [sqlsecurity]: https://msdn.microsoft.com/zh-CN/library/azure/dn133147.aspx
  [technicalarticles]: https://msdn.microsoft.com/zh-CN/library/azure/dn248435.aspx

<!--HONumber=53-->