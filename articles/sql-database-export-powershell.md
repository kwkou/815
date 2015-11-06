<properties 
    pageTitle="使用 PowerShell 创建和导出 Azure SQL 数据库的 BACPAC" 
    description="使用 PowerShell 创建和导出 Azure SQL 数据库的 BACPAC" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="09/05/2015"
	ms.topic="article"
	wacn.date="10/17/2015"/>


# 使用 PowerShell 创建和导出 SQL 数据库的 BACPAC

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-export-powershell)


本文介绍如何通过 PowerShell 手动导出 SQL 数据库的 BACPAC。

BACPAC 是包含数据库架构和数据的 .bacpac 文件。有关详细信息，请参阅[数据层应用程序](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx)中的备份包 (.bacpac)。

> [AZURE.NOTE]Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

BACPAC 导出到 Azure 存储 blob 容器中，你可以在操作成功完成后进行下载。


若要完成本文，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“免费试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)文章中的步骤创建一个。
- 具有 blob 容器的 [Azure 存储帐户](/documentation/articles/storage-create-storage-account)可存储 BACPAC。目前的存储帐户必须使用经典部署模型，因此在创建存储帐户时请选择“经典”。
- Azure PowerShell。你可以通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)下载并安装 Azure PowerShell 模块。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。



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
    $DatabaseName = "nameofdatabasetobackup"
    $BlobName = "filename.bacpac"

在 [Azure 预览门户](https://manage.windowsazure.cn)中，浏览到你的存储帐户以获取这些值。你可以单击存储帐户边栏选项卡中的“所有设置”，然后单击“密钥”，找到主访问密钥。

    $StorageName = "storageaccountname"
    $ContainerName = "blobcontainername"
    $StorageKey = "primaryaccesskey"

## 创建指向服务器和存储帐户指针

运行 **Get-Credential** cmdlet 会打开一个窗口，要求你输入用户名和密码。请输入 SQL 服务器的管理员登录名和密码（而非 Azure 帐户的用户名和密码）。

    $credential = Get-Credential
    $SqlCtx = New-AzureSqlDatabaseServerContext -ServerName $ServerName -Credential $credential

    $StorageCtx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageName -StorageAccountKey $StorageKey
    $Container = Get-AzureStorageContainer -Name $ContainerName -Context $StorageCtx


## 导出数据库

此命令会将导出数据库请求提交到服务。根据数据库的大小，导出操作可能需要一些时间才能完成。

    $exportRequest = Start-AzureSqlDatabaseExport -SqlConnectionContext $SqlCtx -StorageContainer $Container -DatabaseName $DatabaseName -BlobName $BlobName
    

## 监视导出操作的进度

运行 **Start-AzureSqlDatabaseExport** 之后，你可以检查请求的状态。在请求后立即运行此命令通常将返回 **Status : Pending** 或 **Status : Running**，因此你可以运行此命令多次，直到看到输出中显示 **Status : Completed**。

运行此命令将提示你输入密码。请输入 SQL 服务器的管理员密码。


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

- [导入 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell)


## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)
- [SQL 数据库文档](/documentation/services/sql-databases/)

<!---HONumber=74-->