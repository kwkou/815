<properties 
	pageTitle="Azure SQL 数据库连接问题" 
	description="识别和确定 SQL 数据库连接失败。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="07/24/2015" 
	wacn.date="09/15/2015"/>


# SQL 数据库连接问题

本文提供有关在无法连接到 Azure SQL 数据库时，如何使用各种故障排除方法的概述。


## 确定连接问题是否只与某个特定的应用程序相关，或者你是否无法连接到数据库。

使用 SQL Server Management Studio 或 SQLCMD.EXE 验证是否可以连接到数据库。

- 有关使用 SQL Server Management Studio (SSMS) 连接到 SQL 数据库的说明，请参阅[如何使用 SSMS 连接到 Azure SQL 数据库](/documentation/articles/sql-database-connect-to-database)。
- 有关使用 SQLCMD 连接到 SQL 数据库 的说明，请参阅[如何：使用 sqlcmd 连接到 Azure SQL 数据库](https://msdn.microsoft.com/zh-cn/library/azure/ee336280.aspx)。



## 尝试连接到 SQL 数据库的应用程序是否在 Azure 中运行（例如，无法连接到数据库的应用程序是否为云服务或 Web 应用程序）？

确保为服务器/数据库启用了允许所有 Azure 服务的防火墙规则。

- 有关防火墙规则和启用来自 Azure 的连接的信息，请参阅 [Azure SQL 数据库 防火墙](https://msdn.microsoft.com/zh-cn/library/azure/ee621782.aspx#ConnectingFromAzure)。



## 尝试连接到 SQL 数据库的应用程序是否来自专用网络？

确保为服务器/数据库启用了允许从特定网络进行访问的防火墙规则。

- 有关防火墙的一般信息，请参阅 [Azure SQL 数据库 防火墙](https://msdn.microsoft.com/zh-cn/library/azure/ee621782.aspx)。
- 有关设置防火墙规则的说明，请参阅[如何：配置防火墙设置 (Azure SQL 数据库)](https://msdn.microsoft.com/zh-cn/library/azure/jj553530.aspx)。


## 如果防火墙规则配置正确，请尝试[排查 Windows Azure SQL 数据库 连接问题](https://support2.microsoft.com/common/survey.aspx?scid=sw;en;3844&showpage=1)引导式演练。

上述支持文章针对以下 SQL 数据库连接问题提供了帮助：

- 由于防火墙设置问题而无法打开服务器连接 
- 找不到或无法访问服务器 
- 无法登录到服务器 
- 连接超时错误 
- 暂时性错误 
- 由于达到了系统定义的某个限制而导致连接终止 
- 从 SQL Server Management Studio (SSMS) 连接失败 


## 其他信息

- 有关连接到 SQL 数据库 的其他信息，请参阅[以编程方式连接到 Azure SQL 数据库时的准则](https://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx)。   

- 有关具体连接错误的详细信息，请参阅[SQL 数据库客户端程序的错误消息](https://msdn.microsoft.com/zh-cn/library/azure/ff394106.aspx#bkmk_connection_errors) 中的**暂时性故障和连接断开错误**。

- 可通过使用 [**sys.event\_log（Azure SQL 数据库）**](https://msdn.microsoft.com/zh-cn/library/dn270018.aspx)视图查询连接事件，来访问连接事件数据。

- 可通过查询 [**sys.database\_connection\_stats（Azure SQL 数据库）**](https://msdn.microsoft.com/zh-cn/library/dn269986.aspx)视图，来访问数据库连接事件的度量值。

 

<!---HONumber=69-->