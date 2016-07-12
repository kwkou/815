<properties 
	pageTitle="如何使用 Redis 缓存创建 Web 应用 | Azure" 
	description="了解如何使用 Redis 缓存创建 Web 应用" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="05/05/2016"
	wacn.date="05/30/2016"/>

# 如何使用 Redis 缓存创建 Web 应用

> [AZURE.SELECTOR]
- [.NET](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/)
- [ASP.NET](/documentation/articles/cache-web-app-howto/)
- [Node.js](/documentation/articles/cache-nodejs-get-started/)
- [Java](/documentation/articles/cache-java-get-started/)
- [Python](/documentation/articles/cache-python-get-started/)

本教程将向你介绍如何使用 Visual Studio 2015 创建 ASP.NET Web 应用程序并将其部署到 Azure 中的 Web 应用。该示例应用程序显示了数据库中的团队统计信息列表，并显示了使用 Azure Redis 缓存通过缓存存储和检索数据的不同方式。完成本教程后，你将有一个运行的 Web 应用，该应用可以对数据库执行读写操作，已通过 Azure Redis 缓存进行优化，并且托管在 Azure 中。

学习内容：

-	如何在 Visual Studio 中创建 ASP.NET MVC 5 Web 应用程序。
-	如何使用实体框架从数据库访问数据。
-	如何使用 Azure Redis 缓存来存储和检索数据，以便改进数据吞吐量并降低数据库负载。
-	如何使用 Redis 排序的集来检索排名前 5 的团队。
-	如何使用 ARM 模板为应用程序预配 Azure 资源。
-	如何使用 Visual Studio 将应用程序发布到 Azure。

## 先决条件

若要完成本教程，必须具备以下先决条件。

-	[Azure 帐户](#azure-account)
-	[带有用于 .NET 的 Azure SDK 的 Visual Studio 2015](#visual-studio-2015-with-the-azure-sdk-for-net)

###<a name="azure-account"></a> Azure 帐户

完成本教程需要有一个 Azure 帐户。你可以：

* [注册一元 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=redis_cache_hero)。获取可用来尝试付费版 Azure 服务的信用额度。即使在信用额度用完之后，你也可以保留帐户和使用免费的 Azure 服务和功能。

###<a name="visual-studio-2015-with-the-azure-sdk-for-net"></a> 带有用于 .NET 的 Azure SDK 的 Visual Studio 2015

本教程专为带有用于 [.NET 2.8.2 的 Azure SDK](/documentation/articles/dotnet-sdk/) 的 Visual Studio 2015 或更高版而编写。[单击此处下载最新的用于 Visual Studio 2015 的 Azure SDK](http://go.microsoft.com/fwlink/?linkid=518003)。如果尚未安装 Visual Studio，则会随 SDK 一起自动安装。

如果有 Visual Studio 2013，则可以[下载最新的 Azure SDK for Visual Studio 2013](http://go.microsoft.com/fwlink/?LinkID=324322)。某些屏幕看起来可能与本教程中显示的插图不同。

>[AZURE.NOTE] 根据计算机上已有 SDK 依赖项数量的不同，安装 SDK 可能耗时较长，从几分钟到半小时或更长时间不等。

## 创建 Visual Studio 项目

1. 打开 Visual Studio，然后依次单击“文件”、“新建”、“项目”。

2. 展开“模板”列表中的“Visual C#”节点，选择“云”，然后单击“ASP.NET Web 应用程序”。确保选中“.NET Framework 4.5.2”。在“名称”框中键入“ContosoTeamStats”，然后单击“确定”。
 
    ![创建项目][cache-create-project]

3. 选择“MVC”作为项目类型。清除“在云中托管”复选框。在本教程的后续步骤中，你需要[预配 Azure 资源](#provision-the-azure-resources)以及[将应用程序发布到 Azure](#publish-the-application-to-azure)。勾选“在云中托管”即可通过 Visual Studio 预配 Azure Web 应用，如需此方面的示例，请参阅[配合 ASP.NET 和 Visual Studio 使用 Azure 中的 Web Apps 入门](/documentation/articles/web-sites-dotnet-get-started/)。

    ![选择项目模板][cache-select-template]

4. 单击“确定”以创建该项目。

## 创建 ASP.NET MVC 应用程序

在本教程的此部分，你将创建可以从数据库读取和显示团队统计信息的基本应用程序。

-	[添加模型](#add-the-model)
-	[添加控制器](#add-the-controller)
-	[配置视图](#configure-the-views)

###<a name="add-the-model"></a> 添加模型

1. 右键单击解决方案资源管理器中的“模型”，然后选择“添加”>“类”。 

    ![添加模型][cache-model-add-class]

2. 输入 `Team` 作为类名，然后单击“添加”。

    ![添加模型类][cache-model-add-class-dialog]

3. 将 `Team.cs` 文件顶部的 `using` 语句替换为以下 using 语句。


		using System;
		using System.Collections.Generic;
		using System.Data.Entity;
		using System.Data.Entity.SqlServer;


4. 将 `Team` 类的定义替换为以下代码段，其中包含更新的 `Team` 类定义以及某些其他的实体框架帮助器类。有关本教程中使用的实体框架 Code First 方法的详细信息，请参阅[对新数据库使用 Code First](https://msdn.microsoft.com/data/jj193542)。


		public class Team
		{
		    public int ID { get; set; }
		    public string Name { get; set; }
		    public int Wins { get; set; }
		    public int Losses { get; set; }
		    public int Ties { get; set; }
		
		    static public void PlayGames(IEnumerable<Team> teams)
		    {
		        // Simple random generation of statistics.
		        Random r = new Random();
		
		        foreach (var t in teams)
		        {
		            t.Wins = r.Next(33);
		            t.Losses = r.Next(33);
		            t.Ties = r.Next(0, 5);
		        }
		    }
		}
		
		public class TeamContext : DbContext
		{
		    public TeamContext()
		        : base("TeamContext")
		    {
		    }
		
		    public DbSet<Team> Teams { get; set; }
		}
		
		public class TeamInitializer : CreateDatabaseIfNotExists<TeamContext>
		{
		    protected override void Seed(TeamContext context)
		    {
		        var teams = new List<Team>
		        {
		            new Team{Name="Adventure Works Cycles"},
		            new Team{Name="Alpine Ski House"},
		            new Team{Name="Blue Yonder Airlines"},
		            new Team{Name="Coho Vineyard"},
		            new Team{Name="Contoso, Ltd."},
		            new Team{Name="Fabrikam, Inc."},
		            new Team{Name="Lucerne Publishing"},
		            new Team{Name="Northwind Traders"},
		            new Team{Name="Consolidated Messenger"},
		            new Team{Name="Fourth Coffee"},
		            new Team{Name="Graphic Design Institute"},
		            new Team{Name="Nod Publishers"}
		        };
		
		        Team.PlayGames(teams);
		
		        teams.ForEach(t => context.Teams.Add(t));
		        context.SaveChanges();
		    }
		}
		
		public class TeamConfiguration : DbConfiguration
		{
		    public TeamConfiguration()
		    {
		        SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
		    }
		}


2. 在“解决方案资源管理器”中，双击“web.config”将其打开。

    ![Web.config][cache-web-config]

3.  将以下连接字符串添加到 `connectionStrings` 节。连接字符串的名称必须与实体框架数据库上下文类（即 `TeamContext`）的名称相匹配。

		<add name="TeamContext" connectionString="Data Source=(LocalDB)\v11.0;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" providerName="System.Data.SqlClient" />


    添加完此项以后，`connectionStrings` 节应如以下示例所示。


		<connectionStrings>
			<add name="DefaultConnection" connectionString="Data Source=(LocalDb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-ContosoTeamStats-20160216120918.mdf;Initial Catalog=aspnet-ContosoTeamStats-20160216120918;Integrated Security=True"
				providerName="System.Data.SqlClient" />
			<add name="TeamContext" connectionString="Data Source=(LocalDB)\v11.0;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" 	providerName="System.Data.SqlClient" />
		</connectionStrings>

###<a name="add-the-controller"></a> 添加控制器

1. 按“F6”生成项目。 
2. 在“解决方案资源管理器”中，右键单击“Controllers”文件夹，然后选择“添加”，再选择“控制器”。

    ![添加控制器][cache-add-controller]

3. 选择“使用实体框架的包含视图的 MVC 5 控制器”并单击“添加”。如果在单击“添加”后出现错误，请确保你已先生成该项目。

    ![添加控制器类][cache-add-controller-class]

5. 从“模型类”下拉列表中选择“团队(ContosoTeamStats.Models)”。从“数据上下文类”下拉列表中选择“TeamContext (ContosoTeamStats.Models)”。在“控制器”名称文本框中键入 `TeamsController`（如果尚未自动填充）。单击“添加”创建控制器类并添加默认视图。

    ![配置控制器][cache-configure-controller]

4. 在“解决方案资源管理器”中展开“Global.asax”，然后双击“Global.asax.cs”将其打开。

    ![Global.asax.cs][cache-global-asax]

5. 将以下两个 using 语句添加到文件顶部的其他 using 语句下方。


        using System.Data.Entity;
		using ContosoTeamStats.Models;


6. 在 `Application_Start` 方法的末尾添加以下代码行。


	    Database.SetInitializer<TeamContext>(new TeamInitializer());


7. 在“解决方案资源管理器”中展开 `App_Start`，然后双击 `RouteConfig.cs`。

    ![RouteConfig.cs][cache-RouteConfig-cs]

8. 将以下代码的 `RegisterRoutes` 方法中的 `controller = "Home"` 替换为 `controller = "Teams"`，如以下示例所示。


	    routes.MapRoute(
	        name: "Default",
	        url: "{controller}/{action}/{id}",
	        defaults: new { controller = "Teams", action = "Index", id = UrlParameter.Optional }
	    );


###<a name="configure-the-views"></a> 配置视图

1. 在“解决方案资源管理器”中，先展开 **Views** 文件夹，再展开 **Shared** 文件夹，然后双击 **\_Layout.cshtml**。

    ![\_Layout.cshtml][cache-layout-cshtml]

2. 更改 `title` 元素的内容，将 `My ASP.NET Application` 替换为 `Contoso Team Stats`，如以下示例所示。


	    <title>@ViewBag.Title - Contoso Team Stats</title>


3. 在 `body` 节中，更新第一个 `Html.ActionLink` 语句，将 `Application name` 替换为 `Contoso Team Stats`，将 `Home` 替换为 `Teams`。
	-	之前：`@Html.ActionLink("Application name", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })`
	-	之后：`@Html.ActionLink("Contoso Team Stats", "Index", "Teams", new { area = "" }, new { @class = "navbar-brand" })`

    ![代码更改][cache-layout-cshtml-code]

4. 按“Ctrl+F5”构建并运行应用程序。此版本的应用程序直接从数据库读取结果。请注意已通过“使用实体框架的包含视图的 MVC 5 控制器”基架自动添加到应用程序的“新建”、“编辑”、“详细信息”和“删除”操作。在本教程的下一部分，你需要添加优化数据访问所需的 Redis 缓存，并向应用程序提供其他功能。

![入门应用程序][cache-starter-application]

##<a name="configure-the-application-to-use-redis-cache"></a> 配置应用程序以使用 Redis 缓存

在本教程的此部分，你需要对示例应用程序进行配置，以便使用 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) 缓存客户端通过 Azure Redis 缓存实例来存储和检索 Contoso 团队统计信息。

-	[配置应用程序以使用 StackExchange.Redis](#configure-the-application-to-use-stackexchangeredis)
-	[更新 TeamsController 类，以便从缓存或数据库返回结果](#update-the-teamscontroller-class-to-return-results-from-the-cache-or-the-database)
-	[更新用于缓存的 Create、Edit 和 Delete 方法](#update-the-create-edit-and-delete-methods-to-work-with-the-cache)
-	[更新用于缓存的“团队索引”视图](#update-the-teams-index-view-to-work-with-the-cache)


###<a name="configure-the-application-to-use-stackexchangeredis"></a> 配置应用程序以使用 StackExchange.Redis

1. 若要使用 StackExchange.Redis NuGet 包配置客户端应用程序，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。 

    ![管理 NuGet 包][redis-cache-manage-nuget-menu]

2. 将 **StackExchange.Redis** 键入搜索文本框，从结果中选择所需版本，然后单击“安装”。

    ![StackExchange.Redis NuGet 程序包][redis-cache-stack-exchange-nuget]

    NuGet 程序包会给客户端应用程序下载并添加所需的程序集引用，以访问带 StackExchange.Redis 缓存客户端的 Azure Redis Cache。如果你希望使用 **StackExchange.Redis** 客户端库强名称版本，请选择 **StackExchange.Redis.StrongName**；否则选择 **StackExchange.Redis**。

3. 在“解决方案资源管理器”中，展开“Controllers”文件夹，然后双击“TeamsController.cs”将其打开。

    ![团队控制器][cache-teamscontroller]

4. 将以下两个 using 语句添加到 **TeamsController.cs**。

	    using System.Configuration;
        using StackExchange.Redis;

5. 将以下两个属性添加到 `TeamsController` 类。

	    // Redis Connection string info
	    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	    {
            string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
            return ConnectionMultiplexer.Connect(cacheConnection);
	    });
	
	    public static ConnectionMultiplexer Connection
	    {
	        get
	        {
	            return lazyConnection.Value;
	        }
	    }
  
1. 在计算机中创建一个名为 `WebAppPlusCacheAppSecrets.config` 的文件，将其置于不会使用示例应用程序的源代码对其进行检查的位置（如果你决定在某个位置对其进行检查）。在此示例中，`AppSettingsSecrets.config` 文件位于 `C:\AppSecrets\WebAppPlusCacheAppSecrets.config`。

    编辑 `WebAppPlusCacheAppSecrets.config` 文件，添加以下内容。如果在本地运行该应用程序，则可利用此信息连接到 Azure Redis 缓存实例。在本教程中，你随后将预配 Azure Redis 缓存实例并更新缓存名称和密码。如果你不打算在本地运行示例应用程序，则可跳过此文件的创建步骤以及后续的文件引用步骤，因为当你部署到 Azure 时，应用程序会从 Web 应用的应用设置而非从此文件中检索缓存连接信息。由于 `WebAppPlusCacheAppSecrets.config` 不与应用程序一起部署到 Azure，因此除非你要在本地运行应用程序，否则不需要它。


		<appSettings>
		  <add key="CacheConnection" value="MyCache.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=..."/>
		</appSettings>


2. 在“解决方案资源管理器”中，双击“web.config”将其打开。

    ![Web.config][cache-web-config]

3. 将以下 `file` 属性添加到 `appSettings` 元素。如果你使用了其他文件名或位置，请使用这些值来替换示例中显示的值。
	-	之前：`<appSettings>`
	-	之后：` <appSettings file="C:\AppSecrets\WebAppPlusCacheAppSecrets.config">`

    ASP.NET 运行时合并了外部文件的内容以及 `<appSettings>` 元素中的标记。如果找不到指定的文件，运行时将忽略文件属性。应用程序的源代码中将不包括你的机密（连接到缓存的连接字符串）。将 Web 应用部署到 Azure 时，不会部署 `WebAppPlusCacheAppSecrests.config` 文件（这正是你所需要的）。可以通过多种方式在 Azure 中指定这些机密，而在本教程中，当你在后续的教程步骤中[预配 Azure 资源](#provision-the-azure-resources)时，系统将为你自动配置这些机密。有关如何在 Azure 中处理机密的详细信息，请参阅 [Best practices for deploying passwords and other sensitive data to ASP.NET and Azure Web App](http://www.asp.net/identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure)（将密码和其他敏感数据部署到 ASP.NET 和 Azure Web 应用的最佳做法）。


###<a name="update-the-teamscontroller-class-to-return-results-from-the-cache-or-the-database"></a> 更新 TeamsController 类，以便从缓存或数据库返回结果

在此示例中，团队统计信息可以从数据库或缓存中检索。团队统计信息以序列化 `List<Team>` 的形式存储在缓存中，同时以排序集的形式按 Redis 数据类型存储。从排序集检索项目时，可以检索部分项目或所有项目，也可以查询特定项目。在此示例中，你将按胜利数在排序集中查询排名前 5 的团队。

>[AZURE.NOTE] 不需在缓存中以多种格式存储团队统计信息即可使用 Azure Redis 缓存。本教程以多种格式演示了缓存数据时可以使用的不同方法和不同数据类型的一部分。



1. 将以下 using 语句添加到 `TeamsController.cs` 文件顶部，与其他 using 语句放置在一起。

		using System.Diagnostics;
		using Newtonsoft.Json;

2. 将当前的 `public ActionResult Index()` 方法替换为以下实现。


		// GET: Teams
		public ActionResult Index(string actionType, string resultType)
		{
		    List<Team> teams = null;
		
		    switch(actionType)
		    {
		        case "playGames": // Play a new season of games.
		            PlayGames();
		            break;
		
		        case "clearCache": // Clear the results from the cache.
		            ClearCachedTeams();
		            break;
		
		        case "rebuildDB": // Rebuild the database with sample data.
		            RebuildDB();
		            break;
		    }
		
		    // Measure the time it takes to retrieve the results.
		    Stopwatch sw = Stopwatch.StartNew();
		
		    switch(resultType)
		    {
		        case "teamsSortedSet": // Retrieve teams from sorted set.
		            teams = GetFromSortedSet();
		            break;
		
		        case "teamsSortedSetTop5": // Retrieve the top 5 teams from the sorted set.
		            teams = GetFromSortedSetTop5();
		            break;
		
		        case "teamsList": // Retrieve teams from the cached List<Team>.
		            teams = GetFromList();
		            break;
		
		        case "fromDB": // Retrieve results from the database.
		        default:
		            teams = GetFromDB();
		            break;
		    }
		
		    sw.Stop();
		    double ms = sw.ElapsedTicks / (Stopwatch.Frequency / (1000.0));

            // Add the elapsed time of the operation to the ViewBag.msg.
		    ViewBag.msg += " MS: " + ms.ToString();
		
		    return View(teams);
		}


3. 将以下三种方法添加到 `TeamsController` 类，以便实现在以前的代码段中添加的 switch 语句中的 `playGames`、`clearCache` 和 `rebuildDB` 操作类型。

    `PlayGames` 方法可以通过对赛季进行模拟来更新团队统计信息，将结果保存到数据库，然后从缓存中清除现已过时的数据。


	    void PlayGames()
	    {
	        ViewBag.msg += "Updating team statistics. ";
	        // Play a "season" of games.
	        var teams = from t in db.Teams
	                    select t;
	
	        Team.PlayGames(teams);
	
	        db.SaveChanges();
	
	        // Clear any cached results
	        ClearCachedTeams();
	    }


    `RebuildDB` 方法使用默认的团队集来重新初始化数据库，为其生成统计信息，然后从缓存中清除现已过时的数据。
	
        void RebuildDB()
        {
            ViewBag.msg += "Rebuilding DB. ";
            // Delete and re-initialize the database with sample data.
            db.Database.Delete();
            db.Database.Initialize(true);

	        // Clear any cached results
	        ClearCachedTeams();
        }


    `ClearCachedTeams` 方法从缓存中删除任何已缓存的团队统计信息。

	
	    void ClearCachedTeams()
	    {
	        IDatabase cache = Connection.GetDatabase();
	        cache.KeyDelete("teamsList");
	        cache.KeyDelete("teamsSortedSet");
	        ViewBag.msg += "Team data removed from cache. ";
	    } 


4. 将以下四种方法添加到 `TeamsController` 类，以便通过不同的方式从缓存和数据库检索团队统计信息。这些方法均会返回 `List<Team>`，然后通过视图将其显示出来。

    `GetFromDB` 方法从数据库读取团队统计信息。

	    List<Team> GetFromDB()
	    {
	        ViewBag.msg += "Results read from DB. ";
	        var results = from t in db.Teams
	            orderby t.Wins descending
	            select t; 
	
	        return results.ToList<Team>();
	    }


    `GetFromList` 方法以序列化 `List<Team>` 的形式从缓存读取团队统计信息。如果缓存未命中，则会从数据库读取团队统计信息，然后将该信息存储在缓存中留待下次使用。在此示例中，我们将通过 JSON.NET 序列化来序列化进出缓存的 .NET 对象。有关详细信息，请参阅[如何处理 Azure Redis 缓存中的 .NET 对象](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#work-with-net-objects-in-the-cache)。

        List<Team> GetFromList()
        {
            List<Team> teams = null;

            IDatabase cache = Connection.GetDatabase();
            string serializedTeams = cache.StringGet("teamsList");
            if (!String.IsNullOrEmpty(serializedTeams))
            {
                teams = JsonConvert.DeserializeObject<List<Team>>(serializedTeams);

                ViewBag.msg += "List read from cache. ";
            }
            else
            {
                ViewBag.msg += "Teams list cache miss. ";
                // Get from database and store in cache
                teams = GetFromDB();

                ViewBag.msg += "Storing results to cache. ";
                cache.StringSet("teamsList", JsonConvert.SerializeObject(teams));
            }
            return teams;
        }


    `GetFromSortedSet` 方法从缓存的排序集读取团队统计信息。如果缓存未命中，则会从数据库读取团队统计信息，然后将该信息以排序集的形式存储在缓存中。


	    List<Team> GetFromSortedSet()
	    {
            List<Team> teams = null;
            IDatabase cache = Connection.GetDatabase();
            // If the key teamsSortedSet is not present, this method returns a 0 length collection.
            var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", order: Order.Descending);
            if (teamsSortedSet.Count() > 0)
            {
                ViewBag.msg += "Reading sorted set from cache. ";
                teams = new List<Team>();
                foreach (var t in teamsSortedSet)
                {
                    Team tt = JsonConvert.DeserializeObject<Team>(t.Element);
                    teams.Add(tt);
                }
            }
	        else
	        {
	            ViewBag.msg += "Teams sorted set cache miss. ";
	
	            // Read from DB
	            teams = GetFromDB();
	
	            ViewBag.msg += "Storing results to cache. ";
	            foreach (var t in teams)
	            {
	                Console.WriteLine("Adding to sorted set: {0} - {1}", t.Name, t.Wins);
	                cache.SortedSetAdd("teamsSortedSet", JsonConvert.SerializeObject(t), t.Wins);
	            }
	        }
	        return teams;
	    }


    `GetFromSortedSetTop5` 方法从缓存的排序集中读取排名前 5 的团队。该方法首先会检查缓存中是否存在 `teamsSortedSet` 键。如果该键不存在，则会调用 `GetFromSortedSet` 方法以读取团队统计信息并将其存储在缓存中。接下来会对缓存的排序集进行查询，以便返回排名前 5 的团队。


        List<Team> GetFromSortedSetTop5()
        {
            List<Team> teams = null;
            IDatabase cache = Connection.GetDatabase();

            // If the key teamsSortedSet is not present, this method returns a 0 length collection.
            var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
            if(teamsSortedSet.Count() == 0)
            {
                // Load the entire sorted set into the cache.
                GetFromSortedSet();

                // Retrieve the top 5 teams.
                teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
            }

            ViewBag.msg += "Retrieving top 5 teams from cache. ";
            // Get the top 5 teams from the sorted set
            teams = new List<Team>();
            foreach (var team in teamsSortedSet)
            {
                teams.Add(JsonConvert.DeserializeObject<Team>(team.Element));
            }
            return teams;
        }


###<a name="update-the-create-edit-and-delete-methods-to-work-with-the-cache"></a> 更新用于缓存的 Create、Edit 和 Delete 方法

作为此示例一部分生成的基架代码包括用于添加、编辑和删除团队的方法。只要添加、编辑或删除了团队，缓存中的数据就变成了过时的数据。在本部分，你需要修改这三种清除缓存团队的方法，避免出现缓存与数据库不同步的情况。

1. 浏览到 `TeamsController` 类中的 `Create(Team team)` 方法。添加对 `ClearCachedTeams` 方法的调用，如以下示例所示。


	    // POST: Teams/Create
	    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
	    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
	    [HttpPost]
	    [ValidateAntiForgeryToken]
	    public ActionResult Create([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
	    {
	        if (ModelState.IsValid)
	        {
	            db.Teams.Add(team);
	            db.SaveChanges();
	            // When a team is added, the cache is out of date.
	            // Clear the cached teams.
	            ClearCachedTeams();
	            return RedirectToAction("Index");
	        }
	
	        return View(team);
	    }


2. 浏览到 `TeamsController` 类中的 `Edit(Team team)` 方法。添加对 `ClearCachedTeams` 方法的调用，如以下示例所示。


	    // POST: Teams/Edit/5
	    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
	    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
	    [HttpPost]
	    [ValidateAntiForgeryToken]
	    public ActionResult Edit([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
	    {
	        if (ModelState.IsValid)
	        {
	            db.Entry(team).State = EntityState.Modified;
	            db.SaveChanges();
	            // When a team is edited, the cache is out of date.
	            // Clear the cached teams.
	            ClearCachedTeams();
	            return RedirectToAction("Index");
	        }
	        return View(team);
		}


3. 浏览到 `TeamsController` 类中的 `DeleteConfirmed(int id)` 方法。添加对 `ClearCachedTeams` 方法的调用，如以下示例所示。


	    // POST: Teams/Delete/5
	    [HttpPost, ActionName("Delete")]
	    [ValidateAntiForgeryToken]
	    public ActionResult DeleteConfirmed(int id)
	    {
	        Team team = db.Teams.Find(id);
	        db.Teams.Remove(team);
	        db.SaveChanges();
	        // When a team is deleted, the cache is out of date.
	        // Clear the cached teams.
	        ClearCachedTeams();
	        return RedirectToAction("Index");
	    }


###<a name="update-the-teams-index-view-to-work-with-the-cache"></a> 更新用于缓存的“团队索引”视图

1. 在“解决方案资源管理器”中，先展开 **Views** 文件夹，再展开 **Teams** 文件夹，然后双击“Index.cshtml”。

    ![Index.cshtml][cache-views-teams-index-cshtml]

2. 在该文件顶部附近查找以下段落元素。

    ![操作表][cache-teams-index-table]

    这是创建新团队的链接。将段落元素替换为下表。该表的操作链接可用于创建新的团队、举行新赛季的比赛、清除缓存、以多种格式从缓存中检索团队、从数据库检索团队，以及使用最新的示例数据重新构建数据库。


		<table class="table">
		    <tr>
		        <td>
		            @Html.ActionLink("Create New", "Create")
		        </td>
		        <td>
		            @Html.ActionLink("Play Season", "Index", new { actionType = "playGames" })
		        </td>
		        <td>
		            @Html.ActionLink("Clear Cache", "Index", new { actionType = "clearCache" })
		        </td>
		        <td>
		            @Html.ActionLink("List from Cache", "Index", new { resultType = "teamsList" })
		        </td>
		        <td>
		            @Html.ActionLink("Sorted Set from Cache", "Index", new { resultType = "teamsSortedSet" })
		        </td>
		        <td>
		            @Html.ActionLink("Top 5 Teams from Cache", "Index", new { resultType = "teamsSortedSetTop5" })
		        </td>
		        <td>
		            @Html.ActionLink("Load from DB", "Index", new { resultType = "fromDB" })
		        </td>
		        <td>
		            @Html.ActionLink("Rebuild DB", "Index", new { actionType = "rebuildDB" })
		        </td>
		    </tr>    
		</table>


3. 滚动到 **Index.cshtml** 文件底部，然后添加下面的 `tr` 元素，使之成为文件中最后一个表的最后一行。

        <tr><td colspan="5">@ViewBag.Msg</td></tr>

    此行显示 `ViewBag.Msg` 的值，其中包含有关当前操作的状态报告，当前操作是在你单击前一步骤中的某个操作链接时设置的。

    ![状态消息][cache-status-message]

4. 按“F6”生成项目。

##<a name="provision-the-azure-resources"></a> 预配 Azure 资源

若要在 Azure 中托管应用程序，必须先预配应用程序所需的 Azure 服务。本教程中的示例应用程序使用以下 Azure 服务。

-	Azure Redis Cache
-	Azure Web 应用
-	SQL 数据库

从 GitHub 下载[此 ARM 模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-redis-cache-sql-database/azuredeploy.json)，并进行以下修改。

对于参数 **skuName**，请删除允许的值“P1”、“P2”、“P3”和“P4”，因为 Premium Web App 在 Azure 中国区不可用。


完成修改后，请使用以下 PowerShell 命令来部署模板。

	New-AzureRmResourceGroup -Name MyWebAppPlusCache -Location "China East"

    New-AzureRmResourceGroupDeployment -ResourceGroupName MyWebAppPlusCache -TemplateFile azuredeploy.json

注意：可以使用参数对象来指定参数。有关详细信息，请参阅 [Deploy resources with Azure Resource Manager templates](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署资源）。

##<a name="publish-the-application-to-azure"></a> 将应用程序发布到 Azure

在教程的这一步中，你需要将应用程序发布到 Azure 并在云中运行。

1. 在 Visual Studio 中右键单击“ContosoTeamStats”项目，然后选择“发布”。

    ![发布][cache-publish-app]

2. 单击“导入”，然后选择已下载的“发布配置文件”。

	如果你尚未创建 Web 应用，可登录到 [Azure Management Portal](https://manage.windowsazure.cn/)（Azure 经典管理门户）创建一个，然后转到“仪表板”页，在“速览”下下载“发布配置文件”。

4. 单击“验证连接”对设置进行验证，然后单击“发布”。

    ![发布][cache-publish]

    几分钟后，发布过程完成，运行的示例应用程序会启动浏览器。如果在验证或发布时出现 DNS 错误，而应用程序的 Azure 资源的预配过程才刚刚完成，则请稍等片刻，然后重试。

    ![添加的缓存][cache-added-to-application]

下表描述了示例应用程序中的每个操作链接。

| 操作 | 说明 |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 新建 | 创建新的团队。 |
| 举行赛季的比赛 | 举行一赛季的比赛、更新团队统计信息，以及从缓存中清除任何过时的团队数据。 |
| 清除缓存 | 清除缓存中的团队统计信息。 |
| 缓存中的列表 | 检索缓存中的团队统计信息。如果缓存未命中，则从数据库加载统计信息，并将其保存到缓存中以便下次使用。 |
| 缓存中的排序集 | 使用排序集检索缓存中的团队统计信息。如果缓存未命中，则从数据库加载统计信息，并使用排序集将其保存到缓存中。 |
| 缓存中排名前 5 的团队 | 使用排序集检索缓存中排名前 5 的团队。如果缓存未命中，则从数据库加载统计信息，并使用排序集将其保存到缓存中。 |
| 从数据库加载 | 检索数据库中的团队统计信息。 |
| 重新生成数据库 | 重新生成数据库并使用示例团队数据重新加载它。 |
| 编辑/详细信息/删除 | 编辑团队、查看团队详细信息、删除团队。 |


单击部分操作，试验如何从不同的源检索数据。请注意通过不同方式从数据库和缓存检索数据所存在的时间差异。

## 完成应用程序的操作以后，删除相关资源

完成示例性的教程应用程序以后，即可删除所用的 Azure 资源，以便节省成本和资源。如果你的所有资源都包含在同一资源组中，则可通过删除资源组这一个操作来删除所有资源。

1. 登录到 [Azure 门户](https://portal.azure.cn)，然后单击“资源组”。
2. 将资源组的名称键入“筛选项目...”文本框中。
3. 单击资源组右侧的“...”。
4. 单击“删除”。

    ![删除][cache-delete-resource-group]

5. 键入资源组的名称，然后单击“删除”。

    ![确认删除][cache-delete-confirm]

几分钟后，资源组及其包含的所有资源就会被删除。

>[AZURE.IMPORTANT] 请注意，删除资源组这一操作是不可逆的，资源组以及其中的所有资源会被永久删除。请确保你不会意外删除错误的资源组或资源。如果你是在现有的资源组中为托管此示例而创建了相关资源，则可从各自的边栏选项卡逐个删除这些资源。

## 在本地计算机上运行示例应用程序

若要在计算机上本地运行应用程序，则需在 Azure Redis 缓存实例中缓存数据。

-	如果你已根据上一部分所述将应用程序发布到 Azure，则可使用在该步骤中预配的 Azure Redis 缓存实例。
-	如果你有另一个现成的 Azure Redis 缓存实例，则可使用该实例在本地运行此示例。
-	如果你需要创建 Azure Redis 缓存实例，则可按[创建缓存](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#create-a-cache)中的步骤来执行操作。

选定或创建要使用的缓存以后，即可使用以下 PowerShell 命令来获取缓存的属性和访问密钥。

    Get-AzureRmRedisCache -Name <cache name> -ResourceGroupName <resource group name>

    Get-AzureRmRedisCacheKey -Name <cache name> -ResourceGroupName <resource group name>

1. 使用所选编辑器打开你在本教程的[配置应用程序以使用 Redis 缓存](#configure-the-application-to-use-redis-cache)步骤中创建的 `WebAppPlusCacheAppSecrets.config` 文件。

2. 编辑 `value` 属性，将 `MyCache.redis.cache.chinacloudapi.cn` 替换为缓存的主机名，并指定缓存的主密钥或辅助密钥作为密码。


		<appSettings>
		  <add key="CacheConnection" value="MyCache.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=..."/>
		</appSettings>


3. 按“Ctrl+F5”运行应用程序。

>[AZURE.NOTE] 请注意，由于应用程序（包括数据库）是在本地运行，而 Redis 缓存是托管在 Azure 中，因此从表现上看缓存可能不如数据库。为了获得最佳性能，应让客户端应用程序和 Azure Redis 缓存实例位于同一位置。

## 后续步骤

-	在 [ASP.NET](http://asp.net/) 站点上了解有关 [Getting Started with ASP.NET MVC 5](http://www.asp.net/mvc/overview/getting-started/introduction/getting-started)（ASP.NET MVC 5 入门）的详细信息。
-	详细了解[对新数据库使用 Code First](https://msdn.microsoft.com/data/jj193542) 中介绍的本教程所用实体框架需要的方法。
-	详细了解 [Azure 中的 Web 应用](/home/features/web-site)。

-	了解 Azure Redis 缓存高级功能
	-	[如何为高级 Azure Redis 缓存配置暂留](/documentation/articles/cache-how-to-premium-persistence/)
	-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)
	-	[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)
	-	有关高级缓存大小、吞吐量和带宽的更多详细信息，请参阅 [Azure Redis 缓存常见问题](/documentation/articles/cache-faq/#what-redis-cache-offering-and-size-should-i-use)。


<!-- IMAGES -->
[cache-starter-application]: ./media/cache-web-app-howto/cache-starter-application.png
[cache-added-to-application]: ./media/cache-web-app-howto/cache-added-to-application.png
[cache-create-project]: ./media/cache-web-app-howto/cache-create-project.png
[cache-select-template]: ./media/cache-web-app-howto/cache-select-template.png
[cache-model-add-class]: ./media/cache-web-app-howto/cache-model-add-class.png
[cache-model-add-class-dialog]: ./media/cache-web-app-howto/cache-model-add-class-dialog.png
[cache-add-controller]: ./media/cache-web-app-howto/cache-add-controller.png
[cache-add-controller-class]: ./media/cache-web-app-howto/cache-add-controller-class.png
[cache-configure-controller]: ./media/cache-web-app-howto/cache-configure-controller.png
[cache-global-asax]: ./media/cache-web-app-howto/cache-global-asax.png
[cache-RouteConfig-cs]: ./media/cache-web-app-howto/cache-RouteConfig-cs.png
[cache-layout-cshtml]: ./media/cache-web-app-howto/cache-layout-cshtml.png
[cache-layout-cshtml-code]: ./media/cache-web-app-howto/cache-layout-cshtml-code.png
[redis-cache-manage-nuget-menu]: ./media/cache-web-app-howto/redis-cache-manage-nuget-menu.png
[redis-cache-stack-exchange-nuget]: ./media/cache-web-app-howto/redis-cache-stack-exchange-nuget.png
[cache-teamscontroller]: ./media/cache-web-app-howto/cache-teamscontroller.png
[cache-web-config]: ./media/cache-web-app-howto/cache-web-config.png
[cache-views-teams-index-cshtml]: ./media/cache-web-app-howto/cache-views-teams-index-cshtml.png
[cache-teams-index-table]: ./media/cache-web-app-howto/cache-teams-index-table.png
[cache-status-message]: ./media/cache-web-app-howto/cache-status-message.png
[deploybutton]: ./media/cache-web-app-howto/deploybutton.png
[cache-deploy-to-azure-step-1]: ./media/cache-web-app-howto/cache-deploy-to-azure-step-1.png
[cache-deploy-to-azure-step-2]: ./media/cache-web-app-howto/cache-deploy-to-azure-step-2.png
[cache-deploy-to-azure-step-3]: ./media/cache-web-app-howto/cache-deploy-to-azure-step-3.png
[cache-deployment-started]: ./media/cache-web-app-howto/cache-deployment-started.png
[cache-publish-app]: ./media/cache-web-app-howto/cache-publish-app.png
[cache-publish-to-app-service]: ./media/cache-web-app-howto/cache-publish-to-app-service.png
[cache-select-web-app]: ./media/cache-web-app-howto/cache-select-web-app.png
[cache-publish]: ./media/cache-web-app-howto/cache-publish.png
[cache-delete-resource-group]: ./media/cache-web-app-howto/cache-delete-resource-group.png
[cache-delete-confirm]: ./media/cache-web-app-howto/cache-delete-confirm.png


<!---HONumber=Mooncake_0523_2016-->