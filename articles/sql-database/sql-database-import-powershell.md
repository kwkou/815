<properties
    pageTitle="使用 PowerShell 导入 BACPAC 文件以创建 Azure SQL 数据库 | Azure"
    description="使用 PowerShell 导入 BACPAC 文件以创建 Azure SQL 数据库"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="8d78da13-43fe-4447-92e0-0a41d0321fd4"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="data-management"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="sstein" />  



# 使用 PowerShell 导入 BACPAC 文件以创建 Azure SQL 数据库

本文说明如何使用 PowerShell 通过导入 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件来创建 Azure SQL 数据库。

## 先决条件


若要导入 SQL 数据库，需要以下内容：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 要导入的数据库的 BACPAC 文件。BACPAC 需位于 [Azure 存储帐户](/documentation/articles/storage-create-storage-account/) Blob 容器中。



[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]



## 设置适合环境的变量

有几个变量需要你将示例值替换为你的数据库和存储帐户的特定值。

服务器名称应是上一步骤选择的订阅中当前存在的服务器。它应该是要在其中创建数据库的服务器。不支持直接将数据库导入弹性池。但是可以先作为单一数据库导入，然后将该数据库移入池中。

数据库名称是要为新数据库赋予的名称。

    $ResourceGroupName = "resource group name"
    $ServerName = "server name"
    $DatabaseName = "database name"


以下变量来自 BACPAC 所处的存储帐户。在 [Azure 门户](https://portal.azure.cn)中，浏览到存储帐户获取这些值。你可以单击存储帐户边栏选项卡中的“所有设置”，然后单击“密钥”，找到主访问密钥。

Blob 名称是要从中创建数据库的现有 BACPAC 文件的名称。需要包括 .bacpac 扩展名。

    $StorageName = "storageaccountname"
    $StorageKeyType = "StorageAccessKey"
    $StorageUri = "http://$StorageName.blob.core.chinacloudapi.cn/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"


运行 [Get-Credential](https://msdn.microsoft.com/zh-cn/library/azure/hh849815(v=azure.300).aspx) cmdlet 会打开一个窗口，要求输入用户名和密码。请输入 SQL 数据库服务器的管理员登录名和密码（上述 $ServerName），而不是 Azure 帐户的用户名和密码。

    $credential = Get-Credential


## 导入数据库

此命令会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

    $importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $ResourceGroupName –ServerName $ServerName –DatabaseName $DatabaseName –StorageKeytype $StorageKeyType –StorageKey $StorageKey -StorageUri $StorageUri –AdministratorLogin $credential.UserName –AdministratorLoginPassword $credential.Password –Edition Standard –ServiceObjectiveName S0 -DatabaseMaxSizeBytes 50000


## 监视导入操作的进度

运行 [New-AzureRmSqlDatabaseImport](https://msdn.microsoft.com/zh-cn/library/mt707793.aspx) 后，可运行 [Get-AzureRmSqlDatabaseImportExportStatus](https://msdn.microsoft.com/zh-cn/library/mt707794.aspx) 查看请求的状态。

    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink



## SQL 数据库 PowerShell 导入脚本


    $ResourceGroupName = "resourceGroupName"
    $ServerName = "servername"
    $DatabaseName = "databasename"

    $StorageName = "storageaccountname"
    $StorageKeyType = "StorageAccessKey"
    $StorageUri = "http://$StorageName.blob.core.chinacloudapi.cn/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"

    $credential = Get-Credential

    $importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $ResourceGroupName –ServerName $ServerName –DatabaseName $DatabaseName –StorageKeytype $StorageKeyType –StorageKey $StorageKey -StorageUri $StorageUri –AdministratorLogin $credential.UserName –AdministratorLoginPassword $credential.Password –Edition Standard –ServiceObjectiveName S0 -DatabaseMaxSizeBytes 50000

    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink



## 后续步骤
* 若要了解如何连接到导入的 SQL 数据库并对其进行查询，请参阅[使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)。
* 如需 SQL Server 客户顾问团队编写的有关使用 BACPAC 文件进行迁移的博客，请参阅 [使用 BACPAC 文件从 SQL Server 迁移到 Azure SQL 数据库](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)。
* 如需 SQL Server 数据库完整迁移过程的介绍（包括性能建议），请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: link references update-->