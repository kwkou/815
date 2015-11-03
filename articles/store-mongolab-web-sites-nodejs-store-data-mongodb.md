<properties
	pageTitle="使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js Web 应用"
	description="了解如何在 Azure App Service 中创建 Node.js Web 应用，并且该应用会连接到 MongoLab 上托管的 MongoDB 实例。"
	tags="azure-classic-portal"
	services="app-service\web, virtual-machines"
	documentationCenter="nodejs"
	authors="tdykstra"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="08/18/2015"
	wacn.date="10/03/2015"/>






# 使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js Web 应用


*作者：Eric Sedor，MongoLab*

各位冒险家，大家好！ 欢迎使用 MongoDB 即服务。在本教程中你将：

2. [创建应用][create] - 它将是一个简单的 Node.js 应用，用于维护任务列表。
3. [部署应用][deploy] - 通过将一些配置联系在一起，我们轻而易举就能将代码推送到 [Azure 网站](/documentation/services/web-sites/)。
4. [管理数据库][manage] - 最后，我们将向你演示 MongoLab 基于 Web 的数据库管理门户，在此你可轻松搜索、显示和修改数据。

在本教程的任意时间，如有任何问题，请随时发送电子邮件至 [support@mongolab.com](mailto:support@mongolab.com)。

在继续之前，请确保你已安装下列项：

* [Node.js] 版本 0.10.29+

* [Git]

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

<a name="provision"></a>
## 设置数据库

[AZURE.INCLUDE [howto-provision-mongolab](../includes/howto-provision-mongolab.md)]

<a name="create"></a>
## 创建应用程序

在本节中，你将使用 Node.js、Express 和 MongoDB 设置你的开发环境以及为基本任务列表 Web 应用程序布置代码。[Express] 为 node 提供视图控制器框架，而 [Mongoose] 是用于在 node 中与 MongoDB 通信的驱动程序。

### 设置

#### 生成基架和安装模块

1. 在命令行中，创建并导航到 **tasklist** 目录。这将是你的项目目录。
2. 输入以下命令来安装 Express。

		npm install express -g

	`-g` 指示全局模式，将用于提供 <strong>express</strong> 模块，而无需指定目录路径。如果收到 <strong>错误：EPERM，chmod '/usr/local/bin/express'</strong>，请使用 <strong>sudo</strong> 以更高的权限级别运行 npm。

    此命令的输出看上去应如下所示：

		express@4.9.1 C:\Users\mongolab\AppData\Roaming\npm\node_modules\express
		├── merge-descriptors@0.0.2
		├── utils-merge@1.0.0
		├── fresh@0.2.4
		├── cookie@0.1.2
		├── range-parser@1.0.2
		├── escape-html@1.0.1
		├── cookie-signature@1.0.5
		├── finalhandler@0.2.0
		├── vary@1.0.0
		├── media-typer@0.3.0
		├── parseurl@1.3.0
		├── serve-static@1.6.2
		├── methods@1.1.0
		├── path-to-regexp@0.1.3
		├── depd@0.4.5
		├── qs@2.2.3
		├── on-finished@2.1.0 (ee-first@1.0.5)
		├── debug@2.0.0 (ms@0.6.2)
		├── proxy-addr@1.0.1 (ipaddr.js@0.1.2)
		├── etag@1.3.1 (crc@3.0.0)
		├── send@0.9.2 (destroy@1.0.3, ms@0.6.2, mime@1.2.11)
		├── accepts@1.1.0 (negotiator@0.4.7, mime-types@2.0.1)
		└── type-is@1.5.1 (mime-types@2.0.1)

3. 若要创建用于此应用程序的基架，请使用**express** 命令：

    	express

    请注意，本教程使用 Express v4.x.x。如果你已在系统上安装 Express 3 应用生成器，则应先卸载它：

    	npm uninstall -g express

    现在安装版本 4.x.x 的新生成器：

    	npm install -g express-generator

	运行 **express** 命令时，出现的输出看上去应与以下内容相似：

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
		create : ./routes/users.js
		create : ./views
		create : ./views/index.jade
		create : ./views/layout.jade
		create : ./views/error.jade
		create : ./bin
		create : ./bin/www

		install dependencies:
		$ cd . && npm install


	此命令完成后，**tasklist** 目录中应当会有几个新目录和文件。

4. 输入以下命令以安装 **package.json** 文件中描述的模块：

        npm install

    此命令的输出看上去应如下所示：

		cookie-parser@1.3.3 node_modules/cookie-parser
		├── cookie@0.1.2
		└── cookie-signature@1.0.5

		debug@2.0.0 node_modules/debug
		└── ms@0.6.2

		serve-favicon@2.1.4 node_modules/serve-favicon
		├── ms@0.6.2
		├── fresh@0.2.4
		└── etag@1.3.1 (crc@3.0.0)

		morgan@1.3.1 node_modules/morgan
		├── basic-auth@1.0.0
		├── depd@0.4.5
		└── on-finished@2.1.0 (ee-first@1.0.5)

		express@4.9.1 node_modules/express
		├── utils-merge@1.0.0
		├── merge-descriptors@0.0.2
		├── cookie@0.1.2
		├── fresh@0.2.4
		├── escape-html@1.0.1
		├── range-parser@1.0.2
		├── cookie-signature@1.0.5
		├── finalhandler@0.2.0
		├── vary@1.0.0
		├── media-typer@0.3.0
		├── serve-static@1.6.2
		├── parseurl@1.3.0
		├── methods@1.1.0
		├── path-to-regexp@0.1.3
		├── depd@0.4.5
		├── qs@2.2.3
		├── etag@1.3.1 (crc@3.0.0)
		├── on-finished@2.1.0 (ee-first@1.0.5)
		├── proxy-addr@1.0.1 (ipaddr.js@0.1.2)
		├── send@0.9.2 (destroy@1.0.3, ms@0.6.2, mime@1.2.11)
		├── type-is@1.5.1 (mime-types@2.0.1)
		└── accepts@1.1.0 (negotiator@0.4.7, mime-types@2.0.1)

		body-parser@1.8.2 node_modules/body-parser
		├── media-typer@0.3.0
		├── raw-body@1.3.0
		├── bytes@1.0.0
		├── depd@0.4.5
		├── on-finished@2.1.0 (ee-first@1.0.5)
		├── qs@2.2.3
		├── iconv-lite@0.4.4
		└── type-is@1.5.1 (mime-types@2.0.1)

		jade@1.6.0 node_modules/jade
		├── character-parser@1.2.0
		├── commander@2.1.0
		├── void-elements@1.0.0
		├── mkdirp@0.5.0 (minimist@0.0.8)
		├── monocle@1.1.51 (readdirp@0.2.5)
		├── transformers@2.1.0 (promise@2.0.0, css@1.0.8, uglify-js@2.2.5)
		├── constantinople@2.0.1 (uglify-js@2.4.15)
		└── with@3.0.1 (uglify-js@2.4.15)

	**package.json** 文件是 **express** 命令创建的文件之一。此文件包含 Express 应用程序所需的其他模块的列表。稍后，当你将此应用程序部署到 Azure App Service Web Apps 时，会使用此文件确定需要在 Azure 上安装的模块来支持你的应用程序。

5. 接下来，输入以下命令，以在本地安装 Mongoose 模块并将它的条目保存到 **package.json** 文件中：

		npm install mongoose --save

	此命令的输出看上去应如下所示：

		mongoose@3.8.16 node_modules/mongoose
		├── regexp-clone@0.0.1
		├── muri@0.3.1
		├── sliced@0.0.5
		├── hooks@0.2.1
		├── mpath@0.1.1
		├── mpromise@0.4.3
		├── ms@0.1.0
		├── mquery@0.8.0 (debug@0.7.4)
		└── mongodb@1.4.9 (readable-stream@1.0.31, kerberos@0.0.3, bson@0.2.12)

### 代码

现在，我们的环境和基架已准备就绪，我们将通过添加包含你的任务模型的 **task.js** 文件来扩展 **express** 命令创建的基本应用程序。你还会修改现有 **app.js** 并创建新的 **tasklist.js** 控制器文件，以使用该模型。

#### 创建模型

1. 在 **tasklist** 目录中，创建名为 **models** 的新目录。

2. 在 **models** 目录中，创建一个名为 **task.js** 的新文件。此文件将包含你的应用程序创建的任务的模型。

3. 将以下代码添加到 **task.js** 文件中：

        var mongoose = require('mongoose'),
          Schema = mongoose.Schema;

        var TaskSchema = new Schema({
	      itemName      : String,
	      itemCategory  : String,
	      itemCompleted : { type: Boolean, default: false },
	      itemDate      : { type: Date, default: Date.now }
        });

        module.exports = mongoose.model('TaskModel', TaskSchema);

5. 保存并关闭 **task.js **文件。

#### 创建控制器

1. 在 **tasklist/routes** 目录中，创建名为 **tasklist.js** 的新文件，并在文本编辑器中将其打开。

2. 将以下代码添加到 **tasklist.js**。此操作会加载 mongoose 模块和 **task.js** 中定义的任务模型。TaskList 函数用于根据 **connection** 值创建与 MongoDB 服务器的连接，并提供 **showTasks**、**addTask** 和 **completeTasks** 方法：

		var mongoose = require('mongoose'),
 		  task = require('../models/task.js');

		module.exports = TaskList;

		function TaskList(connection) {
		  mongoose.connect(connection);
		}

		TaskList.prototype = {
		  showTasks: function(req, res) {
		    task.find({ itemCompleted : false }, function foundTasks(err, items) {
		    res.render('index', { title: 'My ToDo List', tasks: items })
		    });
		  },

		  addTask: function(req,res) {
		    var item = req.body;
		    var newTask = new task();
		    newTask.itemName = item.itemName;
		    newTask.itemCategory = item.itemCategory;
		    newTask.save(function savedTask(err) {
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

3. 保存 **tasklist.js** 文件。

#### 修改索引视图

1. 将目录更改为 **views** 目录并在文本编辑器中打开 **index.jade** 文件。

2. 将 **index.jade** 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

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
		        input(name="itemName", type="textbox")
		    tr
		      td Item Category:
		      td
		        input(name="itemCategory", type="textbox")
		  input(type="submit", value="Add item")

3. 保存并关闭 **index.jade** 文件。

#### 替换 app.js

1. 在 **tasklist** 目录中，用文本编辑器打开 **app.js** 文件。此文件之前是通过运行 **express** 命令创建的。
2. 将以下代码添加到 **app.js** 文件的开头。这将使用 MongoDB 服务器的连接字符串初始化 **TaskList**：

		var TaskList = require('./routes/tasklist');
		var taskList = new TaskList(process.env.CUSTOMCONNSTR_MONGOLAB_URI);

 	请注意第二行；你访问将在以后配置的环境变量，该变量包含针对你的 mongo 实例的连接信息。如果你出于开发目的运行本地 mongo 实例，则可能要将此值暂时设置为“localhost”，而不是 `process.env.CUSTOMCONNSTR_MONGOLAB_URI`。

3. 找到这些行：

		app.use('/', routes);
		app.use('/users', users);

	并将其替换为：

		app.get('/', taskList.showTasks.bind(taskList));
		app.post('/addtask', taskList.addTask.bind(taskList));
		app.post('/completetask', taskList.completeTask.bind(taskList));

	这将添加 **tasklist.js** 中定义为路由的函数。

4. 若要初始化你的应用，请转到 **app.js** 文件的底部并添加以下行：

		app.listen(3000); // Listen on port 3000
		module.exports = app;

5. 保存 **app.js** 文件。

<a name="deploy"></a>
## 部署应用程序

现在，应用程序已开发完毕，是时候在 Azure App Service 中创建 Web 应用了，以托管该应用程序、配置该 Web 应用并部署代码。本节的关键是使用 MongoDB 连接字符串 (URI)。你将使用此 URI 在你的 Web 应用中配置环境变量，以便将该 URI 与你的代码分开。您应将该 URI 视为敏感信息，因为它包含用于连接到您的数据库的凭据。

本部分中的步骤使用适用于 Mac、Linux 和 Windows 的 Azure 命令行界面在 Azure App Service 中创建新的 Web 应用，然后使用 Git 部署应用程序。若要执行这些步骤，你必须具有 Azure 订阅。

### 安装 Azure CLI

若要安装 Azure CLI，请使用以下命令：

	npm install azure-cli -g

如果你已从 <a href="/develop/nodejs/">Azure 开发人员中心</a>安装了 <strong>Azure SDK for Node.js</strong>，则应该已安装了 Azure CLI。有关详细信息，请参阅 <a href="/documentation/articles/virtual-machines-command-line-tools">Azure CLI</a>。

虽然 Azure CLI 主要针对 Mac 和 Linux 用户而创建，但它们基于 Node.js，应该可在能够运行 Node 的任何系统上使用。

### 导入发布设置

使用 Azure CLI 之前，你必须首先下载包含有关你的订阅信息的文件。执行以下步骤以下载并导入该文件。

1. 从命令行中，输入以下命令以启动浏览器并导航到下载页面。如果出现提示，请使用与您的订阅关联的帐户登录。

		azure account download

	![下载页面][download-publishing-settings]

	文件下载应该会自动开始；如果没有自动开始，你可以单击该页面开头的链接手动下载文件。

2. 文件下载完成后，请使用以下命令导入设置：

		azure account import <path-to-file>

	指定你在上一步中下载的发布设置文件的路径和文件名。命令完成后，你应该会看到与下面类似的输出：

		info:   Executing command account import
		info:   Found subscription: subscriptionname
		info:   Setting default subscription to: subscriptionname
		warn:   The '/Users/mongolab/.azure/publishSettings.xml' file contains sensitive information.
		warn:   Remember to delete it now that it has been imported.
		info:   Account publish settings imported successfully
		info:   account import command OK


3. 在导入完成后，你应删除发布设置文件，因为不再需要该文件并且它包含有关你的 Azure 订阅的敏感信息。

### 创建新 Web 应用并推送你的代码

在 Azure App Service 中创建 Web 应用非常简单。如果这是你的第一个 Azure Web 应用，则你必须使用门户。如果你已拥有至少一个 Azure 网站，则跳到步骤 7。

1. 在 Azure 门户中，单击“新建”。![新建][button-new]
2. 选择“计算 > Web 应用 > 快速创建”。
<!-- ![Create Web App][screen-mongolab-newwebsite] -->
3. 输入 URL 前缀。选择你喜欢的名称，但要注意这必须是唯一的（可能无法使用“mymongoapp”）。
4. 单击“创建 Web 应用”。
5. Web 应用创建完成时，单击 Web Apps 列表中的 Web 应用名称。将显示“Web 应用仪表板”。
<!-- ![Web App Dashboard][screen-mongolab-websitedashboard] -->
6. 在“速览”下单击“在源代码管理中设置部署”，选择 GitHub，然后输入所需的 git 用户名和密码。当推送到你的 Web 应用时，则会使用此密码（在第 9 步中）。  
7. 如果你使用上述步骤创建了 Web 应用，以下命令则会完成该过程。但是，如果你已拥有多个 Web 应用，则可以跳过上述步骤并使用这一相同命令创建新的 Web 应用。从你的 **tasklist** 项目目录中：

		azure site create myuniquewebappname --git  
	将“myuniquewebappname”替换为你的 Web 应用的唯一站点。如果 Web 应用作为此命令的一部分进行创建，则会提示该 Web 应用将位于的数据中心。选择在地理上靠近你的 MongoLab 数据库的数据中心。

	`--git` 参数将创建：* **tasklist** 文件夹中的一个本地 git 存储库（如果尚不存在）。* 一个名为“azure”的 [Git remote]，用于将应用程序发布到 Azure。* 一个 [iisnode.yml] 文件，其中包含 Azure 用于托管 node 应用程序的设置。* 一个阻止将 node-modules 文件夹发布到 .git 的 .gitignore 文件。

	此命令完成后，你将看到与下面类似的输出。请注意，以 **Created site at** 开头的行包含 Web 应用的 URL。

		info:   Executing command site create
		info:   Using location southcentraluswebspace
		info:   Executing `git init`
		info:   Creating default web.config file
		info:   Creating a new web site
		info:   Created web site at  mongodbtasklist.chinacloudsites.cn
		info:   Initializing repository
		info:   Repository initialized
		info:   Executing `git remote add azure http://gitusername@myuniquewebappname.chinacloudsites.cn/mongodbtasklist.git`
		info:   site create command OK

8. 使用以下命令将文件添加然后提交到你的本地 Git 存储库：

		git add .
		git commit -m "adding files"

9. 推送代码：

		git push azure master  

	在 Azure App Service 中将最新 Git 存储库更改推送到的 Web 应用时，你必须指定目标分支为 **master**，因为这将用于 Web 应用内容。如果系统提示你输入密码，请输入你在前面为 Web 应用设置 git 发布时创建的密码。

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
		To https://username@mongodbtasklist.chinacloudsites.cn/MongoDBTasklist.git
 		 * [new branch]      master -> master

即将完成！

### 配置环境
还记得代码中的 process.env.CUSTOMCONNSTR\_MONGOLAB_URI 吗？ 我们需要使用你在配置 MongoLab 数据库期间提供给 Azure 的值填充该环境变量。

#### 获取 MongoLab 连接字符串

[AZURE.INCLUDE [howto-get-connectioninfo-mongolab](../includes/howto-get-connectioninfo-mongolab.md)]

#### 将连接字符串添加到 Web 应用的环境变量中

[AZURE.INCLUDE [howto-save-connectioninfo-mongolab](../includes/howto-save-connectioninfo-mongolab.md)]

## 成功！

从你的项目目录运行 `azure site browse`，以便自动打开浏览器，或者打开浏览器并手动导航到你的 Web 应用 URL (myuniquewebappname.chinacloudsites.cn)：

![显示空白 tasklist 的网页][node-mongo-finished]

<a name="manage"></a>
## 管理数据库

[AZURE.INCLUDE [howto-access-mongolab-ui](../includes/howto-access-mongolab-ui.md)]

祝贺你！ 你刚刚启动了由 MongoLab 托管的 MongoDB 数据库提供支持的 Node.js 应用程序！ 现在，你拥有了一个 MongoLab 数据库，如有任何关于你的数据库的问题或疑虑，或者要获得有关 MongoDB 或 Node 驱动程序本身的帮助，请联系 [support@mongolab.com](mailto:support@mongolab.com)。祝您好运！



[screen-mongolab-websitedashboard]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/screen-mongolab-websitedashboard.png
[screen-mongolab-newwebsite]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/screen-mongolab-newwebsite.png
[button-new]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-new.png
[button-store]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-store.png
[entry-mongolab]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/entry-mongolab.png
[button-connectioninfo]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/button-connectioninfo.png
[screen-connectioninfo]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/dialog-mongolab_connectioninfo.png
[focus-website-connectinfo]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/focus-mongolab-websiteconnectionstring.png
[provision]: #provision
[create]: #create
[deploy]: #deploy
[manage]: #manage
[Node.js]: http://nodejs.org
[MongoDB]: http://www.mongodb.org
[Git]: http://git-scm.com
[Express]: http://expressjs.com
[Mongoose]: http://mongoosejs.com
[for free]: /pricing/free-trial
[Git remote]: http://git-scm.com/docs/git-remote
[azure-sdk-for-node]: https://github.com/WindowsAzure/azure-sdk-for-node
[iisnode.yml]: https://github.com/WindowsAzure/iisnode/blob/master/src/samples/configuration/iisnode.yml
[Azure CLI]: /documentation/articles/virtual-machines-command-line-tools
[Azure Developer Center]: /develop/nodejs/
[Create and deploy a Node.js application to Azure Web Sites]: /develop/nodejs/tutorials/create-a-website-(mac)/
[Continuous deployment using GIT in Azure App Service]: /documentation/articles/web-sites-publish-source-control
[MongoLab]: http://mongolab.com
[Node.js Web Application with Storage on MongoDB (Virtual Machine)]: /develop/nodejs/tutorials/website-with-mongodb-(mac)/
[node-mongo-finished]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/todo_list_noframe.png
[node-mongo-express-results]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/express_output.png
[download-publishing-settings]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/azure-account-download-cli.png
[import-publishing-settings]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/azureimport.png
[mongolab-create]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/mongolab-create.png
[mongolab-view]: ./media/store-mongolab-web-sites-nodejs-store-data-mongodb/mongolab-view.png
 

<!---HONumber=71-->