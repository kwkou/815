<properties
	pageTitle="使用 C# 查询连接到 SQL 数据库 | Azure"
	description="用 C# 编写用于查询和连接到 SQL 数据库的程序。有关 IP 地址、连接字符串、安全登录和免费 Visual Studio 的信息。"
	services="sql-database"
	keywords="c# 数据库查询, c# 查询, 连接到数据库, SQL C#"
	documentationCenter=""
	authors="MightyPen"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="04/25/2016"
	wacn.date="06/14/2016"/>


# 用 C&#x23; 编写用于查询和连接到 SQL 数据库的程序

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-connect-query/)
- [SSMS](/documentation/articles/sql-database-connect-query-ssms/)
- [Excel](/documentation/articles/sql-database-connect-excel/)

了解如何用 C# 编写用于查询和连接到云中的 Azure SQL 数据库。

本文为那些不熟悉 Azure SQL 数据库、C# 和 ADO.NET 的用户介绍每个步骤。对于熟悉 Microsoft SQL Server 和 C# 的用户，可以跳过一些步骤，重点关注那些特定于 SQL 数据库的步骤。


## 先决条件


若要运行 C# 查询代码示例，你必须拥有：


- Azure 帐户和订阅。你可以注册[试用版](/pricing/1rmb-trial)。


- Azure SQL 数据库服务的 **AdventureWorksLT** 演示数据库。
 - 在几分钟内[创建演示数据库](/documentation/articles/sql-database-get-started/)。


- Visual Studio 2013 Update 4（或更高版本）。Microsoft 现在免费提供 Visual Studio Community。
 - [Visual Studio Community，下载](http://www.visualstudio.com/products/visual-studio-community-vs)
 - [Visual Studio 的更多免费选项](http://www.visualstudio.com/products/free-developer-offers-vs.aspx)


<a name="InstallVSForFree" id="InstallVSForFree"></a>

&nbsp;

## 步骤 1：免费安装 Visual Studio Community


如果你需要安装 Visual Studio，可以：

- 使用浏览器导航到 Visual Studio 产品网页（该网页提供免费下载及其他选项），免费安装 Visual Studio Community。

## 步骤 2：在 Visual Studio 中创建新项目


在 Visual Studio 中，按照“C#”>“Windows”>“控制台应用程序”的初学者模板创建新项目。


1. 单击“文件”>“新建”>“项目”。将显示“***”对话框。

2. 在“已安装”下，扩展到 C# 和 Windows，以便“控制台应用程序”选项显示在中间窗格中。

	![“新建项目”对话框][30-VSNewProject]

2. 对于“名称”，请输入 **ConnectAndQuery\_Example**。单击“确定”。


## 步骤 3：添加程序集引用以进行配置处理


我们的 C# 示例使用 .NET Framework 程序集 **System.Configuration.dll**，因此让我们来添加对它的引用。


1. 在“解决方案资源管理器”窗格中，右键单击“引用”>“添加引用”。系统将显示“引用管理器”窗口。

2. 展开“程序集”>“Framework”。

3. 向下滚动，然后单击以突出显示 **System.Configuration**。确保已选中其复选框。

4. 单击“确定”。

5. 通过菜单“生成”>“生成解决方案”编译程序。


## 步骤 4：获取连接字符串

首次使用时，系统会将 Visual Studio 连接到 Azure SQL 数据库的 **AdventureWorksLT** 数据库。


[AZURE.INCLUDE [sql-database-include-connection-string-20-portalshots](../includes/sql-database-include-connection-string-20-portalshots.md)]


## 步骤 5：将连接字符串添加到 App.config 文件


1. 在 Visual Studio 中，从“解决方案资源管理器”窗格中打开 App.config 文件。

2. 如下面的示例 App.config 代码示例所示，添加 **&#x3c;configuration&#x3e; &#x3c;/configuration&#x3e;** 元素。
 - 将 {your\_placeholders} 替换为你自己的实际值：

			<?xml version="1.0" encoding="utf-8" ?>
			<configuration>
			    <startup>
			        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
			    </startup>
		
				<connectionStrings>
					<clear />
					<add name="ConnectionString4NoUserIDNoPassword"
					connectionString="Server=tcp:{your_serverName_here}.database.chinacloudapi.cn,1433; Database={your_databaseName_here}; Connection Timeout=30; Encrypt=True; TrustServerCertificate=False;"
					/>
				</connectionStrings>
			</configuration>


3. 保存 App.config 的更改。

4. 在“解决方案资源管理器”窗格中，右键单击 **App.config** 节点，然后单击“属性”。

5. 将“复制到输出目录”设置为“始终复制”。
 - 这将导致 App.config 文件的内容替换 &#x2a;.exe.config 文件的内容，位于 &#x2a;.exe 文件的生成目录。每次重新编译 &#x2a;.exe 时都会进行此替换。
 - 当系统运行我们的示例 C# 程序时，会读取 &#x2a;.exe.config 文件。

	![复制到输出目录 = 始终复制][50-VSCopyToOutputDirectoryProperty]


## 步骤 6：在示例 C# 代码中粘贴


1. 在 Visual Studio 中，使用“解决方案资源管理器”窗格打开你的 **Program.cs** 文件。

	![在示例 C# 查询代码中粘贴。][40-VSProgramCsOverlay]

2. 通过在下面的示例 C# 代码中粘贴，覆盖 Program.cs 中的所有起始代码。
 - 如果你想要更短的代码示例，可以将整个连接字符串作为文本分配给变量 **SQLConnectionString**。然后可以擦除 **GetConnectionStringFromExeConfig** 和 **GatherPasswordFromConsole** 这两个方法。

			using System;  // C#
			using G = System.Configuration;   // System.Configuration.dll
			using D = System.Data;            // System.Data.dll
			using C = System.Data.SqlClient;  // System.Data.dll
			using T = System.Text;
			
			namespace ConnectAndQuery_Example
			{
				class Program
				{
					static void Main()
					{
						string connectionString4NoUserIDNoPassword,
							password, userName, SQLConnectionString;
			
						// Get most of the connection string from ConnectAndQuery_Example.exe.config
						// file, in the same directory where ConnectAndQuery_Example.exe resides.
						connectionString4NoUserIDNoPassword = Program.GetConnectionStringFromExeConfig
							("ConnectionString4NoUserIDNoPassword");
						// Get the user name from keyboard input.
						Console.WriteLine("Enter your User ID, without the trailing @ and server name: ");
						userName = Console.ReadLine();
						// Get the password from keyboard input.
						password = Program.GatherPasswordFromConsole();
			
						SQLConnectionString = "Password=" + password + ';' +
							"User ID=" + userName + ";" + connectionString4NoUserIDNoPassword;
			
						// Create an SqlConnection from the provided connection string.
						using (C.SqlConnection connection = new C.SqlConnection(SQLConnectionString))
						{
							// Formulate the command.
							C.SqlCommand command = new C.SqlCommand();
							command.Connection = connection;
			
							// Specify the query to be executed.
							command.CommandType = D.CommandType.Text;
							command.CommandText = @"
								SELECT TOP 9 CustomerID, NameStyle, Title, FirstName, LastName
								FROM SalesLT.Customer;  -- In AdventureWorksLT database.
								";
							// Open a connection to database.
							connection.Open();
			
							// Read data returned for the query.
							C.SqlDataReader reader = command.ExecuteReader();
							while (reader.Read())
							{
								Console.WriteLine("Values:  {0}, {1}, {2}, {3}, {4}",
									reader[0], reader[1], reader[2], reader[3], reader[4]);
							}
						}
						Console.WriteLine("View the results here, then press any key to finish...");
						Console.ReadKey(true);
					}
					//----------------------------------------------------------------------------------
			
					static string GetConnectionStringFromExeConfig(string connectionStringNameInConfig)
					{
						G.ConnectionStringSettings connectionStringSettings =
							G.ConfigurationManager.ConnectionStrings[connectionStringNameInConfig];
			
						if (connectionStringSettings == null)
						{
							throw new ApplicationException(String.Format
								("Error. Connection string not found for name '{0}'.",
								connectionStringNameInConfig));
						}
							return connectionStringSettings.ConnectionString;
					}
			
					static string GatherPasswordFromConsole()
					{
						T.StringBuilder passwordBuilder = new T.StringBuilder(32);
						ConsoleKeyInfo key;
						Console.WriteLine("Enter your password: ");
						do
						{
							key = Console.ReadKey(true);
							if (key.Key != ConsoleKey.Backspace)
							{
								passwordBuilder.Append(key.KeyChar);
								Console.Write("*");
							}
							else  // Backspace char was entered.
							{
								// Retreat the cursor, overlay '*' with ' ', retreat again.
								Console.Write("\b \b");
								passwordBuilder.Length = passwordBuilder.Length - 1;
							}
						}
						while (key.Key != ConsoleKey.Enter); // Enter key will end the looping.
						Console.WriteLine(Environment.NewLine);
						return passwordBuilder.ToString();
					}
				}
			}



### 编译程序


1. 在 Visual Studio 中，通过单击菜单“生成”>“生成解决方案”来编译程序。


### 示例程序中的操作摘要


1. 从配置文件中读取大部分 SQL 连接字符串。

2. 从键盘收集用户名和密码，添加它们以完成连接字符串。

3. 使用该连接字符串和 ADO.NET 类连接到 SQL 数据库中的 **AdventureWorksLT** 演示数据库。

4. 发出 SQL **SELECT** 以从 **SalesLT** 表中读取。

5. 将返回的行打印到控制台。


我们努力让 C# 示例简短。但我们添加了代码来读取配置文件，这样才能服务于来自像你一样的客户的多个请求。我们同意，要生产高质量程序，应使用配置文件，而不是 .exe 中的硬编码文本。


> [AZURE.WARNING] 为了代码简短起见，我们选择在此教授示例中不包括用于异常处理的代码以及重试逻辑。但是你与云数据库进行交互的生产程序应包括这两者。
>
> [此处](/documentation/articles/sql-database-develop-csharp-retry-windows/)是具有重试逻辑的代码示例的链接。


## 步骤 7：在服务器防火墙中添加允许的 IP 地址范围


在将客户端计算机的 IP 地址添加到 SQL 数据库防火墙之前，客户端 C# 程序无法连接到 SQL 数据库。你的程序将失败，并显示简单的错误消息，指出必需的 IP 地址。


[AZURE.INCLUDE [sql-database-include-ip-address-22-v12portal](../includes/sql-database-include-ip-address-22-v12portal.md)]



## 步骤 8：运行程序


1. 在 Visual Studio 中，通过菜单“调试”>“启动调试”运行 C# 查询程序。将显示控制台窗口。

2. 按照指导输入用户名和密码。
 - 有几个连接工具需要将你的用户名附加到“@{your\_serverName\_here}”，但对于 ADO.NET 此后缀是可选的。你不必费神键入后缀。

3. 系统将显示数据行。


## 相关链接


- [SQL 数据库的客户端快速入门代码示例](/documentation/articles/sql-database-develop-quick-start-client-code-samples/)

- 如果你的客户端程序运行在 Azure VM 上，请参阅<br/>[适用于 ADO.NET 4.5 和 SQL 数据库 V12 的 1433 之外的端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)，了解 1433 以外的 TCP 端口。



<!-- Image references. -->

[20-OpenInVisualStudioButton]: ./media/sql-database-connect-query/connqry-free-vs-e.png

[30-VSNewProject]: ./media/sql-database-connect-query/connqry-vs-new-project-f.png

[40-VSProgramCsOverlay]: ./media/sql-database-connect-query/connqry-vs-program-cs-overlay-g.png

[50-VSCopyToOutputDirectoryProperty]: ./media/sql-database-connect-query/connqry-vs-appconfig-copytoputputdir-h.png

<!---HONumber=Mooncake_0530_2016-->