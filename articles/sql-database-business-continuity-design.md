<properties 
   pageTitle="SQL 数据库的业务连续性设计" 
   description="选择指南.本部分将提供有关如何选择要使用的 BCDR 功能以及何时使用这些功能的指南。这包括使用 SQL DB 自动获取的内容的说明。"
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="04/25/2016"
   wacn.date="06/14/2016"/>

#业务连续性设计

针对业务连续性设计应用程序需要你回答以下问题：

1. 哪种业务连续性功能适合用于防止我的应用程序发生中断？
2. 我要使用哪个级别的冗余和复制拓扑？

##何时使用异地还原

默认情况下，SQL 数据库为每个数据库提供内置的基本保护。具体的保护方式是，将数据库备份存储在异地冗余的 Azure 存储空间 (GRS) 中。如果你选择此方法，则不需要进行特殊的配置或其他资源分配。借助这些备份，你可以使用异地还原命令来恢复任何区域中的数据库。[在中断后恢复](/documentation/articles/sql-database-disaster-recovery/)部分中介绍了有关使用异地还原恢复应用程序的详细信息。

如果你的应用程序符合以下条件，则你应该使用内置保护功能：

1. 它不是任务关键型应用程序。它没有约束性的 SLA，也就是说，停机 24 小时或更长时间不会导致财务责任。
2. 数据更改频率（例如，每小时事务数）较低。1 小时的 RPO 不会导致丢失大量数据。
3. 应用程序成本敏感，并且无法判断异地复制的额外成本 

> [AZURE.NOTE] 在任何特定的区域，中断期间异地还原不会预先分配计算容量从备份中还原活动的数据库。服务管理与异地还原请求关联的工作负荷的方式是，最大程度地降低对该区域中现有数据库的影响，而这些数据库的容量需求具有优先级。因此，数据库恢复时间将取决于同一区域中同一时间要恢复的其他数据库数量。

##何时使用异地复制

异地复制将在不同于你的主要位置的其他区域中创建副本数据库（辅助数据库）。它可以保证你的数据库在恢复后，具有必要的数据和计算资源来支持应用程序的工作负荷。有关使用故障转移恢复应用程序的信息，请参阅[在中断后恢复](/documentation/articles/sql-database-disaster-recovery/)部分。

如果你的应用程序符合以下条件，则你应该使用异地复制：

1. 它是任务关键型应用程序。它具有严格规定了 RPO 和 RTO 的约束性 SLA。丢失数据和可用性会导致财务责任。 
2. 数据更改频率（例如每分钟或每秒事务数）较高。使用默认保护时发生 1 小时的 RPO 很可能会导致不可接受的数据丢失。
3. 使用异地复制所涉及的成本明显低于潜在财务责任和相关业务损失所付出的代价。


> [AZURE.NOTE] 活动异地复制还支持对辅助数据库进行只读访问，因此为只读工作负荷提供了额外的容量。

##如何启用异地复制

你可以使用 Azure 管理门户或通过调用 REST API 或 PowerShell 命令来启用异地复制。


###Azure 门户

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”
3. 导航到数据库边栏选项卡，选择“异地复制”映射，然后单击“配置异地复制”。
4. 导航到“异地复制”边栏选项卡。选择目标区域。 
5. 导航到“创建辅助数据库”边栏选项卡。选择目标区域中的现有服务器，或创建一个新的服务器。
6. 选择辅助数据库类型（“可读”或“不可读”）
7. 单击“创建”以完成配置

> [AZURE.NOTE] “异地复制”边栏选项卡上的 DR 配对区域将标记为*推荐*。如果使用的是高级层数据库，你可以选择不同的区域。如果使用的是标准数据库，则无法更改区域。高级数据库允许你选择辅助数据库类型（“可读”或“不可读”）。标准数据库只允许你选择“不可读”的辅助数据库。
 
###PowerShell

使用 [New-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603689.aspx) PowerShell cmdlet 可以创建异地复制配置。此命令是同步的，并在主数据库和辅助数据库同步后返回。

为不可读辅助数据库配置异地复制：
		
    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "None"

为可读辅助数据库创建异地复制：

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "All"
		 

###REST API 

在 createMode 设置为 NonReadableSecondary 或 Secondary 的情况下使用[创建数据库](https://msdn.microsoft.com/zh-cn/library/mt163685.aspx) API 可以编程方式创建异地复制辅助数据库。

此 API 是异步的。在它返回后，请使用[获取复制链接](https://msdn.microsoft.com/zh-cn/library/mt600778.aspx) API 检查此操作的状态。完成操作后，响应正文的 replicationState 字段将包含值 CATCHUP。


##如何选择故障转移配置 

在针对业务连续性设计应用程序时，应考虑多个配置选项。所做的选择取决于应用程序部署拓扑，以及应用程序的哪些组件最容易中断。有关指导，请参阅[使用异地复制设计云解决方案以实现灾难恢复](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)。

<!---HONumber=Mooncake_0530_2016-->
