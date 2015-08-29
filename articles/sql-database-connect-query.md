
<properties 
	pageTitle="使用 C# 连接和查询 SQL Database" 
	description="使用 ADO.NET 连接到 Azure SQL Database 云服务上的 AdventureWorks 数据库并与其交互的 C# 客户端代码示例。"
	services="sql-database" 
	documentationCenter="" 
	authors="ckarst" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	wacn.date="" ms.service="sql-database" ms.date="04/14/2015" />


# 使用 C&#x23; 连接和查询 SQL Database

本主题提供一个 C# 代码示例，演示如何使用 ADO.NET 连接到现有的 AdventureWorks SQL 数据库。该示例将编译成一个用于查询数据库并显示结果的控制台应用程序。


## 先决条件


- Azure SQL Database 上的现有 AdventureWorks 数据库。[在数分钟内即可创建一个](/documentation/articles/sql-database-get-started)。
- [装有 .NET Framework 的 Visual Studio](https://www.visualstudio.com/zh-CN/visual-studio-homepage-vs.aspx)


## 步骤 1：控制台应用程序


1. 使用 Visual Studio 创建 C# 控制台应用程序。


![连接和查询](./media/sql-database-connect-query/ConnectandQuery_VisualStudio.png)


## 步骤 2：SQL 代码示例


1. 将以下代码示例复制并粘贴到你的控制台应用程序。


> [AZURE.WARNING]该代码示例在编写时追求简短，以方便用户理解。该示例不适合在生产环境中使用。


此代码不适合在生产环境中使用。如果你想要实施生产就绪代码，以下要求被认为是行业最佳实践：


- 异常处理。
- 针对暂时性错误的重试逻辑。
- 在配置文件中安全存储密码。



### C# 示例的源代码


将此源代码粘贴到你的 **Program.cs** 文件中。


	using System;  // C#
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using System.Data.SqlClient;
	
	namespace ConnectandQuery_Example
	{
		class Program
		{
			static void Main()
			{
				string SQLConnectionString = "[Your_Connection_String]";
				// Create a SqlConnection from the provided connection string.
				using (SqlConnection connection = new SqlConnection(SQLConnectionString))
				{
					// Begin to formulate the command.
					SqlCommand command = new SqlCommand();
					command.Connection = connection;

					// Specify the query to be executed.
					command.CommandType = System.Data.CommandType.Text;
						command.CommandText =
						@"SELECT TOP 10
						CustomerID, NameStyle, Title, FirstName, LastName
						FROM SalesLT.Customer";

					// Open connection to database.
					connection.Open();

					// Read data from the query.
					SqlDataReader reader = command.ExecuteReader();
					while (reader.Read())
					{
						// Formatting will depend on the contents of the query.
						Console.WriteLine("Value: {0}, {1}, {2}, {3}, {4}",
							reader[0], reader[1], reader[2], reader[3], reader[4]);
					}
				}
				Console.WriteLine("Press any key to continue...");
				Console.ReadKey();
			}
		}
	}


## 步骤 3：查找数据库的连接字符串


1. 打开 [Azure 门户](https://manage.windowsazure.cn)。
2. 单击“浏览”>“SQL Database”>“Adventure Works”数据库 >“属性”>“显示数据库连接字符串”。


![门户](.\media\sql-database-connect-query\ConnectandQuery_portal.png)


在数据库连接字符串边栏选项卡上，你将看到 ADO.NET、ODBC、PHP 和 JDBC 的相应连接字符串。


## 步骤 4：替换为实际连接信息


\- 在粘贴的源代码中，将 *[Your_Connection_String]* 占位符替换为连接字符串，并确保将该字符串中的 *your_password_here* 替换为实际密码。


## 步骤 5：运行应用程序


1. 若要生成并运行你的应用程序，请单击“调试”>“开始调试”
2. 程序将在控制台窗口中显示查询结果。
 

<!---HONumber=66-->