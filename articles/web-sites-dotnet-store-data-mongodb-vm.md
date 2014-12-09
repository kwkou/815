<properties linkid="develop-dotnet-website-with-mongodb-vm" urlDisplayName="Website with MongoDB VM" pageTitle="在虚拟机上使用 MongoDB 的 .NET 网站 - Azure" metaKeywords="Azure Git ASP.NET MongoDB, Git .NET, Git MongoDB, ASP.NET MongoDB, Azure MongoDB, Azure ASP.NET, Azure tutorial" description="本教程将向您介绍如何使用 Git 将 ASP.NET 应用程序部署到连接至虚拟机上 MongoDB 的 Azure 网站。" metaCanonical="" services="web-sites,virtual-machines" documentationCenter=".NET" title="创建连接到在 Azure 内的虚拟机上运行的 MongoDB 的 Azure 网站" authors="" solutions="" manager="" editor="" />

# 创建连接到在 Azure 内的虚拟机上运行的 MongoDB 的 Azure 网站

使用 Git，您可以将 ASP.NET 应用程序部署到 Azure 网站。在本教程中，您将构建一个简单的前端 ASP.NET MVC 任务列表应用程序，该程序将连接至在 Azure 内的虚拟机中运行的 MongoDB 数据库。[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。在开发计算机上运行并测试了该 ASP.NET 应用程序后，可使用 Git 将其上载至 Azure 网站。

[WACOM.INCLUDE [create-account-and-websites-and-vms-note](../includes/create-account-and-websites-and-vms-note.md)]

## 概述

在本教程中你将：

-   [创建虚拟机和安装 MongoDB][创建虚拟机和安装 MongoDB]
-   [在开发计算机上开发并运行 My Task List ASP.NET 应用程序][在开发计算机上开发并运行 My Task List ASP.NET 应用程序]
-   [创建 Azure 网站][创建 Azure 网站]
-   [使用 Git 将 ASP.NET 应用程序部署到网站][使用 Git 将 ASP.NET 应用程序部署到网站]

## 背景知识

以下知识对于学习本教程有用但非必需：

-   MongoDB 的 C# 驱动程序。有关针对 MongoDB 开发 C# 应用程序的更多信息，请参见 MongoDB [CSharp 语言中心][CSharp 语言中心]。
-   ASP .NET Web 应用程序框架。可通过 [ASP.net 网站][ASP.net 网站]进行全面了解。
-   ASP .NET MVC Web 应用程序框架。可通过 [ASP.NET MVC 网站][ASP.NET MVC 网站]进行全面了解。
-   Azure。可从 [Azure][Azure] 开始了解。

## 准备工作

在本部分中，您将学习如何在 Azure 中创建虚拟机、安装 MongoDB 以及设置开发环境。

<span id="virtualmachine"></span></a>

### 创建虚拟机和安装 MongoDB

本教程假定您已在 Azure 中创建了一个虚拟机。创建虚拟机后，您需要在该虚拟机上安装 MongoDB：

-   若要创建 Windows 虚拟机并安装 MongoDB，请参见[在 Azure 中，在运行 Windows Server 的虚拟机上安装 MongoDB][在 Azure 中，在运行 Windows Server 的虚拟机上安装 MongoDB]。
-   或者，若要创建 Linux 虚拟机并安装 MongoDB，请参见[在 Azure 中，在运行 CentOS 的虚拟机上安装 MongoDB][在 Azure 中，在运行 CentOS 的虚拟机上安装 MongoDB]。

在 Azure 中创建虚拟机并安装 MongoDB 后，请务必记住该虚拟机的 DNS 名称（例如“testlinuxvm.chinacloudapp.cn”）以及您在终结点中指定的 MongoDB 的外部端口。本教程后面的步骤中将会用到此信息。

### 安装 Visual Studio

首先，安装并运行 [Visual Studio Express 2013 for Web][Visual Studio Express 2013 for Web] 或 [Visual Studio 2013][Visual Studio Express 2013 for Web]。

Visual Studio 是一个 IDE，或集成的开发环境。就像使用 Microsoft Word 编写文档，您将使用 IDE 来创建应用程序。本教程使用的是 Microsoft Visual Studio 2013，但您可以使用 Microsoft Visual Studio Express 2013，这是 Microsoft Visual Studio 的免费版。

<span id="createapp"></span></a>

## 在开发计算机上开发并运行 My Task List ASP.NET 应用程序

在本部分中，您将使用 Visual Studio 创建一个名为“My Task List”的 ASP.NET 应用程序。您将在本地运行该应用程序，但其将连接到您在 Azure 上的虚拟机并使用您在此处创建的 MongoDB 实例。

### 创建应用程序

在 Visual Studio 中，单击“新建项目”。

![新项目开始页面][新项目开始页面]

在“新建项目”窗口中的左侧窗格中，选择“Visual C#”，然后选择“Web”。在中间窗格中，选择“ASP.NET Web 应用程序”。在底部，将项目命名为“MyTaskListApp”，然后单击“确定”。

![新建项目对话框][新建项目对话框]

在“新建 ASP.NET 项目”对话框中，选择“MVC”，然后单击“确定”。

![选择 MVC 模板][选择 MVC 模板]

项目完成后，将显示由模板创建的默认页。

![默认的 ASP.NET MVC 应用程序][默认的 ASP.NET MVC 应用程序]

### 安装 MongoDB C# 驱动程序

MongoDB 通过驱动程序为 C# 应用程序提供客户端支持，您需要在本地开发计算机上安装此驱动程序。C# 驱动程序通过 NuGet 提供。

安装 MongoDB C# 驱动程序的步骤：

1.  在“解决方案资源管理器”中的 **MyTaskListApp** 项目下，右键单击“引用”并选择“管理 NuGet 包”。.

    ![管理 NuGet 包][管理 NuGet 包]

2.  在“管理 NuGet 包”窗口的左侧窗格中，单击“联机”。在右侧的“联机搜索”框中，键入“mongocsharpdriver”。单击“安装”安装此驱动程序。

    ![搜索 MongoDB C# 驱动程序][搜索 MongoDB C# 驱动程序]

3.  单击“我接受”接受 10gen, Inc. 的许可条款。

4.  驱动程序安装完成后，单击“关闭”。
    ![MongoDB C# 驱动程序已安装][MongoDB C# 驱动程序已安装]

MongoDB C# 驱动程序现已安装。对 **MongoDB.Driver.dll** 和 **MongoDB.Bson.dll** 库的引用已添加至项目。

![MongoDB C# 驱动程序引用][MongoDB C# 驱动程序引用]

### 添加模型

在“解决方案资源管理器”内，右键单击 *Models* 文件夹“添加”一个新“类”，并将其命名为 *TaskModel.cs*。在 *TaskModel.cs* 中，将现有代码替换为以下代码：

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

### 添加数据访问层

在“解决方案资源管理器”中，右键单击 *MyTaskListApp* 项目并“添加”一个名为 *DAL* 的“新文件夹”。右键单击 *DAL* 文件夹并“添加”一个新的“类”。将该类文件命名为 *Dal.cs*。在 *Dal.cs* 中，将现有代码替换为以下代码：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using MyTaskListApp.Models;
    using MongoDB.Driver;
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
            private string connectionString = "mongodb://<vm-dns-name>";

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
                    MongoCollection<MyTask> collection = GetTasksCollection();
                    return collection.FindAll().ToList<MyTask>();
                }
                catch (MongoConnectionException)
                {
                    return new List<MyTask >();
                }
            }

            // Creates a Task and inserts it into the collection in MongoDB.
            public void CreateTask(MyTask task)
            {
                MongoCollection<MyTask> collection = GetTasksCollectionForEdit();
                try
                {
                    collection.Insert(task, SafeMode.True);
                }
                catch (MongoCommandException ex)
                {
                    string msg = ex.Message;
                }
            }

            private MongoCollection<MyTask> GetTasksCollection()
            {
                MongoServer server = MongoServer.Create(connectionString);
                MongoDatabase database = server[dbName];
                MongoCollection<MyTask> todoTaskCollection = database.GetCollection<MyTask>(collectionName);
                return todoTaskCollection;
            }

            private MongoCollection<MyTask> GetTasksCollectionForEdit()
            {
                MongoServer server = MongoServer.Create(connectionString);
                MongoDatabase database = server[dbName];
                MongoCollection<MyTask> todoTaskCollection = database.GetCollection<MyTask>(collectionName);
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

### 添加控制器

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

### 设置站点样式

若要更改页面顶部的标题，请在“解决方案资源管理器”中打开 \*Views\\Shared\\\_Layout.cshtml\* 文件，将导航条标头中的“Application name”替换为“My Task List Application”以便其类似如下所示：

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

若要增加创建新任务的功能，右键单击 *Views\\Home\\* 文件夹并“添加”一个“视图”。将该视图命名为 *Create*。将此代码替换为以下代码：

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

![解决方案资源管理器][解决方案资源管理器]

### 设置 MongoDB 连接字符串

在“解决方案资源管理器”中，打开 *DAL/Dal.cs* 文件。找到以下代码行：

    private string connectionString = "mongodb://<vm-dns-name>";

将 `<vm-dns-name>` 替换为运行 MongoDB 的虚拟机（在本教程[创建虚拟机并安装 MongoDB][创建虚拟机和安装 MongoDB] 中创建）的 DNS 名。若要查找虚拟机的 DNS 名，请转至 Azure 管理门户，选择“虚拟机”并找到“DNS 名”。

如果虚拟机的 DNS 名是“testlinuxvm.cloudapp.net”而 MongoDB 在默认端口 27017 进行侦听，连接字符串代码行将如下所示：

    private string connectionString = "mongodb://testlinuxvm.chinacloudapp.cn";

如果虚拟机终结点为 MongoDB 指定了不同的外部端口，您可在连接字符串中指定端口：

    private string connectionString = "mongodb://testlinuxvm.chinacloudapp.cn:12345";

有关 MongoDB 连接字符串的更多信息，请参见[连接][连接]。

### 测试本地部署

若要在开发计算机上运行应用程序，请从“调试”菜单选择“启动调试”或按 **F5**。IIS Express 启动，浏览器打开并显示该引用程序的主页。您可以添加一个新任务，而后将其添加到在 Azure 中的虚拟机上运行的 MongoDB 数据库。

![My Task List 应用程序][My Task List 应用程序]

## <span class="short-header">将应用程序部署到 Azure 网站</span>将 ASP.NET 应用程序部署到 Azure 网站

在本部分中，您将创建一个网站并使用 Git 部署 My Task List ASP.NET 应用程序。

<span id="createwebsite"></span></a>

### 创建 Azure 网站

本部分中，您将创建一个 Azure 网站。

1.  打开 Web 浏览器并浏览至 [Azure 管理门户][Azure 管理门户]。使用您的 Azure 帐户进行登录。
2.  在页面底部，依次单击“+新建”、“网站”和“速览”。
3.  为应用程序的 URL 输入唯一的前缀。
4.  选择区域。
5.  单击“创建网站”。

![创建新网站][创建新网站]

1.  您的网站将快速创建成功，并显示在“网站”列表中。

![WAWSDashboardMyTaskListApp][WAWSDashboardMyTaskListApp]

<span id="deployapp"></span></a>

### 使用 Git 将 ASP.NET 应用程序部署到网站

在本部分中，你将使用 Git 部署 My Task List 应用程序。

1.  在“网站”中单击您网站名，然后单击“仪表板”。在右侧的“速览”下，单击“从源代码管理设置部署”。
2.  在“您的源代码位置”页面，选择“本地 Git 存储库”并单击“下一步”箭头。
3.  Git 存储库将快速创建。请注意生成页面上的说明，这些内容将在后面的部分用到。

    ![Git 存储库准备就绪][Git 存储库准备就绪]

4.  在“将我的本地文件推送至 Azure”下提供了将代码推送至 Azure 的说明。说明内容类似如下所示：

    ![将本地文件推送到 Azure][将本地文件推送到 Azure]

5.  如果没有安装 Git，可使用步骤 1 中的“从此处获取”链接进行安装。
6.  按照步骤 2 中的说明提交您的本地文件。
7.  按照步骤 3 中的说明添加远程 Azure 存储库并将您的文件推送至 Azure 网站。
8.  部署完成后，您将看到以下确认信息：

    ![部署完成][部署完成]

9.  您的 Azure 网站现已可用。查看您的站点的“仪表板”页面，找到“站点 URL”字段以确定站点的 URL。按照本教程的步骤执行，您的站点的 URL 应该是：http://mytasklistapp.chinacloudsites.cn。

## 摘要

现在，您已将 ASP.NET 应用程序成功部署到 Azure 网站。若要查看站点，请单击“仪表板”页面的“站点 URL”字段中的链接。有关针对 MongoDB 开发 C# 应用程序的更多信息，请参见 [CSharp 语言中心][CSharp 语言中心]。

<!-- HYPERLINKS --> <!-- IMAGES --> <!-- TOC BOOKMARKS -->

  [MongoDB]: http://www.mongodb.org
  [创建虚拟机和安装 MongoDB]: #virtualmachine
  [在开发计算机上开发并运行 My Task List ASP.NET 应用程序]: #createapp
  [创建 Azure 网站]: #createwebsite
  [使用 Git 将 ASP.NET 应用程序部署到网站]: #deployapp
  [CSharp 语言中心]: http://docs.mongodb.org/ecosystem/drivers/csharp/
  [ASP.net 网站]: http://www.asp.net/
  [ASP.NET MVC 网站]: http://www.asp.net/mvc
  [Azure]: http://www.windowsazure.cn
  [在 Azure 中，在运行 Windows Server 的虚拟机上安装 MongoDB]: /zh-cn/manage/windows/common-tasks/install-mongodb/
  [在 Azure 中，在运行 CentOS 的虚拟机上安装 MongoDB]: /zh-cn/manage/linux/common-tasks/mongodb-on-a-linux-vm/
  [Visual Studio Express 2013 for Web]: http://www.visualstudio.com/zh-cn/downloads/download-visual-studio-vs
  [新项目开始页面]: ./media/web-sites-dotnet-store-data-mongodb-vm/NewProject.png
  [新建项目对话框]: ./media/web-sites-dotnet-store-data-mongodb-vm/NewProjectMyTaskListApp.png
  [选择 MVC 模板]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013SelectMVCTemplate.png
  [默认的 ASP.NET MVC 应用程序]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013DefaultMVCApplication.png
  [管理 NuGet 包]: ./media/web-sites-dotnet-store-data-mongodb-vm/VS2013ManageNuGetPackages.png
  [搜索 MongoDB C# 驱动程序]: ./media/web-sites-dotnet-store-data-mongodb-vm/SearchforMongoDBCSharpDriver.png
  [MongoDB C# 驱动程序已安装]: ./media/web-sites-dotnet-store-data-mongodb-vm/MongoDBCsharpDriverInstalled.png
  [MongoDB C# 驱动程序引用]: ./media/web-sites-dotnet-store-data-mongodb-vm/MongoDBCSharpDriverReferences.png
  [解决方案资源管理器]: ./media/web-sites-dotnet-store-data-mongodb-vm/SolutionExplorerMyTaskListApp.png
  [连接]: http://www.mongodb.org/display/DOCS/Connections
  [My Task List 应用程序]: ./media/web-sites-dotnet-store-data-mongodb-vm/TaskListAppBlank.png
  [Azure 管理门户]: http://manage.windowsazure.cn
  [创建新网站]: ./media/web-sites-dotnet-store-data-mongodb-vm/WAWSCreateWebSite.png
  [WAWSDashboardMyTaskListApp]: ./media/web-sites-dotnet-store-data-mongodb-vm/WAWSDashboardMyTaskListApp.png
  [Git 存储库准备就绪]: ./media/web-sites-dotnet-store-data-mongodb-vm/RepoReady.png
  [将本地文件推送到 Azure]: ./media/web-sites-dotnet-store-data-mongodb-vm/GitInstructions.png
  [部署完成]: ./media/web-sites-dotnet-store-data-mongodb-vm/GitDeploymentComplete.png
