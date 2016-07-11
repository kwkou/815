<!-- Remove Azure portal -->
<properties
   pageTitle="连接到 Azure SQL 数据仓库 | Azure"
   description="连接到 Azure SQL 数据仓库的连接概述"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="06/20/2016"
   wacn.date="07/11/2016"/>

# 连接到 Azure SQL 数据仓库

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-connect-overview)
- [身份验证](/documentation/articles/sql-data-warehouse-authentication)
- [驱动程序](/documentation/articles/sql-data-warehouse-connection-strings)

连接到 Azure SQL 数据仓库的概述。

## 发现门户中的连接字符串

SQL 数据仓库与 Azure SQL 服务器相关联。若要连接，需要服务器的完全限定名称 (**servername**.database.chinacloudapi.cn*)。

<!-- 若要查找完全限定的服务器名称，请执行以下操作：

1. 转到 [Azure 门户][]。
2. 单击“SQL 数据库”，然后单击想要连接的数据库。本示例使用 AdventureWorksDW 示例数据库。
3. 找到完整的服务器名称。

    ![完整服务器名称][1]
-->

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

但是，SQL 数据仓库是分布式 MPP（大规模并行处理）系统，相比于 SQL Server，会话和请求标识符的公开方式有所不同。

会话和请求由各自的标识符以逻辑方式表示。

| 标识符 | 示例值 |
| :--------- | :------------ |
| 会话 ID | SID123456 |
| 请求 ID | QID123456 |

请注意，会话标识符的前缀为 SID - 会话标识符的简写 - 请求的前缀为 QID，它是查询标识符的简写。

<!-- 你需要此信息帮助在监视查询性能时标识查询。 可以使用 [Azure 门户]和动态管理视图来监视查询性能。-->

此查询标识当前会话。


    SELECT SESSION_ID()
    ;


若要查看当前正在运行或近期曾针对数据仓库运行的所有查询，可以使用以下查询。此操作将创建一个视图，然后运行该视图。


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
    
    SELECT * FROM dbo.vSessionRequests;


## 后续步骤

若要开始使用 Visual Studio 和其他应用程序查询数据仓库，请参阅[使用 Visual Studio 进行查询][]。


<!--Arcticles-->

[使用 Visual Studio 进行查询]: /documentation/articles/sql-data-warehouse-query-visual-studio

<!--Other-->
[Azure 门户]: https://portal.azure.com

<!--Image references-->

[1]: media/sql-data-warehouse-connect-overview/get-server-name.png



<!---HONumber=Mooncake_0704_2016-->