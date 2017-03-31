<properties
    pageTitle="标识 Stretch Database 的数据库和表 - Azure | Azure"
    description="了解如何识别符合 Stretch Database 条件的数据库和表。"
    services="sql-server-stretch-database"
    documentationcenter=""
    author="douglaslMS"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="9253c5a2-f135-4782-95ec-8ff39454c2c0"
    ms.service="sql-server-stretch-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2017"
    wacn.date="03/24/2017"
    ms.author="douglasl" />  


# 通过运行 Stretch Database 顾问，识别符合 Stretch Database 条件的数据库和表
若要识别符合 Stretch Database 条件的数据库和表，请下载 SQL Server 2016 升级顾问，然后运行 Stretch Database 顾问。Stretch Database 顾问还可识别阻碍性问题。

## 下载并安装升级顾问
从[此处](http://go.microsoft.com/fwlink/?LinkID=613421)下载并安装升级顾问。SQL Server 安装介媒体中未包含此工具。

## 运行 Stretch Database 顾问

1. 运行升级顾问。

2. 选择“方案”，然后选择“运行 Stretch Database 顾问”。

3. 在“运行 Stretch Database 顾问”边栏选项卡中，单击“选择要分析的数据库”。

4. 在“选择数据库”边栏选项卡上，输入或选择服务器名称和身份验证信息。单击“连接”。

5. 此时会显示所选服务器上的数据库列表。选择要分析的数据库。单击“选择”。

6. 在“运行 Stretch Database 顾问”边栏选项卡中，单击“运行”。随后将运行分析。

## 查看结果

1. 完成分析后，请在“已分析数据库”边栏选项卡中，选择已分析的数据库之一，以显示“分析结果”边栏选项卡。

    “分析结果”边栏选项卡列出了所选数据库中与默认建议条件匹配的推荐表。

2. 在“分析结果”边栏选项卡上的表列表中，选择一个推荐表以显示“表结果”边栏选项卡。

    如果存在阻止问题，“表结果”边栏选项卡会列出所选表的阻止问题。有关 Stretch Database 顾问检测到的阻止问题的信息，请参阅 [Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)。
    
3. 在“表结果”边栏选项卡的阻止问题列表中，选择其中一个问题，以便显示有关所选问题的详细信息并提出缓解步骤。如果你要为 Stretch Database 配置选定的表，请执行建议的缓解步骤。

## 后续步骤
启用 Stretch Database。

* 若要对**数据库**启用 Stretch Database，请参阅[为数据库启用 Stretch Database ](/documentation/articles/sql-server-stretch-database-enable-database/)。

* 若要在已对数据库启用延伸的情况下对另一个**表**启用 Stretch Database，请参阅[为表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)。

## 另请参阅
[Stretch Database 的限制](/documentation/articles/sql-server-stretch-database-limitations/)
[对数据库启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-database/)
[对表启用 Stretch Database](/documentation/articles/sql-server-stretch-database-enable-table/)

<!---HONumber=Mooncake_0320_2017-->
<!-- Update_Description:update meta properties -->