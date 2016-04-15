<properties 
	pageTitle="使用表存储构建 Web 应用 (Node.js) | Azure" 
	description="本教程以“使用 Express 构建 Web 应用程序”教程为基础，演示如何添加 Azure 存储服务和 Azure 模块。" 
	services="cloud-services, storage" 
	documentationCenter="nodejs" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="01/09/2016" 
	wacn.date="03/24/2016"/>

# 使用存储构建 Node.js Web 应用程序

## 概述

在本教程中，你将通过将用于 Node.js 的 Azure 客户端库与数据管理服务结合使用，来扩展在[使用 Express 生成 Node.js Web 应用程序]教程中创建的应用程序。你将扩展你的应用程序以创建可部署到 Azure 的基于 Web 的任务列表应用程序。用户可以通过任务列表来检索任务、添加新任务以及将任务标记为已完成。

任务项存储在 Azure 存储空间中。Azure 存储空间提供了具有容错能力且可用性非常好的非结构化数据存储。Azure 存储空间包含一些可用来存储和访问数据的数据结构，你可以通过 Azure SDK for Node.js 中包含的 API 或通过 REST API 利用存储服务。有关详细信息，请参阅[在 Azure 中存储和访问数据]。

本教程假定你已完成 [Node.js Web 应用程序]和[使用 Express 生成 Node.js][Node.js Web Application using Express] 教程。

你将学习以下内容：

-   如何操作 Jade 模板引擎
-   如何操作 Azure 数据管理服务

以下是已完成应用程序的屏幕快照：

![Internet Explorer 中已完成的网页](./media/storage-nodejs-use-table-storage-cloud-service-app/getting-started-1.png)

## 在 Web.Config 中设置存储凭据

若要访问 Azure 存储空间，你需要传入存储凭据。为此，您将使用 web.config 应用程序设置。这些设置将作为环境变量传递给 Node，然后再由 Azure SDK 进行读取。

> [AZURE.NOTE] 仅在将应用程序部署到 Azure 时才使用存储凭据。应用程序在模拟器中运行时将使用存储模拟器。

执行下列步骤可检索存储帐户凭据并将这些凭据添加到 web.config 设置中：

1.  如果尚未打开 Azure PowerShell，请通过在“开始”菜单中展开“所有程序”、“Azure”，右键单击“Azure PowerShell”，然后选择“以管理员身份运行”启动 Azure PowerShell。

2.  将目录更改到包含你的应用程序的文件夹。例如，C:\\node\\tasklist\\WebRole1。

3.  从 Azure Powershell 窗口中，输入以下 cmdlet 以检索存储帐户信息：

        PS C:\node\tasklist\WebRole1> Get-AzureStorageAccounts

	这样可以检索与托管服务关联的存储帐户和帐户密钥的列表。

	> [AZURE.NOTE] 由于在你部署服务时 Azure SDK 会创建一个存储帐户，因此在前面的指南中部署你的应用程序之后应当已存在一个存储帐户。

4.  打开 **ServiceDefinition.csdef** 文件，该文件包含将应用程序部署到 Azure 时所使用的环境设置：

        PS C:\node\tasklist> notepad ServiceDefinition.csdef

5.  在 **Environment** 元素下插入以下块，使用你要用于部署的存储帐户的帐户名称和主密钥替换 {STORAGE ACCOUNT} 和 {STORAGE ACCESS KEY}：

        <Variable name="AZURE_STORAGE_ACCOUNT" value="{STORAGE ACCOUNT}" />
        <Variable name="AZURE_STORAGE_ACCESS_KEY" value="{STORAGE ACCESS KEY}" />

	![web.cloud.config 文件内容](./media/storage-nodejs-use-table-storage-cloud-service-app/node37.png)

6.  保存该文件并关闭记事本。

### 安装其他模块

2. 使用以下命令在本地安装 azure、[node-uuid]、[nconf] 和 [async] 模块，并将它们的一个条目保存到 **package.json** 文件：

		PS C:\node\tasklist\WebRole1> npm install azure-storage node-uuid async nconf --save

	此命令的输出看上去应如下所示：

		node-uuid@1.4.1 node_modules\node-uuid

		nconf@0.6.9 node_modules\nconf
		├── ini@1.1.0
		├── async@0.2.9
		└── optimist@0.6.0 (wordwrap@0.0.2, minimist@0.0.8)

        azure-storage@0.1.0 node_modules\azure-storage
		├── extend@1.2.1
		├── xmlbuilder@0.4.3
		├── mime@1.2.11
		├── underscore@1.4.4
		├── validator@3.1.0
		├── node-uuid@1.4.1
		├── xml2js@0.2.7 (sax@0.5.2)
		└── request@2.27.0 (json-stringify-safe@5.0.0, tunnel-agent@0.3.0, aws-sign@0.3.0, forever-agent@0.5.2, qs@0.6.6, oauth-sign@0.3.0, cookie-jar@0.3.0, hawk@1.0.0, form-data@0.1.3, http-signature@0.10.0)

##在 Node 应用程序中使用表服务

在本节中，你将通过添加一个包含你的任务模型的 **task.js** 文件来扩展 **express** 命令创建的基本应用程序。你还将修改现有 **app.js** 并创建使用该模型的新 **tasklist.js** 文件。

### 创建模型

1. 在 **WebRole1** 目录中，创建一个名为 **models** 的新目录。

2. 在 **models** 目录中，创建一个名为 **task.js** 的新文件。此文件将包含你的应用程序创建的任务的模型。

3. 在 **task.js** 文件的开头，添加以下代码来引用所需的库：

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
		    self.storageClient.queryEntities(query, function entitiesQueried(error, result) {
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

6. 保存并关闭 **task.js** 文件。

### 创建控制器

1. 在 **WebRole1/routes** 目录中，创建一个名为 **tasklist.js** 的新文件并在文本编辑器中将其打开。

2. 将以下代码添加到 **tasklist.js**。这将加载 **tasklist.js** 使用的 azure 和 async 模块。这还将定义 **TaskList** 函数，将向该函数传递我们之前定义的 **Task** 对象的一个实例：

		var azure = require('azure-storage');
		var async = require('async');

		module.exports = TaskList;

		function TaskList(task) {
		  this.task = task;
		}

2. 继续向 **tasklist.js** 文件添加用于**showTasks**、**addTask** 和 **completeTasks** 的方法：

		TaskList.prototype = {
		  showTasks: function(req, res) {
		    self = this;
		    var query = azure.TableQuery()
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

3. 保存 **tasklist.js** 文件。

### 修改 app.js

1. 在 **WebRole1** 目录中，用文本编辑器打开 **app.js** 文件。 

2. 在该文件的开头，添加以下内容来加载 azure 模块并设置表名称和分区键：

		var azure = require('azure-storage');
		var tableName = 'tasks';
		var partitionKey = 'hometasks';

3. 在 app.js 文件中，向下滚动到以下行：

		app.use('/', routes);
		app.use('/users', users);

	将上面的行替换为下面显示的代码。这将通过与你的存储帐户的连接初始化 <strong>Task</strong> 的实例。这是 <strong>TaskList</strong> 的密码，TaskList 将使用该密码与表服务进行通信：

		var TaskList = require('./routes/tasklist');
		var Task = require('./models/task');
		var task = new Task(azure.createTableService(), tableName, partitionKey);
		var taskList = new TaskList(task);

		app.get('/', taskList.showTasks.bind(taskList));
		app.post('/addtask', taskList.addTask.bind(taskList));
		app.post('/completetask', taskList.completeTask.bind(taskList));
	
4. 保存 **app.js** 文件。

### 修改索引视图

1. 将目录更改为 **views** 目录并在文本编辑器中打开 **index.jade** 文件。

2. 将 **index.jade** 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

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
		      if tasks != []
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

**views** 目录中的 **layout.jade** 文件用作其他 .**jade** 文件的全局模板。在此步骤中，你将对其进行修改以使用 [Twitter Bootstrap](https://github.com/twbs/bootstrap)（一个可以轻松设计美观 Web 应用的工具包）。

1. 下载并提取 [Twitter Bootstrap](http://getbootstrap.com) 的文件。将 **bootstrap.min.css** 文件从 **bootstrap\\dist\\css** 文件夹复制到你的 tasklist 应用程序的 **public\\stylesheets** 目录中。

2. 从 **views** 文件夹中，用文本编辑器打开 **layout.jade** 并将其内容替换为以下代码：

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

3. 保存 **layout.jade** 文件。

### 在模拟器中运行应用程序

使用以下命令在模拟器中启动应用程序。

	PS C:\node\tasklist\WebRole1> start-azureemulator -launch

浏览器将打开并显示以下页面：

![标题为 My Task List 的网页，其中所含的表包含任务和用于添加新任务的字段。](./media/storage-nodejs-use-table-storage-cloud-service-app/node44.png)

使用窗体添加项，或通过将其标记为完成来删除现有项。

## 将应用程序发布到 Azure


在 Windows PowerShell 窗口中，调用以下 cmdlet 将托管服务重新部署到 Azure。

    PS C:\node\tasklist\WebRole1> Publish-AzureServiceProject -name myuniquename -location datacentername -launch

将 **myuniquename** 替换为此应用程序的唯一名称。将 **datacentername** 替换为 Azure 数据中心的名称，例如“中国北部”。

部署完成后，你将看到如下响应：

	PS C:\node\tasklist> publish-azureserviceproject -servicename tasklist -location "China North"
	WARNING: Publishing tasklist to Azure. This may take several minutes...
	WARNING: 2:18:42 PM - Preparing runtime deployment for service 'tasklist'
	WARNING: 2:18:42 PM - Verifying storage account 'tasklist'...
	WARNING: 2:18:43 PM - Preparing deployment for tasklist with Subscription ID: 65a1016d-0f67-45d2-b838-b8f373d6d52e...
	WARNING: 2:19:01 PM - Connecting...
	WARNING: 2:19:02 PM - Uploading Package to storage service larrystore...
	WARNING: 2:19:40 PM - Upgrading...
	WARNING: 2:22:48 PM - Created Deployment ID: b7134ab29b1249ff84ada2bd157f296a.
	WARNING: 2:22:48 PM - Initializing...
	WARNING: 2:22:49 PM - Instance WebRole1_IN_0 of role WebRole1 is ready.
	WARNING: 2:22:50 PM - Created Website URL: http://tasklist.chinacloudapp.cn/.

与先前一样，由于你指定了 **-launch** 选项，因此在发布完成后，浏览器将打开并显示正在 Azure 中运行的应用程序。

![浏览器窗口中显示 My Task List 页面。URL 表明该页面现在托管在 Azure 上。](./media/storage-nodejs-use-table-storage-cloud-service-app/getting-started-1.png)

## 停止并删除应用程序

在部署应用程序后，你可能希望禁用它，以避免在试用期内产生费用或生成和部署其他应用程序。

Azure 将按使用的服务器小时数对 Web 角色实例计费。你的应用程序部署之后就会开始使用服务器时间，即使相关实例并未运行且处于停止状态也是如此。

以下步骤演示了如何停止和删除应用程序。

1.  在 Windows PowerShell 窗口中，使用以下 cmdlet 以停止上一节中创建的服务部署：

        PS C:\node\tasklist\WebRole1> Stop-AzureService

	停止服务可能需要花费几分钟时间。在服务停止时，你会收到一条指示服务已停止的消息。

3.  若要删除服务，请调用以下 cmdlet：

        PS C:\node\tasklist\WebRole1> Remove-AzureService contosotasklist

	在出现提示时，输入 **Y** 以删除服务。

	删除服务可能需要花费几分钟时间。删除服务后，你将收到一条指示服务已被删除的消息。

  [Node.js Web Application using Express]: /documentation/articles/cloud-services-nodejs-develop-deploy-express-app/
  [使用 Express 生成 Node.js Web 应用程序]: /documentation/articles/cloud-services-nodejs-develop-deploy-express-app/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Node.js Web 应用程序]: /documentation/articles/cloud-services-nodejs-develop-deploy-app/
 

<!---HONumber=Mooncake_0215_2016-->