<properties 
	pageTitle="在 Mac OS 上使用 Python 连接到 SQL Database" 
	description="演示了一个可用于从 Mac 连接到 Azure SQL Database 的 Python 代码示例。该示例使用 pymssql 驱动程序。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" ms.date="07/16/2015" wacn.date=""/>


# 在 Mac OS 上使用 Python 连接到 SQL Database


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题提供以 Python 编写的代码示例。该示例在 Mac 计算机上运行。该示例将使用 **pymssql** 驱动程序连接到 Azure SQL Database。


## 要求


- [Python 2.7.6](https://www.python.org/download/releases/2.7.6/)。
- [FreeTDS](https://github.com/brianb/FreeTDS)
- [Pymssql](https://github.com/pymssql/pymssql)

### 安装所需的模块


打开终端并安装

**1) Homebrew**：从你的终端运行以下命令。这将在你的计算机上下载 Homebrew 包管理器。

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

**2) FreeTDS**：从你的终端运行以下命令。这将在你的计算机上下载 FreeTDS。要使 pymmsql 正常工作，必须安装 FreeTDS。

    brew install FreeTDS
  
**3) Pymmsql**：从你的终端运行以下命令。这将在你的计算机上安装 pymmsql。

    sudo -H pip install pymssql

### 创建数据库并检索连接字符串


请参阅[入门页](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库并检索连接字符串。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 连接到 SQL Database


[Pymssql.connect](http://pymssql.org/en/latest/ref/pymssql.html) 函数用于连接到 SQL Database。

	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')


## 执行 SQL SELECT 语句

[Cursor.execute](http://pymssql.org/en/latest/ref/pymssql.html#pymssql.Cursor.execute) 函数可用于针对 SQL Database 从查询中检索结果集。此函数实际上可接受任何查询，并返回可使用 [cursor.fetchone()](http://pymssql.org/en/latest/ref/pymssql.html#pymssql.Cursor.fetchone) 循环访问的结果集。


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute('SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;')
	row = cursor.fetchone()
	while row:
	    print str(row[0]) + " " + str(row[1]) + " " + str(row[2]) 	
	    row = cursor.fetchone()


## 插入一行，传递参数，然后检索生成的主键

在 SQL Database 中，可以使用 [IDENTITY](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENCE](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express', 'SQLEXPRESS', 0, 0, CURRENT_TIMESTAMP)")
	row = cursor.fetchone()
	while row:
	    print "Inserted Product ID : " +str(row[0])
	    row = cursor.fetchone()


## 事务


此代码示例演示了你可以在其中执行以下操作的事务的用法：


\-开始一个事务

\-插入一行数据

\-回滚事务以撤消插入


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute("BEGIN TRANSACTION")
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New', 'SQLEXPRESS New', 0, 0, CURRENT_TIMESTAMP)")
	cnxn.rollback()

 

<!---HONumber=66-->