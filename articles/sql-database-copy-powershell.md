<properties 
    pageTitle="使用 PowerShell 复制 Azure SQL 数据库 | Azure" 
    description="使用 PowerShell 复制 Azure SQL 数据库" 
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/21/2016"
	wacn.date="03/24/2016"/>


# 使用 PowerShell 复制 Azure SQL 数据库

**单一数据库**

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-copy-powershell/)
- [T-SQL](/documentation/articles/sql-database-copy-transact-sql/)



以下步骤说明如何使用 PowerShell 复制 SQL 数据库。数据库复制操作使用 [Start-AzureSqlDatabaseCopy](https://msdn.microsoft.com/zh-cn/library/dn720220.aspx) cmdlet 语句将 SQL 数据库复制到新的数据库。副本是在相同或不同服务器上创建的数据库快照备份。

> [AZURE.NOTE] Azure SQL 数据库会自动为你可以还原的每个用户数据库创建和维护备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

复制过程完成后，新数据库将完全正常工作，并独立于源数据库。完成复制时，新数据库的事务处理方式将与源数据库保持一致。数据库副本的服务层和性能级别（定价层）都与源数据库相同。在完成该复制后，副本将成为能够完全行使功能的独立数据库。登录名、用户和权限可单独进行管理。


将某个数据库复制到同一逻辑服务器时，可以在这两个数据库上使用相同的登录名。用于复制该数据库的安全主体将成为新数据库上的数据库所有者 (DBO)。所有数据库用户、其权限及安全标识符 (SID) 都复制到数据库副本中。


若要完成本文，你需要以下各项：

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用版”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)文章中的步骤创建一个。
- Azure PowerShell。你可以通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)下载并安装 Azure PowerShell 模块。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。



## 配置你的凭据，然后选择你的订阅

首先必须与 Azure 帐户建立访问连接，因此请启动 PowerShell，然后运行以下 cmdlet。在登录屏幕中，输入登录 Azure 经典门户时所用的相同电子邮件和密码。

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中显示的信息中复制订阅 ID，或者，如果你有多个订阅且需要更多详细信息，可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅以后，你可以运行以下 cmdlet：

	Select-AzureSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureSubscription** 后，将返回到 PowerShell 提示符处。如果你有多个订阅，可以运行 **Get-AzureSubscription** 并验证要使用的订阅显示 **IsCurrent: True**。


## 设置适合特定环境的变量

有几个变量需要你将示例值替换为你的数据库和服务器的特定值。

将占位符值替换为你的环境的值：

    # The name of the server on which the source database resides.
    $ServerName = "sourceServerName"

    # The name of the source database (the database to copy). 
    $DatabaseName = "sourceDatabaseName" 
    
    # The name of the server that hosts the target database. This server must be in the same Azure subscription as the source database server. 
    $PartnerServerName = "partnerServerName"

    # The name of the target database (the name of the copy).
    $PartnerDatabaseName = "partnerDatabaseName" 





## 将 SQL 数据库复制到同一台服务器

此命令会将复制数据库请求提交到服务。根据数据库的大小，复制操作可能需要一些时间才能完成。

    # Copy a database to the same server
    Start-AzureSqlDatabaseCopy -ServerName $ServerName -DatabaseName $DatabaseName -PartnerDatabase $PartnerDatabaseName

## 将 SQL 数据库复制到不同的服务器

此命令会将复制数据库请求提交到服务。根据数据库的大小，复制操作可能需要一些时间才能完成。

    # Copy a database to a different server
    Start-AzureSqlDatabaseCopy -ServerName $ServerName -DatabaseName $DatabaseName -PartnerServer $PartnerServerName -PartnerDatabase $PartnerDatabaseName
    

## 监视复制操作的进度

运行 **Start-AzureSqlDatabaseCopy** 之后，你可以检查复制请求的状态。在请求后立即运行此命令通常将返回 **State : Pending** 或 **State : Running**，因此你可以运行此命令多次，直到看到输出中显示 **State : COMPLETED**。


    Get-AzureSqlDatabaseOperation -ServerName $ServerName -DatabaseName $DatabaseName


## 复制 SQL 数据库 PowerShell 脚本

    # The name of the server where the source database resides
    $ServerName = "sourceServerName"

    # The name of the source database (the database to copy) 
    $DatabaseName = "sourceDatabaseName" 
    
    # The name of the server to host the database copy. This server must be in the same Azure subscription as the source database server
    $PartnerServerName = "partnerServerName"

    # The name of the target database (the name of the copy)
    $PartnerDatabaseName = "partnerDatabaseName" 


    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    Select-AzureSubscription -SubscriptionName "myAzureSubscriptionName"
      
    # Copy a database to a different server (remove the -PartnerServer parameter to copy to the same server)
    Start-AzureSqlDatabaseCopy -ServerName $ServerName -DatabaseName $DatabaseName -PartnerServer $PartnerServerName -PartnerDatabase $PartnerDatabaseName
    
    # Monitor the status of the copy
    Get-AzureSqlDatabaseOperation -ServerName $ServerName -DatabaseName $DatabaseName
    

## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)
- [将数据库导出到 BACPAC](/documentation/articles/sql-database-export-powershell/)


## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0307_2016-->
