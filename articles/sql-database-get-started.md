<properties
	pageTitle="SQL 数据库入门 | Windows Azure"
	description="使用 Azure 门户和 AdventureWorks 示例数据库，在几分钟内创建第一个采用 Azure SQL 数据库（Microsoft 在云中的关系数据库管理服务 (RDBMS)）的云数据库。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="07/15/2015"
	wacn.date=""/>


# 创建你的第一个 Azure SQL Database


本文说明如何在 5 分钟内创建一个示例 Azure SQL 数据库。你将学习：


- 使用 [Azure 门户](http://portal.azure.com/)设置逻辑服务器。
- 创建填有示例数据的数据库。
- 设置数据库的防火墙规则，以配置哪些 IP 地址可以访问你的数据库。


本教程假设你已有 Azure 订阅。如果你没有订阅，可以注册[试用版](/pricing/1rmb-trial/)。


## 步骤 1：登录


1. 登录到 [Azure 门户](http://portal.azure.com/)。
2. 单击“新建”>“数据 + 存储”>“SQL 数据库”。


![新 SQL 数据库][1]


## 步骤 2：创建逻辑服务器



1. 在“SQL 数据库”边栏选项卡中，选择数据库的“名称”，在本示例中为 **AdventureWorks**。
2. 若要创建数据库的逻辑服务器，请依次单击“服务器”、“创建新服务器”。


## 步骤 3：配置服务器


1. 在“服务器”边栏选项卡的“服务器名称”中，输入一个容易记住的唯一名称。
2. 在“服务器管理员登录名”中输入 **AdventureAdmin**。
3. 在“密码”和“确认密码”中输入正确的值。
4. 选择首选地理“位置”。该位置通常比较靠近应用程序的运行位置。
5. 单击**“确定”**。


![创建服务器][2]


## 步骤 4：创建数据库


1. 在“SQL 数据库”边栏选项卡中，单击“选择源”以指定数据库的源。
 - 如果你跳过此步骤，则会创建空数据库。
2. 选择“示例”。
 - 这将创建标准示例数据库的、名为 **AdventureWorks** 的副本数据库。
 - 在 Azure SQL 数据库上，将使用*轻量架构*版本的 AdventureWorks。
3. 单击边栏选项卡底部的“创建”。


## 步骤 5：配置防火墙


以下步骤演示如何指定可访问你数据库的 IP 地址范围。


![浏览服务器][3]


1. 在屏幕左侧的功能区中，依次单击“浏览”、“SQL Server”。
2. 在可用选项中，单击前面创建的 SQL Server。
3. 单击“设置”，然后单击“防火墙”。
4. 通过[必应](http://www.bing.com/search?q=my%20ip%20address)获取你当前的 IP 地址。
5. 在“防火墙设置”中，输入“规则名称”，然后将上一步获取的公共 IP 地址粘贴到“起始 IP”和“结束 IP”字段中。
6. 完成后，单击页面顶部的“保存”。


![防火墙][4]


## 后续步骤


现在，你可以编写一个小型客户端程序用于连接到数据库。有关快速入门代码示例，请单击以下链接之一：


- [使用 C# 连接和查询 SQL Database](/documentation/articles/sql-database-connect-query)
- *即将发布：*SQL 数据库的客户端开发和快速入门示例


<!-- Media references. -->
[1]: ./media/sql-database-get-started/GettingStarted_NewDB.PNG
[2]: ./media/sql-database-get-started/GettingStarted_CreateServer.png
[3]: ./media/sql-database-get-started/GettingStarted_BrowseServer.png
[4]: ./media/sql-database-get-started/GettingStarted_FireWall.png

<!---HONumber=69-->
