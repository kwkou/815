<properties
	pageTitle="在 Windows上使用 Python 连接到 SQL 数据库"
	description="提供了一个可以用于从 Windows 客户端连接到 Azure SQL 数据库的 Python 代码示例。该示例使用 pymssql 驱动程序。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="03/14/2016"
	wacn.date="03/24/2016"/>


# 在 Windows上使用 Python 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)] 



本主题提供以 Python 编写的代码示例。该示例在 Windows 计算机上运行。该示例将使用 **pymssql** 驱动程序连接到 Azure SQL 数据库。


## 先决条件


- [Python 2.7.6](https://www.python.org/download/releases/2.7.6)


### 安装所需的模块


安装 [pymssql](http://www.lfd.uci.edu/~gohlke/pythonlibs/#pymssql)。

确保选择正确的 whl 文件。

例如，如果在 64 位计算机上使用 Python 2.7，请选择：pymssql‑2.1.1‑cp27‑none‑win\_amd64.whl。下载 .whl 文件后，请将它放入 C:/Python27 文件夹。

现在，请从命令行使用 pip 安装 pymssql 驱动程序。使用 cd 命令切换到 C:/Python27 并运行以下命令

	pip install pymssql‑2.1.1‑cp27‑none‑win_amd64.whl

可在[此处](http://stackoverflow.com/questions/4750806/how-to-install-pip-on-windows)找到有关使用 pip 的说明


### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。

## 步骤 1：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../includes/sql-database-include-connection-string-details-20-portalshots.md)]


## 步骤 2：连接


[Pymssql.connect](http://pymssql.org/en/latest/ref/pymssql.html) 函数用于连接到 SQL 数据库。

	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')


## 步骤 3：执行查询

[Cursor.execute](http://pymssql.org/en/latest/ref/pymssql.html#pymssql.Cursor.execute) 函数可用于针对 SQL 数据库从查询中检索结果集。此函数实际上可接受任何查询，并返回可使用 [cursor.fetchone()](http://pymssql.org/en/latest/ref/pymssql.html#pymssql.Cursor.fetchone) 循环访问的结果集。


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute('SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;')
	row = cursor.fetchone()
	while row:
	    print str(row[0]) + " " + str(row[1]) + " " + str(row[2]) 	
	    row = cursor.fetchone()


## 步骤 4：插入行

在本示例中，你将了解如何安全地执行 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) 语句，传递参数以保护应用程序免遭 [SQL 注入](https://technet.microsoft.com/zh-cn/library/ms161953(v=sql.105).aspx) 漏洞的危害，然后检索自动生成的[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express', 'SQLEXPRESS', 0, 0, CURRENT_TIMESTAMP)")
	row = cursor.fetchone()
	while row:
	    print "Inserted Product ID : " +str(row[0])
	    row = cursor.fetchone()


## 步骤 5：回退事务


此代码示例演示了你可以在其中执行以下操作的事务的用法：


-开始一个事务

-插入一行数据

-回滚事务以撤消插入


	import pymssql
	conn = pymssql.connect(server='yourserver.database.chinacloudapi.cn', user='yourusername@yourserver', password='yourpassword', database='AdventureWorks')
	cursor = conn.cursor()
	cursor.execute("BEGIN TRANSACTION")
	cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New', 'SQLEXPRESS New', 0, 0, CURRENT_TIMESTAMP)")
	conn.rollback()

## 后续步骤

有关详细信息，请参阅 [Python 开发人员中心](/develop/python)。

<!---HONumber=Mooncake_0118_2016-->
