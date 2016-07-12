<properties 
    pageTitle="使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库 | Azure" 
    description="使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jeffreyg" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="02/05/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 导入 BACPAC 文件以创建新的 Azure SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-import-powershell/)
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)

本文说明如何使用 PowerShell 通过导入 BACPAC 来创建 Azure SQL 数据库。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。有关详细信息，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)中的备份包 (.bacpac)。

数据库是使用从 Azure 存储 blob 容器导入的 BACPAC 创建的。如果 Azure 存储空间中没有 .bacpac 文件，你可以按照[创建和导出 Azure SQL 数据库的 BACPAC](/documentation/articles/sql-database-export-powershell/) 中的步骤创建一个。

> [AZURE.NOTE] Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。


要导入 SQL 数据库，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- 要导入的数据库的 .bacpac 文件 (BACPAC)。BACPAC 需位于 [Azure 存储帐户（经典）](/documentation/articles/storage-create-storage-account/)blob 容器中。


> [AZURE.IMPORTANT] 本文包含的命令适用于最高版本为 1.0（但不含）的 Azure PowerShell。可以使用 **Get-Module azure | format-table version** 命令查看 Azure PowerShell 的版本。



## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID。你可以从前面的步骤中显示的信息中复制订阅 ID，或者，如果你有多个订阅且需要更多详细信息，可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅 ID 以后，你可以运行以下 cmdlet：

	Select-AzureSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureSubscription** 后，将返回到 PowerShell 提示符处。如果你有多个订阅，可以运行 **Get-AzureSubscription** 并验证所选择的订阅是否显示 **IsCurrent: True**。


## 设置适合环境的变量

有几个变量需要你将示例值替换为你的数据库和存储帐户的特定值。

服务器名称必须是在上一步中选择的订阅中当前存在的服务器，并且必须是你要在其中创建数据库的服务器。

数据库名称是你要给新数据库取的名称。

    $ServerName = "servername"
    $DatabaseName = "databasename"


以下变量来自 BACPAC 所处的存储帐户。在 [Azure 门户](https://manage.windowsazure.cn)中，浏览到你的存储帐户以获取这些值。你可以单击存储帐户边栏选项卡中的“所有设置”，然后单击“密钥”，找到主访问密钥。

Blob 名称是你想要从中创建的数据库的现有 .bacpac 文件的名称。需要包括 .bacpac 扩展名。

    $StorageName = "storageaccountname"
    $ContainerName = "blobcontainername"
    $BlobName = "filename.bacpac"
    $StorageKey = "primaryaccesskey"

## 创建指向服务器和存储帐户指针

运行 **Get-Credential** cmdlet 会打开一个窗口，要求你输入用户名和密码。请输入要创建数据库的 SQL 服务器的管理员登录名和密码（上述 $ServerName），而非 Azure 帐户的用户名和密码。

    $credential = Get-Credential
    $SqlCtx = New-AzureSqlDatabaseServerContext -ServerName $ServerName -Credential $credential

    $StorageCtx = New-AzureStorageContext -StorageAccountName $StorageName -StorageAccountKey $StorageKey
    $Container = Get-AzureStorageContainer -Name $ContainerName -Context $StorageCtx


## 导入数据库

此命令会将导入数据库请求提交到服务。根据数据库的大小，导入操作可能需要一些时间才能完成。

    $importRequest = Start-AzureSqlDatabaseImport -SqlConnectionContext $SqlCtx -StorageContainer $Container -DatabaseName $DatabaseName -BlobName $BlobName
    

## 监视导入操作的进度

运行 **Start-AzureSqlDatabaseImport** 之后，你可以检查请求的状态。

在请求后立即检查状态通常将返回 **Pending** 或 **Running** 状态并显示当前完成百分比，因此你可以运行此命令多次，直到看到输出中显示 **Status : Completed**。

运行此命令将提示你输入密码。请输入 SQL 服务器的管理员登录名和密码。


    Get-AzureSqlDatabaseImportExportStatus -RequestId $ImportRequest.RequestGuid -ServerName $ServerName -Username $credential.UserName
 


## SQL 数据库 PowerShell 导入脚本


    Add-AzureAccount -Environment AzureChinaCloud
    Select-AzureSubscription -SubscriptionId "4cac86b0-1e56-bbbb-aaaa-000000000000"
    
    $ServerName = "servername"
    $DatabaseName = "nameofnewdatabase"

    $StorageName = "storageaccountname"
    $ContainerName = "blobcontainername"
    $BlobName = "filename.bacpac"
    $StorageKey = "primaryaccesskey"
    
    $credential = Get-Credential
    $SqlCtx = New-AzureSqlDatabaseServerContext -ServerName $ServerName -Credential $credential
    
    $StorageCtx = New-AzureStorageContext -StorageAccountName $StorageName -StorageAccountKey $StorageKey
    $Container = Get-AzureStorageContainer -Name $ContainerName -Context $StorageCtx
    
    $ImportRequest = Start-AzureSqlDatabaseImport -SqlConnectionContext $SqlCtx -StorageContainer $Container -DatabaseName $DatabaseName -BlobName $BlobName
    
    Get-AzureSqlDatabaseImportExportStatus -RequestId $ImportRequest.RequestGuid -ServerName $ServerName -Username $credential.UserName
    

## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)




## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0606_2016-->
