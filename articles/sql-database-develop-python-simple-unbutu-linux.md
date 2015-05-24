<properties 
	pageTitle="在 Ubuntu 上使用 Python 和 pymssql 连接到 SQL Database" 
	description="演示了一个可以用来连接到 Azure SQL Database 的 Python 代码示例。该示例在 Unbutu Linux 客户端计算机上运行。"
	services="sql-database" 
	documentationCenter="" 
	authors="MightyPen" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="python" 
	ms.topic="article" 
	ms.date="04/18/2015"
	wacn.date="05/25/2015" 
	ms.author="genemi"/>


# 在 Unbutu Linux 上使用 Python 连接到 SQL Database


<!--
Original author of content is Meet Bhagdev. GeneMi edited and first published.
-->


本主题演示了一个在 Unbutu Linux 客户端计算机上运行的，用于连接到 Azure SQL Database 数据库的 Python 代码示例。


## 要求


- [Python 2.7.6](https://www.python.org/download/releases/2.7.6/)。


### 安装所需的模块


打开终端并导航到你要在其中创建 python 脚本的目录。输入以下命令以安装 **FreeTDS** 和 **pymssql**。

	sudo apt-get --assume-yes update
	sudo apt-get --assume-yes install freetds-dev freetds-bin
	sudo apt-get --assume-yes install python-dev python-pip
	sudo pip install pymssql


### 创建数据库并检索连接字符串


有关如何创建示例数据库并检索连接字符串，请参阅[入门主题](sql-database-get-started)。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。 


## 连接到 SQL Database


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')


## 执行 SQL SELECT 语句


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute('SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;')
	row = cursor.fetchone()
	while row:
	    print str(row[0]) + " " + str(row[1]) + " " + str(row[2]) 	
	    row = cursor.fetchone()


## 插入一行，传递参数，然后检索生成的主键


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express', 'SQLEXPRESS', 0, 0, CURRENT_TIMESTAMP)")
	row = cursor.fetchone()
	while row:
	    print "Inserted Product ID : " +str(row[0])
	    row = cursor.fetchone()


<!--
TODO: The code sample must leave the schema and data in the same state as they were before the sample started. The above INSERT has no matching SQL DELETE.
-->


## 事务


此代码示例演示了你可以在其中执行以下操作的事务的用法：


- 开始一个事务。
- 插入数据行。
- 回滚事务以撤消插入。


		import pymssql
		conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
		cursor = conn.cursor()
		cursor.execute("BEGIN TRANSACTION")
		cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New', 'SQLEXPRESS New', 0, 0, CURRENT_TIMESTAMP)")
		cnxn.rollback()


## 存储过程


<!--
TODO, FIX PROBLEM:
.
This program is not re-runnable. The program needs to clean up after itself, and leave the schema and data in the same state it was before the program started.
.
Can you DROP the stored procedure after it runs?
-->


	with pymssql.connect("yourserver", "yourusername", "yourpassword", "yourdatabase") as conn:
    with conn.cursor(as_dict=True) as cursor:
        cursor.execute("""
        CREATE PROCEDURE FindProductNameName
            @name VARCHAR(100)
        AS BEGIN
            SELECT * FROM Product WHERE Name = @name
        END
        """)
        cursor.callproc('FindPerson', ('Bike'))
        for row in cursor:
            print str(row[0]) + " " + str(row[1]) + " " + str(row[2])

<!--HONumber=55-->