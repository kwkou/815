<properties
	pageTitle="创建具有 .NET 后端的 Windows 应用商店排行榜应用 | Azure 移动服务"
	description="了解如何使用 Azure 移动服务构建具有 .NET 后端的 Windows 应用商店排行榜应用。"
	documentationCenter="windows"
	authors="rmcmurray"
	manager="wpickett"
	editor=""
	services="mobile-services"/>

<tags 
	ms.service="mobile-services" 
	ms.date="05/04/2016"
	wacn.date="06/13/2016"/>

# 使用 Azure 移动服务 .NET 后端创建排行榜应用程序

本教程将说明如何使用具有 .NET 后端的 Azure 移动服务生成 Windows 应用商店应用程序。Azure 移动服务提供可缩放且安全的后端，具有内置身份验证、监视、推送通知和其他功能，以及用来生成移动应用程序的跨平台客户端库。移动服务的 .NET 后端基于 [ASP.NET Web API](http://asp.net/web-api)，可为 .NET 开发人员提供创建 REST API 的绝佳途径。

## 概述

Web API 是一个开源框架，可为 .NET 开发人员提供创建 REST API 的绝佳途径。你可以在 Azure 网站或使用 .NET 后端的 Azure 移动服务上托管 Web API 解决方案，甚至以自定义过程自我托管解决方案。移动服务是专门为移动应用程序设计的托管环境。当你在移动服务上托管 Web API 服务时，除了数据存储以外，还可以获得以下好处：

- 使用社交服务提供商和 Azure Active Directory (AAD) 的内置身份验证。 
- 使用设备特定的通知服务将通知推送到应用程序。
- 一整套可让你从任何应用程序轻松访问服务的客户端库。 
- 内置的日志记录和诊断。

在本教程中你将：

- 使用 Azure 移动服务创建 REST API。
- 将服务发布到 Azure。
- 创建使用服务的 Windows 应用商店应用程序。
- 使用 Entity Framework (EF) 创建外键关系和数据传输对象 (DTO)。
- 使用 ASP.NET Web API 定义自定义 API。

本教程使用 [Visual Studio 2013 最新更新版](http://go.microsoft.com/fwlink/p/?LinkID=390465)。


## 关于示例应用程序

排行榜显示游戏的玩家列表，以及每个玩家的分数和排名。排行榜可作为较大游戏的一部分，或作为单独的应用程序。排行榜是实际的应用程序，但简单易懂且可用于教程。下面是该应用程序的屏幕截图：

![][1]

为了方便说明此应用程序，其中不含任何实际的游戏。你可以添加玩家，以及提交每个玩家的分数。当你提交分数时，移动服务会计算新排名。在后端上，移动服务会创建具有两个表的数据库：

- Player。包含玩家 ID 和姓名。
- PlayerRank。包含玩家的分数和排名。

PlayerRank 具有 Player 的外键。每个玩家各有零个或一个 PlayerRank。

在实际的排行榜应用程序中，PlayerRank 还可能会有游戏 ID，让玩家提交多个游戏的分数。

![][2]

客户端应用程序可对 Player 执行一组完整的 CRUD 操作。它可读取或删除现有的 PlayerRank 实体，但无法直接加以创建或更新。这是因为排名值是由服务所计算的。实际做法是，在客户端提交分数后，由服务更新所有玩家的排名。

在[此处](http://code.msdn.microsoft.com/Leaderboard-App-with-Azure-9acf63af)下载完成的项目。


## 创建项目

启动 Visual Studio，并创建新的 ASP.NET Web 应用程序项目。将项目命名为 Leaderboard。

![][3]

在 Visual Studio 2013 中，ASP.NET Web 应用程序项目包含 Azure 移动服务的模板。请选择此模板，然后单击“确定”。

![][4]
 
项目模板包含示例控制器和数据对象。

![][5]
 
教程中并不需要这些项目，因此你可以将其从项目中删除。此外，请在 WebApiConfig.cs 和 LeaderboardContext.cs 中删除对 TodoItem 的引用。

## 添加数据模型

你将使用 [EF Code First](http://msdn.microsoft.com/zh-cn/data/ee712907#codefirst) 来定义数据库表。在 DataObjects 文件夹下，添加名为 `Player` 的类。

	using Microsoft.WindowsAzure.Mobile.Service;
	
	namespace Leaderboard.DataObjects
	{
	    public class Player : EntityData
	    {
	        public string Name { get; set; }
	    }
	}

添加名为 `PlayerRank` 的另一个类。

	using Microsoft.WindowsAzure.Mobile.Service;
	using System.ComponentModel.DataAnnotations.Schema;
	
	namespace Leaderboard.DataObjects
	{
	    public class PlayerRank : EntityData
	    {
	        public int Score { get; set; }
	        public int Rank { get; set; }
	
	        [ForeignKey("Id")]
	        public virtual Player Player { get; set; }
	    }
	}

请注意，这两个类都继承自 **EntityData** 类。派生自 **EntityData** 可方便应用程序使用数据，并将跨平台客户端库用于 Azure 移动服务。**EntityData** 还可方便应用程序[处理数据库写入冲突](/documentation/articles/mobile-services-windows-store-dotnet-handle-database-conflicts/)。

`PlayerRank` 类具有指向相关 `Player` 实体的[导航属性](http://msdn.microsoft.com/zh-cn/data/jj713564.aspx)。**[ForeignKey]** 属性让 EF 知道 `Player` 属性表示外键。

## 添加 Web API 控制器

接下来，你要为 `Player` 和 `PlayerRank` 添加 Web API 控制器。要添加的并不是普通 Web API 控制器，而是专门针对 Azure 移动服务设计的名为*表控制器*的特殊控制器。

右键单击 Controllers 文件夹，选择“添加”，然后选择“新建基架项”。

![][6]

在“添加基架”对话框中，展开左侧的“通用”，然后选择“Azure 移动服务”。接下来，选择“Azure 移动服务表控制器”。单击**“添加”**。

![][7]
 
在“添加控制器”对话框中：

1.	在“模型类”下，选择“Player”。 
2.	在“数据上下文类”下，选择“MobileServiceContext”。
3.	将控制器命名为“PlayerController”。
4.	单击**“添加”**。


此步骤将名为 PlayerController.cs 的文件添加到项目中。

![][8]

该控制器派生自 **TableController<T>**。此类继承 **ApiController**，但它是专用于 Azure 移动服务的类。
 
- 路由：**TableController** 的默认路径为 `/tables/{table_name}/{id}`，其中，*table\_name* 与实体名称匹配。因此，Player 控制器的路由为 */tables/player/{id}*。这种路由约定使得 **TableController** 与移动服务 [REST API](http://msdn.microsoft.com/zh-cn/library/azure/jj710104.aspx) 相一致。
- 数据访问：对于数据库操作，**TableController** 类使用 **IDomainManager** 接口，该接口定义数据访问的抽象。基架使用 **EntityDomainManager**，这是包装 EF 上下文的 **IDomainManager** 的具体实现。 

现在，请为 PlayerRank 实体添加第二个控制器。请遵循相同的步骤，但选择 PlayerRank 作为模型类。请使用相同的数据上下文类，而不要创建新类。将控制器命名为“PlayerRankController”。

## 使用 DTO 返回相关实体

回想一下，`PlayerRank` 具有相关的 `Player` 实体：

    public class PlayerRank : EntityData
    {
        public int Score { get; set; }
        public int Rank { get; set; }

        [ForeignKey("Id")]
        public virtual Player Player { get; set; }
    }

移动服务客户端库不支持导航属性，并且这些属性将不序列化。例如，下面是 GET `/tables/PlayerRank` 的原始 HTTP 响应：

	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Pragma: no-cache
	Content-Length: 97
	Content-Type: application/json; charset=utf-8
	Expires: 0
	Server: Microsoft-IIS/8.0
	Date: Mon, 21 Apr 2014 17:58:43 GMT
	
	[{"id":"1","rank":1,"score":150},{"id":"2","rank":3,"score":100},{"id":"3","rank":1,"score":150}]

请注意，`Player` 并未包含在对象图形中。若要包含玩家，可以通过定义*数据传输对象 (DTO)* 将对象图形平面化。

DTO 是定义如何通过网络发送数据的对象。如果你希望有线格式看起来与数据库模型不同，即可使用 DTO。若要为 `PlayerRank` 创建 DTO，请在 DataObjects 文件夹中添加名为 `PlayerRankDto` 的新类。

	namespace Leaderboard.DataObjects
	{
	    public class PlayerRankDto
	    {
	        public string Id { get; set; }
	        public string PlayerName { get; set; }
	        public int Score { get; set; }
	        public int Rank { get; set; }
	    }
	}

在 `PlayerRankController` 类中，我们将使用 LINQ **Select** 方法，将 `PlayerRank` 实例转换为 `PlayerRankDto` 实例。按以下方式更新 `GetAllPlayerRank` 和 `GetPlayerRank` 控制器方法：

	// GET tables/PlayerRank
	public IQueryable<PlayerRankDto> GetAllPlayerRank()
	{
	    return Query().Select(x => new PlayerRankDto()
	    {
	        Id = x.Id,
	        PlayerName = x.Player.Name,
	        Score = x.Score,
	        Rank = x.Rank
	    });
	}
	
	// GET tables/PlayerRank/48D68C86-6EA6-4C25-AA33-223FC9A27959
	public SingleResult<PlayerRankDto> GetPlayerRank(string id)
	{
	    var result = Lookup(id).Queryable.Select(x => new PlayerRankDto()
	    {
	        Id = x.Id,
	        PlayerName = x.Player.Name,
	        Score = x.Score,
	        Rank = x.Rank
	    });
	
	    return SingleResult<PlayerRankDto>.Create(result);
	}

做出这些更改后，两个 GET 方法将 `PlayerRankDto` 对象返回到客户端。`PlayerRankDto.PlayerName` 属性设置为玩家姓名。以下是做出此更改后的示例响应：

	HTTP/1.1 200 OK
	Cache-Control: no-cache
	Pragma: no-cache
	Content-Length: 160
	Content-Type: application/json; charset=utf-8
	Expires: 0
	Server: Microsoft-IIS/8.0
	Date: Mon, 21 Apr 2014 19:57:08 GMT
	
	[{"id":"1","playerName":"Alice","score":150,"rank":1},{"id":"2","playerName":"Bob","score":100,"rank":3},{"id":"3","playerName":"Charles","score":150,"rank":1}]

请注意 JSON 负载现在包含玩家姓名。

除了使用 LINQ Select 语句以外，另一种做法是使用 AutoMapper。这种做法需要使用其他设置代码，但可启用从域实体到 DTO 的自动映射。有关详细信息，请参阅[使用 AutoMapper 在 .NET 后端中的数据库类型与客户端类型之间映射](http://blogs.msdn.com/b/azuremobile/archive/2014/05/19/mapping-between-database-types-and-client-type-in-the-net-backend-using-automapper.aspx)。

## 定义自定义 API 来提交分数

`PlayerRank` 实体包含 `Rank` 属性。此值由服务器计算，而我们不希望客户端设置它。客户端应使用自定义 API 提交玩家的分数。当服务器获取新分数时，将更新所有的玩家排名。

首先，将名为 `PlayerScore` 的类添加到 DataObjects 文件夹。

	namespace Leaderboard.DataObjects
	{
	    public class PlayerScore
	    {
	        public string PlayerId { get; set; }
	        public int Score { get; set; }
	    }
	}

在 `PlayerRankController` 类中，将 `MobileServiceContext` 变量从构造函数移到类变量：

    public class PlayerRankController : TableController<PlayerRank>
    {
        // Add this:
        MobileServiceContext context = new MobileServiceContext();

        protected override void Initialize(HttpControllerContext controllerContext)
        {
            base.Initialize(controllerContext);

            // Delete this:
            // MobileServiceContext context = new MobileServiceContext();
            DomainManager = new EntityDomainManager<PlayerRank>(context, Request, Services);
        }


从 `PlayerRankController` 中删除以下方法：

- `PatchPlayerRank`
- `PostPlayerRank`
- `DeletePlayerRank`

然后，将以下代码添加到 `PlayerRankController`：

    [Route("api/score")]
    public async Task<IHttpActionResult> PostPlayerScore(PlayerScore score)
    {
        // Does this player exist?
        var count = context.Players.Where(x => x.Id == score.PlayerId).Count();
        if (count < 1)
        {
            return BadRequest();
        }

        // Try to find the PlayerRank entity for this player. If not found, create a new one.
        PlayerRank rank = await context.PlayerRanks.FindAsync(score.PlayerId);
        if (rank == null)
        {
            rank = new PlayerRank { Id = score.PlayerId };
            rank.Score = score.Score;
            context.PlayerRanks.Add(rank);
        }
        else
        {
            rank.Score = score.Score;
        }

        await context.SaveChangesAsync();

        // Update rankings
        // See http://stackoverflow.com/a/575799
        const string updateCommand =
            "UPDATE r SET Rank = ((SELECT COUNT(*)+1 from {0}.PlayerRanks " +
            "where Score > (select score from {0}.PlayerRanks where Id = r.Id)))" +
            "FROM {0}.PlayerRanks as r";

        string command = String.Format(updateCommand, ServiceSettingsDictionary.GetSchemaName());
        await context.Database.ExecuteSqlCommandAsync(command);

        return Ok();
    }

`PostPlayerScore` 方法采用 `PlayerScore` 实例作为输入。（客户端将在 HTTP POST 请求中发送 `PlayerScore`。） 该方法将执行以下操作：

1.	如果数据库中尚无玩家的 `PlayerRank`，则新增一个。
2.	更新玩家的分数。
3.	运行 SQL 查询，以分批更新所有玩家排名。

**[Route]** 属性为此方法定义一个自定义路由：

	[Route("api/score")]

也可以将方法放入单独的控制器中。没有哪种方法特别好，具体取决于你想要如何组织代码。
若要深入了解 **[Route]** 属性，请参阅 [Web API 中的属性路由](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)。

## 创建 Windows 应用商店应用程序

在本部分中，我将介绍使用移动服务的 Windows 应用商店应用程序。但是，我不会将重点放在 XAML 或 UI 上，而是着重于应用程序逻辑。你可以在[此处](http://code.msdn.microsoft.com/Leaderboard-App-with-Azure-9acf63af)下载完整项目。

将新的 Windows 应用商店应用程序项目添加到解决方案。我使用了空白应用程序 (Windows) 模板。

![][10]
 
使用 NuGet Package Manager 添加移动服务客户端库。在 Visual Studio 中，从“工具”菜单中选择“NuGet Package Manager”。然后选择“Package Manager Console”。在“Package Manager Console”窗口中键入以下命令。

	Install-Package WindowsAzure.MobileServices -Project LeaderboardApp

-Project 开关指定要将包安装到哪个项目。

## 添加模型类

创建名为 Models 的文件夹并添加以下类：

	namespace LeaderboardApp.Models
	{
	    public class Player
	    {
	        public string Id { get; set; }
	        public string Name { get; set; }
	    }
	
	    public class PlayerRank
	    {
	        public string Id { get; set; }
	        public string PlayerName { get; set; }
	        public int Score { get; set; }
	        public int Rank { get; set; }
	    }
	
	    public class PlayerScore
	    {
	        public string PlayerId { get; set; }
	        public int Score { get; set; }
	    }
	}

这些类直接对应于移动服务中的数据实体。
 
## 创建视图模型

模型-视图-视图模型 (MVVM) 是模型-视图-控制器 (MVC) 的变体。MVVM 模式有助于将应用程序逻辑与表示形式区分开来。

- 模型表示域数据（玩家、玩家排名和玩家分数）。
- 视图模型是视图的抽象表示形式。 
- 视图显示视图模型，并向视图模型发送用户输入。对于 Windows 应用商店应用程序，视图在 XAML 中定义。

![][11]

添加名为的 `LeaderboardViewModel` 的类。

	using LeaderboardApp.Models;
	using Microsoft.WindowsAzure.MobileServices;
	using System.ComponentModel;
	using System.Net.Http;
	using System.Threading.Tasks;
	
	namespace LeaderboardApp.ViewModel
	{
	    class LeaderboardViewModel : INotifyPropertyChanged
	    {
	        MobileServiceClient _client;
	
	        public LeaderboardViewModel(MobileServiceClient client)
	        {
	            _client = client;
	        }
	    }
	}

在视图模型上实现 **INotifyPropertyChanged**，使视图模型可以参与数据绑定。

    class LeaderboardViewModel : INotifyPropertyChanged
    {
        MobileServiceClient _client;

        public LeaderboardViewModel(MobileServiceClient client)
        {
            _client = client;
        }

        // New code:
        // INotifyPropertyChanged implementation
        public event PropertyChangedEventHandler PropertyChanged;

        public void NotifyPropertyChanged(string propertyName)
        {
            if (PropertyChanged != null)
            {
                PropertyChanged(this,
                    new PropertyChangedEventArgs(propertyName));
            }
        }    
    }

接下来，添加可查看属性。XAML 将与这些属性建立数据绑定。

    class LeaderboardViewModel : INotifyPropertyChanged
    {
        // ...

        // New code:
        // View model properties
        private MobileServiceCollection<Player, Player> _Players;
        public MobileServiceCollection<Player, Player> Players
        {
            get { return _Players; }
            set
            {
                _Players = value;
                NotifyPropertyChanged("Players");
            }
        }

        private MobileServiceCollection<PlayerRank, PlayerRank> _Ranks;
        public MobileServiceCollection<PlayerRank, PlayerRank> Ranks
        {
            get { return _Ranks; }
            set
            {
                _Ranks = value;
                NotifyPropertyChanged("Ranks");
            }
        }

        private bool _IsPending;
        public bool IsPending
        {
            get { return _IsPending; }
            set
            {
                _IsPending = value;
                NotifyPropertyChanged("IsPending");
            }
        }

        private string _ErrorMessage = null;
        public string ErrorMessage
        {
            get { return _ErrorMessage; }
            set
            {
                _ErrorMessage = value;
                NotifyPropertyChanged("ErrorMessage");
            }
        }
    }

当服务的异步操作挂起时，`IsPending` 属性为 true。`ErrorMessage` 属性包含来自服务的任何错误消息。

最后，添加调用服务层的方法。

    class LeaderboardViewModel : INotifyPropertyChanged
    {
        // ...

        // New code:
        // Service operations
        public async Task GetAllPlayersAsync()
        {
            IsPending = true;
            ErrorMessage = null;

            try
            {
                IMobileServiceTable<Player> table = _client.GetTable<Player>();
                Players = await table.OrderBy(x => x.Name).ToCollectionAsync();
            }
            catch (MobileServiceInvalidOperationException ex)
            {
                ErrorMessage = ex.Message;
            }
            catch (HttpRequestException ex2)
            {
                ErrorMessage = ex2.Message;
            }
            finally
            {
                IsPending = false;
            }
        }

        public async Task AddPlayerAsync(Player player)
        {
            IsPending = true;
            ErrorMessage = null;

            try
            {
                IMobileServiceTable<Player> table = _client.GetTable<Player>();
                await table.InsertAsync(player);
                Players.Add(player);
            }
            catch (MobileServiceInvalidOperationException ex)
            {
                ErrorMessage = ex.Message;
            }
            catch (HttpRequestException ex2)
            {
                ErrorMessage = ex2.Message;
            }
            finally
            {
                IsPending = false;
            }
        }

        public async Task SubmitScoreAsync(Player player, int score)
        {
            IsPending = true;
            ErrorMessage = null;

            var playerScore = new PlayerScore()
            {
                PlayerId = player.Id,
                Score = score
            }; 
            
            try
            {
                await _client.InvokeApiAsync<PlayerScore, object>("score", playerScore);
                await GetAllRanksAsync();
            }
            catch (MobileServiceInvalidOperationException ex)
            {
                ErrorMessage = ex.Message;
            }
            catch (HttpRequestException ex2)
            {
                ErrorMessage = ex2.Message;
            }
            finally
            {
                IsPending = false;
            }
        }

        public async Task GetAllRanksAsync()
        {
            IsPending = true;
            ErrorMessage = null;

            try
            {
                IMobileServiceTable<PlayerRank> table = _client.GetTable<PlayerRank>();
                Ranks = await table.OrderBy(x => x.Rank).ToCollectionAsync();
            }
            catch (MobileServiceInvalidOperationException ex)
            {
                ErrorMessage = ex.Message;
            }
            catch (HttpRequestException ex2)
            {
                ErrorMessage = ex2.Message;
            }
            finally
            {
                IsPending = false;
            }
         }    
    }

## 添加 MobileServiceClient 实例

打开 *App.xaml.cs* 文件并将 **MobileServiceClient** 实例添加到 `App` 类。

	// New code:
	using Microsoft.WindowsAzure.MobileServices;
	
	namespace LeaderboardApp
	{
	    sealed partial class App : Application
	    {
	        // New code.
	        // TODO: Replace 'port' with the actual port number.
	        const string serviceUrl = "http://localhost:port/";
	        public static MobileServiceClient MobileService = new MobileServiceClient(serviceUrl);
	
	
	        // ...
	    }
	}

当你在本地调试时，移动服务将在 IIS Express 上运行。Visual Studio 将分配一个随机端口号，因此本地 URL 为 http://localhost:*port*，其中 *port* 为端口号。若要获取端口号，请按 F5 在 Visual Studio 中启动服务，以进行调试。Visual Studio 将启动浏览器，并导航到服务 URL。你也可以在项目属性中的 **Web** 下查找本地 URL。

## 创建主页面

在主页面中，添加视图模型的实例。然后将视图模型设置为该页面的 **DataContext**。

    public sealed partial class MainPage : Page
    {
        // New code:
        LeaderboardViewModel viewModel = new LeaderboardViewModel(App.MobileService);

        public MainPage()
        {
            this.InitializeComponent();
            // New code:
            this.DataContext = viewModel;
        }

       // ...


前面已经提到，我不会介绍应用程序的所有 XAML。MVVM 模式的优点之一是能够区分表示形式和应用程序逻辑，因此，如果你不喜欢示例应用程序，可以轻松更改 UI。

玩家列表显示在 **ListBox** 中：

	<ListBox Width="200" Height="400" x:Name="PlayerListBox" 
	    ItemsSource="{Binding Players}" DisplayMemberPath="Name"/>

排名显示在 **ListView** 中：

	<ListView x:Name="RankingsListView" ItemsSource="{Binding Ranks}" SelectionMode="None">
	    <!-- Header and styles not shown -->
	    <ListView.ItemTemplate>
	        <DataTemplate>
	            <Grid>
	                <Grid.ColumnDefinitions>
	                    <ColumnDefinition Width="*"/>
	                    <ColumnDefinition Width="2*"/>
	                    <ColumnDefinition Width="*"/>
	                </Grid.ColumnDefinitions>
	                <TextBlock Text="{Binding Path=Rank}"/>
	                <TextBlock Text="{Binding Path=PlayerName}" Grid.Column="1"/>
	                <TextBlock Text="{Binding Path=Score}" Grid.Column="2"/>
	            </Grid>
	        </DataTemplate>
	    </ListView.ItemTemplate>
	</ListView>

所有数据绑定都通过视图模型发生。

## 发布移动服务

在此步骤中，你要将移动服务发布到 Azure，并修改应用程序以使用实时服务。

在“解决方案资源管理器”中，右键单击 Leaderboard 项目并选择“发布”。
 
![][12]

在“发布”对话框中，单击“Azure 移动服务”。

![][13]
 
如果你尚未登录你的 Azure 帐户，请单击“登录”。

![][14]


选择现有的移动服务，或单击“新建”以创建一个新的服务。然后单击“确定”以发布。

![][15]
 
发布过程会自动创建数据库。你不需要配置连接字符串。

现在，你可以将排行榜应用程序连接到实时服务了。你需要以下两项：

- 服务的 URL
- 应用程序密钥

你可以从 Azure 经典门户获取这两项信息。在门户中单击“移动服务”，然后单击该移动服务。仪表板选项卡上列出了服务 URL。若要获取应用程序密钥，请单击“管理密钥”。

![][16]
 
在“管理访问密钥”对话框中，复制应用程序密钥值。

![][17]

 
将服务 URL 和应用程序密钥传递给 **MobileServiceClient** 构造函数。

    sealed partial class App : Application
    {
        // TODO: Replace these strings with the real URL and key.
        const string serviceUrl = "https://yourapp.azure-mobile.net/";
        const string appKey = "YOUR ACCESSS KEY";

        public static MobileServiceClient MobileService = new MobileServiceClient(serviceUrl, appKey);

       // ...

现在，当你运行该应用程序时，它将与实际的服务通信。

## 后续步骤

* [详细了解 Azure 移动服务]
* [详细了解 Web API]
* [处理数据库写入冲突]
* [添加推送通知]（例如，当某人添加新玩家或更新分数时）。
* [身份验证入门]

<!-- Anchors. -->
[Overview]: #overview
[About the sample app]: #about-the-sample-app
[Create the project]: #create-the-project
[Add data models]: #add-data-models
[Add Web API controllers]: #add-web-api-controllers
[Use a DTO to return related entities]: #use-a-dto-to-return-related-entities
[Define a custom API to submit scores]: #define-a-custom-api-to-submit-scores
[Create the Windows Store app]: #create-the-windows-store-app
[Add model classes]: #add-model-classes
[Create a view model]: #create-a-view-model
[Add a MobileServiceClient instance]: #add-a-mobileserviceclient-instance
[Create the main page]: #create-the-main-page
[Publish your mobile service]: #publish-your-mobile-service
[Next Steps]: #next-steps

<!-- Images. -->

[1]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/01leaderboard.png
[2]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/02leaderboard.png
[3]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/03leaderboard.png
[4]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/04leaderboard.png
[5]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/05leaderboard.png
[6]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/06leaderboard.png
[7]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/07leaderboard.png
[8]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/08leaderboard.png
[9]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/09leaderboard.png
[10]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/10leaderboard.png
[11]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/11leaderboard.png
[12]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/12leaderboard.png
[13]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/13leaderboard.png
[14]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/14leaderboard.png
[15]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/15leaderboard.png
[16]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/16leaderboard.png
[17]: ./media/mobile-services-dotnet-backend-windows-store-dotnet-leaderboard/17leaderboard.png

<!-- URLs. -->

[详细了解 Azure 移动服务]: /documentation/services/mobile-services/
[详细了解 Web API]: http://asp.net/web-api
[处理数据库写入冲突]: /documentation/articles/mobile-services-windows-store-dotnet-handle-database-conflicts/
[添加推送通知]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[身份验证入门]: /documentation/articles/mobile-services-dotnet-backend-windows-universal-dotnet-get-started-users/

<!---HONumber=Mooncake_0215_2016-->