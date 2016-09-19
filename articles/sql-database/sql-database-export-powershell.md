<properties 
    pageTitle="使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件" 
    description="使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="08/15/2016"
	wacn.date="09/19/2016" />


# 使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件




本文说明了如何使用 PowerShell 将 Azure SQL 数据库存档到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件（存于 Azure Blob 存储中）。

需要创建 Azure SQL 数据库的存档时，可以将数据库架构和数据导出到 BACPAC 文件。BACPAC 文件只是一个扩展名为 .bacpac 的 ZIP 文件。BACPAC 文件稍后可存于 Azure Blob 存储中，也可存于本地位置的本地存储中。它还可导回 Azure SQL 数据库或 SQL Server 本地安装。

**注意事项**

- 为保证存档的事务处理方式一致，须确保导出期间未发生写入活动，或者正在从 Azure SQL 数据库的[事务处理方式一致性副本](/documentation/article/sql-database-copy/)中导出。
- 存档到 Azure blob 存储的 BACPAC 文件的大小上限为 200 GB。可使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示使用工具将更大的 BACPAC 文件存到恩地存储。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。
- 不支持使用 BACPAC 文件存档到 Azure 高级存储。
- 如果导出操作耗时超过 20 个小时，可能会取消操作。为提高导出过程中的性能，你可以进行如下操作：
 - 暂时提高服务级别。
 - 在导出期间终止所有读取和写入活动。
 - 对所有大型表格上的非 null 值使用[聚集索引](https://msdn.microsoft.com/zh-cn/library/ms190457.aspx)。如果不使用聚集索引，当时间超过 6-12 个小时时，导出可能会失败。原因是导出服务需要完成表格扫描才能尝试导出整个表格。确认表格是否已就导出进行优化的好方法是运行 **DBCC SHOW\_STATISTICS** 并确保 *RANGE\_HI\_KEY* 不是 null 且其值分布良好。相关详细信息，请参阅 [DBCC SHOW\_STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms174384.aspx)。

> [AZURE.NOTE] BACPAC 不能用于备份和还原操作。Azure SQL 数据库会自动为每个用户数据库创建备份。相关详细信息，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。

若要完成本文，需要以下各项：

- Azure 订阅。
- Azure SQL 数据库。
- [Azure 标准存储帐户](/documentation/articles/storage-create-storage-account/)，具有可在标准存储中存储 BACPAC 的 blob 容器。


[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]




## 导出数据库

[New-AzureRmSqlDatabaseExport](https://msdn.microsoft.com/zh-cn/library/mt707796.aspx) cmdlet 向服务提交导出数据库请求。根据数据库的大小，导出操作可能需要一些时间才能完成。

> [AZURE.IMPORTANT] 若要确保获得事务处理一致性 BACPAC 文件，应首先[创建数据库的副本](/documentation/articles/sql-database-copy-powershell/)，然后导出该数据库副本。


     $exportRequest = New-AzureRmSqlDatabaseExport –ResourceGroupName $ResourceGroupName –ServerName $ServerName `
       –DatabaseName $DatabaseName –StorageKeytype $StorageKeytype –StorageKey $StorageKey -StorageUri $BacpacUri `
       –AdministratorLogin $creds.UserName –AdministratorLoginPassword $creds.Password


## 监视导出操作的进度

运行 **New-AzureRmSqlDatabaseExport** 后，可运行 [Get-AzureRmSqlDatabaseImportExportStatus](https://msdn.microsoft.com/zh-cn/library/mt707794.aspx) 来查看请求的状态。请求后立即运行通常会返回“状态: 处理中”。显示“状态 : 成功”时，表示导出完毕。


    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink



## 导出 SQL 数据库示例

以下示例将现有 SQL 数据库导出到 BACPAC，随后显示如何查看导出操作的状态。

若要运行示例，需将几个变量替换为数据库和存储帐户中的特定值。在 [Azure 门户预览](https://portal.azure.cn)中，浏览到存储帐户以获取存储帐户名称、blob 容器名称和密钥值。可单击存储帐户边栏选项卡上的“访问密钥”查找密钥。

将以下 `VARIABLE-VALUES` 替换为特定 Azure 资源中的值。数据库名称就是要导出的现有数据库。



    $subscriptionId = "YOUR AZURE SUBSCRIPTION ID"

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Set-AzureRmContext -SubscriptionId $subscriptionId

    # Database to export
    $DatabaseName = "DATABASE-NAME"
    $ResourceGroupName = "RESOURCE-GROUP-NAME"
    $ServerName = "SERVER-NAME"
    $serverAdmin = "ADMIN-NAME"
    $serverPassword = "ADMIN-PASSWORD" 
    $securePassword = ConvertTo-SecureString –String $serverPassword –AsPlainText -Force
    $creds = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $serverAdmin, $securePassword

    # Generate a unique filename for the BACPAC
    $bacpacFilename = $DatabaseName + (Get-Date).ToString("yyyyMMddHHmm") + ".bacpac"

    # Storage account info for the BACPAC
    $BaseStorageUri = "https://STORAGE-NAME.blob.core.chinacloudapi.cn/BLOB-CONTAINER-NAME/"
    $BacpacUri = $BaseStorageUri + $bacpacFilename
    $StorageKeytype = "StorageAccessKey"
    $StorageKey = "YOUR STORAGE KEY"

    $exportRequest = New-AzureRmSqlDatabaseExport –ResourceGroupName $ResourceGroupName –ServerName $ServerName `
       –DatabaseName $DatabaseName –StorageKeytype $StorageKeytype –StorageKey $StorageKey -StorageUri $BacpacUri `
       –AdministratorLogin $creds.UserName –AdministratorLoginPassword $creds.Password
    $exportRequest

    # Check status of the export
    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink



## 后续步骤

- 若要了解如何使用 Powershell 导入 Azure SQL 数据库，请参阅[使用 PowerShell 导入 BACPAC](/documentation/articles/sql-database-import-powershell/)


## 其他资源

- [New-AzureRmSqlDatabaseExport](https://msdn.microsoft.com/zh-cn/library/mt707796.aspx)
- [Get-AzureRmSqlDatabaseImportExportStatus](https://msdn.microsoft.com/zh-cn/library/mt707794.aspx)

<!---HONumber=Mooncake_0912_2016-->