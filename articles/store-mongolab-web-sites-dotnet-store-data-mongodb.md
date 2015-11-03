<properties 
	pageTitle="使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 .NET Web 应用" 
	description="了解如何在 Azure 网站上创建 ASP.NET Web 应用，用于将数据存储在由 MongoLab 托管的 MongoDB 中。" 
	tags="azure-classic-portal"
	services="app-service\web" 
	documentationCenter=".net" 
	authors="chrisckchang" 
	manager="partners@mongolab.com" 
	editor="mollybos"/>

<tags 
	ms.service="app-service-web" 
	ms.date="08/03/2015" 
	wacn.date="10/03/2015"/>



# 使用 MongoLab 外接程序通过 MongoDB 在 Azure 上创建 .NET Web 应用

  	<!-- The MongoLab workflow is not yet supported in the Preview Portal -->

<p><em>作者：Eric Sedor，MongoLab</em></p>

各位冒险家，大家好！ 欢迎使用 MongoDB 即服务。在本教程中，您将：

1. [创建应用][create] - 它将是一个简单的 C# ASP.NET MVC 应用，用于进行记录。
1. [部署应用][deploy] - 通过将一些配置联系在一起，我们轻而易举就能将代码推送到 [Azure 网站](/documentation/services/web-sites/)。
1. [管理数据库][manage] - 最后，我们将向你演示 MongoLab 基于 Web 的数据库管理门户，在此你可轻松搜索、显示和修改数据。

在本教程的任意时间，如有任何问题，请随时发送电子邮件至 [support@mongolab.com](mailto:support@mongolab.com)。

<a name="provision"></a>
## 设置数据库

[AZURE.INCLUDE [howto-provision-mongolab](../includes/howto-provision-mongolab.md)]

<a name="create"></a>
## 创建应用程序

在本部分中，将介绍如何创建 C# ASP.NET Visual Studio 项目，并演练如何使用 C# MongoDB 驱动程序创建简单的便签应用。你希望能够访问自己的 Web 应用、书写便笺并查看留下的所有便笺。

你将在 Visual Studio Express 2013 for Web 中执行此开发。

### 创建项目
您的示例应用将充分使用 Visual Studio 模板以开始操作。确保使用 .NET Framework 4.5。

1. 选择“文件 > 新建项目”。显示“新建项目”对话框：![NewProject][dialog-mongolab-csharp-newproject]

1. 选择“已安装 > 模板 > Visual C# > Web”。

1. 从 .NET 版本下拉菜单中选择“.NET Framework 4.5”。

1. 选择“MVC 应用程序”。

1. 输入 _mongoNotes_ 作为你的**项目名称**。如果您选择了其他名称，您将需要修改在整个教程中提供的代码。

1. 选择“工具 > 库程序包管理器 > 程序包管理器控制台”。在 PM 控制台中，键入 **Install-Package mongocsharpdriver**，然后按“Enter”。![PMConsole][focus-mongolab-csharp-pmconsole] MongoDB C# 驱动程序与项目集成，并且以下行将会自动添加到 _packages.config_ 文件中：

        < package id="mongocsharpdriver" version="1.9.2" targetFramework="net45" / >

### 添加便笺模型
首先，建立一个便笺模型，该模型只具有日期和文本内容。

1. 在解决方案资源管理器中，右键单击“模型”，然后选择“添加 > 类”。将此新类命名为 *Note.cs*。

1. 将此类的自动生成的代码替换为以下内容：

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using MongoDB.Bson.Serialization.Attributes;
        using MongoDB.Bson.Serialization.IdGenerators;
        using MongoDB.Bson;
                
        namespace mongoNotes.Models
        {
            public class Note
            {
                public Note()
                {
                    Date = DateTime.UtcNow;
                }
                
                private DateTime date;
        
                [BsonId(IdGenerator = typeof(CombGuidGenerator))]
                public Guid Id { get; set; }
        
                [BsonElement("Note")]
                public string Text { get; set; }
        
                [BsonElement("Date")]
                public DateTime Date {
                    get { return date.ToLocalTime(); }
                    set { date = value;}
                }
            }
        }

### 添加数据访问层
建立访问 MongoDB 以检索和保存便笺的方式非常重要。您的数据访问层将充分使用便笺模型，并且稍后会绑定到您的 HomeController 中。

1. 在解决方案资源管理器中，右键单击“mongoNotes”项目，然后选择“添加 > 新建文件夹”。将该文件夹命名为 **DAL**。

1. 在解决方案资源管理器中，右键单击“DAL”，然后选择“添加 > 类”。将此新类命名为 *Dal.cs*。

1. 将此类的自动生成的代码替换为以下内容：

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using mongoNotes.Models;
        using MongoDB.Driver;
        using System.Configuration;

        namespace mongoNotes
        {
            public class Dal : IDisposable
            {
                private MongoServer mongoServer = null;
                private bool disposed = false;
        
                private string connectionString = System.Environment.GetEnvironmentVariable("CUSTOMCONNSTR_MONGOLAB_URI");
                MongoUrl url;
        
                private string dbName = "myMongoApp";
                private string collectionName = "Notes";
        
                // Default constructor.        
                public Dal()
                {
                    url = new MongoUrl(connectionString);
                }
           
                public List<Note> GetAllNotes()
                {
                    try
                    {
                        MongoCollection<Note> collection = GetNotesCollection();
                        return collection.FindAll().ToList<Note>();
                    }
                    catch (MongoConnectionException)
                    {
                        return new List<Note>();
                    }
                }
        
                // Creates a Note and inserts it into the collection in MongoDB.
                public void CreateNote(Note note)
                {
                    MongoCollection<Note> collection = getNotesCollectionForEdit();
                    try
                    {
                        collection.Insert(note);
                    }
                    catch (MongoCommandException ex)
                    {
                        string msg = ex.Message;
                    }
                }
        
                private MongoCollection<Note> GetNotesCollection()
                {
                    MongoClient client = new MongoClient(url);
                    mongoServer = client.GetServer();
                    MongoDatabase database = mongoServer.GetDatabase(dbName);
                    MongoCollection<Note> noteCollection = database.GetCollection<Note>(collectionName);
                    return noteCollection;
                }
        
                private MongoCollection<Note> getNotesCollectionForEdit()
                {
                    MongoClient client = new MongoClient(url);
                    mongoServer = client.GetServer();
                    MongoDatabase database = mongoServer.GetDatabase(dbName);
                    MongoCollection<Note> notesCollection = database.GetCollection<Note>(collectionName);
                    return notesCollection;
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

1. 注意上面的以下代码：
            
        private string connectionString = System.Environment.GetEnvironmentVariable("CUSTOMCONNSTR_MONGOLAB_URI");
        private string dbName = "myMongoApp";  
在此，您会访问一个您稍后将配置的环境变量。如果您出于开发用途正在运行一个本地 mongo 实例，则可能要暂时将此值设置为“localhost”。
  
  设置您的数据库名称。具体而言，将 **dbName** 值设置为你在设置 MongoLab 外接程序时所输入的名称。

1. 最后，检查 **GetNotesCollection()** 中的以下代码：  

        MongoClient client = new MongoClient(url);
        mongoServer = client.GetServer();
        MongoDatabase database = mongoServer.GetDatabase(dbName);
        MongoCollection<Note> noteCollection = database.GetCollection<Note>(collectionName);
  此处未作任何更改；只需注意以上代码可用于获取 MongoCollection 对象，以便执行插入、更新和查询，例如 **GetAllNotes()** 中的以下代码：

        collection.FindAll().ToList<Note>();

有关利用 C# MongoDB 驱动程序的详细信息，请参阅 mongodb.org 中的 [CSharp 驱动程序快速启动](http://www.mongodb.org/display/DOCS/CSharp+Driver+Quickstart "CSharp 驱动程序快速启动")。

### 添加 create 视图
现在，你将添加一个用于创建新便笺的视图。

1. 在解决方案资源管理器中，右键单击“视图 > 主页”条目，然后选择“添加 > 视图”。将此新视图命名为 **Create**，然后单击“添加”。

1. 将此视图 (**Create.cshtml**) 的自动生成的代码替换为以下内容：

        @model mongoNotes.Models.Note
        
        <script src="@Url.Content("~/Scripts/jquery-1.5.1.min.js")" type="text/javascript"></script>
        <script src="@Url.Content("~/Scripts/jquery.validate.min.js")" type="text/javascript"></script>
        <script src="@Url.Content("~/Scripts/jquery.validate.unobtrusive.min.js")" type="text/javascript"></script>
        
        @using (Html.BeginForm("Create", "Home")) {
            @Html.ValidationSummary(true)
            <fieldset>
                <legend>New Note</legend>
                <h3>New Note</h3>       
                <div class="editor-label">
                    @Html.LabelFor(model => model.Text)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.Text)
                </div>
               <p>
                    <input type="submit" value="Create" />
               </p>
           </fieldset>
        }

### 修改 index.cshtml
接下来，添加用于在你的 Web 应用上查看和创建便笺的简单布局。

1. 在“视图 > 主页”下打开 **Index.cshtml**，并将其内容替换为以下内容：  

        @model IEnumerable<mongoNotes.Models.Note>
        
        @{
            ViewBag.Title = "Notes";
        }
        
        <h2>My Notes</h2>

        <table border="1">
            <tr>
                <th>Date</th>
                <th>Note Text</th>      
            </tr>
        
        @foreach (var item in Model) {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.Date)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.Text)
                </td>       
            </tr>
        }
        
        </table>
        <div>  @Html.Partial("Create", new mongoNotes.Models.Note())</div>

### 修改 HomeController.cs
最后，HomeController 需要实例化数据访问层，并针对您的视图应用它。

1. 在解决方案资源管理器的“控制器”下打开 **HomeController.cs**，并将其内容替换为以下内容：  

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using System.Web.Mvc;
        using mongoNotes.Models;
        using System.Configuration;
        
        namespace mongoNotes.Controllers
        {
            public class HomeController : Controller, IDisposable
            {
                private Dal dal = new Dal();
                private bool disposed = false;
                //
                // GET: /Task/
        
                public ActionResult Index()
                {
                    return View(dal.GetAllNotes());
                }
        
                //
                // GET: /Task/Create
        
                public ActionResult Create()
                {
                    return View();
                }
        
                //
                // POST: /Task/Create
        
                [HttpPost]
                public ActionResult Create(Note note)
                {
                    try
                    {
                        dal.CreateNote(note);
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
    
<a name="deploy"></a>
## 部署应用程序

现在，应用程序已开发完毕，是时候在 Azure 网站中创建 Web 应用了，以托管该应用程序、配置该 Web 应用并部署代码。本节的关键是使用 MongoDB 连接字符串 (URI)。你将使用此 URI 在你的 Web 应用中配置环境变量，以便将该 URI 与你的代码分开。您应将该 URI 视为敏感信息，因为它包含用于连接到您的数据库的凭据。

### 创建新的 Web 应用并获取发布设置文件
在 Azure 网站中创建 Web 应用非常简单，特别是 Azure 为 Visual Studio 自动生成发布配置文件时。

1. 在 Azure 门户中，单击“新建”。![新建][button-new]

1. 选择“计算 > Web 应用 > 快速创建”。
	<!-- ![CreateWebApp][screen-mongolab-newwebsite] -->

1. 输入 URL 前缀。选择您喜欢的名称，但要注意此名称必须是唯一的（可能无法使用“mongoNotes”）。

1. 单击“创建 Web 应用”。

1. Web 应用创建完成时，单击 Web Apps 列表中的 Web 应用名称。将显示“Web 应用仪表板”。![WebAppDashboard][screen-mongolab-websitedashboard]

1. 在“速览”下单击“下载发布配置文件”，然后将 .PublishSettings 文件保存到所选目录中。![DownloadPublishProfile][button-website-downloadpublishprofile]

或者，你还可以直接通过 Visual Studio 配置 Web 应用。当你将 Azure 帐户链接到 Visual Studio 时，请按照提示从那里配置 Web 应用。完成后，只需在解决方案资源管理器中右键单击项目名称来部署 Azure。仍需要配置 MongoLab 连接字符串，如以下步骤中所述。

### 获取 MongoLab 连接字符串

[AZURE.INCLUDE [howto-get-connectioninfo-mongolab](../includes/howto-get-connectioninfo-mongolab.md)]

### 将连接字符串添加到 Web 应用的环境变量中

[AZURE.INCLUDE [howto-save-connectioninfo-mongolab](../includes/howto-save-connectioninfo-mongolab.md)]

### 发布 Web 应用
1. 在 Visual Studio 的解决方案资源管理器中，右键单击“mongoNotes”项目，然后选择“发布”。此时会显示“发布”对话框：  
	<!-- ![Publish][dialog-mongolab-vspublish] -->

1. 单击“导入”，然后从所选下载目录中选择 .PublishSettings 文件。此文件将自动填充“发布”对话框中的值。

1. 单击**V**“验证连接”测试该文件。

1. 验证成功后，单击“发布”。发布完成后，会打开一个新的浏览器选项卡，并显示 Web 应用。

1. 输入一些便笺文本，单击“创建”，然后查看结果！![HelloMongoAzure][screen-mongolab-sampleapp]

<a name="manage"></a>
## 管理数据库

[AZURE.INCLUDE [howto-access-mongolab-ui](../includes/howto-access-mongolab-ui.md)]

祝贺你！ 您刚刚启动了由 MongoLab 托管的 MongoDB 数据库提供支持的 C# ASP.NET 应用程序！ 现在，你拥有了 MongoLab 数据库，如有任何关于你的数据库的问题或疑虑，或者要获得有关 MongoDB 或 C# 驱动程序本身的帮助，请联系 [support@mongolab.com](mailto:support@mongolab.com)。祝您好运！

[AZURE.INCLUDE [app-service-web-whats-changed](../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../includes/app-service-web-try-app-service.md)]

[screen-mongolab-sampleapp]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/screen-mongolab-sampleapp.png
[dialog-mongolab-vspublish]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/dialog-mongolab-vspublish.png
[button-website-downloadpublishprofile]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/button-website-downloadpublishprofile.png
[screen-mongolab-websitedashboard]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/screen-mongolab-websitedashboard.png
[screen-mongolab-newwebsite]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/screen-mongolab-newwebsite.png
[button-new]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/button-new.png
[dialog-mongolab-csharp-newproject]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/dialog-mongolab-csharp-newproject.png
[dialog-mongolab-csharp-projecttemplate]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/dialog-mongolab-csharp-projecttemplate.png
[focus-mongolab-csharp-pmconsole]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/focus-mongolab-csharp-pmconsole.png
[button-store]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/button-store.png
[entry-mongolab]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/entry-mongolab.png
[button-connectioninfo]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/button-connectioninfo.png
[screen-connectioninfo]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/dialog-mongolab_connectioninfo.png
[focus-website-connectinfo]: ./media/store-mongolab-web-sites-dotnet-store-data-mongodb/focus-mongolab-websiteconnectionstring.png
[provision]: #provision
[create]: #create
[deploy]: #deploy
[manage]: #manage

 

<!---HONumber=71-->