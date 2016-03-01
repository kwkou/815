<properties
	pageTitle="在 Mac OS X 上配合 Tedious 使用 Node.js 连接到 SQL 数据库"
	description="演示了一个可以用来连接到 Azure SQL 数据库的 Node.js 代码示例。该示例使用 Tedious 驱动程序进行连接。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="12/17/2015"
        wacn.date="01/15/2016"/>


# 在 Mac OS X 上配合 Tedious 使用 Node.js 连接到 SQL 数据库


> [AZURE.SELECTOR]
- [Node.js](/documentation/articles/sql-database-develop-nodejs-simple-mac)
- [Python](/documentation/articles/sql-database-develop-python-simple-mac-osx)
- [Ruby](/documentation/articles/sql-database-develop-ruby-simple-mac-osx)


本主题提供可在 Mac OS X 上运行的 Node.js 代码示例。该示例使用 Tedious 驱动程序连接到 Azure SQL 数据库。


## 先决条件


安装 **node**（除非已在计算机上安装）。


若要在 OSX 10.10 Yosemite 上安装 node.js，你可以下载预编译的二进制包，这样便可以顺利轻松地完成安装。[访问 nodejs.org](http://nodejs.org)，单击“安装”按钮以下载最新的包。

遵循可以安装 **node** 和 **npm** 的安装向导，通过 .dmg 安装该包。npm 是“节点包管理器”，可帮助安装 node.js 的其他包。


将计算机配置为使用 **node** 和 **npm** 之后，请导航到要在其中创建 Node.js 项目的目录，然后输入以下命令。


	npm init
	npm install tedious


**npm init** 将创建节点项目。若要在项目创建期间保留默认值，请一直按 Enter，直到创建了项目。现在，项目目录中会出现 **package.json** 文件。

### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 1：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 2：连接

[new Connection](http://pekim.github.io/tedious/api-connection.html) 函数用于连接到 SQL 数据库。

	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Microsoft Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
	// If no error, then good to proceed.
		console.log("Connected");
	});


## 步骤 3：执行查询


所有 SQL 语句都是使用 [new Request()](http://pekim.github.io/tedious/api-request.html) 函数执行的。如果语句返回了行（例如 SELECT 语句），则你可以使用 [request.on()](http://pekim.github.io/tedious/api-request.html) 函数检索这些行。如果未返回行，[request.on()](http://pekim.github.io/tedious/api-request.html) 函数将返回空列表。


	var Connection = require('tedious').Connection;
	var Request = require('tedious').Request;
	var TYPES = require('tedious').TYPES;
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


## 步骤 4：插入行

在本示例中，你将了解如何安全地执行 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) 语句，传递参数以保护应用程序免遭 [SQL 注入](https://technet.microsoft.com/zh-cn/library/ms161953(v=sql.105).aspx) 漏洞的危害，然后检索自动生成的[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。


	var Connection = require('tedious').Connection;
	var Request = require('tedious').Request
	var TYPES = require('tedious').TYPES;
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

<!---HONumber=Mooncake_0104_2016-->
