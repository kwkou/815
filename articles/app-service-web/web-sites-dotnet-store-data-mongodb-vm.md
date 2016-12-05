<properties
    pageTitle="在 Azure 中创建连接到虚拟机上运行的 MongoDB 的 Web 应用"
    description="本教程介绍如何使用 Git 将 ASP.NET 应用部署到连接至Azure 虚拟机上 MongoDB 的 Azure App Service。"
    tags="azure-portal"
    services="app-service\web, virtual-machines"
    documentationcenter=".net"
    author="cephalin"
    manager="wpickett"
    editor="" />  

<tags
    ms.assetid="adf7a472-ae00-45a8-aec4-06247e21318b"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="02/29/2016"
    wacn.date="12/05/2016"
    ms.author="cephalin" />

# 在 Azure 中创建连接到虚拟机上运行的 MongoDB 的 Web 应用
使用 Git，可以将 ASP.NET 应用程序部署到 Azure App Service Web 应用。在本教程中，你将构建一个简单的前端 ASP.NET MVC 任务列表应用程序，该程序将连接至在 Azure 内的虚拟机中运行的 MongoDB 数据库。[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。在开发计算机上运行并测试 ASP.NET 应用程序后，可使用 Git 将其上传到应用服务 Web 应用。


## 背景知识
以下知识对于学习本教程有用但非必需：

* MongoDB 的 C# 驱动程序。有关针对 MongoDB 开发 C# 应用程序的更多信息，请参阅 [MongoDB CSharp 语言中心][MongoC#LangCenter]。
* ASP .NET Web 应用程序框架。可通过 [ASP.net 网站][ASP.NET]进行全面了解。
* ASP .NET MVC Web 应用程序框架。可通过 [ASP.NET MVC 网站][MVCWebSite]进行全面了解。
* Azure。你可以先阅读 [Azure][WindowsAzure] 上的文章。

## 先决条件
* [Visual Studio Express 2013 for Web][VSEWeb] 或 [Visual Studio 2013][VSUlt]
* [用于 .NET 的 Azure SDK](http://go.microsoft.com/fwlink/p/?linkid=323510&clcid=0x409)
* 一个有效的 Azure 订阅

[AZURE.INCLUDE [create-account-and-websites-note](../../includes/create-account-and-websites-note.md)]

<a id="virtualmachine">

## 创建虚拟机和安装 MongoDB
本教程假定你已在 Azure 中创建了一个虚拟机。创建虚拟机后，你需要在该虚拟机上安装 MongoDB：

* 若要创建 Windows 虚拟机并安装 MongoDB，请参阅 [Install MongoDB on a virtual machine running Windows Server in Azure（在 Azure 中运行 Windows Server 的虚拟机上安装 MongoDB）][InstallMongoOnWindowsVM]。

在 Azure 中创建虚拟机并安装 MongoDB 后，请务必记住该虚拟机的 DNS 名称（例如“testlinuxvm.chinacloudapp.cn”）以及你在终结点中指定的 MongoDB 的外部端口。本教程后面的步骤中将会用到此信息。

## <a id="createapp"></a> 创建应用程序
在本部分中，将使用 Visual Studio 创建一个名为“My Task List”的 ASP.NET 应用程序，并执行到 Azure App Service Web 应用的初始部署。你将在本地运行该应用程序，但其将连接到你在 Azure 上的虚拟机并使用你在此处创建的 MongoDB 实例。

1. 在 Visual Studio 中，单击“新建项目”。
   
    ![新项目开始页面][StartPageNewProject]
2. 在“新建项目”窗口中的左侧窗格中，选择“Visual C#”，然后选择“Web”。在中间窗格中，选择“ASP.NET Web 应用程序”。在底部，将项目命名为“MyTaskListApp”，然后单击“确定”。
   
    ![新建项目对话框][NewProjectMyTaskListApp]
3. 在“新建 ASP.NET 项目”对话框中，选择“MVC”，然后单击“确定”。
   
    ![选择 MVC 模板][VS2013SelectMVCTemplate]  

4. 如果尚未登录 Azure，系统会提示用户登录。按提示登录到 Azure。
5. 登录后即可开始配置应用服务 Web 应用。指定“Web 应用名称”、“应用服务计划”、“资源组”和“区域”，然后单击“创建”。
   
    ![](./media/web-sites-dotnet-store-data-mongodb-vm/VSConfigureWebAppSettings.png)  

6. 项目创建完以后，等待 Web 应用在 Azure 应用服务中创建，如“Azure 应用服务活动”窗口所示。然后，单击“现在将 MyTaskListApp 发布到此 Web 应用”。
7. 单击“发布”。
   
    ![](./media/web-sites-dotnet-store-data-mongodb-vm/VSPublishWeb.png)
   
    将默认的 ASP.NET 应用程序发布到 Azure App Service Web 应用后，将在浏览器中启动该应用程序。

## 安装 MongoDB C# 驱动程序
MongoDB 通过驱动程序为 C# 应用程序提供客户端支持，你需要在本地开发计算机上安装此驱动程序。C# 驱动程序通过 NuGet 提供。

安装 MongoDB C# 驱动程序的步骤：

1. 在“解决方案资源管理器”中右键单击“MyTaskListApp”项目，然后选择“管理 NuGet 包”。
   
    ![管理 NuGet 包][VS2013ManageNuGetPackages]
2. 在“管理 NuGet 包”窗口的左侧窗格中，单击“联机”。在右侧的“联机搜索”框中，键入“mongodb.driver”。单击“安装”安装此驱动程序。
   
    ![搜索 MongoDB C# 驱动程序][SearchforMongoDBCSharpDriver]
3. 单击“我接受”接受 10gen, Inc. 的许可条款。
4. 安装驱动程序后，单击“关闭”。
    ![已安装 MongoDB C# 驱动程序][MongoDBCsharpDriverInstalled]

MongoDB C# 驱动程序现已安装。对 **MongoDB.Bson**、**MongoDB.Driver** 和 **MongoDB.Driver.Core** 库的引用已添加到项目。

![MongoDB C# 驱动程序引用][MongoDBCSharpDriverReferences]

## 添加模型
在“解决方案资源管理器”中，右键单击“Models”文件夹，选择**添加**一个新**类**，并将其命名为 *TaskModel.cs*。在 *TaskModel.cs* 中，将现有代码替换为以下代码：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using MongoDB.Bson.Serialization.Attributes;
    using MongoDB.Bson.Serialization.IdGenerators;
    using MongoDB.Bson;

    namespace MyTaskListApp.Models
    {
        public class MyTask
        {
            [BsonId(IdGenerator = typeof(CombGuidGenerator))]
            public Guid Id { get; set; }

            [BsonElement("Name")]
            public string Name { get; set; }

            [BsonElement("Category")]
            public string Category { get; set; }

            [BsonElement("Date")]
            public DateTime Date { get; set; }

            [BsonElement("CreatedDate")]
            public DateTime CreatedDate { get; set; }

        }
    }

## 添加数据访问层
在“解决方案资源管理器”中，右键单击“MyTaskListApp”项目并选择**添加**一个名为 *DAL* 的**新文件夹**。右键单击“DAL”文件夹并选择**添加**一个新**类**。将该类文件命名为 *Dal.cs*。在 *Dal.cs* 中，将现有代码替换为以下代码：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using MyTaskListApp.Models;
    using MongoDB.Driver;
    using MongoDB.Bson;
    using System.Configuration;


    namespace MyTaskListApp
    {
        public class Dal : IDisposable
        {
            private MongoServer mongoServer = null;
            private bool disposed = false;

            // To do: update the connection string with the DNS name
            // or IP address of your server. 
            //For example, "mongodb://testlinux.chinacloudapp.cn"
            private string connectionString = "mongodb://mongodbsrv20151211.chinacloudapp.cn";

            // This sample uses a database named "Tasks" and a 
            //collection named "TasksList".  The database and collection 
            //will be automatically created if they don't already exist.
            private string dbName = "Tasks";
            private string collectionName = "TasksList";

            // Default constructor.        
            public Dal()
            {
            }

            // Gets all Task items from the MongoDB server.        
            public List<MyTask> GetAllTasks()
            {
                try
                {
                    var collection = GetTasksCollection();
                    return collection.Find(new BsonDocument()).ToList();
                }
                catch (MongoConnectionException)
                {
                    return new List<MyTask>();
                }
            }

            // Creates a Task and inserts it into the collection in MongoDB.
            public void CreateTask(MyTask task)
            {
                var collection = GetTasksCollectionForEdit();
                try
                {
                    collection.InsertOne(task);
                }
                catch (MongoCommandException ex)
                {
                    string msg = ex.Message;
                }
            }

            private IMongoCollection<MyTask> GetTasksCollection()
            {
                MongoClient client = new MongoClient(connectionString);
                var database = client.GetDatabase(dbName);
                var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
                return todoTaskCollection;
            }

            private IMongoCollection<MyTask> GetTasksCollectionForEdit()
            {
                MongoClient client = new MongoClient(connectionString);
                var database = client.GetDatabase(dbName);
                var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
                return todoTaskCollection;
            }

            # region IDisposable

            public void Dispose()
            {
                this.Dispose(true);
                GC.SuppressFinalize(this);
            }

            protected virtual void Dispose(bool disposing)
            {
                if (!this.disposed)
                {
                    if (disposing)
                    {
                        if (mongoServer != null)
                        {
                            this.mongoServer.Disconnect();
                        }
                    }
                }

                this.disposed = true;
            }

            # endregion
        }
    }

## 添加控制器
在“解决方案资源管理器”中打开 *Controllers\\HomeController.cs* 文件，将现有代码替换为以下代码：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using System.Web.Mvc;
    using MyTaskListApp.Models;
    using System.Configuration;

    namespace MyTaskListApp.Controllers
    {
        public class HomeController : Controller, IDisposable
        {
            private Dal dal = new Dal();
            private bool disposed = false;
            //
            // GET: /MyTask/

            public ActionResult Index()
            {
                return View(dal.GetAllTasks());
            }

            //
            // GET: /MyTask/Create

            public ActionResult Create()
            {
                return View();
            }

            //
            // POST: /MyTask/Create

            [HttpPost]
            public ActionResult Create(MyTask task)
            {
                try
                {
                    dal.CreateTask(task);
                    return RedirectToAction("Index");
                }
                catch
                {
                    return View();
                }
            }

            public ActionResult About()
            {
                return View();
            }

            # region IDisposable

            new protected void Dispose()
            {
                this.Dispose(true);
                GC.SuppressFinalize(this);
            }

            new protected virtual void Dispose(bool disposing)
            {
                if (!this.disposed)
                {
                    if (disposing)
                    {
                        this.dal.Dispose();
                    }
                }

                this.disposed = true;
            }

            # endregion

        }
    }

## 设置样式
若要更改页面顶部的标题，请在“解决方案资源管理器”中打开 *Views\\Shared\\_Layout.cshtml* 文件，将导航条标头中的“Application name”替换为“My Task List Application”，以便其类似如下所示：

     @Html.ActionLink("My Task List Application", "Index", "Home", null, new { @class = "navbar-brand" })

为了设置 Task List 菜单，请打开 *\\Views\\Home\\Index.cshtml* 文件并将现有代码替换为以下代码：

    @model IEnumerable<MyTaskListApp.Models.MyTask>

    @{
        ViewBag.Title = "My Task List";
    }

    <h2>My Task List</h2>

    <table border="1">
        <tr>
            <th>Task</th>
            <th>Category</th>
            <th>Date</th>

        </tr>

    @foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Name)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Category)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Date)
            </td>

        </tr>
    }

    </table>
    <div>  @Html.Partial("Create", new MyTaskListApp.Models.MyTask())</div>


若要增加创建新任务的功能，右键单击“Views\\Home\\”文件夹并选择*添加*一个**视图**。将该视图命名为 *Create*。将此代码替换为以下代码：

    @model MyTaskListApp.Models.MyTask

    <script src="@Url.Content("~/Scripts/jquery-1.10.2.min.js")" type="text/javascript"></script>
    <script src="@Url.Content("~/Scripts/jquery.validate.min.js")" type="text/javascript"></script>
    <script src="@Url.Content("~/Scripts/jquery.validate.unobtrusive.min.js")" type="text/javascript"></script>

    @using (Html.BeginForm("Create", "Home")) {
        @Html.ValidationSummary(true)
        <fieldset>
            <legend>New Task</legend>

            <div class="editor-label">
                @Html.LabelFor(model => model.Name)
            </div>
            <div class="editor-field">
                @Html.EditorFor(model => model.Name)
                @Html.ValidationMessageFor(model => model.Name)
            </div>

            <div class="editor-label">
                @Html.LabelFor(model => model.Category)
            </div>
            <div class="editor-field">
                @Html.EditorFor(model => model.Category)
                @Html.ValidationMessageFor(model => model.Category)
            </div>

            <div class="editor-label">
                @Html.LabelFor(model => model.Date)
            </div>
            <div class="editor-field">
                @Html.EditorFor(model => model.Date)
                @Html.ValidationMessageFor(model => model.Date)
            </div>

            <p>
                <input type="submit" value="Create" />
            </p>
        </fieldset>
    }

“解决方案资源管理器”应类似如下所示：

![解决方案资源管理器][SolutionExplorerMyTaskListApp]

## 设置 MongoDB 连接字符串
在“解决方案资源管理器”中，打开 *DAL/Dal.cs* 文件。找到以下代码行：

    private string connectionString = "mongodb://<vm-dns-name>";

将 `<vm-dns-name>` 替换为运行 MongoDB 的虚拟机（在本教程的[创建虚拟机并安装 MongoDB][Create a virtual machine and install MongoDB] 步骤中创建）的 DNS 名。若要查找虚拟机的 DNS 名称，请转到 Azure 门户，选择“虚拟机”并找到“DNS 名称”。

如果虚拟机的 DNS 名是“testlinuxvm.chinacloudapp.cn”而 MongoDB 在默认端口 27017 进行侦听，连接字符串代码行将如下所示：

    private string connectionString = "mongodb://testlinuxvm.chinacloudapp.cn";

如果虚拟机终结点为 MongoDB 指定了不同的外部端口，你可在连接字符串中指定端口：

     private string connectionString = "mongodb://testlinuxvm.chinacloudapp.cn:12345";

有关 MongoDB 连接字符串的详细信息，请参阅[连接][MongoConnectionStrings]。

## 测试本地部署
若要在开发计算机上运行应用程序，请从“调试”菜单选择“启动调试”或按 **F5**。IIS Express 启动，浏览器打开并显示该引用程序的主页。你可以添加一个新任务，而后将其添加到在 Azure 中的虚拟机上运行的 MongoDB 数据库。

![My Task List 应用程序][TaskListAppBlank]

## 发布到 Azure App Service Web 应用
在本部分中，将向 Azure App Service Web 应用发布所做的更改。

1. 在“解决方案资源管理器”中，再次右键单击“MyTaskListApp”，并单击“发布”。
2. 单击“发布”。
   
    现在应该看到 Web 应用在 Azure App Service 中运行并在 Azure 虚拟机中访问 MongoDB 数据库。

## 摘要
现在，已将 ASP.NET 应用程序成功部署到 Azure App Service Web 应用。查看 Web 应用：

1. 登录到 Azure 门户预览。
2. 单击“Web Apps”。
3. 在“Web Apps”列表中选择你的 Web 应用。

有关针对 MongoDB 开发 C# 应用程序的详细信息，请参阅 [CSharp 语言中心][MongoC#LangCenter]。

[AZURE.INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]

<!-- HYPERLINKS -->

[AzurePortal]: http://manage.windowsazure.cn
[WindowsAzure]: http://www.azure.cn
[MongoC#LangCenter]: http://docs.mongodb.org/ecosystem/drivers/csharp/
[MVCWebSite]: http://www.asp.net/mvc
[ASP.NET]: http://www.asp.net/
[MongoConnectionStrings]: http://www.mongodb.org/display/DOCS/Connections
[MongoDB]: http://www.mongodb.org
[InstallMongoOnWindowsVM]: /documentation/articles/virtual-machines-windows-classic-install-mongodb/
[VSEWeb]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs#d-2013-express
[VSUlt]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs

<!-- IMAGES -->


[StartPageNewProject]: ./media/web-sites-dotnet-store-data-mongodb-vm/NewProject.png
[NewProjectMyTaskListApp]: ./media/web-sites-dotnet-store-data-mongodb-vm/NewProjectMyTaskListApp.png
[VS2013SelectMVCTemplate]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013SelectMVCTemplate.png
[VS2013DefaultMVCApplication]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013DefaultMVCApplication.png
[VS2013ManageNuGetPackages]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013ManageNuGetPackages.png
[SearchforMongoDBCSharpDriver]: ./media/web-sites-dotnet-store-data-mongodb-vm/SearchforMongoDBCSharpDriver.png
[MongoDBCsharpDriverInstalled]: ./media/web-sites-dotnet-store-data-mongodb-vm/MongoDBCsharpDriverInstalled.png
[MongoDBCSharpDriverReferences]: ./media/web-sites-dotnet-store-data-mongodb-vm/MongoDBCSharpDriverReferences.png
[SolutionExplorerMyTaskListApp]: ./media/web-sites-dotnet-store-data-mongodb-vm/SolutionExplorerMyTaskListApp.png
[TaskListAppBlank]: ./media/web-sites-dotnet-store-data-mongodb-vm/TaskListAppBlank.png
[WAWSCreateWebSite]: ./media/web-sites-dotnet-store-data-mongodb-vm/WAWSCreateWebSite.png
[WAWSDashboardMyTaskListApp]: ./media/web-sites-dotnet-store-data-mongodb-vm/WAWSDashboardMyTaskListApp.png
[Image9]: ./media/web-sites-dotnet-store-data-mongodb-vm/RepoReady.png
[Image10]: ./media/web-sites-dotnet-store-data-mongodb-vm/GitInstructions.png
[Image11]: ./media/web-sites-dotnet-store-data-mongodb-vm/GitDeploymentComplete.png

<!-- TOC BOOKMARKS -->
[Create a virtual machine and install MongoDB]: #virtualmachine
[Create and run the My Task List ASP.NET application on your development computer]: #createapp
[Create an Azure web site]: #createwebsite
[Deploy the ASP.NET application to the web site using Git]: #deployapp

<!---HONumber=Mooncake_1128_2016-->