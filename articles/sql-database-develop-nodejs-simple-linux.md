<properties 
	pageTitle="在 Ubuntu Linux 上使用 NodeJS 和 Tedious 连接到 SQL 数据库" 
	description="演示了一个可以用来连接到 Azure SQL 数据库 的 NodeJS 代码示例。该示例使用 Tedious 驱动程序进行连接。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="NodeJS" 
	ms.topic="article" 
	ms.date="04/18/2015"
	wacn.date="05/25/2015"  
	ms.author="mebha"/>


# 在 Ubuntu Linux 上使用 NodeJS 和 Tedious 连接到 SQL 数据库


本主题介绍在 Unbutu Linux 上运行的 NodeJS 代码示例。该示例将使用 Tedious 驱动程序连接到 Azure SQL 数据库。


## 所需的软件项


打开终端并安装 **node** 和**npm**（如果尚未在计算机上安装）。


	sudo apt-get install node
	sudo apt-get install npm


使用 **node** 和 **npm** 配置计算机后，导航到你要在其中创建 nodejs 项目的目录，然后输入以下命令。


	sudo npm init
	sudp npm install tedious


**npm init** 将创建节点项目。若要在创建项目过程中保留默认值，请一直按 Enter，直到创建完项目。现在，项目目录中会出现 **package.json** 文件。


### 创建 AdventureWorks 数据库


本主题中的代码示例需要一个 **AdventureWorks** 测试数据库。如果你尚未创建一个这样的数据库，请参阅 [SQL 数据库 入门](sql-database-get-started)。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。 


## 连接到 SQL 数据库


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Windows Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
	// If no error, then good to proceed.
		console.log("Connected");
	});




## 执行 SQL SELECT



	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// When you connect to Azure SQL 数据库, you need these next options.
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


## 插入一行，应用参数，然后检索生成的主键


本部分中的代码示例将向 SQL INSERT 语句应用参数。程序将检索生成的主键值。


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Azure SQL 数据库, you need these next options.
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


## 事务


<!--
TODO, ERRORS FOR MEET BHAGDEV TO FIX:
.
I do not see any Transaction related code in this section? Must fix!
.
Same problem in the next section about Stored procedures. Perhaps you can call sys.sp_helptext?
-->


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Windows Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement2();
	});
	
	var Request = require('tedious').Request;
	var TYPES = require('tedious').TYPES;
	
	function executeStatement2() {
	//TODO
	}


## 存储过程


	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.chinacloudapi.cn',
		// If you are on Windows Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement3();
	});
	
	var Request = require('tedious').Request;
	var TYPES = require('tedious').TYPES;
	
	function executeStatement3() {
	//TODO
	}

<!--HONumber=55-->