<properties linkid="develop-net-tutorials-website-with-mongodb-mongolab" urlDisplayName="Web Site with MongoDB on MongoLab" pageTitle="创建可在 MongoLab 上使用 MongoDB 的网站 (.NET)" metaKeywords="" description="了解如何创建 Azure 网站，用于将数据存储在由 MongoLab 托管的 MongoDB 中。" metaCanonical="" services="web-sites" documentationCenter=".NET" title="使用 MongoLab 外接程序以通过 MongoDB 在 Azure 上创建 C# ASP.NET 应用程序" authors="riande" solutions="" manager="" editor="" />

# 使用 MongoLab 外接程序以通过 MongoDB 在 Azure 上创建 C# ASP.NET 应用程序

*作者：Eric Sedor，MongoLab*

各位冒险家，大家好！欢迎使用 MongoDB 即服务。在本教程中，您将：

1.  [设置数据库][设置数据库] - Azure 应用商店 [MongoLab][MongoLab] 外接程序将为您提供一个托管在 Azure 云中并且由 MongoLab 的云数据库平台管理的 MongoDB 数据库。
2.  [创建应用][创建应用] - 它将是一个用于记录的简单的 C# ASP.NET MVC 应用。
3.  [部署应用程序][部署应用程序] - 通过将一些配置联系在一起，我们将使推送代码易如反掌。
4.  [管理数据库][管理数据库] - 最后，我们将向您演示 MongoLab 基于 Web 的数据库管理门户，在此您可轻松搜索、显示和修改数据。

在本教程的任意时间内，如有任何问题，请随时发送电子邮件至 <support@mongolab.com>。

## 快速启动

如果您已拥有要使用的 Azure 应用程序和网站，或者您比较熟悉 Azure 应用商店，请使用本部分进行快速启动。否则，请继续执行下面的[设置数据库][设置数据库]。

1.  打开 Azure 应用商店。
    ![应用商店][应用商店]
2.  购买 MongoLab 外接程序。
    ![MongoLab][1]
3.  在“外接程序”列表中单击您的 MongoLab 外接程序，然后单击“连接信息”。
    ![“连接信息”按钮][“连接信息”按钮]
4.  将 MONGOLAB\_URI 复制到剪贴板。
    ![连接信ꡯ屠幕][连接信ꡯ屠幕]
    **此 URI 包含您的数据库用户名和密码。将其视为敏感信息而不要共享它。**
5.  将该值添加到您的 Azure Web 应用程序的“配置”菜单中的“连接字符串”列表：
    ![网站连接字符串][网站连接字符串]
6.  对于“名称”，请输入 MONGOLAB\_URI。
7.  对于“值”，请粘贴我们在上一节中获得的连接字符串。
8.  在“类型”下拉列表中选择“自定义”（而不是默认的“SQLAzure”）。
9.  在 Visual Studio 中，通过选择“工具”\>“库程序包管理器”\>“程序包管理器控制台”来安装 Mongo C# 驱动程序。在 PM 控制台中，键入 **Install-Package mongocsharpdriver**，然后按 **Enter**。
10. 在代码中设置挂钩以从环境变量中获得 MongoLab 连接 URI：

        using MongoDB.Driver;  
        ...
        private string connectionString = System.Environment.GetEnvironmentVariable("CUSTOMCONNSTR_MONGOLAB_URI");
        ...
        MongoUrl url = new MongoUrl(connectionString);
        MongoClient client = new MongoClient(url);

    注意：Azure 会向最初声明的连接字符串添加 **CUSTOMCONNSTR\_** 前缀，这正是代码引用 **CUSTOMCONNSTR\_MONGOLAB\_URI** 而不是 **MONGOLAB\_URI** 的原因。

现在，开始完整教程...

## <a name="provision"></a>设置数据库

[WACOM.INCLUDE [howto-provision-mongolab](../includes/howto-provision-mongolab.md)]

## <a name="create"></a>创建应用程序

在本部分中，将介绍如何创建 C# ASP.NET Visual Studio 项目，并演练如何使用 C# MongoDB 驱动程序创建简单的便签应用。您希望能够访问自己的网站、书写便笺并查看已记录的所有便笺。

您将在 Visual Studio Express 2012 for Web 中执行此开发。

### 创建项目

您的示例应用将充分使用 Visual Studio 模板以开始操作。确保使用 .NET Framework 4.0。

1.  选择“文件”\>“新建项目”。将显示“新建项目”对话框：
    ![新建项目][新建项目]
2.  选择“已安装”\>“模板”\>“Visual C#”\>“Web”。
3.  从 .NET 版本下拉菜单中选择“.NET Framework 4”（*注意：此时 Framework 4.5 将不起作用*）。

    ![ProjectFramework][ProjectFramework]

4.  选择“ASP.NET MVC 4 Web 应用程序”。
5.  输入 *mongoNotes* 作为您的**项目名称**。如果您选择了其他名称，您将需要修改在整个教程中提供的代码。
6.  单击“确定”。将显示“项目模板”对话框：
    ![项目模板][项目模板]
7.  选择“Internet 应用程序”，然后单击“确定”。将生成项目。
8.  选择“工具”\>“库程序包管理器”\>“程序包管理器控制台”。在 PM 控制台中，键入 **Install-Package mongocsharpdriver**，然后按 **Enter**。
    ![PM 控制台][PM 控制台]
    MongoDB C# 驱动程序与项目集成，并且以下行将自动添加到 *packages.config* 文件中：

        < package id="mongocsharpdriver" version="1.8" targetFramework="net40" / >

### 添加便笺模型

首先，建立一个便笺模型，该模型只具有日期和文本内容。

1.  在“解决方案资源管理器”中，右键单击“模型”，然后选择“添加”\>“类”。将此新类命名为 *Note.cs*。
2.  将此类的自动生成的代码替换为以下内容：

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

1.  在“解决方案资源管理器”中，右键单击“mongoNotes”，然后选择“添加”\>“新建文件夹”。将该文件夹命名为 **DAL**。
2.  在“解决方案资源管理器”中，右键单击“DAL”，然后选择“添加”\>“类”。将此新类命名为 *Dal.cs*。
3.  将此类的自动生成的代码替换为以下内容：

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
                MongoUrl url = new MongoUrl(connectionString);

                private string dbName = "myMongoApp";
                private string collectionName = "Notes";

                // Default constructor.        
                public Dal()
                {
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

4.  注意上面的以下代码：

        private string connectionString = System.Environment.GetEnvironmentVariable("CUSTOMCONNSTR_MONGOLAB_URI");
        private string dbName = "myMongoApp";  

    在此，您会访问一个您稍后将配置的环境变量。如果您出于开发用途正在运行一个本地 mongo 实例，则可能要暂时将此值设置为“localhost”。

设置您的数据库名称。具体而言，将 **dbName** 值设置为您在设置 MongoLab 外接程序时所输入的名称。

1.  最后，检查 **GetNotesCollection()** 中的以下代码：

        MongoClient client = new MongoClient(url);
        mongoServer = client.GetServer();
        MongoDatabase database = mongoServer.GetDatabase(dbName);
        MongoCollection<Note> noteCollection = database.GetCollection<Note>(collectionName);

    此处未作任何变动；只需注意以上代码可用于获取 MongoCollection 对象，以便执行插入、更新和查询，例如 **GetAllNotes()** 中的以下代码：

        collection.FindAll().ToList<Note>();

有关利用 C# MongoDB 驱动程序的详细信息，请参阅 mongodb.org 中的 [CSharp 驱动程序快速启动][CSharp 驱动程序快速启动]。

### 添加 create 视图

现在，您将添加一个用于创建新便笺的视图。

1.  在“解决方案资源管理器”中，右键单击“视图”\>“主页”项，然后选择“添加”\>“视图”。将此新视图命名为 **Create**，然后单击“添加”。
2.  将此视图 (**Create.cshtml**) 的自动生成的代码替换为以下内容：

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

接下来，删除用于在您的网站上查看和创建便笺的简单布局。

1.  在“视图”\>“主页”下打开 **Index.cshtml**，并将其内容替换为以下内容：

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

1.  在“解决方案资源管理器”的“控制器”下打开 **HomeController.cs**，并将其内容替换为以下内容：

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

## <a name="deploy"></a>部署应用程序

现在，应用程序已开发完毕，是时候创建一个用于托管该应用程序的 Azure 网站，配置该网站，并部署代码了。本节的关键是使用 MongoDB 连接字符串 (URI)。您将使用此 URI 在您的网站中配置环境变量以便将该 URI 与您的代码分开。您应将该 URI 视为敏感信息，因为它包含用于连接到您的数据库的凭据。

### 创建新网站并获取发布设置文件

在 Azure 中创建网站非常简单，特别是 Azure 为 Visual Studio 自动生成发布配置文件时。

1.  在 Azure 门户中，单击“新建”。
    ![新建][新建]
2.  选择“计算”\>“网站”\>“快速创建”。
    ![创建网站][创建网站]
3.  输入 URL 前缀。选择您喜欢的名称，但要注意此名称必须是唯一的（可能无法使用“mongoNotes”）。
4.  单击“创建网站”。
5.  网站创建完成时，单击网站列表中的网站名称。将显示网站仪表板。
    ![网站仪表板][网站仪表板]
6.  在“速览”下单击“下载发布配置文件”，然后将 .PublishSettings 文件保存到所选目录。
    ![下载发布配置文件][下载发布配置文件]

### 获取 MongoLab 连接字符串

[WACOM.INCLUDE [howto-get-connectioninfo-mongolab](../includes/howto-get-connectioninfo-mongolab.md)]

### 将该连接字符串添加到网站的环境变量中

[WACOM.INCLUDE [howto-save-connectioninfo-mongolab](../includes/howto-save-connectioninfo-mongolab.md)]

### 发布网站

1.  在 Visual Studio 的“解决方案资源管理器”中，右键单击“mongoNotes”项目，然后选择“发布”。此时将显示“发布”对话框：
    ![发布][发布]
2.  单击“导入”，然后从所选的下载目录选择 .PublishSettings 文件。此文件将自动填充“发布”对话框中的值。
3.  单击“验证连接”以测试该文件。
4.  验证成功后，单击“发布”。发布完成后，将打开一个新的浏览器选项卡并显示网站。
5.  输入一些便笺文本、单击“创建”，然后查看结果！
    ![HelloMongoAzure][HelloMongoAzure]

## <a name="manage"></a>管理数据库

[WACOM.INCLUDE [howto-access-mongolab-ui](../includes/howto-access-mongolab-ui.md)]

祝贺您！您刚刚启动了由 MongoLab 托管的 MongoDB 数据库提供支持的 C# ASP.NET 应用程序！现在，您拥有了一个 MongoLab 数据库，如有任何关于您的数据库的问题或疑虑，或者要获得有关 MongoDB 或 C# 驱动程序本身的帮助，请联系 <support@mongolab.com>。祝您好运！

  [设置数据库]: #provision
  [MongoLab]: http://mongolab.com
  [创建应用]: #create
  [部署应用程序]: #deploy
  [管理数据库]: #manage
  [应用商店]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/button-store.png
  [1]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/entry-mongolab.png
  [“连接信息”按钮]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/button-connectioninfo.png
  [连接信ꡯ屠幕]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/dialog-mongolab_connectioninfo.png
  [网站连接字符串]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/focus-mongolab-websiteconnectionstring.png
  [新建项目]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/dialog-mongolab-csharp-newproject.png
  [ProjectFramework]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/focus-dotNet-Framework4-mongolab.png
  [项目模板]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/dialog-mongolab-csharp-projecttemplate.png
  [PM 控制台]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/focus-mongolab-csharp-pmconsole.png
  [CSharp 驱动程序快速启动]: http://www.mongodb.org/display/DOCS/CSharp+Driver+Quickstart "CSharp 驱动程序快速启动"
  [新建]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/button-new.png
  [创建网站]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/screen-mongolab-newwebsite.png
  [网站仪表板]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/screen-mongolab-websitedashboard.png
  [下载发布配置文件]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/button-website-downloadpublishprofile.png
  [发布]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/dialog-mongolab-vspublish.png
  [HelloMongoAzure]: ./media/partner-mongodb-web-sites-dotnet-use-mongolab/screen-mongolab-sampleapp.png
