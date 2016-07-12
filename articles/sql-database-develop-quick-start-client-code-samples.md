<properties 
	pageTitle="SQL 数据库的客户端快速入门代码示例 | Azure" 
	description="提供 Node.js on Linux、Python on Mac OS、Java 和 Windows、Enterprise Library 和其他许多平台上的 Azure SQL 数据库客户端的代码示例与驱动程序。"
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="01/10/2016" 
	wacn.date="02/26/2016"/>


# SQL 数据库的客户端快速入门代码示例


本主题提供了可用来连接 Azure SQL 数据库的快速入门代码示例的链接：


- 简短示例连接和查询。
- 重试示例连接和查询，但如果遇到的错误分类为[*暂时性故障*](/documentation/articles/sql-database-develop-error-messages/#bkmk_connection_errors)（例如连接超时），则会自动重试。


示例包括：


- 各种编程语言。
- 可运行你的客户端程序的 Windows、Linux 和 Mac OS 操作系统。
- 用于下载任何所需连接驱动程序的链接。
- 简短的快速入门代码示例。
- 较长的示例，其中包含以自动重试逻辑形式处理的暂时性故障。
- 将关系结果集转换为对象导向格式的代码示例。


> [AZURE.NOTE] 我们正在准备更多语言的代码示例，到时将在本主题中添加这些示例的链接。


## Linux 上的客户端


本部分提供在 Linux 上运行的客户端程序的代码示例主题链接。


| 语言 | 简短示例 | 重试示例 | 关系到对象 |
| :-- | :-- | :-- | :-- |
| Node.js | [Tedious](/documentation/articles/sql-database-develop-nodejs-simple-linux/) | 。 | 。 |
| Python | [FreeTDS、pymssql](/documentation/articles/sql-database-develop-python-simple-ubuntu-linux/) | 。 | 。 |
| Ruby | [FreeTDS、TinyTDS](/documentation/articles/sql-database-develop-ruby-simple-linux/) | 。 | 。 |


## Mac OS 上的客户端


本部分提供在 Mac OS 上运行的客户端程序的代码示例主题链接。


| 语言 | 简短示例 | 重试示例 | 关系到对象 |
| :-- | :-- | :-- | :-- |
| Python | [pymssql](/documentation/articles/sql-database-develop-python-simple-mac-osx/) | 。 | 。 |
| Ruby | [Homebrew<br/>FreeTDS、TinyTDS](/documentation/articles/sql-database-develop-ruby-simple-mac-osx/) | 。 | 。 |


## Windows 上的客户端


本部分提供在 Windows 上运行的客户端程序的代码示例主题链接。


| 语言 | 简短示例 | 重试示例 | 关系到对象 |
| :-- | :-- | :-- | :-- |
| C# | [ADO.NET](/documentation/articles/sql-database-develop-dotnet-simple/) | [ADO.NET 自定义](/documentation/articles/sql-database-develop-csharp-retry-windows/)<br/><br/>[Enterprise Library](/documentation/articles/sql-database-develop-entlib-csharp-retry-windows/) | （实体框架） |
| C++ | [ODBC 驱动程序](http://msdn.microsoft.com/zh-cn/library/azure/hh974312.aspx) | 。 | 。 |
| Java | [Java。JDBC、JDK。Insert、Transaction、Select。](/documentation/articles/sql-database-develop-java-simple-windows/) | 。 | 。 |
| Node.js | [msnodesql](/documentation/articles/sql-database-develop-nodejs-simple-windows/) | 。 | 。 |
| PHP | [ODBC](/documentation/articles/sql-database-develop-php-simple-windows/) | [ODBC 自定义](/documentation/articles/sql-database-develop-php-retry-windows/) | 。 |
| Python | [pymssql](/documentation/articles/sql-database-develop-python-simple-windows/) | 。 | 。 |


## 另请参阅


- [适用于多种语言和平台的 SDK 与工具的下载项目](/downloads/#cmd-line-tools)
- [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries/)
- [暂时性错误的数字代码列表](/documentation/articles/sql-database-develop-error-messages/#bkmk_connection_errors)<br/>&nbsp;
- [Azure SQL 数据库开发：操作指南主题](http://msdn.microsoft.com/zh-cn/library/azure/ee621787.aspx)
- [连接到 SQL 数据库：链接、最佳实践和设计准则](/documentation/articles/sql-database-connect-central-recommendations/)
- [创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)
- [GitHub 上的 Entity Framework 6、EF 7](http://entityframework.codeplex.com)

<!---HONumber=Mooncake_0215_2016-->