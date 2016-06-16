<properties
	pageTitle="用于 SQL 数据库和 SQL Server 的连接库"
	description="列出每个客户端程序可以用来连接到 Azure SQL 数据库或 Microsoft SQL Server 的驱动程序的最低版本号。提供一个链接，通过它可查看由社区而不是 Microsoft 发布的驱动程序的版本信息。"
	services="sql-database"
	documentationCenter=""
	authors="pehteh"
	manager="jeffreyg"
	editor="genemi"/>

<tags
	ms.service="sql-database"
	ms.date="02/23/2016"
	wacn.date="06/14/2016"/>

# 用于 SQL 数据库和 SQL Server 的连接库

此主题列出了客户端程序连接到 Azure SQL 数据库或 Microsoft SQL Server 时可以使用的每个库/驱动程序的最低版本号。

## Microsoft 发布的驱动程序库表

下表显示由 Microsoft 发布的库。“库”列提供可用于下载每个库的链接。“版本”列列出推荐用于与 Azure SQL 数据库和 Microsoft SQL Server 交互的最低版本。

| 平台 | 操作系统 | 供下载的库<br/> | 驱动程序版本<br/> | 驱动程序说明<br/> | 详细信息<br/> |
| :--- | :--- | :--- | :--- | :--- | :-- |
| .NET | 跨平台 (.NET) | [ADO.NET, System .Data .SqlClient](http://www.microsoft.com/zh-cn/download/details.aspx?id=30653) | 4\.5+ | 用于 .NET Framework 的 SQL Server 提供程序 | 。 |
| PHP | Windows | [PHP for SQL Server](http://www.microsoft.com/en-us/download/details.aspx?id=20098) | 2\.0+ | 用于 SQL Server 的 PHP 驱动程序 | [链接](http://msdn.microsoft.com/zh-cn/library/dn865013.aspx) |
| Java | Windows | [JDBC for SQL Server](https://www.microsoft.com/zh-cn/download/details.aspx?id=11774) | 2\.0+ | 通过标准 JDBC API 提供数据库连接性的类型 4 JDBC 驱动程序 | [链接](https://msdn.microsoft.com/zh-cn/library/mt654048.aspx) |
| ODBC | Windows | [ODBC for SQL Server](http://www.microsoft.com/zh-cn/download/details.aspx?id=36434) | 11\.0+ | 用于 SQL Server 的 Microsoft ODBC 驱动程序 | [链接](http://msdn.microsoft.com/zh-cn/library/jj730308.aspx) |
| ODBC | Suse Linux | [ODBC for SQL Server](http://www.microsoft.com/en-us/download/details.aspx?id=34687) | 11\.0+ | 用于 SQL Server 的 Microsoft ODBC 驱动程序 | [链接](https://msdn.microsoft.com/zh-cn/library/hh568451.aspx) |
| ODBC | Redhat Linux | [ODBC for SQL Server](http://www.microsoft.com/en-us/download/details.aspx?id=34687) | 11\.0+ | 用于 SQL Server 的 Microsoft ODBC 驱动程序 | [链接](https://msdn.microsoft.com/zh-cn/library/hh568451.aspx) |

### ODBC 支持

使用数据源名称 (DSN) 向导来定义 Azure SQL 数据库的数据源时，可单击“进行 SQL Server 身份验证时，使用用户输入的登录 ID 和密码”选项，然后选择“连接到 SQL Server 以获取其他配置选项的默认设置”。以“登录 ID”和“密码”的形式输入连接到 Azure SQL 数据库服务器所需的用户名和密码。清除“连接到 SQL Server 以获取默认设置…”复选框。单击“将默认数据库更改为:”，然后输入你的 Azure SQL 数据库的名称（即使该名称不显示在列表中）。请注意，该向导在“将 SQL Server 系统消息的语言更改为:”列表中列出了多种语言。

在此版本中，Azure SQL 数据库仅支持英语，因此请选择“英语”作为语言。Azure SQL 数据库不支持“镜像服务器”或“附加数据库”，因此请将这些项目留空。单击“测试连接”。

使用 SQL Server 2008 Native Client ODBC 驱动程序时，单击“测试连接”按钮可能会出现一个错误，指出 **master.dbo.syscharsets** 不受支持。忽略此错误，保存 DSN，随后可使用它。

### 用于 DB2 和 SQL Server 的 OLEDB（供 DRDA 设计使用）

通过 Microsoft OLE DB Provider for DB2 Version 5.0（数据提供程序），你可以创建面向 IBM DB2 数据库的分布式应用程序。该数据提供程序和用于 DB2 的 Microsoft 网络客户端一起利用 Microsoft SQL Server 数据访问架构，客户端起到分布式关系数据库架构 (DRDA) 应用程序请求方的作用。该数据提供程序将 Microsoft Component Object Model (COM) OLE DB 命令和数据类型转换为 DRDA 协议码位和数据格式。

有关详细信息，请参阅：

- [Microsoft OLE DB Provider for DB2 Version 5.0](http://msdn.microsoft.com/zh-cn/library/dn745875.aspx)
- [适用于 Microsoft SQL Server 2012 的 Microsoft OLEDB Provider for DB2 v4.0](http://www.microsoft.com/zh-cn/download/details.aspx?id=29100)

## 第三方库

> [AZURE.IMPORTANT] 下表显示在第三方的许可条款下发布的库。要使用这些库，你有责任验证并遵守相关的第三方许可。使用者自行承担使用这些库的风险。Microsoft 不对此处提供的信息作出任何明示或暗示的保证，仅出于用户方便起见而提供这些信息。此处的任何内容都不表示 Microsoft 的任何认可。
<br/><br/>由开发人员公共社区使用 GitHub.com 上为 **Azure** 所有的 [azure-content](http://github.com/Azure/azure-content) 存储库对此“第三方库”部分中的信息进行更新和维护。Microsoft 鼓励开发人员更新此部分。Microsoft 员工不计划维护此部分中的信息，部分原因是对于每个特定的第三方库，其他人员比我们更加专业。谢谢。

下表显示了其他公司或社区之类的第三方发布的库。Microsoft 只发布了本主题前面部分中所述的库。

| 平台 | 库 |
| :-- | :-- |
| Ruby | [tinytds *(org, stable)*](https://rubygems.org/gems/tiny_tds/versions/0.7.0) |
| GO | [go-mssqldb *(org, stable)*](https://github.com/denisenkom/go-mssqldb) |
| Python | [pymssql *(org, stable)*](http://pymssql.org/en/stable) |
| Node.js | [Node-MSSQL *(npmjs)*](https://www.npmjs.com/package/node-mssql)<br/><br/>[Node-MSSQL-Connector *(npmjs)*](https://www.npmjs.com/package/node-mssql-connector) |
| C++ | [FreeTDS *(org)*](http://www.freetds.org) |

<!--
https://zh.wikipedia.org/wiki/Draft:Microsoft_SQL_Server_Libraries/Drivers
-->

<!---HONumber=Mooncake_0606_2016-->
