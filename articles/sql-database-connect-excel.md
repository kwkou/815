<properties
	pageTitle="使用 Excel 连接到 Azure SQL 数据库"
	description="用于连接到 Azure SQL 数据库以生成报告和探索数据的 Excel 电子表格。"
	services="sql-database"
	documentationCenter=""
	authors="joseidz"
	manager="jeffreyg"
	editor="jeffreyg"/>


<tags
	ms.service="sql-database"
	ms.date="10/09/2015"
	wacn.date="11/12/2015"/>


# 使用 Excel 连接到 Azure SQL 数据库

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-connect-query)
- [SSMS](/documentation/articles/sql-database-connect-query-ssms)
- [Excel](/documentation/articles/sql-database-connect-excel)

本文介绍如何将 Excel 连接到 Azure SQL 数据库并基于数据库中的数据创建报告。你首先需要有一个 SQL 数据库。如果你没有，请参阅[创建你的第一个 SQL 数据库](/documentation/articles/sql-database-get-started)，以在几分钟内获取数据库，并让其中的示例数据正常运行。本文基于该文中的示例数据撰写，但你可以对自己的数据执行类似的步骤。

你还需要 Excel 的副本。本文使用 [Microsoft Excel 2016](https://products.office.com/zh-CN/)。

## 连接并创建报表

1.	打开 Excel，然后创建一个新的工作簿或打开要连接的工作簿。

2.	在页面顶部的菜单栏中单击“数据”，单击“从其他源”，然后单击“从 SQL Server”。
	
	![选择数据源](./media/sql-database-connect-excel/excel_data_source.png)

	此时将打开“数据连接”向导。

3.	在“连接到数据库服务器”对话框中，以格式 **<*servername*>.database.chinacloudapi.cn** 键入托管要连接到的逻辑服务器的**服务器名称**。例如，**adventureserver.database.chinacloudapi.cn**。

4.	在“登录凭据”部分中，单击“使用以下用户名和密码”，键入在创建 SQL 数据库服务器时为其设置的**用户名**和**密码**，然后单击“下一步”。

	> [AZURE.TIP]Excel 的 [PowerPivot](https://www.microsoft.com/download/details.aspx?id=102) 和 [Power Query](https://www.microsoft.com/download/details.aspx?id=39379) 外接程序提供类似的体验。

5. 在“选择数据库和表”对话框中，从下拉菜单中选择“AdventureWorks”数据库，从表和视图列表中选择“vGetAllCategories”，然后单击“下一步”。

	![选择数据库和表][5]

6. 在“保存数据连接文件并完成”对话框中，单击“完成”。

7. 在“导入数据”对话框中，选择“PivotChart”，然后单击“确定”。

	![选择“导入数据”][2]

8. 在“PivotChart 字段”对话框中，选择以下配置以便为每个类别创建产品计数报告。

	![配置][3]

	如果操作成功，将显示如下所示的内容：

	![成功][4]

## 后续步骤

如果你是软件即服务 (SaaS) 开发人员，请学习有关[弹性数据库池](/documentation/articles/sql-database-elastic-pool)的知识。你可以使用[弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-overview)轻松管理大量的数据库。

<!--Image references-->
[1]: ./media/sql-database-connect-excel/connect-to-database-server.png
[2]: ./media/sql-database-connect-excel/import-data.png
[3]: ./media/sql-database-connect-excel/power-pivot.png
[4]: ./media/sql-database-connect-excel/power-pivot-results.png
[5]: ./media/sql-database-connect-excel/select-database-and-table.png

<!---HONumber=79-->