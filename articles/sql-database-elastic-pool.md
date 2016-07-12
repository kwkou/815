<properties
	pageTitle="SQL 数据库的弹性数据库池 | Azure"
	description="可通过使用池管理成百上千个数据库可通过池分发一组性能单位的一个价格。可随心所欲地移入或移出数据。"
	keywords="弹性数据库,sql 数据库"
	services="sql-database"
	documentationCenter=""
	authors="sidneyh"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="sql-database"
	ms.date="04/04/2016"
	wacn.date="06/14/2016"/>


# 什么是 Azure 弹性数据库池？

SaaS 开发人员必须创建并管理数十、数百或甚至数千个 SQL 数据库。弹性数据库池简化了跨多个数据库的创建、维护和性能管理。随意在池中加减数据库。请参阅[使用 PowerShell](/documentation/articles/sql-database-elastic-pool-create-powershell/) 或 [C#](/documentation/articles/sql-database-elastic-pool-csharp/)。

有关 API 和错误的详细信息，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference/)。

## 工作原理

常见的 SaaS 应用程序模式是单租户数据库模型：每个客户都有一个数据库。每个客户（数据库）对内存、IO 和 CPU 具有不可预知的资源要求。由于需求有高峰和低谷，如何分配资源？ 通常有两个选项：(1) 基于高峰使用情况过度设置资源，因此需要支付额外的费用，或者 (2) 为了节省成本而采用低配，但在高峰期间会出现性能下降而导致客户满意度降低。弹性数据库池通过确保数据库在必要时获取所需的效能资源，同时在可预测的预算内提供简单的资源分配机制，以此来解决此问题。

弹性数据库池可解决此问题。

在 SQL 数据库中，单一数据库在数据库事务单位 (DTU) 中相对衡量数据库处理资源需求的能力，弹性数据库池则在弹性 DTU (eDTU) 中衡量此能力。请参阅 [SQL 数据库简介](/documentation/articles/sql-database-technical-overview/#understand-dtus)，了解有关 DTU 和 eDTU 的详细信息。

对池提供了固定数量的 eDTU，以获得固定价格。在池中，单独的数据库都被赋予了在固定参数内自动缩放的灵活性。高负荷下的数据库可能会消耗更多的 eDTU 以满足需求。轻负荷下的数据库占用较少 eDTU，并且没有任何负荷的数据库不会消耗任何 eDTU。设置整个池（而非单个数据库）的资源简化了管理任务。此外，必须具有该池的可预测预算。

其他的 eDTU 可添加到现有的池，而未发生数据库停机或对数据库产生负面影响。同样，你随时可以从现有池中删除不再需要的额外 eDTU。

并且可以向池添加或缩减数据库。如果可以预测到数据库的资源利用率不足，则将其移出。

## 在池中有哪些数据库？

![弹性数据库池中共享 eDTU 的 SQL 数据库。][1]

最适合添加到弹性数据库池的数据库通常是有时活动，有时不活动。在上述示例中，你可以看到单一数据库的活动、4 个数据库的活动，最后是包含 20 个数据库的弹性数据库池的活动。活动随时间而不同的数据库很适合添加到弹性池，因为它们不是永远都在使用中，而且可以共享 eDTU。并非所有数据库都符合此模式。具有更稳定的资源需求的数据库更适合“基本”、“标准”和“高级”服务层级，这些层级的资源是单独分配的。

[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。


> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。

## 弹性数据库池属性
弹性池和弹性数据库的限制

| 属性 | 说明 |
| :-- | :-- |
| 服务层 | “基本”、“标准”或“高级”。服务层确定性能范围和可配置的存储限制的范围，以及业务连续性选择。池中的每个数据库都有与池相同的服务层。“服务层”也被称为“edition”。|
| 每个池的 eDTU | 池中的数据库可以共享的 eDTU 的数量上限。池中的数据库同时使用的 eDTU 总量不能超过此限制。 |
| 每个池的存储量 | 池中的数据库可以共享的存储量上限。池中的数据库使用的存储总量不能超过此限制。此限制取决于每个池的 eDTU。如果超出此限制，所有数据库都将变成只读。 |
| 每个数据库的 eDTU 上限 | 池中的任何数据库都可以使用的 eDTU 的数量上限，适用于池中的所有数据库。每个数据库的 eDTU 上限并不是资源保障。 |
| 每个数据库的 eDTU 下限 | 能够保证的池中任何数据库的 eDTU 的数量下限，适用于池中的所有数据库。每个数据库的 eDTU 下限可以设置为 0。请注意，池中数据库数目和每个数据库的 eDTU 下限的积不能超过每个池的 eDTU。 |


## 弹性池和弹性数据库的 eDTU 和存储限制


[AZURE.INCLUDE [用于弹性数据库的 SQL 数据库服务层表](../includes/sql-database-service-tiers-table-elastic-db-pools.md)]

## 弹性数据库作业

通过池，可以通过运行**[弹性作业](/documentation/articles/sql-database-elastic-jobs-overview/)**中的脚本来简化管理任务。弹性数据库作业可消除与大量数据库有关的大部分麻烦。若要开始，请参阅[弹性数据库作业入门](/documentation/articles/sql-database-elastic-jobs-getting-started/)。

有关其他工具的详细信息，请参阅[弹性数据库工具学习路线图](https://azure.microsoft.com/documentation/learning-paths/sql-database-elastic-scale)。

## 池中数据库的业务连续性功能

弹性数据库支持 V12 服务器上单一数据库可用的大部分[业务连续性功能](/documentation/articles/sql-database-business-continuity/)（目前以预览版提供）。

### 时间点还原

系统会自动备份弹性数据库池中的数据库，备份保留策略与单一数据库对应的服务层相同。总之，每个层中的数据库具有不同的还原范围：

* 基本池：可还原到过去 7 天内的任何点。
* 标准池：可还原到过去 14 天内的任何点。
* 高级池：可还原到过去 35 天内的任何点。

在预览期，池中的数据库将还原到相同池中的新数据库。删除的数据库始终还原为池外的独立数据库，因而将使用该服务层的最低性能层级。例如，标准池中删除的弹性数据库将还原为 S0 数据库。你可以以编程方式使用 REST API 执行数据库还原操作。PowerShell cmdlet 支持即将推出。

### 异地还原

异地还原功能允许你将池中的数据库恢复到其他区域中的服务器。在使用预览版期间，若要将池中的数据库还原到其他服务器，目标服务器需要有一个名称与源池相同的池。如果需要，请在目标服务器上创建新的池，并在还原该数据库之前为其提供一个相同的名称。如果目标服务器上不存在相同名称的池，则异地还原操作将失败。有关详细信息，请参阅[使用异地还原进行恢复](/documentation/articles/sql-database-disaster-recovery/#recover-using-geo-restore)。


### 异地复制

标准异地复制适用于标准或高级弹性数据库池中的任何数据库。只要服务层相同，异地复制合作关系中的一个或所有数据库就可以处于一个弹性数据库池中。可以使用[PowerShell](/documentation/articles/sql-database-geo-replication-powershell/) 或 [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/) 为弹性数据库池配置异地复制。

### 导入和导出

支持从池中导出数据库。目前不支持直接将数据库导入池，但可以先导入至单一数据库，然后将数据库移入池中。


<!--Image references-->
[1]: ./media/sql-database-elastic-pool/databases.png

<!---HONumber=Mooncake_0606_2016-->
