<properties
	pageTitle="SQL Server Stretch Database概述 | Azure"
	description="了解SQL Server Stretch Database如何透明、安全地将历史数据迁移到 Azure 云。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# SQL Server Stretch Database概述

SQL Server Stretch Database可以透明、安全地将历史数据迁移到 Azure 云。

## SQL Server Stretch Database的优势是什么？
SQL Server Stretch Database提供以下优势：

**以经济高效的方式为冷数据提供可用性**
使用 SQL Server SQL Server Stretch Database将冷暖事务数据从 SQL Server 动态延伸到 Azure。与典型的冷数据存储不同，你的数据将始终联机且可供查询。你可以提供更长的数据保留时限，而不会破坏大型表（如客户订单历史记录）的存储库。受益于 Azure 的低成本，你无需扩展价格不菲的本地存储。你可以选择定价层并在 Azure 门户中配置设置，以掌控价格与数据访问速度。根据需要扩展或缩减。有关详细信息，请访问 [SQL 数据库定价](/home/features/sql-database/pricing/)页。

**不需要更改查询或应用程序**
不管 SQL Server 数据是在本地还是已延伸到云中，你都可以顺畅访问这些数据。你可以设置策略来确定数据的存储位置，SQL Server 将在后台处理数据移动。整个表始终会联机且可供查询。此外，SQL Server Stretch Database不需要对现有查询或应用程序进行任何更改 – 数据位置对应用程序完全透明。

**简化本地数据维护**
减少数据的本地维护和存储。云中数据的备份会自动运行。本地数据的备份运行更快，在维护时段内即可完成。对本地存储的需求大大降低。与在本地 SSD 中添加数据相比，使用 Azure 存储空间可将费用较低 80%。

**即使在迁移期间也能保持数据的安全性**
你可以高枕无忧地将最重要的应用程序安全延伸到云中。SQL Server 的“始终加密”为动态数据提供加密。行级别安全性 (RLS) 和其他高级 SQL Server 安全功能也能配合SQL Server Stretch Database来保护你的数据。

## SQL Server Stretch Database的作用是什么？
为一个 SQL Server 实例、一个数据库和至少一个表启用SQL Server Stretch Database后，SQL Server Stretch Database会以静默方式开始将历史数据迁移到 Azure。

-   如果在单独的某个表中存储了历史数据，你可以迁移整个表。

-   如果表包含历史数据和当前数据，你可以指定筛选器谓词来选择要迁移的行。

SQL Server Stretch Database可以确保在迁移过程中发生错误时不会丢失数据。它还包含重试逻辑用于处理迁移过程中可能发生的连接问题。动态管理视图提供迁移状态。

你可以暂停数据迁移以排查本地服务器上的问题或者最大程度地提供可用网络带宽。

无需更改现有的查询和客户端应用。即使在数据迁移过程中，你也可以持续顺畅访问本地数据和远程数据。远程查询会出现轻微的延迟，但仅当你查询历史数据时，才会遇到这种延迟。

![SQL Server Stretch Database概述][StretchOverviewImage1]

## SQL Server Stretch Database适合你吗？
如果你与下表中的表述相符，则SQL Server Stretch Database可以帮助你满足要求和解决问题。

|如果你是决策人|如果你是数据库管理员|
|------------------------------|-------------------|
|我必须长期保留事务数据。|我的表大小正在失控。|
|我有时需要查询历史数据。|我的用户指出他们想要获得历史数据访问权限，但他们极少使用这些数据。|
|我不想要更新某些应用，包括早期的应用。|我必须不断购置更多的存储。|
|在存储方面，我想要找到一种省钱的方法。|我无法在 SLA 的规定范围内备份或还原这种大型表。|

## 哪种类型的数据库和表符合SQL Server Stretch Database条件？
SQL Server Stretch Database面向包含大量历史数据的事务数据库，这些数据通常存储在少量的表中。这些表可能包含超过十亿行。

如果使用 SQL Server 2016 的历史表功能，则可以使用SQL Server Stretch Database将关联的所有或部分历史记录表迁移到 Azure 上的高性价比存储中。有关详细信息，请参阅[管理版本由系统控制的临时表中历史数据的保留期](https://msdn.microsoft.com/zh-cn/library/mt637341.aspx)。

使用 SQL Server 2016 升级顾问的一项功能 - SQL Server Stretch Database顾问 - 可以识别符合SQL Server Stretch Database条件的数据库和表。有关详细信息，请参阅[识别符合SQL Server Stretch Database条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。若要详细了解潜在的阻碍性问题，请参阅[SQL Server Stretch Database的外围应用限制与阻碍性问题](/documentation/articles/sql-server-stretch-database-limitations/)。

## <a name="FAQ"></a>有关SQL Server Stretch Database的常见问题
**SQL Server Stretch Database是否能够配合 &lt;SQL Server 功能名称&gt; 工作？**
-   关能使某个表符合延伸条件的 SQL Server 功能的列表，请参阅[SQL Server Stretch Database的外围应用限制与阻碍性问题](/documentation/articles/sql-server-stretch-database-limitations/)。

-   （可选）下载 SQL Server 2016 升级顾问并运行SQL Server Stretch Database顾问，以识别符合SQL Server Stretch Database条件的数据库和表。有关详细信息，请参阅[识别符合SQL Server Stretch Database条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

**我是否可以针对另一个本地 SQL Server 实例使用SQL Server Stretch Database？**
不可以。SQL Server Stretch Database不支持使用另一个本地 SQL Server 实例作为远程终结点。

**我是否可以禁用延伸并将迁移的数据移回到本地表？**
是的。有关详细信息，请参阅[禁用SQL Server Stretch Database和移回远程数据](/documentation/articles/sql-server-stretch-database-disable/)。

## 术语
**本地数据库**。本地 SQL Server 数据库。

**远程终结点**。Azure 中包含数据库远程数据的位置。

**本地数据**。根据已启用SQL Server Stretch Database的数据库中表的延伸数据库配置，不会移到 Azure 的数据。

**符合条件的数据**。已启用SQL Server Stretch Database的数据库中尚未移动，但会根据数据库中表的SQL Server Stretch Database配置移到 Azure 的数据。

**远程数据**。已启用SQL Server Stretch Database的数据库中已移到 Azure 的数据。

## 体系结构
SQL Server Stretch Database利用 Azure 中的资源来减轻存档数据存储与查询处理负载。

对某个数据库启用SQL Server Stretch Database后，SQL Server Stretch Database将在本地 SQL Server 中创建安全的链接服务器定义。此链接服务器定义使用远程终结点作为目标。对数据库中的某个表启用SQL Server Stretch Database后，SQL Server Stretch Database将预配远程资源，并开始迁移符合条件的数据（如果已启用迁移）。

针对已启用SQL Server Stretch Database的表的查询，会自动针对本地数据库和远程终结点运行。SQL Server Stretch Database利用 Azure 中的处理能力，通过重新编写查询来针对远程数据运行查询。你可以在新的查询计划中将这种重新编写视为“远程查询”运算符。

![SQL Server Stretch Database体系结构][StretchOverviewImage2]

## 安全性和权限

### 为 SQL Server 实例启用和禁用SQL Server Stretch Database
若要开始为SQL Server Stretch Database配置数据库，必须先使用 sp\_configure 更改“remote data archive”实例级配置选项。此操作需要 sysadmin 或 serveradmin 特权。启用此选项后，即可为SQL Server Stretch Database配置数据库、迁移数据，以及查询远程终结点上的数据。

### 为数据库或表启用和禁用SQL Server Stretch Database
若要为SQL Server Stretch Database配置数据库，你必须拥有 CONTROL DATABASE 权限。此外，必须提供远程终结点的管理员凭据。

若要为SQL Server Stretch Database配置某个表，你必须对该表拥有 ALTER 权限，并且必须为SQL Server Stretch Database配置了数据库。

### 数据访问
只有系统进程才能访问SQL Server Stretch Database后面的链接服务器定义。用户登录名无法通过链接服务器定义向远程终结点发出查询。

SQL Server Stretch Database不会更改现有数据库的权限模型。用户登录名可以通过本地数据库查询已启用SQL Server Stretch Database的表中的数据。对于用户发起的任何操作，本地数据库将执行权限检查，检查方式与它对任何其他对象执行的检查一样。如果你已被授权访问已启用SQL Server Stretch Database的表，则就有权访问该表中你已获授权访问的内容，不管这些数据实际上位于哪个位置。

![SQL Server Stretch Database数据访问模型][StretchOverviewImage3]

## 体验SQL Server Stretch Database
**借助 AdventureWorks 示例数据库体验SQL Server Stretch Database。** 若要获取 AdventureWorks 示例数据库，请从[此处](https://www.microsoft.com/download/details.aspx?id=49502)至少下载数据库文件以及示例和脚本文件。将示例数据库还原到 SQL Server 2016 实例后，解压缩示例文件，然后从 Stretch DB 文件夹打开 Stretch DB Samples 文件。运行此文件中的脚本可以查看启用SQL Server Stretch Database之前和之后数据所用的空间，跟踪数据迁移的进度，以及确认在数据迁移期间和之后是否可以继续查询现有数据和插入新数据。

## 后续步骤
**识别符合SQL Server Stretch Database条件的数据库和表。** 下载 SQL Server 2016 升级顾问并运行SQL Server Stretch Database顾问，以识别符合SQL Server Stretch Database条件的数据库和表。SQL Server Stretch Database顾问还可识别阻碍性问题。有关详细信息，请参阅[识别符合SQL Server Stretch Database条件的数据库和表](/documentation/articles/sql-server-stretch-database-identify-databases/)。

<!--Image references-->
[StretchOverviewImage1]: ./media/sql-server-stretch-database-overview/StretchDBOverview.png
[StretchOverviewImage2]: ./media/sql-server-stretch-database-overview/StretchDBOverview1.png
[StretchOverviewImage3]: ./media/sql-server-stretch-database-overview/StretchDBOverview2.png

<!---HONumber=Mooncake_0307_2016-->