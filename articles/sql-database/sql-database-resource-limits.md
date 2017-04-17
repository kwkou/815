<properties
    pageTitle="Azure SQL 数据库资源限制 | Azure"
    description="本页介绍 Azure SQL 数据库的一些常见资源限制。"
    services="sql-database"
    documentationcenter="na"
    author="janeng"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="884e519f-23bb-4b73-a718-00658629646a"
    ms.service="sql-database"
    ms.custom="overview"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="janeng"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="68153ff51e8bdac7a3239be95ab9871d08b552ac"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-sql-database-resource-limits"></a>Azure SQL 数据库资源限制
## <a name="overview"></a>概述
Azure SQL 数据库使用两种不同的机制管理可用于数据库的资源：**资源调控**和**强制限制**。 本主题介绍了资源管理的这两个主要方面。

## <a name="resource-governance"></a>资源调控
基本、标准和高级服务层的设计目标之一是为了让 Azure SQL 数据库的行为与数据库运行在其自己的计算机上相同，独立于其他数据库。 资源调控模拟了此行为。 如果聚合资源利用率达到分配给数据库的最大可用 CPU、内存、日志 I/O 和数据 I/O 资源数，资源调控会将执行中的查询排队，并在资源释放时将资源分配给排队的查询。

由于在专用计算机上，利用所有可用资源将导致当前执行的查询的执行时间较长，这可能会导致客户端上的命令超时。 达到并发请求数限制后，如果尝试执行新查询，具有积极重试逻辑的应用程序以及对数据库执行查询的应用程序遇到错误消息的频率会很高。

### <a name="recommendations"></a>建议：
在接近数据库的最大利用率时，监视查询的资源利用率以及平均响应时间。 在遇到较长的查询延迟时，通常有三个选择：

1. 减少数据库的传入请求数以防止请求超时和请求积累。
2. 为数据库分配更高的性能级别。
3. 优化查询，以减少每个查询的资源利用率。 有关详细信息，请参阅“Azure SQL 数据库性能指南”一文中的“查询优化/提示”部分。

## <a name="enforcement-of-limits"></a>强制限制
CPU、内存、日志 I/O 和数据 I/O 以外的资源在达到限制时，将通过拒绝新请求来强制实施。 当数据库达到所配置的最大大小限制时，将无法执行会增加数据大小的插入和更新操作，但仍可执行选择和删除操作。 客户端将根据已达到的限制收到[错误消息](/documentation/articles/sql-database-develop-error-messages/)。

例如，会限制与 SQL 数据库的连接数以及可处理的并发请求数。 SQL 数据库允许与数据库的连接数大于并发请求数以支持连接池。 虽然应用程序可以轻松地控制可用的连接数，但并行请求数通常难于估计和控制。 特别是在负载峰值期间，应用程序发送过多请求或数据库达到其资源限制，并且由于长时间运行查询，开始堆积工作线程，可能会遇到错误。

## <a name="service-tiers-and-performance-levels"></a>服务层和性能级别
单一数据库和弹性池都有服务层和性能级别。

### <a name="single-databases"></a>单一数据库
对于单一数据库，数据库服务层和性能级别定义了数据库限制。 下表描述了基本、标准和高级数据库在不同性能级别上的特征。

[AZURE.INCLUDE [SQL DB service tiers table](../../includes/sql-database-service-tiers-table.md)]

### <a name="elastic-pools"></a>弹性池
[弹性池](/documentation/articles/sql-database-elastic-pool/)共享池中数据库中的资源。 下表描述了基本、标准和高级弹性池的特征。

[AZURE.INCLUDE [SQL DB service tiers table for elastic databases](../../includes/sql-database-service-tiers-table-elastic-pools.md)]

有关上述表中列出的每个资源的扩展定义，请参阅[服务层功能和限制](/documentation/articles/sql-database-performance-guidance/#service-tier-capabilities-and-limits)中的描述。 有关服务层的概述，请参阅 [Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers/)。

## <a name="other-sql-database-limits"></a>其他 SQL 数据库限制
| 区域 | 限制 | 说明 |
| --- | --- | --- |
| 使用按订阅自动导出的数据库 |10 |自动导出可创建自定义计划来备份 SQL 数据库。 此功能的预览将于 2017 年 3 月 1 日结束。  |
| 每个服务器的数据库数 |最多 5000 个 |V12 服务器上每个服务器最多允许使用 5000 个数据库。 |
| 每个服务器的 DTU |45000 |V12 服务器上每个服务器有 45000 个 DTU 可用于预配独立数据库和弹性池。 每台服务器允许的独立数据库和池的总数仅受服务器 DTU 数限制。  




## <a name="resources"></a>资源
[Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)

[Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers/)

[SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/)
<!--Update_Description: wording update-->