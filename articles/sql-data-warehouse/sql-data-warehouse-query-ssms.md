<properties
    pageTitle="连接到 Azure SQL 数据仓库 - SSMS | Azure"
    description="使用 SQL Server Management Studio (SSMS) 连接和查询 Azure SQL 数据仓库。"
    services="sql-data-warehouse"
    documentationcenter=""
    author="hirokib"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="299e50b3-e68a-471c-8aee-b0b9874781bd"
    ms.service="sql-data-warehouse"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="12/12/2016"
    ms.author="elbutter;barbkess" />

# 使用 SQL Server Management Studio (SSMS) 连接到 SQL 数据仓库
>[AZURE.SELECTOR]
- [Visual Studio](/documentation/articles/sql-data-warehouse-query-visual-studio/)
- [sqlcmd](/documentation/articles/sql-data-warehouse-get-started-connect-sqlcmd/)
- [SSMS](/documentation/articles/sql-data-warehouse-query-ssms/)

使用 SQL Server Management Studio (SSMS) 连接和查询 Azure SQL 数据仓库。

## 先决条件
要使用本教程，你需要：

* 现有 SQL 数据仓库。若要创建这样一个数据仓库，请参阅[创建 SQL 数据仓库][Create a SQL Data Warehouse]。
* 已安装 SQL Server Management Studio (SSMS)。如果尚无此工具，可免费[安装 SSMS][Install SSMS]。
* 完全限定的 SQL Server 名称。若要查找此名称，请参阅[连接到 SQL 数据仓库][Connect to SQL Data Warehouse]。

## 1\.连接到 SQL 数据仓库
1. 打开 SSMS。
2. 打开对象资源管理器。若要执行此操作，请选择“文件”>“连接对象资源管理器”。
   
    ![SQL Server 对象资源管理器][1]  

3. 填写“连接到服务器”窗口中的字段。
   
    ![连接到服务器][2]  

   
   * **服务器名称**。输入前面标识的**服务器名称**。
   * **身份验证**。选择“SQL Server 身份验证”或“Active Directory 集成身份验证”。
   * “用户名”和“密码”。如果上面选择了 SQL Server 身份验证，请输入用户名和密码。
   * 单击“连接”。
4. 若要浏览，请展开你的 Azure SQL 服务器。你可以查看与服务器关联的数据库。展开 AdventureWorksDW 以查看示例数据库中的表。
   
    ![浏览 AdventureWorksDW][3]  


## 2\.运行示例查询
现在，你已建立了与数据库的连接，接下来让我们编写查询。

1. 在 SQL Server 对象资源管理器中右键单击你的数据库。
2. 选择“新建查询”。此时将打开一个新的查询窗口。
   
    ![新建查询][4]  

3. 将以下 TSQL 查询复制到查询窗口中：
   
    
    	SELECT COUNT(*) FROM dbo.FactInternetSales;
    
4. 运行该查询。若要执行此操作，请单击 `Execute`，或使用以下快捷键：`F5`。
   
    ![运行查询][5]  

5. 查看查询结果。在此示例中，FactInternetSales 表包含 60398 行。
   
    ![查询结果][6]  


## 后续步骤


若要为 Azure Active Directory 身份验证配置环境，请参阅 [SQL 数据仓库身份验证][Authenticate to SQL Data Warehouse]。

<!--Arcticles-->

[Connect to SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-connect-overview/
[Create a SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-get-started-provision/
[Authenticate to SQL Data Warehouse]: /documentation/articles/sql-data-warehouse-authentication/

<!--Other-->
[Azure portal]: https://portal.azure.cn
[Install SSMS]: https://msdn.microsoft.com/zh-CN/library/hh213248.aspx


<!--Image references-->


[1]: ./media/sql-data-warehouse-query-ssms/connect-object-explorer.png
[2]: ./media/sql-data-warehouse-query-ssms/connect-object-explorer1.png
[3]: ./media/sql-data-warehouse-query-ssms/explore-tables.png
[4]: ./media/sql-data-warehouse-query-ssms/new-query.png
[5]: ./media/sql-data-warehouse-query-ssms/execute-query.png
[6]: ./media/sql-data-warehouse-query-ssms/results.png

<!---HONumber=Mooncake_1205_2016-->