<properties
   pageTitle="将 Azure 流分析与 SQL 数据仓库配合使用 | Azure"
   description="有关在开发解决方案时将 Azure 流分析与 Azure SQL 数据仓库配合使用的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/03/2016"
   wacn.date="04/25/2016"/>

# 将 Azure 流分析与 SQL 数据仓库配合使用

Azure 流分析是一种完全托管的服务，可以在云中通过流式数据进行低延迟、高度可用、可缩放且复杂的事件处理。你可以先阅读 [Azure 流分析简介][]了解基本知识。然后，可以参考[开始使用 Azure 流分析][]教程，了解如何使用流分析创建端到端解决方案。

在本文中，你将学习如何使用 Azure SQL 数据仓库数据库作为流分析作业的输出接收器。

## 先决条件

首先，完整运行[开始使用 Azure 流分析][]教程中所述的以下步骤。

1. 创建事件中心输入
2. 配置并启动事件生成器应用程序
3. 预配流分析作业
4. 指定作业输入和查询

然后，创建 Azure SQL 数据仓库数据库

## 指定作业输出：Azure SQL 数据仓库数据库

### 步骤 1

在流分析作业中，单击页面顶部的**“输出”**，然后单击**“添加输出”**。

### 步骤 2

选择“SQL 数据库”，然后单击“下一步”。

![][add-output]

### 步骤 3
在下一页输入以下值：

- 输出别名：输入此作业输出的友好名称。
- 订阅：
	- 如果你的 SQL 数据仓库数据库与此流分析作业位于同一订阅中，请选择“使用当前订阅中的 SQL 数据库”。
	- 如果你的数据库在不同的订阅中，请选择“使用其他订阅中的 SQL 数据库”。
- 数据库：指定目标数据库的名称。
- 服务器名称：为刚刚指定的数据库指定服务器名称。


- 用户名：指定具有数据库写入访问权限的帐户的用户名。
- 密码：提供指定的用户帐户的密码。
- 表：指定数据库中目标表的名称。

![][add-database]

### 步骤 4

单击相应勾选按钮以添加此作业输出，并确保流分析可以成功连接到数据库。

![][test-connection]

成功连接到数据库后，门户底部会显示通知。你可以单击底部的“测试连接”来测试与数据库的连接。

## 后续步骤

有关集成的概述，请参阅 [SQL 数据仓库集成概述][]。

有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][]。

<!--Image references-->

[add-output]: ./media/sql-data-warehouse-integrate-azure-stream-analytics/add-output.png
[server-name]: ./media/sql-data-warehouse-integrate-azure-stream-analytics/dw-server-name.png
[add-database]: ./media/sql-data-warehouse-integrate-azure-stream-analytics/add-database.png
[test-connection]: ./media/sql-data-warehouse-integrate-azure-stream-analytics/test-connection.png

<!--Article references-->

[Azure 流分析简介]: /documentation/articles/stream-analytics-introduction/
[开始使用 Azure 流分析]: /documentation/articles/stream-analytics-get-started/
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[SQL 数据仓库集成概述]: /documentation/articles/sql-data-warehouse-overview-integrate/

<!--MSDN references-->

<!--Other Web references-->
[Azure Stream Analytics documentation]: /documentation/services/stream-analytics/


<!---HONumber=Mooncake_0418_2016-->