<properties linkid="dev-nodejs-tutorials-web-site-with-storage" urlDisplayName=" Website with Storage" pageTitle="使用表存储构建 Node.js 网站 | Microsoft Azure" metaKeywords="Azure table storage Node.js, Azure Node.js application, Azure Node.js tutorial, Azure Node.js example" description="本教程演示如何使用 Azure 表服务在 Azure 网站上托管的 Node 应用程序中存储数据。" metaCanonical="" services="web-sites,storage" documentationCenter="Node.js" title="Node.js Web Application using the Azure Table Service" authors="larryfr" solutions="" manager="" editor="" />





# 使用 Azure 表服务的 Node.js Web 应用程序

本教程向你演示如何使用 Azure 数据管理提供的表服务在 Azure 上托管的 [Node] 应用程序中存储和访问数据。本教程假定你之前有使用 Node 和 [Git] 的经验。

你将了解到以下内容：

* 如何使用 npm（Node 包管理器）安装 Node 模块

* 如何使用 Azure 表服务

* 如何使用适用于 Mac 和 Linux 的 Azure 命令行工具创建 Azure 网站

按照本教程中的说明操作，你将构建一个简单的基于 Web 的任务管理应用程序，该应用程序可用于创建、检索和完成任务。这些任务存储在表服务中。
 
本教程中的项目文件将存储在名为 tasklist 的目录中，已完成的应用程序将与下图类似：

![A web page displaying an empty tasklist][node-table-finished]

> [WACOM.NOTE] 本教程引用 tasklist 文件夹。将省略此文件夹的完整路径，因为路径语义在操作系统之间并不相同。你应在本地文件系统中易于访问的位置创建此文件夹（例如 ~/node/tasklist 或 c:\node\tasklist）

> [WACOM.NOTE] 下面的许多步骤都提到使用命令行。对于这些步骤，请使用适用于你的操作系统的命令行，例如 cmd.exe (Windows) 或 Bash (Unix Shell)。在 OS X 系统上，可以通过 Terminal 应用程序访问命令行。

##先决条件

在按照本文中的说明操作之前，你应确保已安装下列项：

* [Node] 0.10.24 或更高版本

* [Git]

* 文本编辑器

* Web 浏览器

[WACOM.INCLUDE [create-account-and- Websites-note](../includes/create-account-and- Websites-note.md)]

##创建存储帐户

执行下列步骤来创建一个存储帐户。本教程中的后续说明将使用此帐户。

1. 打开你的 Web 浏览器并转到 [Azure 门户]。如果出现提示，请使用你的 Azure 订阅信息进行登录。

2. 在门户底部，单击"+新建"，然后选择"存储帐户"。

	![+new][portal-new]

	![storage account][portal-storage-account]

3. 选择"快速创建"，然后为此存储帐户输入 URL 和区域/地缘组。由于这是一个教程，不需要全球复制，因此请取消选中"启用地域复制"。最后，单击"创建存储帐户"。

	![quick create][portal-quick-create-storage]

	请记下你输入的 URL，因为后续步骤将引用此 URL 作为帐户名称。

4. 在创建存储帐户后，单击页面底部的"管理密钥"。这将显示此存储帐户的主访问密钥和辅助访问密钥。复制并保存主访问密钥，然后单击复选标记。

	![access keys][portal-storage-access-keys]

##安装模块并生成基架

在本节中，你将创建新的 Node 应用程序并使用 npm 添加模块包。对于任务列表应用程序，你将使用 [Express] 和 [Azure] 模块。Express 模块为 Node 提供了一个模型视图控制器框架，而 Azure 模块提供了与表服务的连接。

###安装 Express 并生成基架

1. 在命令行中，将目录更改为 tasklist 目录。如果 tasklist 目录不存在，请创建该目录。

2. 输入以下命令来安装 Express。

		npm install express-generator@4.2.0 -g

    > [WACOM.NOTE] 在某些操作系统上使用"-g"参数时，你可能会收到一条错误 **Error:EPERM, chmod '/usr/local/bin/express'** 和一个尝试以管理员身份运行帐户的请求。如果发生这种情况，请使用 sudo 命令以更高的权限级别运行 npm。

    此命令的输出看上去应如下所示：

		express-generator@4.2.0 C:\Users\username\AppData\Roaming\npm\node_modules\express-generator
		├── mkdirp@0.3.5
		└── commander@1.3.2 (keypress@0.1.0)

	> [WACOM.NOTE] 在安装 Express 模块时使用"-g"参数将全局安装该模块。这样我们就可以访问 express 命令以生成网站基架，而无需键入其他路径信息。**

4. 若要创建将用于此应用程序的基架，请使用 express 命令：

        express

	此命令的输出看上去应如下所示：

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

	此命令完成后，你应当会在 tasklist 目录中拥有几个新目录和文件。

###安装其他模块

package.json 文件是 express 命令创建的文件之一。此文件包含 Express 应用程序所需的其他模块的列表。稍后，在你将此应用程序部署到 Azure 网站时，将使用此文件确定需要在 Azure 上安装哪些模块来支持你的应用程序。

1. 从命令行中，将目录更改为 tasklist 文件夹，然后输入以下命令安装 package.json 文件中描述的模块：

        npm install

    此命令的输出看上去应如下所示：

		debug@0.7.4 node_modules\debug
		
		static-favicon@1.0.2 node_modules\static-favicon
		
		morgan@1.0.1 node_modules\morgan
		└── bytes@0.3.0
		
		cookie-parser@1.0.1 node_modules\cookie-parser
		├── cookie-signature@1.0.3
		└── cookie@0.1.0
		
		body-parser@1.0.2 node_modules\body-parser
		├── qs@0.6.6
		├── raw-body@1.1.7 (string_decoder@0.10.25-1, bytes@1.0.0)
		└── type-is@1.1.0 (mime@1.2.11)
		
		express@4.2.0 node_modules\express
		├── parseurl@1.0.1
		├── merge-descriptors@0.0.2
		├── utils-merge@1.0.0
		├── cookie@0.1.2
		├── escape-html@1.0.1
		├── cookie-signature@1.0.3
		├── debug@0.8.1
		├── fresh@0.2.2
		├── qs@0.6.6
		├── range-parser@1.0.0
		├── methods@1.0.0
		├── buffer-crc32@0.2.1
		├── serve-static@1.1.0
		├── path-to-regexp@0.1.2
		├── send@0.3.0 (debug@0.8.0, mime@1.2.11)
		├── type-is@1.1.0 (mime@1.2.11)
		└── accepts@1.0.1 (negotiator@0.4.7, mime@1.2.11)
		
		jade@1.3.1 node_modules\jade
		├── commander@2.1.0
		├── character-parser@1.2.0
		├── mkdirp@0.3.5
		├── monocle@1.1.51 (readdirp@0.2.5)
		├── constantinople@2.0.1 (uglify-js@2.4.15)
		├── transformers@2.1.0 (promise@2.0.0, css@1.0.8, uglify-js@2.2.5)
		└── with@3.0.0 (uglify-js@2.4.15)


	这将安装 Express 需要的所有默认模块。

2. 接下来，输入以下命令在本地安装 [azure]、[node-uuid]、[nconf] 和 [async] 模块，并将它们的一个条目保存到 package.json 文件：

		npm install azure-storage node-uuid async nconf --save

	此命令的输出看上去应如下所示：

		async@0.9.0 node_modules\async

		node-uuid@1.4.1 node_modules\node-uuid
		
		nconf@0.6.9 node_modules\nconf
		├── ini@1.2.1
		├── async@0.2.9
		└── optimist@0.6.0 (wordwrap@0.0.2, minimist@0.0.10)
		
		azure-storage@0.3.0 node_modules\azure-storage
		├── extend@1.2.1
		├── xmlbuilder@0.4.3
		├── mime@1.2.11
		├── validator@3.1.0
		├── underscore@1.4.4
		├── xml2js@0.2.7 (sax@0.5.2)
		└── request@2.27.0 (forever-agent@0.5.2, aws-sign@0.3.0, json-stringify-safe@5.0.0, tunnel-agent@0.3.0, qs@0.6.6, oauth-sign@0.3.0, cookie-jar@0.3.0, form-data@0.1.4, hawk@1.0.0, http-signature@0.10.0)

##Using Node 应用程序中的表服务

在本节中，你将通过添加一个包含你的任务模型的 task.js 文件来扩展 express 命令创建的基本应用程序。你还将修改现有 app.js 并创建使用该模型的新 tasklist.js 文件。

### 创建模型

1. 在 tasklist 目录中，创建一个名为 models 的新目录。

2. 在 models 目录中，创建一个名为 task.js 的新文件。此文件将包含你的应用程序创建的任务的模型。

3. 在 task.js 文件的开头，添加以下代码来引用所需的库：

        var azure = require('azure-storage');
  		var uuid = require('node-uuid');
		var entityGen = azure.TableUtilities.entityGenerator;

4. 接下来，你将添加代码以定义和导出 Task 对象。此对象负责与表连接。

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

5. 接下来，添加以下代码来定义 Task 对象的其他方法，这些方法允许与表中存储的数据交互：

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

6. 保存并关闭 task.js 文件。

###创建控制器

1. 在 tasklist/routes 目录中，创建一个名为 tasklist.js 的新文件并在文本编辑器中将其打开。

2. 将以下代码添加到 tasklist.js。这将加载 tasklist.js 使用的 azure 和 async 模块。这还将定义 TaskList 函数，将向该函数传递我们之前定义的 Task 对象的一个实例：

		var azure = require('azure-storage');
		var async = require('async');

		module.exports = TaskList;

		function TaskList(task) {
		  this.task = task;
		}

2. 继续向 tasklist.js 文件添加用于 showTasks、addTask 和 completeTasks 的方法：

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
		    var self = this      
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

3. 保存 tasklist.js 文件。

### 修改 app.js

1. 在 tasklist 目录中，用文本编辑器打开 app.js 文件。此文件之前是通过运行 express 命令创建的。

2. 在文件开头，添加以下代码来加载 azure 模块，设置表名称、partitionKey，并设置此示例使用的存储凭据：

		var azure = require('azure-storage');
		var nconf = require('nconf');
		nconf.env()
		     .file({ file: 'config.json'});
		var tableName = nconf.get("TABLE_NAME");
		var partitionKey = nconf.get("PARTITION_KEY");
		var accountName = nconf.get("STORAGE_NAME");
		var accountKey = nconf.get("STORAGE_KEY");

	> [WACOM.NOTE] nconf 将从环境变量或我们稍后将创建的 **config.json** 文件中加载配置值。

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
	
4. 保存 app.js 文件。

###修改索引视图

1. 将目录更改为 views 目录并在文本编辑器中打开 index.jade 文件。

2. 将 index.jade 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

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

3. 保存并关闭 index.jade 文件。

###修改全局布局

views 目录中的 layout.jade 文件用作其他 .jade 文件的全局模板。在此步骤中，你需要修改它以使用 [Twitter Bootstrap](https://github.com/twbs/bootstrap), which is a toolkit that makes it easy to design a nice looking  Website.

1. 下载并提取 [Twitter Bootstrap] 的文件。(http://getbootstrap.com/). Copy the **bootstrap.min.css** file from the **bootstrap\\dist\\css** folder to the **public\\stylesheets** directory of your tasklist application.

2. 从 views 文件夹中，用文本编辑器打开 layout.jade 并将其内容替换为以下代码：

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

3. 保存 layout.jade 文件。

###创建配置文件

config.json 文件包含用于连接到 SQL Database 的连接字符串，应用程序在运行时将读取该文件。若要创建此文件，请执行以下步骤：

1. 在 tasklist 目录中，创建一个名为 config.json 的新文件并在文本编辑器中将其打开。

2. config.json 文件的内容看上去应如下所示：

		{
			"STORAGE_NAME": "storage account name",
			"STORAGE_KEY": "storage access key",
			"PARTITION_KEY": "mytasks",
			"TABLE_NAME": "tasks"
		}

	将 storage account name 替换为你之前创建的存储帐户的名称。将 storage access key 替换为你的存储帐户的主访问密钥。

3. 保存文件。

##在本地运行应用程序

若要在你的本地计算机中测试应用程序，请执行以下步骤：

1. 在命令行中，将目录更改为 tasklist 目录。

2. 使用以下命令在本地启动应用程序：

        npm start

3. 打开 Web 浏览器并导航到 http://127.0.0.1:3000. 此时会显示与下图类似的网页：

    ![A webpage displaying an empty tasklist][node-table-finished]

4. 使用提供的 Item Name（项名称）和 Item Category（项类别）字段输入信息，然后单击 Add item（添加项）。

5. 页面应更新为在 ToDo List 表中显示该项。

    ![An image of the new item in the list of tasks][node-table-list-items]

6. 若要完成任务，只需选中"Complete"（完成）列中的复选框，然后单击 Update tasks（更新任务）。

7. 若要结束 Node 进程，请转到命令行并按 Ctrl 和 C 键。

##将你的应用程序部署到 Azure

本节中的步骤使用 Azure 命令行工具创建一个新的 Azure 网站，然后使用 Git 部署你的应用程序。若要执行这些步骤，你必须具有 Azure 订阅。

> [WACOM.NOTE] 还可以使用 Azure 门户执行这些步骤。有关使用 Azure 门户部署 Node.js 应用程序的步骤，请参阅 [创建 Node.js 应用程序并将其部署到 Azure 网站]。

> [WACOM.NOTE] 如果这是你创建的第一个 Azure 网站，则你必须使用 Azure 门户部署此应用程序。

###创建 Azure 订阅

如果你还没有 Azure 订阅，可以 [免费] 注册。注册后，请按照以下步骤来继续学习本教程。

[WACOM.INCLUDE [antares-iaas-signup](../includes/antares-iaas-signup.md)]

###安装适用于 Mac 和 Linux 的 Azure 命令行工具

若要安装命令行工具，请使用以下命令：
	
	npm install azure-cli -g

> [WACOM.NOTE] 有关详细信息，请参阅 [安装和配置 Azure 跨平台命令行界面](/zh-cn/documentation/articles/xplat-cli/)；

> [WACOM.NOTE] 虽然命令行工具主要针对 Mac 和 Linux 用户而创建，但它们基于 Node.js，应该可在能够运行 Node 的任何系统上使用。

###导入发布设置

在将命令行工具与 Azure 一起使用之前，你必须首先下载包含有关你的订阅的信息的文件。执行以下步骤以下载并导入该文件。

1. 在命令行中，将目录更改为 tasklist 目录。

2. 输入以下命令以启动浏览器并导航到下载页面。如果出现提示，请使用与您的订阅关联的帐户登录。

		azure account download
	
	![The download page][download-publishing-settings]
	
	文件下载应该会自动开始；如果没有自动开始，你可以单击该页面开头的链接手动下载文件。

3. 文件下载完成后，请使用以下命令导入设置：

		azure account import <path-to-file>
		
	Specify the path and file name of the publishing settings file you downloaded in the previous step. Once the command completes, you should see output similar to the following:
	
		info:   Executing command account import
		info:   Setting service endpoint to: management.core.chinacloudapi.cn
		info:   Setting service port to: 443
		info:   Found subscription: YourSubscription
		info:   Setting default subscription to: YourSubscription
		warn:   The 'C:\users\username\downloads\YourSubscription-6-7-2012-credentials.publishsettings' file contains sensitive information.
		warn:   Remember to delete it now that it has been imported.
		info:   Account publish settings imported successfully
		info:   account import command OK

4. 在导入完成后，你应删除发布设置文件，因为不再需要该文件并且它包含有关你的 Azure 订阅的敏感信息。

###创建 Azure 网站

1. 在命令行中，将目录更改为 tasklist 目录。

2. 使用以下命令创建一个新的 Azure 网站

		azure site create --git
		
	系统将提示你输入网站名称以及该网站将位于的数据中心。提供一个唯一名称并选择在地理上接近你的位置的数据中心。
	
	`--git` 参数将在 Azure 中为此网站创建一个 Git 存储库。它还将在当前目录中初始化一个 Git 存储库（如果不存在任何 Git 存储库）。它还将创建一个名为  'azure' 的 [Git remote]，用于将应用程序发布到 Azure。最后，它将创建一个 web.config 文件，其中包含 Azure 用于托管 Node 应用程序的设置。
	
	> [WACOM.NOTE] 如果此命令从已包含 Git 存储库的目录运行，它将不会重新初始化该目录。
	
	> [WACOM.NOTE] 如果省略 `--git` 参数，但该目录包含 Git 存储库，则仍将创建  'azure' remote。
	
	此命令完成后，你将看到与下面类似的输出。请注意，以 Web site created at 开头的行包含网站的 URL。
	
		info:   Executing command site create
		help:   Need a site name
		Name: TableTasklist
		info:   Using location southcentraluswebspace
		info:   Executing `git init`
		info:   Creating default .gitignore file
		info:   Creating a new  Website
		info:   Created  Website at  tabletasklist.azure Websites.net
		info:   Initializing repository
		info:   Repository initialized
		info:   Executing `git remote add azure https://username@tabletasklist.chinacloudsites.cn/TableTasklist.git`
		info:   site create command OK

	> [WACOM.NOTE] 如果这是你订阅的第一个 Azure 网站，系统会指导你使用门户创建网站。有关详细信息，请参阅 [创建 Node.js 应用程序并将其部署到 Azure 网站]。

###发布应用程序

1. 在 Terminal 窗口中，将目录更改为 tasklist 目录（如果你尚未在此目录中）。

2. 使用以下命令将文件添加然后提交到本地 Git 存储库：

		git add .
		git commit -m "adding files"

3. 在将最新 Git 存储库更改推送到 Azure 网站时，你必须指定目标分支为 master，因为这将用于网站内容。

		git push azure master
	
	在部署结束时，你将看到如下语句：
	
		To https://username@tabletasklist.chinacloudsites.cn/TableTasklist.git
 		 * [new branch]      master -> master

4. 推送操作完成后，请浏览到  `azure create site` 命令之前返回的网站 URL 以查看你的应用程序。

###切换到环境变量

前面我们实现了用于查找环境变量或从 config.json 文件中加载值的代码。在接下来的步骤中，你将在网站配置中创建应用程序通过环境变量实际访问的键值对。

1. 从管理门户中，单击"网站"，然后选择你的网站。

	![Open  Website dashboard][go-to-dashboard]

2. 单击"配置"，然后找到页面的"应用程序设置"部分。 

	![configure link][web-configure]

3. 在"应用程序设置"部分的 KEY 字段中输入 STORAGE_NAME，并在 VALUE 字段中输入你的存储帐户的名称。单击复选标记以移到下一个字段。为以下密钥和值重复此过程：

	* STORAGE_KEY - 你的存储帐户的访问密钥
	
	* **PARTITION_KEY** - 'mytasks'

	* **TABLE_NAME** - 'tasks'

	![app settings][app-settings]

4. 最后，单击页面底部的"保存"图标，将此更改提交到运行时环境。

	![app settings save][app-settings-save]

5. 从命令行中，将目录更改为 tasklist 目录，然后输入以下命令以删除 config.json 文件：

		git rm config.json
		git commit -m "Removing config file"

6. 执行以下命令将更改部署到 Azure：

		git push azure master

在将更改部署到 Azure 后，你的 Web 应用程序应当继续工作，因为它现在从"应用程序设置"条目读取连接字符串。若要验证此情况，请在"应用程序设置"中将 STORAGE_KEY 条目的值更改为一个无效值。保存该值后，网站应该会因存储访问密钥设置无效而失败。

##后续步骤

虽然本文中的步骤介绍了使用表服务来存储信息，但你也可以使用 MongoDB。有关详细信息，请参阅 [使用 MongoDB 的 Node.js Web 应用程序]。

##其他资源

[针对 Mac 和 Linux 的 Azure 命令行工具]    
[创建 Node.js 应用程序并将其部署到 Azure 网站]: /zh-cn/documentation/articles/web-sites-nodejs-develop-deploy-mac/
[使用 Git 发布到 Azure 网站]: /zh-cn/documentation/articles/web-sites-publish-source-control/
[Azure 开发人员中心]: /zh-cn/develop/nodejs/


[节点]: http://nodejs.org
[Git]: http://git-scm.com
[Express]: http://expressjs.com
[免费]: http://www.windowsazure.cn
[Git remote]: http://git-scm.com/docs/git-remote

[使用 MongoDB 构建 Node.js Web 应用程序]: /zh-cn/documentation/articles/web-sites-nodejs-store-data-mongodb/
[针对 Mac 和 Linux 的 Azure 命令行工具]: /zh-cn/documentation/articles/xplat-cli/

[使用 Git 发布到 Azure 网站]: /zh-cn/documentation/articles/web-sites-publish-source-control/
[azure]: https://github.com/Azure/azure-sdk-for-node


[Azure 门户]: http://www.windowsazure.cn


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

[创建 Node.js 应用程序并将其部署到 Azure 网站]: /zh-cn/documentation/articles/web-sites-nodejs-develop-deploy-mac/<!--HONumber=41-->
