<properties
	pageTitle="Azure SQL 数据库资源限制"
	description="本页介绍 Azure SQL 数据库的一些常见资源限制。"
	services="sql-database"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />


<tags
	ms.service="sql-database"
	ms.date="03/02/2016"
	wacn.date="06/14/2016" />


# Azure SQL 数据库资源限制

## 概述

Azure SQL 数据库使用两种不同的机制管理可用于数据库的资源：**资源调控**和**强制限制**。本主题将介绍这两个主要方面的资源管理。

## 资源控制
基本、标准和高级服务层的设计目标之一是为了让 Azure SQL 数据库的行为好像数据库运行在其自己的计算机上，完全独立于其他数据库。资源调控模拟了此行为。如果聚合资源利用率达到分配给数据库的最大可用 CPU、内存、日志 I/O 和数据 I/O 资源数，资源调控会将执行中的查询排队，并在资源释放时将资源分配给排队的查询。

由于在专用计算机上，利用所有可用资源将导致当前执行的查询的执行时间较长，这可能会导致客户端上的命令超时。达到并发请求数限制后，如果尝试执行新查询，具有积极重试逻辑的应用程序以及对数据库执行查询的应用程序遇到错误消息的频率会很高。

### 建议：
在接近数据库的最大利用率时，监视查询的资源利用率以及平均响应时间。在遇到较长的查询延迟时，通常有三个选择：

1.	减少数据库的传入请求数以防止请求超时和请求积累。

2.	为数据库分配更高的性能级别。

3.	优化查询，以减少每个查询的资源利用率。有关详细信息，请参阅“Azure SQL 数据库性能指南”一文中的“查询优化/提示”部分。

## 强制实施限制
CPU、内存、日志 I/O 和数据 I/O 以外的资源在达到限制时，将通过拒绝新请求来强制实施。客户端将根据已达到的限制收到[错误消息](/documentation/articles/sql-database-develop-error-messages/)。

例如，会限制与 SQL 数据库的连接数以及可处理的并发请求数。SQL 数据库允许与数据库的连接数大于并发请求数以支持连接池。虽然应用程序可以轻松地控制可用的连接数，但并行请求数通常难于估计和控制。特别是在负载峰值期间，那时应用程序发送太多请求或数据库达到其资源限制，并且由于长时间运行查询，开始堆积工作线程，可能会遇到错误。

## 服务层和性能级别

对于单一数据库，数据库服务层和性能级别定义了数据库限制。下表描述了基本、标准和高级数据库在不同性能级别上的特征。

[AZURE.INCLUDE [SQL 数据库服务层表](../includes/sql-database-service-tiers-table.md)]

[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)共享池中的数据库中的资源。下表描述了基本、标准和高级弹性数据库池的特征。

[AZURE.INCLUDE [用于弹性数据库的 SQL 数据库服务层表](../includes/sql-database-service-tiers-table-elastic-db-pools.md)]

有关上述表中列出的每个资源的扩展定义，请参阅[服务层功能和限制](/documentation/articles/sql-database-performance-guidance/#service-tier-capabilities-and-limits)中的描述。有关服务层的概述，请参阅 [Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers/)。

## 其他 SQL 数据库限制

| 区域 | 限制 | 说明 |
|---|---|---|
| 使用按订阅自动导出的数据库 | 10 | 自动导出允许你创建自定义计划来备份 SQL 数据库。有关详细信息，请参阅 [SQL 数据库：支持自动 SQL 数据库导出](http://weblogs.asp.net/scottgu/windows-azure-july-updates-sql-database-traffic-manager-autoscale-virtual-machines)。|

## 资源

[Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)

[Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers/)

[SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/)

<!---HONumber=Mooncake_0606_2016-->
