<properties 
	pageTitle="到 Azure SQL 数据库的连接中心建议" 
	description="这是一个中心主题，其中提供了有关连接到 Azure SQL 数据库时使用的各种驱动程序（例如 ADO.NET 和 PHP）的更具体主题的链接。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>
	
<tags ms.service="sql-database" ms.date="03/19/2015" wacn.date="04/11/2015"/>



# 到 SQL 数据库的连接中心建议


<!--
GeneMi , 2015-March-19 Thursday 15:41pm
sql-database-connect-central-recommendations.md
sql-database-connect-*.md

Re SqlException, not .HResult, rather .Number.
-->


本主题提供了你可以用来连接到 Azure SQL 数据库以及与之进行交互的各种技术的代码示例的链接。这些技术包括企业库、JDBC 和 PHP。提供的建议是普遍适用的，与具体的连接技术无关。


## 独立于技术的建议


无论你连接到 SQL 数据库时使用哪种具体技术，本部分中的信息都适用。


- [以编程方式连接到 Azure SQL 数据库时的准则](https://msdn.microsoft.com/zh-CN/library/azure/ee336282.aspx) - 讨论了以下内容：
 - 端口
 - 防火墙
 - 连接字符串
- [Azure SQL 数据库资源管理](https://msdn.microsoft.com/zh-CN/library/azure/dn338083.aspx) - 讨论了以下内容：
 - 资源控制
 - 强制实施限制
 - 限制


无论你使用哪种连接技术，SQL 数据库服务器甚至单个数据库的某些防火墙设置都很重要：


- [Azure SQL 数据库防火墙](https://msdn.microsoft.com/zh-CN/library/azure/ee621782.aspx)


### 快速建议


#### 连接


- 在你的客户端连接逻辑中，将默认超时替换为 30 秒。
 - 默认值 15 秒对于依赖于 Internet 的连接而言太短。
- 如果你在使用[连接池](https://msdn.microsoft.com/zh-CN/library/8xx3tyca.aspx)，请在你的程序不活跃地使用连接时将其关闭，而不只是准备着重用它。
 - 除非你的程序会立即（没有停顿）将连接重用于另一操作，否则，建议采用以下模式：
<br/><br/>打开一个连接。
<br/>通过连接执行一个操作。
<br/>关闭连接。<br/><br/>
- 随你的连接逻辑使用重试逻辑，但仅针对暂时性错误使用。当使用 SQL 数据库时，尝试打开连接或发出查询可能会因各种原因而失败。
 - 导致失败的一个持久原因可能是你的连接字符串的格式不正确。
 - 导致失败的一个暂时原因可能是 Azure SQL 数据库系统需要平衡总体的负载。暂时原因会自己消失，这意味着你的程序应当进行重试。
 - 当重试查询时，请首先关闭连接然后打开另一个连接。
- 确保你的 [Azure SQL 数据库防火墙](https://msdn.microsoft.com/zh-CN/library/ee621782.aspx)允许端口 1433 上的传出 TCP 通信。
 - 你可以在 SQL 数据库服务器上或者为单个的数据库配置[防火墙](https://msdn.microsoft.com/zh-CN/library/azure/ee621782.aspx)设置。


#### 身份验证


- 使用 SQL 数据库身份验证而非 Windows 身份验证。
- 指定一个特定的数据库，而不是默认使用  *master* 数据库。
- 有时候，必须为用户名指定后缀 *@yourservername*，但有时候必须省略后缀。这取决于你的工具或 API 是如何编写的。
 - 请查看有关各项技术的详细信息。
- 通过指定[包含的数据库](https://msdn.microsoft.com/zh-CN/library/ff929071.aspx)中的某个用户进行连接。
 - 此方法可以提供较好的性能和稳定性，因为这不需要登录到 master 数据库。
 - 不能对 SQL 数据库使用 Transact-SQL **USE myDatabaseName;** 语句。


### 暂时性错误


某些 SQL 数据库连接错误是持久的，并且没有理由立即重试连接。某些错误是暂时的，建议你的程序自动重试几次。示例：


- *持久的：*如果你在尝试连接时错误拼写了 SQL 数据库服务器的名称，则重试没有意义。
- *暂时的：*如果到 SQL 数据库的正常运行的连接之后被 Azure 限制或负载平衡系统强制中断，建议尝试重新连接。


对 SQL 数据库的调用引发的 [SqlException](https://msdn.microsoft.com/zh-CN/library/system.data.sqlclient.sqlexception.aspx) 在其 **Number** 属性中包括一个数字错误代码。如果错误代码为 1，则它列出为暂时性错误，你的程序应当重试调用。


- [错误消息（Azure SQL 数据库）](https://msdn.microsoft.com/zh-CN/library/azure/ff394106.aspx) - 它的**Connection-Loss Errors** 部分是证明需要重试的暂时性错误的列表。
 - 例如，如果出现错误编号 40613，则应当重试，这表示类似于<br/>*服务器  'theserver' 上的数据库  'mydatabase' 当前不可用之类的消息。*


如果在遇到连接错误（不管是持久的还是暂时的）时需要进一步的帮助，请参阅：


- [Azure SQL 数据库连接问题故障排除](http://support.microsoft.com/zh-CN/kb/2980233)


## 技术


以下主题包含几种连接技术的代码示例：


- [Azure SQL 数据库开发：操作指南主题](https://msdn.microsoft.com/zh-CN/library/azure/ee621787.aspx)


对于包含企业库和暂时性错误处理类的 ADO.NET，请参阅：


- [操作指南：可靠地连接到 Azure SQL 数据库](https://msdn.microsoft.com/zh-CN/library/azure/dn864744.aspx)


对于 Elastic Scale，请参阅：


- [Azure SQL Database Elastic Scale 预览版入门](/documentation/articles/sql-database-elastic-scale-get-started)
- [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)


## 未完成的或过时的帖子


本部分中的链接是指向博客文章或类似来源的，因此它们可能未完成或已过时。但它们可能仍有一些基本价值。


- [针对 Microsoft Azure SQL 数据库中的暂时性错误的重试逻辑](https://social.technet.microsoft.com/wiki/contents/articles/4235.retry-logic-for-transient-failures-in-windows-azure-sql-database.aspx)



<!--HONumber=51-->