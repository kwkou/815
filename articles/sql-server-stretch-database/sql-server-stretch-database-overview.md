<!-- Remove azure portal -->
<properties
	pageTitle="Stretch Database 概述 | Azure"
	description="了解 Stretch Database 如何透明、安全地将冷数据迁移到 Azure 云。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglaslMS"
	manager=""
	editor=""/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="06/27/2016"
	wacn.date="01/03/2017"
	ms.author="douglasl"/>

# SQL Server Stretch Database 概述

SQL Server Stretch Database 可以透明、安全地将历史数据迁移到 Azure 云。

## SQL Server Stretch Database 有哪些优势？
SQL Server Stretch Database 优势如下：

### 经济高效的为冷数据提供可用性
使用 SQL Server Stretch Database 将冷暖事务数据从 SQL Server 动态延伸到 Azure。与典型的冷数据存储不同，用户数据始终联机且可供查询。数据保留时限可加长，无需破坏大型表（如客户订单历史记录）的存储库。 Azure 成本低，因此无需再扩展价格不菲的本地存储。用户可以选择定价层并在 Azure 经典管理门户中配置设置，掌控价格与数据访问速度。按需缩放。有关详细信息，请访问 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)页。

### 无需更改查询或应用程序
不管 SQL Server 数据是在本地还是已延伸到云中，用户你都可以顺畅访问这些数据。数据的存储位置可以进行设置，SQL Server 在后台处理数据移动。整个表始终会联机且可供查询。此外，SQL Server Stretch Database 无需对现有查询或应用程序进行更改 – 数据位置对应用程序完全透明。

### 简化本地数据维护
减少本地维护量和存储。本地数据的备份运行更快，在维护时段内即可完成。云中数据会自动备份。对本地存储的需求大大降低。与本地 SSD 相比， Azure 存储空间的费用降低了 80%。

### 迁移期间保持数据的安全性
用户可放心的将最重要的应用程序安全延伸到云中。SQL Server 的“始终加密”为动态数据提供加密。行级别安全性 (RLS) 和其他高级 SQL Server 安全功能也能配合 SQL Server Stretch Database 保护的数据。

## SQL Server Stretch Database 的作用是什么？
为 SQL Server 实例、数据库或表启用 SQL Server Stretch Database 后，SQL Server Stretch Database 将历史数据迁移到 Azure。

-   如果在单独的某个表中存储了冷数据，你可以迁移整个表。

-   如果表同时包含热数据和冷数据，你可以指定筛选器函数来选择要迁移的行。

**无需更改现有的查询和客户端应用。** 即使在数据迁移过程中，你也可以持续顺畅访问本地数据和远程数据。远程查询会出现轻微的延迟，但仅当你查询冷数据时，才会遇到这种延迟。

**Stretch Database 可确保在迁移过程中发生错误时数据不丢失**。它还包含重试逻辑用于处理迁移过程中可能发生的连接问题。动态管理视图可提供迁移状态。

本地服务器允许中止数据传输，进行故障排查、最大化网络带宽。

![延伸数据库概述][StretchOverviewImage1]

## SQL Server Stretch Database 是否适合自己？
如果用户与下表中的表述相符，可以考虑使用	SQL Server Stretch Database 。

|决策者|数据库管理员|
|------------------------------|-------------------|
|需长期保留事务数据。|表的大小失控。|
|需查询冷数据。|用户想要访问冷数据，但极少使用该数据。|
|需保留现有应用程序，不更新（包括一些早期应用）|需不断购置更多存储。|
|需节省存储成本。|无法在 SLA 的规定范围内备份或还原大型表。|

## 哪种类型的数据库和表符合延伸数据库条件？
Stretch Database 面向的事务数据库中存有大量历史数据，这些数据通常存储在少量表中。这些表可能超过十亿行。

利用 SQL Server 2016 的临时表功能，用户可以 SQL Server Stretch Database 将相关历史记录表(全部或部分)迁移到 Azure 上的存储中，Azure存储更具价格优势。有关详细信息，请参阅[系统版本临时表中的历史数据保留管理](https://msdn.microsoft.com/zh-cn/library/mt637341.aspx)。

使用 SQL Server Stretch Database 顾问(SQL Server 2016 升级顾问的功能) - 可以识别符合 SQL Server Stretch Database 的数据库和表。有关详细信息，请参阅[识别符合 SQL Server Stretch Database 的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。若要详细了解潜在的阻碍性问题，请参阅 [Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)。

## 体验延伸数据库
**借助 AdventureWorks 示例数据库体验延伸数据库。** 若要获取 AdventureWorks 示例数据库，请从[此处](https://www.microsoft.com/download/details.aspx?id=49502)下载数据库文件以及示例和脚本文件。示例数据库需还原到 SQL Server 2016 实例，然后解压示例文件，从 Stretch DB 文件夹中打开 Stretch DB Samples 文件。运行此文件中的脚本查看启用延伸数据库之前和之后数据所用的空间的不同，追踪数据迁移的进度，确认在数据迁移期间以及迁移之后都可以继续查询现有数据，也可以插入新数据。

## 后续步骤
**识别符合 SQL Server Stretch Database 条件的数据库和表。** 下载 SQL Server 2016 升级顾问并运行 SQL Server Stretch Database 顾问，识别符合 SQL Server Stretch Database 条件的数据库和表。SQL Server Stretch Database 顾问还可识别阻碍性问题。有关详细信息，请参阅[识别符合 SQL Server Stretch Database 条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

<!--Image references-->
[StretchOverviewImage1]: ./media/sql-server-stretch-database-overview/StretchDBOverview.png
[StretchOverviewImage2]: ./media/sql-server-stretch-database-overview/StretchDBOverview1.png
[StretchOverviewImage3]: ./media/sql-server-stretch-database-overview/StretchDBOverview2.png

<!---HONumber=Mooncake_Quality_Review_1230_2016-->