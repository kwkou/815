<properties linkid="develop-net-tutorials-multi-tier-web-site-3-web-role" pageTitle="Azure 云服务教程：ASP.NET Web 角色和 Azure 存储表、队列、Blob" metaKeywords="Azure tutorial, Azure storage tutorial, Azure multi-tier tutorial, ASP.NET MVC tutorial, Azure web role tutorial, Azure blobs tutorial, Azure tables tutorial, Azure queues tutorial" description="了解如何使用 ASP.NET MVC 和 Azure 创建多层应用程序。该应用程序运行在云服务中，使用 Web 角色和辅助角色，并使用 Azure 存储表、队列和 Blob。" metaCanonical="" services="cloud-services,storage" documentationCenter=".NET" title="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" authors="tdykstra,riande" solutions="" manager="wpickett" editor="mollybos" />

# 构建 Azure 电子邮件服务应用程序的 Web 角色 - 第 3 部分（共 5 部分）

这是分为五部分的一系列教程的第三个教程，这些教程讲述如何生成和部署 Azure 电子邮件服务示例应用程序。有关此应用程序和教程系列的信息，请参阅[本系列第一个教程][本系列第一个教程]。

在本教程中，你将学习：

-   如何创建一个包含云服务项目的解决方案，该项目带有一个 Web 角色和一个辅助角色。
-   如何在 MVC 4 控制器和视图中使用 Azure 表、Blob 和队列。
-   如何在使用 Azure 表时处理并发冲突。
-   如何配置 Web 角色或 Web 项目以使用 Azure 存储帐户。

## 本教程的章节

-   [创建 Visual Studio 解决方案][创建 Visual Studio 解决方案]
-   [配置跟踪][配置跟踪]
-   [添加代码来有效地处理重新启动。][添加代码来有效地处理重新启动。]
-   [更新存储客户端库 NuGet 包][更新存储客户端库 NuGet 包]
-   [添加对 SCL 1.7 程序集的引用][添加对 SCL 1.7 程序集的引用]
-   [添加可在 Application\_Start 方法中创建表、队列和 Blob 容器的代码][添加可在 Application\_Start 方法中创建表、队列和 Blob 容器的代码]
-   [创建和测试邮件列表][创建和测试邮件列表]
-   [配置 Web 角色以使用你的 Azure 存储测试帐户][配置 Web 角色以使用你的 Azure 存储测试帐户]
-   [创建和测试“订户”控制器和视图][创建和测试“订户”控制器和视图]
-   [创建和测试“邮件”控制器和视图][创建和测试“邮件”控制器和视图]
-   [创建和测试“取消订阅”控制器和视图][创建和测试“取消订阅”控制器和视图]
-   [（可选）构建替代体系结构][（可选）构建替代体系结构]
-   [后续步骤][后续步骤]

## <a name="cloudproject"></a><span class="short-header">创建解决方案</span>创建 Visual Studio 解决方案

你首先创建一个 Visual Studio 解决方案，其中包含一个针对 Web 前端的项目，以及一个针对后端 Azure 辅助角色之一的项目。随后你将添加第二个辅助角色。

（如果你想要在 Azure 网站而不是 Azure 云服务中运行 Web UI，请参阅本教程后面的[替代体系结构][（可选）构建替代体系结构]部分，了解对这些说明所做的更改。）

### 使用 Web 角色和辅助角色创建一个云服务项目

1.  使用提升的权限启动 Visual Studio。

    > [WACOM.NOTE] 对于 Visual Studio 2013，你不一定要使用提升的权限，因为新项目默认使用计算模拟器速成版。

2.  在“文件”菜单中，选择“新建项目”。

    ![“新建项目”菜单][“新建项目”菜单]

3.  展开 **C#**，选择**“已安装模板”**下的“云”，然后选择**“Azure 云服务”**。

4.  将该应用程序命名为 **AzureEmailService**，然后单击“确定”。

    ![“新建项目”对话框][“新建项目”对话框]

5.  在**“新建 Azure 云服务”**对话框中，选择**“ASP.NET Web 角色”**，然后单击向右箭头。

    > [WACOM.NOTE] 已下载的用于本教程的代码为 MVC 4，但你不能使用针对 Visual Studio 2012 撰写的说明在 Visual Studio 2013 中创建 MVC 4 Web 角色。对于 Visual Studio 2013，请执行以下操作：(1) 跳过此处用于创建 Web 角色的步骤，执行辅助角色的步骤。(2) 创建辅助角色后，右键单击**解决方案资源管理器**中的解决方案，然后单击**“添加”**--**“新建项目”**。在**“添加新项目”**对话框的左窗格中展开**“Web”**，然后选择**“Visual Studio 2012”**。(3) 选择**“ASP.NET MVC 4 Web 应用程序”**，将该项目命名为 **MvcWebRole**，然后单击**“确定”**。(4) 在“新建 ASP.NET 项目”对话框中，选择“Internet 应用程序”模板。(5) 右键单击**解决方案资源管理器**中 **AzureEmailService** 下的**“角色”**，然后单击**“添加”**-**“解决方案中的 Web 角色项目”**。(6) 在**“与角色项目关联”**框中，选择 **MvcWebRole** 项目，然后单击**“确定”**。

    ![“新建 Azure 云项目”对话框][“新建 Azure 云项目”对话框]

6.  在右侧列中，将鼠标指针悬停在 **MvcWebRole1** 上，然后单击铅笔图标，以便更改 Web 角色的名称。

7.  输入 MvcWebRole 作为新名称，然后按 Enter。

    ![“新建 Azure 云项目”对话框 - 重命名 Web 角色][“新建 Azure 云项目”对话框 - 重命名 Web 角色]

8.  请按照相同的过程来添加**辅助角色**，将其命名为“WorkerRoleA”，然后单击**“确定”**。

    ![“新建 Azure 云项目”对话框 - 添加辅助角色][“新建 Azure 云项目”对话框 - 添加辅助角色]

9.  在“新建 ASP.NET 项目”对话框中，选择“Internet 应用程序”模板。

10. 在**“视图引擎”**下拉列表中，确保已选中**“Razor”**，然后单击**“确定”**。

    ![“新建项目”对话框][1]

### 设置页眉、菜单和页脚

在本部分中，你将更新显示在管理员 Web UI 每一页上的页眉、页脚和菜单项。该应用程序将提供三组管理员网页：一组用于邮件列表，一组用于邮件列表的订户，一组用于邮件。

1.  在**解决方案资源管理器**中，展开 Views\\Shared 文件夹并打开 \_Layout.cshtml 文件。

    ![解决方案资源管理器中的 \_Layout.cshtml][解决方案资源管理器中的 \_Layout.cshtml]

2.  在 **\<title\>** 元素中，将“我的 ASP.NET MVC 应用程序”更改为“Azure 电子邮件服务”。

3.  在类为“site-title”的 **\<p\>** 元素中，将“在此处显示你的徽标”更改为“Azure 电子邮件服务”，将“主页”更改为“MailingList”。

    ![\_Layout.cshtml 中的标题和页眉][\_Layout.cshtml 中的标题和页眉]

4.  删除菜单部分：

    ![\_Layout.cshtml 中的菜单][\_Layout.cshtml 中的菜单]

5.  在旧菜单所在位置插入新的菜单部分：

        <ul id="menu">
            <li>@Html.ActionLink("Mailing Lists", "Index", "MailingList")</li>
            <li>@Html.ActionLink("Messages", "Index", "Message")</li>
            <li>@Html.ActionLink("Subscribers", "Index", "Subscriber")</li>
        </ul>

6.  在 **\<footer\>** 元素中，将“我的 ASP.NET MVC 应用程序”更改为“Azure 电子邮件服务”。

    ![\_Layout.cshtml 中的页脚][\_Layout.cshtml 中的页脚]

### 在本地运行应用程序

1.  按 Ctrl+F5 运行应用程序。

    随后在默认浏览器中显示该应用程序主页。

    ![主页][主页]

    应用程序在 Azure 计算模拟器中运行。你可以在 Windows 系统任务栏中看到计算模拟器图标：

    ![系统任务栏中的计算模拟器][系统任务栏中的计算模拟器]

## <a name="tracing"></a><span class="short-header">配置跟踪</span>配置跟踪

若要保存跟踪数据，请打开 *WebRole.cs* 文件，添加以下 `ConfigureDiagnostics` 方法。添加可在 `OnStart` 方法中调用新方法的代码。

> [WACOM.NOTE] 对于 Visual Studio 2013，请右键单击 MvcWebRole 项目，单击**“添加现有项”**，然后添加已下载项目中的 *WebRole.cs* 文件，以替代下列步骤（在 *WebRole.cs* 中手动更改代码）。

    private void ConfigureDiagnostics()
    {
        DiagnosticMonitorConfiguration config = DiagnosticMonitor.GetDefaultInitialConfiguration();
        config.Logs.BufferQuotaInMB = 500;
        config.Logs.ScheduledTransferLogLevelFilter = LogLevel.Verbose;
        config.Logs.ScheduledTransferPeriod = TimeSpan.FromMinutes(1d);

        DiagnosticMonitor.Start(
            "Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString",
            config);
    }

    public override bool OnStart()
    {
        ConfigureDiagnostics();
        return base.OnStart();
    }

`ConfigureDiagnostics` 方法在[第二个教程][第二个教程]中介绍。

## <a name="restarts"></a><span class="short-header">重新启动</span>添加代码来有效地处理重新启动。

Azure 云服务应用程序大约每月重新启动两次，以便进行操作系统更新。（有关 OS 更新的详细信息，请参阅[重新启动角色实例进行 OS 升级][重新启动角色实例进行 OS 升级]。）当 Web 应用程序将要关闭时，将引发 `OnStop` 事件。Visual Studio 创建的 Web 角色样板不覆盖 `OnStop` 方法，因此应用程序只有几秒钟的时间在关闭之前完成 HTTP 请求的处理操作。你可以添加代码来覆盖 `OnStop` 方法，以确保正常处理应用程序的关闭操作。

若要处理关闭和重新启动操作，请打开 *WebRole.cs* 文件，添加以下 `OnStop` 方法进行覆盖。

      public override void OnStop()
      {
          Trace.TraceInformation("OnStop called from WebRole");
          var rcCounter = new PerformanceCounter("ASP.NET", "Requests Current", "");
          while (rcCounter.NextValue() > 0)
          {
              Trace.TraceInformation("ASP.NET Requests Current = " + rcCounter.NextValue().ToString());
              System.Threading.Thread.Sleep(1000);
          }           
      }

此代码需要一个附加的 `using` 语句：

      using System.Diagnostics;

在应用程序关闭之前，`OnStop` 方法有长达 5 分钟的退出时间。你可以针对 `OnStop` 方法添加一个 5 分钟的睡眠调用，以便为你的应用程序留出最长的时间来处理当前请求，但如果你的应用程序缩放得当，则应能够在远少于 5 分钟的时间内处理剩下的请求。最好是能够尽快停止，这样应用程序就能够尽快重新启动，然后继续处理请求。

在 Azure 迫使某个角色脱机以后，负载平衡器停止向角色实例发送请求，然后就会调用 `OnStop` 方法。如果你没有角色的另一个实例，则在你的角色完成关闭并重新启动（这通常需要几分钟）之前，不会处理任何请求。这就是 Azure 服务级别协议要求你的每个角色至少有两个实例，以便充分利用正常运行时间保证的一个原因。

在针对 `OnStop` 方法演示的代码中，为 `Requests Current` 创建了 ASP.NET 性能计算器。`Requests Current` 计算器值包含当前的请求数，其中包括排队的请求数、当前正在执行的请求数或等待写入客户端的请求数。每秒钟都会检查 `Requests Current` 值，当该值降到零时，将返回 `OnStop` 方法。一旦 `OnStop` 返回，角色就会关闭。

从 `OnStop` 方法调用而不执行[按需传输][按需传输]时，不会保存跟踪数据。你可以使用 [dbgview][dbgview] 实用程序通过远程桌面连接实时查看 `OnStop` 跟踪信息。

## <a name="updatescl"></a><span class="short-header">更新存储客户端库</span>更新存储客户端库 NuGet 包

> [WACOM.NOTE] 此步骤可能不是必需的。如果 Azure 存储 NuGet 包未显示在“更新”列表中，则已安装的版本是最新的。

与 Azure 存储表、队列和 Blob 一起使用的 API 框架为存储客户端库 (SCL)。此 API 包含在云服务项目模板的 NuGet 包中。不过，在撰写本教程的时候，项目模板包含的是 1.7 版的 SCL，而不是现在的 2.0 版。因此，在开始编写代码之前，你需要更新 NuGet 包。

1.  在 Visual Studio 的**“工具”**菜单上，将鼠标悬停在**“库程序包管理器”**上，然后单击**“管理解决方案的 NuGet 包”**。

    ![菜单中的“管理解决方案的 NuGet 包”][菜单中的“管理解决方案的 NuGet 包”]

2.  在**“管理 NuGet 包”**对话框的左窗格中，选择**“更新”**，然后向下滚动到**“Azure 存储”**包，再单击**“更新”**。

    ![“管理 NuGet 包”对话框中的 Azure 存储包][“管理 NuGet 包”对话框中的 Azure 存储包]

3.  在**“选择项目”**对话框中，请确保两个项目都已选中，然后单击**“确定”**。

    ![在“选择项目”对话框中选择这两个项目][在“选择项目”对话框中选择这两个项目]

4.  接受许可条款以完成包的安装，然后关闭**“管理 NuGet 包”**对话框。

5.  在 WorkerRoleA 项目的 *WorkerRole.cs* 中，如果存在以下 `using` 语句，请将其删除，因为不再需要该语句：

        using Microsoft.WindowsAzure.StorageClient;

1.7 版的 SCL 包括 LINQ 提供程序，简化了表查询的代码编写。在撰写本教程时，2.0 版的表服务层 (TSL) 还没有 LINQ 提供程序。如果你想要使用 LINQ，你仍然可以访问 [Microsoft.WindowsAzure.Storage.Table.DataServices][Microsoft.WindowsAzure.Storage.Table.DataServices] 命名空间中的 SCL 1.7 LINQ 提供程序。2.0 TSL 旨在改进性能，而 1.7 LINQ 提供程序并不能利用所有这些改进。示例应用程序使用 2.0 TSL，因此它不使用 LINQ 进行查询。有关 SCL 和 TSL 2.0 的详细信息，请参阅[本系列最后一个教程][本系列最后一个教程]结尾处的资源。

> [WACOM.NOTE] 存储客户端库 2.1 添加了后端 LINQ 支持，不过本教程不使用 LINQ 进行存储表查询。当前的 SCL 还支持异步编程，但本教程中不显示异步代码。有关异步编程以及对 Azure SCL 使用异步编程的示例代码的详细信息，请参阅下面的电子书籍章节及其随附的可下载项目：[使用.NET 4.5 的异步支持以避免阻塞调用][使用.NET 4.5 的异步支持以避免阻塞调用]。

## <a name="addref2"></a><span class="short-header">添加 SCL 1.7 引用</span>添加对 SCL 1.7 程序集的引用

> [WACOM.NOTE] 如果你使用的是 SCL 2.1 或更高版本（带有最新 SDK 的 Visual Studio 2013），请跳过本部分。

存储客户端库 (SCL) 2.0 版并没有进行诊断所需的所有内容，因此你必须添加对 1.7 版程序集的引用。

1.  右键单击 MvcWebRole 项目，然后选择**“添加引用”**。

2.  单击对话框底部的**“浏览...”**按钮。

3.  导航到以下文件夹：

        C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK012-10\ref

4.  选择 *Microsoft.WindowsAzure.StorageClient.dll*，然后单击**“添加”**。

5.  在**“引用管理器”**对话框中，单击**“确定”**。

6.  对 WorkerRoleA 项目重复此过程。

## <a name="createifnotexists"></a><span class="short-header">App\_Start 代码</span>添加可在 Application\_Start 方法中创建表、队列和 Blob 容器的代码

Web 应用程序将使用 `MailingList` 表、`Message` 表、`azuremailsubscribequeue` 队列和 `azuremailblobcontainer` Blob 容器。你可以使用 Azure 存储资源管理器之类的工具手动创建这些项目，但创建完以后，每次通过新的存储帐户使用该应用程序时，你仍然必须手动执行这些操作。在本部分中，你要添加的代码会在应用程序启动时运行，会查看所需表、队列和 Blob 容器是否存在，并且会在这些项目不存在时创建它们。

你可以将此一次性启动代码添加到 *WebRole.cs* 文件的 `OnStart` 方法中，或者添加到 *Global.asax* 文件中。就本教程来说，你需要初始化 *Global.asax* 文件中的 Azure 存储空间，因为该空间兼容 Azure 网站和 Azure 云服务 Web 角色。

1.  在**解决方案资源管理器**中，展开 *Global.asax*，然后打开 *Global.asax.cs*。

2.  在 `Application_Start` 方法之后添加新的 `CreateTablesQueuesBlobContainers` 方法，然后从 `Application_Start` 方法调用该新方法，如以下示例所示：

        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            WebApiConfig.Register(GlobalConfiguration.Configuration);
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);
            AuthConfig.RegisterAuth();
            // Verify that all of the tables, queues, and blob containers used in this application
            // exist, and create any that don't already exist.
            CreateTablesQueuesBlobContainers();
        }

        private static void CreateTablesQueuesBlobContainers()
        {
            var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));
            // If this is running in an Azure Web Site (not a Cloud Service) use the Web.config file:
            //    var storageAccount = CloudStorageAccount.Parse(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
            var tableClient = storageAccount.CreateCloudTableClient();
            var mailingListTable = tableClient.GetTableReference("MailingList");
            mailingListTable.CreateIfNotExists();
            var messageTable = tableClient.GetTableReference("Message");
            messageTable.CreateIfNotExists();
            var blobClient = storageAccount.CreateCloudBlobClient();
            var blobContainer = blobClient.GetContainerReference("azuremailblobcontainer");
            blobContainer.CreateIfNotExists();
            var queueClient = storageAccount.CreateCloudQueueClient();
            var subscribeQueue = queueClient.GetQueueReference("azuremailsubscribequeue");
            subscribeQueue.CreateIfNotExists();
        }

3.  右键单击 `RoleEnvironment` 下的蓝色波浪线，选择**“解析”**，然后选择**“使用 Microsoft.MicrosoftAzure.ServiceRuntime”**。

    ![rightClick][rightClick]

4.  右键单击 `CloudStorageAccount` 下的蓝色波浪线，选择**“解析”**，然后选择**“使用 Microsoft.MicrosoftAzure.Storage”**。

5.  或者，你可以手动添加以下 using 语句：

        using Microsoft.WindowsAzure.ServiceRuntime;
        using Microsoft.WindowsAzure.Storage;

6.  生成应用程序，以便保存文件并确保没有任何编译错误。

在以下各部分，你将生成 Web 应用程序的组件，并且可以使用开发存储或存储帐户对其进行测试，不需首先手动创建表、队列或 Blob 容器。

## <a name="mailinglist"></a><span class="short-header">邮件列表</span>创建和测试邮件列表控制器和视图

**“邮件列表”**Web UI 可供管理员用来创建、编辑和显示邮件列表，例如“Contoso 大学历史系公告”和“Fabrikam 工程招聘广告”。

### 将 MailingList 实体类添加到“Models”文件夹

`MailingList` 实体类用于 `MailingList` 表中的行，这些行包含有关列表的信息，例如描述以及发送到列表的电子邮件的“发件人”电子邮件地址。

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 `Models` 文件夹，然后选择**“添加现有项”**。

    ![将现有项添加到“Models”文件夹][将现有项添加到“Models”文件夹]

2.  导航到下载了示例应用程序的文件夹，选择 `Models` 文件夹中的 *MailingList.cs* 文件，然后单击“添加”。

3.  打开 *MailingList.cs* 并检查代码。

        public class MailingList : TableEntity
        {
            public MailingList()
            {
                this.RowKey = "mailinglist";
            }

            [Required]
            [RegularExpression(@"[\w]+",
             ErrorMessage = @"Only alphanumeric characters and underscore (_) are allowed.")]
            [Display(Name = "List Name")]
            public string ListName
            {
                get
                {
                    return this.PartitionKey;
                }
                set
                {
                    this.PartitionKey = value;
                }
            }

            [Required]
            [Display(Name = "'From' Email Address")]
            public string FromEmailAddress { get; set; }

            public string Description { get; set; }
        }

    Azure 存储 TSL 2.0 API 要求用于表操作的实体类派生自 [TableEntity][TableEntity]。该类用于定义 `PartitionKey`、`RowKey`、`TimeStamp` 和 `ETag` 字段。`TimeStamp` 和 `ETag` 属性用于系统。你将在教程后面了解到如何将 `ETag` 属性用于并发处理。

    （此外还有 [DynamicTableEntity][DynamicTableEntity] 类，可以在你需要将表行用作键值对的字典集合时使用，这样就不需要使用预定义的模型类。有关详细信息，请参阅 [Azure 存储客户端库 2.0 表深入探讨][Azure 存储客户端库 2.0 表深入探讨]。）

    `mailinglist` 表分区键为列表名称。在此实体类中，分区键值可通过 `PartitionKey` 属性（在 `TableEntity` 类中定义）或 `ListName` 属性（在 `MailingList` 类中定义）访问。`ListName` 属性使用 `PartitionKey` 作为其后备变量。定义 `ListName` 属性以后，你就可以在代码中使用更具描述性的变量名称，更容易对 Web UI 进行编程，因为用于格式化和验证的 DataAnnotations 属性可以添加到 `ListName` 属性，但不能直接添加到 `PartitionKey` 属性。

    `ListName` 属性中的 `RegularExpression` 属性使 MVC 能够验证用户输入，确保输入的列表名称值仅包含字母数字字符或下划线。实施此限制是为了让列表名称简单些，这样就可以方便地在 URL 的查询字符串中使用它们。

    **注意：**如果你希望列表名称格式少一些限制，你可以允许在查询字符串中使用其他字符以及 URL 编码列表名称。不过，某些字符是不允许在 Azure 表分区键或行键中使用的，因此你至少必须排除这些字符。某些字符在分区键或行键字段中是不允许的，或者会引发问题，有关此方面的信息，请参阅[了解表服务数据模型][了解表服务数据模型]和 [PartitionKey 或 RowKey 中的 % 字符][PartitionKey 或 RowKey 中的 % 字符]。

    `MailingList` 类用于定义可将 `RowKey` 设置为硬编码字符串“mailinglist”的默认构造函数，因为此表中的所有邮件列表行都将该值作为其行键。（有关表结构的解释，请参阅[本系列第一个教程][本系列第一个教程]。）可以选择任何适合此用途的常量值，前提是该值不是电子邮件地址，因为电子邮件地址是此表中订户行的行键。

    创建新的 `MailingList` 实体时，必须始终输入列表名称和“发件人”电子邮件地址，使得它们都具有 `Required` 属性。

    `Display` 属性所指定的默认标题将用于 MVC UI 中的字段。

### 添加 MailingList MVC 控制器

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 Controllers 文件夹，然后选择**“添加现有项”**。

    ![将现有项添加到 Controllers 文件夹][将现有项添加到 Controllers 文件夹]

2.  导航到下载了示例应用程序的文件夹，选择 `Controllers` 文件夹中的 *MailingListController.cs* 文件，然后单击“添加”。

3.  打开 *MailingListController.cs* 并检查代码。

    默认构造函数创建适用于 `mailinglist` 表的 `CloudTable` 对象。

        public class MailingListController : Controller
        {
            private CloudTable mailingListTable;

            public MailingListController()
            {
                var storageAccount = Microsoft.WindowsAzure.Storage.CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));
                // If this is running in an Azure Web Site (not a Cloud Service) use the Web.config file:
                //    var storageAccount = Microsoft.WindowsAzure.Storage.CloudStorageAccount.Parse(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

                var tableClient = storageAccount.CreateCloudTableClient();
                mailingListTable = tableClient.GetTableReference("mailinglist");
            }

    代码会从云服务项目设置文件中获取用于 Azure 存储帐户的凭据，以便连接到该存储帐户。（你将在本教程后面部分配置这些设置，然后才测试控制器。）如果你打算在 Azure 网站中运行 MVC 项目，你可以改为从 Web.config 文件中获取连接字符串。

    接下来是一个 `FindRow` 方法，该方法在控制器需要查找 `MailingList` 表的特定邮件列表条目时调用，以便执行相应的操作，例如编辑邮件列表条目。代码会使用传入的分区键和行键值来检索单个 `MailingList` 实体。此控制器编辑的行是使用“MailingList”作为行键的行，因此本来是可以针对行键对“MailingList”进行硬编码的，但指定分区键和行键是一种用于所有控制器中 `FindRow` 方法的模式。

        private MailingList FindRow(string partitionKey, string rowKey)
        {
            var retrieveOperation = TableOperation.Retrieve<MailingList>(partitionKey, rowKey);
            var retrievedResult = mailingListTable.Execute(retrieveOperation);
            var mailingList = retrievedResult.Result as MailingList;
            if (mailingList == null)
            {
                throw new Exception("No mailing list found for: " + partitionKey);
            }

            return mailingList;
        }

    建议将 `MailingList` 控制器中的 `FindRow` 方法（返回邮件列表行）与 `Subscriber` 控制器中的 `FindRow` 方法（返回同一 `mailinglist` 表中的订户行）进行比较。

        private Subscriber FindRow(string partitionKey, string rowKey)
        {
            var retrieveOperation = TableOperation.Retrieve<Subscriber>(partitionKey, rowKey);
            var retrievedResult = mailingListTable.Execute(retrieveOperation);
            var subscriber = retrievedResult.Result as Subscriber;
            if (subscriber == null)
            {
                throw new Exception("No subscriber found for: " + partitionKey + ", " + rowKey);
            }
            return subscriber;
        }

    这两种查询的唯一区别是它们传递到 [TableOperation.Retrieve][TableOperation.Retrieve] 方法的模型类型。模型类型用于指定你希望查询返回的单个或多个行的架构（属性）。单个表的不同行可能具有不同的架构。通常情况下，你在读取某一行时指定的模型类型与创建该行时使用的模型类型相同。

    **“索引”**页显示所有邮件列表行，因此 `Index` 方法中的查询返回的所有 `MailingList` 实体都使用“mailinglist”作为行键（表中的其他行使用电子邮件地址作为行键，并且包含订户信息）。

                var query = new TableQuery<MailingList>().Where(TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.Equal, "mailinglist"));
                lists = mailingListTable.ExecuteQuery(query, reqOptions).ToList();

    `Index` 方法将此查询置于旨在处理超时情况的代码中。

        public ActionResult Index()
        {
            TableRequestOptions reqOptions = new TableRequestOptions()
            {
                MaximumExecutionTime = TimeSpan.FromSeconds(1.5),
                RetryPolicy = new LinearRetry(TimeSpan.FromSeconds(3), 3)
            };
            List<MailingList> lists;
            try
            {
                var query = new TableQuery<MailingList>().Where(TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.Equal, "mailinglist"));
                lists = mailingListTable.ExecuteQuery(query, reqOptions).ToList();
            }
            catch (StorageException se)
            {
                ViewBag.errorMessage = "Timeout error, try again. ";
                Trace.TraceError(se.Message);
                return View("Error");
            }

            return View(lists);
        }

    如果不指定超时参数，API 将自动重试三次，超时限制呈指数增长。对于有用户等待页面显示的 Web 接口来说，这可能会导致出现长到无法令人接受的等待时间。因此，此代码会指定线性重试次数（使得超时限制不会每次都提高），并且会指定一个对于等待的用户来说合理的超时限制。

    当用户单击“创建”页上的“创建”按钮时，MVC 模型联编程序会根据视图中输入的内容创建一个 `MailingList` 实体，而 `HttpPost Create` 方法则会将该实体添加到表中。

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(MailingList mailingList)
        {
            if (ModelState.IsValid)
            {
                var insertOperation = TableOperation.Insert(mailingList);
                mailingListTable.Execute(insertOperation);
                return RedirectToAction("Index");
            }

            return View(mailingList);
        }

    对于“编辑”页，则可用 `HttpGet Edit` 方法来查找行，用 `HttpPost` 方法来更新行。

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit(string partitionKey, string rowKey, MailingList editedMailingList)
        {
            if (ModelState.IsValid)
            {
                var mailingList = new MailingList();
                UpdateModel(mailingList);
                try
                {
                    var replaceOperation = TableOperation.Replace(mailingList);
                    mailingListTable.Execute(replaceOperation);
                    return RedirectToAction("Index");
                }
                catch (StorageException ex)
                {
                    if (ex.RequestInformation.HttpStatusCode == 412)
                    {
                        // Concurrency error
                        var currentMailingList = FindRow(partitionKey, rowKey);
                        if (currentMailingList.FromEmailAddress != editedMailingList.FromEmailAddress)
                        {
                            ModelState.AddModelError("FromEmailAddress", "Current value: " + currentMailingList.FromEmailAddress);
                        }
                        if (currentMailingList.Description != editedMailingList.Description)
                        {
                            ModelState.AddModelError("Description", "Current value: " + currentMailingList.Description);
                        }
                        ModelState.AddModelError(string.Empty, "The record you attempted to edit "
                            + "was modified by another user after you got the original value. The "
                            + "edit operation was canceled and the current values in the database "
                            + "have been displayed. If you still want to edit this record, click "
                            + "the Save button again. Otherwise click the Back to List hyperlink.");
                         ModelState.SetModelValue("ETag", new ValueProviderResult(currentMailingList.ETag, currentMailingList.ETag, null));
                    }
                    else
                    {
                        throw; 
                    }
                }
            }
            return View(editedMailingList);
        }

    try-catch 块用于处理并发错误。如果某个用户选择一个邮件列表进行编辑，打开的**“编辑”**页显示在浏览器中，而另一个用户也要编辑同一个邮件列表，此时就会发生并发异常。发生这种情况时，代码会显示一条警告消息，指示另一个用户更改了哪些字段。TSL API 使用 `ETag` 来检查并发冲突。每次更新一个表行时，将更改 `ETag` 值。当你编辑一个行时，你需要保存 `ETag` 值，而当你执行更新或删除操作时，你需要传入所保存的 `ETag` 值。（`Edit` 视图有一个 ETag 值的隐藏字段。）如果更新操作发现你要更新的记录中的 `ETag` 值不同于你传入到更新操作的 `ETag` 值，则会引发并发异常。如果你不在意并发冲突，则可在传入到更新操作的实体中将 ETag 字段设置为星号（“\*”），这样就会忽略冲突。

    注意：HTTP 412 错误并不是针对并发错误的。其他错误也可能导致 SCL API 引发 HTTP 412 错误。

    就“删除”页面来说，`HttpGet Delete` 用法用于查找需要显示内容的行，`HttpPost` 方法用于删除 `MailingList` 行以及 `MailingList` 表中与之关联的任何 `Subscriber` 行。

        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public ActionResult DeleteConfirmed(string partitionKey)
        {
            // Delete all rows for this mailing list, that is, 
            // Subscriber rows as well as MailingList rows.
            // Therefore, no need to specify row key.
            var query = new TableQuery<MailingList>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, partitionKey));
            var listRows = mailingListTable.ExecuteQuery(query).ToList();
            var batchOperation = new TableBatchOperation();
            int itemsInBatch = 0;
            foreach (MailingList listRow in listRows)
            {
                batchOperation.Delete(listRow);
                itemsInBatch++;
                if (itemsInBatch == 100)
                {
                    mailingListTable.ExecuteBatch(batchOperation);
                    itemsInBatch = 0;
                    batchOperation = new TableBatchOperation();
                }
            }
            if (itemsInBatch > 0)
            {
                mailingListTable.ExecuteBatch(batchOperation);
            }
            return RedirectToAction("Index");
        }

    如果需要删除大量订户，该代码会以批处理的方式删除记录。删除一个行的事务成本与按批处理方式删除 100 行相同。在一次批处理中能够执行的最大操作数为 100。

    虽然循环会处理 `MailingList` 行和 `Subscriber` 行，但它会将这些行全部读取到 `MailingList` 实体类中，因为 `Delete` 操作所需的字段仅为 `PartitionKey`、`RowKey` 和 `ETag` 字段。

### 添加 MailingList MVC 视图

1.  在**解决方案资源管理器**中，在 MVC 项目的 *Views* 文件夹中创建一个新的文件夹，然后将其命名为 *MailingList*。

2.  右键单击新的*“Views\\MailingList”*文件夹，然后选择**“添加现有项”**。

    ![将现有项添加到 Views 文件夹][将现有项添加到 Views 文件夹]

3.  导航到已下载示例应用程序的文件夹，选择 *Views\\MailingList* 文件夹中的所有四个 .cshtml 文件，然后单击“添加”。

4.  打开 *Edit.cshtml* 文件并检查代码。

        @model MvcWebRole.Models.MailingList
                @{
            ViewBag.Title = "Edit Mailing List";
        }
                <h2>Edit Mailing List</h2>
              @using (Html.BeginForm()) {
              @Html.AntiForgeryToken()
              @Html.ValidationSummary(true)
              @Html.HiddenFor(model => model.ETag)
              <fieldset>
                <legend>MailingList</legend>
                <div class="editor-label">
                    @Html.LabelFor(model => model.ListName)
                </div>
                <div class="editor-field">
                    @Html.DisplayFor(model => model.ListName)
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.Description)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.Description)
                    @Html.ValidationMessageFor(model => model.Description)
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.FromEmailAddress)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.FromEmailAddress)
                    @Html.ValidationMessageFor(model => model.FromEmailAddress)
                </div>
                <p>
                    <input type="submit" value="Save" />
                </p>
            </fieldset>
        }
        <div>
            @Html.ActionLink("Back to List", "Index")
        </div>      
        @section Scripts {
            @Scripts.Render("~/bundles/jqueryval")
        }

    此代码是针对 MVC 视图的典型代码。请注意所提供的隐藏字段，该字段可保存处理并发冲突所用的 `ETag` 值。另请注意，`ListName` 字段有一个 `DisplayFor` 帮助器而非 `EditorFor` 帮助器。我们并没有允许“编辑”页更改列表名称，因为这需要在控制器中提供复杂的代码：这样 `HttpPost Edit` 方法就必须删除现有邮件列表行以及所有关联的订户行，并使用新键值将它们重新插入。在生产应用程序中，你可能觉得提高复杂性是值得的。你后面会看到，`Subscriber` 控制器确实允许更改列表名称，因为一次只影响一行。

    *Create.cshtml* 和 *Delete.cshtml* 代码类似于 *Edit.cshtml*。

5.  打开 *Index.cshtml* 并检查代码。

        @model IEnumerable<MvcWebRole.Models.MailingList>
        @{
            ViewBag.Title = "Mailing Lists";
        }
        <h2>Mailing Lists</h2>
        <p>
            @Html.ActionLink("Create New", "Create")
        </p>
        <table>
            <tr>
                <th>
                    @Html.DisplayNameFor(model => model.ListName)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.Description)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.FromEmailAddress)
                </th>
                <th></th>
            </tr>
        @foreach (var item in Model) {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.ListName)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.Description)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.FromEmailAddress)
                </td>
                <td>
                    @Html.ActionLink("Edit", "Edit", new { PartitionKey = item.PartitionKey, RowKey=item.RowKey }) |
                    @Html.ActionLink("Delete", "Delete", new { PartitionKey = item.PartitionKey, RowKey=item.RowKey  })
                </td>
            </tr>
        }
        </table>

    此代码也是针对 MVC 视图的典型代码。**“编辑”**和**“删除”**超链接用于指定分区键和行键的查询字符串参数，以便标识特定行。对于 `MailingList` 实体来说，实际上只需要分区键，因为行键始终为“MailingList”，但二者均会保留，目的是让所有控制器和视图的 MVC 视图代码保持一致。

### 使 MailingList 成为默认控制器

1.  打开 *App\_Start* 文件夹中的 *Route.config.cs*。

2.  在指定默认值的行中，将默认控制器从“主页”更改为“MailingList”。

         routes.MapRoute(
             name: "Default",
             url: "{controller}/{action}/{id}",
             defaults: new { controller = "MailingList", action = "Index", id = UrlParameter.Optional }

## <a name="configurestorage"></a><span class="short-header">配置存储</span>配置 Web 角色，以使用你的 Azure 存储测试帐户

你需要输入在本地运行项目时要使用的测试存储帐户的设置。若要添加新的设置，你必须在云中和本地都添加它，不过你可以稍后更改云中设置的值。你可以在以后为辅助角色 A 添加相同的设置。

（如果你想要在 Azure 网站而不是 Azure 云服务中运行 Web UI，请参阅本教程后面的[替代体系结构][（可选）构建替代体系结构]部分，了解对这些说明所做的更改。）

1.  在**“解决方案资源管理器”**中，右键单击 **AzureEmailService** 云项目中**“角色”**下的**“MvcWebRole”**，然后选择**“属性”**。

    ![Web 角色属性][Web 角色属性]

2.  请确保在**“服务配置”**下拉列表中选择了“所有配置”。

3.  选择**“设置”**选项卡，然后单击**“添加设置”**。

4.  在“名称”列中输入“StorageConnectionString”。

5.  在“类型”下拉列表中选择“连接字符串”。

6.  单击行的右端的省略号 (**...**) 按钮，打开**“存储帐户连接字符串”**对话框。

    ![右键单击属性][右键单击属性]

7.  在**“创建存储连接字符串”**对话框中，单击**“你的订阅”**单选按钮，然后单击**“下载发布设置”**链接。

    > [WACOM.NOTE] 最新 SDK 不需下载任何东西，从下拉列表的可用存储帐户中选择即可。

    **注意：**如果你为教程 2 配置了存储设置，而你在同一台计算机上学习本教程，则不需再次下载设置，只需单击**“你的订阅”**，然后选择正确的**订阅**和**帐户名称**即可。

    ![右键单击属性][2]

    当你单击**“下载发布设置”**链接时，Visual Studio 会启动默认浏览器的新实例，其中包含 Azure 管理门户下载发布设置页的 URL。如果你尚未登录到门户中，系统会提示你登录。登录后，浏览器会提示你保存发布设置。记下保存设置的位置。

    ![发布设置][发布设置]

8.  在**“创建存储连接字符串”**对话框中，单击**“导入”**，然后导航到你在上一步保存的发布设置文件。

9.  选择想要使用的订阅和存储帐户，然后单击**“确定”**。

    ![选择存储帐户][选择存储帐户]

10. 按照你所使用的针对 `StorageConnectionString` 连接字符串的相同过程，设置 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 连接字符串。

    你无需再次下载发布设置文件。当你单击 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 连接字符串的省略号时，你会发现，**“创建存储连接字符串”**对话框会记住你的订阅信息。当你单击**“你的订阅”**单选按钮时，你需要做的就是选择你此前已经选择过的同一**订阅**和**帐户名**，然后单击**“确定”**。

11. 按照你用于 MvcWebRole 角色的两个连接字符串的同一过程，设置 WorkerRoleA 角色的连接字符串。

使用**“添加设置”**按钮添加新字符串时，新设置会添加到 *ServiceDefinition.csdf* 文件以及两个 *.cscfg* 配置文件的 XML 中。以下 XML 由 Visual Studio 添加到 *ServiceDefinition.csdf* 文件中。

      <ConfigurationSettings>
        <Setting name="StorageConnectionString" />
      </ConfigurationSettings>

以下 XML 将添加到每个 *.cscfg* 配置文件。

       <Setting name="StorageConnectionString"
       value="DefaultEndpointsProtocol=https;
       AccountName=azuremailstorage;
       AccountKey=[your account key]" />

你可以将设置手动添加到 *ServiceDefinition.csdf* 文件和两个 *.cscfg* 配置文件中，但使用属性编辑器对于连接字符串来说具有下列优势：

-   你只需在一个位置添加新设置，然后正确的设置 XML 就会添加到所有三个文件中。
-   将为这三个设置文件生成正确的 XML。*ServiceDefinition.csdf* 文件用于定义必须存在于每个 *.cscfg* 配置文件中的设置。如果 *ServiceDefinition.csdf* 文件和两个 *.cscfg* 配置文件的设置不一致，你可能会从 Visual Studio 中收到以下错误消息：“当前的服务模型不同步。请确保服务配置和定义文件有效。”

    ![无效的服务配置和定义文件错误][无效的服务配置和定义文件错误]

如果收到此错误，则必须解决不一致性问题，然后才能使用属性编辑器。

### 测试应用程序

1.  按 CTRL+F5 运行该项目。

    ![空的 MailingList 索引页][空的 MailingList 索引页]

2.  使用**“创建”**功能添加一些邮件列表，然后尝试**“编辑”**和**“删除”**功能，确保其有效。

    ![包含行的 MailingList 索引页][包含行的 MailingList 索引页]

## <a name="subscriber"></a><span class="short-header">订户</span>创建和测试“订户”控制器和视图

**“订户”**Web UI 由管理员用来向邮件列表添加新订户，以及编辑、显示和删除现有订户。

### 将“订户”实体类添加到“Models”文件夹

`Subscriber` 实体类用于 `MailingList` 表中包含列表订户信息的行。这些行包含用户电子邮件地址以及该地址是否已验证等信息。

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 *Models* 文件夹，然后选择**“添加现有项”**。

2.  导航到你在其中下载了示例应用程序的文件夹，选择 *Models* 文件夹中的 *Subscriber.cs* 文件，然后单击“添加”。

3.  打开 *Subscriber.cs* 并检查代码。

            public class Subscriber : TableEntity
            {
                [Required]
                public string ListName
                {
                    get
                    {
                        return this.PartitionKey;
                    }
                    set
                    {
                        this.PartitionKey = value;
                    }
                }

                [Required]
                [Display(Name = "Email Address")]
                public string EmailAddress
                {
                    get
                    {
                        return this.RowKey;
                    }
                    set
                    {
                        this.RowKey = value;
                    }
                }

                public string SubscriberGUID { get; set; }

                public bool? Verified { get; set; }
            }

    与 `MailingList` 实体类一样，`Subscriber` 实体类用于在 `mailinglist` 表中进行行的读取和写入操作。`Subscriber` 行将电子邮件地址而不是常量“mailinglist”用作行键。（有关表结构的解释，请参阅[本系列第一个教程][本系列第一个教程]。）因此，在定义 `EmailAddress` 属性时，会将其定义为使用 `RowKey` 属性作为其后备字段，这种方式与 `ListName` 使用 `PartitionKey` 作为其后备字段是一样的。如前所述，这样你就可以为属性设置用于格式化和验证的 DataAnnotations 特性。

    `SubscriberGUID` 值在创建 `Subscriber` 实体时生成。它用在订阅和取消订阅链接中，用于确保只有经过授权的用户才能通过电子邮件进行订阅或取消订阅操作。
    在一开始为新订户创建某个行时，`Verified` 值为 `false`。仅当新订户单击欢迎电子邮件中的“确认”超链接以后，`Verified` 值才会更改为 `true`。如果将电子邮件发送到订户的 `Verified` 为 `false` 的列表，则不会向该订户发送邮件。

    根据定义，`Subscriber` 实体中的 `Verified` 属性可以为 null。当你指定某个查询应返回 `Subscriber` 实体时，可能会出现某些被检索的行没有 `Verified` 属性的情况。因此，`Subscriber` 实体会将其 `Verified` 属性定义为可以为 null，这样就可以在某个查询所返回的表行没有*“已验证”*属性的情况下，更准确地反映某一行的实际内容。你可能会习惯于使用 SQL Server 表，该表的每一行都具有相同的架构。在 Azure 存储表中，每一行都只是属性的集合，每行都可以有不同的属性集。例如，在 Azure 电子邮件服务示例应用程序中，使用“MailingList”作为行键的行没有 `Verified` 属性。如果查询返回的表行没有 `Verified` 属性，则在实例化 `Subscriber` 实体类时，实体对象中的 `Verified` 属性将为 null。如果属性无法为 null，则不管是 `Verified` 为 `false` 的行还是根本没有 `Verified` 属性的行，你得到的值都是 `false`。因此，使用 Azure 表时，最佳做法是将实体类的每个属性均设置为可以为 null，这样就可以准确读取使用不同实体类或不同版本的最新实体类创建的行。

### 添加订户 MVC 控制器

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 *Controllers* 文件夹，然后选择**“添加现有项”**。

2.  导航到你在其中下载了示例应用程序的文件夹，选择 *Controllers* 文件夹中的 *SubscriberController.cs* 文件，然后单击“添加”。（确保你获得的是 *Subscriber.cs* 而不是 *Subscribe.cs*；你将稍后添加 *Subscribe.cs*。）

3.  打开 *SubscriberController.cs* 并检查代码。

    此控制器中的大多数代码与你在 `MailingList` 控制器中看到的代码类似。甚至连表名也是相同的，因为订户信息保留在 `MailingList` 表中。在 `FindRow` 方法后你会看到 `GetListNames` 方法。此方法用于获取**“创建”**和**“编辑”**页上的下拉列表的数据，你可以在这些页面中选择要通过电子邮件地址进行订阅的邮件列表。

        private List<MailingList> GetListNames()
        {
            var query = (new TableQuery<MailingList>().Where(TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.Equal, "mailinglist")));
            var lists = mailingListTable.ExecuteQuery(query).ToList();
            return lists;
        }

    该查询与你在 `MailingList` 控制器中看到的查询相同。就下拉列表来说，你需要的是包含邮件列表信息的行，因此你只选择 RowKey 为“mailinglist”的行。

    就那些用于检索**“索引”**页数据的方法来说，你需要的是包含订户信息的行，因此你选择 RowKey 不为“MailingList”的所有行。

        public ActionResult Index()
        {
            var query = (new TableQuery<Subscriber>().Where(TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.NotEqual, "mailinglist")));
            var subscribers = mailingListTable.ExecuteQuery(query).ToList();
            return View(subscribers);
        }

    请注意，该查询指定将数据读入 `Subscriber` 对象中（通过指定 `<Subscriber>`），而数据读取则是从 `mailinglist` 表读取。

    **注意：**订户数量可能会增长到很大，导致无法在单个查询中通过这种方式进行处理。在本教程将来的版本中，我们希望实现分页功能，并演示如何处理继续标记。当你执行的查询会返回 1,000 多行时，你需要处理继续标记：Azure 会返回 1,000 行和一个继续标记，你可以使用该标记执行另一个从之前的查询中断的地方开始的查询。（Azure 存储资源管理器不处理继续标记；因此，其查询不会返回 1,000 多行。）有关大型结果集和继续标记的详细信息，请参阅[如何充分利用 Azure 表][如何充分利用 Azure 表]和 [Azure 表：做好处理继续标记的足够准备][Azure 表：做好处理继续标记的足够准备]。

    在 `HttpGet Create` 方法中，你为下拉列表设置数据；而在 `HttpPost` 方法中，你在保存新实体前设置默认值。

        public ActionResult Create()
        {
            var lists = GetListNames();
            ViewBag.ListName = new SelectList(lists, "ListName", "Description");
            var model = new Subscriber() { Verified = false };
            return View(model);
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(Subscriber subscriber)
        {
            if (ModelState.IsValid)
            {
                subscriber.SubscriberGUID = Guid.NewGuid().ToString();
                if (subscriber.Verified.HasValue == false)
                {
                    subscriber.Verified = false;
                }

                var insertOperation = TableOperation.Insert(subscriber);
                mailingListTable.Execute(insertOperation);
                return RedirectToAction("Index");
            }

            var lists = GetListNames();
            ViewBag.ListName = new SelectList(lists, "ListName", "Description", subscriber.ListName);

            return View(subscriber);
        }

    `HttpPost Edit` 页比你在 `MailingList` 控制器中看到的要复杂，因为 `Subscriber` 页可用于更改列表名称或电子邮件地址，这二者都是键字段。如果用户更改了这其中一个字段，你必须删除现有记录并添加一个新记录，而不能只是更新现有记录。以下代码演示了编辑方法的那一部分内容，该方法用于处理键更改和非键更改的不同过程：

            if (ModelState.IsValid)
            {
                try
                {
                    UpdateModel(editedSubscriber, string.Empty, null, excludeProperties);
                    if (editedSubscriber.PartitionKey == partitionKey && editedSubscriber.RowKey == rowKey)
                    {
                        //Keys didn't change -- Update the row
                        var replaceOperation = TableOperation.Replace(editedSubscriber);
                        mailingListTable.Execute(replaceOperation);
                    }
                    else
                    {
                        // Keys changed, delete the old record and insert the new one.
                        if (editedSubscriber.PartitionKey != partitionKey)
                        {
                            // PartitionKey changed, can't do delete/insert in a batch.
                            var deleteOperation = TableOperation.Delete(new Subscriber { PartitionKey = partitionKey, RowKey = rowKey, ETag = editedSubscriber.ETag });
                            mailingListTable.Execute(deleteOperation);
                            var insertOperation = TableOperation.Insert(editedSubscriber);
                            mailingListTable.Execute(insertOperation);
                        }
                        else
                        {
                            // RowKey changed, do delete/insert in a batch.
                            var batchOperation = new TableBatchOperation();
                            batchOperation.Delete(new Subscriber { PartitionKey = partitionKey, RowKey = rowKey, ETag = editedSubscriber.ETag });
                            batchOperation.Insert(editedSubscriber);
                            mailingListTable.ExecuteBatch(batchOperation);
                        }
                    }
                    return RedirectToAction("Index");

    MVC 模型联编程序传递到 `Edit` 方法的参数包括原始列表名称和电子邮件地址值（在 `partitionKey` 和 `rowKey` 参数中），以及用户输入的值（在 `listName` 和 `emailAddress` 参数中）：

        public ActionResult Edit(string partitionKey, string rowKey, string listName, string emailAddress)

    传递到 `UpdateModel` 方法的参数不包括模型绑定中的 `PartitionKey` 和 `RowKey` 属性：

            var excludeProperties = new string[] { "PartitionKey", "RowKey" };

    之所以这样做，是因为 `ListName` 和 `EmailAddress` 属性使用 `PartitionKey` 和 `RowKey` 作为其后备属性，而用户可能已更改了这其中的某个值。当模型联编程序通过设置 `ListName` 属性来更新模型时，`PartitionKey` 属性也会自动获得更新。如果模型联编程序在更新 `ListName` 属性后要使用 `PartitionKey` 属性的原始值来更新它，则会覆盖 `ListName` 属性所设置的新值。`EmailAddress` 属性使用相同的方式自动更新 `RowKey` 属性。

    更新 `editedSubscriber` 模型对象后，代码就会判断分区键或行键是否已更改。如果其中某个键值已更改，则必须删除现有订户行并插入一个新行。如果仅更改了行键，则可通过原子批处理事务方式完成删除和插入操作。

    请注意，代码会创建一个要传入到 `Delete` 操作的新实体：

            // RowKey changed, do delete/insert in a batch.
            var batchOperation = new TableBatchOperation();
            batchOperation.Delete(new Subscriber { PartitionKey = partitionKey, RowKey = rowKey, ETag = editedSubscriber.ETag });
            batchOperation.Insert(editedSubscriber);
            mailingListTable.ExecuteBatch(batchOperation);

    采用批处理方式传入到操作中的实体必须是不同的实体。例如，你不能创建一个 `Subscriber` 实体，将其传入到 `Delete` 操作中，然后更改同一 `Subscriber` 实体中的某个值，再将其传入到 `Insert` 操作中。如果你这样做了，进行属性更改后的实体的状态对于删除操作和插入操作都是有效的。

    **注意：**批处理操作必须全部在同一分区进行。由于更改列表名称会更改分区键，因此无法在事务中进行这种更改。

### 添加订户 MVC 视图

1.  在**解决方案资源管理器**中，在 MVC 项目的 *Views* 文件夹中创建一个新的文件夹，然后将其命名为 *Subscriber*。

2.  右键单击新的*“Views\\Subscriber”*文件夹，然后选择**“添加现有项”**。

3.  导航到已下载示例应用程序的文件夹，选择 *Views\\Subscriber* 文件夹中的所有五个 .cshtml 文件，然后单击“添加”。

4.  打开 *Edit.cshtml* 文件并检查代码。

        @model MvcWebRole.Models.Subscriber

        @{
            ViewBag.Title = "Edit Subscriber";
        }

        <h2>Edit Subscriber</h2>

        @using (Html.BeginForm()) {
            @Html.AntiForgeryToken()
            @Html.ValidationSummary(true)
             @Html.HiddenFor(model => model.SubscriberGUID)
             @Html.HiddenFor(model => model.ETag)
             <fieldset>
                <legend>Subscriber</legend>
                <div class="display-label">
                     @Html.DisplayNameFor(model => model.ListName)
                </div>
                <div class="editor-field">
                    @Html.DropDownList("ListName", String.Empty)
                    @Html.ValidationMessageFor(model => model.ListName)
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.EmailAddress)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.EmailAddress)
                    @Html.ValidationMessageFor(model => model.EmailAddress)
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.Verified)
                </div>
                <div class="display-field">
                    @Html.EditorFor(model => model.Verified)
                </div>
                <p>
                    <input type="submit" value="Save" />
                </p>
            </fieldset>
        }

        <div>
            @Html.ActionLink("Back to List", "Index")
        </div>

        @section Scripts {
            @Scripts.Render("~/bundles/jqueryval")
        }

    此代码与你此前看到的针对 `MailingList`**“编辑”**视图的代码类似。`SubscriberGUID` 值不显示，可见不会在 `HttpPost` 控制器方法的窗体字段中自动提供值。因此，将提供一个隐藏字段，用于保留该值。

    其他视图包含的代码类似于你所看到的针对 `MailingList` 控制器的代码。

### 测试应用程序

1.  通过按 CTRL+F5 运行项目，然后单击**“订户”**。

    ![空的订户索引页][空的订户索引页]

2.  使用**“创建”**功能添加一些邮件列表，然后尝试**“编辑”**和**“删除”**功能，确保其有效。

    ![包含行的订户索引页][包含行的订户索引页]

## <a name="message"></a><span class="short-header">邮件</span>创建和测试“邮件”控制器和视图

**“邮件”**Web UI 由管理员用于创建、编辑和显示计划要发送到邮件列表的邮件的相关信息。

### 将“邮件”实体类添加到“Models”文件夹

`Message` 实体类用于 `Message` 表中包含邮件（计划要发送到列表）信息的行。这些行包含主题行、要向其发送邮件的列表、邮件的计划发送日期等信息。

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 *Models* 文件夹，然后选择**“添加现有项”**。

2.  导航到下载了示例应用程序的文件夹，选择 Models 文件夹中的 *Message.cs* 文件，然后单击“添加”。

3.  打开 *Message.cs* 并检查代码。

        public class Message : TableEntity
        {
            private DateTime? _scheduledDate;
            private long _messageRef;

            public Message()
            {
                this.MessageRef = DateTime.Now.Ticks;
                this.Status = "Pending";
            }

            [Required]
            [Display(Name = "Scheduled Date")]
            // DataType.Date shows Date only (not time) and allows easy hook-up of jQuery DatePicker
            [DataType(DataType.Date)]
            public DateTime? ScheduledDate 
            {
                get
                {
                    return _scheduledDate;
                }
                set
                {
                    _scheduledDate = value;
                    this.PartitionKey = value.Value.ToString("yyyy-MM-dd");
                }
            }

            public long MessageRef 
            {
                get
                {
                    return _messageRef;
                }
                set
                {
                    _messageRef = value;
                    this.RowKey = "message" + value.ToString();
                }
            }

            [Required]
            [Display(Name = "List Name")]
            public string ListName { get; set; }

            [Required]
            [Display(Name = "Subject Line")]
            public string SubjectLine { get; set; }

            // Pending, Queuing, Processing, Complete
            public string Status { get; set; }
        }

    `Message` 类用于订阅一个默认构造函数，该函数将 `MessageRef` 属性设置为一个针对邮件的唯一值。由于此值是行键的一部分，`MessageRef` 属性的 setter 也会自动设置 `RowKey` 属性。`MessageRef` 属性 setter 将“message”字样和 `MessageRef` 值连接在一起，然后将其置于 `RowKey` 属性中。

    从 `DateTime.Now` 中获取 `Ticks` 值后，即可创建 `MessageRef` 值。这样就可以确保在默认情况下，当你在 Web UI 中显示邮件时，这些邮件的显示顺序与给定计划日期的邮件创建顺序是相同的（`ScheduledDate` 为分区键）。你可以使用 GUID 来确保邮件行是唯一的，但这样一来，默认检索顺序就会是随机的。

    默认构造函数还会将新的 `message` 行的默认状态设置为“挂起”。

    有关 `Message` 表结构的详细信息，请参阅[本系列第一个教程][本系列第一个教程]。

### 添加邮件 MVC 控制器

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 Controllers 文件夹，然后选择**“添加现有项”**。

2.  导航到下载了示例应用程序的文件夹，选择 *Controllers* 文件夹中的 *MessageController.cs* 文件，然后单击“添加”。

3.  打开 *MessageController.cs* 并检查代码。

    此控制器中的大多数代码与你在 `Subscriber` 控制器中看到的代码类似。此处的新内容是用于 Blob 的代码。就每封电子邮件来说，邮件的 HTML 内容和纯文本内容会以 .htm 和 .txt 文件形式上载并存储到 Blob 中。

    Blob 存储在 Blob 容器中。Azure 电子邮件服务应用程序将其所有 Blob 存储在单个名为“azuremailblobcontainer”的 Blob 容器中，而控制器构造函数中的代码会获得对此 Blob 容器的引用：

        public class MessageController : Controller
        {
            private TableServiceContext serviceContext;
            private static CloudBlobContainer blobContainer;

              public MessageController()
            {
                var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString"));
                // If this is running in an Azure Web Site (not a Cloud Service) use the Web.config file:
                //    var storageAccount = CloudStorageAccount.Parse(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

                // Get context object for working with tables and a reference to the blob container.
                var tableClient = storageAccount.CreateCloudTableClient();
            serviceContext = tableClient.GetTableServiceContext();
                var blobClient = storageAccount.CreateCloudBlobClient();
                blobContainer = blobClient.GetContainerReference("azuremailblobcontainer");
            }

    对于用户选择上载的每个文件，MVC 视图都会提供一个 `HttpPostedFile` 对象，其中包含有关该文件的信息。当用户创建新的邮件时，系统会使用 `HttpPostedFile` 对象将文件保存到 Blob。用户在编辑某封邮件时，可以选择上载一个替换文件或保留 Blob 不变。

    控制器包含一个方法，`HttpPost Create` 和 `HttpPost Edit` 方法可以调用该方法来保存 Blob：

        private void SaveBlob(string blobName, HttpPostedFileBase httpPostedFile)
        {
            // Retrieve reference to a blob. 
            CloudBlockBlob blob = blobContainer.GetBlockBlobReference(blobName);
            // Create the blob or overwrite the existing blob by uploading a local file.
            using (var fileStream = httpPostedFile.InputStream)
            {
                blob.UploadFromStream(fileStream);
            }
        }

    `HttpPost Create` 方法会保存这两个 Blob，然后添加 `Message` 表行。将 `MessageRef` 值与文件扩展名“.htm”或“.txt”连接到一起即为 Blob 的名称。

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create(Message message, HttpPostedFileBase file, HttpPostedFileBase txtFile)
        {
            if (file == null)
            {
                ModelState.AddModelError(string.Empty, "Please provide an HTML file path");
            }

            if (txtFile == null)
            {
                ModelState.AddModelError(string.Empty, "Please provide a Text file path");
            }

            if (ModelState.IsValid)
            {
                SaveBlob(message.MessageRef + ".htm", file);
                SaveBlob(message.MessageRef + ".txt", txtFile);

                var insertOperation = TableOperation.Insert(message);
                messageTable.Execute(insertOperation);

                return RedirectToAction("Index");
            }

            var lists = GetListNames();
            ViewBag.ListName = new SelectList(lists, "ListName", "Description");
            return View(message);
        }

    `HttpGet Edit` 方法用于验证检索到的邮件是否处于 `Pending` 状态，如果是，则当辅助角色 B 开始处理邮件时，用户就不能更改该邮件。类似的代码也见于 `HttpPost Edit` 方法以及 `Delete` 和 `DeleteConfirmed` 方法。

        public ActionResult Edit(string partitionKey, string rowKey)
        {
            var message = FindRow(partitionKey, rowKey);
            if (message.Status != "Pending")
            {
                throw new Exception("Message can't be edited because it isn't in Pending status.");
            }

            var lists = GetListNames();
            ViewBag.ListName = new SelectList(lists, "ListName", "Description", message.ListName);
            return View(message);
        }

    在 `HttpPost Edit` 方法中，仅当用户选择上载新文件时，代码才保存新 Blob。以下代码省略了方法的并发处理部分，该部分与你此前看到的针对 `MailingList` 控制器的部分是相同的。

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Edit(string partitionKey, string rowKey, Message editedMsg,
            DateTime scheduledDate, HttpPostedFileBase httpFile, HttpPostedFileBase txtFile)
        {
            if (ModelState.IsValid)
            {
                var excludePropLst = new List<string>();
                excludePropLst.Add("PartitionKey");
                excludePropLst.Add("RowKey");

                if (httpFile == null)
                {
                    // They didn't enter a path or navigate to a file, so don't update the file.
                    excludePropLst.Add("HtmlPath");
                }
                else
                {
                    // They DID enter a path or navigate to a file, assume it's changed.
                    SaveBlob(editedMsg.MessageRef + ".htm", httpFile);
                }

                if (txtFile == null)
                {
                    excludePropLst.Add("TextPath");
                }
                else
                {
                    SaveBlob(editedMsg.MessageRef + ".txt", txtFile);
                }

                string[] excludeProperties = excludePropLst.ToArray();

                try
                {
                    UpdateModel(editedMsg, string.Empty, null, excludeProperties);
                    if (editedMsg.PartitionKey == partitionKey)
                    {
                        // Keys didn't change -- update the row.
                        var replaceOperation = TableOperation.Replace(editedMsg);
                        messageTable.Execute(replaceOperation);
                    }
                    else
                    {
                        // Partition key changed -- delete and insert the row.
                        // (Partition key has scheduled date which may be changed;
                        // row key has MessageRef which does not change.)
                        var deleteOperation = TableOperation.Delete(new Message { PartitionKey = partitionKey, RowKey = rowKey, ETag = editedMsg.ETag });
                        messageTable.Execute(deleteOperation);
                        var insertOperation = TableOperation.Insert(editedMsg);
                        messageTable.Execute(insertOperation);
                    }
                    return RedirectToAction("Index");
                }

    如果更改了计划的日期，则会更改分区键，则必须删除一个行并插入一个行。这不能在事务中完成，因为会影响多个分区。

    `HttpPost Delete` 方法在删除表中的行时删除 Blob：

        [HttpPost, ActionName("Delete")]
        public ActionResult DeleteConfirmed(String partitionKey, string rowKey)
        {
            // Get the row again to make sure it's still in Pending status.
            var message = FindRow(partitionKey, rowKey);
            if (message.Status != "Pending")
            {
                throw new Exception("Message can't be deleted because it isn't in Pending status.");
            }

            DeleteBlob(message.MessageRef + ".htm");
            DeleteBlob(message.MessageRef + ".txt");
            var deleteOperation = TableOperation.Delete(message);
            messageTable.Execute(deleteOperation);
            return RedirectToAction("Index");
        }

        private void DeleteBlob(string blobName)
        {
            var blob = blobContainer.GetBlockBlobReference(blobName);
            blob.Delete();
        }

### 添加邮件 MVC 视图

1.  在**解决方案资源管理器**中，在 MVC 项目的 *Views* 文件夹中创建一个新的文件夹，然后将其命名为 `Message`。

2.  右键单击新的*“Views\\Message”*文件夹，然后选择**“添加现有项”**。

3.  导航到已下载示例应用程序的文件夹，选择 *Views\\Message* 文件夹中的所有五个 .cshtml 文件，然后单击“添加”。

4.  打开 *Edit.cshtml* 文件并检查代码。

        @model MvcWebRole.Models.Message

        @{
            ViewBag.Title = "Edit Message";
        }

        <h2>Edit Message</h2>

        @using (Html.BeginForm("Edit", "Message", FormMethod.Post, new { enctype = "multipart/form-data" }))
        {
            @Html.AntiForgeryToken()
            @Html.ValidationSummary(true)
            @Html.HiddenFor(model => model.ETag)
            <fieldset>
                <legend>Message</legend>
                @Html.HiddenFor(model => model.MessageRef)
                @Html.HiddenFor(model => model.PartitionKey)
                @Html.HiddenFor(model => model.RowKey)
                <div class="editor-label">
                    @Html.LabelFor(model => model.ListName, "MailingList")
                </div>
                <div class="editor-field">
                    @Html.DropDownList("ListName", String.Empty)
                    @Html.ValidationMessageFor(model => model.ListName)
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.SubjectLine)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.SubjectLine)
                    @Html.ValidationMessageFor(model => model.SubjectLine)
                </div>
                <div class="editor-label">
                    HTML Path: Leave blank to keep current HTML File.
                </div>
                <div class="editor-field">
                    <input type="file" name="file" />
                </div>
                <div class="editor-label">
                    Text Path: Leave blank to keep current Text File.
                </div>
                <div class="editor-field">
                    <input type="file" name="TxtFile" />
                </div>
                <div class="editor-label">
                    @Html.LabelFor(model => model.ScheduledDate)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.ScheduledDate)
                    @Html.ValidationMessageFor(model => model.ScheduledDate)
                </div>
                <div class="display-label">
                    @Html.DisplayNameFor(model => model.Status)
                </div>
                <div class="editor-field">
                    @Html.EditorFor(model => model.Status)
                </div>
               <p>
                    <input type="submit" value="Save" />
                </p>
            </fieldset>
        }

        <div>
            @Html.ActionLink("Back to List", "Index")
        </div>

        @section Scripts {
            @Scripts.Render("~/bundles/jqueryval")
        }

    `HttpPost Edit` 方法需要分区键和行键，因此代码在隐藏字段中添加这些键。隐藏字段以前在 `Subscriber` 控制器中并不需要，这是因为：(a) `Subscriber` 模型中的 `ListName` 和 `EmailAddress` 属性会更新 `PartitionKey` 和 `RowKey` 属性；(b) `ListName` 和 `EmailAddress` 属性包含在“编辑”视图的 `EditorFor` 帮助器中。当 `Subscriber` 模型的 MVC 模型联编程序更新 `ListName` 属性时，`PartitionKey` 属性会自动进行更新，而当 MVC 模型联编程序更新 `Subscriber` 模型中的 `EmailAddress` 属性时，`RowKey` 属性会自动进行更新。在 `Message` 模型中，映射到分区键和行键的字段不是可编辑字段，因此不能那样进行设置。

    隐藏字段也是针对 `MessageRef` 属性提供的。该字段是与分区键相同的值，但提供该字段是为了使 `HttpPost Edit` 方法中的代码更清楚明了。提供 `MessageRef` 隐藏字段可以让 `HttpPost Edit` 方法中的代码通过该名称引用 `MessageRef` 值，同时为 Blob 构造文件名。

5.  打开 *Index.cshtml* 文件并检查代码。

        @model IEnumerable<MvcWebRole.Models.Message>

        @{
            ViewBag.Title = "Messages";
        }

        <h2>Messages</h2>

        <p>
            @Html.ActionLink("Create New", "Create")
        </p>
        <table>
            <tr>
                <th>
                    @Html.DisplayNameFor(model => model.ListName)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.SubjectLine)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.ScheduledDate)
                </th>
                <th>
                    @Html.DisplayNameFor(model => model.Status)
                </th>
                <th></th>
            </tr>
            @foreach (var item in Model)
            {
                <tr>
                    <td>
                        @Html.DisplayFor(modelItem => item.ListName)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.SubjectLine)
                    </td>
                    <td>
                        @Html.DisplayFor(modelItem => item.ScheduledDate)
                    </td>
                    <td>
                        @item.Status
                    </td>
                    <td>
                        @if (item.Status == "Pending")
                        {
                            @Html.ActionLink("Edit", "Edit", new { PartitionKey = item.PartitionKey, RowKey = item.RowKey })  @: | 
                            @Html.ActionLink("Delete", "Delete", new { PartitionKey = item.PartitionKey, RowKey = item.RowKey }) @: |
                        }
                        @Html.ActionLink("Details", "Details", new { PartitionKey = item.PartitionKey, RowKey = item.RowKey })
                    </td>
                </tr>
            }

        </table>

    此处与其他**“索引”**视图的区别是，**“编辑”**链接和**“删除”**链接仅仅是针对处于 `Pending` 状态的邮件显示的：

        @if (item.Status == "Pending")
        {
            @Html.ActionLink("Edit", "Edit", new { PartitionKey = item.PartitionKey, RowKey = item.RowKey })  @: | 
            @Html.ActionLink("Delete", "Delete", new { PartitionKey = item.PartitionKey, RowKey = item.RowKey }) @: |
        }

    这样可以在辅助角色 A 已开始处理邮件的情况下，防止用户对邮件进行更改。

    其他视图包含的代码类似于**“编辑”**视图或其他你所看到的针对其他控制器的视图。

### 测试应用程序

1.  通过按 CTRL+F5 运行项目，然后单击**“邮件”**。

    ![空邮件索引页][空邮件索引页]

2.  使用**“创建”**功能添加一些邮件列表，然后尝试**“编辑”**和**“删除”**功能，确保其有效。

    ![包含行的订户索引页][3]

## <a name="unsubscribe"></a><span class="short-header">取消订阅</span>创建和测试“取消订阅”控制器和视图

接下来，你将实现用于取消订阅过程的 UI。

**注意：**本教程只生成用于取消订阅过程而非订阅过程的控制器。如[第一个教程][本系列第一个教程]所述，在我们针对服务方法实施适当的安全措施之前，用于订阅过程的 UI 和服务方法省略不讲。在实施安全措施之前，你可以使用**“订户”**管理员页通过电子邮件地址来订阅列表。

### 将“取消订阅”视图模型添加到 Models 文件夹

`UnsubscribeVM` 视图模型用于在 `Unsubscribe` 控制器及其视图之间传递数据。

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 `Models` 文件夹，然后选择**“添加现有项”**。

2.  导航到下载了示例应用程序的文件夹，选择 Models 文件夹中的 `UnsubscribeVM.cs` 文件，然后单击“添加”。

3.  打开 `UnsubscribeVM.cs` 并检查代码。

        public class UnsubscribeVM
        {
            public string EmailAddress { get; set; }
            public string ListName { get; set; }
            public string ListDescription { get; set; }
            public string SubscriberGUID { get; set; }
            public bool? Confirmed { get; set; }
        }

    取消订阅链接包含 `SubscriberGUID`。该值用于从 `MailingList` 表获取电子邮件地址、列表名称以及列表说明。视图显示电子邮件地址以及要从其取消订阅的列表的说明，并显示一个“确认”按钮，用户必须单击该按钮才能完成取消订阅过程。

### 添加“取消订阅”控制器

1.  在**解决方案资源管理器**中，右键单击 MVC 项目中的 `Controllers` 文件夹，然后选择**“添加现有项”**。

2.  导航到你在其中下载了示例应用程序的文件夹，选择 *Controllers* 文件夹中的 *UnsubscribeController.cs* 文件，然后单击“添加”。

3.  打开 *UnsubscribeController.cs* 并检查代码。

    此控制器有一个 `HttpGet Index` 方法，用于显示初始的取消订阅页，以及一个 `HttpPost Index` 方法，用于处理**“确认”**或**“取消”**按钮。

    `HttpGet Index` 方法在查询字符串中使用 GUID 和列表名称来获取订户的 `MailingList` 表行。然后，它将视图所需的所有信息置于视图模型中，并显示**“取消订阅”**页。它将 `Confirmed` 属性设置为 null，目的是告诉视图显示初始版本的**“取消订阅”**页。

         public ActionResult Index(string id, string listName)
         {
             if (string.IsNullOrEmpty(id) == true || string.IsNullOrEmpty(listName))
             {
                 ViewBag.errorMessage = "Empty subscriber ID or list name.";
                 return View("Error");
             }
             string filter = TableQuery.CombineFilters(
                 TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, listName),
                 TableOperators.And,
                 TableQuery.GenerateFilterCondition("SubscriberGUID", QueryComparisons.Equal, id));
             var query = new TableQuery<Subscriber>().Where(filter);
             var subscriber = mailingListTable.ExecuteQuery(query).ToList().Single();
             if (subscriber == null)
             {
                 ViewBag.Message = "You are already unsubscribed";
                 return View("Message");
             }
             var unsubscribeVM = new UnsubscribeVM();
             unsubscribeVM.EmailAddress = MaskEmail(subscriber.EmailAddress);
             unsubscribeVM.ListDescription = FindRow(subscriber.ListName, "mailinglist").Description;
             unsubscribeVM.SubscriberGUID = id;
             unsubscribeVM.Confirmed = null;
             return View(unsubscribeVM);
         }

    注意：SubscriberGUID 不在分区键或行键中，因此当分区大小（邮件列表中的电子邮件地址数目）增加时，此查询的性能会下降。有关如何采用备用方法使此查询更具伸缩性的详细信息，请参阅[本系列第一个教程][本系列第一个教程]。

    `HttpPost Index` 方法再一次使用 GUID 和列表名称来获取订户信息并填充视图模型属性。然后，如果单击了**“确认”**按钮，它会删除 `MailingList` 表中的订户行。如果按下了**“确认”**按钮，则还会将 `Confirm` 属性设置为 `true`，否则会将 `Confirm` 属性设置为 `false`。`Confirm` 属性的值用来告诉视图显示**“取消订阅”**页的已确认版本或已删除版本。

        [HttpPost] 
        [ValidateAntiForgeryToken]
        public ActionResult Index(string subscriberGUID, string listName, string action)
        {
            string filter = TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, listName),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("SubscriberGUID", QueryComparisons.Equal, subscriberGUID));
            var query = new TableQuery<Subscriber>().Where(filter);
            var subscriber = mailingListTable.ExecuteQuery(query).ToList().Single();

            var unsubscribeVM = new UnsubscribeVM();
            unsubscribeVM.EmailAddress = MaskEmail(subscriber.EmailAddress);
            unsubscribeVM.ListDescription = FindRow(subscriber.ListName, "mailinglist").Description;
            unsubscribeVM.SubscriberGUID = subscriberGUID;
            unsubscribeVM.Confirmed = false;

            if (action == "Confirm")
            {
                unsubscribeVM.Confirmed = true;
                var deleteOperation = TableOperation.Delete(subscriber);
                mailingListTable.Execute(deleteOperation);
            }

            return View(unsubscribeVM);
        }

### 创建 MVC 视图

1.  在**解决方案资源管理器**中，在 MVC 项目的 *Views* 文件夹中创建一个新的文件夹，然后将其命名为 *Unsubscribe*。

2.  右键单击新的*“Views\\Unsubscribe”*文件夹，然后选择**“添加现有项”**。

3.  导航到你在其中下载了示例应用程序的文件夹，选择 *Views\\Unsubscribe* 文件夹中的 *Index.cshtml* 文件，然后单击“添加”。

4.  打开 *Index.cshtml* 文件并检查代码。

        @model MvcWebRole.Models.UnsubscribeVM

        @{
            ViewBag.Title = "Unsubscribe";
            Layout = null;
        }

        <h2>Email List Subscription Service</h2>

        @using (Html.BeginForm()) {
            @Html.AntiForgeryToken()
            @Html.ValidationSummary(true)
            <fieldset>
                <legend>Unsubscribe from Mailing List</legend>
                @Html.HiddenFor(model => model.SubscriberGUID)
                @Html.HiddenFor(model => model.EmailAddress)
                @Html.HiddenFor(model => model.ListName)
                @if (Model.Confirmed == null) {
                    <p>
                        Do you want to unsubscribe  @Html.DisplayFor(model => model.EmailAddress) from:  @Html.DisplayFor(model => model.ListDescription)?
                   </p>
                    <br />
                    <p>
                        <input type="submit" value="Confirm" name="action"/> 
                        &nbsp; &nbsp;
                        <input type="submit" value="Cancel" name="action"/>
                    </p>
                }
                @if (Model.Confirmed == false) {
                    <p>
                        @Html.DisplayFor(model => model.EmailAddress)  will NOT be unsubscribed from: @Html.DisplayFor(model => model.ListDescription).
                    </p>
                }
                @if (Model.Confirmed == true) {
                    <p>
                        @Html.DisplayFor(model => model.EmailAddress)  has been unsubscribed from:  @Html.DisplayFor(model => model.ListDescription).
                    </p>
                }
            </fieldset>
        }

        @section Scripts {
            @Scripts.Render("~/bundles/jqueryval")
        }

    `Layout = null` 行指定不应使用 \_Layout.cshtml 文件来显示此页。**“取消订阅”**页显示不带管理员页所用页眉和页脚的非常简单的 UI。

    在页面正文中，`Confirmed` 属性用于确定将要显示在页面上的内容：如果属性为 null，则显示**“确认”**和**“取消”**按钮；如果属性为 true，则显示“取消订阅已确认”邮件；如果属性为 false，则显示“取消订阅已取消”邮件。

### 测试应用程序

1.  通过按 CTRL-F5 运行项目，然后单击**“订户”**。

2.  单击**“创建”**，创建一个针任何邮件列表的新订户，该邮件列表是你此前进行测试时创建的。

    让浏览器窗口在**“订户索引”**页上处于打开状态。

3.  打开 Azure 存储资源管理器，然后选择你的测试用的存储帐户。

4.  单击“存储类型”下的“表”，选择 **MailingList** 表，然后单击“查询”。

5.  双击所添加的订户行。

    ![Azure 存储空间资源管理器][Azure 存储空间资源管理器]

6.  在**“编辑实体”**对话框中，选择并复制 `SubscriberGUID` 值。

    ![Azure 存储空间资源管理器][4]

7.  切换回你的浏览器窗口。在浏览器的地址栏中，将 URL 中的“Subscriber”更改为“unsubscribe?ID=[guidvalue]&listName=[listname]”，其中 [guidvalue] 是你从 Azure 存储资源管理器中复制的 GUID，[listname] 是邮件列表的名称。例如：

        http://127.0.0.1/unsubscribe?ID=b7860242-7c2f-48fb-9d27-d18908ddc9aa&listName=contoso1

    将显示要求用户确认的那个**“取消订阅”**页：

    ![取消订阅页][取消订阅页]

8.  单击**“确认”**，你将看到一个确认，指出已取消通过电子邮件地址进行的订阅。

    ![取消订阅已确认页][取消订阅已确认页]

9.  返回到“订户索引”页，验证订户行是否已不再存在。

## <a name="alternativearchitecture"></a><span class="short-header">替代体系结构</span>（可选）构建替代体系结构

如果你希望构建替代体系结构，也就是说，在 Azure 网站而非 Azure 云服务 Web 角色中运行 Web UI，则应对说明进行以下更改。

-   当你创建解决方案时，请先创建 **ASP.NET MVC 4 Web 应用程序**项目，然后向解决方案添加带辅助角色的 **Azure 云服务**项目。

-   将 Azure 存储空间连接字符串存储在 Web.config 文件而非云服务设置文件中。（这仅适用于 Azure 网站。如果你尝试使用 Web.config 文件来保存 Azure 云服务 Web 角色中的存储空间连接字符串，你会收到一个 HTTP 500 错误。）

将一个新的名为 `StorageConnectionString` 的连接字符串添加到 *Web.config* 文件中，如以下示例所示：

           <connectionStrings>
              <add name="DefaultConnection" connectionString="Data Source=(LocalDb)\v11.0;Initial Catalog=aspnet-MvcWebRole-20121010185535;Integrated Security=SSPI;AttachDBFilename=|DataDirectory|\aspnet-MvcWebRole-20121010185535.mdf" providerName="System.Data.SqlClient" />
              <add name="StorageConnectionString" connectionString="DefaultEndpointsProtocol=https;AccountName=[accountname];AccountKey=[primarykey]" />
           </connectionStrings>

从 [Azure 管理门户][Azure 管理门户]获取连接字符串的值：选择**“存储”**选项卡和你的存储帐户，然后单击页面底部的**“管理密钥”**。

-   当你看到代码中存在 `RoleEnvironment.GetConfigurationSettingValue("StorageConnectionString")` 时，将其替换为 `ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString`。

## <a name="nextsteps"></a><span class="short -header">后续步骤</span>后续步骤

如[本系列第一个教程][本系列第一个教程]所述，在本教程中，在我们实施共享机密来确保 ASP.NET Web API 服务方法的安全之前，我们不会详细讲述如何构建订阅过程。不过，使用 IP 限制也可以保护服务方法，而且你可以通过复制已下载项目中的以下文件来添加订阅功能。

适用于 ASP.NET Web API 服务方法：

-   Controllers\\SubscribeAPI.cs

适用于订户在单击电子邮件中通过服务方法生成的“确认”链接后获取的网页：

-   Models\\SubscribeVM.cs
-   Controllers\\SubscribeController.cs
-   Views\\Subscribe\\Index.cshtml

在[下一教程][下一教程]中，你将对辅助角色 A 这个用于计划电子邮件发送的辅助角色执行配置和编程操作。

有关如何使用 Azure 存储表、队列和 Blob 的其他资源的链接，请参阅[本系列最后一个教程][5]。

<div><a href="/zh-cn/develop/net/tutorials/multi-tier-web-site/4-worker-role-a/" class="site-arrowboxcta download-cta">教程 4</a></div>

  [本系列第一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
  [创建 Visual Studio 解决方案]: #cloudproject
  [配置跟踪]: #tracing
  [添加代码来有效地处理重新启动。]: #restarts
  [更新存储客户端库 NuGet 包]: #updatescl
  [添加对 SCL 1.7 程序集的引用]: #addref2
  [添加可在 Application\_Start 方法中创建表、队列和 Blob 容器的代码]: #createifnotexists
  [创建和测试邮件列表]: #mailinglist
  [配置 Web 角色以使用你的 Azure 存储测试帐户]: #configurestorage
  [创建和测试“订户”控制器和视图]: #subscriber
  [创建和测试“邮件”控制器和视图]: #message
  [创建和测试“取消订阅”控制器和视图]: #unsubscribe
  [（可选）构建替代体系结构]: #alternativearchitecture
  [后续步骤]: #nextsteps
  [“新建项目”菜单]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-file-new-project.png
  [“新建项目”对话框]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-new-cloud-project.png
  [“新建 Azure 云项目”对话框]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-new-cloud-service-dialog.png
  [“新建 Azure 云项目”对话框 - 重命名 Web 角色]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-new-cloud-service-dialog-rename.png
  [“新建 Azure 云项目”对话框 - 添加辅助角色]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-new-cloud-service-add-worker-a.png
  [1]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-new-mvc4-project.png
  [解决方案资源管理器中的 \_Layout.cshtml]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-opening-layout-cshtml.png
  [\_Layout.cshtml 中的标题和页眉]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-title-and-logo-in-layout.png
  [\_Layout.cshtml 中的菜单]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-menu-in-layout.png
  [\_Layout.cshtml 中的页脚]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-footer-in-layout.png
  [主页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-home-page-before-adding-controllers.png
  [系统任务栏中的计算模拟器]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-compute-emulator-icon.png
  [第二个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/2-download-and-run/
  [重新启动角色实例进行 OS 升级]: http://blogs.msdn.com/b/kwill/archive/2012/09/19/role-instance-restarts-due-to-os-upgrades.aspx
  [按需传输]: http://msdn.microsoft.com/zh-cn/library/windowsazure/gg433075.aspx
  [dbgview]: http://technet.microsoft.com/zh-cn/sysinternals/bb896647.aspx
  [菜单中的“管理解决方案的 NuGet 包”]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-manage-nuget-for-solution.png
  [“管理 NuGet 包”对话框中的 Azure 存储包]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-update-storage-nuget-pkg.png
  [在“选择项目”对话框中选择这两个项目]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-nuget-select-projects.png
  [Microsoft.WindowsAzure.Storage.Table.DataServices]: http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.dataservices.aspx
  [本系列最后一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/
  [使用.NET 4.5 的异步支持以避免阻塞调用]: http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices#async
  [rightClick]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-4.png
  [将现有项添加到“Models”文件夹]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-add-existing-item-to-models.png
  [TableEntity]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.storage.table.tableentity.aspx
  [DynamicTableEntity]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.storage.table.dynamictableentity.aspx
  [Azure 存储客户端库 2.0 表深入探讨]: http://blogs.msdn.com/b/windowsazurestorage/archive/2012/11/06/windows-azure-storage-client-library-2-0-tables-deep-dive.aspx
  [了解表服务数据模型]: http://msdn.microsoft.com/zh-cn/library/windowsazure/dd179338.aspx
  [PartitionKey 或 RowKey 中的 % 字符]: http://blogs.msdn.com/b/windowsazurestorage/archive/2012/05/28/partitionkey-or-rowkey-containing-the-percent-character-causes-some-windows-azure-tables-apis-to-fail.aspx
  [将现有项添加到 Controllers 文件夹]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-add-existing-item-to-controllers.png
  [TableOperation.Retrieve]: http://msdn.microsoft.com/zh-cn/library/windowsazure/microsoft.windowsazure.storage.table.tableoperation.retrieve.aspx
  [将现有项添加到 Views 文件夹]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-add-existing-item-to-views.png
  [Web 角色属性]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-mvcwebrole-properties-menu.png
  [右键单击属性]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-elip.png
  [2]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-enter.png
  [发布设置]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-3.png
  [选择存储帐户]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-5.png
  [无效的服务配置和定义文件错误]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-er1.png
  [空的 MailingList 索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-mailing-list-empty-index-page.png
  [包含行的 MailingList 索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-mailing-list-index-page.png
  [如何充分利用 Azure 表]: http://blogs.msdn.com/b/windowsazurestorage/archive/2010/11/06/how-to-get-most-out-of-windows-azure-tables.aspx
  [Azure 表：做好处理继续标记的足够准备]: http://blog.smarx.com/posts/windows-azure-tables-expect-continuation-tokens-seriously
  [空的订户索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-subscribers-empty-index-page.png
  [包含行的订户索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-subscribers-index-page.png
  [空邮件索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-message-empty-index-page.png
  [3]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-message-index-page.png
  [Azure 存储空间资源管理器]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-ase-unsubscribe.png
  [4]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-ase-edit-entity-unsubscribe.png
  [取消订阅页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-unsubscribe-query-page.png
  [取消订阅已确认页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-web-role/mtas-unsubscribe-confirmation-page.png
  [Azure 管理门户]: http://manage.windowsazure.cn
  [下一教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/4-worker-role-a/
  [5]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/#nextsteps
