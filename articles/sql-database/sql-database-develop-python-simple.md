<properties
    pageTitle="使用 Python 连接到 SQL 数据库 | Azure"
    description="演示可用于连接到 Azure SQL 数据库的 Python 代码示例。"
    services="sql-database"
    documentationcenter=""
    author="meet-bhagdev"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="452ad236-7a15-4f19-8ea7-df528052a3ad"
    ms.service="sql-database"
    ms.custom="development"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="01/03/2016"
    wacn.date="02/14/2017"
    ms.author="meetb" />

# 使用 Python 连接到 SQL 数据库


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本主题说明如何使用 Python 来连接和查询 Azure SQL 数据库。可以从 Windows、Ubuntu Linux 或 Mac 平台运行此示例。


## 步骤 1：创建 SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。务必根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。创建数据库后，请确保根据[入门页](/documentation/articles/sql-database-get-started/)中所述，通过启用防火墙规则来启用对 IP 地址的访问

## 步骤 2：配置开发环境
### **Mac OS**
打开终端并导航到要在其中创建 python 脚本的目录。输入以下命令安装 **brew**、**FreeTDS** 和 **pyodbc**。pyodbc 使用 FreeTDS on MacOS 连接到 SQL 数据库。

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    brew uninstall FreeTDS #if you have an existing installed FreeTDS
    brew update
    brew doctor
    brew install freetds --with-unixodbc
    sudo pip install pyodbc==3.1.1

### **Linux (Ubuntu)**
打开终端并导航到要在其中创建 python 脚本的目录。输入以下命令安装**适用于 Linux 的 Microsoft ODBC 驱动程序**和 **pyodbc**。pyodbc 使用 Linux 上的 Microsoft ODBC 驱动程序连接到 SQL 数据库。

    sudo su
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
    curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql.list
    exit
    sudo apt-get update
    sudo apt-get install msodbcsql mssql-tools unixodbc-dev-utf16
    sudo pip install pyodbc==3.1.1

### **Windows**
安装 [Microsoft ODBC 驱动程序 13.1](https://www.microsoft.com/zh-cn/download/details.aspx?id=53339)。pyodbc 使用 Linux 上的 Microsoft ODBC 驱动程序连接到 SQL 数据库。

然后使用 pip 安装 pyodbc

    pip install pyodbc==3.1.1

可在[此处](http://stackoverflow.com/questions/4750806/how-to-install-pip-on-windows)找到有关使用 pip 的说明

## 步骤 3：运行示例代码
创建名为 **sql\_sample.py** 的文件并在其中粘贴以下代码。可以从命令行中使用以下命令运行此操作：
	
	python sql_sample.py

### 连接到 SQL 数据库
[pyodbc.connect](https://mkleehammer.github.io/pyodbc/api-connection.html) 函数用于连接到 SQL 数据库。

    import pyodbc
    server = 'yourserver.database.chinacloudapi.cn'
    database = 'yourdatabase'
    username = 'yourusername'
    password = 'yourpassword'
    #for mac
    #driver = '{/usr/local/lib/libtdsodbc.so}'
    #for linux of windows
    driver= '{ODBC Driver 13 for SQL Server}'
    cnxn = pyodbc.connect('DRIVER='+driver+';PORT=1433;SERVER='+server+';PORT=1443;DATABASE='+database+';UID='+username+';PWD='+ password)
    cursor = cnxn.cursor()
    cursor.execute("select @@VERSION")
    row = cursor.fetchone()
    if row:
        print row

### 执行 SQL SELECT 语句
[Cursor.execute](https://mkleehammer.github.io/pyodbc/api-cursor.html) 函数可用于针对 SQL 数据库从查询中检索结果集。此函数实际上可接受任何查询，并返回可使用 [cursor.fetchone()](https://mkleehammer.github.io/pyodbc/api-cursor.html) 循环访问的结果集。

    import pyodbc
    server = 'yourserver.database.chinacloudapi.cn'
    database = 'yourdatabase'
    username = 'yourusername'
    password = 'yourpassword'
    #for mac
    driver = '{/usr/local/lib/libtdsodbc.so}'
    #for linux or windows
    driver= '{ODBC Driver 13 for SQL Server}'
    cnxn = pyodbc.connect('DRIVER='+driver+';PORT=1433;SERVER='+server+';PORT=1443;DATABASE='+database+';UID='+username+';PWD='+ password)
    cursor = cnxn.cursor()
    cursor.execute("select @@VERSION")
    row = cursor.fetchone()
    while row:
        print str(row[0]) + " " + str(row[1]) + " " + str(row[2])     
        row = cursor.fetchone()


### 插入一行，传递参数，然后检索生成的主键

在 SQL 数据库中，可以使用 [IDENTITY](https://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENCE](https://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。

    import pyodbc
    server = 'yourserver.database.chinacloudapi.cn'
    database = 'yourdatabase'
    username = 'yourusername'
    password = 'yourpassword'
    #for mac
    #driver = '{/usr/local/lib/libtdsodbc.so}'
    #for linux or windows
    driver= '{ODBC Driver 13 for SQL Server}'
    cnxn = pyodbc.connect('DRIVER='+driver+';PORT=1433;SERVER='+server+';PORT=1443;DATABASE='+database+';UID='+username+';PWD='+ password)
    cursor = cnxn.cursor()
    cursor.execute("select @@VERSION")
    row = cursor.fetchone()
    while row:
        print "Inserted Product ID : " +str(row[0])
        row = cursor.fetchone()


### 事务
此代码示例演示了可以在其中执行以下操作的事务的用法：

* 开始一个事务
* 插入一行数据
* 回滚事务以撤消插入

在 sql\_sample.py 中粘贴以下代码。

    import pyodbc
    server = 'yourserver.database.chinacloudapi.cn'
    database = 'yourdatabase'
    username = 'yourusername'
    password = 'yourpassword'
    #for mac
    #driver = '{/usr/local/lib/libtdsodbc.so}'
    #for linux or windows
    driver= '{ODBC Driver 13 for SQL Server}'
    cnxn = pyodbc.connect('DRIVER='+driver+';PORT=1433;SERVER='+server+';PORT=1443;DATABASE='+database+';UID='+username+';PWD='+ password)
    cursor = cnxn.cursor()
    cursor.execute("BEGIN TRANSACTION")
    cursor.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New', 'SQLEXPRESS New', 0, 0, CURRENT_TIMESTAMP)")
    cnxn.rollback()

## 后续步骤

* 参阅 [SQL 数据库开发概述](/documentation/articles/sql-database-develop-overview/)
* 有关 [Microsoft Python Driver for SQL Server](https://msdn.microsoft.com/zh-cn/library/mt652092.aspx) 的详细信息
* 访问 [Python 开发人员中心](/develop/python/)。

## 其他资源 

* [多租户 SaaS 应用程序和 Azure SQL 数据库的设计模式](/documentation/articles/sql-database-design-patterns-multi-tenancy-saas-applications/)
* 浏览所有 [SQL 数据库的功能](/home/features/sql-database/)

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: whole content update(steps, scripts)-->