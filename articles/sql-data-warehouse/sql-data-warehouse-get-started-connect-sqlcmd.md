<!-- Remove Visual Studio temprorily, doc added next time -->
<properties
   pageTitle="查询 Azure SQL 数据仓库 (sqlcmd)| Azure"
   description="使用 sqlcmd 命令行实用工具查询 Azure SQL 数据仓库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/16/2016"
   wacn.date="07/04/2016"/>

# 查询 Azure SQL 数据仓库 (sqlcmd)

> [AZURE.SELECTOR]
- [sqlcmd](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/)
<!-- - [Visual Studio](/documentation/articles/sql-data-warehouse-query-visual-studio/) -->

本演练使用 sqlcmd 命令行实用工具来查询 Azure SQL 数据仓库。

## 先决条件

若要逐步完成本教程，你需要：

## 获取完全限定的 Azure SQL 服务器名称

若要连接到数据库，你需要服务器的完整名称 (***servername**.database.chinacloudapi.cn*)，该名称中包含要连接到的数据库。

1. 转到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 浏览到要连接到的数据库。
3. 点击“仪表板”，找出完整的服务器名称（我们将在下面的步骤中使用此名称）：

## 使用 sqlcmd 连接到 SQL 数据仓库

## 1\.连接

若要开始使用 sqlcmd，请打开命令提示符并输入 **sqlcmd**，后跟 SQL 数据仓库数据库的连接字符串。连接字符串需包含以下必需参数：

+ **服务器 (-S)：**采用 < 服务器名称 >.database.chinacloudapi.cn 格式的服务器
+ **数据库 (-d)：**数据库名称。
+ **用户 (-U)：**采用  < 用户 > 格式的服务器用户
+ **密码 (-P)：**与用户关联的密码
+ **启用带引号的标识符 (-I)：**必须启用带引号的标识符才能连接到 SQL 数据仓库实例。

例如，你的连接字符串可能如下所示：


	C:\>sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I

> [AZURE.NOTE] 当前需要启用了带引号标识符的 -I 选项来连接到 SQL 数据仓库。

## 2\.查询

连接后，可以对实例发出任何支持的 Transact-SQL 语句。在此示例中，查询以交互模式进行提交。


	C:\>sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I
	1> SELECT name FROM sys.tables;
	2> GO
	3> QUIT


后续示例演示如何通过使用 -Q 选项或将 SQL 输送到 sqlcmd 从而在批处理模式下运行查询。


	C:\>sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"



	C:\>"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out


## 后续步骤

若要了解有关所有 sqlcmd 选项的信息，请参阅 [sqlcmd 文档][sqlcmd.exe]。

<!--Articles-->
[connecting with PowerBI]: /documentation/articles/sql-data-warehouse-integrate-power-bi/


<!--Other-->
[sqlcmd.exe]: https://msdn.microsoft.com/zh-cn/library/ms162773.aspx
[Microsoft Command Line Utilities 11 for SQL Server]: http://go.microsoft.com/fwlink/?LinkId=321501
[Azure portal]: https://manage.windowsazure.cn

<!--Image references-->


<!---HONumber=Mooncake_0627_2016-->