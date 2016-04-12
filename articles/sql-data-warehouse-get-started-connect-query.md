<properties
   pageTitle="入门：连接到 Azure SQL 数据仓库 | Azure"
   description="开始连接到 SQL 数据仓库并运行一些查询。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="twounder"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/03/2016"
   wacn.date="04/11/2016"/>

# 使用 Visual Studio 进行连接和查询

> [AZURE.SELECTOR]
- [Visual Studio](/documentation/articles/sql-data-warehouse-get-started-connect)
- [SQLCMD](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd)

本演练说明如何使用 Visual Studio 在短时间内连接和查询 Azure SQL 数据仓库数据库。在本演练中，你将：

+ 安装必备软件
+ 连接到包含 AdventureWorksDW 示例数据库的数据库
+ 对示例数据库执行查询  

## 先决条件

+ Visual Studio 2013/2015 - 若要下载并安装 Visual Studio 2015 和/或 SSDT，请参阅[安装 Visual Studio 和 SSDT](/documentation/articles/sql-data-warehouse-install-visual-studio)

## 获取完全限定的 Azure SQL 服务器名称

若要连接到数据库，你需要服务器的完整名称 (****servername**.database.chinacloudapi.cn*)，该名称中包含要连接到的数据库。

1. 通过以下命令查找我们需要的服务器信息。本示例使用 AdventureWorksDW 示例数据库, DataWarehouse资源组。
 
	PS C:\> Get-AzureRmSqlServer -ResourceGroupName "DataWarehouse" -DatabaseName "AdventureWorksDW"  

## 连接到 SQL 数据库

1. 打开 Visual Studio。
2. 从“视图”菜单打开“SQL Server 对象资源管理器”
 
![][2]

3. 单击“添加 SQL Server”按钮

![][3]

4. 输入我们在前面捕获的*服务器名称*
5. 在“身份验证”列表中，选择“SQL Server 身份验证”。
6. 输入你创建 SQL 数据库服务器时指定的“登录名”和“密码”，然后单击“连接”。

## 运行示例查询

现在，我们已注册了服务器，接下来继续编写查询。

1. 在 SSDT 中单击用户数据库。

2. 单击“新建查询”按钮。此时将打开一个新窗口。

![][4]

3. 将以下代码键入查询窗口：

	```
	SELECT COUNT(*) FROM dbo.FactInternetSales;
	```

4. 运行该查询。

	若要运行查询，请单击绿色箭头，或使用以下快捷键：`CTRL`+`SHIFT`+`E`。





<!--Image references-->

[1]: ./media/sql-data-warehouse-get-started-connect-query/get-server-name.png
[2]: ./media/sql-data-warehouse-get-started-connect-query/open-ssdt.png
[3]: ./media/sql-data-warehouse-get-started-connect-query/connection-dialog.png
[4]: ./media/sql-data-warehouse-get-started-connect-query/new-query.png
<!---HONumber=Mooncake_0321_2016-->