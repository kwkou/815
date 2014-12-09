<properties linkid="develop-nodejs-tutorials-web-site-with-mongodb-mongolab" urlDisplayName="Web site with MongoDB" pageTitle="Node.js web site with MongoDB on MongoLab - Azure" metaKeywords="" description="Learn how to create a Node.js Azure Web Site that connects to a MongoDB instance hosted on MongoLab." metaCanonical="" services="web-sites,virtual-machines" documentationCenter="Node.js" title="Create a Node.js Application on Azure with MongoDB using the MongoLab Add-On" authors="larryf" solutions="" manager="" editor="" />

# 使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序

*作者：Eric Sedor，MongoLab*

各位冒险家，大家好！欢迎使用 MongoDB 即服务。在本教程中你将：

1.  [设置数据库][设置数据库] - Azure 应用商店 [MongoLab][MongoLab] 外接程序将为你提供一个托管在 Azure 云中并由 MongoLab 的云数据库平台管理的 MongoDB 数据库。
2.  [创建应用程序][创建应用程序] - 它将是一个用于维护任务列表的简单 Node.js 应用程序。
3.  [部署应用程序][部署应用程序] - 通过将一些配置联系在一起，我们将使推送代码易如反掌。
4.  [管理数据库][管理数据库] - 最后，我们将向你演示 MongoLab 基于 Web 的数据库管理门户，在此你可轻松搜索、显示和修改数据。

在本教程的任意时间，如有任何问题，请随时发送电子邮件至 [support@mongolab.com](mailto:support@mongolab.com)。

在继续之前，请确保你已安装下列项：

-   [Node.js][Node.js] 版本 0.8.14+

-   [Git][Git]

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 快速入门

如果你比较熟悉 Azure 应用商店，请使用本节快速入门。否则，请继续执行下面的[设置数据库][设置数据库]。

1.  打开 Azure 应用商店。
    
    ![应用商店][应用商店]
2.  单击 MongoLab 外接程序。
    
    ![MongoLab][1]
3.  单击“外接程序”列表中你的 MongoLab 外接程序，然后单击“连接信息”。
    
    ![连接信息按钮][连接信息按钮]
4.  将 MONGOLAB\_URI 复制到剪贴板。
    ![连接信息屏幕][连接信息屏幕]

    **此 URI 包含你的数据库用户名和密码。将其视为敏感信息而不要共享它。**
5.  将该值添加到你的 Azure Web 应用程序的“配置”菜单中的“连接字符串”列表：
    
    ![网站连接字符串][网站连接字符串]
6.  对于“名称”，请输入 MONGOLAB\_URI。
7.  对于“值”，请粘贴我们在上一节中获得的连接字符串。
8.  在“类型”下拉列表中选择“自定义”（而不是默认的“SQLAzure”）。
9.  运行`npm install mongoose` 以获取 Mongoose（一个 MongoDB Node 驱动程序）。
10. 在代码中设置挂钩以便从环境变量获得 MongoLab 连接 URI 并连接：

        var mongoose = require('mongoose');  
        ...
        var connectionString = process.env.CUSTOMCONNSTR_MONGOLAB_URI
        ...
        mongoose.connect(connectionString);

注意：Azure 会向最初声明的连接字符串添加 **CUSTOMCONNSTR\_** 前缀，这正是代码引用 **CUSTOMCONNSTR\_MONGOLAB\_URI** 而不是 **MONGOLAB\_URI** 的原因。

现在，开始完整教程...

## <a name="provision"></a>设置数据库

[WACOM.INCLUDE [howto-provision-mongolab](../includes/howto-provision-mongolab.md)]

## <a name="create"></a>创建应用程序

在本节中，你将使用 Node.js、Express 和 MongoDB 设置你的开发环境以及为基本任务列表 Web 应用程序布置代码。[Express][Express] 为 Node 提供视图控制器框架，而 [Mongoose][Mongoose] 是用于在 Node 中与 MongoDB 通信的驱动程序。

### 设置

#### 生成基架和安装模块

1.  在命令行中，创建并导航到 **tasklist** 目录。这将是你的项目目录。
2.  输入以下命令来安装 Express。

        npm install express -g

    `-g` 指示全局模式，将用于提供 **express** 模块，而无需指定目录路径。如果你收到 **Error:EPERM, chmod '/usr/local/bin/express'**，请使用 **sudo** 以更高的权限级别运行 npm。

    此命令的输出看上去应如下所示：

        express@3.3.4 C:\Users\larryfr\AppData\Roaming\npm\node_modules\express
        ├── methods@0.0.1
        ├── fresh@0.1.0
        ├── cookie-signature@1.0.1
        ├── range-parser@0.0.4
        ├── buffer-crc32@0.2.1
        ├── cookie@0.1.0
        ├── debug@0.7.2
        ├── mkdirp@0.3.5
        ├── commander@1.2.0 (keypress@0.1.0)
        ├── send@0.1.3 (mime@1.2.9)
        └── connect@2.8.4 (uid2@0.0.2, pause@0.0.1, qs@0.6.5, bytes@0.2.0, formidable@1.0.14)

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

4.  输入以下命令以安装 **package.json** 文件中描述的模块：

        npm install

    此命令的输出看上去应如下所示：

        express@3.3.4 node_modules\express
        ├── methods@0.0.1
        ├── fresh@0.1.0
        ├── range-parser@0.0.4
        ├── cookie-signature@1.0.1
        ├── buffer-crc32@0.2.1
        ├── cookie@0.1.0
        ├── debug@0.7.2
        ├── mkdirp@0.3.5
        ├── commander@1.2.0 (keypress@0.1.0)
        ├── send@0.1.3 (mime@1.2.9)
        └── connect@2.8.4 (uid2@0.0.2, pause@0.0.1, qs@0.6.5, bytes@0.2.0, formidable@1.0.14)

        jade@0.33.0 node_modules\jade
        ├── character-parser@1.0.2
        ├── mkdirp@0.3.5
        ├── commander@1.2.0 (keypress@0.1.0)
        ├── with@1.1.0 (uglify-js@2.3.6)
        ├── constantinople@1.0.1 (uglify-js@2.3.6)
        ├── transformers@2.0.1 (promise@2.0.0, css@1.0.8, uglify-js@2.2.5)
        └── monocle@0.1.48 (readdirp@0.2.5)

    **package.json** 文件是 **express** 命令创建的文件之一。此文件包含 Express 应用程序所需的其他模块的列表。稍后，在你将此应用程序部署到 Azure 网站时，将使用此文件确定需要在 Azure 上安装哪些模块来支持你的应用程序。

5.  接下来，输入以下命令在本地安装 Mongoose 模块并将它的一个条目保存到 **package.json** 文件：

        npm install mongoose --save

    此命令的输出看上去应如下所示：

        mongoose@3.6.15 node_modules\mongoose
        ├── regexp-clone@0.0.1
        ├── sliced@0.0.3
        ├── muri@0.3.1
        ├── hooks@0.2.1
        ├── mpath@0.1.1
        ├── ms@0.1.0
        ├── mpromise@0.2.1 (sliced@0.0.4)
        └── mongodb@1.3.11 (bson@0.1.9, kerberos@0.0.3)

    你可以放心地忽略有关安装 C++ bson 分析器的任何消息。

### 代码

现在，我们的环境和基架已准备就绪，我们将通过添加一个包含你的任务模型的 **task.js** 文件来扩展 **express** 命令创建的基本应用程序。你还将修改现有 **app.js** 并创建新的 **tasklist.js** 控制器文件以使用该模型。

#### 创建模型

1.  在 **tasklist** 目录中，创建一个名为 **models** 的新目录。

2.  在 **models** 目录中，创建一个名为 **task.js** 的新文件。此文件将包含你的应用程序创建的任务的模型。

3.  将以下代码添加到 **task.js** 文件：

        var mongoose = require('mongoose')
          , Schema = mongoose.Schema;

        var TaskSchema = new Schema({
            itemName      : String
          , itemCategory  : String
          , itemCompleted : { type: Boolean, default: false }
          , itemDate      : { type: Date, default: Date.now }
        });

        module.exports = mongoose.model('TaskModel', TaskSchema);

4.  保存并关闭 **task.js** 文件。

#### 创建控制器

1.  在 **tasklist/routes** 目录中，创建一个名为 **tasklist.js** 的新文件并在文本编辑器中将其打开。

2.  将以下代码添加到 **tasklist.js**。这将加载 mongoose 模块和 **task.js** 中定义的任务模型。TaskList 函数用于根据 **connection** 值创建与 MongoDB 服务器的连接，并提供 **showTasks**、**addTask** 和 **completeTasks** 方法：

        var mongoose = require('mongoose')
          , task = require('../models/task.js');

        module.exports = TaskList;

        function TaskList(connection) {
          mongoose.connect(connection);
        }

        TaskList.prototype = {
          showTasks: function(req, res) {
            task.find({itemCompleted: false}, function foundTasks(err, items) {
              res.render('index',{title: 'My ToDo List ', tasks: items})
            });
          },

          addTask: function(req,res) {
            var item = req.body.item;
            newTask = new task();
            newTask.itemName = item.name;
            newTask.itemCategory = item.category;
            newTask.save(function savedTask(err){
              if(err) {
                throw err;
              }
            });
            res.redirect('/');
          },


          completeTask: function(req,res) {
            var completedTasks = req.body;
            for(taskId in completedTasks) {
              if(completedTasks[taskId]=='true') {
                var conditions = { _id: taskId };
                var updates = { itemCompleted: completedTasks[taskId] };
                task.update(conditions, updates, function updatedTask(err) {
                  if(err) {
                    throw err;
                  }
                });
              }
            }
            res.redirect('/');
          }
        }

3.  保存 **tasklist.js** 文件。

#### 修改索引视图

1.  将目录更改为 **views** 目录并在文本编辑器中打开 **index.jade** 文件。

2.  将 **index.jade** 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

        h1 #{title}
        form(action="/completetask", method="post")
          table(border="1")
            tr
              td Name
              td Category
              td Date
              td Complete
            each task in tasks
              tr
                td #{task.itemName}
                td #{task.itemCategory}
                - var day   = task.itemDate.getDate();
                - var month = task.itemDate.getMonth() + 1;
                - var year  = task.itemDate.getFullYear();
                td #{month + "/" + day + "/" + year}
                td
                  input(type="checkbox", name="#{task._id}", value="#{!task.itemCompleted}", checked=task.itemCompleted)
          input(type="submit", value="Update tasks")
        hr
        form(action="/addtask", method="post")
          table(border="1") 
            tr
              td Item Name: 
              td 
                input(name="item[name]", type="textbox")
            tr
              td Item Category: 
              td 
                input(name="item[category]", type="textbox")
          input(type="submit", value="Add item")

3.  保存并关闭 **index.jade** 文件。

#### 替换 app.js

1.  在 **tasklist** 目录中，用文本编辑器打开 **app.js** 文件。此文件之前是通过运行 **express** 命令创建的。
2.  将以下代码添加到 **app.js** 文件的开头。这将使用 MongoDB 服务器的连接字符串初始化 **TaskList**：

        var TaskList = require('./routes/tasklist');
        var taskList = new TaskList(process.env.CUSTOMCONNSTR_MONGOLAB_URI);

    请注意第二行；你访问将在以后配置的环境变量，该变量包含针对你的 mongo 实例的连接信息。如果你出于开发用途正在运行一个本地 mongo 实例，则可能要暂时将此值设置为“localhost”，而不是 `process.env.CUSTOMCONNSTR_MONGOLAB_URI`。

3.  找到以`app.get` 开头的行并且用以下行替换它们：

        app.get('/', taskList.showTasks.bind(taskList));
        app.post('/addtask', taskList.addTask.bind(taskList));
        app.post('/completetask', taskList.completeTask.bind(taskList));

    这将添加在 **tasklist.js** 中定义为路由的函数。

4.  保存 **app.js** 文件。

## <a name="deploy"></a>部署应用程序

现在，应用程序已开发完毕，是时候创建一个用于托管该应用程序的 Azure 网站，配置该网站，并部署代码了。本节的关键是使用 MongoDB 连接字符串 (URI)。你将使用此 URI 在你的网站中配置环境变量以便将该 URI 与你的代码分开。你应将该 URI 视为敏感信息，因为它包含用于连接到你的数据库的凭据。

本节中的步骤使用 Azure 命令行工具创建一个新的 Azure 网站，然后使用 Git 部署你的应用程序。若要执行这些步骤，你必须具有 Azure 订阅。

### 安装适用于 Mac 和 Linux 的 Azure 命令行工具

若要安装命令行工具，请使用以下命令：

    npm install azure-cli -g

如果你已从 [Azure 开发人员中心][Azure 开发人员中心]安装 **Azure SDK for Node.js**，则应该已安装命令行工具。有关详细信息，请参阅[适用于 Mac 和 Linux 的 Azure 命令行工具][适用于 Mac 和 Linux 的 Azure 命令行工具]。

虽然 Azure 命令行工具主要针对 Mac 和 Linux 用户而创建，但它们基于 Node.js，应该可在能够运行 Node 的任何系统上使用。

### 导入发布设置

在将命令行工具与 Azure 一起使用之前，你必须首先下载包含有关你的订阅的信息的文件。执行以下步骤以下载并导入该文件。

1.  从命令行中，输入以下命令以启动浏览器并导航到下载页面。如果出现提示，请使用与你的订阅关联的帐户进行登录。

        azure account download

    ![下载页面][下载页面]

    文件下载应该会自动开始；如果没有自动开始，你可以单击该页面开头的链接手动下载文件。

2.  文件下载完成后，请使用以下命令导入设置：

        azure account import <path-to-file>

    指定你在上一步中下载的发布设置文件的路径和文件名。命令完成后，你应该会看到与下面类似的输出：

        info:   Executing command account import
        info:   Found subscription: subscriptionname
        info:   Setting default subscription to: subscriptionname
        warn:   The '/Users/user1/.azure/publishSettings.xml' file contains sensitive information.
        warn:   Remember to delete it now that it has been imported.
        info:   Account publish settings imported successfully
        info:   account import command OK

3.  在导入完成后，你应删除发布设置文件，因为不再需要该文件并且它包含有关你的 Azure 订阅的敏感信息。

### 创建新网站并推送你的代码

在 Azure 中创建网站非常简单。如果这是你的第一个 Azure 网站，则你必须使用门户。如果你已拥有至少一个 Azure 网站，则跳到步骤 7。

1.  在 Azure 门户中，单击“新建”。
     
    ![新建][新建]
2.  选择“计算”\>“网站”\>“快速创建”。
      
    ![创建网站][创建网站]
3.  输入 URL 前缀。选择你喜欢的名称，但要注意这必须是唯一的（可能无法使用“mymongoapp”）。
4.  单击“创建网站”。
5.  网站创建完成时，单击网站列表中的网站名称。将显示网站仪表板。
    
    ![网站仪表板][网站仪表板]
6.  单击“速览”下的“设置 Git 发布”，然后输入你选择的 Git 用户名和密码。在推送到你的网站时，将会用到此密码（在步骤 9 中）。
7.  如果你使用上述步骤创建了网站，以下命令将完成该过程。但是，如果你已拥有多个 Azure 网站，则可以跳过上述步骤并使用这一相同命令创建新的网站。从你的 **tasklist** 项目目录中：

        azure site create myuniquesitename --git  

    将“myuniquesitename”替换为你的网站的唯一站点名称。如果网站作为此命令的一部分创建，则系统将提示你输入该网站将位于的数据中心。选择在地理上接近你的 MongoLab 数据库的数据中心。

    该`--git` 参数将创建：
    A. **tasklist** 文件夹中的一个本地 Git 存储库（如果尚不存在）。
    A. 一个名为“azure”的 [Git remote][Git remote]，用于将应用程序发布到 Azure。
    A. 一个 [iisnode.yml][iisnode.yml] 文件，其中包含 Azure 用于托管 Node 应用程序的设置。
    A. 一个阻止将 node-modules 文件夹发布到 .git 的 .gitignore 文件。

    此命令完成后，你将看到与下面类似的输出。请注意，以 **Created web site at** 开头的行包含网站的 URL。

        info:   Executing command site create
        info:   Using location southcentraluswebspace
        info:   Executing `git init`
        info:   Creating default web.config file
        info:   Creating a new web site
        info:   Created web site at  mongodbtasklist.azurewebsites.net
        info:   Initializing repository
        info:   Repository initialized
        info:   Executing `git remote add azure http://gitusername@myuniquesitename.azurewebsites.net/mongodbtasklist.git`
        info:   site create command OK

8.  使用以下命令将文件添加然后提交到你的本地 Git 存储库：

        git add .
        git commit -m "adding files"

9.  推送代码：

        git push azure master  

    在将最新 Git 存储库更改推送到 Azure 网站时，你必须指定目标分支为 **master**，因为这将用于网站内容。如果系统提示你输入密码，请输入你在前面为网站设置 Git 发布时创建的密码。

    你将看到与下面类似的输出。在进行部署时，Azure 将下载所有 npm 模块。

        Counting objects: 17, done.
        Delta compression using up to 8 threads.
        Compressing objects: 100% (13/13), done.
        Writing objects: 100% (17/17), 3.21 KiB, done.
        Total 17 (delta 0), reused 0 (delta 0)
        remote: New deployment received.
        remote: Updating branch 'master'.
        remote: Preparing deployment for commit id 'ef276f3042'.
        remote: Preparing files for deployment.
        remote: Running NPM.
        ...
        remote: Deploying Web.config to enable Node.js activation.
        remote: Deployment successful.
        To https://username@mongodbtasklist.azurewebsites.net/MongoDBTasklist.git
         * [new branch]      master -> master

即将完成！

### 配置环境

还记得代码中的 process.env.CUSTOMCONNSTR\_MONGOLAB\_URI 吗？我们需要使用你在配置 MongoLab 数据库期间提供给 Azure 的值填充该环境变量。

#### 获取 MongoLab 连接字符串

[WACOM.INCLUDE [howto-get-connectioninfo-mongolab](../includes/howto-get-connectioninfo-mongolab.md)]

#### 将该连接字符串添加到网站的环境变量中

[WACOM.INCLUDE [howto-save-connectioninfo-mongolab](../includes/howto-save-connectioninfo-mongolab.md)]

## 成功！

从你的项目目录运行`azure site browse` 以便自动打开浏览器，或者打开浏览器并手动导航到你的网站 URL (myuniquesite.azurewebsites.net)：

![显示空白 tasklist 的网页][显示空白 tasklist 的网页]

## <a name="manage"></a>管理数据库

[WACOM.INCLUDE [howto-access-mongolab-ui](../includes/howto-access-mongolab-ui.md)]

祝贺你！你刚刚启动了由 MongoLab 托管的 MongoDB 数据库提供支持的 Node.js 应用程序！现在，你拥有了一个 MongoLab 数据库，如有任何关于你的数据库的问题或疑虑，或者要获得有关 MongoDB 或 Node 驱动程序本身的帮助，请联系 [support@mongolab.com](mailto:support@mongolab.com)。祝你好运！

  [设置数据库]: #provision
  [MongoLab]: http://mongolab.com
  [创建应用程序]: #create
  [部署应用程序]: #deploy
  [管理数据库]: #manage
  [\<a href="mailto:support@mongolab.com"\>support@mongolab.com\</a\>]: mailto:support@mongolab.com
  [Node.js]: http://nodejs.org
  [Git]: http://git-scm.com
  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  [应用商店]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-store.png
  [1]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/entry-mongolab.png
  [连接信息按钮]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-connectioninfo.png
  [连接信息屏幕]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/dialog-mongolab_connectioninfo.png
  [网站连接字符串]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/focus-mongolab-websiteconnectionstring.png
  [howto-provision-mongolab]: ../includes/howto-provision-mongolab.md
  [Express]: http://expressjs.com
  [Mongoose]: http://mongoosejs.com
  [Azure 开发人员中心]: /zh-cn/develop/nodejs/
  [适用于 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/develop/nodejs/how-to-guides/command-line-tools/
  [下载页面]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/azure-account-download-cli.png
  [新建]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-new.png
  [创建网站]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/screen-mongolab-newwebsite.png
  [网站仪表板]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/screen-mongolab-websitedashboard.png
  [Git remote]: http://git-scm.com/docs/git-remote
  [iisnode.yml]: https://github.com/WindowsAzure/iisnode/blob/master/src/samples/configuration/iisnode.yml
  [howto-get-connectioninfo-mongolab]: ../includes/howto-get-connectioninfo-mongolab.md
  [howto-save-connectioninfo-mongolab]: ../includes/howto-save-connectioninfo-mongolab.md
  [显示空白 tasklist 的网页]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/todo_list_noframe.png
  [howto-access-mongolab-ui]: ../includes/howto-access-mongolab-ui.md
