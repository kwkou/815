<properties pageTitle="通过运行SQL Server Stretch Database顾问来识别符合SQL Server Stretch Database条件的数据库和表 | Azure"
	description="了解如何识别符合SQL Server Stretch Database条件的数据库和表。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# 通过运行SQL Server Stretch Database顾问来识别符合SQL Server Stretch Database条件的数据库和表

若要识别符合SQL Server Stretch Database条件的数据库和表，请下载 SQL Server 2016 升级顾问，然后运行SQL Server Stretch Database顾问。SQL Server Stretch Database顾问还可识别阻碍性问题。

## 下载并安装升级顾问
从[此处](http://go.microsoft.com/fwlink/?LinkID=613421)下载并安装升级顾问。SQL Server 安装介质中未包含此工具。

## 运行SQL Server Stretch Database顾问

1.  运行升级顾问。

2.  选择“方案”，然后选择“运行SQL Server Stretch Database顾问”。

3.  在“运行SQL Server Stretch Database顾问”边栏选项卡中，单击“选择要分析的数据库”。

4.  在“选择数据库”边栏选项卡中，单击“SQL 实例”。

5.  在“连接到 SQL 实例”边栏选项卡中，输入 SQL Server 实例的名称。单击“连接”。

6.  在“选择数据库”边栏选项卡中，选择要分析的数据库。然后单击“选择”。

7.  在“运行SQL Server Stretch Database顾问”边栏选项卡中，单击“运行”。随后将运行分析。

## 查看结果

1.  完成分析后，请在“SQL Server Stretch Database顾问”边栏选项卡中选择已分析的数据库之一，以显示“分析结果”边栏选项卡。

    “分析结果”边栏选项卡列出了所选数据库中与默认建议条件匹配的推荐表。（可选）调整最小大小和行计数，以扩大或缩小推荐表的列表。

2.  在“分析结果”边栏选项卡上的推荐表列表中，选择一个推荐表以显示“表结果”边栏选项卡。

    “表结果”边栏选项卡列出了所选表的阻碍性问题。有关SQL Server Stretch Database顾问检测到的阻碍性问题的信息，请参阅[SQL Server Stretch Database的外围应用限制与阻碍性问题](/documentation/articles/sql-server-stretch-database-limitations/)。

3.  在“表结果”边栏选项卡上的阻碍性问题列表中，选择一个问题以显示“规则结果”边栏选项卡。

    “规则结果”边栏选项卡描述了所选问题并提供了建议的缓解步骤。如果你要为SQL Server Stretch Database配置选定的表，请执行建议的缓解步骤。

## 后续步骤
启用SQL Server Stretch Database。

-   若要对**数据库**启用SQL Server Stretch Database，请参阅[为数据库启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)。

-   若要在已对数据库启用延伸的情况下对另一个**表**启用SQL Server Stretch Database，请参阅[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)。

## 另请参阅
[SQL Server Stretch Database的外围应用限制与阻碍性问题](/documentation/articles/sql-server-stretch-database-limitations/)
[为数据库启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)
[为表启用SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)

<!---HONumber=Mooncake_0307_2016-->