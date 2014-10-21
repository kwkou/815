<properties linkid="develop-nodejs-tutorials-web-site-with-sql-database" urlDisplayName="Web site with SQL Database" pageTitle="Node.js web site with SQL Database - Azure tutorial" metaKeywords="" description="Learn how to create a Node.js website that accesses a SQL Database and is deployed to Azure" metaCanonical="" services="web-sites,sql-database" documentationCenter="Node.js" title="Node.js Web Application using the Azure SQL Database" authors="larryfr" solutions="" manager="" editor="" />

# 使用 Azure SQL Database 创建 Node.js Web 应用程序

本教程向你演示如何使用 Azure 数据管理提供的 SQL Database，在 Azure 上托管的 [Node][Node] 应用程序中存储和访问数据。本教程假定你之前有使用 Node 和 [Git][Git] 的经验。

你将了解到以下内容：

-   如何使用 Azure 管理门户创建 Azure 网站和 SQL Database

-   如何使用 npm（Node 包管理器）安装 Node 模块

-   如何结合使用 SQL Database 与 node-sqlserver 模块

-   如何使用应用程序设置指定应用程序的运行时值

按照本教程中的说明操作，你将构建一个简单的基于 Web 的任务管理应用程序，该应用程序可用于创建、检索和完成任务。这些任务存储在 SQL Database 中。

本教程中的项目文件将存储在名为 **tasklist** 的目录中，已完成的应用程序将与下图类似：

![显示空白 tasklist 的网页][显示空白 tasklist 的网页]

<div class="dev-callout">
<b>说明</b>
<p>本教程中使用的 Microsoft Driver for Node.JS for SQL Server 目前只有预览版本，它所依赖的运行时组件仅在 Microsoft Windows 和 Azure 操作系统上可用。</p>
</div>

<div class="dev-callout">
<strong>说明</strong>
<p>本教程引用 <strong>tasklist</strong> 文件夹。将省略此文件夹的完整路径，因为路径语义在操作系统之间并不相同。你应在本地文件系统中易于访问的位置创建此文件夹（例如 <strong>~/node/tasklist</strong> 或 <strong>c:\node\tasklist</strong>）</p>
</div>

<div class="dev-callout">
<strong>说明</strong>
<p>下面的许多步骤都提到使用命令行。对于这些步骤，请使用适用于你的操作系统的命令行，例如 <strong>cmd.exe</strong> (Windows) 或 <strong>Bash</strong> (Unix Shell)。在 OS X 系统上，可以通过 Terminal 应用程序访问命令行。</p>
</div>

## 先决条件

在按照本文中的说明操作之前，你应确保已安装下列项：

-   [Node][Node] 0.6.14 或更高版本

-   [Git][Git]

-   Microsoft SQL Server 本机客户端库 - 作为 [Microsoft SQL Server 2012 功能包][Microsoft SQL Server 2012 功能包]的一部分提供

-   文本编辑器

-   Web 浏览器

<!--div chunk="../../Shared/Chunks/create-account-and-websites-note.md" /-->

## 创建网站和数据库

按照以下步骤创建 Azure 网站和 SQL Database：

1.  登录到 [Azure 管理门户][Azure 管理门户]。
2.  单击该门户左下角的“+ 新建”图标。

    ![创建新的 Azure 网站][创建新的 Azure 网站]

3.  单击“网站”，然后单击“自定义创建”。

    ![自定义创建新的网站][自定义创建新的网站]

    输入“URL”的值，从“数据库”下拉列表中选择“新建 SQL Database”，并在“区域”下拉列表中选择网站的数据中心。单击对话框底部的箭头。

    ![填写网站详细信息][填写网站详细信息]

4.  输入数据库的“名称”的值，选择“版本”[（Web 版或企业版）][（Web 版或企业版）]，再依次选择数据库的“最大大小”、“排序规则”和“新建 SQL Database 服务器”。单击对话框底部的箭头。

    ![填写 SQL Database 设置][填写 SQL Database 设置]

5.  输入管理员名称和密码（并确认密码），选择你将在其中创建新的 SQL Database 服务器的区域，选中“允许 Azure 服务访问服务器”框。

    ![新建 SQL Database 服务器][新建 SQL Database 服务器]

    创建网站后，你会看到文本“网站‘[SITENAME]’创建已成功完成”。现在，你可以启用 Git 发布。

6.  单击网站列表中显示的网站的名称以打开该网站的“快速启动”仪表板。

    ![打开网站仪表板][打开网站仪表板]

7.  在“快速启动”页的底部，单击“设置 Git 发布”。

    ![设置 Git 发布][设置 Git 发布]

8.  若要启用 Git 发布，你必须提供用户名和密码。记下你创建的用户名和密码。（如果你之前已设置 Git 存储库，则将跳过此步骤。）

    ![创建发布凭据][创建发布凭据]

    设置存储库需要花费几秒钟的时间。

9.  在你的存储库已就绪后，将显示有关将应用程序文件推送到存储库的说明。记下这些说明 - 稍后你将使用它们。

    ![Git 说明][Git 说明]

## 获取 SQL Database 连接信息

若要连接到正在 Azure 网站中运行的 SQL Database 实例，你将需要连接信息。若要获取 SQL Database 连接信息，请按照以下步骤操作：

1.  从 Azure 管理门户中，单击“链接的资源”，然后单击数据库名称。

    ![链接的资源][链接的资源]

2.  单击“查看连接字符串”。

    ![连接字符串][连接字符串]

3.  从结果对话框的“ODBC”部分，记下稍后将要使用到的连接字符串。

## 设计任务表

若要创建用于存储 tasklist 应用程序项的数据库表，请执行下列步骤：

1.  从 Azure 管理门户中，选择你的 SQL Database，然后单击该页面底部的“管理”。如果你收到一条内容为当前 IP 不属于防火墙规则的消息，请选择“确定”以添加该 IP 地址。

    ![“管理”按钮][“管理”按钮]

2.  使用你前面在创建数据库服务器时选择的登录名和密码进行登录。

    ![数据库管理登录][数据库管理登录]

3.  从页面左下角，选择“设计”，然后选择“新建表”。

    ![新建表][新建表]

4.  输入“tasks”作为“表名”，选中“ID”列的“是否标识?”。

    ![表名设置为 tasks 且已选中“是否标识”][表名设置为 tasks 且已选中“是否标识”]

5.  将“Column1”更改为“名称”，将“Column2”更改为“类别”。通过单击“添加列”按钮来添加两个新列。将第一个新列命名为“已创建”且类型为“date”。将第二个新列命名为“已完成”且类型为“bit”。这两个新列都应标记“是否必需?”。

    ![表设计已完成][表设计已完成]

6.  单击“保存”按钮保存对表所做的更改。现在，你可以关闭 SQL Database 管理页面。

## 安装模块并生成基架

在本节中，你将创建一个新的 Node 应用程序并使用 npm 添加模块包。对于任务列表应用程序，你将使用 [express][express] 和 [node-sqlserver][node-sqlserver] 模块。Express 模块为 Node 提供“模型视图控制器”框架，而 node-sqlserver 模块提供与 Azure SQL Database 的连接。

### 安装 Express 并生成基架

1.  在命令行中，将目录更改为 **tasklist** 目录。如果 **tasklist** 目录不存在，请创建该目录。

2.  输入以下命令来安装 Express。

        npm install express -g

    <div class="dev-callout">
<strong>说明</strong>
<p>在某些操作系统上使用&ldquo;-g&rdquo;参数时，你可能会收到一条错误 <strong>Error:EPERM, chmod '/usr/local/bin/express'</strong> 和一个尝试以管理员身份运行帐户的请求。如果发生这种情况，请使用 <strong>sudo</strong> 命令以更高的权限级别运行 npm。</p>
</div>

    此命令的输出看上去应如下所示：

        express@2.5.9 /usr/local/lib/node_modules/express
        ├── mime@1.2.4
        ├── mkdirp@0.3.0
        ├── qs@0.4.2
        └── connect@1.8.7

    <div class="dev-callout">
<strong>说明</strong>
<p>在安装 Express 模块时使用&ldquo;-g&rdquo;参数将全局安装该模块。这样我们就可以访问 <strong>express</strong> 命令以生成网站基架，而无需键入其他路径信息。</p>
</div>

3.  若要创建将用于此应用程序的基架，请使用 **express** 命令：

        express

    此命令的输出看上去应如下所示：

        create : .
        create : ./package.json
        create : ./app.js
        create : ./public
        create : ./public/javascripts
        create : ./public/images
        create : ./public/stylesheets
        create : ./public/stylesheets/style.css
        create : ./routes
        create : ./routes/index.js
        create : ./views
        create : ./views/layout.jade
        create : ./views/index.jade

        dont forget to install dependencies:
        $ cd . && npm install

    此命令完成后，你应当会在 **tasklist** 目录中拥有几个新目录和文件。

### 安装其他模块

1.  从命令行中，将目录更改为 **tasklist** 文件夹，然后输入以下命令安装 **package.json** 文件中描述的模块：

        npm install

    此命令的输出看上去应如下所示：

        express@2.5.8 ./node_modules/express
        ├── mime@1.2.4
        ├── qs@0.4.2
        ├── mkdirp@0.3.0
        └── connect@1.8.7
        jade@0.26.0 ./node_modules/jade
        ├── commander@0.5.2
        └── mkdirp@0.3.0

    这将安装 Express 需要的所有默认模块。

2.  接下来，使用以下命令添加 nconf 模块。此模块将由应用程序用来从配置文件中读取数据库连接字符串。

    npm install nconf -save

3.  接下来，从[下载中心][下载中心]下载 Microsoft Driver for Node.JS for SQL Server 的二进制版本。

4.  将存档解压缩到 **tasklist\\node\_modules** 目录。

5.  运行 **tasklist\\node\_modules** 目录中的 **msnodesql-install.cmd** 文件。这将在 **node\_modules** 下创建 **msnodesql** 子目录，并将驱动程序文件移到此新的目录结构中。

6.  如果不再需要 **msnodesql-install.cmd** 文件，则将其删除。

## 在 Node 应用程序中使用 SQL Database

在此部分，你将通过修改现有 **app.js** 来扩展 **express** 命令创建的基本应用程序，并创建一个新 **index.js** 文件来使用前面创建的数据库。

### 修改控制器

1.  在 **tasklist/routes** 目录中，用文本编辑器打开 **index.js** 文件。

2.  将 **index.js** 文件中的现有代码替换为以下代码。这将加载 msnodesql 和 nconf 模块，然后使用 nconf 从名为 **SQL\_CONN** 的环境变量或 **config.json** 文件中的 **SQL\_CONN** 值中加载连接字符串。

        var sql = require('msnodesql')
            , nconf = require('nconf');

        nconf.env()
             .file({ file: 'config.json' });
        var conn = nconf.get("SQL_CONN");

3.  继续通过添加 **index** 和 **updateItem** 方法来添加到 **index.js** 文件。**index** 方法会从数据库中返回所有未完成的任务，而 **updateItem** 会将选中的任务标记为已完成。

        exports.index = function(req, res) {
            var select = "select * from tasks where completed = 0";
            sql.query(conn, select, function(err, items) {
                if(err)
                    throw err;
                res.render('index', { title: 'My ToDo List ', tasks: items });
            });
        };

        exports.updateItem = function(req, res) {
            var item = req.body.item;
            if(item) {
                var insert = "insert into tasks (name, category, created, completed) values (?, ?, GETDATE(), 0)";
                sql.query(conn, insert, [item.name, item.category], function(err) {
                    if(err)
                        throw err;
                    res.redirect('/');
                });
            } else {
                var completed = req.body.completed;
                if(!completed.forEach)
                    completed = [completed];
                var update = "update tasks set completed = 1 where id in (" + completed.join(",") + ")";
                sql.query(conn, update, function(err) {
                    if(err)
                        throw err;
                    res.redirect('/');
                });
            }
        }

4.  保存 **index.js** 文件。

### 修改 app.js

1.  在 **tasklist** 目录中，用文本编辑器打开 **app.js** 文件。此文件之前是通过运行 **express** 命令创建的。

2.  在 app.js 文件中，向下滚动到以下代码位置。

        app.configure('development', function(){
        app.use(express.errorHandler());
        });

3.  现在插入以下代码。

        app.get('/', routes.index);
        app.post('/', routes.updateItem);

这样会将新路由添加到你之前在 **index.js** 文件中添加的 **updateItem** 方法中。

4.  保存 **app.js** 文件。

### 修改索引视图

1.  将目录更改为 **views** 目录并在文本编辑器中打开 **index.jade** 文件。

2.  将 **index.jade** 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

        h1= title
        br

        form(action="/", method="post")
          table(class="table table-striped table-bordered")
            thead
              tr
                td Name
                td Category
                td Date
                td Complete
            tbody
            each task in tasks
              tr
                td #{task.name}
                td #{task.category}
                td #{task.created}
                td
                  input(type="checkbox", name="completed", value="#{task.ID}", checked=task.completed == 1)
          button(type="submit", class="btn") Update tasks
        hr

        form(action="/", method="post", class="well")
          label Item Name:
          input(name="item[name]", type="textbox")
          label Item Category:
          input(name="item[category]", type="textbox")
          br
          button(type="submit", class="btn") Add Item

3.  保存并关闭 **index.jade** 文件。

### 修改全局布局

**views** 目录中的 **layout.jade** 文件用作其他 **.jade** 文件的全局模板。在此步骤中，你将对其进行修改以使用 [Twitter Bootstrap][Twitter Bootstrap]（一个可以轻松设计美观网站的工具包）。

1.  下载并提取 [Twitter Bootstrap][1] 的文件。将 **bootstrap.min.css** 文件从 **bootstrap\\css** 文件夹复制到你的 tasklist 应用程序的 **public\\stylesheets** 目录中。

2.  从 **views** 文件夹中，用文本编辑器打开 **layout.jade** 并将其内容替换为以下代码：

        !!!html
        html
          head
            title= title
            meta(http-equiv='X-UA-Compatible', content='IE=10')
            link(rel='stylesheet', href='/stylesheets/style.css')
            link(rel='stylesheet', href='/stylesheets/bootstrap.min.css')
          body(class='app')
            div(class='navbar navbar-fixed-top')
              .navbar-inner
                .container
                  a(class='brand', href='/') My Tasks
            .container!= body

3.  保存 **layout.jade** 文件。

### 创建配置文件

**config.json** 文件包含用于连接到 SQL Database 的连接字符串，并在运行时由 **index.js** 文件进行读取。若要创建该文件，请执行以下步骤：

1.  在 **tasklist** 目录中，创建一个名为 **config.json** 的新文件并在文本编辑器中将其打开。

2.  **config.json** 文件的内容看上去应如下所示：

        {
          "SQL_CONN" : "connection_string"
        }

    将 **connection\_string** 替换为前面返回的 ODBC 连接字符串值。

3.  保存文件。

## 在本地运行应用程序

若要在你的本地计算机中测试应用程序，请执行以下步骤：

1.  在命令行中，将目录更改为 **tasklist** 目录。

2.  使用以下命令在本地启动应用程序：

        node app.js

3.  打开 Web 浏览器并导航到 <http://127.0.0.1:3000>。此时会显示与下图类似的网页：

    ![显示空白 tasklist 的网页][2]

4.  使用提供的 **Item Name**（项名称）和 **Item Category**（项类别）字段输入信息，然后单击 **Add item**（添加项）。

5.  页面应更新为在 ToDo List 中显示该项。

    ![任务列表中新项的图像][任务列表中新项的图像]

6.  若要完成任务，只需选中“Complete”（完成）列中的复选框，然后单击 **Update tasks**（更新任务）。

7.  若要停止 Node 进程，请转到命令行并按 **Ctrl** 和 **C** 键。

## 将你的应用程序部署到 Azure

在此部分，你将使用在创建网站后接收到的部署步骤来将应用程序发布到 Azure。

### 发布应用程序

1.  从命令行上，将目录改为 **tasklist** 目录（如果你尚未在此目录中）。

2.  使用下列命令初始化应用程序的本地 Git 存储库、向其中添加应用程序文件，最后将文件推送到 Azure

        git init
        git add .
        git commit -m "adding files"
        git remote add azure [URL for remote repository]
        git push azure master

    在部署结束时，你将看到如下语句：

        To https://username@tabletasklist.chinacloudsites.cn/TableTasklist.git
         * [new branch]      master -> master

3.  在完成推送操作后，请浏览到 **[http://[site][http://[site] name].chinacloudsites.cn/** 查看你的应用程序。

### 切换到环境变量

前面我们实现了在 **SQL\_CONN** 环境变量中查找连接字符串或从 **config.json** 文件加载该值的代码。在接下来的步骤中，你将在网站配置中创建应用程序通过环境变量实际访问的键/值对。

1.  从管理门户中，单击“网站”，然后选择你的网站。

    ![打开网站仪表板][打开网站仪表板]

2.  单击**“配置”**，然后找到页面的**“应用程序设置”**部分。

    ![配置链接][配置链接]

3.  在“应用程序设置”部分，在“键”字段中输入 **SQL\_CONN**，在“值”字段中输入 ODBC 连接字符串。最后，单击复选标记。

    ![应用程序设置][应用程序设置]

4.  最后，单击页面底部的**“保存”**图标，将此更改提交到运行时环境。

    ![保存应用程序设置][保存应用程序设置]

5.  从命令行中，将目录更改为 **tasklist** 目录，然后输入以下命令以删除 **config.json** 文件：

        git rm config.json
        git commit -m "Removing config file"

6.  执行以下命令将更改部署到 Azure：

        git push azure master

在将更改部署到 Azure 后，你的 Web 应用程序应当继续工作，因为它现在从**“应用程序设置”**条目读取连接字符串。若要对此进行验证，请在“应用程序设置”中将 **SQL\_CONN** 条目的值更改为一个无效值。在你保存该值后，网站会因连接字符串无效而失败。

## 后续步骤

-   [使用 MongoDB 构建 Node.js Web 应用程序][使用 MongoDB 构建 Node.js Web 应用程序]

-   [使用表存储构建 Node.js Web 应用程序]

## 其他资源


   [适用于 Mac 和 Linux 的 Azure 命令行工具][适用于 Mac 和 Linux 的 Azure 命令行工具] 
   [创建 Node.js 应用程序并将其部署到 Azure 网站](/en-us/develop/nodejs/tutorials/create-a-website-(mac))
   [使用 Git 发布到 Azure 网站](/en-us/develop/nodejs/common-tasks/publishing-with-git/)
   [Azure 开发人员中心](/en-us/develop/nodejs/)
   [使用表存储构建 Node.js Web 应用程序](/en-us/develop/nodejs/tutorials/web-site-with-storage/)


<!-- URLs. -->

  [Node]: http://nodejs.org
  [Git]: http://git-scm.com
  [显示空白 tasklist 的网页]: ./media/sql-database-nodejs-use-web-site/sql_todo_final.png
  [Microsoft SQL Server 2012 功能包]: http://www.microsoft.com/zh-cn/download/details.aspx?id=29065
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [创建新的 Azure 网站]: ./media/sql-database-nodejs-use-web-site/new_website.jpg
  [自定义创建新的网站]: ./media/sql-database-nodejs-use-web-site/custom_create.png
  [填写网站详细信息]: ./media/sql-database-nodejs-use-web-site/website_details_sqlazure.jpg
  [（Web 版或企业版）]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee621788.aspx
  [填写 SQL Database 设置]: ./media/sql-database-nodejs-use-web-site/database_settings.jpg
  [新建 SQL Database 服务器]: ./media/sql-database-nodejs-use-web-site/create_server.jpg
  [打开网站仪表板]: ./media/sql-database-nodejs-use-web-site/go_to_dashboard.png
  [设置 Git 发布]: ./media/sql-database-nodejs-use-web-site/setup_git_publishing.png
  [创建发布凭据]: ./media/sql-database-nodejs-use-web-site/git-deployment-credentials.png
  [Git 说明]: ./media/sql-database-nodejs-use-web-site/git-instructions.png
  [链接的资源]: ./media/sql-database-nodejs-use-web-site/linked_resources.jpg
  [连接字符串]: ./media/sql-database-nodejs-use-web-site/connection_string.jpg
  [“管理”按钮]: ./media/sql-database-nodejs-use-web-site/sql-manage.png
  [数据库管理登录]: ./media/sql-database-nodejs-use-web-site/sqlazurelogin.png
  [新建表]: ./media/sql-database-nodejs-use-web-site/new-table.png
  [表名设置为 tasks 且已选中“是否标识”]: ./media/sql-database-nodejs-use-web-site/table-name-identity.png
  [表设计已完成]: ./media/sql-database-nodejs-use-web-site/table-columns.png
  [express]: http://expressjs.com
  [node-sqlserver]: https://github.com/WindowsAzure/node-sqlserver
  [下载中心]: http://www.microsoft.com/en-us/download/details.aspx?id=29995
  [Twitter Bootstrap]: https://github.com/twbs/bootstrap
  [1]: http://getbootstrap.com/
  [2]: ./media/sql-database-nodejs-use-web-site/sql_todo_empty.png
  [任务列表中新项的图像]: ./media/sql-database-nodejs-use-web-site/sql_todo_list.png
  [http://[site]: http://[site
  [配置链接]: ./media/sql-database-nodejs-use-web-site/sql-task-configure.png
  [应用程序设置]: ./media/sql-database-nodejs-use-web-site/appsettings.png
  [保存应用程序设置]: ./media/sql-database-nodejs-use-web-site/savebutton.png
  [使用 MongoDB 构建 Node.js Web 应用程序]: ../store-mongolab-web-sites-nodejs-store-data-mongodb/
  [使用 Git 发布到 Azure 网站]: ../CommonTasks/publishing-with-git
  [适用于 Mac 和 Linux 的 Azure 命令行工具]: /en-us/develop/nodejs/how-to-guides/command-line-tools/
