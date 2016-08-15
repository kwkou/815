<properties 
    pageTitle="使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库 | Azure" 
    description="使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="07/06/2016"
    wacn.date="08/15/2016"/>

# 使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-import-powershell/)
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)

本文说明如何使用 PowerShell 通过导入 BACPAC 来创建 Azure SQL 数据库。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。有关详细信息，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)中的备份包 (.bacpac)。

数据库是使用从 Azure 存储 blob 容器导入的 BACPAC 创建的。如果 Azure 存储空间中没有 .bacpac 文件，你可以按照[使用 PowerShell 将 Azure SQL 数据库存档到 BACPAC 文件](/documentation/articles/sql-database-export-powershell/)中的步骤创建一个。

> [AZURE.NOTE] Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。


要导入 SQL 数据库，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 要导入的数据库的 .bacpac 文件 (BACPAC)。BACPAC 需位于 [Azure 存储帐户（经典）](/documentation/articles/storage-create-storage-account/)blob 容器中。



[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]



## 设置适合环境的变量

有几个变量需要你将示例值替换为你的数据库和存储帐户的特定值。

服务器名称必须是在上一步中选择的订阅中当前存在的服务器，并且必须是你要在其中创建数据库的服务器。注意：不支持直接将数据库导入弹性池，但可以先导入至单一数据库，然后将数据库移入池中。

数据库名称是你要给新数据库取的名称。

    $ResourceGroupName = "resource group name"
    $ServerName = "server name"
    $DatabaseName = "database name"


以下变量来自 BACPAC 所处的存储帐户。在 [Azure 门户](https://portal.azure.cn)中，浏览到你的存储帐户以获取这些值。你可以单击存储帐户边栏选项卡中的“所有设置”，然后单击“密钥”，找到主访问密钥。

Blob 名称是你想要从中创建的数据库的现有 .bacpac 文件的名称。需要包括 .bacpac 扩展名。

    $StorageName = "storageaccountname"
    $StorageKeyType = "storageKeyType"
    $StorageUri = "http://storageaccountname.blob.core.chinacloudapi.cn/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"


运行 **Get-Credential** cmdlet 会打开一个窗口，要求你输入用户名和密码。请输入 SQL 数据库服务器的管理员登录名和密码（上述 $ServerName），而非 Azure 帐户的用户名和密码。

    $credential = Get-Credential


## 导入数据库

此命令会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

    $importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $ResourceGroupName –ServerName $ServerName –DatabaseName $DatabaseName –StorageKeytype $StorageKeyType –StorageKey $StorageKey StorageUri $StorageUri –AdministratorLogin $credential.UserName –AdministratorPassword $credential.Password –Edition Standard –ServiceObjectiveName S0 -DatabaseMxSize 50000
    

## 监视导入操作的进度

运行“New-AzureRmSqlDatabaseImport”之后，你可以检查请求的状态。

在请求后立即检查状态通常将返回 **Pending** 或 **Running** 状态并显示当前完成百分比，因此你可以运行此命令多次，直到看到输出中显示 **Status : Completed**。

运行此命令将提示你输入密码。请输入 SQL 服务器的管理员登录名和密码。


    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest .OperationStatusLink
 


## SQL 数据库 PowerShell 导入脚本


    $ResourceGroupName = "resourceGroupName"
    $ServerName = "servername"
    $DatabaseName = "databasename"

    $StorageName = "storageaccountname"
    $StorageKeyType = "storageKeyType"
    $StorageUri = "http://storageaccountname.blob.core.chinacloudapi.cn/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"

    $credential = Get-Credential

    $importRequest = New-AzureRmSqlDatabaseImport –ResourceGroupName $ResourceGroupName –ServerName $ServerName –DatabaseName $DatabaseName –StorageKeytype $StorageKeyType –StorageKey $StorageKey  StorageUri $StorageUri –AdministratorLogin $credential.UserName –AdministratorPassword $credential.Password –Edition Standard –ServiceObjectiveName S0 -DatabaseMxSize 50000
 
    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest .OperationStatusLink

    

## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)




## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0808_2016-->