 <properties
	pageTitle="如何使用批处理来改善 Azure SQL 数据库应用程序的性能"
	description="本主题提供有关数据库批处理操作大幅改善 Azure SQL 数据库应用程序速度和缩放性的证据。尽管这些批处理方法适用于任何 SQL Server 数据库，但本文将重点放在 Azure 上。"
	services="sql-database"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" />


<tags
	ms.service="sql-database"
	ms.date="02/04/2016"
	wacn.date="03/29/2016" />

# 如何使用批处理来改善 SQL 数据库应用程序的性能

对 Azure SQL 数据库执行批处理操作可以大幅改善应用程序的性能和缩放性。为了帮助你了解优点，本文的第一部分包含一些示例测试结果用于比较对 SQL 数据库发出的顺序请求和分批请求。本文的余下部分介绍了帮助你在 Azure 应用程序中成功使用批处理的方法、方案和注意事项。

**作者**：Jason Roth、Silvano Coriani、Trent Swanson (Full Scale 180 Inc)

**审校**：Conor Cunningham、Michael Thomassy

## 为什么批处理对 SQL 数据库很重要？
对远程服务的批处理调用是提高性能和可伸缩性的常用策略。对于任何与远程服务的交互（如序列化、网络传输和反序列化），都有固定的处理开销。将很多单独的事务打包为一个批处理操作可最大限度降低这些成本。

在本文中，我们要比较各种 SQL 数据库批处理策略和情形。尽管这些策略对于使用 SQL Server 的本地应用程序也很重要，但是将批处理用于 SQL 数据库主要是基于以下两个原因：

- 在访问 SQL 数据库时可能有更长的网络延迟，特别是当你从同一 SQL 数据库数据中心外部访问 Azure 时。
- SQL 数据库的多租户特征意味着数据访问层的效率与数据库的总体缩放性关联。SQL 数据库必须防止任何单个租户/用户独占数据库资源，从而对其他租户不利。在使用量超过预定义的配额时，SQL 数据库可减小吞吐量或引发限制异常。一些提高效率的措施（如批处理），允许你在达到这些配额前在 SQL 数据库上做更多的工作。 
- 批处理对于使用多个数据库或联合的体系结构也很有效（分片）。与每个数据库单位的交互效率仍是影响总体伸缩性的关键因素。 

使用 SQL 数据库的一个好处是你不必管理托管数据库的服务器。但是，这个托管的基础结构也意味着你必须重新考虑数据库优化。你将不再致力于改进数据库硬件或网络基础结构。Azure 将控制这些环境。你可以控制的主要方面是你的应用程序如何与 SQL 数据库交互。批处理就是这些优化措施之一。

本文的第一部分比较了使用 SQL 数据库的 .NET 应用程序可用的各种批处理方法。最后两个部分介绍批处理准则和方案。

## 批处理策略

### 有关本主题中计时结果的注意事项
>[AZURE.NOTE] 结果并不是基准，而是用于显示**相对性能**。计时基于至少运行 10 次测试后的平均值。操作将插入空表。这些测试将在 V12 以前的版本中测量，不一定对应于在使用新[服务层](/documentation/articles/sql-database-service-tiers/)的 V12 数据库中可能会获得的吞吐量。批处理技术的相对优势应该类似。

### 事务
通过讨论事务来开始讲述批处理似乎有点奇怪。但是使用客户端事务具有提高性能的微妙服务器端批处理效果。可以使用几行代码来添加事务，因此这提供了一个快速提高顺序操作的性能的方法。

请注意以下 C# 代码，其中包含对一个简单表执行的插入和更新操作序列。

	List<string> dbOperations = new List<string>();
	dbOperations.Add("update MyTable set mytext = 'updated text' where id = 1");
	dbOperations.Add("update MyTable set mytext = 'updated text' where id = 2");
	dbOperations.Add("update MyTable set mytext = 'updated text' where id = 3");
	dbOperations.Add("insert MyTable values ('new value',1)");
	dbOperations.Add("insert MyTable values ('new value',2)");
	dbOperations.Add("insert MyTable values ('new value',3)");

以下 ADO.NET 代码可按顺序执行这些操作。

	using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	{
	    conn.Open();
	
	    foreach(string commandString in dbOperations)
	    {
	        SqlCommand cmd = new SqlCommand(commandString, conn);
	        cmd.ExecuteNonQuery();                   
	    }
	}

优化此代码的最佳方法是实现这些调用的某种客户端批处理。但是有一个提高此代码性能的简单方法，即在事务中包装此调用序列。以下是使用事务的同一代码。

	using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	{
	    conn.Open();
	    SqlTransaction transaction = conn.BeginTransaction();
	
	    foreach (string commandString in dbOperations)
	    {
	        SqlCommand cmd = new SqlCommand(commandString, conn, transaction);
	        cmd.ExecuteNonQuery();
	    }
	
	    transaction.Commit();
	}

事务实际在这两个示例中都用到了。在第一个示例中，每个单个调用就是一个隐式事务。在第二个示例中，用一个显式事务包装了所有调用。按照[预写事务日志](https://msdn.microsoft.com/zh-cn/library/ms186259.aspx)的文档中所述，在事务提交时将日志记录刷新到磁盘。因此通过在事务中包含更多调用，写入事务日志可能延迟，直到提交事务。实际上，你正在为写入服务器的事务日志启用批处理。

下表显示一些即席测试结果。这些测试执行具有事务和不具有事务的相同的顺序插入。为了更具对比性，第一组测试是从笔记本电脑针对 Azure 中的数据库远程运行的。第二组测试是从位于同一 Azure 数据中心（中国北部）内的云服务和数据库运行的。下表显示具有和不具有事务的一系列顺序插入所用的时间（毫秒）。

**本地到 Azure**：

| 操作 | 无事务（毫秒） | 事务（毫秒） |
|---|---|---|
| 1 | 130 | 402 |
| 10 | 1208 | 1226 |
| 100 | 12662 | 10395 |
| 1000 | 128852 | 102917 |


**Azure 到 Azure（同一数据中心）**：

| 操作 | 无事务（毫秒） | 事务（毫秒） |
|---|---|---|
| 1 | 21 | 26 |
| 10 | 220 | 56 |
| 100 | 2145 | 341 |
| 1000 | 21479 | 2756 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

根据前面的测试结果，在事务中包装一个操作实际上会降低性能。但是，当你增加单个事务中的操作数时，性能提高将变得很明显。当所有操作发生在 Azure 数据中心内时，性能差异也更明显。从 SQL 数据库数据中心外部使用 Azure 增加的延迟时间将超过使用事务带来的性能提高。

尽管使用事务可以提高性能，请继续[遵循事务和连接的最佳做法](https://msdn.microsoft.com/zh-cn/library/ms187484.aspx)。使事务尽可能短，并在工作完成后关闭数据库连接。前一个示例中的 using 语句可确保在后续代码阻塞完成时关闭连接。

前一个示例演示你可以将一个本地事务添加到任何具有两行的 ADO.NET 代码。事务提供了一个快速提高代码性能的方法，这些代码用于执行顺序插入、更新和删除操作。但是，为了实现最佳性能，请考虑进一步更改代码，以利用客户端批处理（如表值参数）。

有关 ADO.NET 中事务的详细信息，请参阅 [ADO.NET 中的本地事务](https://msdn.microsoft.com/zh-cn/library/vstudio/2k2hy99x.aspx)。

### 表值参数
表值参数支持用户定义的表类型作为 Transact-SQL 语句、存储过程和函数的参数。使用这个客户端批处理方法，你可以在表值参数中发送多行数据。若要使用表值参数，请首先定义表类型。以下 Transact-SQL 语句将创建一个名为 **MyTableType** 的表类型。

	CREATE TYPE MyTableType AS TABLE 
	( mytext TEXT,
	  num INT );
 

在代码中，你将创建一个与表类型具有相同名称和类型的 **DataTable**。在文本查询或存储过程调用的参数中传递此 **DataTable**。以下示例显示了这个方法：

	using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	{
	    connection.Open();
	
	    DataTable table = new DataTable();
	    // Add columns and rows. The following is a simple example.
	    table.Columns.Add("mytext", typeof(string));
	    table.Columns.Add("num", typeof(int));    
	    for (var i = 0; i < 10; i++)
	    {
	        table.Rows.Add(DateTime.Now.ToString(), DateTime.Now.Millisecond);
	    }
	
	    SqlCommand cmd = new SqlCommand(
	        "INSERT INTO MyTable(mytext, num) SELECT mytext, num FROM @TestTvp",
	        connection);
	                
	    cmd.Parameters.Add(
	        new SqlParameter()
	        {
	            ParameterName = "@TestTvp",
	            SqlDbType = SqlDbType.Structured,
	            TypeName = "MyTableType",
	            Value = table,
	        });
	
	    cmd.ExecuteNonQuery();
	}

在前一示例中，**SqlCommand** 对象从表值参数 **@TestTvp** 插入行。使用 **DataTable** 方法将以前创建的 **SqlCommand.Parameters.Add** 对象分配到此参数。对一个调用中的插入进行批处理将显著提高顺序插入的性能。

若要进一步改进前一个示例，请使用存储过程来替代基于文本的命令。以下 Transact-SQL 命令创建一个采用 **SimpleTestTableType** 表值参数的存储过程。

	CREATE PROCEDURE [dbo].[sp_InsertRows] 
	@TestTvp as MyTableType READONLY
	AS
	BEGIN
	INSERT INTO MyTable(mytext, num) 
	SELECT mytext, num FROM @TestTvp
	END
	GO

然后将前一个代码示例中的 **SqlCommand** 对象声明更改为以下内容：

	SqlCommand cmd = new SqlCommand("sp_InsertRows", connection);
	cmd.CommandType = CommandType.StoredProcedure;

在大多数情况下，表值参数具有与其他批处理方法等效或更高的性能。人们通常使用表值参数，因为它们比其他选项更灵活。例如，其他方法（如 SQL 大容量复制）仅允许插入新行。但是使用表值参数，你可以在存储过程中使用逻辑来决定更新哪些行和插入哪些行。还可以修改表类型来包含“操作”列，该列指示指定的行应插入、更新还是删除。

下表显示使用表值参数的即席测试结果（毫秒）。

| 操作 | 本地到 Azure（毫秒） | 同一 Azure 数据中心（毫秒） |
|---|---|---|
| 1 | 124 | 32 |
| 10 | 131 | 25 |
| 100 | 338 | 51 |
| 1000 | 2615 | 382 |
| 10000 | 23830 | 3586 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

批处理带来的性能提升非常明显。在前面的顺序测试中，从数据中心外部执行 1000 个操作花了 129 秒，而从数据中心内部执行该操作花了 21 秒。使用表值参数后，从数据中心外部执行 1000 个操作只花了 2.6 秒，在数据中心内部执行该操作只花了 0.4 秒。

有关表值参数的详细信息，请参阅[表值参数](https://msdn.microsoft.com/zh-cn/library/bb510489.aspx)。

### SQL 批量复制
SQL 批量复制是将大量数据插入目标数据库的另一方法。.NET 应用程序可使用 **SqlBulkCopy** 类来执行批量插入操作。**SqlBulkCopy** 的功能类似于命令行工具 **Bcp.exe** 或 Transact-SQL 语句 **BULK INSERT**。以下代码示例显示如何将源 **DataTable** 中的行 table 批量复制到 SQL Server 中的目标表 MyTable。

	using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	{
	    connection.Open();
	
	    using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
	    {
	        bulkCopy.DestinationTableName = "MyTable";
	        bulkCopy.ColumnMappings.Add("mytext", "mytext");
	        bulkCopy.ColumnMappings.Add("num", "num");
	        bulkCopy.WriteToServer(table);
	    }
	}

在某些情况下，批量复制的效果好于表值参数。请参阅主题[表值参数](https://msdn.microsoft.com/zh-cn/library/bb510489.aspx)中表值参数与 BULK INSERT 操作的对比表。

以下即席测试结果显示具有 **SqlBulkCopy** 的批处理性能（毫秒）。

| 操作 | 本地到 Azure（毫秒） | 同一 Azure 数据中心（毫秒） |
|---|---|---|
| 1 | 433 | 57 |
| 10 | 441 | 32 |
| 100 | 636 | 53 |
| 1000 | 2535 | 341 |
| 10000 | 21605 | 2737 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

在较小的批大小中，使用表值参数的效果好于使用 **SqlBulkCopy** 类的效果。但是，对于涉及 1,000 和 10,000 行的测试，使用 **SqlBulkCopy** 时比使用表值参数时快 12-31%。与表值参数一样，**SqlBulkCopy** 是执行批处理插入的一个可选方法，特别是在与非批处理操作的性能作对比时。

有关 ADO.NET 中的批量复制的详细信息，请参阅 [SQL Server 中的批量复制操作](https://msdn.microsoft.com/zh-cn/library/7ek5da1a.aspx)。

### 多行参数化 INSERT 语句
对于小的批处理，一个替代方法是构造插入多个行的大型参数化 INSERT 语句。以下代码示例演示了这个方法。

	using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	{
	    connection.Open();
	
	    string insertCommand = "INSERT INTO [MyTable] ( mytext, num ) " +
	        "VALUES (@p1, @p2), (@p3, @p4), (@p5, @p6), (@p7, @p8), (@p9, @p10)";
	
	    SqlCommand cmd = new SqlCommand(insertCommand, connection);
	
	    for (int i = 1; i <= 10; i += 2)
	    {
	        cmd.Parameters.Add(new SqlParameter("@p" + i.ToString(), "test"));
	        cmd.Parameters.Add(new SqlParameter("@p" + (i+1).ToString(), i));
	    }
	
	    cmd.ExecuteNonQuery();
	}
 

此示例用于演示基本概念。一个更现实的方案是对所需的实体执行循环，以同时构造查询字符串和命令参数。最多可使用 2100 个查询参数，因此这限制了可以此方式处理的总行数。

以下即席测试结果显示此类插入语句的性能（毫秒）。

| 操作 | 表值参数（毫秒） | 单语句 INSERT（毫秒） |
|---|---|---|
| 1 | 32 | 20 |
| 10 | 30 | 25 |
| 100 | 33 | 51 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

此方法对于小于 100 行的批处理稍微提高了一下速度。尽管提高不大，但是此方法仍是一个可选方案，它可能在特定的应用程序方案下工作得很好。

### DataAdapter
**DataAdapter** 类允许你修改 **DataSet** 对象，然后将更改作为 INSERT、UPDATE 和 DELETE 操作提交。如果你正在这样使用 **DataAdapter**，请注意必须为每个不同的操作发出单独的调用。为了提高性能，请将 **UpdateBatchSize** 属性值设置为应同时批处理的操作数。有关详细信息，请参阅[使用 DataAdapter 执行批处理操作](https://msdn.microsoft.com/zh-cn/library/aadf8fk2.aspx)。

### 实体框架
实体框架当前不支持批处理。社区中的各个开发人员已尝试寻找解决方法，例如重写 **SaveChanges** 方法。但是这些解决方法通常很复杂，而且仅适用于特定应用程序和数据模型。实体框架 codeplex 项目当前已有关于此功能请求的讨论页。要查看此讨论，请参阅[设计会议备忘录 - 2012 年 8 月 2 日](http://entityframework.codeplex.com/wikipage?title=Design%20Meeting%20Notes%20-%20August%202%2c%202012)。

### XML
为了完整性，我们认为有必要将 XML 作为一种批处理策略来讲述。但是，使用 XML 与其他方法相比没有什么优势，而且还有几个缺点。此方法类似于表值参数，但是 XML 文件或字符串将传递到存储过程而非用户定义的表。存储过程分析该存储过程中的命令。

此方法有几个缺点：

- 使用 XML 可能很繁琐，而且容易出错。
- 在数据库上分析 XML 可能占用大量 CPU。
- 在大多数情况下，使用此方法比使用表值参数慢。

由于上述原因，不建议将 XML 用于批处理查询。

## 批处理注意事项
以下部分提供有关在 SQL 数据库应用程序中使用批处理的更多指南。

### 权衡
根据你的体系结构，批处理可能涉及性能和弹性之间的权衡。例如，请考虑你的角色意外停止的方案。如果丢失一行数据，其影响小于丢失一大批未提交的行的影响。当你在指定的时间窗口内将行发送到数据库前缓冲它们时，丢失数据的风险更大。

因为要进行权衡，因此需要评估你批处理的操作类型。对相对不重要的数据进行更激进的批处理（即更大的批次，涉及更长的时间窗口）。

### 批大小
在我们的测试中，将大的批次拆分为更小的块通常没有好处。实际上，这种拆分通常导致比提交单个大批次还要慢的性能。例如，考虑要插入 1000 行的一个方案。下表显示在拆分为更小的批次时使用表值参数插入 1000 行所需的时间。

| 批大小 | 迭代 | 表值参数（毫秒） |
| -------- | --- | --- |
| 1000 | 1 | 347 |
| 500 | 2 | 355 |
| 100 | 10 | 465 |
| 50 | 20 | 630 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

你可以看到对于 1000 行，在一次提交它们时性能最佳。在其他测试中（未在此处显示），将 10000 行拆分为两个包含 5000 行的批可略微提高性能。但是这些测试的表架构相对简单，因此你应对自己的特定数据和批大小执行测试，以验证这些结果。

要考虑的另一个因素是如果总批大小变得太大，SQL 数据库可能限制并拒绝提交该批。为了获得最佳结果，请测试你的特定方案来确定哪个批大小更合适。使批大小在运行时是可配置的，以允许基于性能或错误进行快速调整。

最后，平衡批大小和与批处理有关的风险。如果出现暂时性错误或角色失败，请考虑重试操作或丢失批中数据的后果。

### 并行处理
如果你采用减小批大小但是使用多个线程来执行工作怎么样？ 我们的测试再次显示几个较小的多线程批次的性能通常比单个较大的批次性能差。以下测试尝试在一个或多个并行批次中插入 1000 行。此测试显示多个同时执行的批次实际上降低了性能。

| 批大小 [迭代] | 两个线程（毫秒） | 四个线程（毫秒） | 六个线程（毫秒） |
| -------- | --- | --- | --- |
| 1000 [1] | 277 | 315 | 266 |
| 500 [2] | 548 | 278 | 256 |
| 250 [4] | 405 | 329 | 265 |
| 100 [10] | 488 | 439 | 391 |

>[AZURE.NOTE] 结果并非基准。请参阅[有关本主题中计时结果的注意事项](#note-about-timing-results-in-this-topic)。

并行度导致性能下降可能有以下几个原因：

- 有多个同时执行的网络调用而非一个。
- 针对一个表的多个操作可能导致争用和阻塞。
- 存在与多线程关联的开销。
- 打开多个连接的开销超过了并行处理带来的好处。

如果你针对不同的表或数据库，可能发现使用此策略会提高一点性能。在数据库分片或联合方案下，可能使用此方法。分片使用多个数据库并将不同数据路由到每个数据库。如果每个小批次针对不同数据库，则并行执行操作可能更有效。但是，性能提升并不明显，无法由此决定在你的解决方案中使用数据库分片。

在一些设计中，并行执行较小的批次可能导致将提高吞储量的请求置于负荷不大的系统中。在这种情况下，即使处理单个更大的批次更快，并行处理多个批也可能更有效。

如果使用并行执行，请考虑控制最大工作线程数。较小的数可能导致争用减少并且执行时间缩短。此外，请注意这将增加目标数据库的连接和事务负载。

### 相关性能因素
有关数据库性能的通常准则也影响批处理。例如，对于具有大的主键或很多非聚集索引的表，插入性能会下降。

如果表值参数使用存储过程，你可以在该过程开头使用命令 **SET NOCOUNT ON**。此语句禁止返回过程中受影响的行的计数。但是，在我们的测试中，使用 **SET NOCOUNT ON** 对性能没有影响或导致性能下降。测试存储过程很简单，它只有来自表值参数的一个 **INSERT** 命令。更复杂的存储过程可能从此语句受益。但是不要认为将 **SET NOCOUNT ON** 添加到你的存储过程会自动提高性能。为了了解该影响，请用包含和不包含 **SET NOCOUNT ON** 语句来测试你的存储过程。

## 批处理方案
以下部分说明如何在三种应用程序方案下使用表值参数。第一种方案显示缓冲和批处理如何可以配合工作。第二种方案通过在单个存储过程调用中执行主-从操作来提高性能。最后一种方案显示如何在“UPSERT”操作中使用表值参数。

### 缓冲
尽管一些方案中使用批处理很合适，但是也有很多方案下可以通过延迟处理来利用批处理。但是，在出现意外故障时，延迟处理会使数据丢失的风险加大。请务必了解此风险并考虑后果。

例如，假设有一个跟踪每个用户的导航历史记录的 Web 应用程序。对于每个页请求，该应用程序将创建一个数据库调用来记录用户的页面浏览量。但是可以通过缓冲用户的导航活动，然后成批将此数据发送到数据库来提高性能和缩放性。你可用经历的时间和/或缓冲区大小来触发数据库更新。例如，一个规则可能指定在 20 秒后或缓冲区达到 1000 项时应处理批。

以下代码示例使用[反应扩展 - Rx](https://msdn.microsoft.com/data/gg577609) 来处理监视类引发的缓冲事件。当缓冲区已满或达到超时值时，使用表值参数将该批用户数据发送到数据库。

以下 NavHistoryData 类对用户导航详细信息建模。它包含基本信息，如用户标识符、访问的 URL 和访问时间。

	public class NavHistoryData
	{
	    public NavHistoryData(int userId, string url, DateTime accessTime)
	    { UserId = userId; URL = url; AccessTime = accessTime; }
	    public int UserId { get; set; }
	    public string URL { get; set; }
	    public DateTime AccessTime { get; set; }
	}

NavHistoryDataMonitor 类负责将用户导航数据缓冲到数据库。它包含一个方法 RecordUserNavigationEntry，该方法通过引发 **OnAdded** 事件来响应。以下代码显示一个构造函数逻辑，它使用 Rx 基于该事件来创建可查看的集合。然后它使用 Buffer 方法来订阅这个可查看的集合。该重载指定应每隔 20 秒或 1000 项发送一次缓冲区。

	public NavHistoryDataMonitor()
	{
	    var observableData =
	        Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");
	
	    observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);           
	}

处理程序将所有缓冲的项转换为表值类型，然后将此类型传递到处理该批的存储过程。以下代码显示 NavHistoryDataEventArgs 和 NavHistoryDataMonitor 类的完整定义。

	public class NavHistoryDataEventArgs : System.EventArgs
	{
	    public NavHistoryDataEventArgs(NavHistoryData data) { Data = data; }
	    public NavHistoryData Data { get; set; }
	}
	
	public class NavHistoryDataMonitor
	{
	    public event EventHandler<NavHistoryDataEventArgs> OnAdded;
	
	    public NavHistoryDataMonitor()
	    {
	        var observableData =
	            Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");
	
	        observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);           
	    }
	
	    public void RecordUserNavigationEntry(NavHistoryData data)
	    {    
	        if (OnAdded != null)
	            OnAdded(this, new NavHistoryDataEventArgs(data));
	    }
	
	    protected void Handler(IList<EventPattern<NavHistoryDataEventArgs>> items)
	    {
	        DataTable navHistoryBatch = new DataTable("NavigationHistoryBatch");
	        navHistoryBatch.Columns.Add("UserId", typeof(int));
	        navHistoryBatch.Columns.Add("URL", typeof(string));
	        navHistoryBatch.Columns.Add("AccessTime", typeof(DateTime));
	        foreach (EventPattern<NavHistoryDataEventArgs> item in items)
	        {
	            NavHistoryData data = item.EventArgs.Data;
	            navHistoryBatch.Rows.Add(data.UserId, data.URL, data.AccessTime);
	        }
	
	        using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
	        {
	            connection.Open();
	
	            SqlCommand cmd = new SqlCommand("sp_RecordUserNavigation", connection);
	            cmd.CommandType = CommandType.StoredProcedure;
	
	            cmd.Parameters.Add(
	                new SqlParameter()
	                {
	                    ParameterName = "@NavHistoryBatch",
	                    SqlDbType = SqlDbType.Structured,
	                    TypeName = "NavigationHistoryTableType",
	                    Value = navHistoryBatch,
	                });
	
	            cmd.ExecuteNonQuery();
	        }
	    }
	}

为了使用此缓冲类，应用程序将会创建静态 NavHistoryDataMonitor 对象。每次用户访问页时，该应用程序都会调用 NavHistoryDataMonitor.RecordUserNavigationEntry 方法。缓冲逻辑继续执行，以将这些项成批发送到数据库。

### 主从
表值参数对于简单 INSERT 方案很有用。但是，它对于涉及多个表的成批插入作用不大。“主/从”方案是一个很好的示例。主表标识主实体。一个或多个从表存储有关实体的更多数据。在此方案中，外键关系将详细信息的关系强制实施到唯一的主实体。请考虑 PurchaseOrder 表的简化版本和它的关联 OrderDetail 表。以下 Transact-SQL 创建包含以下四个列的 PurchaseOrder 表：OrderID、OrderDate、CustomerID 和 Status。

	CREATE TABLE [dbo].[PurchaseOrder](
	[OrderID] [int] IDENTITY(1,1) NOT NULL,
	[OrderDate] [datetime] NOT NULL,
	[CustomerID] [int] NOT NULL,
	[Status] [nvarchar](50) NOT NULL,
	 CONSTRAINT [PrimaryKey_PurchaseOrder] 
	PRIMARY KEY CLUSTERED ( [OrderID] ASC ))

每个订单包含一个或多个产品采购。此信息存储在 PurchaseOrderDetail 表中。以下 Transact-SQL 创建包含以下五个列的 PurchaseOrderDetail 表：OrderID、OrderDetailID、ProductID、UnitPrice 和 OrderQty。

	CREATE TABLE [dbo].[PurchaseOrderDetail](
	[OrderID] [int] NOT NULL,
	[OrderDetailID] [int] IDENTITY(1,1) NOT NULL,
	[ProductID] [int] NOT NULL,
	[UnitPrice] [money] NULL,
	[OrderQty] [smallint] NULL,
	 CONSTRAINT [PrimaryKey_PurchaseOrderDetail] PRIMARY KEY CLUSTERED 
	( [OrderID] ASC, [OrderDetailID] ASC ))

OrderID 表中的 PurchaseOrderDetail 列必须引用 PurchaseOrder 表的订单。外键的以下定义实施此约束。

	ALTER TABLE [dbo].[PurchaseOrderDetail]  WITH CHECK ADD 
	CONSTRAINT [FK_OrderID_PurchaseOrder] FOREIGN KEY([OrderID])
	REFERENCES [dbo].[PurchaseOrder] ([OrderID])

为了使用表值参数，你对每个目标表必须具有一个用户定义的表类型。

	CREATE TYPE PurchaseOrderTableType AS TABLE 
	( OrderID INT,
	  OrderDate DATETIME,
	  CustomerID INT,
	  Status NVARCHAR(50) );
	GO
	
	CREATE TYPE PurchaseOrderDetailTableType AS TABLE 
	( OrderID INT,
	  ProductID INT,
	  UnitPrice MONEY,
	  OrderQty SMALLINT );
	GO

然后定义接受这些类型的表的存储过程。此过程允许应用程序对单个调用中的一组订单和订单详细信息在本地进行批处理。以下 Transact-SQL 提供此采购订单示例的完整存储过程声明。

	CREATE PROCEDURE sp_InsertOrdersBatch (
	@orders as PurchaseOrderTableType READONLY,
	@details as PurchaseOrderDetailTableType READONLY )
	AS
	SET NOCOUNT ON;
	
	-- Table that connects the order identifiers in the @orders
	-- table with the actual order identifiers in the PurchaseOrder table
	DECLARE @IdentityLink AS TABLE ( 
	SubmittedKey int, 
	ActualKey int, 
	RowNumber int identity(1,1)
	);
	 
	      -- Add new orders to the PurchaseOrder table, storing the actual
	-- order identifiers in the @IdentityLink table   
	INSERT INTO PurchaseOrder ([OrderDate], [CustomerID], [Status])
	OUTPUT inserted.OrderID INTO @IdentityLink (ActualKey)
	SELECT [OrderDate], [CustomerID], [Status] FROM @orders ORDER BY OrderID;
	
	-- Match the passed-in order identifiers with the actual identifiers
	-- and complete the @IdentityLink table for use with inserting the details
	WITH OrderedRows As (
	SELECT OrderID, ROW_NUMBER () OVER (ORDER BY OrderID) As RowNumber 
	FROM @orders
	)
	UPDATE @IdentityLink SET SubmittedKey = M.OrderID
	FROM @IdentityLink L JOIN OrderedRows M ON L.RowNumber = M.RowNumber;
	
	-- Insert the order details into the PurchaseOrderDetail table, 
	      -- using the actual order identifiers of the master table, PurchaseOrder
	INSERT INTO PurchaseOrderDetail (
	[OrderID],
	[ProductID],
	[UnitPrice],
	[OrderQty] )
	SELECT L.ActualKey, D.ProductID, D.UnitPrice, D.OrderQty
	FROM @details D
	JOIN @IdentityLink L ON L.SubmittedKey = D.OrderID;
	GO

在此示例中，本地定义的 @IdentityLink 表存储新插入的行的实际 OrderID 值。这些订单标识符与 OrderID 和 @orders 表值参数中的临时 @details 值不同。因此，@IdentityLink 表然后将 OrderID 参数的 @orders 值与 OrderID 表中新行的实际 PurchaseOrder 值关联。执行此步骤后，@IdentityLink 表可以通过满足外键约束的实际 OrderID 方便插入订单详细信息。

可以从代码或其他 Transact-SQL 调用使用此存储过程。有关代码示例，请参阅本文的表值参数部分。以下 Transact-SQL 显示如何调用 sp\_InsertOrdersBatch。

	declare @orders as PurchaseOrderTableType
	declare @details as PurchaseOrderDetailTableType
	
	INSERT @orders 
	([OrderID], [OrderDate], [CustomerID], [Status])
	VALUES(1, '1/1/2013', 1125, 'Complete'),
	(2, '1/13/2013', 348, 'Processing'),
	(3, '1/12/2013', 2504, 'Shipped')
	
	INSERT @details
	([OrderID], [ProductID], [UnitPrice], [OrderQty])
	VALUES(1, 10, $11.50, 1),
	(1, 12, $1.58, 1),
	(2, 23, $2.57, 2),
	(3, 4, $10.00, 1)
	
	exec sp_InsertOrdersBatch @orders, @details

此解决方案允许每个批使用从 1 开始的一组 OrderID 值。这些临时 OrderID 值描述批中的关系，但是在执行插入操作时确定实际 OrderID 值。你可以重复运行前一示例中的相同语句，在数据库中生成唯一订单。为此，请考虑在使用此批处理方法时添加防止重复订单的更多代码或数据库逻辑。

此示例演示使用表值参数可以成批执行更复杂的数据库操作（如主-从操作）。

### UPSERT
另一批处理方案涉及同时更新现有行和插入新行。此操作有时称为“UPSERT”（更新 + 插入）操作。不用单独调用 INSERT 和 UPDATE，MERGE 语句最适合此任务。MERGE 语句可以在单个调用中执行插入和更新操作。

可以将表值参数用于 MERGE 语句以执行更新和插入。例如，请考虑使用包含以下列的简化 Employee 表：EmployeeID、FirstName、LastName、SocialSecurityNumber：

	CREATE TABLE [dbo].[Employee](
	[EmployeeID] [int] IDENTITY(1,1) NOT NULL,
	[FirstName] [nvarchar](50) NOT NULL,
	[LastName] [nvarchar](50) NOT NULL,
	[SocialSecurityNumber] [nvarchar](50) NOT NULL,
	 CONSTRAINT [PrimaryKey_Employee] PRIMARY KEY CLUSTERED 
	([EmployeeID] ASC ))
 
在此示例中，你可以使用 SocialSecurityNumber 是唯一的这个事实来执行多个员工的 MERGE。首先，创建用户定义的表类型：

	CREATE TYPE EmployeeTableType AS TABLE 
	( Employee_ID INT,
	  FirstName NVARCHAR(50),
	  LastName NVARCHAR(50),
	  SocialSecurityNumber NVARCHAR(50) );
	GO

接下来，创建一个使用 MERGE 语句的存储过程或编写包含该语句的代码来执行更新和插入。以下示例使用针对类型为 EmployeeTableType 的表值参数 @employees 的 MERGE 语句。@employees 表的内容未在此处显示。

	MERGE Employee AS target
	USING (SELECT [FirstName], [LastName], [SocialSecurityNumber] FROM @employees) 
	AS source ([FirstName], [LastName], [SocialSecurityNumber])
	ON (target.[SocialSecurityNumber] = source.[SocialSecurityNumber])
	WHEN MATCHED THEN 
	UPDATE SET
	target.FirstName = source.FirstName, 
	target.LastName = source.LastName
	WHEN NOT MATCHED THEN
	   INSERT ([FirstName], [LastName], [SocialSecurityNumber])
	   VALUES (source.[FirstName], source.[LastName], source.[SocialSecurityNumber]);

有关详细信息，请参阅 MERGE 语句的文档和示例。尽管可以在包含单独 INSERT 和 UPDATE 操作的多步骤存储过程调用中完成同样的工作，但是 MERGE 语句更有效。数据库代码还可以构造直接使用 MERGE 语句的 Transact-SQL 调用而无需对 INSERT 和 UPDATE 使用两个数据库调用。

## 建议摘要

以下列表提供了本主题中讨论的批处理建议的摘要：

- 使用缓冲和批处理可提高 SQL 数据库应用程序的性能和缩放性。
- 了解批处理/缓冲和弹性之间的权衡问题。在角色失败期间，可能遗失一批尚未处理的商务关键数据，这种风险超过批处理带来的性能优点。
- 尝试将所有数据库调用纳入单一数据中心以缩短延迟。
- 如果选择单个批处理方法，使用表值参数可实现最佳性能和灵活性。
- 要实现最快速的插入性能，请遵循以下常规准则，但是要针对你的方案进行测试：
	- 对于 < 100 行，使用单个参数化的 INSERT 命令。
	- 对于 < 1000 行，使用表值参数。
	- 对于 >= 1000 行，使用 SqlBulkCopy。
- 对于更新和删除操作，请将表值参数用于存储过程逻辑，该逻辑确定对表参数中每行的正确操作。
- 批大小准则：
	- 使用可满足你的应用程序和业务需求的最大批大小。
	- 掌握好大批次带来的性能提升与临时或灾难性故障的风险之间的平衡。批中数据重试或丢失的后果是什么？ 
	- 测试最大批大小，以验证 SQL 数据库不拒绝它。
	- 创建控制批处理的配置设置，如批大小或缓冲时间窗口。这些设置提供灵活性。你可以在生产中更改批处理行为，而无需重新部署云服务。
- 避免并行执行对一个数据库中单个表进行操作的批处理。如果选择将单个批分配给多个工作线程，请运行测试来确定理想的线程数。在达到某个阈值后，线程增加将导致性能下降而非提升。
- 请考虑对大小和时间进行缓冲，为更多方案实现批处理。

## 后续步骤

本文着重于与批处理相关的数据库设计和代码编写技术，以及如何改善应用程序的性能和缩放性。但这只是整体策略中的一个因素。关于其他可改善性能和缩放性的方式，请参阅 [Azure SQL 数据库的单一数据库性能指导](/documentation/articles/sql-database-performance-guidance/)及[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。

<!---HONumber=Mooncake_0321_2016-->