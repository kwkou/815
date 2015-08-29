<properties
	pageTitle="使用 Excel 连接到 Azure SQL Database"
	description="用于连接到 Azure SQL Database 以生成报告和探索数据的 Excel 电子表格。"
	services="sql-database"
	documentationCenter=""
	authors="joseidz"
	manager="joseidz"
	editor="joseidz"/>


<tags
	ms.service="sql-database" ms.date="07/09/2015" wacn.date=""/>


# 使用 Excel 连接到 Azure SQL Database
将 Excel 连接到 Azure SQL Database，并基于数据库中的数据创建报告。

## 先决条件
- 已设置并运行 Azure SQL Database。若要创建新的 SQL Database，请参阅 [Microsoft Azure SQL Database 入门](/documentation/articles/sql-database-get-started)。
- [Microsoft Excel 2013](https://products.office.com/zh-cn/)（或 Microsoft Excel 2010）

## 连接到 SQL Database 并创建报告
1.	打开 Excel。
2.	在页面顶部的菜单栏中，单击“数据”。
3.	单击“从其他源”，然后单击“从 SQL Server”。“数据连接向导”即会打开。

	![数据连接向导][1]
4.	在“服务器名称”框中，键入 Azure SQL Database 服务器名称。示例：

	 	adventureserver.database.chinacloudapi.cn
5.	在“登录凭据”部分中，选择“使用以下用户名和密码”，然后键入 SQL Database 服务器的正确凭据。然后单击“下一步”。

	

6. 在“选择数据库和表”对话框中，从下拉菜单中选择“AdventureWorks”数据库，从表和视图列表中选择“vGetAllCategories”，然后单击“下一步”。

	![选择数据库和表][5]
7. 在“保存数据连接文件并完成”对话框中，单击“完成”。
8. 在“导入数据”对话框中，选择“PivotChart”，然后单击“确定”。

	![选择“导入数据”][2]
9. 在“PivotChart 字段”对话框中，选择以下配置以便为每个类别创建产品计数报告。

	![配置][3]
10.	如果操作成功，将显示如下所示的内容：

	![成功][4]

## 后续步骤

如果你是软件即服务 (SaaS) 开发人员，请学习有关[弹性数据库池](/documentation/articles/sql-database-elastic-pool)的知识。你可以使用[弹性数据库作业](/documentation/articles/sql-database-elastic-jobs-overview)轻松管理大量的数据库。

<!--Image references-->
[1]: ./media/sql-database-connect-excel/connect-to-database-server.png
[2]: ./media/sql-database-connect-excel/import-data.png
[3]: ./media/sql-database-connect-excel/power-pivot.png
[4]: ./media/sql-database-connect-excel/power-pivot-results.png
[5]: ./media/sql-database-connect-excel/select-database-and-table.png

<!---HONumber=66-->