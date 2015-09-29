<properties 
	pageTitle="从 .NET (C#) 使用 SQL 数据库" 
	description="使用本快速入门教程中的示例代码可以生成一个包含 C# 代码并由云中强大的 Azure SQL 数据库关系数据库支持的现代应用程序。"
	services="sql-database" 
	documentationCenter="" 
	authors="tobbox" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="07/16/2015" 
	wacn.date="09/15/2015"/>


# 从 .NET (C#) 使用 SQL 数据库 


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


## 要求

### .NET framework

已在 Windows 上预装了 .NET Framework。对于 Linux 和 Mac OS X，可从 [Mono 项目](http://www.mono-project.com/)下载 .NET Framework。

### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库并检索连接字符串。

## 连接到 SQL 数据库

[System.Data.SqlClient.SqlConnection 类](https://msdn.microsoft.com/zh-CN/library/system.data.sqlclient.sqlconnection.aspx)用于连接到 SQL 数据库。

```
using System.Data.SqlClient;

class Sample
{
  static void Main()
  {
	  using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
	  {
		  conn.Open();	
	  }
  }
}	
```

## 执行查询并检索结果集 

[System.Data.SqlClient.SqlCommand](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlcommand.aspx) 和 [SqlDataReader](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqldatareader.aspx) 类可用于针对 SQL 数据库 从查询中检索结果集。请注意，System.Data.SqlClient 也支持将数据检索到脱机 [System.Data.DataSet](https://msdn.microsoft.com/zh-cn/library/system.data.dataset.aspx) 中。   

```
using System;
using System.Data.SqlClient;

class Sample
{
	static void Main()
	{
	  using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
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

```


## 插入一行，传递参数，然后检索生成的主键值 

在 SQL 数据库 中，可以使用 [IDENTITY](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENECE](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。在本示例中，你将了解如何执行 [insert 语句](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx)，安全传递用于防止 [SQL 注入](https://msdn.microsoft.com/magazine/cc163917.aspx)的参数，然后检索自动生成的主键值。  

可以使用 [System.Data.SqlClient.SqlCommand](https://msdn.microsoft.com/zh-CN/library/system.data.sqlclient.sqlcommand.aspx) 类中的 [ExecuteScalar](https://msdn.microsoft.com/zh-CN/library/system.data.sqlclient.sqlcommand.executescalar.aspx) 方法执行语句，然后检索此语句返回的第一列和行。可以使用 INSERT 语句的 [OUTPUT](https://msdn.microsoft.com/zh-CN/library/ms177564.aspx) 子句将插入的值作为结果集返回到调用应用程序。请注意，[UPDATE](https://msdn.microsoft.com/zh-CN/library/ms177523.aspx)、[DELETE](https://msdn.microsoft.com/zh-CN/library/ms189835.aspx) 和 [MERGE](https://msdn.microsoft.com/zh-CN/library/bb510625.aspx) 语句也支持 OUTPUT。如果插入了多个行，你应使用 [ExecuteReader](https://msdn.microsoft.com/zh-CN/library/system.data.sqlclient.sqlcommand.executereader.aspx) 方法来检索所有行的插入值。

```
class Sample
{
    static void Main()
    {
		using(var conn = new SqlConnection("Server=tcp:yourserver.database.chinacloudapi.cn,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
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
```

 

<!---HONumber=69-->