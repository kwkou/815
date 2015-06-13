<properties 
	urlDisplayName="How to connect to an Azure SQL database using SSMS" 
	pageTitle="如何使用 SSMS 连接到 Azure SQL Database" metaKeywords="" 
	description="了解如何使用 SSMS 连接到 Azure SQL Database" 
	metaCanonical="" 
	services="sql-database" 
	documentationCenter="" 
	title="How to connect to an Azure SQL database using SSMS" 
	authors="sidneyh" solutions="" 
	manager="jhubbard" editor="" />

<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/02/2015" 
	wacn.date="05/25/2015" 
	ms.author="sidneyh" />

# 如何使用 SSMS 连接到 Azure SQL Database

本主题介绍使用 SQL Server Management Studio 连接到 Microsoft Azure SQL Database 的步骤。

## 先决条件
* 设置并运行了 Azure SQL Database。若要创建新的 SQL Database，请参阅 [Microsoft Azure SQL Database 入门](sql-database-get-started)。
* SQL Database 的管理员名称和密码。
* SQL Server Management Studio 2014。若要获取该工具，请参阅[下载 SQL Express](http://www.hanselman.com/blog/DownloadSQLServerExpress.aspx)。

## 连接到 SQL Database 的实例
1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 单击"浏览"按钮，然后单击"SQL Database"(b)。 

	![Click Browse and SQL Database][1]
3. 选择"SQL Database"(a) 后，单击服务器上你要连接到的数据库名称 (b)。

	![Click the name of a database][2]
4. 选择名称后 (a)，单击"属性"(b)。复制服务器的名称 (c) 和管理员的名称 (d)。管理员名称和密码是创建 SQL Database 时创建的。你必须使用该密码来继续下一步。 

	![Click SQL Server, Settings, and Property][3]
5. 打开 SQL Server Management Studio 2014。 
6. 在"连接到服务器"对话框中，将服务器名称粘贴到"服务器名称"框中 (a)。将"身份验证"设置为"SQL Server 身份验证"(b)。将服务器管理员名称粘贴到"登录"框中 (c)，然后键入管理员的密码 (d)。然后单击"选项"(e)。

	![SSMS login dialog box][4]
7. 在"连接属性"选项卡中，将"连接到数据库"框设置为 **master**（或连接到的任何其他数据库）。然后单击"连接"。

	![Set to master and click Connect][5]

## 排查连接问题

在出现问题时，请参阅[排查 Azure SQL Database 连接问题](https://support.microsoft.com/kb/2980233/)。有关可能出现的问题与解答列表，请参阅[排查 Microsoft Azure SQL Database 连接问题](https://support2.microsoft.com/common/survey.aspx?scid=sw;en;3844&showpage=1)。


## 后续步骤
你可以使用 Transact-SQL 语句来创建或管理数据库。请参阅 [CREATE DATABASE (Azure SQL Database)](https://msdn.microsoft.com/zh-cn/library/dn268335.aspx) 和[使用 SQL Server Management Studio 管理 Azure SQL Database](sql-database-manage-azure-ssms)。你还可以将事件记录到 Azure 存储空间。请参阅 [SQL Database 审核入门](sql-database-auditing-get-started)。

<!--Image references-->

[1]:./media/sql-database-connect-to-database/browse-vms.png
[2]:./media/sql-database-connect-to-database/sql-databases.png
[3]:./media/sql-database-connect-to-database/blades.png
[4]:./media/sql-database-connect-to-database/ssms-connect-to-server.png
[5]:./media/sql-database-connect-to-database/ssms-master.png

<!--HONumber=55-->