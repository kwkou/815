<properties linkid="develop-node-how-to-sql-database" urlDisplayName="SQL Database" pageTitle="How to use SQL Database (Node.js) - Azure feature guide" metaKeywords="" description="Learn how to use Azure SQL Database from Node.js." metaCanonical="" services="sql-database" documentationCenter="Node.js" title="How to Access Azure SQL Database from Node.js" authors="larryfr" solutions="" manager="" editor="" />

# 如何从 Node.js 访问 Azure SQL Database

本指南介绍使用 Microsoft 的 Node.JS for SQL Server 驱动程序访问 Azure SQL Database 的基础知识。所涉及的任务包括**创建 SQL Database** 和**连接 SQL Database**。本指南介绍从[预览版管理门户][预览版管理门户]创建 SQL Database 的步骤。

## 目录

-   [概念][概念]
-   [如何：设置环境][如何：设置环境]
-   [如何：创建 SQL Database][如何：创建 SQL Database]
-   [如何：获取 SQL Database 连接信息][如何：获取 SQL Database 连接信息]
-   [如何：连接到 SQL Database 实例][如何：连接到 SQL Database 实例]
-   [Azure 部署注意事项][Azure 部署注意事项]
-   [后续步骤][后续步骤]

## <span id="Concepts"></span></a>概念

### 什么是 Azure SQL Database

Azure SQL Database 为 Azure 提供关系数据库管理系统并且基于 SQL Server 技术。使用 SQL Database 实例，你可以轻松地配置关系数据库解决方案并将其部署到云中，并且利用能够为企业级可用性、可缩放性和安全性提供内置数据保护和自愈优势的分布式数据中心。

## 什么是 Microsoft 的 Node.JS for SQL Server 驱动程序

Microsoft 的 Node.JS for SQL Server 驱动程序允许开发人员从 Node.js 应用程序访问存储在 Microsoft SQL Server 或 Azure SQL Database 中的数据。该驱动程序目前只是一个预览版本；更多的功能待完成后会集成到项目中。有关该驱动程序的详细信息，请参阅 Microsoft 的 Node.JS for SQL Server 驱动程序项目的 [Github 页面][Github 页面]和相关的[维基][维基]页面。

<div class="dev-callout">
<b>说明</b>
<p>Microsoft 的 Node.JS for SQL Server 驱动程序目前只有预览版本，它所依赖的运行时组件仅在 Microsoft Windows 和 Azure 操作系统上可用。</p>
</div>

## <span id="Setup"></span></a>如何：设置环境

### 安装 SQL Server Native Client

用于 Node.js 的 Microsoft SQL Server 驱动程序依赖 SQL Server Native Client。尽管在应用程序部署到 Azure 时 Native Client 会自动出现，但它可能不存在于你的本地开发环境中。你可以从 [Microsoft SQL Server 2012 功能包][Microsoft SQL Server 2012 功能包]下载页面安装 SQL Server Native Client。

<div class="dev-callout">
<b>说明</b>
<p>SQL Server Native Client 只适用于 Microsoft Windows 操作系统。尽管在 Azure 上可以本机方式使用该驱动程序，但如果你是在 Microsoft Windows 以外的操作系统上开发应用程序，则无法使用本文中的信息本地测试你的应用程序。</p>
</div>

### 安装 Node.js

可以从 [][]<http://nodejs.org/#download></a> 安装 Node.js。如果安装包不适用于你的操作系统，你可以从源代码生成 Node.js。

## <span id="CreateServer"></span></a>如何：创建 SQL Database

按照以下步骤创建 Azure SQL Database：

1.  登录[预览版管理门户][预览版管理门户]。
2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  单击“SQL Database”，然后单击“自定义创建”。

    ![自定义创建新的 SQL Database][自定义创建新的 SQL Database]

4.  输入数据库的“名称”的值，选择“版本”（Web 版或企业版），再依次选择数据库的“最大大小”、“排序规则”和“新建 SQL Database 服务器”。单击对话框底部的箭头。（请注意，如果你之前已经创建了 SQL Database，则可以从“选择服务器”下拉菜单中选择一个现有服务器。）

    ![填写 SQL Database 设置][填写 SQL Database 设置]

5.  输入管理员名称和密码（并确认密码），选择你将在其中创建新的 SQL Database 的区域，并选中`Allow Azure Services to access the server` 框。

    ![新建 SQL Database 服务器][新建 SQL Database 服务器]

若要查看服务器和数据库信息，请在预览版管理门户中单击“SQL Database”。随后可以单击“数据库”或“服务器”来查看相关信息。

![查看服务器和数据库信息][查看服务器和数据库信息]

## <span id="ConnectionInfo"></span></a>如何：获取 SQL Database 连接信息

若要获取 SQL Database 连接信息，请在门户中单击“SQL Database”，然后单击该数据库的名称。

![查看数据库信息][查看数据库信息]

然后，单击“显示连接字符串”。

![显示连接字符串][显示连接字符串]

在结果窗口的 ODBC 部分，记下连接字符串的值。从 Node 应用程序连接到 SQL Database 时会用到该连接字符串。你的密码将是你创建 SQL Database 时所用的密码。

## <span id="Connect"></span></a>如何：连接到 SQL Database 实例

### 安装 node-sqlserver

Microsoft 的 Node.JS for SQL Server 驱动程序以 node-sqlserver 本机模块的形式提供。可以从[下载中心][下载中心]获取该模块的二进制版本。若要使用二进制版本，请执行以下步骤：

1.  将二进制存档解压缩到你的应用程序的 **node\_modules** 目录。
2.  运行从存档中解压缩的 **node-sqlserver-install.cmd** 文件。这将在 **node\_modules** 下创建 **node-sqlserver** 子目录并将驱动程序文件移到此新的目录结构中。
3.  删除不再需要的 **node-sqlserver-install.cmd** 文件。

<div class="dev-callout">
<b>说明</b>
<p>你还可以使用 npm 实用工具安装 node-sqlserver 模块；不过这会调用 node-gyp 在你的系统上生成该模块的二进制版本。</p>
</div>

### 指定连接字符串

若要使用 node-sqlserver，你的应用程序中必须包含它并且必须指定连接字符串。连接字符串应该是本文[如何：获取 SQL Database 连接信息][如何：获取 SQL Database 连接信息]一节中返回的 ODBC 值。该代码应类似下面这样：

    var sql = require('node-sqlserver');
    var conn_str = "Driver={SQL Server Native Client 10.0};Server=tcp:{dbservername}.database.chinacloudapi.cn,1433;Database={database};Uid={username};Pwd={password};Encrypt=yes;Connection Timeout=30;";

### 查询数据库

可以使用 **query** 方法，通过指定 Transact-SQL 语句来执行查询。当你查看网页时，以下代码创建 HTTP 服务器并从 **Test** 表的 **ID**、**Column1** 和 **Column2** 行返回数据：

    var http = require('http')
    var port = process.env.port||3000;
    http.createServer(function (req, res) {
        sql.query(conn_str, "SELECT * FROM TestTable", function (err, results) {
            if (err) {
                res.writeHead(500, { 'Content-Type': 'text/plain' });
                res.write("Got error :-( " + err);
                res.end("");
                return;
            }
            res.writeHead(200, { 'Content-Type': 'text/plain' });
            for (var i = 0; i < results.length; i++) {
                res.write("ID: " + results[i].ID + " Column1: " + results[i].Column1 + " Column2: " + results[i].Column2);
            }
            res.end("; Done.");
        });
    }).listen(port);

虽然上面的示例阐释了如何一次性在结果集中返回所有行，但你也可以返回一个允许你订阅事件的语句对象。这样，你就可以接收返回的单个行和列。下面的示例演示如何执行此操作：

    var stmt = sql.query(conn_str, "SELECT * FROM TestTable");
    stmt.on('meta', function (meta) { console.log("We've received the metadata"); });
    stmt.on('row', function (idx) { console.log("We've started receiving a row"); });
    stmt.on('column', function (idx, data, more) { console.log(idx + ":" + data);});
    stmt.on('done', function () { console.log("All done!"); });
    stmt.on('error', function (err) { console.log("We had an error: " + err); });

## <span id="Deploy"></span></a>Azure 部署注意事项

<div class="dev-callout">
<b>说明</b>
<p>以下信息基于 Microsoft 的 Node.JS for SQL Server 驱动程序的预览版本。本节中的信息可能不是有关将 node-sqlserver 模块与 Azure 结合使用的最新信息。有关最新信息，请参阅 Github 上的 <a href="https://github.com/WindowsAzure/node-sqlserver">Microsoft 的 Node.JS for SQL Server 驱动程序项目页</a>。</p>
</div>

Azure 不会在运行时动态安装 node-sqlserver 模块，因此你必须确保你的应用程序部署包括该模块的二进制版本。你可以通过确保以下目录结构存在且包含下面描述的文件，来确认你的部署中确实包含该模块的二进制版本：

    application directory
        node_modules
            node-sqlserver
                lib

**node-sqlserver** 目录应包含一个 **package.json** 文件。**lib** 目录应包含一个 **sql.js** 和一个 **sqlserver.node** 文件，后者是 node-sqlserver 模块的编译形式。

有关将 Node.js 应用程序部署到 Windows Azure 的详细信息，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][创建 Node.js 应用程序并将其部署到 Azure 网站]和 [Node.js 云服务][Node.js 云服务]。

## <span id="NextSteps"></span></a>后续步骤

-   [Microsoft 的 Node.JS for SQL Server 驱动程序介绍][Microsoft 的 Node.JS for SQL Server 驱动程序介绍]
-   [Github.com 上 Microsoft 的 Node.JS for SQL Server 驱动程序][Github 页面]

  [预览版管理门户]: https://manage.windowsazure.cn
  [概念]: #Concepts
  [如何：设置环境]: #Setup
  [如何：创建 SQL Database]: #CreateServer
  [如何：获取 SQL Database 连接信息]: #ConnectionInfo
  [如何：连接到 SQL Database 实例]: #Connect
  [Azure 部署注意事项]: #Deploy
  [后续步骤]: #NextSteps
  [Github 页面]: https://github.com/WindowsAzure/node-sqlserver
  [维基]: https://github.com/WindowsAzure/node-sqlserver/wiki
  [Microsoft SQL Server 2012 功能包]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29065
  []: http://nodejs.org/#download
  [创建新的 Azure 网站]: ./media/sql-database-nodejs-how-to-use/new_website.jpg
  [自定义创建新的 SQL Database]: ./media/sql-database-nodejs-how-to-use/create_custom_sql_db.jpg
  [填写 SQL Database 设置]: ./media/sql-database-nodejs-how-to-use/new-sql-db.png
  [新建 SQL Database 服务器]: ./media/sql-database-nodejs-how-to-use/db-server-settings.png
  [查看服务器和数据库信息]: ./media/sql-database-nodejs-how-to-use/sql-dbs-portal.png
  [查看数据库信息]: ./media/sql-database-nodejs-how-to-use/go-to-db-info.png
  [显示连接字符串]: ./media/sql-database-nodejs-how-to-use/show-connection-string.png
  [下载中心]: http://www.microsoft.com/en-us/download/details.aspx?id=29995
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /en-us/develop/nodejs/tutorials/create-a-website-(mac)/
  [Node.js 云服务]: /en-us/develop/nodejs/tutorials/getting-started/
  [Microsoft 的 Node.JS for SQL Server 驱动程序介绍]: http://blogs.msdn.com/b/sqlphp/archive/2012/06/08/introducing-the-microsoft-driver-for-node-js-for-sql-server.aspx
