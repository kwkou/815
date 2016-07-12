<properties
	pageTitle="SQL 数据库的 C# 重试逻辑 | Azure"
	description="C# 示例包含用于与 Azure SQL 数据库可靠交互的重试逻辑。"
	services="sql-database"
	documentationCenter=""
	authors="annemill"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="04/08/2016"
	wacn.date="05/16/2016"/>


# 代码示例：用于连接到 SQL 数据库的 C# 重试逻辑


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]



本主题提供一个用于演示自定义重试逻辑的 C# 代码示例。重试逻辑旨在正常处理暂时性错误或暂时性故障，在程序等待几秒并重试后，这种错误或故障往往会自行消失。


用于连接本地 Microsoft SQL Server 的 ADO.NET 类也可以连接到 Azure SQL 数据库。但是，ADO.NET 类本身无法提供生产环境中所需的稳定性和可靠性。客户端程序可能会遇到暂时性故障，在此情况下，客户端程序应该能够自行从故障中静默正常恢复和继续。


暂时性故障的几个示例包括：


- 通过 Internet 建立的连接受到短暂的网络中断，然后可以重建连接。
- 云计算涉及到负载平衡，可以短暂阻止连接或查询尝试。


## A.识别暂时性错误


程序必须区分暂时性错误与持久性错误。如果程序发现拼错了目标数据库名称，“找不到此类数据库”错误将持续出现，并且每次当你重新运行程序时都会再次出现。

以下网页提供了分类为暂时性故障的错误编列表：

- [SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/#bkmk_connection_errors)
  - 请参阅顶部部分，了解暂时性错误和连接断开错误。


本主题稍后的 C# 代码示例列出了名为“TransientErrorNumbers”的字段中的暂时性错误数量。



## B.C# 代码示例


当前主题中的 C# 代码示例包含用于处理暂时性错误的自定义检测与重试逻辑。此示例假定已安装 .NET Framework 4.5.1 或更高版本。


代码示例遵循一些基本指导原则或者无论你使用哪种技术来与 Azure SQL 数据库交互都适用的建议。你可以在此处查看常规建议：


- [连接到 SQL 数据库：链接、最佳实践和设计准则](/documentation/articles/sql-database-connect-central-recommendations/)


C# 代码示例包含一个名为 Program.cs 的文件。下一节中提供其代码。


### B.1 捕获和编译代码示例


可以使用以下步骤编译该示例：


1. 在[免费的 Visual Studio Community 版本](https://www.visualstudio.com/products/visual-studio-community-vs)中，基于 C# 控制台应用程序模板创建一个新项目。
  - “文件”>“新建”>“项目”>“已安装”>“模板”> Visual C# > Windows >“经典桌面”>“控制台应用程序”
  - 将该项目命名为 **RetryAdo2**。
2. 打开“解决方案资源管理器”窗格。
  - 查看项目的名称。
  - 查看 Program.cs 文件的名称。
3. 打开 Program.cs 文件。
4. 将 Program.cs 文件的内容全部替换为下面的代码块中的代码。
5. 单击菜单“生成”>“生成解决方案”。


#### 要粘贴的 C# 源代码


将此代码粘贴到你的 **Program.cs** 文件中。


然后，你必须编辑服务器名称、密码等字符串。你可以在名为 **GetSqlConnectionStringBuilder** 的方法中找到这些字符串。

	
	using System;   // C#
	using G = System.Collections.Generic;
	using S = System.Data.SqlClient;
	using T = System.Threading;
	   
	namespace RetryAdo2
	{
	   public class Program
	   {
	      static public int Main(string[] args)
	      {
	         bool succeeded = false;
	         int totalNumberOfTimesToTry = 4;
	         int retryIntervalSeconds = 10;
	   
	         for (int tries = 1;
	            tries <= totalNumberOfTimesToTry;
	            tries++)
	         {
	            try
	            {
	               if (tries > 1)
	               {
	                  Console.WriteLine
	                     ("Transient error encountered. Will begin attempt number {0} of {1} max...",
	                     tries, totalNumberOfTimesToTry
	                     );
	                  T.Thread.Sleep(1000 * retryIntervalSeconds);
	                  retryIntervalSeconds = Convert.ToInt32
	                     (retryIntervalSeconds * 1.5);
	               }
	               AccessDatabase();
	               succeeded = true;
	               break;
	            }
	   
	            catch (S.SqlException sqlExc)
	            {
	               if (TransientErrorNumbers.Contains
	                  (sqlExc.Number) == true)
	               {
	                  Console.WriteLine("{0}: transient occurred.", sqlExc.Number);
	                  continue;
	               }
	               else
	               {
	                  Console.WriteLine(sqlExc);
	                  succeeded = false;
	                  break;
	               }
	            }
	   
	            catch (TestSqlException sqlExc)
	            {
	               if (TransientErrorNumbers.Contains
	                  (sqlExc.Number) == true)
	               {
	                  Console.WriteLine("{0}: transient occurred. (TESTING.)", sqlExc.Number);
	                  continue;
	               }
	               else
	               {
	                  Console.WriteLine(sqlExc);
	                  succeeded = false;
	                  break;
	               }
	            }
	   
	            catch (Exception Exc)
	            {
	               Console.WriteLine(Exc);
	               succeeded = false;
	               break;
	            }
	         }
	   
	         if (succeeded == true)
	         {
	            return 0;
	         }
	         else
	         {
	            Console.WriteLine("ERROR: Unable to access the database!");
	            return 1;
	         }
	      }
	   
	      /// <summary>
	      /// Connects to the database, reads,
	      /// prints results to the console.
	      /// </summary>
	      static public void AccessDatabase()
	      {
	         //throw new TestSqlException(4060); //(7654321);  // Uncomment for testing.
	   
	         using (var sqlConnection = new S.SqlConnection
	                  (GetSqlConnectionString()))
	         {
	            using (var dbCommand = sqlConnection.CreateCommand())
	            {
	               dbCommand.CommandText = @"
	SELECT TOP 3
	      ob.name,
	      CAST(ob.object_id as nvarchar(32)) as [object_id]
	   FROM sys.objects as ob
	   WHERE ob.type='IT'
	   ORDER BY ob.name;";
	   
	               sqlConnection.Open();
	               var dataReader = dbCommand.ExecuteReader();
	   
	               while (dataReader.Read())
	               {
	                  Console.WriteLine("{0}\t{1}",
	                     dataReader.GetString(0),
	                     dataReader.GetString(1));
	               }
	            }
	         }
	      }
	   
	      /// <summary>
	      /// You must edit the four 'my' string values.
	      /// </summary>
	      /// <returns>An ADO.NET connection string.</returns>
	      static private string GetSqlConnectionString()
	      {
	         // Prepare the connection string to Azure SQL Database.
	         var sqlConnectionSB = new S.SqlConnectionStringBuilder();
	   
	         // Change these values to your values.
	         sqlConnectionSB.DataSource = "tcp:myazuresqldbserver.database.chinacloudapi.cn,1433"; //["Server"]
	         sqlConnectionSB.InitialCatalog = "MyDatabase"; //["Database"]
	   
	         sqlConnectionSB.UserID = "MyLogin";  // "@yourservername"  as suffix sometimes.
	         sqlConnectionSB.Password = "MyPassword";
	         sqlConnectionSB.IntegratedSecurity = false;
	   
	         // Adjust these values if you like. (ADO.NET 4.5.1 or later.)
	         sqlConnectionSB.ConnectRetryCount = 3;
	         sqlConnectionSB.ConnectRetryInterval = 10;  // Seconds.
	   
	         // Leave these values as they are.
	         sqlConnectionSB.IntegratedSecurity = false;
	         sqlConnectionSB.Encrypt = true;
	         sqlConnectionSB.ConnectTimeout = 30;
	   
	         return sqlConnectionSB.ToString();
	      }
	   
	      static public G.List<int> TransientErrorNumbers =
	         new G.List<int> { 4060, 40197, 40501, 40613,
	         49918, 49919, 49920, 11001 };
	   }
	   
	   /// <summary>
	   /// For testing retry logic, you can have method
	   /// AccessDatabase start by throwing a new
	   /// TestSqlException with a Number that does
	   /// or does not match a transient error number
	   /// present in TransientErrorNumbers.
	   /// </summary>
	   internal class TestSqlException : ApplicationException
	   {
	      internal TestSqlException(int testErrorNumber)
	      { this.Number = testErrorNumber; }
	   
	      internal int Number
	      { get; set; }
	   }
	}




## C.运行该程序


**RetryAdo2.exe** 可执行文件未输入任何参数。运行 .exe：

1. 在你已编译 RetryAdo2.exe 二进制的位置打开一个控制台窗口。
2. 运行 RetryAdo2.exe，不用输入参数。

	
		database_firewall_rules_table   245575913
		filestream_tombstone_2073058421 2073058421
		filetable_updates_2105058535    2105058535




## D.测试重试逻辑的方法

有许多方法可模拟暂时性错误测试重试逻辑。


### D.1 触发测试异常

代码示例包括：

- 名为 **TestSqlException** 的小型辅助类，这是一个名为 **Number** 的属性。
- `//throw new TestSqlException(4060);`，你可以取消它的注释。

如果你取消注释 throw 后再重新编译，则 **RetryAdo2.exe** 的下一次运行输出将类似以下内容。

	
	[C:\_MainW\VS15\RetryAdo2\RetryAdo2\bin\Debug]
	>> RetryAdo2.exe
	4060: transient occurred. (TESTING.)
	Transient error encountered. Will begin attempt number 2 of 4 max...
	4060: transient occurred. (TESTING.)
	Transient error encountered. Will begin attempt number 3 of 4 max...
	4060: transient occurred. (TESTING.)
	Transient error encountered. Will begin attempt number 4 of 4 max...
	4060: transient occurred. (TESTING.)
	ERROR: Unable to access the database!
	
	[C:\_MainW\VS15\RetryAdo2\RetryAdo2\bin\Debug]
	>>




#### 使用永久性错误重新测试


要证明代码正确处理永久性错误，重新运行之前的测试，但不要使用 4060 等真实的暂时性错误编号。改用无意义的编号 7654321。该程序应将此视为永久性错误，并应绕过任何重试。



### D.2 从网络断开

1. 从网络中断开客户端计算机。
  - 对于台式电脑，拔下网线。
  - 对于笔记本电脑，按功能组合键关闭网络适配器。
2. 启动 RetryAdo2.exe，并等待控制台显示第一个暂时性错误，例如 11001。
3. RetryAdo2.exe 继续运行时，重新连接网络。
4. 在后续重试中看到控制台报告成功。


### D.3 暂时拼错服务器名称

1. 暂时将 40615 作为又一个错误编号添加到 **TransientErrorNumbers**，并重新编译。
2. 在行上设置断点：`new S.SqlConnectionStringBuilder()`。
3. 使用“编辑并继续”功能特意拼错服务器名称，以下几行。
  - 运行程序并返回到断点。
  - 发生错误 40615。
4. 修复拼写错误。
5. 运行程序，成功完成。
6. 删除 40615，重新编译。


## E.相关链接

- [SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples/)
- [试用 SQL 数据库：使用 C# 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp/)

<!---HONumber=Mooncake_0509_2016-->
