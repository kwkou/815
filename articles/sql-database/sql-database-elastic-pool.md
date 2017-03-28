<properties
    pageTitle="什么是 Azure 弹性池？| Azure"
    description="使用池管理成百上千个数据库可通过池分发一组性能单位的价格。可随心所欲地移入或移出数据。"
    keywords="弹性池,sql 数据库"
    services="sql-database"
    documentationcenter=""
    author="ddove"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="b46e7fdc-2238-4b3b-a944-8ab36c5bdb8e"
    ms.service="sql-database"
    ms.custom="multiple databases"
    ms.devlang="NA"
    ms.date="02/01/2017"
    wacn.date="03/24/2017"
    ms.author="ddove"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />  


# 什么是 Azure SQL 弹性池？
可以在 Azure 逻辑服务器上为 SQL 弹性池提供固定数目和固定价格的 eDTU，在该服务器上的数据库池中共享。SQL DB 弹性池是一种简单的低成本高效益的解决方案，用于管理使用模式变化很大且不可预测的多个数据库的性能目标。

> [AZURE.NOTE]
>弹性池在所有 Azure 区域中均已正式发布 (GA)，但印度西部除外，这些区域当前仅发布了预览版。将尽快在此区域中正式发布弹性池。
>
>

## 工作原理

常见的 SaaS 应用程序模式是单租户数据库模型：每个客户都有自己的数据库。每个客户（数据库）对内存、IO 和 CPU 具有不可预知的资源要求。由于需求有高峰和低谷，那么应该如何经济高效地分配资源呢？ 通常有两个选项：(1) 基于高峰使用情况过度设置资源，因此需要支付额外的费用，或者 (2) 为了节省成本而采用低配，但在高峰期间会出现性能下降而导致客户满意度降低。弹性池通过确保数据库在需要时获得所需的性能资源来解决此问题。它们在可预测的预算内提供简单的资源分配机制。若要了解有关使用弹性池的 SaaS 应用程序的设计模式的详细信息，请参阅[多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)。


在 SQL 数据库中，数据库处理资源需求能力的相对衡量标准为单一数据库的数据库事务单位 (DTU)，对于弹性池中的数据库，则是弹性 DTU (eDTU)。请参阅 [SQL 数据库简介](/documentation/articles/sql-database-technical-overview/)，了解有关 DTU 和 eDTU 的详细信息。

为池提供了固定数量的 eDTU，以获得固定价格。在池中，单独的数据库都被赋予了在固定参数内自动缩放的灵活性。高负载下的数据库可能会消耗更多的 eDTU 来满足需求。轻负载下的数据库占用较少 eDTU，没有任何负载的数据库不会消耗任何 eDTU。设置整个池（而非单个数据库）的资源简化了管理任务。此外，必须具有该池的可预测预算。

可将更多 eDTU 添加现有池而不会造成数据库关闭，除非需要移动数据库以便提供更多计算资源来预留新 eDTU。同样，可随时从现有池中删除不再需要的额外 eDTU。

并且可以向池添加或缩减数据库。如果可以预测到数据库的资源利用率不足，则将其移出。

## 池中有哪些数据库？
![弹性池中共享 eDTU 的 SQL 数据库。][1]  


最适合添加到弹性池的数据库通常有时活动，有时不活动。上述示例中，可以看到单一数据库的活动、4 个数据库的活动，最后是包含 20 个数据库的弹性池的活动。活动随时间而不同的数据库很适合添加到弹性池，因为它们不是永远都在使用中，而且可以共享 eDTU。并非所有数据库都符合此模式。具有更稳定资源需求的数据库更适合“基本”、“标准”和“高级”服务层级，这些层级的资源是单独分配的。

[弹性池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。

##<a name="eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases"></a><a name="elastic-database-pool-pricing-tier-recommendations"></a><a name="edtu-and-storage-limits-for-elastic-pools-and-elastic-databases"></a> 弹性池的 eDTU 和存储限制

下表描述了基本、标准和高级弹性池的特征。

[AZURE.INCLUDE [用于弹性数据库的 SQL DB 服务层表](../../includes/sql-database-service-tiers-table-elastic-pools.md)]

如果使用了弹性池的所有 DTU，那么池中的每个数据库将接收相同数量的资源来处理查询。SQL 数据库服务通过确保相等的计算时间片段，在数据库之间提供资源共享的公平性。弹性池资源共享公平性是在将每个数据库的 DTU 最小值设为非零值时，对另外为每个数据库保证的任意资源量的补充。

##<a name="elastic-database-pool-properties"></a> 弹性池属性

下表介绍弹性池和入池数据库的限制。

### 弹性池的限制
| 属性 | 说明 |
|:--- |:--- |
| 服务层 |“基本”、“标准”或“高级”。服务层确定性能范围和可配置的存储限制的范围，以及业务连续性选择。池中的每个数据库都有与池相同的服务层。“服务层”也被称为“版本”。 |
| 每个池的 eDTU 数 |池中的数据库可以共享的 eDTU 的数量上限。池中的数据库同时使用的 eDTU 总量不能超过此限制。 |
| 每个池的最大存储空间 (GB) |池中的数据库可以共享的存储量上限（以 GB 为单位）。池中的数据库使用的存储总量不能超过此限制。此限制取决于每个池的 eDTU。如果超出此限制，所有数据库都将变成只读。每个数据库的最大存储是指数据文件的最大存储，不包括日志文件使用的空间。 |
| 每个池的数据库的最大数目 |每个池允许的数据库的最大数目。 |
| 每个池的最大并发工作线程数 |池中可用于所有数据库的最大并发工作线程（请求）数。 |
| 每个池的最大并发登录数 |池中所有数据库的最大并发登录数。 |
| 每个池的最大并发会话数 |池中可用于所有数据库的最大会话数。 |

### 入池数据库的限制
| 属性 | 说明 |
|:--- |:--- |
| 每个数据库的最大 eDTU 数 |根据池中其他数据库的 eDTU 使用率，池中任何数据库可以使用的 eDTU 的最大数目。每个数据库的 eDTU 上限并不是数据库的资源保障。这是应用于池中所有数据库的全局设置。将每个数据库的最大 eDTU 数设置得足够高，以处理数据库使用高峰情况。因为池通常会假定数据库存在热使用模式和冷使用模式，在这些模式中并非所有数据库同时处于高峰使用状态，所以预期会存在某种程度的过量使用情况。例如，假设每个数据库的高峰使用量为 20 个 eDTU，并且池中 100 个数据库仅有 20% 同时处于高峰使用中。如果将每个数据库的 eDTU 最大值设为 20 个 eDTU，则可以认为超量 5 倍使用该池是合理的，并且将每个池的 eDTU 数设为 400。 |
| 每个数据库的最小 eDTU 数 |池中任何数据库可以保证的 eDTU 最小数目。这是应用于池中所有数据库的全局设置。每个数据库的最小 eDTU 可能设为 0，这也是默认值。此属性值可以设置为介于 0 和每个数据库的平均 eDTU 使用量之间的任意值。池中数据库的数目和每个数据库的 eDTU 下限的积不能超过每个池的 eDTU 数。例如，如果一个池有 20 个数据库，每个数据库的 eDTU 最小值设为 10 个 eDTU，则池的 eDTU 数目必须大于或等于 200 个 eDTU。 |
| 每个数据库的最大存储空间 (GB) |池中一个数据库的最大存储空间。入池数据库共享池的存储空间，因此数据库存储空间限制为小于池的剩余存储空间和每个数据库的最大存储空间。每个数据库的最大存储是指数据文件的最大大小，不包括日志文件使用的空间。 |


## 池中数据库的业务连续性功能
入池数据库通常支持和可用于单一数据库相同的[业务连续性功能](/documentation/articles/sql-database-business-continuity/)。

### 时间点还原
时间点还原使用自动的数据库备份将池中的数据库恢复到特定的时间点。请参阅[时点还原](/documentation/articles/sql-database-recovery-using-backups/#point-in-time-restore)

### 异地还原
当数据库因其所在的区域发生事故而不可用时，异地还原会提供默认的恢复选项。请参阅[还原 Azure SQL 数据库或故障转移到辅助数据库](/documentation/articles/sql-database-disaster-recovery/)

### 活动异地复制

对于具有异地还原无法提供的更强烈的恢复要求的应用程序，可使用 [Azure 门户预览](/documentation/articles/sql-database-geo-replication-portal/)、[PowerShell](/documentation/articles/sql-database-geo-replication-powershell/) 或 [Transact-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/) 配置活动异地复制。

## 其他资源
* [Microsoft 虚拟大学视频课程：Azure SQL 数据库弹性功能](https://mva.microsoft.com/zh-cn/training-courses/elastic-database-capabilities-with-azure-sql-db-16554)

<!--Image references-->

[1]: ./media/sql-database-elastic-pool/databases.png

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: add link reference to MVA resource-->