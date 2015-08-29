<properties 
	pageTitle="在 Windows上使用 Python 连接到 SQL Database" 
	description="提供了一个可以用于从 Windows 客户端连接到 Azure SQL Database 的 Python 代码示例。该示例使用 pyodbc 驱动程序。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	wacn.date="" ms.service="sql-database" ms.date="04/18/2015" />


# 在 Windows上使用 Python 连接到 SQL Database


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


本主题提供以 Python 编写的代码示例。该示例在 Windows 计算机上运行。该示例将使用 **pyodbc** 驱动程序连接到 Azure SQL Database。


## 要求


- [Python 2.7.6](https://www.python.org/download/releases/2.7.6/)


### 安装所需的模块


安装 [pymssql](http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql)。

确保选择正确的 whl 文件。

例如，如果在 64 位计算机上使用 Python 2.7，请选择：pymssql-2.1.1-cp27-none-win_amd64.whl。下载 .whl 文件后，请将它放入 C:/Python27 文件夹。

现在，请从命令行使用 pip 安装 pymssql 驱动程序。使用 cd 命令切换到 C:/Python27 并运行以下命令
	
	pip install pymssql-2.1.1-cp27-none-win_amd64.whl

可在[此处](http://stackoverflow.com/questions/4750806/how-to-install-pip-on-windows)找到有关使用 pip 的说明


### 创建数据库并检索连接字符串


请参阅[入门主题](/documentation/articles/sql-database-get-started)，以了解如何创建示例数据库和检索连接字符串。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 连接到 SQL Database


[Pymssql.connect](http://pymssql.org/en/latest/ref/pymssql.html) 函数用于连接到 SQL Database。

	import pymssql
	conn = pymssql.connect(server='yourserver.database.windows.net', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')



<!--
TODO: Again, Does Python allow you to somehow split a very long line of code into multiple lines, for better display?
-->

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

在 SQL Database 中，可以使用 [IDENTITY](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENCE](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键](https://msdn.microsoft.com/library/ms179610.aspx)值。


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

<!---HONumber=66-->