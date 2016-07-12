<properties 
	pageTitle="EntLib 重试连接到 SQL 数据库 | Azure"
	description="Enterprise Library 旨在简化云服务的客户端程序的多个任务，包括集成暂时性故障的重试逻辑。"
	services="sql-database"
	documentationCenter=""
	authors="annemill"
	manager="jhubbard"
	editor="" />


<tags 
	ms.service="sql-database" 
	ms.date="03/15/2016"
    wacn.date="03/24/2016"/>


# 代码示例：Enterprise Library 6 中用于连接到 SQL 数据库的 C&#x23; 重试逻辑


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)] 



本主题提供了一个用于演示 Enterprise Library (EntLib) 的完整代码示例。EntLib 简化了与云服务（如 Azure SQL 数据库）进行交互的客户端程序的许多任务。我们的示例重点介绍包括暂时性故障的重试逻辑在内的重要任务。


EntLib 类旨在区分两种类别的运行时错误：

- 永远不会自行更正的错误，如拼写错误的服务器名称。
- 暂时性故障，例如，在 Azure 系统进行负载平衡的情况下，服务器在接受新连接时暂停了几秒钟。


Enterprise Library 6 (EntLib60) 是最新版本，并已于 2013 年 4 月发布。

- Microsoft 已向公众发布源代码。
- Microsoft 没有进一步维护源代码的计划。


## 先决条件


#### .NET Framework 4.0 或更高版本


必须安装 Microsoft.NET Framework 4.0 或更高版本。在撰写本文时版本 4.6 可用，建议使用最新版本。


#### Visual Studio Community 版本是免费的


你需要有方法编译此示例中的源代码。一种方法是安装[免费的 Microsoft Visual Studio *Community* 版本](http://www.visualstudio.com/products/free-developer-offers-vs.aspx)。


你可能需要向 MSDN 注册你的电子邮件地址。步骤与下面类似：


1. [转到 MSDN](http://msdn.microsoft.com)。
2. 单击顶部附近的“MSDN 订阅”。
3. 单击“立即注册”。
4. 在表单中填写你的信息。
5. 单击底部的“创建帐户”。


#### Enterprise Library 6 (EntLib60)


可以安装 EntLib60 的方式：


- 使用 Visual Studio 中的 *NuGet* 包管理器功能：
 - 在 NuGet 中，搜索 **enterpriselibrary**。


- 在“[EntLib60 的家庭文档主题](http://msdn.microsoft.com/zh-cn/library/dn169621.aspx)”中，找到标记为“下载”的行，然后单击 [Microsoft Enterprise Library 6](http://go.microsoft.com/fwlink/?linkid=290898)，以下载该二进制 .DLL 程序集文件。


EntLib60 包含若干个 .DLL 程序集文件，这些文件的名称均以相同的前缀 **Microsoft.Practices.EnterpriseLibrary.&#x2a;.dll** 开头，但此代码示例仅关注以下两个程序集：

- Microsoft.Practices.EnterpriseLibrary.**TransientFaultHandling**.dll
- Microsoft.Practices.EnterpriseLibrary.**TransientFaultHandling.Data**.dll


## EntLib 类如何组合在一起


EntLib 类用于构造其他 EntLib 类。在此代码示例中，构造和使用顺序如下面所列出：


1. 构造 **ExponentialBackoff** 对象。
2. 构造 **SqlDatabaseTransientErrorDetectionStrategy** 对象。
3. 构造 **RetryPolicy** 对象。输入参数包括：
 - **ExponentialBackoff** 对象。
 - **SqlDatabaseTransientErrorDetectionStrategy** 对象。
4. 构造 **ReliableSqlConnection** 对象。输入参数包括：
 - 一个 **String** 对象 - 包含服务器名称和其他连接信息。
 - **RetryPolicy** 对象。
5. 调用以通过 **RetryPolicy .ExecuteAction** 方法进行连接。
6. 调用 **ReliableSqlConnection .CreateCommand** 方法。
 - 返回 **System.SqlClient.Data.DbCommand** 对象，该对象是 ADO.NET 的一部分。
7. 调用以通过 **RetryPolicy .ExecuteAction** 方法进行查询。


## 编译并运行代码示例


Program.cs 源代码示例将在本主题后面部分提供。可以使用以下步骤编译和运行该示例：


1. 在 Visual Studio 中，通过 C# 控制台应用程序模板创建一个新项目。

2. 在“解决方案资源管理器”窗格中，通过将初始内容替换为本主题后面提供的 Program.cs 代码来编辑几乎是空的 Program.cs 文件。

3. 使用 Visual Studio 中的“生成”>“生成解决方案”菜单编译你的项目。

4. 在 cmd.exe 命令窗口中，按如下所示运行程序。图中还显示了运行后的实际输出：
 
	
		[C:\MyVS\EntLib60Retry\EntLib60Retry\bin\Debug]
		>> EntLib60Retry.exe
		
		database_firewall_rules_table   245575913
		filestream_tombstone_2073058421 2073058421
		filetable_updates_2105058535    2105058535
		
		[C:\MyVS\EntLib60Retry\EntLib60Retry\bin\Debug]
		>>



&nbsp;


## Program.cs 源代码


下面的 Program.cs 文件中包含此 EntLib 示例的所有源代码。

	 
	using     System;   // C#
	using G = System.Collections.Generic;
	using D = System.Data;
	using C = System.Data.SqlClient;
	using X = System.Text;
	using H = System.Threading;
	using Y = Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling;
	
	namespace EntLib60Retry
	{
	   class Program
	   {
	      static void Main(string[] args)
	      {
	         Program program = new Program();
	
	         if (program.Run(args) == false)
	         {
	            Console.WriteLine("Something was unable to complete.  :-( ");
	         }
	      }
	
	      bool Run(string[] _args)
	      {
	         int retryIntervalSeconds = 10;
	         bool returnBool = false;
	
	         this.InitializeEntLibObjects();
	
	         for (int tries = 1; tries <= 3; tries++)
	         {
	            if (tries > 1)
	            {
	               H.Thread.Sleep(1000 * retryIntervalSeconds);
	               retryIntervalSeconds = Convert.ToInt32(retryIntervalSeconds * 1.5);
	            }
	
	            try
	            {
	               this.reliableSqlConnection = new Y.ReliableSqlConnection(
	                  this.sqlConnectionSB.ToString(),
	                  this.retryPolicy, null
	                  );
	               this.retryPolicy.ExecuteAction(this.OpenTheConnection_action);  // Open the connection.
	
	               this.dbCommand = this.reliableSqlConnection.CreateCommand();
	               this.dbCommand.CommandText = @"SELECT TOP 3 ob.name, CAST(ob.object_id as nvarchar(32)) as [object_id] FROM sys.objects as ob WHERE ob.type='IT' ORDER BY ob.name;";
	
	               // We retry connection .Open after transient faults, but
	               // we do not retry commands that use the connection.
	               this.IssueTheQuery();  // Run the query, loop through results.
	
	               returnBool = true;
	               break;
	            }
	
	            catch (C.SqlException sqlExc)
	            {
	               if (false == this.sqlDatabaseTransientErrorDetectionStrategy.IsTransient(sqlExc))
	               {
	                  throw sqlExc;
	               }
	            }
	         }
	         return returnBool;
	      }
	
	      void OpenTheConnection()  // Called by .ExecuteAction.
	      {
	         this.reliableSqlConnection.Open();
	      }
	
	      void IssueTheQuery()      // Called by .ExecuteAction.
	      {
	         X.StringBuilder sBuilder = new X.StringBuilder(256);
	
	         this.dataReader = this.dbCommand.ExecuteReader();
	
	         while (this.dataReader.Read())
	         {
	            sBuilder.Length = 0;
	            sBuilder.Append(this.dataReader.GetString(0));
	            sBuilder.Append("\t");
	            sBuilder.Append(this.dataReader.GetString(1));
	
	            Console.WriteLine(sBuilder.ToString());
	         }
	      }
	
	      void InitializeEntLibObjects()
	      {
	         this.InitializeSqlConnectionStringBuilder();
	
	         this.exponentialBackoff = new Y.ExponentialBackoff(
	            "exponentialBackoff",
	             3,                    // Maximum number of times to retry, then let exception bubble up.
	             TimeSpan.FromMilliseconds(10000), // Minimum interval between retries.
	             TimeSpan.FromMilliseconds(30000), // Maximum interval between retries.
	             TimeSpan.FromMilliseconds( 2000)  // For random differences in interval between retries.
	            );
	         this.sqlDatabaseTransientErrorDetectionStrategy =
	            new Y.SqlDatabaseTransientErrorDetectionStrategy();
	         this.retryPolicy = new Y.RetryPolicy(
	               this.sqlDatabaseTransientErrorDetectionStrategy,
	               this.exponentialBackoff
	               );
	         this.OpenTheConnection_action = delegate() { this.OpenTheConnection(); };
	      }
	
	      void InitializeSqlConnectionStringBuilder()
	      {
	         // Prepare the connection string to Azure SQL Database.
	         this.sqlConnectionSB = new C.SqlConnectionStringBuilder();
	
	         // Change these values to your values.
	         this.sqlConnectionSB["Server"] = "tcp:myazuresqldbserver.database.chinacloudapi.cn,1433";
	         this.sqlConnectionSB["User ID"] = "MyLogin";  // "@yourservername"  as suffix sometimes.
	         this.sqlConnectionSB["Password"] = "MyPassword";
	         this.sqlConnectionSB["Database"] = "MyDatabase";
	
	         // Leave these values as they are.
	         this.sqlConnectionSB["Trusted_Connection"] = false;
	         this.sqlConnectionSB["Integrated Security"] = false;
	         this.sqlConnectionSB["Encrypt"] = true;
	         this.sqlConnectionSB["Connection Timeout"] = 30;
	      }
	
	      Program()   // Constructor.
	      {
	         int[] arrayOfTransientErrorNumbers =
	                { 4060, 10928, 10929, 40197, 40501, 40613, 49918, 49919, 49920
	                    //,11001   // 11001 for testing, pretend network error is transient.
	                    //,18456   // 18456 for testing, purposely misspell database name.
	                };
	         listTransientErrorNumbers = new G.List<int>(arrayOfTransientErrorNumbers);
	      }
	
	      // Fields.
	      private G.List<int> listTransientErrorNumbers;
	
	      private Y.ReliableSqlConnection reliableSqlConnection;
	      private Y.ExponentialBackoff exponentialBackoff;
	      private Y.SqlDatabaseTransientErrorDetectionStrategy
	                   sqlDatabaseTransientErrorDetectionStrategy;
	      private Y.RetryPolicy retryPolicy;
	
	      private Action OpenTheConnection_action;
	
	      private C.SqlConnectionStringBuilder sqlConnectionSB;
	      private D.IDbCommand dbCommand;
	      private D.IDataReader dataReader;
	   }
	}



&nbsp;


## 相关链接


- [Enterprise Library 6 – 2013 年 4 月](http://msdn.microsoft.com/zh-cn/library/dn169621.aspx)中提供了大量链接来帮助你了解更多信息。
 - 如果你想要查看源代码，本主题顶部的按钮提供了[下载 EntLib60 源代码](http://go.microsoft.com/fwlink/p/?LinkID=290898)。


- Microsoft 提供的 .PDF 格式的免费电子书：
[Microsoft Enterprise Library 版本 2 开发人员指南](http://www.microsoft.com/en-us/download/details.aspx?id=41145)。


- [Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling 命名空间](http://msdn.microsoft.com/zh-cn/library/microsoft.practices.enterpriselibrary.transientfaulthandling.aspx)


- [Enterprise Library 6 类库参考](http://msdn.microsoft.com/zh-cn/library/dn170426.aspx)


- [代码示例：使用 ADO.NET 连接到 SQL 数据库的 C# 重试逻辑](/documentation/articles/sql-database-develop-csharp-retry-windows/)


- [SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples/)

<!---HONumber=Mooncake_0104_2016-->
