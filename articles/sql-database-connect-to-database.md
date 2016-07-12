<properties
   pageTitle="如何使用 SSMS 连接到 Azure SQL 数据库 | Azure"
   description="了解如何使用 SSMS 连接到 Azure SQL 数据库。"
   services="sql-database"
   documentationCenter=""
   authors="sidneyh"
   manager="jeffreyg"
   editor=""
   tags=""/>
<tags
   ms.service="sql-database"
   ms.date="09/14/2015"
   wacn.date="12/22/2015" />

# 使用 SQL Server Management Studio 进行连接 (SSMS)

通过以下步骤使用 SQL Server Management Studio (SSMS) 连接和查询 SQL 数据库。

## 先决条件

* SQL Server Management Studio (SSMS) - 若要下载最新版本的 SSMS，请参阅[下载 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。
* [Azure SQL 数据库入门](/documentation/articles/sql-database-get-started/)中所述的 AdventureWorks 示例数据库。


## 获取完全限定的 Azure SQL 服务器名称

若要连接到数据库，你需要服务器的完整名称 (***servername**.database.chinacloudapi.cn*)，该名称中包含要连接到的数据库。

1. 转到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 浏览到要连接到的数据库。
3. 找到完整的服务器名称：

    ![完全限定的服务器名称][6]

    使用下述步骤 3 中的完全限定的服务器名称。



## 连接到 SQL 数据库

1. 打开 SSMS。
2. 单击“连接”>“数据库引擎...”

    ![连接 > 数据库引擎][7]

2. 在“连接到服务器”窗口的“服务器名称”框中，键入格式为 *&lt;服务器名称>*.**database.chinacloudapi.cn** 的服务器名称。
3. 在“身份验证”列表中，选择“SQL Server 身份验证”。
4. 输入你创建 SQL 数据库服务器时指定的“登录名”和“密码”，然后单击“连接”。

	![连接到服务器对话框][2]



### 如果连接失败
确保创建的逻辑服务器防火墙允许来自本地计算机的连接。

## 运行示例查询

1. 在“对象资源管理器”中，导航到 **AdventureWorks** 数据库。
2. 右键单击数据库，然后选择“新建查询”。

	![新建查询][4]

3. 在查询窗口中，复制并粘贴以下代码：

		SELECT
		CustomerId
		,Title
		,FirstName
		,LastName
		,CompanyName
		FROM SalesLT.Customer;

4. 单击“运行”按钮。以下屏幕截图显示查询成功。

	![成功][5]




## 后续步骤
你可以使用 Transact-SQL 语句来创建或管理数据库。有关详细信息，请参阅 [CREATE DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)和[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms/)。你还可以将事件记录到 Azure 存储空间。有关详细信息，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started/)。

<!--Image references-->

[1]: ./media/sql-database-connect-to-database/1-download.png
[2]: ./media/sql-database-connect-to-database/2-connect.png
[3]: ./media/sql-database-connect-to-database/3-connect-to-database.png
[4]: ./media/sql-database-connect-to-database/4-run-query.png
[5]: ./media/sql-database-connect-to-database/5-success.png
[6]: ./media/sql-database-connect-to-database/server-name.png
[7]: ./media/sql-database-connect-to-database/connect-dbengine.png

<!---HONumber=Mooncake_1207_2015-->