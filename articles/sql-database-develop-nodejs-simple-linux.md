<properties
	pageTitle="在 Ubuntu Linux 上配合 Tedious 使用 Node.js 连接到 SQL 数据库"
	description="演示了一个可以用来连接到 Azure SQL 数据库的 Node.js 代码示例。该示例使用 Tedious 驱动程序进行连接。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="03/30/2016"
	wacn.date="05/16/2016"/>


# 在 Ubuntu Linux 上配合 Tedious 使用 Node.js 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题演示一个在 Ubuntu Linux 上运行的 Node.js 代码示例。该示例将使用 Tedious 驱动程序连接到 Azure SQL 数据库。

## 步骤 1：配置开发环境

[使用 Tedious Node.js Driver for SQL Server 的先决条件](https://msdn.microsoft.com/zh-cn/library/mt652094.aspx#Ubuntu-Linux)

## 步骤 2：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 3：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 4：连接

[new Connection](http://pekim.github.io/tedious/api-connection.html) 函数用于连接到 SQL 数据库。

	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
	// If no error, then good to proceed.
		console.log("Connected");
	});


## 步骤 5：执行查询


所有 SQL 语句都是使用 [new Request()](http://pekim.github.io/tedious/api-request.html) 函数执行的。如果语句返回了行（例如 SELECT 语句），则你可以使用 [request.on()](http://pekim.github.io/tedious/api-request.html) 函数检索这些行。如果未返回行，[request.on()](http://pekim.github.io/tedious/api-request.html) 函数将返回空列表。


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// When you connect to Azure SQL Database, you need these next options.
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement();
	});

	var Request = require('tedious').Request;
	var TYPES = require('tedious').TYPES;

	function executeStatement() {
		request = new Request("SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;", function(err) {
	  	if (err) {
	   		console.log(err);}
		});
		var result = "";
		request.on('row', function(columns) {
		    columns.forEach(function(column) {
		      if (column.value === null) {
		        console.log('NULL');
		      } else {
		        result+= column.value + " ";
		      }
		    });
		    console.log(result);
		    result ="";
		});

		request.on('done', function(rowCount, more) {
		console.log(rowCount + ' rows returned');
		});
		connection.execSql(request);
	}


## 步骤 6：插入行

在本示例中，你将了解如何安全地执行 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) 语句，传递参数以保护应用程序免遭 [SQL 注入](https://technet.microsoft.com/zh-cn/library/ms161953(v=sql.105).aspx) 漏洞的危害，然后检索自动生成的[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Azure SQL Database, you need these next options.
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement1();
	});

	var Request = require('tedious').Request
	var TYPES = require('tedious').TYPES;

	function executeStatement1() {
		request = new Request("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES (@Name, @Number, @Cost, @Price, CURRENT_TIMESTAMP);", function(err) {
		 if (err) {
		 	console.log(err);}
		});
		request.addParameter('Name', TYPES.NVarChar,'SQL Server Express 2014');
		request.addParameter('Number', TYPES.NVarChar , 'SQLEXPRESS2014');
		request.addParameter('Cost', TYPES.Int, 11);
		request.addParameter('Price', TYPES.Int,11);
		request.on('row', function(columns) {
		    columns.forEach(function(column) {
		      if (column.value === null) {
		        console.log('NULL');
		      } else {
		        console.log("Product id of inserted item is " + column.value);
		      }
		    });
		});		
		connection.execSql(request);
	}


## 后续步骤

有关详细信息，请参阅 [Node.js 开发人员中心](/develop/nodejs)。

<!---HONumber=Mooncake_0509_2016-->
