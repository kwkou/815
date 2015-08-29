<properties 
	pageTitle="连接到 SQL Database：链接、最佳实践和设计准则" 
	description="一个入门主题，其中收集了通过 ADO.NET 和 PHP 等技术连接到 Azure SQL Database 的客户端程序的相关链接和建议。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>
	
<tags 
	ms.service="sql-database" ms.date="03/19/2015" wacn.date=""/>



# 连接到 SQL Database：链接、最佳实践和设计准则


<!--
GeneMi , 2015-April-21 Tuesday 12:44pm
sql-database-connect-central-recommendations.md
sql-database-connect-*.md

Add link to sql-database-develop-quick-start-client-code-samples.md.
Add a paragraph about why transient errors legitimately occur sometimes.
-->


本主题能够很好地帮助你学习如何与 Azure SQL Database 建立客户端连接。其中提供了你可以用来连接到 SQL 数据库以及与之进行交互的各种技术的代码示例的链接。这些技术包括企业库、JDBC、PHP，等等。提供的建议是普遍适用的，与具体的连接技术或编程语言无关。


##独立于技术的建议


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


## 建议：身份验证


- 使用 SQL 数据库身份验证而非 Windows 身份验证。
- 指定一个特定的数据库，而不是默认使用 *master* 数据库。
- 有时候，必须为用户名指定后缀 *@yourservername*，但有时候必须省略后缀。这取决于你的工具或 API 是如何编写的。
 - 请查看有关各项技术的详细信息。
- 通过指定[包含的数据库](http://msdn.microsoft.com/zh-cn/library/ff929071.aspx)中的某个用户进行连接。
 - 此方法可以提供较好的性能和稳定性，因为这不需要登录到 master 数据库。
 - 不能对 SQL 数据库使用 Transact-SQL **USE myDatabaseName;** 语句。




## 建议：连接

- 在你的客户端连接逻辑中，将默认超时替换为 30 秒。
 - 默认值 15 秒对于依赖于 Internet 的连接而言太短。
- 确保你的 [Azure SQL Database 防火墙](http://msdn.microsoft.com/zh-cn/library/ee621782.aspx)允许端口 1433 上的传出 TCP 通信。
 - 你可以在 SQL Database 服务器上或者为单个的数据库配置[防火墙](http://msdn.microsoft.com/zh-cn/library/azure/ee621782.aspx)设置。
- 如果你在使用[连接池](http://msdn.microsoft.com/library/8xx3tyca.aspx)，请在你的程序不活跃地使用连接时将其关闭，而不只是准备着重用它。
 - 除非你的程序会立即（没有停顿）将连接重用于另一操作，否则，建议采用以下模式：<br/><br/>打开连接。<br/>通过连接执行一个操作。<br/>关闭连接。<br/><br/>
- 随你的连接逻辑使用重试逻辑，但仅针对暂时性错误使用。当使用 SQL 数据库时，尝试打开连接或发出查询可能会因各种原因而失败。
 - 导致失败的一个持久原因可能是你的连接字符串的格式不正确。
 - 导致失败的一个暂时原因可能是 Azure SQL 数据库系统需要平衡总体的负载。暂时原因会自己消失，这意味着你的程序应当进行重试。
 - 当重试查询时，请首先关闭连接然后打开另一个连接。



下一部分更详细说明了重试逻辑和暂时性故障处理。


## 暂时性错误和重试逻辑


云服务（例如 Azure 和其 SQL Database 服务）有无止尽的平衡工作负荷和资源管理挑战。如果正在从同一台计算机提供的两个数据库同时进行极繁重的处理，则管理系统可能会检测是否需要将一个数据库的工作负荷转移到有多余容量的另一个资源。


在转移期间，数据库可能暂时无法使用。这可能会阻止新连接，或导致客户端程序断开连接。但资源转移是暂时的，可能在几分钟或几秒钟后就会自行解决。完成转移后，客户端程序可以重新建立连接，然后恢复其工作。与其让客户端程序发生可避免的失败，不如在处理过程中暂停。


发生任何 SQL Database 错误时，将引发 [SqlException](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlexception.aspx)。SqlException 在其 **Number** 属性中包括一个数字错误代码。如果错误代码指出是暂时性错误，你的程序应当重试调用。


- [错误消息 (Azure SQL Database)](http://msdn.microsoft.com/zh-cn/library/azure/ff394106.aspx)
 - “暂时性错误”、“连接断开错误”部分是证明需要自动重试的暂时性错误的列表。
 - 例如，如果出现错误编号 40613，则应当重试，这表示类似于<br/>服务器“theserver”上的数据库“mydatabase”当前不可用。


暂时性*错误*有时也称为暂时性*故障*。本主题将这两个术语视为同义。


如果在遇到连接错误（不管是否是暂时的）时需要进一步的帮助，请参阅：


- [Azure SQL 数据库连接问题故障排除](http://support.microsoft.com/zh-CN/kb/2980233)


有关演示重试逻辑的代码示例主题的链接，请参阅：


- [SQL Database 的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)


<a id="gatewaynoretry" name="gatewaynoretry">&nbsp;</a>


## 网关不再提供 V12 中的重试逻辑


在 V12 以前的版本是，Azure SQL Database 具有可用作代理的网关，用于缓冲数据库和客户端程序之间的所有交互。网关是一个附加的网络跃点，有时会增大数据库访问延迟。


V12 取消了网关。因此现在：


- 客户端程序将与数据库*直接*交互，这更有效率。
- 消除了网关在错误消息以及与程序的其他通信方面的些轻微失真。
 - 对程序而言，SQL Database 和 SQL Server 比较相似。


#### 已取消重试逻辑


网关针对某些暂时性错误提供了重试逻辑。程序现在必定能更全面处理暂时性错误。有关重试逻辑的代码示例，请参阅：


- [SQL Database 的客户端开发和快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)
 - 提供包含重试逻辑的代码示例的链接，以及更简单的连接与查询示例的链接。
- [如何：可靠地连接到 Azure SQL 数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn864744.aspx)
- [如何：使用 ADO.NET 和企业库连接到 Azure SQL Database](http://msdn.microsoft.com/zh-cn/library/azure/dn961167.aspx)
- [如何：使用 ADO.NET 连接到 Azure SQL Database](http://msdn.microsoft.com/zh-cn/library/azure/ee336243.aspx)


## 技术


以下主题包含多个语言和驱动程序技术的代码示例链接，可让你从客户端程序连接到 Azure SQL Database。


针对在 Windows 和 Linux 上运行的客户端提供各种代码示例。


**一般示例：**有各种不同的程序设计语言，包括 PHP、Python、Node.js 和 .NET CSharp 的代码示例。此外，还提供在 Windows、Linux 和 Mac OS X 上运行的客户端示例。


- [SQL Database 的客户端开发和快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)
- [Azure SQL 数据库开发：操作指南主题](http://msdn.microsoft.com/zh-cn/library/azure/ee621787.aspx)

**弹性缩放：**有关连接到弹性缩放数据库的信息，请参阅：


- [Azure SQL Database Elastic Scale 预览版入门](/documentation/articles/sql-database-elastic-scale-get-started)
- [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)


## 另请参阅


- [创建你的第一个 Azure SQL Database](/documentation/articles/sql-database-get-started)

<!---HONumber=66-->