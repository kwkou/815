<properties 
	pageTitle="在 Windows上使用 Node.js 连接到 SQL 数据库" 
	description="演示了一个可以用来连接到 Azure SQL 数据库 的 NodeJS 代码示例。该示例在 Windows 客户端计算机上运行。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.date="07/30/2015" 
	wacn.date="09/15/2015"/>


# 在 Windows上使用 Node.js 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题演示了一个可以用来连接到 Azure SQL 数据库的 Node.js 代码示例。该 Node.js 程序在 Windows 客户端计算机上运行。若要管理连接，请使用 msnodesql 驱动程序。


## 要求


你的客户端开发计算机上必须存在以下软件项。


-  Node.js – [版本 0.8.9（32 位版本）](http://blog.nodejs.org/2012/09/11/node-v0-8-9-stable/)。滚动页面，然后单击 32 位 x86 Windows 安装程序（而不是 64 位 Windows x64 安装程序）对应的“下载”链接。
- [Python 2.7.6](https://www.python.org/download/releases/2.7.6/) - 适用于 x86 或 x64 的安装程序。 
- [Visual C++ 2010](https://app.vssps.visualstudio.com/profile/review?download=true&family=VisualStudioCExpress&release=VisualStudio2010&type=web&slcid=0x409&context=eyJwZSI6MSwicGMiOjEsImljIjoxLCJhbyI6MCwiYW0iOjEsIm9wIjpudWxsLCJhZCI6bnVsbCwiZmEiOjAsImF1IjpudWxsLCJjdiI6OTY4OTg2MzU1LCJmcyI6MCwic3UiOjAsImVyIjoxfQ2) - 可从 Microsoft 下载免费的 Express 版本。
- SQL Server Native Client 11.0 - 在 [SQL Server 2012 Feature Pack](https://www.microsoft.com/zh-CN/download/details.aspx?id=29065) 中作为 Microsoft SQL Server 2012 Native Client 提供。


### 安装所需的模块

满足这些要求后，请确保在 Node.js 版本 0.8.9 上操作。可以在命令行终端上使用以下命令来检查是否使用了此版本：node -v。<br>在 **cmd.exe** 命令行窗口中，导航到项目目录 - 例如 C:\\NodeJSSQLProject。按照显示的顺序输入以下命令。

	npm init
	npm install msnodesql

接下来，请导航到 node\_modules\\msnodesql 文件夹并运行 **msnodesql-0.2.1-v0.8-ia32** 可执行文件。遵循安装向导中的步骤操作，并在完成后单击“完成”。此时，应已安装 Node.js SQL Server 驱动程序。遵循后续步骤来获取连接字符串，然后，你应能从 Node.js 应用程序连接到 Azure SQL 数据库。

### 创建数据库并检索连接字符串
 
请参阅[入门主题](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库和检索连接字符串。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 连接到 SQL Database


- 将以下代码复制到项目目录中的某个 .js 文件。


		var http = require('http');
		var sql = require('msnodesql');
		var http = require('http');
		var fs = require('fs');
		var useTrustedConnection = false;
		var conn_str = "Driver={SQL Server Native Client 11.0};Server=tcp:yourserver.database.chinacloudapi.cn;" + 
		(useTrustedConnection == true ? "Trusted_Connection={Yes};" : "UID=yourusername;PWD=yourpassword;") + 
		"Database={AdventureWorks};"
		sql.open(conn_str, function (err, conn) {
		    if (err) {
		        console.log("Error opening the connection!");
		        return;
		    }
		    else
		        console.log("Successfuly connected");
		});	


- 现在，请通过发出以下命令运行该 .js 文件。


		node index.js


## 执行 SQL SELECT 语句


	var http = require('http');
	var sql = require('msnodesql');
	var http = require('http');
	var fs = require('fs');
	var useTrustedConnection = false;
	var conn_str = "Driver={SQL Server Native Client 11.0};Server=tcp:yourserver.database.chinacloudapi.cn;" + 
	(useTrustedConnection == true ? "Trusted_Connection={Yes};" : "UID=yourusername;PWD=yourpassword;") + 
	"Database={AdventureWorks};"
	sql.open(conn_str, function (err, conn) {
	    if (err) {
	        console.log("Error opening the connection!");
	        return;
	    }
	    else
	        console.log("Successfuly connected");
	
	
	    conn.queryRaw("SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;", function (err, results) {
	        if (err) {
	            console.log("Error running query1!");
	            return;
	        }
	        for (var i = 0; i < results.rows.length; i++) {
	            console.log(results.rows[i]);
	        }
	    });
	});


## 插入一行，传递参数，然后检索生成的主键


	var http = require('http');
	var sql = require('msnodesql');
	var http = require('http');
	var fs = require('fs');
	var useTrustedConnection = false;
	var conn_str = "Driver={SQL Server Native Client 11.0};Server=tcp:yourserver.database.chinacloudapi.cn;" + 
	(useTrustedConnection == true ? "Trusted_Connection={Yes};" : "UID=yourusername;PWD=yourpassword;") + 
	"Database={AdventureWorks};"
	sql.open(conn_str, function (err, conn) {
	    if (err) {
	        console.log("Error opening the connection!");
	        return;
	    }
	    else
	        console.log("Successfuly connected");
	
	
	    conn.queryRaw("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express', 'SQLEXPRESS', 0, 0, CURRENT_TIMESTAMP)", function (err, results) {
	        if (err) {
	            console.log("Error running query!");
	            return;
	        }
	        for (var i = 0; i < results.rows.length; i++) {
	            console.log("Product ID Inserted : "+results.rows[i]);
	        }
	    });
	});


## 事务


方法 **conn.beginTransactions** 在 Azure SQL 数据库中不起作用。请遵循代码示例在 SQL 数据库中执行事务。


	var http = require('http');
	var sql = require('msnodesql');
	var http = require('http');
	var fs = require('fs');
	var useTrustedConnection = false;
	var conn_str = "Driver={SQL Server Native Client 11.0};Server=tcp:yourserver.database.chinacloudapi.cn;" + 
	(useTrustedConnection == true ? "Trusted_Connection={Yes};" : "UID=yourusername;PWD=yourpassword;") + 
	"Database={AdventureWorks};"
	sql.open(conn_str, function (err, conn) {
	    if (err) {
	        console.log("Error opening the connection!");
	        return;
	    }
	    else
	        console.log("Successfuly connected");
	
	
	    conn.queryRaw("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New ', 'SQLEXPRESS New', 1, 1, CURRENT_TIMESTAMP)", function (err, results) {
	        if (err) {
	            console.log("Error running query!");
	            return;
	        }
	        for (var i = 0; i < results.rows.length; i++) {
	            console.log("Product ID Inserted : "+results.rows[i]);
	        }
	    });
	    
	    conn.queryRaw("ROLLBACK TRANSACTION; ", function (err, results) {
            	if (err) {
        		console.log("Rollback failed");
        		return;
        	}
    	    });
	});


## 存储过程


要正常运行此代码示例，必须事先准备或创建一个不输入任何参数的存储过程。可以使用 SQL Server Management Studio (SSMS.exe) 等工具创建存储过程。


	var http = require('http');
	var sql = require('msnodesql');
	var http = require('http');
	var fs = require('fs');
	var useTrustedConnection = false;
	var conn_str = "Driver={SQL Server Native Client 11.0};Server=tcp:yourserver.database.chinacloudapi.cn;" + 
	(useTrustedConnection == true ? "Trusted_Connection={Yes};" : "UID=yourusername;PWD=yourpassword;") + 
	"Database={AdventureWorks};"
	sql.open(conn_str, function (err, conn) {
	    if (err) {
	        console.log("Error opening the connection!");
	        return;
	    }
	    else
	        console.log("Successfuly connected");
		
	    conn.query("exec NameOfStoredProcedure", function (err, results) {
	    	if (err) {
			console.log("Error running query8!");
			return;
		}
	    });
	});

 

<!---HONumber=69-->