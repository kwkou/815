<properties
	pageTitle="使用 .NET (C#) 连接到 SQL 数据库"
	description="使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的现代应用程序。"
	services="sql-database"
	documentationCenter=""
	authors="tobbox"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="12/17/2015"
        wacn.date="01/15/2016"/>


# 从 .NET (C#) 使用 SQL 数据库


> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-develop-dotnet-simple)
- [PHP](/documentation/articles/sql-database-develop-php-simple-windows)
- [Python](/documentation/articles/sql-database-develop-python-simple-windows)
- [Ruby](/documentation/articles/sql-database-develop-ruby-simple-windows)
- [Java](/documentation/articles/sql-database-develop-java-simple-windows)
- [Node.js](/documentation/articles/sql-database-develop-nodejs-simple-windows)


## 先决条件

### .NET framework

已在 Windows 上预装了 .NET Framework。对于 Linux 和 Mac OS X，可从 [Mono 项目](http://www.mono-project.com)下载 .NET Framework。

### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 1：获取连接字符串

[AZURE.INCLUDE [sql-database-include-connection-string-dotnet-20-portalshots](../includes/sql-database-include-connection-string-dotnet-20-portalshots.md)]

## 步骤 2：连接

[System.Data.SqlClient.SqlConnection 类](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.aspx)用于连接到 SQL 数据库。


	using System.Data.SqlClient;
	
	class Sample
	{
	  static void Main()
	  {
		  using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
		  {
			  conn.Open();
		  }
	  }
	}


## 步骤 3：执行查询

[System.Data.SqlClient.SqlCommand](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.aspx) 和 [SqlDataReader](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqldatareader.aspx) 类可用于针对 SQL 数据库从查询中检索结果集。请注意，System.Data.SqlClient 也支持将数据检索到脱机 [System.Data.DataSet](https://msdn.microsoft.com/zh-cn/library/system.data.dataset.aspx) 中。

	
	using System;
	using System.Data.SqlClient;
	
	class Sample
	{
		static void Main()
		{
		  using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
			{
				var cmd = conn.CreateCommand();
				cmd.CommandText = @"
						SELECT
							c.CustomerID
							,c.CompanyName
							,COUNT(soh.SalesOrderID) AS OrderCount
						FROM SalesLT.Customer AS c
						LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID
						GROUP BY c.CustomerID, c.CompanyName
						ORDER BY OrderCount DESC;";
	
				conn.Open();
	
				using(var reader = cmd.ExecuteReader())
				{
					while(reader.Read())
					{
						Console.WriteLine("ID: {0} Name: {1} Order Count: {2}", reader.GetInt32(0), reader.GetString(1), reader.GetInt32(2));
					}
				}					
			}
		}
	}


## 步骤 4：插入行

在本示例中，你将了解如何安全地执行 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) 语句，传递参数以保护应用程序免遭 [SQL 注入](https://technet.microsoft.com/zh-cn/library/ms161953(v=sql.105).aspx) 漏洞的危害，然后检索自动生成的[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。

	
	using System;
	using System.Data.SqlClient;
	
	class Sample
	{
	    static void Main()
	    {
			using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
	        {
	            var cmd = conn.CreateCommand();
	            cmd.CommandText = @"
	                INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate)
	                OUTPUT INSERTED.ProductID
	                VALUES (@Name, @Number, @Cost, @Price, CURRENT_TIMESTAMP)";
	
	            cmd.Parameters.AddWithValue("@Name", "SQL Server Express");
	            cmd.Parameters.AddWithValue("@Number", "SQLEXPRESS1");
	            cmd.Parameters.AddWithValue("@Cost", 0);
	            cmd.Parameters.AddWithValue("@Price", 0);
	
	            conn.Open();
	
	            int insertedProductId = (int)cmd.ExecuteScalar();
	
	            Console.WriteLine("Product ID {0} inserted.", insertedProductId);
	        }
	    }
	}


<!---HONumber=Mooncake_0104_2016-->
