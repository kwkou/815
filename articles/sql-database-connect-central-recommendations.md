<properties 
	pageTitle="连接到 SQL 数据库：链接、最佳实践和设计准则" 
	description="一个入门主题，其中收集了通过 ADO.NET 和 PHP 等技术连接到 Azure SQL 数据库的客户端程序的相关链接和最佳实践建议。" 
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="09/15/2015" 
	wacn.date="10/17/2015"/>


# 连接到 SQL 数据库：最佳实践和设计准则


本主题能够很好地帮助你学习如何与 Azure SQL 数据库建立客户端连接。其中提供了你可以用来连接到 SQL 数据库以及与之进行交互的各种技术的代码示例的链接。这些技术包括企业库、JDBC、PHP，等等。无论你使用何种具体技术连接到 SQL 数据库，所提供的信息均适用。


## 独立于技术的建议


- [以编程方式连接到 Azure SQL 数据库时的准则](http://msdn.microsoft.com/zh-cn/library/azure/ee336282.aspx) - 讨论了以下内容：
 - [端口和防火墙](/documentation/articles/sql-database-configure-firewall-settings)
 - 连接字符串
- [Azure SQL 数据库资源管理](https://msdn.microsoft.com/zh-CN/library/azure/dn338083.aspx) - 讨论了以下内容：
 - 资源控制
 - 强制实施限制
 - 限制


## 身份验证建议


- 使用 Azure SQL 数据库身份验证，而不使用 Windows 身份验证，因为其在 Azure SQL 数据库中不可用。
- 指定一个特定的数据库，而不是默认使用 *master* 数据库。
 - 你无法使用 SQL 数据库上的 Transact-SQL **USE myDatabaseName;** 语句切换到其他数据库。


### 包含用户


将一个人以用户身份添加到你的 SQL 数据库时，你可以选择：

- 为 **master** 数据库添加一个带有密码的*登录*，然后将相应的*用户*添加到同一服务器上的一个或多个其他数据库。

- 将一个带有密码的*包含用户*添加到一个或多个数据库，且没有链接到 **master** 中的任何*登录*。


包含用户方法既有优点也有缺点：

- 优点是当数据库中的所有用户都是包含用户时，可以轻松地将数据库从一个 Azure SQL 数据库服务器移至另一个服务器。

- 缺点是日常管理面临更大困难。例如：
 - 与删除一个登录相比，删除多个包含用户更具挑战性。
 - 如果一个人在多个数据库中都是包含用户，则可能有更多的密码需要记住或更新。


有关进一步信息，请参阅[包含数据库](http://msdn.microsoft.com/zh-cn/library/ff929071.aspx)。


## 连接建议


- 在你的客户端连接逻辑中，将默认超时替换为 30 秒。
 - 默认值 15 秒对于依赖于 Internet 的连接而言太短。


- 确保你的 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure)允许端口 1433 上的传出 TCP 通信。
 - 你可以在 SQL 数据库服务器上或者为单个的数据库配置防火墙设置。


- 如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开虚拟机上的端口范围 11000-11999 和 14000-14999。请单击[此处](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)了解详细信息。


- 若要处理*暂时性故障*，请将[*重试*逻辑](#TransientFaultsAndRetryLogicGm)添加到与 Azure SQL 数据库交互的客户端程序。


### 连接池


如果你在使用[连接池](http://msdn.microsoft.com/zh-cn/library/8xx3tyca.aspx)，请在你的程序不活跃地使用连接时将其关闭，而不是准备重用它。

除非你的程序会立即（没有停顿）将连接重用于另一操作，否则，建议采用以下模式：

- 打开一个连接。
- 通过连接执行一个操作。
- 关闭连接。


#### 在使用池时引发的异常


启用了连接池并且发生超时错误或其他登录错误时，将引发异常。后续连接尝试将在接下来的 5 秒钟内停止，这种行为称为*阻塞期*。

如果应用程序尝试在阻塞期内连接，将再次引发第一个异常。阻塞期结束后，后续故障会导致一个新的阻塞期，其持续时间是前一阻塞期的两倍。

阻塞期最长持续时间为 60 秒。


### V12 中的非 1433 端口


与 Azure SQL 数据库 V12 建立的客户端连接有时会绕过代理直接与数据库交互。除 1433 以外的端口变得非常重要。有关详细信息，请参阅：<br/>[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12)


下一部分更详细说明了重试逻辑和暂时性故障处理。



<a name="TransientFaultsAndRetryLogicGm" id="TransientFaultsAndRetryLogicGm"></a>

&nbsp;

## 暂时性错误和重试逻辑


Azure 系统能够在 SQL 数据库服务中出现大量工作负载时动态地重新配置服务器。

但是，重新配置可能会导致客户端程序失去其与 SQL 数据库的连接。此错误称为*暂时性故障*。

在重试之间大约等待 6 到 60 秒之后，客户端程序可以尝试重新建立连接。你必须在客户端中提供重试逻辑。

有关演示重试逻辑的代码示例，请参阅：[SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)


### 暂时性故障的错误编号


发生任何 SQL 数据库错误时，将引发 [SqlException](http://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlexception.aspx)。**SqlException** 在其 **Number** 属性中包括一个数字错误代码。如果错误代码指出是暂时性错误，你的程序应当重试调用。


- [SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/#bkmk_connection_errors)
 - “暂时性错误”、“连接断开错误”部分是证明需要自动重试的暂时性错误的列表。
 - 例如，如果出现错误编号 40613，则应当重试，这表示类似于<br/>服务器“theserver”上的数据库“mydatabase”当前不可用。


有关详细信息，请参阅：
- [Azure SQL 数据库开发：操作指南主题](http://msdn.microsoft.com/zh-cn/library/azure/ee621787.aspx) 
- [Azure SQL 数据库连接问题故障排除](http://support.microsoft.com/zh-cn/kb/2980233/)


## 技术


以下主题包含多个语言和驱动程序技术的代码示例链接，可让你从客户端程序连接到 Azure SQL 数据库。


针对在 Windows、Linux 和 Mac OS X 上运行的客户端提供各种代码示例。


**一般示例：**有各种不同程序设计语言的[代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)，包括 PHP、Python、Node.js 和 .NET CSharp。此外，还提供在 Windows、Linux 和 Mac OS X 上运行的客户端示例。


**弹性缩放：**有关连接到弹性缩放数据库的信息，请参阅：

- [Azure SQL 数据库 Elastic Scale 预览版入门](/documentation/articles/sql-database-elastic-scale-get-started)
- [依赖于数据的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing)


**驱动程序库：**有关连接驱动程序库的信息（包括建议的版本），请参阅：

- [用于 SQL 数据库和 SQL Server 的连接库](/documentation/articles/sql-database-libraries)

<!---HONumber=74-->