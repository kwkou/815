<properties 
	pageTitle="在 Mac OS X 上使用 Ruby 和 TinyTDS 连接到 SQL Database (Yosemite)" 
	description="提供可在 Mac OS X (Yosemite) 上运行的，用于连接到 Azure SQL Database 的 Ruby 代码示例。"
	services="sql-database" 
	documentationCenter="" 
	authors="ajlam" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" ms.date="07/20/2015" wacn.date=""/>


# 在 Mac OS X 上使用 Ruby 连接到 SQL Database (Yosemite)

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

本主题演示了一个在装有 Yosemite 的 Mac 计算机上运行的，用于连接到 Azure SQL Database 数据库的 Ruby 代码示例。

## 安装所需的模块

打开终端并安装以下组件：

**1) Homebrew**：从你的终端运行以下命令。这将在你的计算机上下载 Homebrew 包管理器。

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

**2) FreeTDS**：从你的终端运行以下命令。这将在你的计算机上安装 FreeTDS。要使 TinyTDS 正常工作，必须安装该组件。

    brew install FreeTDS

**3) TinyTDS**：从你的终端运行以下命令。这将在你的计算机上安装 TinyTDS。

    sudo ARCHFLAGS="-arch x86_64" gem install tiny_tds

## 创建数据库并检索连接字符串

Ruby 示例依赖于 AdventureWorks 示例数据库。如果你尚未创建 AdventureWorks，可以通过以下主题了解如何创建：[创建你的第一个 Azure SQL Database](/documentation/articles/sql-database-get-started)

本主题还将说明如何检索数据库连接字符串。

## 连接到 SQL Database

[TinyTDS::Client](https://github.com/rails-sqlserver/tiny_tds) 函数用于连接到 SQL Database。

    require 'tiny_tds' 
    client = TinyTds::Client.new username: 'yourusername@yourserver', password: 'yourpassword', 
    host: 'yourserver.database.chinacloudapi.cn', port: 1433, 
    database: 'AdventureWorks', azure:true 

## 执行 SELECT 语句并检索结果集

[TinyTds::Result](https://github.com/rails-sqlserver/tiny_tds) 函数用于检索针对 SQL Database 执行的查询所返回的结果集。此函数接受查询并返回结果集。可以使用 [result.each do |row|](https://github.com/rails-sqlserver/tiny_tds) 循环访问结果集。

    require 'tiny_tds'  
    print 'test'     
    client = TinyTds::Client.new username: 'yourusername@yourserver', password: 'yourpassword', 
    host: 'yourserver.database.chinacloudapi.cn', port: 1433, 
    database: 'AdventureWorks', azure:true 
    results = client.execute("select * from SalesLT.Product") 
    results.each do |row| 
    puts row 
    end 

## 插入一行，传递参数，然后检索生成的主键值

代码示例：

- 将参数传递给要插入行中的值。
- 插入行。
- 检索为主键生成的值。

在 SQL Database 中，可以使用 [IDENTITY](http://msdn.microsoft.com/zh-cn/library/ms186775.aspx) 属性和 [SEQUENCE](http://msdn.microsoft.com/zh-cn/library/ff878058.aspx) 对象自动生成[主键](http://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。

若要配合使用 TinyTDS 和 Azure，建议你运行多个 `SET` 语句来更改当前会话处理特定信息的方式。建议使用代码示例中所提供的 `SET` 语句。例如，即使未显式指定列的可为 null 状态，`SET ANSI_NULL_DFLT_ON` 也可以让你创建新列来允许 null 值。

为了符合 Microsoft SQL Server [日期时间](http://msdn.microsoft.com/zh-cn/library/ms187819.aspx)格式，请使用 [strftime](http://ruby-doc.org/core-2.2.0/Time.html#method-i-strftime) 函数转换成对应的日期时间格式。

    require 'tiny_tds' 
    client = TinyTds::Client.new username: 'yourusername@yourserver', password: 'yourpassword', 
    host: 'yourserver.database.chinacloudapi.cn', port: 1433, 
    database: 'AdventureWorks', azure:true 
    results = client.execute("SET ANSI_NULLS ON")
    results = client.execute("SET CURSOR_CLOSE_ON_COMMIT OFF")
    results = client.execute("SET ANSI_NULL_DFLT_ON ON")
    results = client.execute("SET IMPLICIT_TRANSACTIONS OFF")
    results = client.execute("SET ANSI_PADDING ON")
    results = client.execute("SET QUOTED_IDENTIFIER ON")
    results = client.execute("SET ANSI_WARNINGS ON")
    results = client.execute("SET CONCAT_NULL_YIELDS_NULL ON")
    require 'date'
    t = Time.now
    curr_date = t.strftime("%Y-%m-%d %H:%M:%S.%L") 
    results = client.execute("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) 
    OUTPUT INSERTED.ProductID VALUES ('SQL Server Express New', 'SQLEXPRESS New', 0, 0, '#{curr_date}' )")
    results.each do |row| 
    puts row
    end

<!---HONumber=66-->