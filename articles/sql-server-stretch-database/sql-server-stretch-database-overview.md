<!-- Remove azure portal -->
<properties
    pageTitle="将冷数据存档到 Azure - Stretch Database | Azure"
    description="了解 Stretch Database 如何透明、安全地将冷数据迁移到 Azure 云。"
    services="sql-server-stretch-database"
    documentationcenter=""
    author="douglaslMS"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="c360dc10-a02b-446f-91a0-278358f7a297"
    ms.service="sql-server-stretch-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="06/27/2016"
    wacn.date="03/24/2017"
    ms.author="douglasl" />  


# Stretch Database 简介
Stretch Database 可以透明、安全地将冷数据迁移到 Azure 云。

如果只是想要立即开始使用 Stretch Database，请参阅[通过运行“为数据库启用延伸”向导开始操作](/documentation/articles/sql-server-stretch-database-wizard/)。

## Stretch Database 的优势是什么？
Stretch Database 提供以下优势：

### 经济高效的为冷数据提供可用性
使用 SQL Server Stretch Database 将冷暖事务数据从 SQL Server 动态延伸到 Azure。与典型的冷数据存储不同，你的数据将始终联机且可供查询。你可以提供更长的数据保留时限，而不会破坏大型表（如客户订单历史记录）的存储库。受益于 Azure 的低成本，你无需扩展价格不菲的本地存储。可以在 Azure 门户预览中选择定价层和配置设置，以掌控费用。根据需要扩展或缩减。有关详细信息，请访问 [SQL Server Stretch Database 定价](/pricing/details/sql-server-stretch-database/)页。

### 无需对查询或应用程序进行更改
无缝访问 SQL Server 数据，无论该数据在本地还是已延伸到云。你可以设置策略来确定数据的存储位置，SQL Server 将在后台处理数据移动。整个表始终会联机且可供查询。此外，Stretch Database 不需要对现有查询或应用程序进行任何更改 - 数据位置对应用程序完全透明。

### 简化本地数据维护
减少数据的本地维护和存储。本地数据的备份运行更快，在维护时段内即可完成。云中数据的备份会自动运行。对本地存储的需求大大降低。与在本地 SSD 中添加数据相比，使用 Azure 存储可将费用较低 80%。

### 即使在迁移期间也确保数据安全
你可以高枕无忧地将最重要的应用程序安全延伸到云中。SQL Server 的 Always Encrypted 为动态数据提供加密。行级别安全性 (RLS) 和其他高级 SQL Server 安全功能也能配合 Stretch Database 保护你的数据。

## Stretch Database 的作用是什么？
为一个 SQL Server 实例、一个数据库和至少一个表启用 Stretch Database 后，Stretch Database 会以静默方式开始将冷数据迁移到 Azure。

* 如果在单独的某个表中存储了冷数据，你可以迁移整个表。

* 如果表同时包含热数据和冷数据，你可以指定筛选器函数以选择要迁移的行。

**无需更改现有的查询和客户端应用。** 即使在数据迁移过程中，你也可以持续无缝访问本地数据和远程数据。远程查询会出现轻微的延迟，但仅当你查询冷数据时，才会遇到这种延迟。

**Stretch Database 可以确保在迁移过程中发生错误时不会丢失数据**。它还包含重试逻辑，用于处理迁移过程中可能发生的连接问题。动态管理视图提供迁移状态。

**你可以暂停数据迁移**以排查本地服务器上的问题或者最大程度地提供可用网络带宽。

![Stretch Database 概述][StretchOverviewImage1]

## Stretch Database 适合你吗？
如果你与下表中的表述相符，则 Stretch Database 可以帮助你满足要求和解决问题。

| 如果你是决策人 | 如果你是数据库管理员|
|------------------------------|-------------------|
| 我必须长期保留事务数据。 | 我的表大小正在失控。|
| 我有时需要查询冷数据。|我的用户指出他们想要访问冷数据，但他们极少使用这些数据。|
| 我不想要更新某些应用，包括早期的应用。 |我必须不断购置更多的存储。 |
| 在存储方面，我想要找到一种省钱的方法。 |我无法在 SLA 的规定范围内备份或还原这种大型表。 |

## 哪种类型的数据库和表符合 Stretch Database 条件？
Stretch Database 面向包含大量冷数据的事务数据库，这些数据通常存储在少量的表中。这些表可能包含超过十亿行。

如果使用 SQL Server 2016 的临时表功能，则可以使用 Stretch Database 将关联的所有或部分历史记录表迁移到 Azure 上的高性价比存储中。有关详细信息，请参阅[管理版本由系统控制的临时表中历史数据的保留期](https://msdn.microsoft.com/zh-cn/library/mt637341.aspx)。

使用 SQL Server 2016 升级顾问的一项功能 Stretch Database 顾问，可以识别符合 Stretch Database 条件的数据库和表。有关详细信息，请参阅[识别符合 Stretch Database 条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。若要详细了解潜在的阻碍性问题，请参阅 [Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)。

## 体验 Stretch Database
**借助 AdventureWorks 示例数据库体验 Stretch Database。** 若要获取 AdventureWorks 示例数据库，请从[此处](https://www.microsoft.com/download/details.aspx?id=49502)至少下载数据库文件以及示例和脚本文件。将示例数据库还原到 SQL Server 2016 实例后，解压缩示例文件，然后从 Stretch DB 文件夹打开 Stretch DB Samples 文件。运行此文件中的脚本可以查看启用 Stretch Database 之前和之后数据所用的空间、跟踪数据迁移的进度，以及确认在数据迁移期间和之后是否可以继续查询现有数据和插入新数据。

## 后续步骤
**识别符合 Stretch Database 条件的数据库和表。** 下载 SQL Server 2016 升级顾问并运行 Stretch Database 顾问，以识别符合 Stretch Database 条件的数据库和表。Stretch Database 顾问还可识别阻碍性问题。有关详细信息，请参阅[识别符合 Stretch Database 条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

<!--Image references-->
[StretchOverviewImage1]: ./media/sql-server-stretch-database-overview/StretchDBOverview.png
[StretchOverviewImage2]: ./media/sql-server-stretch-database-overview/StretchDBOverview1.png
[StretchOverviewImage3]: ./media/sql-server-stretch-database-overview/StretchDBOverview2.png

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update meta properties;wording update-->