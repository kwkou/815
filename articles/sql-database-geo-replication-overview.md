<properties
	pageTitle="Azure SQL 数据库的活动异地复制"
	description="本主题介绍 SQL 数据库的活动异地复制及其用法。"
	services="sql-database"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />


<tags
	ms.service="sql-database"
	ms.date="03/07/2016"
	wacn.date="06/14/2016" />

# Azure SQL 数据库的活动异地复制

## 概述
活动异地复制功能实现了一个机制，用于在同一 Azure 区域或不同区域中提供数据库冗余（异地冗余）。活动异地复制以异步方式将已提交的事务从数据库复制到不同服务器上的最多四个数据库副本中。在配置活动异地复制时，会在指定的服务器上创建辅助数据库。原始数据库将变成主数据库。主数据库以异步方式将已提交的事务复制到每个辅助数据库。尽管在任意给定时间，辅助数据库可能略微滞后于主数据库，但系统可以保证辅助数据在事务处理上始终与提交到主数据库的更改相一致。

活动异地复制的主要优势之一在于能够提供数据库级别的灾难恢复解决方案且恢复时间非常短。当将辅助数据库放在不同区域中的服务器上时，你可以为应用程序增添极大的恢复能力。跨区域冗余使应用程序能够在自然灾害、灾难性人为失误或恶意行为导致整个或部分数据中心永久性数据丢失后得以恢复。下图显示了在高级数据库上配置的活动异地复制示例，其中主数据库位于中国北部区域，而辅助数据库位于中国东部区域。

另一个关键优势是辅助数据库是可读的，并且可用于卸载只读工作负荷，如报表作业。如果只想使用辅助数据库进行负载平衡，你可以在主数据库所在的同一区域中创建它。然而，这将不会增加应用程序的灾难性故障的恢复能力。

可以用到活动异地复制的其他场合包括：

- **数据库迁移**：可以使用活动异地复制将数据库从一台服务器联机迁移到另一台服务器，且只会造成极少量的停机时间。
- **应用程序升级**：可以在应用程序升级期间创建额外的辅助数据库作为故障回复副本。

若要真正实现业务连续性，只需添加数据中心之间的数据库冗余性即可，这只是该解决方案的一部分功能。在发生灾难性故障后，端对端地恢复应用程序（服务）需要恢复构成该服务的所有组件以及所有依赖服务。这些组件的示例包括客户端软件（例如，使用自定义 JavaScript 的浏览器）、Web 前端、存储和 DNS。所有组件必须能够弹性应对相同的故障，并在应用程序的恢复时间目标 (RTO) 值内变为可用，这一点非常关键。因此，你需要识别所有依赖服务，并了解它们提供的保证和功能。然后，必须执行适当的步骤来确保对你的服务所依赖的服务执行故障转移期间，你的服务能够正常运行。有关设计用于灾难恢复的解决方案的详细信息，请参阅[使用活动异地复制设计灾难恢复云解决方案](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)。

## 活动异地复制的功能
活动异地复制提供以下基本功能：

- **自动异步复制**：只能通过添加到现有数据库创建辅助数据库。辅助数据库只可在不同的 Azure SQL 数据库服务器中创建。创建完成之后，辅助数据库将填充从主数据库复制的数据。这个过程称为种子设定。创建辅助数据库并设定其种子后，会自动以异步方式将主数据库发生的更新复制到辅助数据库。也就是说，会先在主数据库上提交事务，然后将事务复制到辅助数据库。SQL 数据库可保证完成种子设定后，辅助数据库在事务上仍始终保持一致。有关创建辅助数据库的详细信息，请参阅[使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)和[使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。

- **多个辅助数据库**：两个或更多个辅助数据库可以提高主数据库和应用程序的冗余和保护级别。如果存在多个辅助数据库，即使其中一个辅助数据库发生故障，应用程序仍会受到保护。如果只有一个辅助数据库，一旦它发生故障，应用程序就会遭受更高的风险，直到创建了新的辅助数据库。

- **可读的辅助数据库**：应用程序可以使用访问主数据库时所用的相同或不同的安全主体来访问辅助数据库以执行只读操作。辅助数据库在快照隔离模式下运行，以确保对主数据库的更新的复制（日志重播）不会因辅助数据库上执行的查询操作而延迟。

>[AZURE.NOTE] 如果日志重播正在从主数据库接收的内容存在架构更新，则它将在辅助数据库上延迟，因为此过程需要辅助数据库上的架构锁。

- **弹性池数据库的活动异地复制**：可以为高级弹性数据库池中的任何数据库配置活动异地复制。辅助数据库可以位于另一个高级弹性数据库池。对于常规的数据库，辅助数据库可以是弹性数据库池，反过来也一样，只要服务层相同。有关如何配置弹性池数据库的异地复制的详细信息，请参阅[使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)和[使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。  

- **辅助数据库的可配置性能级别**：可以使用低于主数据库的性能级别创建辅助数据库。主数据库和辅助数据库都需要有相同的服务层。不建议将此选项用于具有高数据库写入活动的应用程序，因为它可能会导致复制延迟时间增长，因此在故障转移后具有大量数据丢失的高风险。此外，在故障转移后，应用程序的性能将受到影响，直到新的主数据库升级到更高的性能级别。在 Azure 门户上的日志 IO 百分比图表提供了一种评估维持复制负荷所需的辅助数据库的最小性能级别的好办法。例如，如果你的主数据库是 P6 (1000 DTU)，其日志 IO 百分比为 50%，则辅助数据库需要至少应为 P4 (500 DTU)。你还可以使用 [sys.resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx) 或 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 数据库视图检索日志 IO 数据。有关 SQL 数据库性能级别的详细信息，请参阅 [SQL 数据库选项和性能](/documentation/articles/sql-database-service-tiers/)。有关如何配置辅助数据库示例的性能级别的详细信息，请参阅[使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)和[使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。

- **用户控制的故障转移和故障回复**：辅助数据库可以通过应用程序或用户的显示操作，随时切换到主角色。在实际中断期间，应使用“计划外”选项，这会立即将辅助数据库升级为主数据库。当出现故障的主数据库恢复并再次可用时，系统会自动将其标记为辅助数据库并使其与新的主数据库保持最新状态。由于复制的异步特性，在计划外故障转移期间，如果主数据库在将最新的更改复制到辅助数据库之前出现故障，则可能会丢失少量数据。当具有多个辅助数据库的主数据库进行故障转移时，系统将自动重新配置复制关系，并将剩余辅助数据库链接到新升级的主数据库，无需任何用户的干预。导致了故障转移的中断得到缓解后，可能需要将应用程序返回到主要区域。为此，应使用“计划内”选项调用故障转移命令。有关如何使用 T-SQL 激活故障转移的详细信息，请参阅 [使用 Transact-SQL 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-transact-sql/)。有关如何使用 PowerShell 激活故障转移的详细信息，请参阅[使用 PowerShell 为 Azure SQL 数据库配置异地复制](/documentation/articles/sql-database-geo-replication-powershell/)。

- **保持凭据和防火墙规则同步**：我们建议将[数据库防火墙规则](/documentation/articles/sql-database-firewall-configure/)用于异地复制数据库，以便可以使用数据库复制这些规则，以确保所有辅助数据库具有与主数据库相同的防火墙配置。这样就不再需要客户手动配置和维护承载主数据库和辅助数据库的服务器上的防火墙规则。同样，将[包含的数据库用户](/documentation/articles/sql-database-manage-logins/)用于数据访问可确保主数据库和辅助数据库始终具有相同的用户凭据，以便在发生故障转移时，将不会因登录名和密码不匹配而产生中断。通过添加 [Azure Active Directory](/documentation/articles/active-directory-whatis/)，客户可以管理主数据库和辅助数据库的用户访问权限，且不再需要同时管理数据库中的凭据。

- **Azure 资源管理器 API 和基于角色的安全性**：活动异地复制包括一组用于管理的 [Azure 资源管理器 (ARM) API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx)，其中包括 [基于 ARM 的 PowerShell cmdlet](/documentation/articles/sql-database-geo-replication-powershell/)。这些 API 需要使用资源组，并支持基于角色的安全 (RBAC)。有关如何实现访问角色的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。

>[AZURE.NOTE] 向后兼容支持现有 [Azure SQL 服务管理（经典）REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx) 和 [Azure SQL 数据库（经典）cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn546723.aspx)。不过，仅在基于 ARM 的 [Azure SQL REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163571.aspx) 和基于 ARM 的 [Azure SQL 数据库 PowerShell cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt574084.aspx) 中支持活动异地复制的多种新功能。

## 防止丢失关键数据
由于广域网的延迟时间较长，连续复制使用了异步复制机制。这样，在发生故障时，会不可避免地丢失某些数据。但是，某些应用程序可能要求不能有数据丢失。为了保护这些关键更新，应用程序开发人员可以在提交事务后立即调用 [sp\_wait\_for\_database\_copy\_sync](https://msdn.microsoft.com/zh-cn/library/dn467644.aspx) 系统过程。调用 **sp\_wait\_for\_database\_copy\_sync** 会阻止调用线程，直到已将上次提交的事务复制到辅助数据库。该过程会等到辅助数据库确认所有排队的事务为止。**sp\_wait\_for\_database\_copy\_sync** 将划归到特定的连续复制链接。对主数据库具有连接权限的任何用户都可以调用此过程。

>[AZURE.NOTE] **sp\_wait\_for\_database\_copy\_sync** 过程调用导致的延迟可能会很明显。延迟取决于当前事务日志长度的大小，并且在复制整个日志之前，将不会返回。除非绝对必要，否则请避免调用此过程。

## 后续步骤
有关 SQL 数据库的其他业务连续性功能的信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

<!---HONumber=Mooncake_0606_2016-->