<properties
   pageTitle="连接到 SQL 数据仓库 | Azure"
   description="有关在开发解决方案时连接到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# 连接到 SQL 数据仓库 
若要连接到 SQL 数据仓库，需要传入安全凭据进行身份验证。建立连接时，你还会看到特定的连接设置已配置为建立查询会话的一部分。

本文详细说明有关连接到 SQL 数据仓库的以下方面：

- 身份验证
- 连接设置
- 会话和请求标识符


## 身份验证
若要连接到 SQL 数据仓库，需要提供以下信息：

- 完全限定的服务器名称 
- 指定 SQL 身份验证
- 用户名 
- 密码
- 默认数据库（可选）

请务必注意，用户必须使用 SQL 身份验证。目前不支持受信任身份验证。

默认情况下，你的连接将连接到 master 数据库而不是用户数据库。若要连接到用户数据库，可以选择执行以下两项操作之一：

1. 在 SSDT 或应用程序连接字符串中将你的服务器注册到 SQL Server 对象资源管理器时指定默认数据库。例如，包含 ODBC 连接的 InitialCatalog 参数。
2. 在 SSDT 中创建会话之前先突出显示用户数据库。

> [AZURE.NOTE] 有关使用 SSDT 连接到 SQL 数据仓库的指导，请参阅[连接和查询][]入门文章。

再次强调，必须注意，不支持使用 Transact-SQL 语句 **USE<your DB>** 更改连接的数据库

## 应用程序连接协议
可以使用以下任一协议连接到 SQL 数据仓库：

- ADO.NET
- ODBC
- PHP
- JDBC

### 示例 ADO.NET 连接字符串

```
Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

### 示例 ODBC 连接字符串

```
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.chinacloudapi.cn,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

### 示例 PHP 连接字符串

```
Server: {your_server}.database.chinacloudapi.cn,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( "sqlsrv:server = tcp:{your_server}.database.chinacloudapi.cn,1433; Database = {your_database}", "{your_user_name}", "{your_password_here}");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( "Error connecting to SQL Server." );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array("UID" => "{your_user_name}", "pwd" => "{your_password_here}", "Database" => "{your_database}", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);\r\n$serverName = "tcp:{your_server}.database.chinacloudapi.cn,1433";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

### 示例 JDBC 连接字符串

```
jdbc:sqlserver://yourserver.database.chinacloudapi.cn:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.chinacloudapi.cn;loginTimeout=30;
```

## 连接设置
SQL 数据仓库在连接和创建对象期间标准化一些设置。无法重写这些设置。

| 数据库设置 | 值 |
| :----------------- | :--------------------------- |
| ANSI\_NULLS | 开 |
| QUOTED\_IDENTIFIERS | 开 |
| NO\_COUNT | 关 |
| DATEFORMAT | mdy |
| DATEFIRST | 7 |
| 数据库排序规则 | SQL\_Latin1\_General\_CP1\_CI\_AS |

## 会话和请求
创建连接和会话之后，可以编写查询并将其提交到 SQL 数据仓库。

每个查询由一个或多个请求标识符表示。在该连接上提交的所有查询都是单个会话的一部分，因此将以单个会话标识符表示。

但是，SQL 数据仓库是分布式 MPP 系统，相比于 SQL Server，会话和请求标识符的公开方式有所不同。

会话和请求由各自的标识符以逻辑方式表示。
	
| 标识符 | 示例值 |
| :--------- | :------------ |
| 会话 ID | SID123456 |
| 请求 ID | QID123456 |

请注意，会话标识符的前缀为 SID - 会话标识符的简写 - 请求的前缀为 QID，它是查询标识符的简写。

你需要此信息帮助在监视查询性能时标识查询。可以使用 [Azure 门户] 和动态管理视图来监视查询性能。

使用以下函数识别当前正在使用哪个会话：

```
SELECT SESSION_ID()
;
```

若要查看当前正在运行或近期曾针对数据仓库运行的所有查询，可以使用类似于下面的查询：

```
CREATE VIEW dbo.vSessionRequests
AS
SELECT 	 s.[session_id]									AS Session_ID
		,s.[status]										AS Session_Status
		,s.[login_name]									AS Session_LoginName
		,s.[login_time]									AS Session_LoginTime
        ,r.[request_id]									AS Request_ID
		,r.[status]										AS Request_Status
		,r.[submit_time]								AS Request_SubmitTime
		,r.[start_time]									AS Request_StartTime
		,r.[end_compile_time]							AS Request_EndCompileTime
		,r.[end_time]									AS Request_EndTime
		,r.[total_elapsed_time]							AS Request_TotalElapsedDuration_ms
        ,DATEDIFF(ms,[submit_time],[start_time])		AS Request_InitiateDuration_ms
        ,DATEDIFF(ms,[start_time],[end_compile_time])	AS Request_CompileDuration_ms
        ,DATEDIFF(ms,[end_compile_time],[end_time])		AS Request_ExecDuration_ms
		,[label]										AS Request_QueryLabel
		,[command]										AS Request_Command
		,[database_id]									AS Request_Database_ID
FROM    sys.dm_pdw_exec_requests r
JOIN    sys.dm_pdw_exec_sessions s	ON	r.[session_id] = s.[session_id]
WHERE   s.[session_id] <> SESSION_ID()
;
```

请注意，此查询已封装在视图中，因此可以将其合并到解决方案。但是，若要查看结果，你需要创建并执行视图。

## 后续步骤
连接后，可以开始设计你的表。有关更多详细信息，请参阅[表设计]一文。

<!--Image references-->

<!--Azure.com references-->
[连接和查询]: /documentation/articles/sql-data-warehouse-get-started-connect/
[表设计]: /documentation/articles/sql-data-warehouse-develop-table-design/

<!--MSDN references-->

<!--Other references-->

<!---HONumber=Mooncake_0215_2016-->