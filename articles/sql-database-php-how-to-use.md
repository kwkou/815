<properties linkid="develop-php-sql-database" urlDisplayName="SQL Database" pageTitle="How to use SQL Database (PHP) - Azure feature guides" metaKeywords="Azure SQL Database PHP, SQL Database PHP" description="Learn how to create and connect to an Azure SQL Database from PHP." metaCanonical="" services="sql-database" documentationCenter="PHP" title="How to Access Azure SQL Database from PHP" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 如何从 PHP 访问 Azure SQL Database

本指南将介绍通过 PHP 使用 Azure SQL Database 的基础知识。通过 PHP 编写示例。所涉及的任务包括**创建 SQL Database** 和**连接 SQL Database**。本指南介绍从[管理门户][管理门户]创建 SQL Database 的步骤。有关从生产门户执行这些任务的信息，请参阅 [PHP 和 SQL Database 入门][PHP 和 SQL Database 入门]。有关详细信息，请参阅[后续步骤][后续步骤]一节。

## 什么是 Azure SQL Database

Azure SQL Database 为 Azure 提供关系数据库管理系统并且基于 SQL Server 技术。使用 SQL Database 实例，你可以轻松地配置关系数据库解决方案并将其部署到云中，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 目录

-   [概念][概念]
-   [如何：设置环境][如何：设置环境]
-   [如何：创建 SQL Database][如何：创建 SQL Database]
-   [如何：获取 SQL Database 连接信息][如何：获取 SQL Database 连接信息]
-   [如何：连接到 SQL Database 实例][如何：连接到 SQL Database 实例]
-   [后续步骤][后续步骤]

## <span id="Concepts"></span></a>概念

由于 Azure SQL Database 是基于 SQL Server 技术构建的，因此通过 PHP 访问 SQL Database 与通过 PHP 访问 SQL Server 非常相似。你可以本地部署应用程序（使用 SQL Server），然后通过仅更改连接字符串连接到 SQL Database。但是，SQL Database 和 SQL Server 之间有一些可能影响你的应用程序的差别。有关详细信息，请参阅[指导原则和限制 (SQL Database)][指导原则和限制 (SQL Database)]。

通过 PHP 访问 SQL Database 的推荐方法是使用 [Microsoft Drivers for PHP for SQL Server][Microsoft Drivers for PHP for SQL Server]。（本文中的示例将使用这些驱动程序。）Microsoft Drivers for PHP for SQL Server 只在 Windows 上运行。

## <span id="Setup"></span></a>如何：设置环境

设置部署环境的推荐方法是使用 [Microsoft Web 平台安装程序][Microsoft Web 平台安装程序]。Web 平台安装程序将允许你选择 Web 开发平台的元素，并自动安装和配置这些元素。通过下载 Web 平台安装程序和选择安装 WebMatrix、PHP for WebMatrix 和 SQL Server Express，将为你设置完整的开发环境。

此外，还可以手动设置你的环境：

-   安装 PHP 并配置 IIS：[][]<http://php.net/manual/zh/install.windows.iis7.php></a>。
-   下载并安装 SQL Server Express：[][1]<http://www.microsoft.com/zh-cn/download/details.aspx?id=29062></a>
-   下载并安装 Microsoft Drivers for PHP for SQL Server：[][2]<http://php.net/manual/zh/sqlsrv.requirements.php></a>。

## <span id="CreateServer"></span></a>如何：创建 SQL Database

按照以下步骤创建 Azure SQL Database：

1.  登录到[管理门户][管理门户]。
2.  单击该门户左下角的**“+ 新建”**图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  依次单击“数据服务”、“SQL DATABASE”和“自定义创建”。

    ![自定义创建新的 SQL Database][自定义创建新的 SQL Database]

4.  输入数据库的“名称”的值，选择“版本”（Web 版或企业版），再依次选择数据库的“最大大小”、“排序规则”和“新建 SQL Database 服务器”。单击对话框底部的箭头。（请注意，如果你之前已经创建了 SQL Database，则可以从“选择服务器”下拉菜单中选择一个现有服务器。）

    ![填写 SQL Database 设置][填写 SQL Database 设置]

5.  输入管理员名称和密码（并确认密码），选择你将在其中创建新的 SQL Database 的区域，并选中`Allow Azure Services to access the server` 框。

    ![新建 SQL Database 服务器][新建 SQL Database 服务器]

若要查看服务器和数据库信息，请在管理门户中单击“SQL Database”。随后可以单击“数据库”或“服务器”来查看相关信息。

![查看服务器和数据库信息][查看服务器和数据库信息]

## <span id="ConnectionInfo"></span></a>如何：获取 SQL Database 连接信息

若要获取 SQL Database 连接信息，请在门户中单击“SQL Database”，然后单击该数据库的名称。

![查看数据库信息][查看数据库信息]

然后，单击“查看 ADO.NET、ODBC、PHP 和 JDBC 的 SQL Database 连接字符串”。

![显示连接字符串][显示连接字符串]

在生成的窗口的 PHP 部分，记下 **SERVER**、**DATABASE** 和 **USERNAME** 的值。你的密码将是你创建 SQL Database 时所用的密码。

## <span id="Connect"></span></a>如何：连接到 SQL Database 实例

下面的示例演示如何使用 **SQLSRV** 和 **PDO\_SQLSRV** 扩展来连接到以下名称的 SQL Database：`testdb`. 你将需要上一节中获取的信息。将`SERVER_ID` 替换为你的 10 位数服务器 ID（上一节中获取的 SERVER 值的前 10 个字符），并将正确的值（你的用户名称和密码）分配给`$user` 和 `$pwd` 变量。

##### SQLSRV

    $server = "tcp:<value of SERVER from section above>";
    $user = "<value of USERNAME from section above>"@SERVER_ID;
    $pwd = "password";
    $db = "testdb";

    $conn = sqlsrv_connect($server, array("UID"=>$user, "PWD"=>$pwd, "Database"=>$db));

    if($conn === false){
        die(print_r(sqlsrv_errors()));
    }

##### PDO\_SQLSRV

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

## <span id="NextSteps"></span></a>后续步骤

如前所述，使用 SQL Database 与使用 SQL Server 非常相似。建立与 SQL Database 的连接之后（如上所述），你可以使用 **SQLSRV** 或 **PDO\_SQLSRV** API 来插入、检索、更新和删除数据。有关 **SQLSRV** 和 **PDO\_SQLSRV** API 的信息，请参阅 [Microsoft Drivers for PHP for SQL Server][3] 文档。但是，SQL Database 和 SQL Server 之间存在一些可能影响你的应用程序的差异。有关详细信息，请参阅[指导原则和限制 (SQL Database)][指导原则和限制 (SQL Database)]。

以下位置提供了演示如何在 Azure 上通过 PHP 使用 SQL Database 的示例：<https://github.com/WindowsAzure/azure-sdk-for-php-samples/tree/master/tasklist-sqlazure>。

  [管理门户]: https://manage.windowsazure.cn
  [PHP 和 SQL Database 入门]: http://blogs.msdn.com/b/brian_swan/archive/2010/02/12/getting-started-with-php-and-sql-azure.aspx
  [后续步骤]: #NextSteps
  [概念]: #Concepts
  [如何：设置环境]: #Setup
  [如何：创建 SQL Database]: #CreateServer
  [如何：获取 SQL Database 连接信息]: #ConnectionInfo
  [如何：连接到 SQL Database 实例]: #Connect
  [指导原则和限制 (SQL Database)]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff394102.aspx
  [Microsoft Drivers for PHP for SQL Server]: http://www.microsoft.com/download/en/details.aspx?id=20098
  [Microsoft Web 平台安装程序]: http://go.microsoft.com/fwlink/?LinkId=253447
  []: http://php.net/manual/zh/install.windows.iis7.php
  [1]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29062
  [2]: http://php.net/manual/zh/sqlsrv.requirements.php
  [创建新的 Azure 网站]: ./media/sql-database-php-how-to-use-sql-database/plus-new.png
  [自定义创建新的 SQL Database]: ./media/sql-database-php-how-to-use-sql-database/create_custom_sql_db-2.png
  [填写 SQL Database 设置]: ./media/sql-database-php-how-to-use-sql-database/new-sql-db.png
  [新建 SQL Database 服务器]: ./media/sql-database-php-how-to-use-sql-database/db-server-settings.png
  [查看服务器和数据库信息]: ./media/sql-database-php-how-to-use-sql-database/sql-dbs-portal.png
  [查看数据库信息]: ./media/sql-database-php-how-to-use-sql-database/go-to-db-info.png
  [显示连接字符串]: ./media/sql-database-php-how-to-use-sql-database/show-connection-string-2.png
  [3]: http://msdn.microsoft.com/zh-cn/library/dd638075(SQL.10).aspx
