<properties 
	pageTitle="用于连接到 SQL 数据库的 C# 重试逻辑 | Azure" 
	description="C# 示例包含用于与 Azure SQL 数据库可靠交互的重试逻辑。" 
	services="sql-database" 
	documentationCenter="" 
	authors="annemill"
	manager="jhubbard"
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="03/15/2016"
	wacn.date="03/24/2016"/>


# 代码示例：用于连接到 SQL 数据库的 C# 重试逻辑


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)] 



本主题提供一个用于演示自定义重试逻辑的 C# 代码示例。重试逻辑旨在正常处理暂时错误或 *暂时性故障*，在程序等待几秒并重试后，这种错误或故障往往会自行消失。


用于连接本地 Microsoft SQL Server 的 ADO.NET 类也可以连接到 Azure SQL 数据库。但是，ADO.NET 类本身无法提供生产环境中所需的稳定性和可靠性。客户端程序可能会遇到暂时性故障，在此情况下，客户端程序应该能够自行从故障中静默正常恢复。


暂时性故障的几个示例包括：


- 通过 Internet 建立的连接受到短暂的网络中断，然后可以重建连接。

- 云计算涉及到负载平衡，可以短暂阻止连接或查询尝试。


## 识别暂时性错误


程序必须区分暂时性错误与持久性错误。如果程序发现拼错了目标数据库名称，“找不到此类数据库”错误将持续出现，并且每次当你重新运行程序时都会再次出现。


以下网页提供了分类为暂时性故障的错误编列表：


- [SQL 数据库客户端程序的错误消息](/documentation/articles/sql-database-develop-error-messages/#bkmk_connection_errors)
 - 请参阅顶部部分，了解*暂时性错误和连接断开错误*。


## C# 代码示例


当前主题中的 C# 代码示例包含用于处理暂时性错误的自定义检测与重试逻辑。此示例假定已安装 .NET Framework 4.5.1 或更高版本。


代码示例遵循一些基本指导原则或者无论你使用哪种技术来与 Azure SQL 数据库交互都适用的建议。你可以在此处查看常规建议：


- [连接到 SQL 数据库：链接、最佳实践和设计准则](/documentation/articles/sql-database-connect-central-recommendations)


C# 代码示例包含一个名为 Program.cs 的文件。将其代码粘贴到下一节中。


### 捕获和编译代码示例


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
	using D = System.Data;
	using C = System.Data.SqlClient;
	using X = System.Text;
	using H = System.Threading;
	
	namespace RetryAdo2
	{
		class Program
		{
			static void Main(string[] args)
			{
				Program program = new Program();
				bool returnBool;
	
				returnBool = program.Run(args);
				if (returnBool == false)
				{
					Console.WriteLine("Something failed.  :-( ");
				}
				return;
			}
	
			bool Run(string[] _args)
			{
				C.SqlConnectionStringBuilder sqlConnectionSB;
				C.SqlConnection sqlConnection;
				D.IDbCommand dbCommand;
				D.IDataReader dataReader;
				X.StringBuilder sBuilder = new X.StringBuilder(256);
				int retryIntervalSeconds = 10;
				bool returnBool = false;
	
				for (int tries = 1; tries <= 5; tries++)
				{
					try
					{
						if (tries > 1)
						{
							H.Thread.Sleep(1000 * retryIntervalSeconds);
							retryIntervalSeconds = Convert.ToInt32(retryIntervalSeconds * 1.5);
						}
						this.GetSqlConnectionStringBuilder(out sqlConnectionSB);
	
						sqlConnection = new C.SqlConnection(sqlConnectionSB.ToString());
	
						dbCommand = sqlConnection.CreateCommand();
						dbCommand.CommandText = @"SELECT TOP 3 ob.name, CAST(ob.object_id as nvarchar(32)) as [object_id] FROM sys.objects as ob WHERE ob.type='IT' ORDER BY ob.name;";
	
						sqlConnection.Open();
						dataReader = dbCommand.ExecuteReader();
	
						while (dataReader.Read())
						{
							sBuilder.Length = 0;
							sBuilder.Append(dataReader.GetString(0));
							sBuilder.Append("\t");
							sBuilder.Append(dataReader.GetString(1));
	
							Console.WriteLine(sBuilder.ToString());
						}
						returnBool = true;
						break;
					}
	
					catch (C.SqlException sqlExc)
					{
						if (this.m_listTransientErrorNumbers.Contains(sqlExc.Number) == true)
						{ continue; }
						else
						{ throw sqlExc; }
					}
				}
				return returnBool;
			}
	
			void GetSqlConnectionStringBuilder(out C.SqlConnectionStringBuilder _sqlConnectionSB)
			{
				// Prepare the connection string to Azure SQL Database.
				_sqlConnectionSB = new C.SqlConnectionStringBuilder();
	
				// Change these values to your values.
				_sqlConnectionSB["Server"] = "tcp:myazuresqldbserver.database.chinacloudapi.cn,1433";
				_sqlConnectionSB["User ID"] = "MyLogin";  // "@yourservername"  as suffix sometimes.
				_sqlConnectionSB["Password"] = "MyPassword";
				_sqlConnectionSB["Database"] = "MyDatabase";
	
				// Adjust these values if you like. (.NET 4.5.1 or later.)
				_sqlConnectionSB["ConnectRetryCount"] = 3;
				_sqlConnectionSB["ConnectRetryInterval"] = 10;  // Seconds.
	
				// Leave these values as they are.
				_sqlConnectionSB["Trusted_Connection"] = false;
				_sqlConnectionSB["Integrated Security"] = false;
				_sqlConnectionSB["Encrypt"] = true;
				_sqlConnectionSB["Connection Timeout"] = 30;
			}
	
			Program()   // Constructor.
			{
				int[] arrayOfTransientErrorNumbers =
					{ 4060, 40197, 40501, 40613, 49918, 49919, 49920
						//,11001   // 11001 for testing, pretend network error is transient.
					};
				m_listTransientErrorNumbers = new G.List<int>(arrayOfTransientErrorNumbers);
			}
	
			private G.List<int> m_listTransientErrorNumbers;
		}
	}



### 运行该程序


**RetryAdo2.exe** 可执行文件未输入任何参数。若要在 Visual Studio 中运行 .exe，请执行以下操作:


1. 在 **Main** 方法中的 **return;** 语句上设置一个断点。

2. 单击绿色启动箭头按钮。将显示控制台窗口。

3. 当调试器在 **Main** 结尾处停止时，切换到控制台窗口。

4. 将看到可能等同于以下内容的三行：


```
database_firewall_rules_table   245575913
filestream_tombstone_2073058421 2073058421
filetable_updates_2105058535    2105058535
```


## 测试重试逻辑


测试重试逻辑的一个简便方法是：


1. 将 **11001** 临时添加到应视为暂时性错误的 **SqlConnection.Number** 值的集合中。

2. 重新编译你的程序。

3. 从网络中断开客户端计算机。

4. 在调试器中运行你的程序，并在循环中设有断点。
 - 第一个循环将失败，错误为 11001。

5. 当该程序在第二个循环期间在断点处停留时，将计算机重新连接到网络。

6. 继续运行你的程序。它在第二个循环期间运行成功。


### 另一种测试方法


另一种方法可以是向你的程序添加逻辑以识别命令行参数值“test”。在响应参数时，该程序会：


1. 临时追加无用字母以使 SQL 数据库服务器名称拼写错误。

2. 临时将 **40615** 添加到暂时性错误列表。

3. 在其第二个循环开始时（意味着这是其第一个重试循环），该程序会：
 - 撤消服务器拼写错误。
 - 从暂时性列表中删除 40615。


使用“test”参数运行该程序，并验证它是否第一次失败，但然后在其第二个循环成功。


## 相关链接


- [SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples)

- [试用 SQL 数据库：使用 C# 通过适用于 .NET 的 SQL 数据库库创建 SQL 数据库](/documentation/articles/sql-database-get-started-csharp)

<!---HONumber=Mooncake_0104_2016-->
