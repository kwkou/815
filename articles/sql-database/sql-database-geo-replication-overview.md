<properties
	pageTitle="Azure SQL 数据库的活动异地复制"
	description="你可以使用活动异地复制在任何 Azure 数据中心设置 4 个数据库副本。"
	services="sql-database"
	documentationCenter="na"
	authors="stevestein"
	manager="jhubbard"
	editor="monicar" />  



<tags
	ms.service="sql-database"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="NA"
	ms.date="09/26/2016"
	wacn.date="10/31/2016"
	ms.author="sstein" />

# 概述：SQL 数据库的活动异地复制

使用活动异地复制可在相同或不同数据中心位置（区域）中最多配置四个可读的辅助数据库。在数据中心发生服务中断或无法连接到主数据库时，可以使用辅助数据库进行查询和故障转移。

>[AZURE.NOTE] 活动异地复制（可读辅助数据库）现在可供所有服务层中的所有数据库使用。

 可以使用 [PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)、[Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/) 或 [REST API - 创建或更新数据库](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)配置活动异地复制。

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-replication-overview/)/
- [配置：PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)
- [配置：T-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/)

如果主数据库因某种原因而出现故障，或者只是需要脱机，则可以*故障转移*到任何辅助数据库。激活故障转移以后，即可将主数据库故障转移到某个辅助数据库，然后所有其他辅助数据库就会自动链接到新的主数据库。

可以使用 [PowerShell](/documentation/articles/sql-database-geo-replication-failover-powershell/)、[Transact-SQL](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)、[REST API - 计划故障转移](https://msdn.microsoft.com/zh-cn/ibrary/azure/mt575007.aspx)  或 [REST API - 非计划故障转移](https://msdn.microsoft.com/zh-cn/library/azure/mt582027.aspx)故障转移到辅助数据库。


> [AZURE.SELECTOR]
- [故障转移：PowerShell](/documentation/articles/sql-database-geo-replication-failover-powershell/)
- [故障转移：T-SQL](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)

故障转移后，请确保在新的主机上配置服务器和数据库的身份验证要求。有关详细信息，请参阅 [SQL Database security after disaster recovery](/documentation/articles/sql-database-geo-replication-security-config/)（灾难恢复后的 Azure SQL 数据库安全性）。


活动异地复制功能实现了一个机制，用于在同一 Azure 区域或不同区域中提供数据库冗余（异地冗余）。活动异地复制使用用于隔离的读取已提交快照隔离 (RCSI) 以异步方式将已提交的事务从数据库复制到不同服务器上的最多四个数据库副本中。在配置活动异地复制时，会在指定的服务器上创建辅助数据库。原始数据库将变成主数据库。主数据库以异步方式将已提交的事务复制到每个辅助数据库。只会复制完整事务。尽管在任意给定时间，辅助数据库可能略微滞后于主数据库，但系统可以保证辅助数据永不包含部分事务。[Overview of Business Continuity](/documentation/articles/sql-database-business-continuity/)（业务连续性概述）中提供了具体的 RPO 数据。

活动异地复制的主要优势之一在于能够提供数据库级别的灾难恢复解决方案且恢复时间较短。将辅助数据库放在不同区域中的服务器上时，可以为应用程序增添极大的恢复能力。跨区域冗余使应用程序能够在自然灾害、灾难性人为失误或恶意行为导致整个或部分数据中心永久性数据丢失后得以恢复。下图显示了配置的活动异地复制示例，其中主数据库位于中国北部区域，辅助数据库位于中国东部区域。



另一个关键优势是辅助数据库是可读的，并且可用于卸载只读工作负荷，如报表作业。如果只想使用辅助数据库进行负载均衡，可以在主数据库所在的同一区域中创建它。在同一区域中创建辅助数据库不会增加应用程序灾难性故障的恢复能力。

可以用到活动异地复制的其他场合包括：

- **数据库迁移**：可以使用活动异地复制将数据库从一台服务器联机迁移到另一台服务器，且只会造成极少量的停机时间。
- **应用程序升级**：可以在应用程序升级期间创建额外的辅助数据库作为故障回复副本。

若要真正实现业务连续性，只需添加数据中心之间的数据库冗余性即可，这只是该解决方案的一部分功能。在发生灾难性故障后，端对端地恢复应用程序（服务）需要恢复构成该服务的所有组件以及所有依赖服务。这些组件的示例包括客户端软件（例如，使用自定义 JavaScript 的浏览器）、Web 前端、存储和 DNS。所有组件必须能够弹性应对相同的故障，并在应用程序的恢复时间目标 (RTO) 值内变为可用，这一点非常关键。因此，你需要识别所有依赖服务，并了解它们提供的保证和功能。然后，必须执行适当的步骤来确保对你的服务所依赖的服务执行故障转移期间，你的服务能够正常运行。有关设计用于灾难恢复的解决方案的详细信息，请参阅[使用活动异地复制设计灾难恢复云解决方案](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)。

## 活动异地复制的功能
活动异地复制提供以下基本功能：

- **自动异步复制**：只能通过添加到现有数据库创建辅助数据库。辅助数据库只可在不同的 Azure SQL 数据库服务器中创建。创建完成之后，辅助数据库将填充从主数据库复制的数据。这个过程称为种子设定。创建辅助数据库并设定其种子后，会自动以异步方式将主数据库发生的更新复制到辅助数据库。异步复制是指先在主数据库上提交事务，然后将事务复制到辅助数据库。

- **多个辅助数据库**：两个或更多个辅助数据库可以提高主数据库和应用程序的冗余和保护级别。如果存在多个辅助数据库，即使其中一个辅助数据库发生故障，应用程序仍会受到保护。如果只有一个辅助数据库，一旦它发生故障，应用程序就会遭受更高的风险，直到创建了新的辅助数据库。

- **可读的辅助数据库**：应用程序可以使用与访问主数据库时所用的相同或不同的安全主体来访问辅助数据库以执行只读操作。辅助数据库在快照隔离模式下运行，以确保对主数据库的更新的复制（日志重播）不会因辅助数据库上执行的查询操作而延迟。

>[AZURE.NOTE] 如果日志重播正在从主数据库接收的内容存在架构更新，则它将在辅助数据库上延迟，因为此过程需要辅助数据库上的架构锁。

- **弹性池数据库的活动异地复制**：可以为任何弹性数据库池中的任何数据库配置活动异地复制。辅助数据库可以位于另一个弹性数据库池。对于常规的数据库，辅助数据库可以是弹性数据库池，反过来也一样，只要服务层相同。

- **辅助数据库的可配置性能级别**：可以使用低于主数据库的性能级别创建辅助数据库。主数据库和辅助数据库都需要有相同的服务层。不建议将此选项用于具有高数据库写入活动的应用程序，因为复制延迟时间的增长会加大故障转移后大量数据丢失的风险。此外，在故障转移后，应用程序的性能将受到影响，直到新的主数据库升级到更高的性能级别。在 Azure 门户上的日志 IO 百分比图表提供了一种评估维持复制负荷所需的辅助数据库的最小性能级别的好办法。例如，如果你的主数据库是 P6 (1000 DTU)，其日志 IO 百分比为 50%，则辅助数据库需要至少应为 P4 (500 DTU)。你还可以使用 [sys.resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx) 或 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 数据库视图检索日志 IO 数据。有关 SQL 数据库性能级别的详细信息，请参阅 [SQL Database options and performance](/documentation/articles/sql-database-service-tiers/)（SQL 数据库选项和性能）。

- **用户控制的故障转移和故障回复**：应用程序或用户可随时将辅助数据库显式切换到主角色。在实际中断期间，应使用“计划外”选项，这会立即将辅助数据库升级为主数据库。当出现故障的主数据库恢复并再次可用时，系统会自动将恢复的主数据库标记为辅助数据库并使其与新的主数据库保持最新状态。由于复制的异步特性，在计划外故障转移期间，如果主数据库在将最新的更改复制到辅助数据库之前出现故障，则可能会丢失少量数据。当具有多个辅助数据库的主数据库进行故障转移时，系统将自动重新配置复制关系，并将剩余辅助数据库链接到新升级的主数据库，无需任何用户的干预。导致了故障转移的中断得到缓解后，可能需要将应用程序返回到主要区域。为此，应使用“计划内”选项调用故障转移命令。

- **保持凭据和防火墙规则同步**：建议将[数据库防火墙规则](/documentation/articles/sql-database-firewall-configure/)用于异地复制数据库，以便这些规则可以和数据库一起复制，从而确保所有辅助数据库具有与主数据库相同的防火墙规则。这样就不再需要客户手动配置和维护承载主数据库和辅助数据库的服务器上的防火墙规则。同样，将[包含的数据库用户](/documentation/articles/sql-database-manage-logins/)用于数据访问可确保主数据库和辅助数据库始终具有相同的用户凭据，以便在故障转移期间，不会因登录名和密码不匹配而产生中断。通过添加 [Azure Active Directory](/documentation/articles/active-directory-whatis/)，客户可以管理主数据库和辅助数据库的用户访问权限，且不再需要同时管理数据库中的凭据。

## 升级或降级主数据库
无需断开连接任何辅助数据库，即可将主数据库升级或降级到不同的性能级别（在相同的服务层中）。升级时，建议先升级辅助数据库，然后再升级主数据库。降级时，应反转顺序：先降级主数据库，然后降级辅助数据库。

辅助数据库必须与主数据库位于相同的服务层中，因此，将主数据库迁移到不同的服务层需要终止异地复制链接并重命名辅助数据库，或直接删除辅助数据库。然后，将主数据库迁移到新服务层并重新配置异地复制。默认情况下，将自动创建新的辅助数据库，它的性能级别与主数据库相同。

## 防止丢失关键数据
由于广域网的延迟时间较长，连续复制使用了异步复制机制。在发生故障时，异步复制会不可避免地丢失某些数据。但是，某些应用程序可能要求不能有数据丢失。为了保护这些关键更新，应用程序开发人员可以在提交事务后立即调用 [sp\_wait\_for\_database\_copy\_sync](https://msdn.microsoft.com/zh-cn/library/dn467644.aspx) 系统过程。调用 **sp\_wait\_for\_database\_copy\_sync** 会阻止调用线程，直到将上次提交的事务复制到辅助数据库。该过程将一直等待，直到辅助数据库确认所有排队的事务。**sp\_wait\_for\_database\_copy\_sync** 限于特定的连续复制链接。对主数据库具有连接权限的任何用户都可以调用此过程。

>[AZURE.NOTE] **sp\_wait\_for\_database\_copy\_sync** 过程调用导致的延迟可能会很明显。延迟取决于当前事务日志长度的大小，此调用在复制整个日志之前不会返回。除非绝对必要，否则请避免调用此过程。

## 以编程方式管理活动异地复制

如上所述，也可以使用 Azure PowerShell 和 REST API 以编程方式管理活动异地复制。下表描述了可用的命令集。

- **Azure Resource Manager API 和基于角色的安全性**：活动异地复制包括一组用于管理的 [Azure Resource Manager API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx)，其中包括[基于 Azure Resource Manager 的 PowerShell cmdlet](/documentation/articles/sql-database-geo-replication-powershell/)。这些 API 需要使用资源组，并支持基于角色的安全 (RBAC)。有关如何实现访问角色的详细信息，请参阅 [Azure Role-Based Access Control](/documentation/articles/role-based-access-control-configure/)（Azure 基于角色的访问控制）。

>[AZURE.NOTE] 只有使用基于 [Azure Resource Manager](/documentation/articles/resource-group-overview/) 的 [Azure SQL REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx) 和 [Azure SQL 数据库 PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt574084.aspx) 才支持活动异地复制的许多新功能。向后兼容性支持现有[（经典）REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx) 和 [Azure SQL 数据库（经典）cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn546723.aspx)，因此建议使用基于 Azure Resource Manager 的 API。


### Transact-SQL

|命令|说明|
|-------|-----------|
|[ALTER DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx)|使用 ADD SECONDARY ON SERVER 参数为现有数据库创建辅助数据库，并开始数据复制|
|[ALTER DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx)|使用 FAILOVER 或 FORCE\_FAILOVER\_ALLOW\_DATA\_LOSS 将辅助数据库切换为主数据库，启动故障转移
|[ALTER DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx)|使用 REMOVE SECONDARY ON SERVER 终止 SQL 数据库和指定的辅助数据库之间的数据复制。|
|[sys.geo\_replication\_links（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575501.aspx)|返回 Azure SQL 数据库逻辑服务器上每个数据库的所有现有的复制链接的信息。|
|[sys.dm\_geo\_replication\_link\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575504.aspx)|获取指定 SQL 数据库的复制链接的上次复制时间、上次复制滞后时间和其他信息。|
|[sys.dm\_operation\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn270022.aspx)|显示所有数据库操作的状态，包括复制链接的状态。|
|[sp\_wait\_for\_database\_copy\_sync（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn467644.aspx)|使应用程序等待，直到所有提交的事务被复制，并由活动次要数据库确认。|
||||

### PowerShell

|Cmdlet|说明|
|------|-----------|
|[Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648.aspx)|获取一个或多个数据库。|
|[New-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603689.aspx)|为现有数据库创建辅助数据库，并开始数据复制。|
|[Set-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt619393.aspx)|将辅助数据库切换为主数据库，启动故障转移。|
|[Remove-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt603457.aspx)|终止 SQL 数据库和指定的辅助数据库之间的数据复制。|
|[Get-AzureRmSqlDatabaseReplicationLink](https://msdn.microsoft.com/zh-cn/library/mt619330.aspx)|获取 Azure SQL 数据库和资源组或 SQL Server 之间的异地复制链接。|
||||

### REST API

|API|说明|
|---|-----------|
|[创建或更新数据库 (createMode=Restore)](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)|创建、更新或还原主数据库或辅助数据库。|
|[获取创建或更新数据库状态](https://msdn.microsoft.com/zh-cn/library/azure/mt643934.aspx)|返回创建操作过程中的状态。|
|[将辅助数据库设为主数据库（计划故障转移）](https://msdn.microsoft.com/zh-cn/ibrary/azure/mt575007.aspx)|提升异地复制合作关系中的辅助数据库，使其成为新的主数据库。|
|[将辅助数据库设为主数据库（非计划故障转移）](https://msdn.microsoft.com/zh-cn/library/azure/mt582027.aspx)|强制故障转移到辅助数据库，并将辅助数据库设为主数据库。|
|[获取复制链接](https://msdn.microsoft.com/zh-cn/library/azure/mt600929.aspx)|获取异地复制合作关系中指定 SQL 数据库的所有复制链接。它检索 sys.geo\_replication\_links 目录视图中可见的信息。|
|[获取复制链接](https://msdn.microsoft.com/zh-cn/library/azure/mt600778.aspx)|获取异地复制合作关系中指定 SQL 数据库的特定复制链接。它检索 sys.geo\_replication\_links 目录视图中可见的信息。|
||||



## 后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)。
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)。
- 若要了解新主服务器和数据库的身份验证要求，请参阅 [SQL Database security after disaster recovery](/documentation/articles/sql-database-geo-replication-security-config/)（灾难恢复后的 SQL 数据库安全性）。

<!---HONumber=Mooncake_1024_2016-->