<properties linkid="develop-php-sql-database" urlDisplayName="SQL 数据库" pageTitle="如何使用 SQL 数据库 (PHP) - Azure 功能指南" metaKeywords="Azure SQL 数据库 PHP, SQL 数据库 PHP" description="了解如何通过 PHP 创建并连接到 Azure SQL 数据库。" metaCanonical="" services="sql-database" documentationCenter="PHP" title="How to Access Azure SQL 数据库 from PHP" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-management" 
   ms.date="03/25/2015"
   wacn.date="05/25/2015"
   ms.author="kaivalyh"/>

# 如何从 PHP 访问 Azure SQL 数据库 

本指南将介绍通过 PHP 使用 Azure SQL 数据库 的基础知识。示例是使用 PHP 编写的。所涉及的任务包括**创建 SQL 数据库** 和**连接 SQL 数据库**。本指南介绍从[管理门户][management-portal]创建 SQL 数据库 的步骤。有关从生产门户执行这些任务的信息，请参阅 [PHP 和 SQL 数据库 入门][prod-portal-instructions]。有关更多信息，请参阅本教程末尾的[后续步骤](#NextSteps) 部分。

## 什么是 Azure SQL 数据库

Azure SQL 数据库 为 Azure 提供关系数据库管理系统并且基于 SQL Server 技术。使用 SQL 数据库 实例，你可以轻松地配置关系数据库解决方案并将其部署到云中，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 目录

* [概念](#Concepts)
* [如何：设置环境](#Setup)
* [如何：创建 SQL 数据库](#CreateServer)
* [如何：获取 SQL 数据库 连接信息](#ConnectionInfo)
* [如何：连接到 SQL 数据库 实例](#Connect)
* [后续步骤](#NextSteps)

## <a id="Concepts"></a>概念
由于 Azure SQL 数据库 是基于 SQL Server 技术构建的，因此通过 PHP 访问 SQL 数据库 与通过 PHP 访问 SQL Server 非常相似。你可以本地部署应用程序（使用 SQL Server），然后通过仅更改连接字符串连接到 SQL 数据库。但是，SQL 数据库 和 SQL Server 之间有一些可能影响你的应用程序的差别。有关详细信息，请参阅[指导原则和限制 (SQL 数据库)][limitations]。

通过 PHP 访问 SQL 数据库 的推荐方法是使用 [Microsoft Drivers for PHP for SQL Server][download-drivers]。（本文中的示例将使用这些驱动程序。）Microsoft Drivers for PHP for SQL Server 只在 Windows 上运行。

## <a id="Setup"></a>如何：设置环境

设置部署环境的推荐方法是使用 [Microsoft Web 平台安装程序][wpi-installer]。Web 平台安装程序将允许你选择 Web 开发平台的元素，并自动安装和配置这些元素。通过下载 Web 平台安装程序和选择安装 WebMatrix、PHP for WebMatrix 和 SQL Server Express，将为你设置完整的开发环境。

此外，还可以手动设置你的环境：

* 安装 PHP 并配置 IIS：[http://php.net/manual/zh/install.windows.iis7.php][manual-config]。
* 下载并安装 SQL Server Express：[http://www.microsoft.com/zh-cn/download/details.aspx?id=29062][install-sql-express]
* 下载并安装 Microsoft Drivers for PHP for SQL Server：[http://php.net/manual/zh/sqlsrv.requirements.php][install-drivers]。

## <a id="CreateServer"></a>如何：创建 SQL 数据库

按照以下步骤创建 Azure SQL 数据库：

1. 登录到[管理门户][management-portal]。
2. 单击该门户左下角的"新建"图标。

	![Create New Azure  Website][new- Website]

3. 依次单击"数据服务"、"SQL 数据库"和"快速创建"。提供数据库的名称，指定是要使用现有的还是新的数据库服务器或区域，以及管理员名称和密码。

	![Custom Create a new SQL 数据库][quick-create]


若要查看服务器和数据库信息，请在管理门户中单击"SQL 数据库"。随后可以单击"数据库"或"服务器"来查看相关信息。

![View server and database information][sql-dbs-servers]

## <a id="ConnectionInfo"></a>如何：获取 SQL 数据库 连接信息

若要获取 SQL 数据库 连接信息，请在门户中单击"SQL 数据库"，然后单击该数据库的名称。

![View database information][go-to-db-info]

然后，单击"查看 ADO.NET、ODBC、PHP 和 JDBC 的 SQL 数据库 连接字符串"。

![Show connection strings][show-connection-string]

在结果窗口的 PHP 部分中，记下 **SERVER**、**DATABASE** 和 **USERNAME** 的值。你的密码将是你创建 SQL 数据库 时所用的密码。

## <a id="Connect"></a>如何：连接到 SQL 数据库 实例

以下示例演示如何使用 **SQLSRV** 和 **PDO_SQLSRV** 扩展来连接到名为 `testdb` 的 SQL 数据库。你将需要上一节中获取的信息。将 `SERVER_ID` 替换为你的 10 位数服务器 ID（上一节中获取的 SERVER 值的前 10 个字符），并将正确的值（你的用户名称和密码）分配给`$user`变量和`$pwd`变量。

##### SQLSRV

	$server = "tcp:<value of SERVER from section above>";
	$user = "<value of USERNAME from section above>"@SERVER_ID;
	$pwd = "password";
	$db = "testdb";

	$conn = sqlsrv_connect($server, array("UID"=>$user, "PWD"=>$pwd, "Database"=>$db));

	if($conn === false){
		die(print_r(sqlsrv_errors()));
	}

##### PDO_SQLSRV

	$server = "tcp:<value of SERVER from section above>";
	$user = "<value of USERNAME from section above>"@SERVER_ID;
	$pwd = "password";
	$db = "testdb";

	try{
		$conn = new PDO( "sqlsrv:Server= $server ; Database = $db ", $user, $pwd);
		$conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );
	}
	catch(Exception $e){
		die(print_r($e));
	}


## <a id="NextSteps"></a>后续步骤
如前所述，使用 SQL 数据库 与使用 SQL Server 非常相似。建立与 SQL 数据库 的连接之后（如上所述），你可以使用 **SQLSRV** 或 **PDO\_SQLSRV** API 来插入、检索、更新和删除数据。有关 **SQLSRV** 和 **PDO\_SQLSRV** API 的信息，请参阅 [Microsoft Drivers for PHP for SQL Server 文档][driver-docs]。但是，SQL 数据库 和 SQL Server 之间存在一些可能影响你的应用程序的差异。有关详细信息，请参阅[指导原则和限制 (SQL 数据库)][limitations]。

<https://github.com/WindowsAzure/azure-sdk-for-php-samples/tree/master/tasklist-sqlazure> 中提供了演示如何在 Azure 上通过 PHP 使用 SQL 数据库 的示例。

[download-drivers]: http://www.microsoft.com/download/en/details.aspx?id=20098
[limitations]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff394102.aspx
[odbc-php]: http://www.php.net/odbc
[manual-config]: http://php.net/manual/zh/install.windows.iis7.php
[install-drivers]: http://php.net/manual/zh/sqlsrv.requirements.php
[driver-docs]: http://msdn.microsoft.com/zh-cn/library/dd638075(SQL.10).aspx
[access-php-odbc]: http://social.technet.microsoft.com/wiki/contents/articles/accessing-sql-azure-from-php.aspx
[install-sql-express]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
[management-portal]: https://manage.windowsazure.cn
[prod-portal-instructions]: http://blogs.msdn.com/b/brian_swan/archive/2010/02/12/getting-started-with-php-and-sql-azure.aspx
[new- Website]: ./media/sql-database-php-how-to-use-sql-database/plus-new.png
[custom-create]: ./media/sql-database-php-how-to-use-sql-database/create_custom_sql_db-2.png
[database-settings]: ./media/sql-database-php-how-to-use-sql-database/new-sql-db.png
[create-server]: ./media/sql-database-php-how-to-use-sql-database/db-server-settings.png
[sql-dbs-servers]: ./media/sql-database-php-how-to-use-sql-database/sql-dbs-portal.png
[wpi-installer]: http://go.microsoft.com/fwlink/?LinkId=253447
[go-to-db-info]: ./media/sql-database-php-how-to-use-sql-database/go-to-db-info.png
[show-connection-string]: ./media/sql-database-php-how-to-use-sql-database/show-connection-string-2.png
[quick-create]: ./media/sql-database-php-how-to-use-sql-database/create-new-sql.png

<!--HONumber=55-->