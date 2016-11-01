<properties 
   pageTitle="SQL 数据库灾难恢复" 
   description="了解在发生区域性的数据中心中断或故障后，如何使用 Azure SQL 数据库活动异地复制和异地还原功能来恢复数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="CarlRabeler" 
   manager="jhubbard" 
   editor="monicar"/>  


<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA" 
   ms.date="10/13/2016"
   wacn.date="10/31/2016"
   ms.author="carlrab"/>

# 还原 Azure SQL 数据库或故障转移到辅助数据库

Azure SQL 数据库提供以下功能，以便在服务中断后进行恢复：

- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [异地还原](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore)

若要了解业务连续性方案以及支持这些方案的功能，请参阅 [业务连续性](/documentation/articles/sql-database-business-continuity/)。

### 准备好应对中断情况

为了使用活动异地复制或异地冗余备份成功恢复到其他数据区域，需要为下一次数据中心中断准备服务器，以便在需要时使其成为新的主服务器，还需要记录、测试各项明确定义的步骤，确保顺利恢复数据。准备步骤包括：

- 标识其他区域中要成为新的主服务器的逻辑服务器。使用活动异地复制时，至少标识一个辅助服务器（可能需要标识每个辅助服务器）。对于异地还原，这个服务器通常位于数据库所在区域的[配对区域]（中国东部和中国北部）中。
- 标识（并选择性定义）用户访问新的主数据库时所需的服务器级防火墙规则。
- 确定要如何将用户重定向到新的主服务器，例如通过更改连接字符串或更改 DNS 条目。
- 标识（并选择性创建）新主服务器的 master 数据库中必须存在的登录信息，并确保这些登录信息在 master 数据库中具有相应权限（若有）。相关详细信息，请参阅[灾难恢复后的 SQL 数据库安全性](/documentation/articles/sql-database-geo-replication-security-config/)
- 标识需要更新才可映射到新的主数据库的警报规则。
- 记录当前主数据库上的审核配置
- 执行[灾难恢复演练](/documentation/articles/sql-database-disaster-recovery-drills/)。若要模拟中断情况进行异地还原，可删除或重命名源数据库以引发应用程序连接失败。若要模拟中断进行活动异地复制，可禁用连接到数据库的 Web 应用程序或虚拟机，或者故障转移数据库以引发应用程序连接失败。

## 何时启动恢复

恢复操作会影响应用程序。需更改 SQL 连接字符串或使用 DNS 重定向，可能导致参数数据丢失。因此，仅当中断的持续时间可能超过应用程序的恢复时间目标时，才应执行此操作。如果应用程序已部署到生产环境，你应该定期监视应用程序的运行状况，并使用以下数据点来声明有必要进行恢复：

1.	应用程序层与数据库之间的连接发生永久性故障。
2.	Azure 经典管理门户显示了警报，指出区域中的某个事件造成广泛影响。
3.	Azure SQL 数据库服务器标记为“已降级”。

根据应用程序的停机容忍度和可能的业务责任，可以考虑下列恢复选项。

使用[获取可恢复数据库](https://msdn.microsoft.com/zh-cn/library/dn800985.aspx) (*LastAvailableBackupDate*) 获取最新的异地复制还原点。

## 等待服务恢复

Azure 团队会努力尽快还原服务可用性，但视根本原因而定，有可能需要数小时或数天的时间。如果你的应用程序可以容忍长时间停机，则可以等待恢复完成。在此情况下，你不需要采取任何操作。可在 [Azure 服务运行状况仪表板](/support/service-dashboard/)上查看当前服务状态。在区域恢复后，应用程序的可用性将会还原。

## 故障转移到异地复制的辅助数据库

如果应用程序停机可能会带来业务责任，则应当在应用程序中使用异地复制的数据库。这样，应用程序在发生中断时，就可以快速还原其他区域的可用性。了解如何[配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。

若要还原数据库的可用性，必须使用其中一种受支持的方法，启动到异地复制的辅助数据库的故障转移。

请参考下列指南之一，故障转移到异地复制的辅助数据库：

- [使用 PowerShell 故障转移到异地复制的辅助数据库](/documentation/articles/sql-database-geo-replication-powershell/)
- [使用 T-SQL 故障转移到异地复制的辅助数据库](/documentation/articles/sql-database-geo-replication-transact-sql/) 

## 使用异地还原进行恢复

如果应用程序停机不会带来业务责任，则可以使用异地还原作为恢复应用程序数据库的方法。它会从其最新的异地冗余备份创建数据库的副本。

请参考下列指南之一，将数据库异地还原到新的区域：

- [使用 PowerShell 将数据库异地还原到新的区域](/documentation/articles/sql-database-geo-restore-powershell/) 

## 恢复后配置数据库

在服务中断后，如果你使用异地复制进行故障转移或使用异地还原进行恢复，则必须确保已正确配置与新数据库的连接，以便恢复正常的应用程序功能。以下任务清单可帮助你准备好将恢复的数据库投入生产。

### 更新连接字符串

因为恢复的数据库将位于不同的服务器中，所以必须更新应用程序的连接字符串以指向该服务器。

若要深入了解如何更改连接字符串，请参阅[连接库](/documentation/articles/sql-database-libraries/)的相应开发语言。

### 配置防火墙规则

需确保服务器和数据库上配置的防火墙规则与主服务器和主数据库上配置的防火墙规则匹配。有关详细信息，请参阅[如何：配置防火墙设置（Azure SQL 数据库）](/documentation/articles/sql-database-configure-firewall-settings-powershell/)。


### 配置登录名和数据库用户

需确保应用程序使用的所有登录名都存在于托管已恢复数据库的服务器上。相关详细信息，请参阅[异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config/)

>[AZURE.NOTE] 应在灾难恢复演练期间配置并测试服务器防火墙规则和登录（及其权限）。在服务中断期间，这些服务器级对象及其配置可能不可用。

### 设置遥测警报

需确保更新现有的警报规则设置，以便映射到恢复的数据库和不同的服务器。

有关数据库警报规则的详细信息，请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)和[跟踪服务运行状况](/documentation/articles/insights-service-health/)。

### 启用审核

如果需要通过审核来访问数据库，你需要在恢复数据库后启用审核。如果客户端应用程序使用 *.database.secure.chinacloudapi.cn 模式的安全连接字符串，就充分表明需要审核。有关详细信息，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started/)。


## 后续步骤

- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)
- 若要了解业务连续性设计和恢复方案，请参阅[连续性方案](/documentation/articles/sql-database-business-continuity/)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)

<!---HONumber=Mooncake_1024_2016-->