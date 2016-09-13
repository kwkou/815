<properties
   pageTitle="云业务连续性 - 时间点还原 - SQL 数据库 | Azure"
   description="了解有关时间点还原的信息，它让你能够将 Azure SQL 数据库回滚到之前的时间点（最多 35 天）。"
   services="sql-database"
   documentationCenter=""
   authors="stevestein"
   manager="jhubbard"
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/17/2016"
   wacn.date="07/18/2016"/>

# 概述：SQL 数据库时间点还原

> [AZURE.SELECTOR]
- [业务连续性概述](/documentation/articles/sql-datbase-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

使用时间点还原，你可以通过 [SQL 数据库自动还原](/documentation/articles/sql-database-automated-backups/)将现有数据库作为新数据库还原到同一逻辑服务器上的旧时间点。无法覆盖现有数据库。你可以通过 [PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell/) 或 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx) 还原到旧时间点。

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell/)

可以将数据库还原到任何性能级别或还原到弹性池。你需要确保在服务器或池上具有足够的 DTU 配额，牢记还原将创建新的数据库且还原的数据库的服务层和性能级别可能与活动数据库的当前状态有所不同。还原完成后，还原的数据库将是完全可联机访问的正常数据库，并基于其服务层和性能级别按正常费率计费。

若要找出最早的可用还原点，请使用[获取数据库](https://msdn.microsoft.com/zh-cn/library/dn505708.aspx) (RecoveryPeriodStartDate) 获取最早的还原点（非异地复制的还原点）。

如果你出于恢复的目的而还原数据库，可以将还原的数据库作为原始数据库的替代数据库，或使用它来检索数据，然后更新原始数据库。如果还原的数据库旨在替换原始数据库，那么你应该验证性能级别和/或服务层是否合适，如有必要，还应改变该数据库的规模。你可以使用 T-SQL 中的 ALTER DATABASE 命令来重命名原始数据库，然后为还原的数据库指定原有的名称。

如果你计划从还原的数据库检索数据，则将需要分别编写和执行任何所需的数据恢复脚本。尽管还原操作可能需要很长时间才能完成，但整个过程中，都可在数据库列表中看见该数据库。如果在还原期间删除数据库，将取消还原操作，且不会向你收费。

## 时间点还原的恢复时间

还原数据库所需的时间取决于许多因素，其中包括数据库的大小、事务日志数、选择的时间点和需重播以重构所选点时状态的活动量。对于非常大和/或活动的数据库，还原可能要花费几个小时。大部分数据库还原操作可在 12 小时内完成。

## 备份/还原与复制/导出/导入

时间点还原是针对意外数据丢失或损坏的情况下恢复基本版、标准版和高级版数据库的建议方法。备份和还原成本更低（除非备份量过多，否则你无需支付任何备份费用，而对于为确保事务性一致导出而进行的数据库复制，以及 BACPAC 文件存储，则需要支付相关费用）、无需管理（备份是自动进行的，但必须自行管理和计划导出）且具有更好的 RPO（相较复制/导出/导入操作，可以更精准地（一分钟）还原到特定时间）和更优的恢复时间（从备份还原通常比导入更快，后者涉及创建架构、禁用索引、加载数据然后为每个表格单独启用索引的各项步骤）。请注意，对于超出保留期的长期存档，仍然建议使用复制和导出解决方案。


## 摘要

自动备份和时间点还原保护数据库免于遭受意外的数据损坏或数据删除。所有 SQL 数据库均适用此零成本、零管理的解决方案。对于短期恢复的需求，相较复制/导出/导入这种替代解决方案，备份和还原具有显著优势。我们鼓励使用时间点还原作为你的业务连续性战略的一部分，并且仅在需要进行长期存档或数据迁移时才使用导出。


## 后续步骤

- 如需使用 PowerShell 恢复到某个时间点的详细步骤，请参阅[使用 PowerShell 进行的时间点还原](/documentation/articles/sql-database-point-in-time-restore-powershell/)。
- 有关如何使用 REST API 恢复到某个时间点的信息，请参阅 [Point-In-Time Restore using the REST API（使用 REST API 进行的时间点还原）](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)。

- 有关如何从用户或应用程序错误进行恢复的完整讨论，请参阅[用户错误恢复](/documentation/articles/sql-database-user-error-recovery/)。

## 其他资源

- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

<!---HONumber=Mooncake_0711_2016-->