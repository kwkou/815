<properties
	pageTitle="如何使用 Azure 存储空间执行 SQL Server 备份和还原 | Azure"
	description="将 SQL Server 和 SQL 数据库备份到 Azure 存储空间。介绍了将 SQL 数据库备份到 Azure 存储空间的好处，以及需要哪些 SQL Server 和 Azure 存储空间组件"
	services="sql-database, virtual-machines"
	documentationCenter=""
	authors="carlrabeler"
	manager="jeffreyg"
	editor="tysonn"/>

<tags
	ms.service="sql-database"
	ms.date="02/16/2016"
	wacn.date="03/24/2016"/>



# 如何将 Azure 存储空间用于 SQL Server 备份和还原

## 概述

SQL Server 2012 SP1 CU2 中发布了可将 SQL Server 备份写入 Azure Blob 存储服务的功能。你可以使用此功能将数据从本地 SQL Server 数据库或 Azure 虚拟机中的 SQL Server 数据库备份到 Azure Blob 服务或从中进行还原。备份到云具有以下优点，即，实现可用性、无地域复制场外存储限制，以及可以轻松将数据迁移到云和从云中迁移数据。你可以使用 Transact-SQL 或 SMO 来发布 BACKUP 或 RESTORE 语句。此外，如果数据库文件存储在 Azure blob 中并且你使用的是 SQL Server 2016，你可以使用 [file-snapshot backup](http://msdn.microsoft.com/zh-cn/library/mt169363.aspx) 执行几乎即时的备份和令人难以置信的快速还原。

## 使用 Azure Blob 服务执行 SQL Server 备份的优点

存储管理、存储故障产生的风险、访问场外存储以及对设备进行配置是一些普遍存在的备份难题。本地数据库实例和 Azure 虚拟机数据库实例都存在这些难题。下面列出了使用 Azure Blob 存储服务存储进行 SQL Server 备份的一些主要优点：

* 灵活、可靠且无限制的场外存储：在 Azure blob 中存储你的备份是一种方便、灵活且易于访问的场外选项。为 SQL Server 备份创建场外存储就像修改现有脚本/作业以使用 **BACKUP TO URL** 语法一样简单。场外存储通常应当远离生产数据库位置，以防止某个灾难可能同时影响场外和生产数据库位置。通过选择[异地复制 Azure blob](/documentation/articles/storage-redundancy/)，你可以在发生可能影响整个地区的灾难时进一步加强保护。 
* 备份存档：在对备份进行存档时，Azure Blob 存储服务提供了可替代常用磁带存储方式的更好方式。选择磁带存储时可能需要将数据实际运输到场外设施，并且需要采取一些介质保护措施。在 Azure Blob 存储中存储备份可提供即时、具有高可用性且持久的存档方式。
* 无硬件管理开销：使用 Azure 服务没有硬件管理开销。Azure 服务可管理硬件并提供地域冗余复制和硬件故障防护。
* 当前，对于在 Azure 虚拟机中运行的 SQL Server 实例，可以通过创建附加磁盘来备份到 Azure Blob 存储服务。不过，你只能将有限数量的磁盘附加到用于备份的 Azure 虚拟机。对特大实例的限制为 16 个磁盘；对较小实例的磁盘限制数更少。通过启用直接备份到 Azure blob，你可以访问几乎无限的存储。
* 存储在 Azure Blob 中的备份可随时从任何位置使用，并可供轻松访问以还原到本地 SQL Server 或运行在 Azure 虚拟机中的其他 SQL Server，而无需进行数据库附加/分离或者下载和附加 VHD。
* 成本优势：只需为所使用的服务付费。作为场外和备份存档方式可能更加划算。有关详细信息，请参阅 [Azure 定价计算器](/pricing/calculator)和 [Azure 定价文章](/pricing/overview)。
* 利用存储快照：如果数据库文件存储在 Azure blob 中并且你使用的是 SQL Server 2016，你可以使用 [file-snapshot backup](http://msdn.microsoft.com/zh-cn/library/mt169363.aspx) 执行几乎即时的备份和令人难以置信的快速还原。

有关更多详细信息，请参阅[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](http://go.microsoft.com/fwlink/?LinkId=271617)。

以下两部分介绍了 Azure Blob 存储服务，以及备份到 Azure Blob 存储服务或从中进行还原时使用的 SQL Server 组件。了解这些组件以及它们之间的交互对备份到 Azure Blob 存储服务或从中进行还原来说至关重要。

创建 Azure 帐户是这个过程的第一步。有关使用 SQL Server 2014 创建存储帐户和执行简单还原操作的完整演练，请参阅[开始使用 Azure 存储服务执行 SQL Server 备份和还原](https://msdn.microsoft.com/zh-cn/library/jj720558(v=sql.120).aspx)。有关使用 SQL Server 2014 创建存储帐户和执行简单还原操作的完整演练，请参阅[教程：将 Azure Blob 存储服务用于 SQL Server 2016 数据库](https://msdn.microsoft.com/zh-cn/library/dn466438.aspx)。

## Azure Blob 存储服务组件

* 存储帐户：存储帐户是所有存储服务的起点。若要访问 Azure Blob 存储服务，请先创建一个 Azure 存储帐户。有关 Azure Blob 存储服务的详细信息，请参阅[如何使用 Azure Blob 存储服务](/documentation/articles/storage-dotnet-how-to-use-blobs/)

* 容器：容器提供一组 Blob 集，并且可存储无限数量的 Blob。若要将 SQL Server 备份写入到 Azure Blob 服务，你必须至少创建一个根容器。

* Blob：任何类型和大小的文件。使用以下 URL 格式可访问 Blob：`https://<storage account>.blob.core.chinacloudapi.cn/<container>/<blob>`
有关页 Blob 的详细信息，请参阅[了解块 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)

## SQL Server 组件

* URL：URL 指定到唯一备份文件的统一资源标识符 (URI)。URL 用于提供 SQL Server 备份文件的位置和名称。URL 必须指向实际 blob，而不是仅指向容器。如果 blob 不存在，则会创建一个。如果指定了现有 blob，除非指定了 > WITH FORMAT 选项，否则 BACKUP 将失败。
下面是你将在 BACKUP 命令中指定的 URL 示例：
**`http[s]://ACCOUNTNAME.Blob.core.chinacloudapi.cn/<CONTAINER>/<FILENAME.bak>`

<b>注意：</b>HTTPS 不是必需的，但建议使用它。
<b>重要提示</b>
如果你选择将备份文件复制并上载到 Azure Blob 存储服务中，并且打算使用此文件执行还原操作，则必须将页 Blob 类型作为存储选项。从块 Blob 类型执行 RESTORE 命令将失败并报错。

* 凭据：连接到 Azure Blob 存储服务并向其进行身份验证所需的信息将存储为凭据。为了使 SQL Server 将备份写入 Azure Blob 或从中进行还原，必须创建 SQL Server 凭据。有关详细信息，请参阅 [SQL Server 凭据](https://msdn.microsoft.com/zh-cn/library/ms189522.aspx)。

## 使用 Azure Blob 执行 SQL Server 数据库备份和还原 - 概念和任务：

**概念、注意事项和代码示例：**

[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](http://go.microsoft.com/fwlink/?LinkId=271617)

**教程入门：**

[教程：将 Azure Blob 存储服务用于 SQL Server 2016 数据库](https://msdn.microsoft.com/zh-cn/library/dn466438.aspx)

**最佳实践、疑难解答：**

[备份和还原最佳实践（Azure Blob 存储服务）](http://go.microsoft.com/fwlink/?LinkId=272394)

<!---HONumber=Mooncake_0118_2016-->
