<properties 
	pageTitle="在 Windows 上使用 NodeJS 连接到 SQL Database" 
	description="演示了一个可以用来连接到 Azure SQL Database 的 NodeJS 代码示例。该示例在 Windows 客户端计算机上运行。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="nodejs" 
	ms.topic="article" 
	ms.date="04/18/2015"
	wacn.date="05/25/2015"  
	ms.author="mebha"/>


# 在 Windows 上使用 NodeJS 连接到 SQL Database


<!--
2015-04-18
Original author is Meet Bhagdev (mebha; or on github 'meet-bhagdev'). GeneMi did an edit pass, and did the first publish.
-->


本主题演示了一个可以用来连接到 Azure SQL Database 的 NodeJS 代码示例。该 NodeJS 程序在 Windows 客户端计算机上运行。若要管理连接，请使用 msnodesql 驱动程序。


## 要求


你的客户端开发计算机上必须存在以下软件项。


<!--
TODO, FIX ERROR?
Is the Python 2.7.6 download truly necessary for the NodeJS sample to work, or was there a copy-paste error? If Python not needed in this NodeJS topic, we must remove that bullet line and its child bullet line.
.
2015-04-18 12:33pm, I (GeneMi) am taking an educated chance and am removing the Python download lines. Should they be re-added to the live text?
.
- [Python 2.7.6](https://www.python.org/download/releases/2.7.6), the installer for either x86 or x64. 
 - The x64 version is probably preferable. You want the "Installer" link, not the "program database" link.
-->


-  Node.js -[版本 0.8.9（32 位版本）](http://blog.nodejs.org/2012/09/11/node-v0-8-9-stable)。滚动页面，然后单击 32 位 x86 Windows 安装程序（而不是 64 位 x86 Windows 安装程序）对应的"下载"链接。
- [Python 2.7.6](https://www.python.org/download/releases/2.7.6) - 适用于 x86 或 x64 的安装程序。
- [Visual C++ 2010](https://app.vssps.visualstudio.com/profile/review?download=true&family=VisualStudioCExpress&release=VisualStudio2010&type=web&slcid=0x409&context=eyJwZSI6MSwicGMiOjEsImljIjoxLCJhbyI6MCwiYW0iOjEsIm9wIjpudWxsLCJhZCI6bnVsbCwiZmEiOjAsImF1IjpudWxsLCJjdiI6OTY4OTg2MzU1LCJmcyI6MCwic3UiOjAsImVyIjoxfQ2) - 可从 Microsoft 下载免费的 Express 版本。
- SQL Server Native Client 11.0 - 在 [SQL Server 2012 Feature Pack](http://www.microsoft.com/zh-CN/download/details.aspx?id=29065) 中作为 Microsoft SQL Server 2012 Native Client 提供。


### 安装所需的模块


在 **cmd.exe** 命令行窗口中，导航到 msnodesql 所在的目录。按照显示的顺序输入以下命令。


	npm install msnodesql
	npm install -g node-gyp


现在，你已安装了 gyp，请导航到目录 *YourProjectDirectory* 并选择 **node_modules\msnodesql**。然后，在 **cmd.exe** 窗口中发出以下命令。


	node-gyp configure 
	node-gyp build


接下来，导航到目录 **build\release**。复制 **sqlserver.node** 文件并将其粘贴到 **msnodesql\lib** 目录。根据需要替换旧文件。


### 创建数据库并检索连接字符串
 
有关如何创建示例数据库并检索连接字符串，请参阅[入门主题](sql-database-get-started)。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。 


## 连接到 SQL Database


<!--
TODO, to Meet: In this code sample, the connection string lines are excessively long, and will not display well in a web browser. Please split them into 2 or 3 shorter lines.
-->


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


方法 **conn.beginTransactions** 在 Azure SQL Database 中不起作用。请遵循代码示例在 SQL 数据库中执行事务。


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

<!--HONumber=55-->