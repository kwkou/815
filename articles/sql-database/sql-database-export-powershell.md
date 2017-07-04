<properties
    pageTitle="PowerShell：将 Azure SQL 数据库导出到 BACPAC 文件 | Azure"
    description="使用 PowerShell 将 Azure SQL 数据库导出到 BACPAC 文件"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="9439dd83-812f-4688-97ea-2a89a864d1f3"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="sstein"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />  



# 使用 PowerShell 将 Azure SQL 数据库或 SQL Server 导出到 BACPAC 文件

本文说明了如何使用 PowerShell 将 Azure SQL 数据库或 SQL Server 数据库导出到 BACPAC 文件（存储在 Azure Blob 存储中）。如需大致了解如何导出到 BACPAC 文件，请参阅[导出到 BACPAC](/documentation/articles/sql-database-export/)。

> [AZURE.NOTE]
>也可使用 [Azure 门户](/documentation/articles/sql-database-export-portal/)、[SQL Server Management Studio](/documentation/articles/sql-database-export-ssms/) 或 [SQLPackage](/documentation/articles/sql-database-export-sqlpackage/) 将 Azure SQL 数据库文件导出到 BACPAC 文件。
>

## 先决条件

若要完成本文，需要以下各项：

- Azure 订阅。
- Azure SQL 数据库。
- [Azure 标准存储帐户](/documentation/articles/storage-create-storage-account/)，具有可在标准存储中存储 BACPAC 的 blob 容器。


[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 导出数据库
[New-AzureRmSqlDatabaseExport](https://msdn.microsoft.com/zh-cn/library/azure/mt707796(v=azure.300).aspx) cmdlet 向服务提交导出数据库请求。根据数据库的大小，导出操作可能需要一些时间才能完成。

> [AZURE.IMPORTANT] 若要确保获得事务处理一致性 BACPAC 文件，应首先[创建数据库的副本](/documentation/articles/sql-database-copy-powershell/)，然后导出该数据库副本。


     $exportRequest = New-AzureRmSqlDatabaseExport –ResourceGroupName $ResourceGroupName –ServerName $ServerName `
       –DatabaseName $DatabaseName –StorageKeytype $StorageKeytype –StorageKey $StorageKey -StorageUri $BacpacUri `
       –AdministratorLogin $creds.UserName –AdministratorLoginPassword $creds.Password


## 监视导出操作的进度
运行 [New-AzureRmSqlDatabaseExport](https://msdn.microsoft.com/zh-cn/library/azure/mt603644(v=azure.300).aspx) 后，可运行 [Get-AzureRmSqlDatabaseImportExportStatus](https://msdn.microsoft.com/zh-cn/library/azure/mt707794(v=azure.300).aspx) 来查看请求的状态。如果请求后立即运行，通常会返回“状态: 处理中”。显示“状态: 成功”时，表示导出完毕。

    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink



## 导出 SQL 数据库示例
以下示例将现有 SQL 数据库导出到 BACPAC，随后显示如何查看导出操作的状态。

若要运行示例，需将几个变量替换为数据库和存储帐户中的特定值。在 [Azure 门户](https://portal.azure.cn)中，浏览存储帐户，以获取存储帐户名称、blob 容器名称和密钥值。可单击存储帐户边栏选项卡上的“访问密钥”查找密钥。

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
* 若要了解如何使用 Powershell 导入 Azure SQL 数据库，请参阅[使用 PowerShell 导入 BACPAC](/documentation/articles/sql-database-import-powershell/)。
* 若要了解如何使用 SQLPackage 导入 BACPAC，请参阅[使用 SqlPackage 将 BACPAC 导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-sqlpackage/)
* 若要了解如何使用 Azure 门户导入 BACPAC，请参阅[使用 Azure 门户将 BACPAC 导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-portal/)
* 如需 SQL Server 数据库完整迁移过程的介绍（包括性能建议），请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
* 若要了解如何将 BACPAC 导入到 SQL Server 数据库，请参阅[将 BACPCAC 导入 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/hh710052.aspx)



## 其他资源
* [New-AzureRmSqlDatabaseExport](https://msdn.microsoft.com/zh-cn/library/azure/mt707796(v=azure.300).aspx)
* [Get-AzureRmSqlDatabaseImportExportStatus](https://msdn.microsoft.com/zh-cn/library/azure/mt707794(v=azure.300).aspx)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: simplify overview content, update link references-->