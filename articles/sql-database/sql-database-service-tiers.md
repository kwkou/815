<properties
	pageTitle="SQL 数据库性能和选项：服务层 | Azure"
	description="比较 SQL 数据库服务层的性能和业务连续性功能，以便在缩放的同时，在成本与功能之间找到平衡点。"
	keywords="数据库选项,数据库性能"
	services="sql-database"
	documentationCenter=""
	authors="CarlRabeler"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="08/10/2016"
	wacn.date="09/28/2016"/>

# SQL 数据库选项和性能：了解每个服务层提供的功能

[Azure SQL 数据库](/documentation/articles/sql-database-technical-overview/)具有多个服务层，用于处理不同的工作负荷。你可以随时[更改服务层](/documentation/articles/sql-database-scale-up-powershell/)，并将给应用程序造成的停机时间降至最低（通常平均为 4 秒以下）。你还可以使用定义的特征和定价[创建单一数据库](/documentation/articles/sql-database-get-started/)。也可以通过[创建弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-powershell/)来管理多个数据库。在这两种情况下，层包括“基本”、“标准”和“高级”。这些层中的数据库选项类似于单个数据库和弹性池，只是针对弹性池提供了其他注意事项。本文提供了单一数据库和弹性数据库的服务层的详细信息。

## 服务层和数据库选项
基本、标准和高级服务层都提供 99.99% 的运行时间 SLA 和可预测的性能、灵活的业务连续性选项、安全功能和按小时计费。下表提供了最适用于不同应用程序工作负荷的层的示例。

| 服务层 | 目标工作负荷 |
|---|---|
| **基本** | 最适合小型数据库，通常支持在给定时间执行一个活动操作。示例包括用于开发或测试的数据库，或不常使用的小型应用程序。 |
| **标准** | 大多数云应用程序的首选选项，支持多个并发查询。示例包括工作组或 Web 应用程序。 |
| **高级** | 专为高事务量设计，支持大量并发用户，并且需要最高级别的业务连续性功能。示例包括支持任务关键型应用程序的数据库。 |

>[AZURE.NOTE] Web Edition 和 Business Edition 已停用。如果你打算继续使用 Web 和 Business Edition，请阅读[版本停用常见问题](/pricing/details/sql-database/)。

## 单一数据库服务层和性能级别
对于单一数据库，每个服务层内都具有多个性能级别。你可以灵活选择最能满足你的工作负荷需求的级别。如果你需要增加或减少工作负荷，可以轻松更改数据库层。有关详细信息，请参阅[更改数据库服务层和性能级别](/documentation/articles/sql-database-scale-up-powershell/)。

此处列出的性能特征适用于使用 [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/) 创建的数据库。在 Azure 中的基础硬件托管多个数据库的情况下，你的数据库仍可确保获得一系列资源，数据库的预期性能特征将不受影响。

[AZURE.INCLUDE [SQL 数据库服务层表](../../includes/sql-database-service-tiers-table.md)]

若要更好地了解 DTU，请参阅本主题中的 [DTU 部分](#understanding-dtus)。

>[AZURE.NOTE] 如需本服务层表中所有其他行的详细说明，请参阅[服务层功能和限制](/documentation/articles/sql-database-performance-guidance/#service-tier-capabilities-and-limits)。

## 弹性池服务层和性能 (eDTU)
除了创建和缩放单一数据库外，你还可以选择管理[弹性池](/documentation/articles/sql-database-elastic-pool/)中的多个数据库。弹性池中的所有数据库共享一组公用资源。性能特征由 *弹性数据库事务单位* (eDTU) 数度量。与单一数据库一样，弹性池有三个服务层：**基本**、**标准**和**高级**。对于池，这三个服务层仍定义整体性能限制和多个功能。

池允许弹性数据库共享和使用 DTU 资源，而无需为该池中的数据库分配特定性能级别。例如，标准池中的单一数据库可使用 0 个 eDTU 到最大数据库 eDTU 数（配置池时设置的）运转。这允许多个具有不同工作负荷的数据库有效地使用可用于整个池的 eDTU 资源。有关详细信息，请参阅[弹性池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。

下表描述了池服务层的特征。

[AZURE.INCLUDE [用于弹性数据库的 SQL 数据库服务层表](../../includes/sql-database-service-tiers-table-elastic-db-pools.md)]

池中的每个数据库也遵循该层的单一数据库特征。例如，基本池具有每池最大会话数为 4800 - 28800 的限制，但该池中单个数据库具有 300 个会话数的数据库限制（即在上一节中指定的单个基本数据库的限制）。

## 了解 DTU

[AZURE.INCLUDE [SQL 数据库 DTU 说明](../../includes/sql-database-understanding-dtus.md)]

## 选择服务层

要决定服务层，先确定数据库是否将是独立数据库，或是弹性池的一部分。

### 选择独立数据库的服务层

要决定独立数据库的服务层，先确定所需的数据库功能以选择 SQL 数据库版本：

- 数据库大小（基本数据库最大 5 GB、标准数据库最大 250 GB、高级数据库 500 GB 到 1 TB - 具体取决于性能级别）
- 数据库备份保留期（基本数据库 7 天、标准数据库 35 天、高级数据库 35 天）

确定了 SQL 数据库版本后，就可以确定数据库的性能级别（DTU 数）。可以根据实际经验猜测，然后[动态地增加或减少](/documentation/articles/sql-database-scale-up-powershell/)。还可以使用 [DTU 计算器](http://dtucalculator.azurewebsites.net/)估计所需的 DTU 数。

### 选择弹性数据库池的服务层。

要决定弹性数据库池的服务层，先确定所需的数据库功能，然后选择池的服务层。

- 数据库大小（基本数据库 2GB、标准数据库 250 GB 和高级数据库 500 GB）
- 数据库备份保留期（基本数据库 7 天、标准数据库 35 天、高级数据库 35 天）
- 每个池的数据库数（基本池 400 个、标准池 400 个、高级池 50 个）
- 每个池的最大存储（基本池 117 GB、标准池 1200、高级池 750）

确定了池的服务层后，就可以确定池的性能级别 (eDTU)。可以根据实际经验猜测，然后[动态地增加或减少](sql-database-elastic-pool-manage-portal.md#change-performance-settings-of-a-pool)。还可以使用 [DTU 计算器](http://dtucalculator.azurewebsites.net/)估计池中单个数据库所需的 DTU 数，帮助设置该池的上限。

## 后续步骤
- [SQL 数据库定价](/pricing/details/sql-database/)中提供了有关这些层的价格详细信息。
- 了解有关[弹性数据库池](/documentation/articles/sql-database-elastic-pool-guidance/)和[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)的详细信息。
- 了解如何[监视、管理弹性池和调整其大小](/documentation/articles/sql-database-elastic-pool-manage-powershell/)以及如何[监视单个数据库的性能](/documentation/articles/sql-database-single-database-monitor/)。
- 现已了解有关 SQL 数据库层的信息，欢迎使用[试用版](/pricing/1rmb-trial)来试用这些层，并了解[如何创建第一个 SQL 数据库](/documentation/articles/sql-database-get-started/)。

## 其他资源

有关多租户软件即服务 (SaaS) 数据库应用程序的常见数据体系结构模式的信息，请参阅[包含 Azure SQL 数据库的多租户 SaaS 应用程序的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)。

<!---HONumber=Mooncake_0919_2016-->