<properties 
	urlDisplayName="How to connect to an Azure SQL 数据库 using SSMS" 
	pageTitle="如何使用 SSMS 连接到 Azure SQL 数据库| Windows Azure" 
	metaKeywords=""
	description="了解如何使用 SSMS 连接到 Azure SQL 数据库。"
	metaCanonical=""
	services="sql-database"
	documentationCenter=""
	title="How to connect to an Azure SQL 数据库 using SSMS" 
	authors="sidneyh" solutions=""
	manager="jhubbard" editor="" />

<tags
	ms.service="sql-database"
	ms.date="07/15/2015"
	wacn.date="09/15/2015" />

# 使用 SQL Server Management Studio 进行连接

使用以下步骤来安装 SQL Server Management Studio (SSMS)，然后使用 SSMS 来连接和查询 SQL 数据库。

## 先决条件
* [Windows Azure SQL 数据库入门](/documentation/articles/sql-database-get-started)中所述的 SQL 数据库 AdventureWorks 示例数据库。

## 安装并启动 SQL Server Management Studio (SSMS)
1. 转到 [SQL Server 2014 Express](http://www.microsoft.com/zh-cn/download/details.aspx?id=42299) 的下载页，单击“下载”，然后选择 32 位版本 (x86) 或 64 位版本 (x64) 的 MgmtStudio 下载文件。

	![MgtmtStudio32BIT 或 MgmtStudio64BIT][1]
2.	如果使用默认设置安装 SSMS，请根据提示操作。
3.	下载后，在电脑上搜索 SQL Server 2014 Management Studio，然后启动 SSMS。


## 连接到 SQL Database
1. 打开 SSMS。
2. 在“连接到服务器”窗口的“服务器名称”框中，输入格式为 *&lt;服务器名称>*.**database.windows.net** 的服务器名称。
3. 在“身份验证”列表中，选择“SQL Server 身份验证”。
4. 输入你创建 SQL 数据库服务器时指定的**登录名**和**密码**。

	![连接到服务器对话框][2]
5. 单击“选项”按钮。
6. 在“连接到数据库”框中输入 **AdventureWorks**，然后单击“连接”。

	![连接到数据库][3]

### 如果连接失败
确保创建的逻辑服务器防火墙允许来自本地计算机的连接。有关详细信息，请参阅[如何：配置防火墙设置（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/azure/jj553530.aspx)。

## 运行示例查询

1. 在“对象资源管理器”中，导航到 **AdventureWorks** 数据库。
2. 右键单击数据库，然后选择“新建查询”。

	![新建查询][4]

3. 在查询窗口中，复制并粘贴以下代码。

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
你可以使用 Transact-SQL 语句来创建或管理数据库。有关详细信息，请参阅 [CREATE DATABASE（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx)和[使用 SQL Server Management Studio 管理 Azure SQL 数据库](/documentation/articles/sql-database-manage-azure-ssms)。你还可以将事件记录到 Azure 存储空间。有关详细信息，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started)。

<!--Image references-->

[1]: ./media/sql-database-connect-to-database/1-download.png
[2]: ./media/sql-database-connect-to-database/2-connect.png
[3]: ./media/sql-database-connect-to-database/3-connect-to-database.png
[4]: ./media/sql-database-connect-to-database/4-run-query.png
[5]: ./media/sql-database-connect-to-database/5-success.png

<!---HONumber=69-->