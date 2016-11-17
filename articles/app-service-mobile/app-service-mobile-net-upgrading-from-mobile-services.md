<properties
	pageTitle="从移动服务升级到 Azure 应用服务"
	description="了解如何轻松将移动服务应用程序升级到应用服务移动应用"
	services="app-service\mobile"
	documentationCenter=""
	authors="mattchenderson"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="07/25/2016"
	wacn.date="09/26/2016"
	ms.author="adrianha"/>

# 将现有 .NET Azure 移动服务升级到应用服务

应用服务移动应用是使用 Azure 构建移动应用程序的新方式。有关的详细信息，请参阅 [What are Mobile Apps?]（什么是移动应用？）

本主题介绍如何将现有 .NET 后端应用程序从 Azure 移动服务升级到新的应用服务移动应用。执行此升级时，现有移动服务应用程序可以继续正常运行。如果需要升级 Node.js 后端应用程序，请参阅 [Upgrading your Node.js Mobile Services](/documentation/articles/app-service-mobile-node-backend-upgrading-from-mobile-services/)（升级 Node.js 移动服务）。

将某个移动后端升级到 Azure 应用服务后，该后端即可访问所有应用服务功能，同时会根据[应用服务定价]而不是移动服务定价进行计费。

##迁移与升级

[AZURE.INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

>[AZURE.TIP] 建议在升级之前先[执行迁移](/documentation/articles/app-service-mobile-migrating-from-mobile-services/)。这样，就能在同一个应用服务计划中放置两个版本的应用程序，且无需支付额外的费用。

###移动应用 .NET 服务器 SDK 改进

升级到新[移动应用 SDK](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/) 可获得以下好处：

- NuGet 依赖项有更大的灵活性。宿主环境不再提供其自身的 NuGet 包版本，因此可以使用替代的兼容版本。但是，如果移动服务器 SDK 或依赖项有新的关键 Bug 修复或安全更新，则必须手动更新服务。

- 移动 SDK 有更大的灵活性。可以明确控制要设置哪些功能和路由，例如身份验证、表 API 和推送注册终结点。有关详细信息，请参阅 [How to use the .NET server SDK for Azure Mobile Apps](/documentation/articles/app-service-mobile-net-upgrading-from-mobile-services/#server-project-setup)（如何使用适用于 Azure 移动应用的 .NET 服务器 SDK）。

- 支持其他 ASP.NET 项目类型和路由。现在，可以在与移动后端项目相同的项目中托管 MVC 和 Web API 控制器。

- 支持新的应用服务身份验证功能，允许跨 Web 应用和移动应用使用常见的身份验证配置。

##<a name="overview"></a>基本升级概述

在许多情况下，只需切换到新的移动应用服务器 SDK 并将代码重新发布到新的移动应用实例，即可完成升级。但在某些情况下则需要一些额外的配置，例如高级身份验证方案和使用计划作业。后续部分将逐一介绍。

>[AZURE.TIP] 建议先阅读并充分了解本主题的余下内容，然后再开始升级。请记下以下概括的所有功能。

移动服务客户端 SDK 与新的移动应用服务器 SDK **不**兼容。为了提供应用程序的服务连续性，不应该将更改发布到当前正在为发布的客户端提供服务的站点。而应该创建新的移动应用作为副本。可以在同一个应用服务计划中放置此应用程序，以免产生额外的财务成本。

这样，就会获得两个版本的应用程序：一个保持不变并为已发布的现有应用程序提供服务，另一个则可升级且目标为新的客户端版本。可根据自己的步调移动和测试代码，但应确保所做的任何 bug 修复都已应用到这两个版本。觉得已将所需数量的现有客户端应用升级到最新版本后，可以根据需要删除已迁移的原始应用。

完整的升级过程大致如下：

1. 创建新的移动应用
2. 更新项目以使用新的服务器 SDK
3. 发布新版客户端应用程序
4. （可选）删除已迁移的原始实例

##<a name="mobile-app-version"></a>创建第二个应用程序实例
升级的第一个步骤是创建用于托管新版应用程序的移动应用资源。如果已迁移现有移动服务，则需要在同一个托管计划中创建此版本。打开 [Azure 门户预览]，导航到已迁移的应用程序。记下运行该应用程序的应用服务计划。

接下来，根据 [.NET 后端创建说明](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#create-app)创建第二个应用程序实例。当系统提示选择应用服务计划或“托管计划”时，请选择已迁移的应用程序的计划。

可能需要使用与移动服务中相同的数据库和通知中心。可以复制这些值，方法是打开 [Azure 门户预览]，导航到原始应用程序，然后单击“设置”>“应用程序设置”。在“连接字符串”下，复制 `MS_NotificationHubConnectionString` 和 `MS_TableConnectionString`。导航到新的升级站点并粘贴这些值，覆盖任何现有值。针对应用所需的任何其他应用程序设置重复此过程。如果未使用迁移的服务，可以从 [Azure 经典管理门户]上“移动服务”部分中的“配置”选项卡读取连接字符串和应用设置。

为应用程序制作 ASP.NET 项目的副本，然后将其发布到新站点。通过使用新 URL 更新的客户端应用程序副本验证一切是否正常工作。

## 更新服务器项目

移动应用包含一个新的[移动应用服务器 SDK]，该 SDK 提供许多与移动服务运行时相同的功能。首先，应该删除对移动服务包的所有引用。在 NuGet 包管理器中，搜索 `WindowsAzure.MobileServices.Backend`。大多数应用在此处有多个对应的包，包括 `WindowsAzure.MobileServices.Backend.Tables` 和 `WindowsAzure.MobileServices.Backend.Entity`。在这种情况下，请从依赖性树中级别最低的包（例如 `Entity`）开始删除包。出现提示时，请不要删除所有依赖包。重复此过程，直到已删除 `WindowsAzure.MobileServices.Backend` 本身。

此时，项目将不再引用移动服务 SDK。

接下来，添加对移动应用 SDK 的引用。对于这种升级，大多数开发人员都会下载并安装 `Microsoft.Azure.Mobile.Server.Quickstart` 包，因为这可以提取整个所需集。

到时将有不少因 SDK 之间的差异而发生的编译器错误，但这些错误都很容易处理，本部分的余下内容将予以说明。

### <a name="server-project-setup"></a> 基本配置

然后，在 WebApiConfig.cs 中，将：

        // Use this class to set configuration options for your mobile service
        ConfigOptions options = new ConfigOptions();

        // Use this class to set WebAPI configuration options
        HttpConfiguration config = ServiceConfig.Initialize(new ConfigBuilder(options));

替换为

        HttpConfiguration config = new HttpConfiguration();
        new MobileAppConfiguration()
            .UseDefaultConfiguration()
        .ApplyTo(config);

>[AZURE.NOTE] 如果想要详细了解新的 .NET 服务器 SDK 以及如何在应用中添加/删除功能，请参阅 [How to use the .NET server SDK]（如何使用 .NET 服务器 SDK）主题。

如果应用使用身份验证功能，则还需要注册 OWIN 中间件。在此情况下，应该将上述配置代码移入新的 OWIN 启动类。

1. 如果 NuGet 包 `Microsoft.Owin.Host.SystemWeb` 尚未包含在项目中，请添加该包。
2. 在 Visual Studio 中，右键单击项目，然后选择“添加”->“新建项”。选择“Web”->“常规”->“OWIN 启动类”。
3. 将 MobileAppConfiguration 的上述代码从 `WebApiConfig.Register()` 移到新启动类的 `Configuration()` 方法。

确保 `Configuration()` 方法的末尾为：

        app.UseWebApi(config)
        app.UseAppServiceAuthentication(config);

存在其他一些与身份验证相关的更改，下面的完全身份验证部分将介绍这些内容。

### 处理数据

在移动服务中，移动应用名称用作 Entity Framework 设置中的默认架构名称。

若要确保架构与以前引用的一样，请使用以下代码设置 DbContext 中适用于应用程序的架构：

        string schema = System.Configuration.ConfigurationManager.AppSettings.Get("MS_MobileServiceName");

如果执行上述操作，请确保设置 MS\_MobileServiceName。如果应用程序以前已自定义此架构，则也可以提供其他架构名称。

### 系统属性

#### 命名

在 Azure 移动服务服务器 SDK 中，系统属性始终包含属性的双下划线 (`__`) 前缀：

- \_\_createdAt
- \_\_updatedAt
- \_\_deleted
- \_\_version

移动服务客户端 SDK 具有特殊的逻辑用于分析这种格式的系统属性。

在 Azure 移动应用中，系统属性不再使用特殊格式，而是使用以下名称：

- createdAt
- updatedAt
- deleted
- 版本

移动应用客户端 SDK 使用新系统属性名称，因此不需要对客户端代码进行任何更改。但是，如果要直接对服务进行 REST 调用，则应该相应地更改查询。

#### 本地存储

更改系统属性的名称意味着用于移动服务的脱机同步本地数据库与移动应用不兼容。在将挂起的更改发送到服务器之前，应尽可能避免将客户端应用从移动服务升级到移动应用。然后，升级的应用应使用新的数据库文件名。

如果客户端应用是从移动服务升级到移动应用，但同时在操作队列中有挂起的脱机更改，则系统数据库必须更新才能使用新的列名。在 iOS 上，可以使用轻量迁移更改列名，从而做到这一点。在 Android 和 .NET 托管客户端上，应该编写自定义 SQL 来重命名数据对象表的列。

在 iOS 上，应该根据以下列表更改数据实体的核心数据架构。请注意，属性 `createdAt`、`updatedAt` 和 `version` 不再有 `ms_` 前缀：

| 属性 | 类型 | 注意 |
|---------- |  ------ | -----------------------------------------------------|
| id | 字符串（标记为必需） | 远程存储中的主键 |
| createdAt | 日期 | （可选）映射到 createdAt 系统属性 |
| updatedAt | 日期 | （可选）映射到 updatedAt 系统属性 |
| 版本 | 字符串 | （可选）用于检测冲突，映射到版本 |

#### 查询系统属性

在 Azure 移动服务中，默认不会发送系统属性，而是仅当使用查询字符串 `__systemProperties` 请求时才发送系统属性。相反，在 Azure 移动应用中，**始终会选择**系统属性，因为它们是服务器 SDK 对象模型的一部分。

此项更改主要影响域管理器的自定义实现，例如 `MappedEntityDomainManager` 的扩展。在移动服务中，如果客户端永远不请求任何系统属性，则可以使用 `MappedEntityDomainManager`，它实际上不会映射所有属性。但是，在 Azure 移动应用中，这些未映射的属性将导致 GET 查询出错。

解决此问题的最简单方法是修改 DTO，使它们继承自 `ITableData` 而不是 `EntityData`。然后，将 `[NotMapped]` 属性添加到应省略的字段。

例如，以下示例定义 `TodoItem` 且不使用任何系统属性：

    using System.ComponentModel.DataAnnotations.Schema;

    public class TodoItem : ITableData
    {
        public string Text { get; set; }

        public bool Complete { get; set; }

        public string Id { get; set; }

        [NotMapped]
        public DateTimeOffset? CreatedAt { get; set; }

        [NotMapped]
        public DateTimeOffset? UpdatedAt { get; set; }

        [NotMapped]
        public bool Deleted { get; set; }

        [NotMapped]
        public byte[] Version { get; set; }
    }

注意：如果收到有关 `NotMapped` 的错误，请添加对程序集 `System.ComponentModel.DataAnnotations` 的引用。

### CORS

移动服务通过包装 ASP.NET CORS 解决方案，融入了对 CORS 的部分支持。此包装层现已删除，使开发人员拥有更多控制权，因此，可以直接利用 [ASP.NET CORS 支持](http://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api)。

是否使用 CORS 的主要考虑范畴是必须允许 `eTag` 和 `Location` 标头，客户端 SDK 才能正常工作。

### 推送通知
对于推送，可能会发现服务器 SDK 中遗漏的主项是 PushRegistrationHandler 类。注册在移动应用中的处理方式稍有不同，默认情况下启用不带标记的注册。可以使用自定义 API 实现标记管理。有关详细信息，请参阅[注册标记](/documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/#tags)说明。

### 计划的作业
移动应用中未内置计划的作业，因此在 .NET 后端中的任何现有作业都必须单独升级。一种做法是在移动应用代码站点上创建计划的 [Web 作业]。也可以设置用于保存作业代码的控制器，并设置按预期计划在终结点上运行的 [Azure 计划程序]。

### 其他更改
移动客户端要使用的所有 ApiControllers 现在必须具有 `[MobileAppController]` 属性。默认不再包含此属性，因此，其他 ApiControllers 不受移动格式化程序的影响。

`ApiServices` 对象不再是 SDK 的一部分。若要访问移动应用设置，可以使用以下代码：

    MobileAppSettingsDictionary settings = this.Configuration.GetMobileAppSettingsProvider().GetMobileAppSettings();

同样，现在可以使用标准的 ASP.NET 跟踪写入实现日志记录：

    ITraceWriter traceWriter = this.Configuration.Services.GetTraceWriter();
    traceWriter.Info("Hello, World");  

##<a name="authentication"></a>身份验证注意事项

移动服务的身份验证组件现已移入应用服务身份验证/授权功能。可以阅读 [Add authentication to your mobile app](/documentation/articles/app-service-mobile-ios-get-started-users/)（将身份验证添加到移动应用）主题，了解如何为站点启用此功能。

对于某些提供程序（例如 AAD），应该可以利用复制应用程序的现有注册。只需导航到标识提供者的门户，并将新的重定向 URL 添加到注册即可。然后，使用客户端 ID 和机密配置应用服务身份验证/授权。

### 控制器操作授权
现在，必须更改 `[AuthorizeLevel(AuthorizationLevel.User)]` 属性的所有实例才能使用标准的 ASP.NET `[Authorize]` 属性。此外，默认情况下，控制器现在是匿名的，如同在其他 ASP.NET 应用程序中一样。如果以前一直在使用其他某个 AuthorizeLevel 选项（例如 Admin 或 Application），请注意，现在这些选项都不再存在。可以改为设置共享机密的 AuthorizationFilters，或者配置 AAD 服务主体，安全启用服务到服务的调用。

### 获取其他用户信息

可以获取其他用户信息，包括通过 `GetAppServiceIdentityAsync()` 方法访问令牌：

        MicrosoftCredentials creds = await this.User.GetAppServiceIdentityAsync<MicrosoftCredentials>();

此外，如果应用程序依赖于用户 ID（例如，将它们存储在数据库中），请务必注意，移动服务与应用服务移动应用之间的用户 ID 是不相同的。但是，仍然可以获取移动服务用户 ID。所有 ProviderCredentials 子类具有 UserId 属性。继续分析前面的示例：

        string mobileServicesUserId = creds.Provider + ":" + creds.UserId;

如果应用程序依赖于用户 ID，必须尽可能使用相同的标识提供者注册。用户 ID 的范围通常限定在已使用的应用程序注册，因此引入新的注册可能会让用户在匹配其数据时发生问题。

### 自定义身份验证

如果应用程序使用自定义的身份验证解决方案，需要确保已升级的站点有权访问系统。遵循 [.NET server SDK overview]（.NET 服务器 SDK 概述）中适用于自定义身份验证的新说明来集成解决方案。请注意，自定义身份验证组件仍以预览版提供。

##<a name="updating-clients"></a>更新客户端
在获得可正常运行的移动应用后端之后，可以在使用它的新版客户端应用程序上操作。移动应用还包含新版客户端 SDK。与上述的服务器升级类似，需先删除所有对移动服务 SDK 的引用，然后再安装移动应用版本。

版本间的其中一个主要更改是构造函数不再需要应用程序密钥。现在只需传入移动应用的 URL。例如，在 .NET 客户端中，`MobileServiceClient` 构造函数现在是：

        public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://contoso.chinacloudsites.cn", // URL of the Mobile App
        );

可以通过以下链接，阅读有关安装新 SDK 以及使用新结构的信息：

- [iOS 3.0.0 或更高版本](/documentation/articles/app-service-mobile-ios-how-to-use-client-library/)
- [.NET (Windows/Xamarin) 2.0.0 或更高版本](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/)

如果应用程序使用推送通知，请记下每个平台的特定注册说明，因为相关的注册也发生了一些更改。

准备好新客户端版本后，请尝试对已升级的服务器项目运行该版本。确认该版本正常工作后，可将新版应用程序发布给客户。最后，在客户收到这些更新后，便可以删除应用的移动服务版本。现在，已使用最新的移动应用服务器 SDK 完全升级到应用服务移动应用。

<!-- URLs. -->

[Azure 门户预览]: https://portal.azure.cn/
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[What are Mobile Apps?]: /documentation/articles/app-service-mobile-value-prop/
[I already use web sites and mobile services - how does App Service help me?]: /documentation/articles/app-service-mobile-value-prop-migration-from-mobile-services
[移动应用服务器 SDK]: http://www.nuget.org/packages/microsoft.azure.mobile.server
[Create a Mobile App]: /documentation/articles/app-service-mobile-xamarin-ios-get-started/
[Add push notifications to your mobile app]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-push/
[Add authentication to your mobile app]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-users/
[Azure 计划程序]: /documentation/services/scheduler/
[Web 作业]: /documentation/articles/websites-webjobs-resources/
[How to use the .NET server SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Migrate from Mobile Services to an App Service Mobile App]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/
[Migrate your existing Mobile Service to App Service]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/
[应用服务定价]: /pricing/details/app-service/
[.NET server SDK overview]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/

<!---HONumber=Mooncake_0919_2016-->