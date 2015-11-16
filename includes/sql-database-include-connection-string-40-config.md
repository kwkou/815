
<!--
includes/sql-database-include-connection-string-40-config.md

Latest Freshness check:  2015-09-04 , GeneMi.

## Connection string
-->


### 用于连接字符串安全的示例配置文件


在 C# 代码中将连接字符串作为文本是不安全的。最好将连接字符串置于配置文件中。在这里，你可以随时编辑字符串，无需重新编译。

让我们假定你已编译的 C# 程序已命名为 **ConsoleApplication1.exe**，并且此 .exe 驻留在 **bin\\debug** 目录。

在此示例中，你的连接字符串的大部分存储在名为 **ConsoleApplication1.exe.config** 的配置文件中。此配置文件必须也驻留在 **bin\\debug**。

在下面配置文件的 XML 中你将看到一个名为 **ConnectionString4NoUserIDNoPassword** 的连接字符串。C# 代码查找此字符串。

你必须编辑占位符中的实际名称：

- {your\_serverName\_here}
- {your\_databaseName\_here}



		<?xml version="1.0" encoding="utf-8" ?>
		<configuration>
		    <startup> 
		        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
		    </startup>
		
		    <connectionStrings>
		        <clear />
		        <add name="ConnectionString4NoUserIDNoPassword"
		        providerName="System.Data.ProviderName"
		
		        connectionString=
				"Server=tcp:{your_serverName_here}.database.chinacloudapi.cn,1433;
				Database={your_databaseName_here};
				Connection Timeout=30;
				Encrypt=True;
				TrustServerCertificate=False;" />
		    </connectionStrings>
		</configuration>



在本演示中，我们选择省略两个参数：

- User ID={your\_userName\_here};
- Password={your\_password\_here};


你可以加入它们，但有时最好让你的程序通过用户的键盘输入获取这些值。视情况而定。



<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->

<!---HONumber=79-->