<properties linkid="develop-node-website-with-mongodb-mac" urlDisplayName="Web site with MongoDB" pageTitle="在 VM 上使用 MongoDB 构建 Node.js 网站 - Azure 教程" metaKeywords="Azure tutorial MongoDB, MongoDB store data, access data MongoDB Node, Azure Node app" description="本教程将介绍如何使用 MongoDB 在托管在 Azure 上的 Node 应用程序中存储和访问数据。" metaCanonical="http://azure.microsoft.com/zh-cn/documentation/articles/store-mongolab-web-sites-nodejs-store-data-mongodb/" services="web-sites,virtual-machines" documentationCenter="Node.js" title="使用 MongoDB 中的存储的 Node.js Web 应用程序（虚拟机）" authors="larryfr"  solutions="" writer="" manager="" editor=""  />

# 在虚拟机中使用 MongoDB 在 Azure 上创建 Node.js 应用程序

本教程将介绍如何使用托管在 Azure 虚拟机上的 [MongoDB][MongoDB] 存储数据，并从托管在 Azure 网站上的 [Node][Node] 应用程序访问数据。[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。

你将了解到以下内容：

-   如何从 VM Depot 设置运行 Ubuntu 和 MongoDB 的虚拟机。
-   如何从 Node 应用程序中访问 MongoDB
-   如何使用 Azure 跨平台工具创建 Azure 网站

按照本教程中的说明操作，你将构建一个简单的基于 Web 的任务管理应用程序，该应用程序可用于创建、检索和完成任务。这些任务存储在 MongoDB 中。

> [WACOM.NOTE] 本教程使用安装在虚拟机上的 MongoDB 实例。如果您更喜欢使用 MongoLabs 提供的托管的 MongoDB 实例，请参阅[使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序][使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序]。

本教程中的项目文件将存储在名为 **tasklist** 的目录中，已完成的应用程序将与下图类似：

![显示空白 tasklist 的网页][显示空白 tasklist 的网页]

> [WACOM.NOTE] 下面的许多步骤都提到使用命令行。对于这些步骤，请使用适用于您的操作系统的命令行，例如 **Windows PowerShell** (Windows) 或 **Bash** (Unix Shell)。在 OS X 系统上，可以通过 Terminal 应用程序访问命令行。

## 先决条件

本教程中，使用 Node.js 的步骤必须确保开发环境中安装了最新版本的 [Node.js][Node]。

此外，[Git][Git] 必须可以在开发环境中从命令行访问，因为 Git 用于将应用程序部署到 Azure 网站。

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 创建虚拟机

<!--This tutorial assumes you have created a virtual machine in Azure. After creating the virtual machine you need to install MongoDB on the virtual machine:  * To create a Linux virtual machine and install MongoDB, see [Installing MongoDB on a Linux Virtual machine].  After you have created the virtual machine in Azure and installed MongoDB, be sure to remember the DNS name of the virtual machine ("testlinuxvm.chinacloudapp.cn", for example) and the external port for MongoDB that you specified in the endpoint.  You will need this information later in the tutorial.-->

尽管可以创建新 VM，然后根据 [MongoDB 安装指南][MongoDB 安装指南]安装 MongoDB，但这项工作的大部分已由社区执行，且在 VM Depot 中可用。以下步骤将介绍如何使用来自已安装和配置 Mongo DB 的 VM Depot 的映像。

1.  登录到“Azure 管理门户”[][]，选择“虚拟机”，再选择“映像”，然后选择“VM Depot”。

    ![选择 VM Depot 的屏幕截图][选择 VM Depot 的屏幕截图]

2.  选择包含 MongoDB 的映像。在此案例中，我选择了 Ubuntu，以将列表缩小到只包含基于 Ubuntu Linux 分发的映像。最后，我选择了基于强化的 Ubuntu 映像的 MongoDB v2.2.3。

    ![选择的基于强化的 Ubuntu 映像的 MongoDB v2.2.3 的屏幕截图][选择的基于强化的 Ubuntu 映像的 MongoDB v2.2.3 的屏幕截图]

    > [WACOM.NOTE] 请确保选择“更多”，以查看映像的所有信息。某些映像可能包含需要在使用该映像创建 VM 之后执行的额外配置。

    单击底部的箭头继续到下一个屏幕。

3.  选择用于存储此映像的 VHD 的地区和存储帐户。单击复选标记以继续。

    ![选择存储帐户的屏幕截图][选择存储帐户的屏幕截图]

    > [WACOM.NOTE] 这将会启动一个将映像从 VM Depot 复制到指定存储帐户的复制过程。此过程可能需要花费较长时间，15 分钟或更长时间。

4.  映像的状态更改为“挂起的注册”之后，选择“注册”，然后为新映像输入一个友好名称。单击复选标记以继续。

    ![注册映像的屏幕截图][注册映像的屏幕截图]

5.  映像的状态更改为“可用”之后，选择“+ 新增”、“虚拟机”、“从库中”。系统要求“选择映像”时，选择“我的映像”，然后选择在之前的步骤中创建的映像。单击箭头以继续。

    ![映像的屏幕截图][映像的屏幕截图]

6.  提供 VM 名称、大小和用户名。单击箭头以继续。

    ![VM 名称、用户名等的屏幕截图][VM 名称、用户名等的屏幕截图]

    > [WACOM.NOTE] 对于本教程，您无需使用 SSH 远程连接到您的 VM。如果您对通过 SSH 使用证书不熟悉，请选择“使用密码”，并提供一个密码。
    >
    > 有关如何在安装了 Linux VM 的 Azure 上使用 SSH 的详细信息，请参阅[如何在安装了 Linux 的 Azure 上使用 SSH][如何在安装了 Linux 的 Azure 上使用 SSH]。

7.  选择是使用新的云服务还是使用现有云服务，并选择要创建 VM 的区域。单击箭头继续操作。

    ![VM 配置的屏幕截图][VM 配置的屏幕截图]

8.  为 VM 设置额外终结点。由于我们要在此 VM 上访问 MongoDB，请使用以下信息添加一个新终结点：

    -   名称 - MongoDB
    -   协议 - TCP
    -   公用端口 - 27017
    -   专用端口 - 27017

    要公开 MongoDB Web 门户，请使用以下信息添加另一个终结点：

    -   名称 - MongoDBWeb
    -   协议 - TCP
    -   公用端口 - 28017
    -   专用端口 - 28017

    最后选择复选标记以配置虚拟机。

    ![终结点配置的屏幕截图][终结点配置的屏幕截图]

9.  虚拟机的状态更改为“正在运行”之后，您应该可以打开 Web 浏览器并访问 **http://\<您的 VM DNS 名称\>.chinacloudapp.cn:28017/**，以确认 MongoDB 正在运行。该页面底部的日志将显示服务相关信息，类似于以下内容：

        Fri Mar  7 18:57:16 [initandlisten] MongoDB starting : pid=1019 port=27017 dbpath=/var/lib/mongodb 64-bit host=localhost.localdomain
           18:57:16 [initandlisten] db version v2.2.3, pdfile version 4.5
           18:57:16 [initandlisten] git version: f570771a5d8a3846eb7586eaffcf4c2f4a96bf08
           18:57:16 [initandlisten] build info: Linux ip-10-2-29-40 2.6.21.7-2.ec2.v1.2.fc8xen #1 SMP Fri Nov 20 17:48:28 EST 2009 x86_64 BOOST_LIB_VERSION=1_49
           18:57:16 [initandlisten] options: { config: "/etc/mongodb.conf", dbpath: "/var/lib/mongodb", logappend: "true", logpath: "/var/log/mongodb/mongodb.log" }
           18:57:16 [initandlisten] journal dir=/var/lib/mongodb/journal
           18:57:16 [initandlisten] recover : no journal files present, no recovery needed
           18:57:17 [initandlisten] waiting for connections on port 27017

    如果日志显示错误，请参阅 [MongoDB 文档][MongoDB 文档]查看故障排除步骤。

## 安装模块并生成基架

在本部分中，您将在开发环境中创建一个新的 Node 应用程序并使用 npm 添加模块包。对于任务列表应用程序，您将使用 [Express][Express] 和 [Mongoose][Mongoose] 模块。Express 模块为 Node 提供模型视图控制器框架，而 Mongoose 是与 MongoDB 通信的驱动程序。

### 安装 Express 并生成基架

1.  在命令行中，将目录更改为 **tasklist** 目录。如果 **tasklist** 目录不存在，请创建该目录。

    > [WACOM.NOTE] 本教程将引用 **tasklist** 文件夹。将省略此文件夹的完整路径，因为路径语义在操作系统之间并不相同。你应在本地文件系统中易于访问的位置创建此文件夹（例如 **~/node/tasklist** 或 **c:\\node\\tasklist**）

2.  请输入以下命令来安装 Express 命令。

    npm install express-generator -g

    > [WACOM.NOTE] 在某些操作系统上使用“-g”参数时，您可能会收到一条错误 ***Error:EPERM, chmod '/usr/local/bin/express'*** 和一个尝试以管理员身份运行帐户的请求。如果发生这种情况，请使用 `sudo` 命令在更高的权限级别运行 npm。

    此命令的输出看上去应如下所示：

        express-generator@4.0.0 C:\Users\username\AppData\Roaming\npm\node_modules\express-generator
        ├── mkdirp@0.3.5
        └── commander@1.3.2 (keypress@0.1.0)                                                                         

    > [WACOM.NOTE] 在安装 Express 模块时使用“-g”参数将全局安装该模块。这样我们就可以访问 ***express*** 命令以生成网站基架，而无需键入其他路径信息。

3.  若要创建将用于此应用程序的基架，请使用 **express** 命令：

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

    此命令完成后，你应当会在 **tasklist** 目录中拥有几个新目录和文件。

4.  将 **tasklist/bin/www** 文件复制到 **tasklist** 文件夹中名为 **server.js** 的文件。Azure 网站预期 Node.js 应用程序的入口点为 **server.js** 或 **app.js**。由于 **app.js** 已存在，但它不是入口点，因此我们必须使用 **server.js**。

5.  修改 **server.js** 文件以从下行中删除其中一个“.”字符。

        var app = require('../app');

    修改后的行应当如下所示。

        var app = require('./app');

    这是必需的，因为 **server.js**（以前的 **bin/www**）现在与必需的 **app.js** 文件位于相同的文件夹中。

### 安装其他模块

**package.json** 文件是 **express** 命令创建的文件之一。此文件包含 Express 应用程序所需的其他模块的列表。稍后，在你将此应用程序部署到 Azure 网站时，将使用此文件确定需要在 Azure 上安装哪些模块来支持你的应用程序。

1.  从 **tasklist** 文件夹中，使用以下命令安装 **package.json** 文件中所述的模块：

        npm install

    此命令的输出看上去应如下所示：

        debug@0.7.4 node_modules\debug

        cookie-parser@1.0.1 node_modules\cookie-parser
        ├── cookie-signature@1.0.3
        └── cookie@0.1.0

        morgan@1.0.0 node_modules\morgan
        └── bytes@0.2.1

        body-parser@1.0.2 node_modules\body-parser
        ├── qs@0.6.6
        ├── raw-body@1.1.4 (bytes@0.3.0)
        └── type-is@1.1.0 (mime@1.2.11)

        express@4.0.0 node_modules\express
        ├── methods@0.1.0
        ├── parseurl@1.0.1
        ├── merge-descriptors@0.0.2
        ├── utils-merge@1.0.0
        ├── escape-html@1.0.1
        ├── cookie-signature@1.0.3
        ├── fresh@0.2.2
        ├── range-parser@1.0.0
        ├── buffer-crc32@0.2.1
        ├── qs@0.6.6
        ├── cookie@0.1.0
        ├── path-to-regexp@0.1.2
        ├── send@0.2.0 (mime@1.2.11)
        ├── type-is@1.0.0 (mime@1.2.11)
        ├── accepts@1.0.0 (negotiator@0.3.0, mime@1.2.11)
        └── serve-static@1.0.1 (send@0.1.4)

        jade@1.3.1 node_modules\jade
        ├── character-parser@1.2.0
        ├── commander@2.1.0
        ├── mkdirp@0.3.5
        ├── monocle@1.1.51 (readdirp@0.2.5)
        ├── constantinople@2.0.0 (uglify-js@2.4.13)
        ├── with@3.0.0 (uglify-js@2.4.13)
        └── transformers@2.1.0 (promise@2.0.0, css@1.0.8, uglify-js@2.2.5)                                                                

    这将安装 Express 应用程序使用的所有默认模块。

2.  接下来，输入以下命令在本地安装 Mongoose 模块并将它的一个条目保存到 **package.json** 文件：

        npm install mongoose --save

    此命令的输出看上去应如下所示：

        mongoose@3.8.8 node_modules\mongoose
        ├── regexp-clone@0.0.1
        ├── sliced@0.0.5
        ├── muri@0.3.1
        ├── hooks@0.2.1
        ├── mpath@0.1.1
        ├── mpromise@0.4.3
        ├── ms@0.1.0
        ├── mquery@0.5.3
        └── mongodb@1.3.23 (kerberos@0.0.3, bson@0.2.5)         

    > [WACOM.NOTE] 您可以放心地忽略有关安装 C++ bson 分析器的任何消息。

## 在 Node 应用程序中使用 MongoDB

在本节中，你将通过添加一个包含你的任务模型的 **task.js** 文件来扩展 **express** 命令创建的基本应用程序。你还将修改现有 **app.js** 并创建新的 **tasklist.js** 控制器文件以使用该模型。

### 创建模型

1.  在 **tasklist** 目录中，创建一个名为 **models** 的新目录。

2.  在 **models** 目录中，创建一个名为 **task.js** 的新文件。此文件将包含你的应用程序创建的任务的模型。

3.  在 **task.js** 文件的开头，添加以下代码来引用所需的库：

        var mongoose = require('mongoose'),
            Schema = mongoose.Schema;

4.  接下来，添加代码以定义和导出模型。此模型将用于执行与 MongoDB 数据库的交互。

        var TaskSchema = new Schema({
            itemName      : String,
            itemCategory  : String,
            itemCompleted : { type: Boolean, default: false },
            itemDate      : { type: Date, default: Date.now }
        });

        module.exports = mongoose.model('TaskModel', TaskSchema);

5.  保存并关闭 **task.js** 文件。

### 创建控制器

1.  在 **tasklist/routes** 目录中，创建一个名为 **tasklist.js** 的新文件并在文本编辑器中将其打开。

2.  将以下代码添加到 **tasklist.js**。这将加载 mongoose 模块和 **task.js** 中定义的任务模型。TaskList 函数用于根据 **connection** 值创建与 MongoDB 服务器的连接：

        var mongoose = require('mongoose'),
            task = require('../models/task.js');

        module.exports = TaskList;

        function TaskList(connection) {
          mongoose.connect(connection);
        }

3.  继续向 **tasklist.js** 文件添加用于 **showTasks**、**addTask** 和 **completeTasks** 的方法：

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

4.  保存 **tasklist.js** 文件。

### 修改 app.js

1.  在 **tasklist** 目录中，用文本编辑器打开 **app.js** 文件。此文件之前是通过运行 **express** 命令创建的。

2.  将以下代码添加到 **app.js** 文件的开头。这将使用 MongoDB 服务器的连接字符串初始化 **TaskList**：

        var TaskList = require('./routes/tasklist');
        var taskList = new TaskList(process.env.MONGODB_URI);

    请注意第二行；你访问将在以后配置的环境变量，该变量包含针对你的 mongo 实例的连接信息。如果您有一个出于开发目的运行的本地 mongo 实例，您可能希望将此值暂时设置为“localhost”而不是 process.env.MONGODB\_URI。

3.  查找以下行：

        app.use('/', routes);
        app.use('/users', users);

    将上述两行替换为以下内容：

        app.get('/', taskList.showTasks.bind(taskList));
        app.post('/addtask', taskList.addTask.bind(taskList));
        app.post('/completetask', taskList.completeTask.bind(taskList));

    这将添加在 **tasklist.js** 中定义为路由的函数。

4.  保存 **app.js** 文件。

### 修改索引视图

1.  将目录更改为 **views** 目录并在文本编辑器中打开 **index.jade** 文件。

2.  将 **index.jade** 文件的内容替换为以下代码。这将定义用于显示现有任务的视图，以及用于添加新任务和将现有任务标记为已完成的表单。

        h1= title
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

<!-- ##Run your application locally  To test the application on your local machine, perform the following steps:  1. From the command-line, change directories to the **tasklist** directory.  2. Set the MONGODB_URI environment variable on your development environment to point to the virtual machine hosting MongoDB. In the examples below, replace __mymongodb__ with your virtual machine name.      On a Windows system, use the following to set the environment variable.          set MONGODB_URI=mongodb://mymongodb.chinacloudapp.cn/tasks      On an OS X or Linux-based system, use the following to set the environment variable.          set MONGODB_URI=mongodb://mymongodb.chinacloudapp.cn/tasks         export MONGODB_URI      This will instruct the application to connect to MongoDB on the __mymongodb.cloudapp.net__ virtual machine created earlier, and to use a DB named 'tasks'.  2. Use the following command to launch the application locally:          node app.js  3. Open a web browser and navigate to http://localhost:3000. This should display a web page similar to the following:      ![A webpage displaying an empty tasklist][node-mongo-finished]  4. Use the provided fields for **Item Name** and **Item Category** to enter information, and then click **Add item**.      ![An image of the add item field with populated values.][node-mongo-add-item]  5. The page should update to display the item in the ToDo List table.      ![An image of the new item in the list of tasks][node-mongo-list-items]  6. To complete a task, simply check the checkbox in the Complete column, and then click **Update tasks**. While there is no visual change after clicking **Update tasks**, the document entry in MongoDB has now been marked as completed.  7. To stop the node process, go to the command-line and press the **CTRL** and **C** keys. -->

## 将你的应用程序部署到 Azure

本节中的步骤使用 Azure 命令行工具创建一个新的 Azure 网站，然后使用 Git 部署你的应用程序。若要执行这些步骤，你必须具有 Azure 订阅。

> [WACOM.NOTE] 还可以使用 Azure 门户执行这些步骤。有关使用 Azure 门户部署 Node.js 应用程序的步骤，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][创建 Node.js 应用程序并将其部署到 Azure 网站]。

> [WACOM.NOTE] 如果这是您创建的第一个 Azure 网站，则您必须使用 Azure 门户部署此应用程序。

### 安装 Azure 跨平台命令行接口

Azure 跨平台命令行接口 (xplat-cli) 用于为 Azure 服务执行管理操作。如果您的开发环境中尚未安装和配置 xplat-cli，有关说明请参阅[安装和配置 Azure 跨平台命令行接口][安装和配置 Azure 跨平台命令行接口]。

### 创建 Azure 网站

1.  在命令行中，将目录更改为 **tasklist** 目录。

2.  使用以下命令创建一个新的 Azure 网站。将“myuniquesitename”替换为您的网站的唯一站点名称。此值用作生成的网站的 URL 的一部分。

        azure site create myuniquesitename --git

    系统将提示您输入该站点将位于的数据中心。选择在地理上接近您的位置的数据中心。

    `--git` 参数将在 **tasklist** 文件夹中本地创建 Git 存储库（如果不存在任何 Git 存储库）。它还将创建一个名为“azure”的 [Git remote][Git remote]，用于将应用程序发布到 Azure。它将创建一个 [iisnode.yml][iisnode.yml]，其中包含 Azure 用于托管 Node 应用程序的设置。最后，它还将创建一个 .gitignore 文件，以防止将 node-modules 文件夹发布到 .git。

    > [WACOM.NOTE] 如果此命令从已包含 Git 存储库的目录运行，它将不会重新初始化该目录。

    > [WACOM.NOTE] 如果省略“--git”参数，但该目录包含 Git 存储库，则仍将创建“azure”remote。

    此命令完成后，你将看到与下面类似的输出。请注意，以 **Created web site at** 开头的行包含网站的 URL。

        info:   Executing command site create
        info:   Using location southcentraluswebspace
        info:   Executing `git init`
        info:   Creating default web.config file
        info:   Creating a new web site
        info:   Created web site at  mongodbtasklist.chinacloudsites.cn
        info:   Initializing repository
        info:   Repository initialized
        info:   Executing `git remote add azure http://username@mongodbtasklist.chinacloudsites.cn/mongodbtasklist.git`
        info:   site create command OK

    > [WACOM.NOTE] 如果这是您订阅的第一个 Azure 网站，系统会指导您使用门户创建网站。有关详细信息，请参阅[创建 Node.js 应用程序并部署到 Azure 网站][创建 Node.js 应用程序并将其部署到 Azure 网站]。

### 设置 MONGODB\_URI 环境变量

应用程序需要 MongoDB 实例的连接字符串可用于 MONGODB\_URI 环境变量中。要为网站设置该值，请使用以下命令：

    azure site config add MONGODB_URI=mongodb://mymongodb.chinacloudapp.cn/tasks

这将为网站创建一个新的应用程序设置，该设置将用于填充网站要读取的 MONGODB\_URI 环境变量。将值“mymongodb.chinacloudapp.cn”替换为用于安装 MongoDB 的虚拟机的名称。

### 发布应用程序

1.  在 Terminal 窗口中，将目录更改为 **tasklist** 目录（如果你尚未在此目录中）。

2.  使用以下命令将文件添加然后提交到本地 Git 存储库：

        git add .
        git commit -m "adding files"

3.  在将最新 Git 存储库更改推送到 Azure 网站时，你必须指定目标分支为 **master**，因为这将用于网站内容。

        git push azure master

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

4.  推送操作完成后，请通过使用 `azure site browse` 命令来浏览到网站以查看您的应用程序。

## 后续步骤

虽然本文中的步骤介绍的是使用 MongoDB 存储信息，但您也可以使用 Azure 表服务。有关详细信息，请参阅[使用 Azure 表服务的 Node.js Web 应用程序][使用 Azure 表服务的 Node.js Web 应用程序]。

要了解如何使用 MongoLab 提供的托管的 MongoDB 实例，请参阅[使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序][使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序]。

要了解如何保护 MongoDB，请参阅 [MongoDB 安全][MongoDB 安全]。

## 其他资源

[适用于 Mac 和 Linux 的 Azure 命令行工具][创建 Node.js 应用程序并将其部署到 Azure 网站]
[使用 Git 发布到 Azure 网站][使用 Git 发布到 Azure 网站]

  [MongoDB]: http://www.mongodb.org
  [Node]: http://nodejs.org
  [使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 Node.js 应用程序]: /zh-cn/develop/nodejs/tutorials/website-with-mongodb-mongolab/
  [显示空白 tasklist 的网页]: ./media/store-mongodb-web-sites-nodejs-use-mac/todo_list_empty.png
  [Git]: http://git-scm.com
  [MongoDB 安装指南]: http://docs.mongodb.org/manual/installation/
  []: https://manage.windowsazure.cn/
  [选择 VM Depot 的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/browsedepot.png
  [选择的基于强化的 Ubuntu 映像的 MongoDB v2.2.3 的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/selectimage.png
  [选择存储帐户的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/storageaccount.png
  [注册映像的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/register.png
  [映像的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/myimages.png
  [VM 名称、用户名等的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/vmname.png
  [如何在安装了 Linux 的 Azure 上使用 SSH]: http://azure.microsoft.com/zh-cn/documentation/articles/linux-use-ssh-key/
  [VM 配置的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/vmconfig.png
  [终结点配置的屏幕截图]: ./media/web-sites-nodejs-store-data-mongodb/endpoints.png
  [MongoDB 文档]: http://docs.mongodb.org/manual/
  [Express]: http://expressjs.com
  [Mongoose]: http://mongoosejs.com
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /zh-cn/develop/nodejs/tutorials/create-a-website-(mac)/
  [安装和配置 Azure 跨平台命令行接口]: /zh-cn/documentation/articles/xplat-cli/
  [Git remote]: http://git-scm.com/docs/git-remote
  [iisnode.yml]: https://github.com/WindowsAzure/iisnode/blob/master/src/samples/configuration/iisnode.yml
  [使用 Azure 表服务的 Node.js Web 应用程序]: /zh-cn/develop/nodejs/tutorials/web-site-with-storage/
  [MongoDB 安全]: http://docs.mongodb.org/manual/security/
  [使用 Git 发布到 Azure 网站]: /zh-cn/develop/nodejs/common-tasks/publishing-with-git/
