<properties 
   pageTitle="SQL 数据库灾难恢复" 
   description="了解在发生区域性的数据中心中断或故障后，如何使用 Azure SQL 数据库活动异地复制和异地还原功能来恢复数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="05/10/2016"
   wacn.date="06/14/2016"/>

# 还原 Azure SQL 数据库或故障转移到辅助数据库

Azure SQL 数据库提供以下功能，以便在服务中断后进行恢复：

- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)

若要了解有关应对灾难以及何时恢复数据库的信息，请访问[业务连续性设计](/documentation/articles/sql-database-business-continuity-design/)页。

## 何时启动恢复

恢复操作会影响应用程序。它要求更改 SQL 连接字符串，并可能会导致数据永久丢失。因此，仅当中断的持续时间超过了应用程序的 RTO 时，才应执行恢复操作。如果应用程序已部署到生产环境，你应该定期监视应用程序的运行状况，并使用以下数据点来声明有必要进行恢复：

1.	应用程序层与数据库之间的连接发生永久性故障。
2.	Azure 管理门户显示了警报，指出区域中的某个事件会造成广泛的影响。
3.	Azure SQL 数据库服务器标记为已降级。 

根据应用程序的停机容忍度和可能的业务责任，可以考虑下列恢复选项。

## 等待服务恢复

Azure 团队会努力尽快还原服务可用性，但视根本原因而定，有可能需要数小时或数天的时间。如果你的应用程序可以容忍长时间停机，则可以等待恢复完成。在此情况下，你不需要采取任何操作。在区域恢复后，应用程序的可用性将会还原。

## 故障转移到异地复制的辅助数据库

如果应用程序停机可能会带来业务责任，则应当在应用程序中使用异地复制的数据库。这样，应用程序在发生中断时，就可以快速还原其他区域的可用性。

若要还原数据库的可用性，必须使用其中一种受支持的方法，启动到异地复制的辅助数据库的故障转移。


请参考下列指南之一，故障转移到异地复制的辅助数据库：

- [使用 PowerShell 故障转移到异地复制的辅助数据库](/documentation/articles/sql-database-geo-replication-powershell/)
- [使用 T-SQL 故障转移到异地复制的辅助数据库](/documentation/articles/sql-database-geo-replication-transact-sql/) 



## 使用异地还原进行恢复

如果应用程序停机不会带来业务责任，则可以使用异地还原作为恢复应用程序数据库的方法。它会从其最新的异地冗余备份创建数据库的副本。

请参考下列指南之一，将数据库异地还原到新的区域：

- [使用 PowerShell 将数据库异地还原到新的区域](/documentation/articles/sql-database-geo-restore-powershell/) 


## 恢复后配置数据库

如果使用异地还原选项的异地复制故障转移，在服务中断后恢复应用程序，则必须确保已正确配置与新数据库的连接，以便恢复正常的应用程序功能。以下任务清单可帮助你准备好将恢复的数据库投入生产。

### 更新连接字符串

因为恢复的数据库将位于不同的服务器中，所以必须更新应用程序的连接字符串以指向该服务器。

有关更改连接字符串的详细信息，请参阅[与 Azure SQL 数据库的连接：中心建议](/documentation/articles/sql-database-connect-central-recommendations/)。

### 配置防火墙规则

需确保服务器和数据库上配置的防火墙规则与主服务器和主数据库上配置的防火墙规则匹配。有关详细信息，请参阅[如何：配置防火墙设置（Azure SQL 数据库）](/documentation/articles/sql-database-configure-firewall-settings-powershell/)。


### 配置登录名和数据库用户

需确保应用程序使用的所有登录名都存在于托管已恢复数据库的服务器上。有关详细信息，请参阅“如何在灾难恢复期间管理安全性”。有关详细信息，请参阅[异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config/)

>[AZURE.NOTE] 如果使用异地还原选项在服务中断后恢复，应在 DR 演练期间配置服务器防火墙规则和登录，以确保主服务器仍可用于检索其配置。因为异地还原会使用数据库备份，所以在服务中断期间可能无法使用此服务器级别配置。演练之后，可以删除已还原的数据库，但保留服务器及其配置，以供恢复过程使用。有关 DR 演练的详细信息，请参阅[执行灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills/)。

### 设置遥测警报

需确保更新现有的警报规则设置，以便映射到恢复的数据库和不同的服务器。

有关数据库警报规则的详细信息，请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)和[跟踪服务运行状况](/documentation/articles/insights-service-health/)。

### 启用审核

如果需要通过审核来访问数据库，你需要在恢复数据库后启用审核。如果客户端应用程序使用 *.database.secure.chinacloudapi.cn 模式的安全连接字符串，就充分表明需要审核。有关详细信息，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started/)。




## 其他资源


- [SQL 数据库业务连续性和灾难恢复](/documentation/articles/sql-database-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config/)
- [SQL 数据库 BCDR 常见问题](/documentation/articles/sql-database-bcdr-faq/)

<!---HONumber=Mooncake_0530_2016-->
