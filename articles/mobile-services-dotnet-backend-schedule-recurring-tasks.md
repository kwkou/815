<properties  pageTitle="Schedule Backend Tasks with Scheduler - Mobile Services" metaKeywords="" description="Use the Windows Azure Mobile Services Scheduler to schedule jobs for your mobile app." metaCanonical="" services="" documentationCenter="Mobile" title="Schedule recurring jobs in Mobile Services" authors=""  solutions="" writer="" manager="" editor=""  />

# 在移动服务中计划定期作业

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-schedule-recurring-tasks/" title=".NET backend" class="current">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-schedule-recurring-tasks/"  title="JavaScript backend" >JavaScript 后端</a>
</div>

本主题说明如何使用管理门户中的作业计划程序功能来定义服务器脚本代码，该代码将基于你定义的计划执行。在此情况下，脚本将定期检查远程服务（在本主题中为 Twitter），并在新表中存储结果。可以计划的其他一些定期任务包括：

-   存档旧的或重复的数据记录。
-   请求和存储外部数据，例如推文、RSS 条目和地点信息。
-   处理存储的图像或调整其大小。

本教程将引导你完成以下步骤，以使用作业计划程序创建一个计划的作业，该作业将从 Twitter 请求推文数据，并在新的 Updates 表中存储推文：

-   [注册以获取 Twitter 访问权限并存储凭据][]
-   [下载并安装 LINQ to Twitter 库][]
-   [创建新的 Updates 表][]
-   [创建新的计划作业][]
-   [在本地测试计划的作业][]
-   [发布服务并注册作业][]

> [WACOM.NOTE]本教程使用第三方 LINQ to Twitter 库来简化对 Twitter v1.1 API 进行的 OAuth 2.0 访问。若要完成本教程，你必须下载并安装 LINQ to Twitter NuGet 包。有关详细信息，请参阅 [LINQ to Twitter CodePlex 项目][]。

<a name="get-oauth-credentials"></a>
## 注册以获取 Twitter v1.1 API 访问权限并存储凭据

[WACOM.INCLUDE [mobile-services-register-twitter-access][]]

<ol start="7">
<li><p>在 Visual Studio 的解决方案资源管理器中，打开移动服务项目的 web.config 文件，找到 "MS\_TwitterConsumerKey" 和 "MS\_TwitterConsumerSecret" 应用程序设置，然后将这些密钥的值替换为你在门户中设置的 Twitter 使用者密钥值和使用者机密值。</p></li>

<li><p>在同一节中，添加以下新的应用程序设置，并将占位符替换为你在门户中设为应用程序设置的 Twitter 访问令牌值和访问令牌机密值：</p>

<pre><code>&lt;add key="TWITTER_ACCESS_TOKEN" value=""your_access_token"" /&gt;
&lt;add key="TWITTER_ACCESS_TOKEN_SECRET" value=""your_access_token_secret"" /&gt;</code></pre>

    <p>移动服务在本地计算机上运行时将使用这些存储的设置，让你在发布计划的作业之前对作业进行测试。在 Azure 中运行时，移动服务将改用门户中设置的值，并忽略这些项目设置。</p></li>
</ol>

<a name="install-linq2twitter"></a>
## 下载并安装 LINQ to Twitter 库

1.  在 Visual Studio 的“解决方案资源管理器” 中，右键单击项目名称，然后选择“管理 NuGet 包” 。

2.  在左窗格中，选择“联机” 类别，搜索 `linq2twitter`，在 "linqtotwitter" 程序包上单击“安装” ，然后阅读并接受许可协议。

    ![][]

    随后即会将 Linq to Twitter 库添加到你的移动服务项目。

接下来，你需要创建一个用于存储推文的新表。

<a name="create-table"></a>
## 创建新的 Updates 表

1.  在 Visual Studio 的解决方案资源管理器中，右键单击 DataObjects 文件夹，展开“添加”，单击“类”，在“名称”中键入 `Updates`，然后单击“添加” 。

    此时将为 Updates 类创建一个新的项目文件。

2.  右键单击“引用”，单击“添加引用...”，在“程序集”下选择“框架”，选中“System.ComponentModel.DataAnnotations”，然后单击“确定” 。

    ![][1]

    此时将会添加一个新的程序集引用。

3.  在此新类中添加以下 "using" 语句：

        using Microsoft.WindowsAzure.Mobile.Service;
        using System.ComponentModel.DataAnnotations;

4.  将 "Updates" 类定义替换为以下代码：

        public class Updates 
        {
        [Key]
        public int UpdateId { get; set; }
        public long TweetId { get; set; }
        public string Text { get; set; }
        public string Author { get; set; }
        public DateTime Date { get; set; }
        }

5.  展开 Models 文件夹，打开数据模型上下文文件（名为 *service\_name*Context.cs），并添加以下将返回类型化 "DbSet" 的属性：

        public DbSet<Updates> Updates { get; set; }

    服务使用首次访问 DbSet 时在数据库中创建的 Updates 表来存储推文数据。

    > [WACOM.NOTE] 使用默认数据库初始值设定项时，只要实体框架在代码优先模型定义中检测到数据模型更改，它就会删除并重新创建数据库。若要进行此数据模型更改并维护数据库中的现有数据，必须使用代码优先迁移。不能为 Azure 中的 SQL Database 使用默认的初始值设定项。有关详细信息，请参阅[如何使用代码优先迁移更新数据模型][]。

接下来，请创建计划的作业，用于访问 Twitter 并在新的 Updates 表中存储推文数据。

<a name="add-job"></a>
## 创建新的计划作业

1.  展开 ScheduledJobs 文件夹并打开 SampleJob.cs 项目文件。

    此类继承自 "ScheduledJob"，表示可在 Azure 管理门户中计划的、按固定计划或按需运行的作业。

2.  将 SampleJob.cs 的内容替换为以下代码：

         using System;
        using System.Linq;
        using System.Threading;
        using System.Threading.Tasks;
        using System.Web.Http;
        using Microsoft.WindowsAzure.Mobile.Service;
        using Microsoft.WindowsAzure.Mobile.Service.ScheduledJobs;
        using LinqToTwitter;
        using todolistService.Models;
        using todolistService.DataObjects;

        namespace todolistService
        {
        // A simple scheduled job which can be invoked manually by submitting an HTTP
        // POST request to the path "/jobs/sample".
        public class SampleJob :ScheduledJob
            {
        private todolistContext context;
        private string accessToken;
        private string accessTokenSecret;

        protected override void Initialize(ScheduledJobDescriptor scheduledJobDescriptor, CancellationToken cancellationToken)
                {
        base.Initialize(scheduledJobDescriptor, cancellationToken);

        // Create a new context with the supplied schema name.
        context = new todolistContext(Services.Settings.Name);
                }

        public async override Task ExecuteAsync()
                {            
        // Try to get the stored Twitter access token from app settings.  
        if (!(Services.Settings.TryGetValue("TWITTER_ACCESS_TOKEN", out accessToken) |
        Services.Settings.TryGetValue("TWITTER_ACCESS_TOKEN_SECRET", out accessTokenSecret)))
                    {
        Services.Log.Error("Could not retrieve Twitter access credentials.");
                    }

        // Create a new authorizer to access Twitter v1.1 APIs
        // using single-user OAUth 2.0 credentials.
        MvcAuthorizer auth = new MvcAuthorizer();
        SingleUserInMemoryCredentialStore store = 
        new SingleUserInMemoryCredentialStore()
                    {
        ConsumerKey = Services.Settings.TwitterConsumerKey,
        ConsumerSecret = Services.Settings.TwitterConsumerSecret,
        OAuthToken = accessToken,
        OAuthTokenSecret = accessTokenSecret
                    };

        // Set the credentials for the authorizer.
        auth.CredentialStore = store;

        // Create a new LINQ to Twitter context.
        TwitterContext twitter = new TwitterContext(auth);

        // Get the ID of the most recent stored tweet.
        long lastTweetId = 0;
        if (context.Updates.Count() > 0)
                    {
        lastTweetId = (from u in context.Updates
        orderby u.TweetId descending
        select u).Take(1).SingleOrDefault()
        .TweetId;
                    }

        // Execute a search that returns a filtered result.
        var response = await (from s in twitter.Search
        where s.Type == SearchType.Search
        && s.Query == "%23mobileservices"
        && s.SinceID == Convert.ToUInt64(lastTweetId + 1)
        && s.ResultType == ResultType.Recent
        select s).SingleOrDefaultAsync();

        // Remove retweets and replies and log the number of tweets.
        var filteredTweets = response.Statuses
        .Where(t => !t.Text.StartsWith("RT") && t.InReplyToUserID == 0);
        Services.Log.Info("Fetched " + filteredTweets.Count()
        + " new tweets from Twitter.");

        // Store new tweets in the Updates table.
        foreach (Status tweet in filteredTweets)
                    {
        Updates newTweet =
        new Updates
                            {
        TweetId = Convert.ToInt64(tweet.StatusID),
        Text = tweet.Text,
        Author = tweet.User.Name,
        Date = tweet.CreatedAt
                            };

        context.Updates.Add(newTweet);
                    }

        await context.SaveChangesAsync();
                }
        protected override void Dispose(bool disposing)
                {
        base.Dispose(disposing);
        if (disposing)
                    {
        context.Dispose();
                    }
                }
            }
        }

    在上述代码中，必须将字符串 *todolistService* 和 *todolistContext* 替换为已下载项目的命名空间和 DbContext（分别为 *mobile\_service\_name*Service 和 *mobile\_service\_name*Context）。

    在上述代码中，"ExecuteAsync" 重写方法使用存储的凭据调用 Twitter 查询 API，以请求包含哈希标记 `#mobileservices` 的最新推文。在表中存储结果之前，将从结果中删除重复的推文和回复。

<a name="run-job-locally"></a>
## 在本地测试计划的作业

将计划作业发布到 Azure 以及注册到门户之前，可以在本地对其进行测试。

1.  在 Visual Studio 中，如果已将移动服务项目设置为启动项目，请按 F5。

    此时将启动移动服务项目，并显示包含欢迎页的新浏览器窗口。

2.  单击“试用”，然后单击“POST 作业/{jobName}” 。

    ![][2]

3.  单击“试用此项”，键入 `Sample` 作为 "{jobName}" 参数值，然后单击“发送” 。

    ![][3]

    此时将向 Sample 作业终结点发送一个新的 POST 请求。在本地服务中，"ExecuteAsync" 方法已启动。你可以在此方法中设置一个断点来调试代码。

4.  在服务器资源管理器中，依次展开“数据连接”、“MSTableConnectionString”和“表”，右键单击“Updates”，然后单击“显示表数据” 。

    新推文现已作为行输入到数据表中。

<a name="register-job"></a>
## 发布服务并注册新作业

必须在“计划程序”选项卡中注册该作业，使移动服务能够根据你定义的计划运行该作业 。

1.  将移动服务项目重新发布到 Azure。

2.  在 [Azure 管理门户][]中单击“移动服务”，然后单击你的应用程序。

    ![][4]

3.  单击“计划程序”选项卡，然后单击“+创建” 。

    ![][5]

    > [WACOM.NOTE]如果在*免费*版本级别中运行你的移动服务，你一次只能运行一个计划的作业。在付费版本级别中，一次最多可以运行 10 个计划的作业。

4.  在计划程序对话框中，为“作业名称”输入 *SampleJob*，设置计划间隔和单位，然后单击勾选按钮 。

    ![][6]

    此时将创建一个名为 "SampleJob" 的新作业。

5.  单击刚刚创建的新作业，然后单击“运行一次”以测试脚本 。

    ![][7]

    此时将会执行该作业，不过它在计划程序中保持为禁用状态。你随时可以通过此页启用该作业及更改其计划。

    > [WACOM.NOTE]仍可使用 POST 请求来启动计划的作业。但是，系统默认向用户授权，也就是说，该请求的标头中必须包含应用程序密钥。

6.  （可选）在 [Azure 管理门户][]中，单击与你的移动服务关联的数据库对应的“管理”。

    ![][8]

7.  在管理门户中，执行一个查询以查看应用程序所做的更改。你的查询应类似于以下查询，不过，请使用你的移动服务名称作为架构名称，而不要使用 `todolist`。

        SELECT * FROM [todolist].[Updates]

祝贺你！你已成功地在移动服务中创建了一个新的计划作业。在你禁用或修改此作业之前，它将会按计划执行。

  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-schedule-recurring-tasks/ ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-schedule-recurring-tasks/ "JavaScript 后端"
  [注册以获取 Twitter 访问权限并存储凭据]: #get-oauth-credentials
  [下载并安装 LINQ to Twitter 库]: #install-linq2twitter
  [创建新的 Updates 表]: #create-table
  [创建新的计划作业]: #add-job
  [在本地测试计划的作业]: #run-job-locally
  [发布服务并注册作业]: #register-job
  [LINQ to Twitter CodePlex 项目]: http://linqtotwitter.codeplex.com/
  [mobile-services-register-twitter-access]: ../includes/mobile-services-register-twitter-access.md
  []: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/add-linq2twitter-nuget-package.png
  [1]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/add-component-model-reference.png
  [如何使用代码优先迁移更新数据模型]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-use-code-first-migrations
  [2]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/mobile-service-start-page.png
  [3]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/mobile-service-try-this-out.png
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [4]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/mobile-services-selection.png
  [5]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/mobile-schedule-new-job-cli.png
  [6]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/create-new-job.png
  [7]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/sample-job-run-once.png
  [8]: ./media/mobile-services-dotnet-backend-schedule-recurring-tasks/manage-sql-azure-database.png
