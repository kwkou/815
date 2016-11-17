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
   ms.devlang="NA"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/06/2016"
   wacn.date="10/17/2016"
   ms.author="barbkess;sonyama"/>  


# 查询 Azure SQL 数据仓库 (sqlcmd)

> [AZURE.SELECTOR]
- [Visual Studio](/documentation/articles/sql-data-warehouse-query-visual-studio/)
- [sqlcmd](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/)

本演练使用 [sqlcmd][] 命令行实用程序来查询 Azure SQL 数据仓库。

## 1\.连接

若要开始使用 [sqlcmd][]，请打开命令提示符并输入 **sqlcmd**，后跟 SQL 数据仓库数据库的连接字符串。连接字符串需要以下参数：

+ **服务器 (-S)：**采用 < 服务器名称 >.database.chinacloudapi.cn 格式的服务器
+ **数据库 (-d)：**数据库名称。
+ **启用带引号的标识符 (-I)：**必须启用带引号的标识符才能连接到 SQL 数据仓库实例。

若要使用 SQL Server 身份验证，需添加用户名/密码参数：

+ **用户 (-U)：**采用 `<`用户`>` 格式的服务器用户
+ **密码 (-P)：**与用户关联的密码

例如，你的连接字符串可能如下所示：


	C:\>sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I


若要使用 Azure Active Directory 集成身份验证，需添加 Azure Active Directory 参数：

+ **Azure Active Directory 身份验证 (-G)：**使用 Azure Active Directory 进行身份验证

例如，你的连接字符串可能如下所示：


	C:\>sqlcmd -S MySqlDw.database.chinacloudapi.cn -d Adventure_Works -U myuser -P myP@ssword -I


> [AZURE.NOTE] 需[启用 Azure Active Directory 身份验证](/documentation/articles/sql-data-warehouse-authentication/)才能使用 Active Directory 进行身份验证。

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

请参阅 [sqlcmd 文档][sqlcmd]，详细了解 sqlcmd 中的可用选项。

<!--Image references-->


<!--Article references-->

<!--MSDN references--> 
[sqlcmd]: https://msdn.microsoft.com/zh-cn/library/ms162773.aspx
[Azure portal]: https://portal.azure.cn

<!--Other Web references-->

<!---HONumber=Mooncake_1010_2016-->