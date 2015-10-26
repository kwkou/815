<properties
	pageTitle="使用 C# 查询 SQL 数据库 | Windows Azure"
	description="有关使用 IP 地址、连接字符串、安全登录配置文件和免费 Visual Studio 让 C# 程序能够使用 ADO.NET 连接到云中的 Azure SQL 数据库的详细信息。"
	services="sql-database"
	documentationCenter=""
	authors="MightyPen"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="09/09/2015"
	wacn.date="10/17/2015"/>


# 使用 C&#x23; 连接和查询 SQL 数据库


你想要编写一个 C# 程序，该程序使用 ADO.NET 连接到云中的 Azure SQL 数据库。

本主题为那些不熟悉 Azure SQL 数据库和 C# 的用户介绍每个步骤。对于熟悉 Microsoft SQL Server 和 C# 的用户，可以跳过一些步骤，重点关注那些特定于 SQL 数据库的步骤。


## 先决条件


若要运行 C# 代码示例，你必须拥有：


- Azure 帐户和订阅。你可以注册[免费试用版](http://wacn-ppe.chinacloudsites.cn/zh-cn/pricing/1rmb-trial/)。


- Azure SQL 数据库服务的 **AdventureWorksLT** 演示数据库。
 - 在几分钟内[创建演示数据库](/documentation/articles/sql-database-get-started)。


- Visual Studio 2013 Update 4（或更高版本）。Microsoft 现在*免费*提供 Visual Studio Community。
 - [Visual Studio Community，下载](http://www.visualstudio.com/products/visual-studio-community-vs)
 - [Visual Studio 的更多免费选项](http://www.visualstudio.com/products/free-developer-offers-vs.aspx)
 - 或者，利用本主题中介绍 [Azure 预览门户](http://www.windowsazure.cn/)的后续[步骤](#InstallVSForFree)，指导你安装 Visual Studio。


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
2. 单击“浏览”>“SQL 数据库”>“Adventure Works”数据库 >“属性”>“显示数据库连接字符串”。


![门户](./media/sql-database-connect-query/ConnectandQuery_portal.png)


在数据库连接字符串边栏选项卡上，你可以看到 ADO.NET、ODBC、PHP 和 JDBC 的相应连接字符串。


## 步骤 4：替换为实际连接信息


- 在粘贴的源代码中，将 *[Your\_Connection\_String]* 占位符替换为连接字符串，并确保将该字符串中的 *your\_password\_here* 替换为实际密码。


## 步骤 5：运行应用程序


1. 若要生成并运行你的应用程序，请单击“调试”>“开始调试”
2. 程序将在控制台窗口中显示查询结果。
 

<!---HONumber=69-->