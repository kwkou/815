<properties
	pageTitle="从移动服务升级到 Azure 应用服务 - Node.js"
	description="了解如何轻松将移动服务应用程序升级到应用服务移动应用"
	services="app-service\mobile"
	documentationCenter=""
	authors="adrianhall"
	manager="yochayk"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile"
	ms.devlang="node"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="11/21/2016"
	ms.author="adrianha"/>

# 将现有 Node.js Azure 移动服务升级到应用服务

应用服务移动应用是使用 Azure 构建移动应用程序的新方式。有关的详细信息，请参阅 [What are Mobile Apps?]（什么是移动应用？）

本主题介绍如何将现有 Node.js 后端应用程序从 Azure 移动服务升级到新的应用服务移动应用。执行此升级时，现有移动服务应用程序可以继续正常运行。如果需要升级 Node.js 后端应用程序，请参阅 [Upgrading your .NET Mobile Services](/documentation/articles/app-service-mobile-net-upgrading-from-mobile-services/)（升级 .NET 移动服务）。

将某个移动后端升级到 Azure 应用服务后，该后端即可访问所有应用服务功能，同时会根据[应用服务定价]而不是移动服务定价进行计费。

## 迁移与升级

[AZURE.INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

>[AZURE.TIP] 建议在升级之前先[执行迁移](/documentation/articles/app-service-mobile-migrating-from-mobile-services/)。这样，就能在同一个应用服务计划中放置两个版本的应用程序，且无需支付额外的费用。

### 移动应用 Node.js 服务器 SDK 改进

升级到新版[移动应用 SDK](https://www.npmjs.com/package/azure-mobile-apps) 可获得许多改进，包括：

- 新的轻量型 Node SDK 基于 [Express 框架](http://expressjs.com/en/index.html)，与新推出的 Node 版本功能保持一致。可以使用 Express 中间件自定义应用程序行为。

- 移动服务 SDK 相比，性能有明显改进。

- 现在，可以将网站与移动后端托管在一起；同样，可以很轻松地将 Azure 移动 SDK 添加到任何现有 express.v4 应用程序。

- 移动应用 SDK 为跨平台和本地开发而构建，可以在 Windows、Linux 和 OSX 平台上本地开发与运行。现在，可以方便地使用常见的 Node 开发技术，例如，在部署之前运行 [Mocha](https://mochajs.org/) 测试。

## <a name="overview"></a>基本升级概述

为了帮助升级 Node.js 后端，Azure 应用服务提供了兼容包。升级后，将会获得可部署到新应用服务站点的全新站点。

移动服务客户端 SDK 与新的移动应用服务器 SDK **不**兼容。为了提供应用程序的服务连续性，不应该将更改发布到当前正在为发布的客户端提供服务的站点。而应该创建新的移动应用作为副本。可以在同一个应用服务计划中放置此应用程序，以免产生额外的财务成本。

这样，就会获得两个版本的应用程序：一个保持不变并为已发布的现有应用程序提供服务，另一个则可升级且目标为新的客户端版本。可根据自己的步调移动和测试代码，但应确保所做的任何 bug 修复都已应用到这两个版本。觉得已将所需数量的现有客户端应用升级到最新版本后，可以根据需要删除已迁移的原始应用。如果托管在与移动移动相同的应用服务计划中，则不会产生任何额外的成本。

完整的升级过程大致如下：

1. 下载现有的（已迁移）Azure 移动服务。
2. 使用兼容包将项目转换为 Azure 移动应用。
3. 更正任何差异（例如身份验证设置）。
4. 将转换后的 Azure 移动应用项目部署到新的 应用服务。
4. 发布使用新移动应用的新版客户端应用程序。
5. （可选）删除已迁移的原始移动服务应用。

当已迁移的原始移动服务没有任何流量时即可删除。

## <a name="install-npm-package"></a>安装必备组件

应在本地计算机上安装 [Node]。还应安装兼容包。安装 Node 后，可以从新的 cmd 或 PowerShell 命令提示符运行以下命令：

```npm i -g azure-mobile-apps-compatibility```

## <a name="obtain-ams-scripts"></a>获取 Azure 移动服务脚本

- 登录到 [Azure 门户预览]。
- 使用“所有资源”或“应用程序服务”找到移动服务站点。
- 在站点内单击“工具”->“Kudu”->“转到”，打开 Kudu 站点。
- 单击“调试控制台”->“PowerShell”打开调试控制台。
- 依次单击每个目录导航到 `site/wwwroot/App_Data/config`
- 单击 `scripts` 目录旁边的下载图标。

随后将下载 ZIP 格式的脚本。在本地计算机上创建新目录，然后在该目录中解压缩 `scripts.ZIP` 文件。此时会创建 `scripts` 目录。

## <a name="scaffold-app"></a>创建新 Azure 移动应用后端的基架

从包含脚本目录的目录运行以下命令：

```scaffold-mobile-app scripts out```

此时将在 `out` 目录中创建带有基架的 Azure 移动应用后端。最好将 `out` 目录签入所选的源代码存储库（但不一定要这样做）。

## <a name="deploy-ama-app"></a>部署 Azure 移动应用后端

在部署期间，需要执行以下操作：

1. 在 [Azure 门户预览]中创建新的移动应用。
2. 对连接的数据库运行 `createViews.sql` 脚本。
3. 将链接到移动服务的数据库链接到新的应用服务。
4. 将任何其他资源（例如通知中心）链接到新的应用服务。
5. 将生成的代码部署到新站点。

### 创建新的移动应用

1. 在 [Azure 门户预览]登录。

2. 单击“+新建”>“Web + 移动”>“移动应用”，然后提供移动应用后端名称。

3. 对于“资源组”，请选择现有资源组，或创建新组（使用与应用相同的名称。）
 
	可以选择其他应用服务计划或创建新的计划。有关应用服务计划以及如何在不同定价层和所需位置中创建新计划的详细信息，请参阅 [Azure App Service plans in-depth overview](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)（Azure 应用服务计划深入概述）。

4. 对于“应用服务计划”，请选择默认计划（位于[标准层](/pricing/details/app-service/)）。还可以选择不同的计划，或[创建一个新计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/#create-an-app-service-plan)。应用服务计划的设置将确定与应用关联的[位置、功能、成本和计算资源](/pricing/details/app-service/)。

	做出有关计划的决定后，单击“创建”。随后将创建移动应用后端。


### 运行 CreateViews.SQL

带有基架的应用包含名为 `createViews.sql` 的文件。必须对目标数据库执行此脚本。可以通过“设置”边栏选项卡下面的“连接字符串”从已迁移的移动服务获取目标数据库的连接字符串。该字符串名为 `MS_TableConnectionString`。

可以从 SQL Server Management Studio 或 Visual Studio 内部运行此脚本。

### 将数据库链接到应用服务

将现有数据库链接到应用服务：

- 在 [Azure 门户预览]中，打开应用服务。
- 选择“所有设置”->“数据连接”。
- 单击“+ 添加”。
- 在下拉列表中，选择“SQL 数据库”
- 在“SQL 数据库”下面选择现有数据库，然后单击“选择”。
- 在“连接字符串”下面，输入数据库的用户名和密码，然后单击“确定”。
- 在“添加数据连接”边栏选项卡中，单击“确定”。

查看已迁移的移动服务中目标数据库的连接字符串，即可找到用户名和密码。


### 设置身份验证

Azure 移动应用允许在服务中配置 Azure Active Directory 和 Microsoft 身份验证。自定义身份验证需要另行开发。有关详细信息，请参阅 [Authentication Concepts]（身份验证概念）文档和 [Authentication Quickstart]（身份验证快速入门）文档。

## <a name="updating-clients"></a>更新移动客户端

在获得可正常运行的移动应用后端之后，可以在使用它的新版客户端应用程序上操作。移动应用还包含新版客户端 SDK。与上述的服务器升级类似，需先删除所有对移动服务 SDK 的引用，然后再安装移动应用版本。

版本间的其中一个主要更改是构造函数不再需要应用程序密钥。现在只需传入移动应用的 URL。例如，在 .NET 客户端中，`MobileServiceClient` 构造函数现在是：

        public static MobileServiceClient MobileService = new MobileServiceClient(
            "https://contoso.chinacloudsites.cn", // URL of the Mobile App
        );

可以通过以下链接，阅读有关安装新 SDK 以及使用新结构的信息：

- [Android 2.2 或更高版本](/documentation/articles/app-service-mobile-android-how-to-use-client-library/)
- [iOS 3.0.0 或更高版本](/documentation/articles/app-service-mobile-ios-how-to-use-client-library/)
- [.NET (Windows/Xamarin) 2.0.0 或更高版本](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/)
- [Apache Cordova 2.0 或更高版本](/documentation/articles/app-service-mobile-cordova-how-to-use-client-library/)

如果应用程序使用推送通知，请记下每个平台的特定注册说明，因为相关的注册也发生了一些更改。

准备好新客户端版本后，请尝试对已升级的服务器项目运行该版本。确认该版本正常工作后，可将新版应用程序发布给客户。最后，在客户收到这些更新后，便可以删除应用的移动服务版本。现在，已使用最新的移动应用服务器 SDK 完全升级到应用服务移动应用。

<!-- URLs. -->

[Azure Classic Management Portal]: https://manage.windowsazure.cn/
[What are Mobile Apps?]: /documentation/articles/app-service-mobile-value-prop/
[I already use web sites and mobile services - how does App Service help me?]: /documentation/articles/app-service-mobile-value-prop-migration-from-mobile-services
[Mobile App Server SDK]: https://www.npmjs.com/package/azure-mobile-apps
[Create a Mobile App]: /documentation/articles/app-service-mobile-xamarin-ios-get-started/
[Add push notifications to your mobile app]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-push/
[Add authentication to your mobile app]: /documentation/articles/app-service-mobile-xamarin-ios-get-started-users/
[Azure Scheduler]: /documentation/services/scheduler/
[Web Job]: /documentation/articles/websites-webjobs-resources/
[How to use the .NET server SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Migrate from Mobile Services to an App Service Mobile App]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/
[Migrate your existing Mobile Service to App Service]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/
[应用服务定价]: /pricing/details/app-service/
[.NET server SDK overview]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Authentication Concepts]: /documentation/articles/app-service-authentication-overview/
[Authentication Quickstart]: /documentation/articles/app-service-mobile-auth/

[Azure 门户预览]: https://portal.azure.cn/
[OData]: http://www.odata.org
[Promise]: https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise
[basicapp sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/basic-app
[todo sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/todo
[samples directory on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples
[static-schema sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/static-schema
[QueryJS]: https://github.com/Azure/queryjs
[Node.js Tools 1.1 for Visual Studio]: https://github.com/Microsoft/nodejstools/releases/tag/v1.1-RC.2.1
[mssql Node.js package]: https://www.npmjs.com/package/mssql
[Microsoft SQL Server 2014 Express]: http://www.microsoft.com/server-cloud/Products/sql-server-editions/sql-server-express.aspx
[ExpressJS Middleware]: http://expressjs.com/guide/using-middleware.html
[Winston]: https://github.com/winstonjs/winston

<!---HONumber=Mooncake_0919_2016-->