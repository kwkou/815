<properties 
	pageTitle="在 SQL 数据库更新版 V12 中创建数据库" 
	description="演示如何在 Azure SQL 数据库更新版 V12 中创建数据库" 
	services="sql-database" 
	documentationCenter="" 
	authors="sonalmm" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="04/28/2015" 
	wacn.date="09/15/2015"/>


# 在 SQL 数据库 V12 中创建数据库


<!--
True author is: authors="sonalmm" , ms.author="sonalm".
-->


[注册](https://manage.windowsazure.cn) SQL 数据库 V12[（在某些区域以预览版提供）](/documentation/articles/sql-database-v12-whats-new/#V12AzureSqlDbPreviewGaTa)，以利用 Azure 上的下一代 SQL 数据库。若要开始，你需要订阅 Azure。你可以注册 [Azure 试用版](/pricing/1rmb-trial)并查看[价格](/home/features/sql-database/pricing/)信息。


| 创建数据库 | 屏幕截图 |
| :--- | :--- |
| 1\.登录到 [https://manage.windowsazure.cn](https://manage.windowsazure.cn)。 | ![新的 Azure 门户][1] |
| 2\.在页面底部的左侧单击“新建”。 | ![启动新服务][2]|
| 3\.单击“SQL Database”。| ![可选择的不同服务][3] |
| 4\.此时将打开“SQL Database”边栏选项卡。在“名称”字段中，指定数据库名称。 | ![命名数据库][4] |
| 5\.在“SQL Database”边栏选项卡中，单击“服务器”。此时将打开“服务器”边栏选项卡，其中提供了两个选项，让你创建新的服务器，或使用现有的服务器。| ![选择服务器的类型][4] |
|5a.如果你选择“使用现有的服务器”选项，请选择一个服务器，然后单击“选择”。然后，完成步骤 6 及其后面各步骤的所有操作。| ![从列表中选择服务器][5]| 
|5b.如果你选择“创建新的服务器”，则会打开“新建服务器”边栏选项卡。指定服务器名称、服务器管理员登录名和密码。单击“位置”选择服务器位置。 | ![填写创建新服务器的选项][9]| 
|5c.“新建服务器”边栏选项卡中提供了相应的选项让你创建包含 V12 更新的新服务器。若要了解有关 V12 服务器功能的详细信息，请查看 [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new/)。| ![选择 V12 服务器][6]|
|5d.在“新建服务器”边栏选项卡上做出选择，然后单击“确定”。随后，你将会转到“SQL Database”边栏选项卡，请完成创建数据库所要执行的余下操作。 | ![完成“新建服务器”边栏选项卡中的操作][10]|
|6\.单击“选择源”。可选择从中创建数据库的各种源类型包括：空白数据库、示例数据库或数据库的备份。| ![选择数据库的源][10]|
|7\.单击“创建”。随即会创建一个包含 SQL Database V12 功能的新数据库。 |![创建新数据库][10]

## 相关链接

- [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new/)

<!--Image references-->
[1]: ./media/sql-database-create/firstscreenportal.png
[2]: ./media/sql-database-create/new.png
[3]: ./media/sql-database-create/sqldatabase.png
[4]: ./media/sql-database-create/databasename.png
[5]: ./media/sql-database-create/useexistingserver.PNG
[6]: ./media/sql-database-create/v12server.PNG
[7]: ./media/sql-database-create/pricingtierdetails.png
[8]: ./media/sql-database-create/finishnewserverblade.png
[9]: ./media/sql-database-create/createnewserver.png
[10]: ./media/sql-database-create/selectsource.png
[11]: ./media/sql-database-create/resourcegroup.png
[12]: ./media/sql-database-create/create.png

<!---HONumber=69-->