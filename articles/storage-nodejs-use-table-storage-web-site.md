<properties
	pageTitle="使用 Azure 表服务的 Node.js Web 应用"
	description="本教程演示如何使用 Azure 表服务在 Azure Web 应用所托管的 Node.js 应用程序中存储数据。"
	tags="azure-portal"
	services="app-service\web, storage"
	documentationCenter="nodejs"
	authors="MikeWasson"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/08/2016"
	wacn.date="05/24/2016"/>



# 使用 Azure 表服务的 Node.js Web 应用

## 概述
本教程演示如何使用 Azure 数据管理提供的表服务来存储和访问在 Azure Web 应用上托管的 [node] 应用程序的数据。本教程假定你之前有使用 node 和[Git]的经验。

你将学习以下内容：

* 如何使用 npm（Node 包管理器）安装 Node 模块

* 如何使用 Azure 表服务

* 如何使用 Azure CLI 创建 Web 应用。

按照本教程中的说明操作，构建基于 Web 的简单“待办事项”应用程序，该应用程序可用于创建、检索和完成任务。这些任务存储在表服务中。

下面是已完成的应用程序：

![显示空白 tasklist 的网页][node-table-finished]

## 先决条件

在按照本文中的说明操作之前，请确保已安装下列项：

* [node] 0.10.24 或更高版本

* [Git]


[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 创建存储帐户

创建 Azure 存储帐户。应用会使用此帐户存储待办事项。

1. 打开你的 Web 浏览器并转到 [Azure 门户]。如果出现提示，请使用你的 Azure 订阅信息进行登录。

2. 在门户底部，单击“+新建”，然后选择“存储帐户”。

	![\+新建][portal-new]

	![存储帐户][portal-storage-account]

3. 选择“快速创建”，然后为此存储帐户输入 URL 和区域/地缘组。由于这是一个教程，不需要全球复制，因此请取消选中“启用异地复制”。最后，单击“创建存储帐户”。

	![快速创建][portal-quick-create-storage]

	请记下你输入的 URL，因为后续步骤将引用此 URL 作为帐户名称。

4. 在创建存储帐户后，单击页面底部的“管理密钥”。这将显示此存储帐户的主访问密钥和辅助访问密钥。复制并保存主访问密钥，然后单击复选标记。

	![访问密钥][portal-storage-access-keys]

## 安装模块并生成基架

在本节中，你将创建新的 Node 应用程序并使用 npm 添加模块包。对于此应用程序，你将使用 [Express] 和 [Azure] 模块。Express 模块为 Node 提供了一个模型视图控制器框架，而 Azure 模块提供了与表服务的连接。

### 安装 Express 并生成基架

1. 在命令行中，创建名为 **tasklist** 的新目录并切换到该目录。  

2. 请输入以下命令来安装 Express 模块。

		npm install express-generator@4.2.0 -g

    根据操作系统，可能需要将“sudo”放在命令之前：

		sudo npm install express-generator@4.2.0 -g

    输出显示类似于以下示例：

		express-generator@4.2.0 /usr/local/lib/node_modules/express-generator
		├── mkdirp@0.3.5
		└── commander@1.3.2 (keypress@0.1.0)

	> [AZURE.NOTE]“-g”参数会全局安装模块。这样我们使用 **express** 生成 Web 应用基架，而无需键入其他路径信息。

4. 若要创建应用程序的基架，请输入 **express** 命令：

        express

	此命令的输出显示类似于以下示例：

		   create : .
		   create : ./package.json
		   create : ./app.js
		   create : ./public
		   create : ./public/images
		   create : ./routes
		   create : ./routes/index.js
		   create : ./routes/users.js
		   create : ./public/stylesheets
		   create : ./public/stylesheets/style.css
		   create : ./views
		   create : ./views/index.jade
		   create : ./views/layout.jade
		   create : ./views/error.jade
		   create : ./public/javascripts
		   create : ./bin
		   create : ./bin/www
		
		   install dependencies:
		     $ cd . && npm install
		
		   run the app:
		     $ DEBUG=my-application ./bin/www

	此时 **tasklist** 目录中会有几个新目录和文件。

### 安装其他模块

**express** 创建的其中一个文件是 **package.json**。此文件包含模块依赖项的列表。之后，当你将应用程序部署到 Azure Web 应用时，此文件会确定需要在 Azure 上安装的模块。

在命令行中，输入以下命令，以安装 **package.json** 文件中描述的模块。可能需要使用“sudo”。

        npm install

此命令的输出显示类似于以下示例：

	debug@0.7.4 node_modules\debug

	cookie-parser@1.0.1 node_modules\cookie-parser
	├── cookie-signature@1.0.3
	└── cookie@0.1.0

    [...]


接下来，输入以下命令，以安装 [azure]、[node-uuid]、[nconf] 和 [async] 模块：

	npm install azure-storage node-uuid async nconf --save

使用 **--保存**标志将这些模块的条目添加到 **package.json** 文件中。

此命令的输出显示类似于以下示例：

	async@0.9.0 node_modules\async

	node-uuid@1.4.1 node_modules\node-uuid

	nconf@0.6.9 node_modules\nconf
	├── ini@1.2.1
	├── async@0.2.9
	└── optimist@0.6.0 (wordwrap@0.0.2, minimist@0.0.10)

	[...]


## 创建应用程序

现在我们准备好生成应用程序。

### 创建模型

*模型*是表示应用程序中的数据的对象。对于应用程序，唯一的模型是任务对象，表示待办事项列表中的项。任务将具有以下字段：

- PartitionKey
- RowKey
- 名称（字符串）
- 类别（字符串）
- 已完成（布尔值）

**PartitionKey** 和 **RowKey** 被表服务用作表键。有关详细信息，请参阅[了解表服务数据模型](https://msdn.microsoft.com/zh-cn/library/azure/dd179338.aspx)。


1. 在 **tasklist** 目录中，创建名为 **models** 的新目录。

2. 在 **models** 目录中，创建一个名为 **task.js** 的新文件。此文件将包含你的应用程序创建的任务的模型。

3. 在 **task.js** 文件的开头，添加以下代码来引用所需的库：

        var azure = require('azure-storage');
  		var uuid = require('node-uuid');
		var entityGen = azure.TableUtilities.entityGenerator;

4. 添加以下代码，以定义和导出 Task 对象。此对象负责与表连接。

  		module.exports = Task;

		function Task(storageClient, tableName, partitionKey) {
		  this.storageClient = storageClient;
		  this.tableName = tableName;
		  this.partitionKey = partitionKey;
		  this.storageClient.createTableIfNotExists(tableName, function tableCreated(error) {
		    if(error) {
		      throw error;
		    }
		  });
		};

5. 添加以下代码来定义 Task 对象上的其他方法，利用这些方法可以实现与表中存储的数据进行交互：

		Task.prototype = {
		  find: function(query, callback) {
		    self = this;
		    self.storageClient.queryEntities(this.tableName, query, null, function entitiesQueried(error, result) {
		      if(error) {
		        callback(error);
		      } else {
		        callback(null, result.entries);
		      }
		    });
		  },

		  addItem: function(item, callback) {
		    self = this;
		    // use entityGenerator to set types
			// NOTE: RowKey must be a string type, even though
            // it contains a GUID in this example.
		    var itemDescriptor = {
		      PartitionKey: entityGen.String(self.partitionKey),
		      RowKey: entityGen.String(uuid()),
		      name: entityGen.String(item.name),
		      category: entityGen.String(item.category),
		      completed: entityGen.Boolean(false)
		    };
		    self.storageClient.insertEntity(self.tableName, itemDescriptor, function entityInserted(error) {
		      if(error){  
		        callback(error);
		      }
		      callback(null);
		    });
		  },

		  updateItem: function(rKey, callback) {
		    self = this;
		    self.storageClient.retrieveEntity(self.tableName, self.partitionKey, rKey, function entityQueried(error, entity) {
		      if(error) {
		        callback(error);
		      }
		      entity.completed._ = true;
		      self.storageClient.updateEntity(self.tableName, entity, function entityUpdated(error) {
		        if(error) {
		          callback(error);
		        }
		        callback(null);
		      });
		    });
		  }
		}

6. 保存并关闭 **task.js **文件。

### 添加控制器

*控制器*处理 HTTP 请求并渲染 HTML 响应。

1. 在 **tasklist/routes** 目录中，创建名为 **tasklist.js** 的新文件，并在文本编辑器中将其打开。

2. 将以下代码添加到 **tasklist.js**。这将加载 **tasklist.js** 使用的 azure 和 async 模块。这还将定义 **TaskList** 函数，将向该函数传递我们之前定义的 **Task** 对象的一个实例：

		var azure = require('azure-storage');
		var async = require('async');

		module.exports = TaskList;

3. 定义 **TaskList** 对象。

		function TaskList(task) {
		  this.task = task;
		}


4. 将以下方法添加到 **TaskList**。

		TaskList.prototype = {
		  showTasks: function(req, res) {
		    self = this;
		    var query = new azure.TableQuery()
		      .where('completed eq ?', false);
		    self.task.find(query, function itemsFound(error, items) {
		      res.render('index',{title: 'My ToDo List ', tasks: items});
		    });
		  },

		  addTask: function(req,res) {
		    var self = this;
		    var item = req.body.item;
		    self.task.addItem(item, function itemAdded(error) {
		      if(error) {
		        throw error;
		      }
		      res.redirect('/');
		    });
		  },

		  completeTask: function(req,res) {
		    var self = this;
		    var completedTasks = Object.keys(req.body);
		    async.forEach(completedTasks, function taskIterator(completedTask, callback) {
		      self.task.updateItem(completedTask, function itemsUpdated(error) {
		        if(error){
		          callback(error);
		        } else {
		          callback(null);
		        }
		      });
		    }, function goHome(error){
		      if(error) {
		        throw error;
		      } else {
		       res.redirect('/');
		      }
		    });
		  }
		}


### 修改 app.js

1. 在 **tasklist** 目录中，打开 **app.js** 文件。此文件之前是通过运行 **express** 命令创建的。

2. 在文件开头，添加以下代码来加载 azure 模块，设置表名称、partitionKey，并设置此示例使用的存储凭据：

		var azure = require('azure-storage');
		var nconf = require('nconf');
		nconf.env()
		     .file({ file: 'config.json', search: true });
		var tableName = nconf.get("TABLE_NAME");
		var partitionKey = nconf.get("PARTITION_KEY");
		var accountName = nconf.get("STORAGE_NAME");
		var accountKey = nconf.get("STORAGE_KEY");

	> [AZURE.NOTE]nconf 将从环境变量或我们稍后将创建的 **config.json** 文件中加载配置值。

3. 在 app.js 文件中，向下滚动到以下行：

		app.use('/', routes);
		app.use('/users', users);

	将上面的行替换为下面显示的代码。这将通过与你的存储帐户的连接初始化 <strong>Task</strong> 的实例。这是 <strong>TaskList</strong> 的密码，TaskList 将使用该密码与表服务进行通信：

		var TaskList = require('./routes/tasklist');
		var Task = require('./models/task');
		var task = new Task(azure.createTableService(accountName, accountKey), tableName, partitionKey);
		var taskList = new TaskList(task);

		app.get('/', taskList.showTasks.bind(taskList));
		app.post('/addtask', taskList.addTask.bind(taskList));
		app.post('/completetask', taskList.completeTask.bind(taskList));
	
4. 保存 **app.js** 文件。

### 修改索引视图

1. 在文本编辑器中打开 **tasklist/views/index.jade** 文件。

2. 将文件的全部内容替换为以下代码。这会定义显示现有任务的视图，并包括用于添加新任务和将现有任务标记为已完成的表单。

		extends layout

		block content
		  h1= title
		  br
		
		  form(action="/completetask", method="post")
		    table.table.table-striped.table-bordered
		      tr
		        td Name
		        td Category
		        td Date
		        td Complete
		      if (typeof tasks === "undefined")
		        tr
		          td 
		      else
		        each task in tasks
		          tr
		            td #{task.name._}
		            td #{task.category._}
		            - var day   = task.Timestamp._.getDate();
		            - var month = task.Timestamp._.getMonth() + 1;
		            - var year  = task.Timestamp._.getFullYear();
		            td #{month + "/" + day + "/" + year}
		            td
		              input(type="checkbox", name="#{task.RowKey._}", value="#{!task.completed._}", checked=task.completed._)
		    button.btn(type="submit") Update tasks
		  hr
		  form.well(action="/addtask", method="post")
		    label Item Name: 
		    input(name="item[name]", type="textbox")
		    label Item Category: 
		    input(name="item[category]", type="textbox")
		    br
		    button.btn(type="submit") Add item

3. 保存并关闭 **index.jade** 文件。

### 修改全局布局

**views** 目录中的 **Layout.jade** 文件是其他 **.jade** 文件的全局模板。在此步骤中，你将对其进行修改，以使用 [Twitter Bootstrap](https://github.com/twbs/bootstrap)（一个可以轻松设计美观 Web 应用的工具包）。

下载并提取 [Twitter Bootstrap](http://getbootstrap.com/) 的文件。将 **bootstrap.min.css** 文件从 Bootstrap **css** 文件夹复制到应用程序的 **public/stylesheets** 目录中。

在 **views** 文件夹中，打开 **layout.jade** 并将整体内容替换为以下代码：

	doctype html
	html
	  head
	    title= title
	    link(rel='stylesheet', href='/stylesheets/bootstrap.min.css')
	    link(rel='stylesheet', href='/stylesheets/style.css')
	  body.app
	    nav.navbar.navbar-default
	      div.navbar-header
	      a.navbar-brand(href='/') My Tasks
	    block content

### 创建 config 文件

若要在本地运行应用，我们则会将 Azure 存储的凭据置于 config 文件中。使用以下 JSON 创建名为 **config.json* * 的文件：

	{
		"STORAGE_NAME": "<storage account name>",
		"STORAGE_KEY": "<storage access key>",
		"PARTITION_KEY": "mytasks",
		"TABLE_NAME": "tasks"
	}

将**存储帐户名称**替换为你之前创建的存储帐户的名称，并将**存储访问密钥**替换为你的存储帐户的主访问密钥。例如：

	{
	    "STORAGE_NAME": "nodejsappstorage",
	    "STORAGE_KEY": "KG0oDd..."
	    "PARTITION_KEY": "mytasks",
	    "TABLE_NAME": "tasks"
	}

将此文件保存为比 **tasklist** 目录*高一个目录级*，像这样：

	parent/
	  |-- config.json
	  |-- tasklist/

这样做的目的是避免将 config 文件签入源代码管理中，其中该文件可能会成为公共文件。当我们将应用部署到 Azure 时，我们会使用环境变量而不是 config 文件。


## 在本地运行应用程序

若要在你的本地计算机中测试应用程序，请执行以下步骤：

1. 在命令行中，将目录更改为 **tasklist** 目录。

2. 使用以下命令在本地启动应用程序：

        npm start

3. 打开 Web 浏览器并导航到 http://127.0.0.1:3000。

	此时会显示类似于以下示例的网页。

![显示空白 tasklist 的网页][node-table-finished]

4. 若要创建新的待办事项，请输入名称和类别，然后单击“添加项”。 

6. 若要将任务标记为完成，请选中“完成”，然后单击“更新任务”。

![任务列表中新项的图像][node-table-list-items]

即使应用程序在本地运行，它也会将数据存储在 Azure 表服务中。

## 将你的应用程序部署到 Azure

本部分中的步骤使用 Azure 命令行工具在 Azure 中创建新的 Web 应用，然后使用 Git 部署应用程序。若要执行这些步骤，你必须具有 Azure 订阅。

> [AZURE.NOTE]还可以使用 [Azure 门户](https://manage.windowsazure.cn/)执行这些步骤。请参阅[在 Azure 中生成和部署 Node.js Web 应用]。
>
> 如果这是你创建的第一个 Web 应用，则你必须使用 Azure 门户部署此应用程序。

若要开始，请在命令行中输入以下命令以安装 [Azure CLI]：

	npm install azure-cli -g

### 导入发布设置

在此步骤中，将下载包含有关你的订阅的信息的文件。

1. 输入以下命令：

		azure account download

	此命令启动浏览器并导航到下载页面。如果出现提示，请使用与你的 Azure 订阅关联的帐户登录。

	<!-- ![The download page][download-publishing-settings] -->
	文件下载会自动开始；如果没有自动开始，你可以单击该页面开头的链接手动下载文件。保存文件并记下文件路径。

2. 输入以下命令以导入设置：

		azure account import <path-to-file>

	指定你在上一步中下载的发布设置文件的路径和文件名。

3. 导入设置后，请删除发布设置文件。因为不再需要该文件并且它包含有关你的 Azure 订阅的敏感信息。

### 创建 Web 应用

1. 在命令行中，将目录更改为 **tasklist** 目录。

2. 使用以下命令创建新的 Web 应用。

		azure site create --git

	系统将提示 Web 应用名称和位置。提供唯一的名称并选择与你的 Azure 存储帐户的相同的地理位置。

	`--git` 参数在 Azure 中为此 Web 应用创建 Git 存储库。如果不存在，则还会初始化当前目录中的 Git 存储库，并添加名为“azure”的 [Git remote]，用于将应用程序发布到 Azure。最后，它会创建 **web.config** 文件，其中包含 Azure 用于托管 node 应用程序的设置。如果省略 `--git` 参数，但目录包含 Git 存储库，命令仍会创建“azure”remote。

	此命令完成后，你将看到与下面类似的输出。请注意，以 **Website created at** 开头的行包含 Web 应用的 URL。

		info:   Executing command site create
		help:   Need a site name
		Name: TableTasklist
		info:   Using location southcentraluswebspace
		info:   Executing `git init`
		info:   Creating default .gitignore file
		info:   Creating a new web site
		info:   Created web site at  tabletasklist.chinacloudsites.cn
		info:   Initializing repository
		info:   Repository initialized
		info:   Executing `git remote add azure https://username@tabletasklist.chinacloudsites.cn/TableTasklist.git`
		info:   site create command OK

	> [AZURE.NOTE]如果这是你的订阅的第一个 Web 应用，系统会指示你使用 Azure 门户创建 Web 应用。有关详细信息，请参阅[在 Azure 中生成和部署 Node.js Web 应用]。

### 切换到环境变量

前面我们实现了用于查找环境变量或从 **config.json** 文件中加载值的代码。在接下来的步骤中，你将在 Web 应用配置中创建应用程序通过环境变量实际访问的键值对。

1. 从管理门户中，单击“ Web 应用”，然后选择你的 Web 应用。

	![打开 Web 应用仪表板][go-to-dashboard]

2. 单击“配置”，然后找到页面的“应用设置”部分。

	![配置链接][web-configure]

3. 在“应用设置”部分的 **KEY** 字段中输入 **STORAGE\_NAME**，并在 **VALUE**字段中输入你的存储帐户的名称。单击复选标记以移到下一个字段。为以下密钥和值重复此过程：

	* **STORAGE\_KEY** - 你的存储帐户的访问密钥
	
	* **PARTITION\_KEY** -“mytasks”

	* **TABLE\_NAME** -“tasks”

	![应用程序设置][app-settings]

4. 最后，单击页面底部的“保存”图标，将此更改提交到运行时环境。

	![保存应用程序设置][app-settings-save]

5. 从命令行中，将目录更改为 **tasklist** 目录，然后输入以下命令以删除 **config.json** 文件：

		git rm config.json
		git commit -m "Removing config file"

6. 执行以下命令将更改部署到 Azure：

		git push azure master

在将更改部署到 Azure 后，你的 Web 应用应当继续工作，因为它现在从“应用设置”条目读取连接字符串。若要验证此情况，请在“应用设置”中将 **STORAGE\_KEY** 条目的值更改为一个无效值。保存该值后， Web 应用应该会因存储访问密钥设置无效而失败。

### 发布应用程序

若要发布应用，将代码文件提交到 Git 中，随后推送到 azure/master。

1. 设置部署凭据。

		azure site deployment user set <name> <password>

2. 添加并提交应用程序文件。

		git add .
		git commit -m "adding files"

3. 将提交的文件推送到 Web 应用：

		git push azure master

	使用 **master** 作为目标分支。在部署结束时，你将看到类似于以下示例的语句：

		To https://username@tabletasklist.chinacloudsites.cn/TableTasklist.git
 		 * [new branch]      master -> master

4. 推送操作完成后，浏览到 `azure create site` 命令之前返回的 Web 应用URL，以查看你的应用程序。

## 其他资源
[Azure 命令行界面][Azure CLI]
[创建 Node.js 应用程序并将其部署到 Azure Web 应用]: /documentation/articles/web-sites-nodejs-develop-deploy-mac/
[使用 Git 发布到 Azure Web 应用]: /documentation/articles/web-sites-publish-source-control/
[Azure 开发人员中心]: /develop/nodejs/



[node]: http://nodejs.org
[Git]: http://git-scm.com
[Express]: http://expressjs.com
[for free]: http://www.azure.cn
[Git remote]: http://git-scm.com/docs/git-remote

[使用 MongoDB 的 Node.js Web 应用]: /documentation/articles/web-sites-nodejs-store-data-mongodb/
[Azure CLI]: /documentation/articles/xplat-cli-install/

[使用 Git 发布到 Azure Web 应用]: /documentation/articles/web-sites-publish-source-control/
[azure]: https://github.com/Azure/azure-sdk-for-node
[node-uuid]: https://www.npmjs.com/package/node-uuid
[nconf]: https://www.npmjs.com/package/nconf
[async]: https://www.npmjs.com/package/async

[Azure 门户]: http://manage.windowsazure.cn


[node-table-finished]: ./media/storage-nodejs-use-table-storage-web-site/table_todo_empty.png
[node-table-list-items]: ./media/storage-nodejs-use-table-storage-web-site/table_todo_list.png
[download-publishing-settings]: ./media/storage-nodejs-use-table-storage-web-site/azure-account-download-cli.png
[portal-new]: ./media/storage-nodejs-use-table-storage-web-site/plus-new.png
[portal-storage-account]: ./media/storage-nodejs-use-table-storage-web-site/new-storage.png
[portal-quick-create-storage]: ./media/storage-nodejs-use-table-storage-web-site/quick-storage.png
[portal-storage-access-keys]: ./media/storage-nodejs-use-table-storage-web-site/manage-access-keys.png

[go-to-dashboard]: ./media/storage-nodejs-use-table-storage-web-site/go_to_dashboard.png
[web-configure]: ./media/storage-nodejs-use-table-storage-web-site/sql-task-configure.png
[app-settings-save]: ./media/storage-nodejs-use-table-storage-web-site/savebutton.png
[app-settings]: ./media/storage-nodejs-use-table-storage-web-site/storage-tasks-appsettings.png

[Create and deploy a Node.js application to an Azure  Website]: /documentation/articles/web-sites-nodejs-develop-deploy-mac/

<!---HONumber=74-->