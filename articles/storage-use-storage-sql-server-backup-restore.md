<properties linkid="manage-services-storage-SQL-Server-backup" urlDisplayName="Storage for SQL Server backups" pageTitle="如何将 Azure 存储空间用于 SQL Server 备份和还原 | Azure" metaKeywords="" description="" metaCanonical="" services="storage" documentationCenter="" title="How to Use Azure Storage for SQL Server Backup and Restore" authors="karaman" solutions="" manager="clairt" editor="tysonn" />



<h1 id="SQLServerBackupandRestoretostorage">如何将 Azure 存储空间用于 SQL Server 备份和还原</h1>

SQL Server 2012 SP1 CU2 中发布了可将 SQL Server 备份写入 Azure Blob 存储服务的功能。你可以使用此功能将数据从本地 SQL Server 数据库或 Azure 虚拟机中的 SQL Server 数据库备份到 Azure Blob 服务或从中进行还原。备份到云具有以下优点，即，实现可用性、无地域复制场外存储限制，以及可以轻松将数据迁移到云和从云中迁移数据。   在此版本中，你可以使用 T-SQL 或 SMO 来发布 BACKUP 或 RESTORE 语句。无法使用"SQL Server Management Studio 备份或还原"向导来备份到 Azure Blob 存储服务或从中进行还原。

<h2> 使用 Azure Blob 服务执行 SQL Server 备份的优点</h2>

存储管理、存储故障产生的风险、访问场外存储以及对设备进行配置是一些普遍存在的备份难题。对于在 Azure 虚拟机中运行的 SQL Server，配置和备份 VHD 或配置附加驱动器将面临一些额外挑战。下面列出了使用 Azure Blob 存储服务存储进行 SQL Server 备份的一些主要优点：

* 灵活、可靠且无场外存储限制：在 Azure Blob 服务中存储备份非常方便、灵活且可轻松访问场外存储。为 SQL Server 备份创建场外存储就像修改现有脚本/作业一样简单。场外存储通常应当远离生产数据库位置，以防止某个灾难可能同时影响场外和生产数据库位置。通过选择地域复制 Blob 存储，你可以在发生可能影响整个地区的灾难时进一步加强保护。此外，可随时随地且轻松地访问备份数据以进行还原。
* 备份存档：在对备份进行存档时，Azure Blob 存储服务提供了可替代常用磁带存储方式的更好方式。选择磁带存储时可能需要将数据实际运输到场外设施，并且需要采取一些介质保护措施。在 Azure Blob 存储中存储备份可提供即时、具有高可用性且持久的存档方式。
* 无硬件管理开销：使用 Azure 服务没有硬件管理开销。Azure 服务可管理硬件并提供地域冗余复制和硬件故障防护。
* 当前，对于在 Azure 虚拟机中运行的 SQL Server 实例，可以通过创建附加的磁盘来备份到 Azure Blob 存储服务。不过，你只能将有限数量的磁盘附加到 Azure 虚拟机。对特大实例的限制为 16 个磁盘；对较小实例的磁盘限制数更少。通过直接备份到 Azure Blob 存储，你可以绕过 16 个磁盘这一限制。
* 此外，目前存储在 Azure Blob 存储服务中的备份文件可供本地 SQL Server 或运行在 Azure 虚拟机中的其他 SQL Server 直接访问，而无需进行数据库附加/分离或者下载和附加 VHD。
* 成本优势：只需为所使用的服务付费。作为场外和备份存档方式可能更加划算。有关详细信息，请参阅 [Azure 定价计算器](/zh-cn/pricing/calculator/#data-management "Pricing Calculator")和 [Azure 定价文章](/zh-cn/pricing/ "Pricing article")。

有关更多详细信息，请参阅[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj919148.aspx)。

以下两部分介绍了 Azure Blob 存储服务，以及备份到 Azure Blob 存储服务或从中进行还原时使用的 SQL Server 组件。了解这些组件以及它们之间的交互对备份到 Azure Blob 存储服务或从中进行还原来说至关重要。 

创建 Azure 帐户是这个过程的第一步。SQL Server 使用 Azure 存储帐户名及其访问密钥值来对存储服务进行身份验证，然后读取 Blob 并将其写入存储服务。SQL Server 凭据存储此身份验证信息，并且将在备份或还原期间使用这些信息。 

有关创建存储帐户和执行简单还原操作的完整演练，请参阅[开始使用 Azure 存储服务执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj720558.aspx)。

## Azure Blob 存储服务组件 

* 存储帐户：存储帐户是所有存储服务的起点。若要访问 Azure Blob 存储服务，请先创建一个 Azure 存储帐户。存储帐户名及其访问密钥属性是对 Azure Blob 存储服务及其组件进行身份验证所必需的。 
有关 Azure Blob 存储服务的详细信息，请参阅[如何使用 Azure Blob 存储服务](/zh-cn/develop/net/how-to-guides/blob-storage-v17/)。

* 容器：容器提供一组 Blob 集，并且可存储无限数量的 Blob。若要将 SQL Server 备份写入到 Azure Blob 服务，你必须至少创建一个根容器。 

* Blob：任何类型和大小的文件。可将两类 Blob 存储到 Azure 存储服务中：块 Blob 和页 Blob。SQL Server 备份使用页 Blob 作为 Blob 类型。使用以下 URL 格式可访问 Blob：`https://<storage account>.blob.core.chinacloudapi.cn/<container>/<blob>`
有关页 Blob 的更多信息，请参见[了解块 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。

## SQL Server 组件

* URL：URL 指定到唯一备份文件的统一资源标识符 (URI)。URL 用于提供 SQL Server 备份文件的位置和名称。在此实现中，唯一有效的 URL 是指向 Azure 存储帐户中的页 Blob 的 URL。URL 必须指向实际 Blob，而不是仅指向容器。如果 Blob 不存在，则会创建一个。如果指定了现有 Blob，BACKUP 将失败，除非指定 > WITH FORMAT 选项。 
下面是你将在 BACKUP 命令中指定的 URL 示例： 
**`http[s]://ACCOUNTNAME.Blob.core.chinacloudapi.cn/<CONTAINER>/<FILENAME.bak>`

<b>注意：</b> HTTPS 不是必需的，但建议使用它。
<b>重要说明</b>
如果你选择将备份文件复制并上载到 Azure Blob 存储服务中，并且打算使用此文件执行还原操作，则必须将页 Blob 类型作为存储选项。从块 Blob 类型执行 RESTORE 命令将失败并报错。 

* 凭据：连接到 Azure Blob 存储服务并通过其进行身份验证所需的信息将存储为凭据。为了使 SQL Server 将备份写入 Azure Blob 或从中进行还原，必须创建 SQL Server 凭据。凭据存储存储帐户的名称和存储帐户访问密钥。创建凭据后，必须在发布 BACKUP/RESTORE 语句时在 WITH CREDENTIAL 选项中指定该凭据。有关如何查看、复制或重新生成存储帐户访问密钥的详细信息，请参阅[存储帐户访问密钥](http://msdn.microsoft.com/zh-cn/library/azure/hh531566.aspx)。
有关如何创建 SQL Server 凭据的分步说明，请参阅[开始使用 Azure 存储服务执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj720558.aspx)。

## 使用 Azure Blob 执行 SQL Server 数据库备份和还原 - 概念和任务：

**概念、注意事项和代码示例：**

[使用 Azure Blob 存储服务执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj919148.aspx)

**教程入门：**

[开始使用 Azure Blob 存储服务执行 SQL Server 备份和还原](http://msdn.microsoft.com/zh-cn/library/jj720558.aspx "Tutorial")

**最佳实践、疑难解答：**
	
[备份和还原最佳实践（Azure Blob 存储服务）](http://msdn.microsoft.com/zh-cn/library/jj919149.aspx)




	




<!--HONumber=41-->
