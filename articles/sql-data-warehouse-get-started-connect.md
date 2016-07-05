<properties
   pageTitle="使用 Visual Studio 连接到 SQL 数据仓库 | Azure"
   description="开始连接到 SQL 数据仓库并运行一些查询。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/13/2016"
   wacn.date="05/30/2016"/>

# 使用 Visual Studio 连接到 SQL 数据仓库

> [AZURE.SELECTOR]
- [Visual Studio](/documentation/articles/sql-data-warehouse-get-started-connect)
- [SQLCMD](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd)

本演练说明如何使用 Visual Studio 中的 SQL Server Data Tools (SSDT) 扩展在短时间内连接到 Azure SQL 数据仓库。连接后，你将运行一个简单的查询。

## 先决条件

+ SQL 数据仓库中的 AdventureWorksDW 示例数据。若要创建此数据，请参阅[创建 SQL 数据仓库][]。
+ SQL Server Data Tools for Visual Studio。有关安装指说明和选项，请参阅[安装 Visual Studio 和 SSDT][]。

## 步骤 1：查找完全限定的 Azure SQL 服务器名称

你的数据库与 Azure SQL 服务器相关联。若要连接到你的数据库，需要服务器的完全限定名称 (**servername**.chinacloudapi.cn)。

若要查找完全限定的服务器名称，请执行以下操作。

## 步骤 1：通过以下命令查找我们需要的服务器信息。本示例使用 AdventureWorksDW 示例数据库,Group1 资源组。
 
	PS C:\> Get-AzureRmSqlServer -ResourceGroupName "Group1" -DatabaseName "AdventureWorksDW"  

## 步骤 2：连接到你的 SQL 数据库

1. 打开 Visual Studio 2013 或 2015。
2. 打开 SQL Server 对象资源管理器。为此，请选择“视图”>“SQL Server 对象资源管理器”。

    ![SQL Server 对象资源管理器][2]

3. 单击“添加 SQL Server”图标。

    ![添加 SQL 服务器][3]

4. 填写“连接到服务器”窗口中的字段。

    ![连接到服务器][4]

    - **服务器名称**。输入前面标识的**服务器名称**。
    - **身份验证**。选择“SQL Server 身份验证”或“Active Directory 集成身份验证”。
    - “用户名”和“密码”。如果上面选择了 SQL Server 身份验证，请输入用户名和密码。
    - 单击“连接”。

5. 若要浏览，请展开你的 Azure SQL 服务器。你可以查看与服务器关联的数据库。展开 AdventureWorksDW 以查看示例数据库中的表。

    ![浏览 AdventureWorksDW][5]

## 步骤 3：运行示例查询

现在，你已建立了与数据库的连接，接下来让我们编写查询。

1. 在 SQL Server 对象资源管理器中右键单击你的数据库。

2. 选择“新建查询”。此时将打开一个新的查询窗口。

    ![新建查询][6]

3. 将以下 TSQL 查询复制到查询窗口中：

	```
	SELECT COUNT(*) FROM dbo.FactInternetSales;
	```

4. 运行该查询。为此，请单击绿色箭头，或使用以下快捷键：`CTRL`+`SHIFT`+`E`。

    ![运行查询][7]

5. 查看查询结果。在此示例中，FactInternetSales 表包含 60398 行。

    ![查询结果][8]

## 后续步骤

既然你可以执行连接和查询，接下来请尝试[使用 PowerBI 可视化数据][]。

若要为 Windows 身份验证配置环境，请参阅 [使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库][]。

<!--Arcticles-->
[创建 SQL 数据仓库]: /documentation/articles/sql-data-warehouse-get-started-provision
[安装 Visual Studio 和 SSDT]: /documentation/articles/sql-data-warehouse-install-visual-studio
[使用 Azure Active Directory 身份验证连接到 SQL 数据库或 SQL 数据仓库]: /documentation/articles/sql-database/sql-database-aad-authentication
[使用 PowerBI 可视化数据]: /documentation/articles/sql-data-warehouse-get-started-visualize-with-power-bi

<!--Other-->
[Azure 经典门户]: https://manage.windowsazure.cn

<!--Image references-->

[1]: ./media/sql-data-warehouse-get-started-connect/get-server-name.png
[2]: ./media/sql-data-warehouse-get-started-connect/open-ssdt.png
[3]: ./media/sql-data-warehouse-get-started-connect/add-server.png
[4]: ./media/sql-data-warehouse-get-started-connect/connection-dialog.png
[5]: ./media/sql-data-warehouse-get-started-connect/explore-sample.png
[6]: ./media/sql-data-warehouse-get-started-connect/new-query2.png
[7]: ./media/sql-data-warehouse-get-started-connect/run-query.png
[8]: ./media/sql-data-warehouse-get-started-connect/query-results.png
<!---HONumber=Mooncake_0523_2016-->