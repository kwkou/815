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
	ms.date="04/06/2016"
	wacn.date="05/16/2016" />


# 使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-export-powershell/)


本文说明了如何使用 PowerShell 将 Azure SQL 数据库存档到存储在 Azurte blob 存储中的 BACPAC 文件。

需要创建 Azure SQL 数据库的存档时，可以将数据库架构和数据导出到 BACPAC 文件。BACPAC 文件只是一个扩展名为 BACPAC 的 ZIP 文件。之后可将 BACPAC 文件存储在 Azure blob 存储中或存储在本地位置的本地存储中，之后可以导入回 Azure SQL 数据库或 SQL Server 本地安装。

注意事项

- 为保持存档的事务处理方式一致性，必须确保从 Azure SQL 数据库的事务处理方式一致性副本中导出或导入时没有发生写入活动。
- 存档到 Azure blob 存储的 BACPAC 文件的大小上限为 200 GB。可使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用工具将更大的 BACPAC 文件存档到本地存储。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。
- 不支持使用 BACPAC 文件存档到 Azure 高级存储。
- 如果导出操作超过 20 个小时，操作可能被取消。为提高导出过程中的性能，你可以进行如下操作：
 - 暂时提高服务级别。 
 - 在导出过程中终止所有读取和写入活动。
 - 在所有大型表格上使用聚集索引。如果不使用聚集索引，当时间超过 6-12 个小时时，导出可能会失败。因为导出服务需要完成表格扫描才能尝试导出整个表格。
 
> [AZURE.NOTE] BACPAC 不能用于备份和还原操作。Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

若要完成本文，你需要以下各项：

- Azure 订阅。 
- Azure SQL 数据库。 
- 具有 blob 容器的 [Azure 标准存储帐户](/documentation/articles/storage-create-storage-account/)可在标准存储中存储 BACPAC。
- Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中显示的信息中复制订阅 ID，或者，如果你有多个订阅且需要更多详细信息，可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅以后，你可以运行以下 cmdlet：

	Select-AzureSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureSubscription** 后，将返回到 PowerShell 提示符处。如果你有多个订阅，可以运行 **Get-AzureSubscription** 并验证要使用的订阅显示 **IsCurrent: True**。


## 设置适合特定环境的变量

有几个变量需要你将示例值替换为你的数据库和存储帐户的特定值。

将服务器和数据库名称替换为当前存在于你的帐户中的服务器和数据库。对于 blob 名称，输入将创建的 BACPAC 文件名。输入想要使用的任意 BACPAC 文件名，但必须包括 .bacpac 扩展名。

    $ServerName = "servername"
    $DatabaseName = "nameofdatabasetoexport"
    $BlobName = "filename.bacpac"

在 [Azure 管理门户](https://manage.windowsazure.cn)中，浏览到你的存储帐户以获取这些值。你可以在存储帐户的下方单击“管理访问密钥”来找到主访问密钥。

    $StorageName = "storageaccountname"
    $ContainerName = "blobcontainername"
    $StorageKey = "primaryaccesskey"

## 创建指向服务器和存储帐户指针

运行 **Get-Credential** cmdlet 会打开一个窗口，要求你输入用户名和密码。请输入 SQL Server 的管理员登录名和密码（而非 Azure 帐户的用户名和密码）。

    $credential = Get-Credential
    $SqlCtx = New-AzureSqlDatabaseServerContext -ServerName $ServerName -Credential $credential

    $StorageCtx = New-AzureStorageContext -StorageAccountName $StorageName -StorageAccountKey $StorageKey
    $Container = Get-AzureStorageContainer -Name $ContainerName -Context $StorageCtx


## 导出数据库

此命令会将导出数据库请求提交到服务。根据数据库的大小，导出操作可能需要一些时间才能完成。

> [AZURE.IMPORTANT] 若要确保获得事务处理一致性 BACPAC 文件，应首先[创建数据库的副本](/documentation/articles/sql-database-copy-powershell/)，然后导出该数据库副本。


    $exportRequest = Start-AzureSqlDatabaseExport -SqlConnectionContext $SqlCtx -StorageContainer $Container -DatabaseName $DatabaseName -BlobName $BlobName
    

## 监视导出操作的进度

运行 **Start-AzureSqlDatabaseExport** 之后，你可以检查请求的状态。在请求后立即运行此命令通常将返回 **Status : Pending** 或 **Status : Running**，因此你可以运行此命令多次，直到看到输出中显示 **Status : Completed**。

运行此命令将提示你输入密码。请输入 SQL Server 的管理员密码。


    Get-AzureSqlDatabaseImportExportStatus -RequestId $exportRequest.RequestGuid -ServerName $ServerName -Username $credential.UserName
    


## 导出 SQL 数据库 PowerShell 脚本


    Add-AzureAccount -Environment AzureChinaCloud
    Select-AzureSubscription -SubscriptionId "4cac86b0-1e56-bbbb-aaaa-000000000000"
    
    $ServerName = "servername"
    $DatabaseName = "databasename"
    $BlobName = "bacpacfilename"
    
    $StorageName = "storageaccountname"
    $ContainerName = "blobcontainername"
    $StorageKey = "primaryaccesskey"
    
    $credential = Get-Credential
    $SqlCtx = New-AzureSqlDatabaseServerContext -ServerName $ServerName -Credential $credential
    
    $StorageCtx = New-AzureStorageContext -StorageAccountName $StorageName -StorageAccountKey $StorageKey
    $Container = Get-AzureStorageContainer -Name $ContainerName -Context $StorageCtx
    
    $exportRequest = Start-AzureSqlDatabaseExport -SqlConnectionContext $SqlCtx -StorageContainer $Container -DatabaseName $DatabaseName -BlobName $BlobName
    
    Get-AzureSqlDatabaseImportExportStatus -RequestId $exportRequest.RequestGuid -ServerName $ServerName -Username $credential.UserName
    


## 后续步骤

- [导入 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell/)


## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0509_2016-->
