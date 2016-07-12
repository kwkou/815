<properties
	pageTitle="修复 SQL 连接错误和暂时性错误 | Azure"
	description="了解如何排查、诊断和防止 Azure SQL 数据库中的 SQL 连接错误或暂时性错误。"
	keywords="sql 连接, 连接字符串, 连接问题, 暂时性错误, 连接错误"
	services="sql-database"
	documentationCenter=""
	authors="dalechen"
	manager="felixwu"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="03/30/2016"
	wacn.date="05/16/2016"/>


# 排查、诊断和防止 SQL 数据库中的 SQL 连接错误和暂时性错误

本文介绍如何防止、排查、诊断和减少客户端应用程序在与 Azure SQL 数据库交互时发生的连接错误和暂时性错误。了解如何配置重试逻辑、生成连接字符串以及调整其他连接设置。

<a id="i-transient-faults" name="i-transient-faults"></a>

## 暂时性错误（暂时性故障）

暂时性错误（也称为暂时性故障）存在很快解决自身问题的根本原因。当 Azure 系统快速地将硬件资源转移到负载平衡更好的各种工作负荷时，偶尔会发生暂时性错误。大多数这些重新配置事件通常在 60 秒内就能完成。在进行这种重新配置的过程中，可能会遇到与 Azure SQL 数据库的连接性问题。连接到 Azure SQL 数据库的应用程序应当构建为能预见这些暂时性错误，并能通过在它们的代码内实现重试逻辑来处理它们，而不是以应用程序错误的形式呈现给用户。

如果客户端程序使用 ADO.NET，系统将会引发 **SqlException**，使你的程序知道已发生暂时性错误。你可以将 **Number** 属性与 [SQL 数据库客户端应用程序的 SQL 错误代码](/documentation/articles/sql-database-develop-error-messages/)主题顶部附近的暂时性错误列表进行比较。

<a id="connection-versus-command" name="connection-versus-command"></a>

### 连接与命令

你将重试 SQL 连接或重新建立连接，具体取决于以下各项：

* **在尝试连接期间发生暂时性错误**：应该延迟数秒后再重试连接。

* **执行某个 SQL 查询命令期间发生暂时性错误**：不应该立即重试该命令。而应在一定的延迟之后建立新的连接。然后可以重试该命令。


<a id="j-retry-logic-transient-faults" name="j-retry-logic-transient-faults"></a>

### 针对暂时性错误的重试逻辑


在偶尔会遇到暂时性错误的客户端程序中包含重试逻辑可以让它变得更稳健。


如果你的程序通过第三方中间件与 Azure SQL 数据库通信，请咨询供应商该中间件是否包含暂时性错误的重试逻辑。

<a id="principles-for-retry" name="principles-for-retry"></a>

#### 重试原则


- 如果错误是暂时性的，则应重新尝试打开连接。


- 不应直接重试由于暂时性错误而失败的 SQL SELECT 语句。
 - 而是建立新的连接，然后重试 SELECT。


- SQL UPDATE 语句由于暂时性错误而失败时，应该先建立新的连接，再重试 UPDATE。
 - 重试逻辑必须确保整个数据库事务完成，或整个事务已回滚。


#### 其他重试注意事项


- 下班后自动启动的批处理程序以及在凌晨之前完成的批处理程序在每次重试前经过较长的时间间隔。


- 用户界面程序应该解释用户将在长时间等待后放弃操作的倾向。
 - 但是，解决方案不得每隔几秒钟重试，因为该策略可能会使系统填满请求。


#### 增大重试间隔



我们建议在第一次重试前延迟 5 秒钟。如果在少于 5 秒的延迟后重试，云服务有超载的风险。对于后续的每次重试，延迟应以指数级增大，最大值为 60 秒。

[SQL Server 连接池 (ADO.NET)](http://msdn.microsoft.com/zh-cn/library/8xx3tyca.aspx) 中提供了有关使用 ADO.NET 的客户端的*阻塞期*的说明。

你还可能想要设置程序在自行终止之前的重试次数上限。


#### 重试逻辑代码示例


以下位置提供了采用各种编程语言的重试逻辑代码示例：

- [快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples/)


<a id="k-test-retry-logic" name="k-test-retry-logic"></a>

#### 测试重试逻辑


若要测试重试逻辑，必须模拟或生成程序仍在运行时可更正的错误。


##### 通过断开网络连接进行测试


可以测试重试逻辑的方法，就是在程序运行时断中客户端计算机与网络的连接。将产生错误：

- **SqlException.Number** = 11001
- 消息：“不存在已知的这种主机”


首次重试时，程序可以更正拼写错误，然后尝试连接。


若要使此操作可行，请从网络中断开计算机的连接，再启动你的程序。然后，你的程序将识别促使它执行以下操作的运行时参数：

1. 暂时将 11001 添加到视为暂时性故障的错误列表。
2. 像往常一样尝试首次连接。
3. 在捕获该错误后，从列表中删除 11001。
4. 显示一条消息，告知用户要将计算机接入网络。
 - 通过使用“Console.ReadLine”方法或具有“确定”按钮的对话框来暂停进一步的执行。将计算机接入网络后，用户按 Enter 键。
5. 重新尝试连接，预期将会成功。


##### 通过在连接时拼错数据库名称进行测试


在首次连接尝试之前，程序可以故意拼错用户名。将产生错误：

- **SqlException.Number** = 18456
- 消息：“用户 'WRONG\_MyUserName' 的登录失败。”


首次重试时，程序可以更正拼写错误，然后尝试连接。


要在实践中进行此测试，你的程序需要能够识别促使它执行以下操作的运行时参数：

1. 暂时将 18456 添加到视为暂时性故障的错误列表。
2. 故意将“WRONG\_”添加到用户名。
3. 在捕获该错误后，从列表中删除 18456。
4. 从用户名中删除“WRONG\_”。
5. 重新尝试连接，预期将会成功。

<a id="net-sqlconnection-parameters-for-connection-retry" name="net-sqlconnection-parameters-for-connection-retry"></a>

### 连接重试的 .NET SqlConnection 参数


如果客户端程序使用 .NET Framework 类“System.Data.SqlClient.SqlConnection”连接 Azure SQL 数据库，应使用 .NET 4.6.1 或更高版本，以便可以利用其连接重试功能。该功能的详细信息在[此处](http://go.microsoft.com/fwlink/?linkid=393996)。


<!--
2015-11-30, FwLink 393996 points to dn632678.aspx, which links to a downloadable .docx related to SqlClient and SQL Server 2014.
-->


为 **SqlConnection** 对象生成[连接字符串](http://msdn.microsoft.com/zh-cn/library/System.Data.SqlClient.SqlConnection.connectionstring.aspx)时，应在以下参数之间协调值：

- ConnectRetryCount &nbsp;&nbsp;（默认值为 1。范围是从 0 到 255。）
- ConnectRetryInterval &nbsp;&nbsp;（默认值为 1 秒。范围是从 1 到 60。）
- 连接超时值 &nbsp;&nbsp;（默认值为 15 秒。范围是从 0 到 2147483647）


具体而言，所选的值应使以下等式成立：

- 连接超时值 = ConnectRetryCount * ConnectionRetryInterval

例如，如果计数 = 3 且间隔 = 10 秒，超时值仅为 29 秒未给系统足够的时间进行其第三次也是最后一次连接重试，因为：29 < 3 * 10。

<a id="connection-versus-command" name="connection-versus-command"></a>

### 连接与命令


**ConnectRetryCount** 和 **ConnectRetryInterval** 参数使 **SqlConnection** 对象在重试连接操作时不用通知或麻烦你的程序（例如，将控制权返还给你的程序）。在以下情况下可能会进行重试：

- mySqlConnection.Open 方法调用
- mySqlConnection.Execute 方法调用

有个很微妙的地方。如果你正在执行“查询”时发生暂时性错误，“SqlConnection”对象不会重试连接操作，因而肯定不会重试你的查询。但是，**SqlConnection** 在发送要执行的查询前会非常快速地检查连接。如果快速检查检测到连接问题，**SqlConnection** 会重试连接操作。如果重试成功，则会发送查询以执行。


#### ConnectRetryCount 是否应结合应用程序重试逻辑？

假设你的应用程序具有功能强大的自定义重试逻辑。它可能会重试连接操作 4 次。如果你将 **ConnectRetryInterval** 和 **ConnectRetryCount** = 3 添加到连接字符串，则将重试计数提高到 4 * 3 = 12 次重试。你可能未打算重试这么多次。

<a id="a-connection-connection-string" name="a-connection-connection-string"></a>

## 与 Azure SQL 数据库的连接

<a id="c-connection-string" name="c-connection-string"></a>

### 连接：连接字符串


连接到 Azure SQL 数据库所需的连接字符串与连接到 Microsoft SQL Server 所需的字符串稍有不同。可以从 [Azure 管理门户](https://manage.windowsazure.cn)复制数据库的连接字符串。


[AZURE.INCLUDE [sql-database-include-connection-string-20-portalshots](../includes/sql-database-include-connection-string-20-portalshots.md)]


<a id="b-connection-ip-address" name="b-connection-ip-address"></a>

### 连接：IP 地址


必须将 SQL 数据库服务器配置为接受来自托管客户端程序的计算机的通信。为此，可以通过 [Azure 管理门户](https://manage.windowsazure.cn)编辑防火墙设置。


如果你忘记了配置 IP 地址，你的程序将失败，并显示简单的错误消息，指出所需的 IP 地址。


<a id="c-connection-ports" name="c-connection-ports"></a>

### 连接：端口


通常，只需确保在托管客户端程序的计算机上已打开端口 1433 进行出站通信。


例如，当客户端程序托管在 Windows 计算机上时，主机上的 Windows 防火墙允许你打开端口 1433：


1. 打开“控制面板”
2. &gt;“所有控制面板项”
3. &gt;“Windows 防火墙”
4. &gt;“高级设置”
5. &gt;“出站规则”
6. &gt;“操作”
7. &gt;“新建规则”


如果你的客户端程序托管在 Azure 虚拟机 (VM) 上，你应该阅读：<br/>[用于 ADO.NET 4.5 和 SQL 数据库 V12 的非 1433 端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)。


有关配置端口和 IP 地址的背景信息，请参阅：
[Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)


<a id="d-connection-ado-net-4-5" name="d-connection-ado-net-4-5"></a>

### 连接：ADO.NET 4.6.1


如果你的程序使用 **System.Data.SqlClient.SqlConnection** 等 ADO.NET 类来连接到 Azure SQL 数据库，我们建议使用 .NET Framework 4.6.1 或更高版本。


ADO.NET 4.6.1：

- 对于 Azure SQL 数据库，使用 **SqlConnection.Open** 方法打开连接可以获得更高的可靠性。此 **Open** 方法现在结合了应对暂时性故障的最佳效果重试机制，用于处理连接超时期间发生的特定错误。
- 支持连接池。这包括有效验证提供给程序的连接对象是否正常运行的功能。



如果你要从连接池使用连接对象，我们建议，如果程序未立即使用连接，应暂时关闭连接。重新打开连接比创建新连接的开销更低。


如果使用的是 ADO.NET 4.0 或更旧版本，我们建议升级到最新的 ADO.NET。

- 从 2015 年 11 月开始，你可以[下载 ADO.NET 4.6.1](http://blogs.msdn.com/b/dotnet/archive/2015/11/30/net-framework-4-6-1-is-now-available.aspx)。


<a id="e-diagnostics-test-utilities-connect" name="e-diagnostics-test-utilities-connect"></a>

## 诊断

<a id="d-test-whether-utilities-can-connect" name="d-test-whether-utilities-can-connect"></a>

### 诊断：测试实用程序是否可以连接


如果你的程序无法连接到 Azure SQL 数据库，有一个诊断选项可让你尝试使用一个实用程序来进行连接。理想的情况下，该实用程序将使用你的程序所用的同一个库进行连接。


你可以在任何 Windows 计算机上尝试以下实用程序：

- SQL Server Management Studio (ssms.exe)，使用 ADO.NET 进行连接。
- sqlcmd.exe，使用 [ODBC](http://msdn.microsoft.com/zh-cn/library/jj730308.aspx) 进行连接。


完成连接后，测试一个简短的 SQL SELECT 查询是否可以正常工作。


<a id="f-diagnostics-check-open-ports" name="f-diagnostics-check-open-ports"></a>

### 诊断：检查打开的端口


假设你怀疑连接尝试由于端口问题而失败。在你的计算机上，可以运行报告端口配置的实用程序。


在 Linux 上以下实用程序可能会有所帮助：

- `netstat -nap`
- `nmap -sS -O 127.0.0.1`
 - （将示例值更改为你的 IP 地址。）


在 Windows 上，[PortQry.exe](http://www.microsoft.com/en-us/download/details.aspx?id=17148) 实用程序可能很有用。以下是在 Azure SQL 数据库服务器上查询端口情况，以及在便携式计算机上运行的的示例执行：
 

	[C:\Users\johndoe]
	>> portqry.exe -n johndoesvr9.database.chinacloudapi.cn -p tcp -e 1433
	
	Querying target system called:
	 johndoesvr9.database.chinacloudapi.cn
	
	Attempting to resolve name to IP address...
	Name resolved to 23.100.117.95
	
	querying...
	TCP port 1433 (ms-sql-s service): LISTENING
	
	[C:\Users\johndoe]
	>>



<a id="g-diagnostics-log-your-errors" name="g-diagnostics-log-your-errors"></a>

### 诊断：记录你的错误


有时，诊断间歇性问题的最好方式是每隔数天或数周检测常规模式。


你的客户端可以通过记录其所遇到的所有错误来帮助你进行诊断。你可以将日志条目与 Azure SQL 数据库本身内部记录的错误数据相关联。


Enterprise Library 6 (EntLib60) 提供了 .NET 托管类来帮助进行日志记录：

- [5 - 如同从圆木上掉下来一样简单：使用日志记录应用程序块](http://msdn.microsoft.com/zh-cn/library/dn440731.aspx)


<a id="h-diagnostics-examine-logs-errors" name="h-diagnostics-examine-logs-errors"></a>

### 诊断：在系统日志中检查错误


下面是在日志中查询错误和其他信息的一些 Transact-SQL SELECT 语句。


| 日志查询 | 说明 |
| :-- | :-- |
| `SELECT e.*`<br/>`FROM sys.event_log AS e`<br/>`WHERE e.database_name = 'myDbName'`<br/>`AND e.event_category = 'connectivity'`<br/>`AND 2 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, e.end_time, GetUtcDate())`<br/>`ORDER BY e.event_category,`<br/>&nbsp;&nbsp;`e.event_type, e.end_time;` | [sys.event\_log](http://msdn.microsoft.com/zh-cn/library/dn270018.aspx) 视图提供了单个事件的相关信息，包括可能导致暂时性错误或连接失败的一些事件。<br/><br/>理想的情况下，可以将 **start\_time** 或 **end\_time** 值与有关客户端程序何时遇到问题的信息相关联。<br/><br/>**提示：**必须连接到 **master** 数据库才能运行此操作。 |
| `SELECT c.*`<br/>`FROM sys.database_connection_stats AS c`<br/>`WHERE c.database_name = 'myDbName'`<br/>`AND 24 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, c.end_time, GetUtcDate())`<br/>`ORDER BY c.end_time;` | [sys.database\_connection\_stats](http://msdn.microsoft.com/zh-cn/library/dn269986.aspx) 视图针对其他诊断提供事件类型的聚合计数。<br/><br/>**提示：**必须连接到 **master** 数据库才能运行此操作。 |

<a id="d-search-for-problem-events-in-the-sql-database-log" name="d-search-for-problem-events-in-the-sql-database-log"></a>

### 诊断：在 SQL 数据库日志中搜索问题事件


可以在 Azure SQL 数据库的日志中搜索有关问题事件的条目。在 **master** 数据库中尝试运行以下 Transact-SQL SELECT 语句：


```
SELECT
   object_name
  ,CAST(f.event_data as XML).value
      ('(/event/@timestamp)[1]', 'datetime2')                      AS [timestamp]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="error"]/value)[1]', 'int')             AS [error]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="state"]/value)[1]', 'int')             AS [state]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="is_success"]/value)[1]', 'bit')        AS [is_success]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="database_name"]/value)[1]', 'sysname') AS [database_name]
FROM
  sys.fn_xe_telemetry_blob_target_read_file('el', null, null, null) AS f
WHERE
  object_name != 'login_event'  -- Login events are numerous.
  and
  '2015-06-21' < CAST(f.event_data as XML).value
        ('(/event/@timestamp)[1]', 'datetime2')
ORDER BY
  [timestamp] DESC
;
```


#### 将返回 sys.fn\_xe\_telemetry\_blob\_target\_read\_file 中的若干行


下面是返回行的类似内容。显示的 null 值在其他行中通常不是 null。


```
object_name                   timestamp                    error  state  is_success  database_name

database_xml_deadlock_report  2015-10-16 20:28:01.0090000  NULL   NULL   NULL        AdventureWorks
```


<a id="l-enterprise-library-6" name="l-enterprise-library-6"></a>

## Enterprise Library 6


Enterprise Library 6 (EntLib60) 是 .NET 类的框架，可帮助你实施云服务（包括 Azure SQL 数据库服务）的稳健客户端。要找到 EntLib60 可以提供帮助的各个领域的相关专题，可以首先访问：

- [Enterprise Library 6 - 2013 年 4 月](http://msdn.microsoft.com/zh-cn/library/dn169621%28v=pandp.60%29.aspx)


用于处理暂时性错误的重试逻辑是 EntLib60 可以提供帮助的一个领域：

- [4 - 坚持不懈是一切成功的秘密：使用暂时性故障处理应用程序块](http://msdn.microsoft.com/zh-cn/library/dn440719%28v=pandp.60%29.aspx)


在其重试逻辑中使用 EntLib60 的简短 C# 代码示例可从以下链接中找到：

- [代码示例：Enterprise Library 6 中的用 C# 编写的用于连接 SQL 数据库的重试逻辑](/documentation/articles/sql-database-develop-entlib-csharp-retry-windows/)


> [AZURE.NOTE] EntLib60 的源代码可公开[下载](http://go.microsoft.com/fwlink/p/?LinkID=290898)。Microsoft 不打算对 EntLib 做进一步的功能或维护更新。

<a id="entlib60-classes-for-transient-errors-and-retry" name="entlib60-classes-for-transient-errors-and-retry"></a>

### 用于暂时性错误和重试的 EntLib60 类


以下 EntLib60 类对重试逻辑特别有用。所有这些类都包含在 **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling** 命名空间或其子级中：

在命名空间 **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling** 中：

- **RetryPolicy** 类
 - **ExecuteAction** 方法


- **ExponentialBackoff** 类


- **SqlDatabaseTransientErrorDetectionStrategy** 类


- **ReliableSqlConnection** 类
 - **ExecuteCommand** 方法


在命名空间 **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling.TestSupport** 中：

- **AlwaysTransientErrorDetectionStrategy** 类

- **NeverTransientErrorDetectionStrategy** 类


以下是 EntLib60 相关信息的链接：

- 免费[书籍下载：Microsoft Enterprise Library 版本 2 开发人员指南](http://www.microsoft.com/en-us/download/details.aspx?id=41145)

- 最佳实践：[有关重试的一般性指南](/documentation/articles/best-practices-retry-general/)深入探讨了重试逻辑。

- [Enterprise Library - 暂时性故障处理应用程序块 6.0](http://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling) 的 NuGet 下载

<a id="entlib60-the-logging-block" name="entlib60-the-logging-block"></a>

### EntLib60：日志记录块


- 日志记录块是极其灵活且可配置的解决方案，可让你：
 - 创建日志消息并将其存储在各种不同的位置。
 - 分类与筛选消息。
 - 收集有助于调试和跟踪的上下文信息，以及用于满足审核和一般日志记录要求的上下文信息。


- 日志记录块可以从日志目标抽象化日志记录功能，使应用程序代码保持一致，无论目标日志记录存储的位置和类型为何。


有关详细信息，请参阅：
[5 - 像滚圆木一样容易：使用日志记录应用程序块](https://msdn.microsoft.com/zh-cn/library/dn440731%28v=pandp.60%29.aspx)

<a id="entlib60-istransient-method-source-code" name="entlib60-istransient-method-source-code"></a>

### EntLib60 IsTransient 方法的源代码


接下来，**SqlDatabaseTransientErrorDetectionStrategy** 类包含 **IsTransient** 方法的 C# 源代码。该源代码阐明了哪些错误被视为暂时性错误并值得重试（从 2013 年 4 月起）。

为了注重易读性，我们在此副本中删除了大量的 **//comment** 行。

	
	public bool IsTransient(Exception ex)
	{
	  if (ex != null)
	  {
	    SqlException sqlException;
	    if ((sqlException = ex as SqlException) != null)
	    {
	      // Enumerate through all errors found in the exception.
	      foreach (SqlError err in sqlException.Errors)
	      {
	        switch (err.Number)
	        {
	            // SQL Error Code: 40501
	            // The service is currently busy. Retry the request after 10 seconds.
	            // Code: (reason code to be decoded).
	          case ThrottlingCondition.ThrottlingErrorNumber:
	            // Decode the reason code from the error message to
	            // determine the grounds for throttling.
	            var condition = ThrottlingCondition.FromError(err);
	
	            // Attach the decoded values as additional attributes to
	            // the original SQL exception.
	            sqlException.Data[condition.ThrottlingMode.GetType().Name] =
	              condition.ThrottlingMode.ToString();
	            sqlException.Data[condition.GetType().Name] = condition;
	
	            return true;
	
	          case 10928:
	          case 10929:
	          case 10053:
	          case 10054:
	          case 10060:
	          case 40197:
	          case 40540:
	          case 40613:
	          case 40143:
	          case 233:
	          case 64:
	            // DBNETLIB Error Code: 20
	            // The instance of SQL Server you attempted to connect to
	            // does not support encryption.
	          case (int)ProcessNetLibErrorCode.EncryptionNotSupported:
	            return true;
	        }
	      }
	    }
	    else if (ex is TimeoutException)
	    {
	      return true;
	    }
	    else
	    {
	      EntityException entityException;
	      if ((entityException = ex as EntityException) != null)
	      {
	        return this.IsTransient(entityException.InnerException);
	      }
	    }
	  }
	
	  return false;
	}



## 详细信息


## 后续步骤

- 有关其他常见的 Azure SQL 数据库连接问题的疑难解答，请访问 [Azure SQL 数据库的常见连接问题疑难解答](/documentation/articles/sql-database-troubleshoot-common-connection-issues/)。

- [SQL Server 连接池 (ADO.NET)](http://msdn.microsoft.com/zh-cn/library/8xx3tyca.aspx)


- [重试是 Apache 2.0 授权的通用重试库，它以 **Python** 编写，可以简化向几乎任何程序添加重试行为的任务。](https://pypi.python.org/pypi/retrying)

<!---HONumber=Mooncake_0509_2016-->
