<properties
	pageTitle="在 Windows 上使用 Ruby 和 TinyTDS 连接到 SQL 数据库"
	description="提供可在 Windows 上运行的，用于连接到 Azure SQL 数据库的 Ruby 代码示例。"
	services="sql-database"
	documentationCenter=""
	authors="meet-bhagdev"
	manager="jeffreyg"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.date="03/14/2016"
	wacn.date="03/24/2016"/>


# 在 Windows上使用 Ruby 连接到 SQL 数据库


<!--
Older Selector technique, with dynamic drop-down lists.
 [A ZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)]
-->

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../includes/sql-database-develop-includes-selector-language-platform-depth.md)] 




本主题演示了一个在使用 Windows 8.1 的 Windows 计算机上运行的，用于连接到 Azure SQL 数据库的 Ruby 代码示例。

## 先决条件

###安装所需的模块

打开终端并安装以下组件：

**1) Ruby：**如果你的计算机上没有 Ruby，请安装它。若是新的 Ruby 用户，建议使用 Ruby 2.1.X 安装程序。这些安装程序提供兼容且更新的稳定语言和扩展的包列表 (gem)。[转到 Ruby 下载页](http://rubyinstaller.org/downloads)并下载适当的 2.1.x 安装程序。例如，如果你使用 64 位计算机，请下载 **Ruby 2.1.6 (x64)** 安装程序。
<br/><br/>下载安装程序后，请执行以下操作：


- 双击安装文件启动安装程序。

- 选择你的语言，并同意接受条款。

- 在安装设置屏幕上，选中“将 Ruby 可执行文件添加到你的路径”及“将 .rb 和 .rbw 文件与此 Ruby 安装相关联”旁边的复选框。


**2) DevKit：**从 [RubyInstaller 页](http://rubyinstaller.org/downloads)下载 DevKit

下载完成后，执行以下操作：


- 双击安装证书。系统将询问你要文件解压缩到哪个位置。

- 单击“...”按钮，然后选择“C:\\DevKit”。你可能事先需要通过单击“新建文件夹”来创建此文件夹。

- 单击“确定”，然后单击“解压缩”以解压缩文件。


现在，请打开命令提示符并输入以下命令：

	> chdir C:\DevKit
	> ruby dk.rb init
	> ruby dk.rb install

现在，你已安装了全功能的 Ruby 和 RubyGems！


**3) TinyTDS：**导航到 C:\\DevKit 并从终端运行以下命令。这将在你的计算机上安装 TinyTDS。

	gem inst tiny_tds --pre

### SQL 数据库

请参阅[入门页](/documentation/articles/sql-database-get-started/)，以了解如何创建示例数据库。必须根据指南创建 **AdventureWorks 数据库模板**。下面所示的示例只适用于 **AdventureWorks 架构**。


## 步骤 1：获取连接详细信息

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## 步骤 2：连接

[TinyTDS::Client](https://github.com/rails-sqlserver/tiny_tds) 函数用于连接到 SQL 数据库。

    require 'tiny_tds'
    client = TinyTds::Client.new username: 'yourusername@yourserver', password: 'yourpassword',
    host: 'yourserver.database.chinacloudapi.cn', port: 1433,
    database: 'AdventureWorks', azure:true

## 步骤 3：执行查询

复制以下代码并将它粘贴到空文件中。将文件命名为 test.rb。然后，在命令提示符下输入以下命令以执行该文件：

	ruby test.rb

在代码示例中，[TinyTds::Result](https://github.com/rails-sqlserver/tiny_tds) 函数用于检索针对 SQL 数据库执行的查询所返回的结果集。此函数接受查询并返回结果集。可以使用 [result.each do |row|](https://github.com/rails-sqlserver/tiny_tds) 循环访问结果集。

    require 'tiny_tds'  
    print 'test'     
    client = TinyTds::Client.new username: 'yourusername@yourserver', password: 'yourpassword',
    host: 'yourserver.database.chinacloudapi.cn', port: 1433,
    database: 'AdventureWorks', azure:true
    results = client.execute("SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC")
    results.each do |row|
    puts row
    end

## 步骤 4：插入行

在本示例中，你将了解如何安全地执行 [INSERT](https://msdn.microsoft.com/zh-cn/library/ms174335.aspx) 语句，传递参数以保护应用程序免遭 [SQL 注入](https://technet.microsoft.com/zh-cn/library/ms161953(v=sql.105).aspx) 漏洞的危害，然后检索自动生成的[主键](https://msdn.microsoft.com/zh-cn/library/ms179610.aspx)值。

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

<!---HONumber=Mooncake_0118_2016-->
