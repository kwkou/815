<properties 
	pageTitle="在 Windows 上使用 Python 连接到 SQL Database" 
	description="演示了一个可用于从 Windows 客户端连接到 Azure SQL Database 的 Python 代码示例。该示例使用了 pyodbc 驱动程序。"
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


# 在 Windows 上使用 Python 连接到 SQL Database


<!--
2015-04-18
Original content written by Meet Bhagdev, then edited by GeneMi.
-->


本主题提供以 Python 编写的代码示例。该示例在 Windows 计算机上运行。该示例将使用 **pyodbc** 驱动程序连接到 Azure SQL Database。


## 要求


- [Python 2.7.6](https://www.python.org/download/releases/2.7.6/)


### 安装所需的模块


导航到目录 **C:\Python27\Scripts**，然后运行以下命令。


	pip install --allow-external pyodbc --allow-unverified pyodbc pyodbc


### 创建数据库并检索连接字符串


有关如何创建示例数据库并检索连接字符串，请参阅[入门主题](sql-database-get-started)。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。 


## 连接到 SQL Database


	import pyodbc
	cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=tcp:yourservername.database.chinacloudapi.cn;DATABASE=AdventureWorks;UID=yourusername;PWD=yourpassword')
	cursor = cnxn.cursor())


<!--
TODO: Again, Does Python allow you to somehow split a very long line of code into multiple lines, for better display?
-->


## 执行 SQL SELECT


	import pyodbc
	cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=tcp:yourserver.database.chinacloudapi.cn;DATABASE=AdventureWorks;UID=yourusername;PWD=yourpassword')
	cursor = cnxn.cursor()
	cursor.execute('SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;')
	row = cursor.fetchone()
	while row:
	    print str(row[0]) + " " + str(row[1]) + " " + str(row[2]) 	
	    row = cursor.fetchone()


## 插入一行，传递参数，然后检索生成的主键


	import pyodbc
	cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=tcp:yourserver.database.chinacloudapi.cn;DATABASE=AdventureWorks;UID=yourusername;PWD=yourpassword')
	cursor = cnxn.cursor()
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express', 'SQLEXPRESS', 0, 0, CURRENT_TIMESTAMP)")
	row = cursor.fetchone()
	while row:
	    print "Inserted Product ID : " +str(row[0])
	    row = cursor.fetchone()


## 事务


	import pyodbc
	cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=tcp:yourserver.database.chinacloudapi.cn;DATABASE=AdventureWorks;UID=yourusername;PWD=yourpassword')
	cursor = cnxn.cursor()
	cursor.execute("BEGIN TRANSACTION")
	cursor.execute("DELETE FROM test WHERE value = 1;")
	cnxn.rollback()


<!--
TODO: Hmm, could we just as easily issue another cursor.execute('ROLLBACK TRNASACTION;')?
If so, perhaps we should at least include a sentence explaining that the option is viable?
-->


## 存储过程


我们将使用 **pyodbc** 驱动程序连接到 SQL Database。从 2015 年 4 月开始，此驱动程序将附带限制，它不再支持存储过程中的输出参数。因此，我们执行的存储过程将返回行结果集形式的信息。在该存储过程的 Transact-SQL 源代码中，靠近结尾处有一个用于生成和发出结果集的 SQL SELECT 语句。



<!--
TODO: I commented out these next sentences because they seem false. For example, I would expect that the Python program could issue a Transact-SQL string for a CREATE PROCEDURE statement, just as the Python program can issue an INSERT statement. Right?
.
Additionally you will have to use a database management tool such as SSMS to create your stored procedure. There is no way to create a stored procedure using pyodbc.
-->


<!--
TODO: Does AdventureWorks db have any stored procedure that returns a results set?
Or can we use a regular system stored procedure that is a native part of SQL Database, maybe like sys.sp_helptext !
-->


	import pyodbc
	cnxn = pyodbc.connect('DRIVER={SQL Server};SERVER=tcp:yourserver.database.chinacloudapi.cn;DATABASE=AdventureWorks;UID=yourusername;PWD=yourpassword')
	cursor = cnxn.cursor()
	cursor.execute("execute sys.sp_helptext 'SalesLT.vGetAllCategories';")

<!--HONumber=55-->