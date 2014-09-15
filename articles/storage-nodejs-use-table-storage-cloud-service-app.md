<properties linkid="dev-nodejs-basic-web-app-with-storage" urlDisplayName="Web App with Storage" pageTitle="Web app with table storage (Node.js) | Microsoft Azure" metaKeywords="Azure Node.js hello world tutorial, Azure Node.js hello world, Azure Node.js Getting Started tutorial, Azure Node.js tutorial, Azure Node.js Express tutorial" description="A tutorial that builds on the Web App with Express tutorial by adding Azure Storage services and the Azure module." metaCanonical="" services="cloud-services,storage" documentationCenter="Node.js" title="Node.js Web Application using Storage" authors="larryfr" solutions="" manager="" editor="" />

# 使用存储构建 Node.js Web 应用程序

在本教程中，你将通过使用针对 Node.js 的 Windows Azure
客户端库来扩展在[使用 Express 构建 Node.js Web 应用程序][]
教程中创建的应用程序以操作数据管理服务。你将
扩展你的应用程序以创建可部署到 Azure 的基于 Web 的任务列表
应用程序。用户可以通过任务列表来检索任务、
添加新任务以及将任务标记为已完成。

任务项存储在 Azure 存储空间中。Azure 存储空间
提供了具有容错能力且可用性非常好的非结构化数据存储。
Azure 存储空间包含一些可用来存储和访问数据的
数据结构，你可以通过 Azure SDK for Node.js 中
包含的 API 或通过 REST API 利用存储
服务。有关详细信息，请参阅[在 Azure 中存储和访问数据][]。

本教程假定你已完成 [Node.js Web 应用程序][]
和[使用 Express 构建 Node.js][使用 Express 构建 Node.js Web 应用程序] 教程。

你将了解到以下内容：

-   如何操作 Jade 模板引擎
-   如何操作 Azure 数据管理服务

以下是已完成应用程序的屏幕快照：

![Internet Explorer 中已完成的网页][]

## 在 Web.Config 中设置存储凭据

若要访问 Azure 存储空间，你需要传入存储凭据。
若要执行此操作，你将利用 web.config 应用程序设置。
这些设置将作为环境变量传递给 Node，然后再由 Azure SDK 进行
读取。

**说明**

仅在将应用程序部署到 Azure 时才使用存储凭据。应用程序在模拟器中运行时将使用存储模拟器。

执行下列步骤检索存储帐户凭据并将这些凭据添加
到 web.config 设置中：

1.  如果尚未打开 Azure PowerShell，请通过在**“开始”**菜单中展开**“所有程序”、“Azure”**，右键单击**“Azure PowerShell”**，然后选择**“以管理员身份运行”**启动 Azure PowerShell。

2.  将目录更改到包含你的应用程序的文件夹。例如，C:\\node\\tasklist\\WebRole1。

3.  从 Azure Powershell 窗口中，输入以下 cmdlet 以检索存储帐户信息：

        PS C:\node\tasklist\WebRole1> Get-AzureStorageAccounts

    这样可以检索与托管服务关联的存储帐户和帐户密钥的列表。

    **说明**

    由于在你部署服务时 Azure SDK 会创建一个存储帐户，因此在前面的指南中部署你的应用程序之后应当已存在一个存储帐户。

4.  打开 web.cloud.config 文件，该文件包含将应用程序部署到 Azure 时所使用的环境设置：

        PS C:\node\tasklist\WebRole1> notepad web.cloud.config

5.  在 **configuration** 元素下插入以下块，使用你要用于部署的存储帐户的帐户名称和主密钥替换 {STORAGE ACCOUNT} 和 {STORAGE ACCESS KEY}：

        <appSettings>
        <add key="AZURE_STORAGE_ACCOUNT" value="{STORAGE ACCOUNT}"/>
        <add key="AZURE_STORAGE_ACCESS_KEY" value="{STORAGE ACCESS KEY}"/>
        </appSettings>

    ![web.cloud.config 文件内容][]

6.  保存该文件并关闭记事本。

## 安装模块

若要使用 Azure 数据管理服务，必须为 Node 安装 Azure 模块。
你还必须安装 node-uuid 模块，因为将使用此
模块来生成全局唯一标识符 (UUID)。
若要安装这些模块，请输入以下命令：

    PS C:\node\tasklist\WebRole1> npm install node-uuid azure

在命令完成后，这些模块已添加到 **node\_modules**
文件夹。执行下列步骤以在应用程序中使用这些
模块：

1.  打开 app.js 文件：

        PS C:\node\tasklist\WebRole1> notepad app.js

2.  在以 `var app - express();` 结尾的行后添加以下代码

        var uuid = require('node-uuid');
        var Home = require('./home');
        var azure = require('azure');

3.  添加用于创建存储表客户端的代码，以传入存储帐户和访问密钥信息。

    **说明**

    在模拟器中运行时，即使存储帐户信息是通过 web.config 提供的，SDK 也会自动使用模拟器。

        var client = azure.createTableService();

4.  接下来，在 Azure 存储空间中创建一个名为“tasks”的表。下面的逻辑将创建一个新表（如果该表不存在）并使用一些默认数据填充该表。

        //创建表
        client.createTableIfNotExists('tasks', function(error){
        if(error){
        throw error;
            }

        var item = {
        name:'Add readonly task list',
        category:'Site work',
        date: '12/01/2011',
        RowKey:uuid(),
        PartitionKey :'partition1',
        completed:false
            };

        client.insertEntity('tasks', item, function(){});

        });

5.  查找下面的两行：

        app.use('/', routes);
        app.use('/users', users);

    将上面的两行替换为下面的代码，这将创建一个主控制器实例并将对 **/** 或 **/home** 的所有请求路由至该实例。

        var home = new Home(client);
        app.get('/', home.showItems.bind(home));
        app.get('/home', home.showItems.bind(home));

    请注意，你现在要将命令委派给 Home 对象，而不是以内联方式处理请求。若要确保这些引用在主控制器内在本地正确解析，将需要 **bind** 命令。

## 创建主控制器

你现在必须创建一个主控制器，它用于处理针对任务列表站点的所有请求。
执行下列步骤以创建控制器：

1.  在记事本中新建一个 home.js 文件。该文件将包含用于
    为任务列表处理逻辑的控制器代码。

        PS C:\node\tasklist\WebRole1> notepad home.js

2.  将内容替换为以下代码并保存该文件。下面的
    代码使用了 javascript 模块模式。它将导出一个
    Home 函数。Home 原型包含用于处理实际请求的函数。

        var azure=require('azure');
        module.exports = Home;

        function Home (client) {
        this.client = client;
        };

        Home.prototype = {
        showItems:function (req, res) {
        var self = this;
        this.getItems(false, function (resp, tasklist) {
        if (!tasklist) {
        tasklist = [];
                    }           
        self.showResults(res, tasklist);
                });
            },

        getItems:function (allItems, callback) {
        var query = azure.TableQuery
        .select()
        .from('tasks');

        if (!allItems) {
        query = query.where('completed eq ?', 'false');
                }
        this.client.queryEntities(query, callback);
             },

        showResults:function (res, tasklist) {
        res.render('home', { 
        title:'Todo list', 
        layout:false, 
        tasklist:tasklist });
             },
        };

    你的主控制器现在包含以下三个函数：

    -   *showItems* 处理请求。
    -   *getItems* 使用表客户端从 tasks 表中检索打开的任务项。
        请注意，查询可以应用其他筛选器；例如，
        上面的查询筛选器只
        显示 completed 等于 false 的任务。
    -   *showResults* 调用 Express 呈现函数来呈现
        使用主视图（将在下一节中创建）
        的页面。

### 修改主视图

Jade 模板引擎使用的标记语法不及 HTML 的详细，它是用于
操作 Express 的默认引擎。执行下列
步骤创建一个支持显示任务列表项
的视图：

1.  从 Windows PowerShell 命令窗口中，使用以下命令
    编辑 home.jade 文件：

        PS C:\node\tasklist\WebRole1\views> notepad home.jade

2.  将 home.jade 文件的内容替换为以下代码并保存
    该文件。下面的表单包含用于读取和更新
    任务项的功能。（请注意，目前，主控制器仅支持读取；
    你将在后面对此进行更改。）该表单包含
    任务列表中每一项的详细信息。

        html
        head
        title Index
        body
        h1 My ToDo List

        form
        table(border="1")
        tr
        td Name
        td Category
        td Date
        td Complete

        each item in tasklist
        tr
        td #{item.name}
        td #{item.category} 
        td #{item.date} 
        td 
        input(type="checkbox", name="completed", value="#{item.RowKey}") 

## 在计算模拟器中运行应用程序

1.  在 Windows PowerShell 窗口中，输入以下 cmdlet 以在
    计算模拟器中启动你的服务并显示调用你的服务的
    一个网页。

        PS C:\node\tasklist\WebRole1> Start-AzureEmulator -launch

    你的浏览器将显示以下页面，其中显示了从 Azure 存储空间中检索到的任务项：

    ![显示了“My Tasklist”页的 Internet Explorer，该页的表中有一项内容。][]

## 添加新任务功能

在本节中，你将更新应用程序以支持添加新
任务项。

### 向 app.js 添加新路由

在 app.js 文件中，查找以下行：

    app.get('/home', home.showItems.bind(home));

在此行下，添加以下内容：

    app.post('/home/newitem', home.newItem.bind(home));

### 添加 Node-UUID 模块

若要使用 node-uuid 模块创建唯一标识符，请在 home.js 文件
顶部在导入该模块的第一行后面
添加以下行。

![突出显示 module.exports = Home 行的 home.js 文件。][]

       var uuid = require('node-uuid');

### 向主控制器添加新建项功能

若要实现新建项功能，请创建 **newItem** 函数。
在 home.js 文件中，将以下代码粘贴到最后一个函数后，
然后保存该文件。

![突出显示了 showresults 函数][]

       newItem:function (req, res) {
    var self = this;
    var createItem = function (resp, tasklist) {
    if (!tasklist) {
    tasklist = [];
               }

    var count = tasklist.length;

    var item = req.body.item;
    item.RowKey = uuid();
    item.PartitionKey = 'partition1';
    item.completed = false;

    self.client.insertEntity('tasks', item, function (error) {
    if(error){  
    throw error;
                   }
    self.showItems(req, res);
               });
           };

    this.getItems(true, createItem);
       },

**newItem** 函数将执行以下任务：

-   从正文中提取已发布的项。
-   为新项设置 **RowKey** 和 **PartitionKey** 值。
    这些值是向 Azure 表中插入新项所必需的。
    将为 **RowKey** 值生成 UUID。
-   通过调用 **insertEntity** 函数
    将该项插入 tasks 表中。
-   通过调用 **getItems** 函数来呈现页面。

### 向主视图添加新建项表单

现在，对视图进行更新，通过添加一个新表单来允许用户添加项。
将以下代码粘贴在 home.jade 文件的末尾，然后保存该文件。

**说明**

在 Jade 中，空格是有意义的，因此不要删除下面的任何一个空格。

        hr
    form(action="/home/newitem", method="post")
    table(border="1")    
    tr
    td Item Name: 
    td 
    input(name="item[name]", type="textbox")
    tr
    td Item Category: 
    td 
    input(name="item[category]", type="textbox")
    tr
    td Item Date: 
    td 
    input(name="item[date]", type="textbox")
    input(type="submit", value="Add item")

### 在模拟器中运行应用程序

1.  因为 Azure 模拟器已在运行，因此你可以浏览
    更新后的应用程序：

        PS C:\node\tasklist\WebRole1> start http://localhost:81/home

    浏览器将打开并显示以下页面：

    ![标题为 My Task List 的网页，其中所含的表包含任务和用于添加新任务的字段。][]

2.  为**“项目名称”**输入：“New task functionality”、为“项目类别” 输入：“Site work”，为“项目日期”**Item Date**输入："12/02/2011". 然后单击**“添加项目”**。

    该项将添加到 Azure 存储空间中的任务表，并显示为以下屏幕快照中所示的内容。

    ![将任务添加到该列表后，标题为 My Task List 的网页，其中所含表包含相关任务。][]

## 将应用程序重新发布到 Azure

现在应用程序已完成，你将通过更新现有托管服务的部署来
将应用程序发布到 Azure。

1.  在 Windows PowerShell 窗口中，调用以下 cmdlet 将
    托管服务重新部署到 Azure。你的存储设置和位置
    在前面已经保存，因此无需重新输入。

        PS C:\node\tasklist\WebRole1> Publish-AzureServiceProject -name myuniquename -location datacentername -launch

    部署完成后，你将看到如下响应：

    ![部署期间显示的状态消息。][]

    与先前一样，由于你指定了 **-launch** 选项，因此在发布完成后，浏览器将打开并显示正在 Azure 中运行的应用程序。

    ![浏览器窗口中显示 My Task List 页面。URL 表明该页面现在托管在 Azure 上。][]

## 停止并删除应用程序

在部署应用程序后，你可能希望禁用它，以避免在免费试用期内
产生费用或生成和部署其他应用程序。

Azure 按使用服务器的小时数对 Web 角色实例进行收费。
你的应用程序一旦部署，就开始使用服务器时间，即使
实例未运行并处于停止状态也是如此。

以下步骤演示了如何停止和删除应用程序。

1.  在 Windows PowerShell 窗口中，使用以下 cmdlet 停止
    在上一节中创建的服务部署：

        PS C:\node\tasklist\WebRole1> Stop-AzureService

    停止服务可能需要花费几分钟时间。在服务停止时，你会收到一条指示服务已停止的消息。

    ![指示服务已停止的状态消息。][]

2.  若要删除服务，请调用以下 cmdlet：

        PS C:\node\tasklist\WebRole1> Remove-AzureService contosotasklist

    在出现提示时，输入 **Y** 以删除服务。

    删除服务可能需要花费几分钟时间。在服务被删除后，你将收到一条指示服务已被删除的消息。

    ![指示服务已被删除的状态消息。][]

  [使用 Express 构建 Node.js Web 应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-nodejs-develop-deploy-express-app/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Node.js Web 应用程序]: http://azure.microsoft.com/zh-cn/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [Internet Explorer 中已完成的网页]: ./media/storage-nodejs-use-table-storage-cloud-service-app/getting-started-1.png
  [web.cloud.config 文件内容]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node37.png
  [显示了“My Tasklist”页的 Internet Explorer，该页的表中有一项内容。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node40.png
  [突出显示 module.exports = Home 行的 home.js 文件。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node42.png
  [突出显示了 showresults 函数]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node43.png
  [标题为 My Task List 的网页，其中所含的表包含任务和用于添加新任务的字段。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node44.png
  [将任务添加到该列表后，标题为 My Task List 的网页，其中所含表包含相关任务。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node45.png
  [部署期间显示的状态消息。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node35.png
  [浏览器窗口中显示 My Task List 页面。URL 表明该页面现在托管在 Azure 上。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node47.png
  [指示服务已停止的状态消息。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node48.png
  [指示服务已被删除的状态消息。]: ./media/storage-nodejs-use-table-storage-cloud-service-app/node49.png
