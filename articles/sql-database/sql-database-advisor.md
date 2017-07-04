<properties 
   pageTitle="Azure SQL 数据库顾问" 
   description="Azure SQL 数据库顾问为你的现有 SQL 数据库提供建议，这样可以提高当前的查询性能。" 
   services="sql-database" 
   documentationCenter="" 
   authors="stevestein" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-management" 
   ms.date="09/30/2016"
   wacn.date="01/24/2017"
   ms.author="sstein"/>

# SQL 数据库顾问

> [AZURE.SELECTOR]
- [SQL 数据库顾问概述](/documentation/articles/sql-database-advisor/)
- [Azure 门户](/documentation/articles/sql-database-advisor-portal/)

Azure SQL 数据库可以学习和适应你的应用程序并提供自定义的建议，使你能够将 SQL 数据库的性能最大化。Azure SQL 数据库顾问提供有关创建和删除索引、参数化查询，以及解决架构问题的建议。该顾问通过分析 SQL 数据库的使用历史记录来评估性能。提供的建议最适合运行数据库的典型工作负荷。

以下建议适用于 V12 服务器（建议不适用于 V11 服务器）。现在你可以将创建和删除索引建议设置为自动应用，请参阅[自动索引管理](/documentation/articles/sql-database-advisor-portal/#enable-automatic-index-management)以获取详细信息。

## 创建索引建议 

当 SQL 数据库服务检测到丢失的索引（若已创建）有益于你的数据库工作负荷时，就会出现**创建索引**建议（仅限非群集索引）。

## 删除索引建议

当 SQL 数据库服务检测到重复的索引时，就会出现**删除索引**建议（当前处于预览状态，且仅适用于重复的索引）。

##<a name="viewing-recommendations"></a> 参数化查询建议

当一个或多个正在持续被重新编译但都以相同的查询执行计划结束的查询时，就会出现**参数化查询**建议。这种状态提供了一个应用强制参数化的机会，其允许查询计划进行缓存并在将来可以被重复使用，从而改善性能和减少资源使用。

对 SQL Server 发出的每个查询一开始需要进行编译，生成执行计划。每个生成的计划将添加到计划缓存中，相同查询的后续执行可以重复使用该缓存中的此计划，而无需进一步编译。

发送包含非参数化值的查询的应用程序可能会导致性能开销，其中对于使用不同参数值的每个此类查询，将重新编译执行计划。在许多情况下，使用不同参数值的相同查询将生成相同的执行计划，但这些计划将仍然分别添加到计划缓存中。重新编译执行计划会占用数据库资源、增加查询持续时间并使计划缓存溢出，从而导致系统从缓存中逐出计划。可以通过对数据库设置强制参数化选项来更改 SQL Server 的此行为。

为了帮助你估计此建议的影响，将为你提供实际 CPU 使用率和预计 CPU 使用率（就像已应用建议一样）之间的比较。除了节省 CPU 外，查询持续时间会因编译花费的时间而减少。计划缓存上的开销也会更低，从而允许大部分计划保留在缓存中并可重复使用。可以通过单击“应用”命令来快速轻松地应用此建议。

应用此建议后，它将在几分钟之内对数据库启用强制参数化，并将启动监视进程（大约持续 24 小时）。经过这段时间后，即可看到验证报告，该报表显示应用此建议之前和之后的 24 小时内数据库的 CPU 使用率。SQL 数据库顾问具有安全机制，在检测到性能衰退的情况下，可以自动还原已应用的建议。

## 修复架构问题建议

当 SQL 数据库服务发现 Azure SQL 数据库上发生的架构相关 SQL 错误的数量有异常时，就会出现**修复架构问题**建议。此建议通常在数据库在一个小时内遭遇到多个架构相关的错误（无效的列名称、无效的对象名称等）时出现。

“架构问题”是 SQL Server 中的一类语法错误，当 SQL 查询的定义与数据库架构的定义不一致时发生。例如，目标表中可能缺少查询所需的某个列，反之亦然。

当 Azure SQL 数据库服务发现 Azure SQL 数据库上发生的架构相关 SQL 错误的数量有异常时，就会出现“修复架构问题”建议。下表显示与架构问题相关的错误：

|SQL 错误代码|消息|
|--------------|-------|
|201|过程或函数“*”需要参数“*”，但未提供该参数。|
|207|列名称“*”无效。|
|208|对象名“*”无效。 |
|213|列名或所提供值的数目与表定义不匹配。 |
|2812|找不到存储过程“*”。 |
|8144|为过程或函数 * 指定了过多的参数。 |

## 后续步骤

监视建议并继续应用它们以优化性能。数据库工作负荷是动态的，并且不断地更改。SQL 数据库顾问将继续监视和提供可能提高数据库性能的建议。

 - 有关如何在 Azure 门户中使用 SQL 数据库顾问的步骤，请参阅 [Azure 门户中的 SQL 数据库顾问](/documentation/articles/sql-database-advisor-portal/)。
 - 若要了解排名靠前的查询的性能影响，请参阅[查询性能见解](/documentation/articles/sql-database-query-performance/)。

## 其他资源

- [查询存储](https://msdn.microsoft.com/zh-cn/library/dn817826.aspx)
- [CREATE INDEX](https://msdn.microsoft.com/zh-cn/library/ms188783.aspx)
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)

<!---HONumber=Mooncake_1024_2016-->