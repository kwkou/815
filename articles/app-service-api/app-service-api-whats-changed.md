<properties
	pageTitle="应用服务 API 应用 - 功能更改 | Azure"
	description="了解 Azure 应用服务中 API 应用的新增功能"
	services="app-service\api"
	documentationCenter=".net"
	authors="mohitsriv"
	manager="wpickett"
	editor="tdykstra"/>

<tags
	ms.service="app-service-api"
	ms.date="06/29/2016"
	wacn.date=""/>

# 应用服务 API 应用 - 功能更改

在 2015 年 11 月召开的 Connect() 活动中，对 Azure 应用服务[宣告](https://azure.microsoft.com/blog/azure-app-service-updates-november-2015/)了许多改进之处。这些改进包括 API 应用的基础更改，进一步配合移动和 Web 应用、减少概念计数以及改善部署和运行时性能。从 2015 年 11 月 30 日起，使用 Azure 管理门户或最新工具创建的新 API 应用将反映这些更改。本文描述这些更改，以及如何重新部署现有应用以充分利用这些功能。

## 功能更改
API 应用的重要功能 – 身份验证、CORS 和 API 元数据 – 已直接转移到应用服务。做出此项更改后，功能可以跨 Web、移动和 API 应用使用。事实上，这三个组件共享 Resource Manager 中的同一个 **microsoft.web/sites** 资源类型。不再需要使用或者随 API 应用一起提供 API 应用网关。此外，这样可以更轻松地使用 Azure API 管理，因为只有一个 API 管理网关。

![API 应用概述](./media/app-service-api-whats-changed/api-apps-overview.png)

API 应用更新的关键设计原则是让用户以所选的语言直接使用 API。如果 API 已部署为 Web 应用或移动应用，则无需重新部署应用即可充分利用新功能。针对目前正在使用 API 应用预览版的用户，下面提供了详细的迁移指南。

### 身份验证
现有的周全 API 应用、移动服务/应用和 Web 应用身份验证功能已得到统一，并在管理门户上单个 Azure 应用服务身份验证边栏选项卡中提供。有关应用服务中身份验证服务的简介，请参阅 [Expanding App Service authentication / authorization](https://azure.microsoft.com/blog/announcing-app-service-authentication-authorization/)（扩展应用服务身份验证/授权）。

对于 API 方案，提供了一些相关的新功能：

- **支持直接使用 Azure Active Directory**，无需客户端代码交换会话令牌的 AAD 令牌：客户端可以根据持有者令牌规范，在 Authorization 标头中只包含 AAD 令牌。这也意味着客户端或服务器端上不需要有任何特定于应用服务的 SDK。
- **服务到服务或“内部”访问**：如果有后台程序进程或其他某个客户端需要在不提供接口的情况下访问 API，可以使用 AAD 服务主体来请求令牌，并将它传递给应用服务，以便在应用程序中进行身份验证。
- **延迟授权**：许多应用程序对于应用程序的不同部分有各种不同的访问限制。用户可能希望某些 API 可公开访问，但另一些 API 要求登录。原始身份验证/授权功能采用“全有或全无”的模式，整个站点都要求登录。此选项仍然存在，但也可以选择允许应用程序代码在应用服务对用户进行身份验证之后提出访问决策。
 
有关新身份验证功能的详细信息，请参阅 [Authentication and authorization for API Apps in Azure App Service](/documentation/articles/app-service-api-authentication/)（Azure 应用服务中 API 应用的身份验证和授权）。有关如何将现有 API 应用从以前的 API 应用模型迁移到新模型的详细信息，请参阅本文稍后的[迁移现有 API 应用](#migrating-existing-api-apps)。
 
### CORS
现在，Azure 管理门户中提供了一个边栏选项卡用于配置 CORS，而不再提供以逗号分隔的 **MS\_CrossDomainOrigins** 应用设置。也可以使用 Azure PowerShell、CLI 或[资源浏览器](https://resources.azure.com/)等 Resource Manager 工具进行配置。在 **&lt;site name&gt;/web** 资源的 **Microsoft.Web/sites/config** 资源类型中设置 **cors** 属性。例如：

    {
        "cors": {
            "allowedOrigins": [
                "https://localhost:44300"
            ]
        }
    } 

### API 元数据
Web 应用、移动应用和 API 应用中各自提供了 API 定义边栏选项卡。在管理门户中，可以指定相对 URL，或者提供指向 API 的 Swagger 2.0 表示形式所在的终结点的绝对 URL。也可以使用 Resource Manager 工具进行配置。在 **&lt;site name&gt;/web** 资源的 **Microsoft.Web/sites/config** 资源类型中设置 **apiDefinition** 属性。例如：

    {
        "apiDefinition":
        {
            "url": "https://myStorageAccount.blob.core.chinacloudapi.cn/swagger/apiDefinition.json"
        }
    }

目前，元数据终结点需要在不经过身份验证的情况下即可公开访问，使许多下游客户端（例如 Visual Studio REST API 客户端生成和 PowerApps 的“添加 API”流）能够使用它。这意味着，如果使用应用服务身份验证并想要从应用本身内部公开 API 定义，则需要使用前面所述的“延迟身份验证”选项，使 Swagger 元数据的路由公开。

## 管理门户
在门户中选择“新建”>“Web + 移动”>“API 应用”可以创建体现本文所述新功能的 API 应用。选择“浏览”>“API 应用”只会显示这些新的 API 应用。浏览到 API 应用后，边栏选项卡中的布局和功能将与 Web 应用及移动应用中的布局和功能相同。唯一的差别在于快速入门内容和设置顺序。

## Visual Studio

大多数 Web 应用工具适用于新的 API 应用，因为它们共享相同的基础 **Microsoft.Web/sites** 资源类型。但是，应该将 Azure Visual Studio 工具升级到 2.8.1 或更高版本，因为它公开一些特定于 API 的功能。从 [Azure 下载页](/downloads/)下载 SDK。

随着应用服务类型的合理化，发布功能也已在“发布”>“Azure 应用服务”下面统一：

![API 应用发布](./media/app-service-api-whats-changed/api-apps-publish.png)

有关 SDK 2.8.1 的详细信息，请阅读通告[博客文章](https://azure.microsoft.com/blog/announcing-azure-sdk-2-8-1-for-net/)。

或者，可以从管理门户手动导入发布配置文件来启用发布。但是，云资源管理器、代码生成和 API 应用选择/创建功能需要 SDK 2.8.1 或更高版本。

## 迁移现有 API 应用
如果将自定义 API 部署到了以前的 API 应用预览版，请在 2015 年 12 月 31 日之前迁移到新的 API 应用模型。由于旧模型和新模型都基于应用服务中托管的 Web API，因此大多数现有代码可重复使用。

### 托管和重新部署
重新部署的步骤与将任何现有 Web API 部署到应用服务的步骤相同。步骤：

1. 创建一个空的 API 应用。为此，可以在门户中使用“新建”>“API 应用”、在 Visual Studio 中使用“发布”，或者使用 Resource Manager 工具。如果使用 Resource Manager 工具或模板，请在 **Microsoft.Web/sites** 资源类型中将 **kind** 值设置为 **api**，使管理门户中的快速启动和设置面向 API 方案。
2. 使用应用服务支持的任何部署机制，将项目连接并部署到空 API 应用。有关详细信息，请参阅 [Azure 应用服务部署文档](/documentation/articles/web-sites-deploy/)。
  
### 身份验证
应用服务身份验证服务支持以前的 API 应用模型所提供的相同功能。如果使用会话令牌并且需要 SDK，请使用以下客户端与服务器 SDK：

- 客户端：[Azure 移动客户端 SDK](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/)
- 服务器：[Microsoft Azure 移动应用 .NET 身份验证扩展](http://www.nuget.org/packages/Microsoft.Azure.Mobile.Server.Authentication/)

如果改用应用服务 alpha SDK，则以下 SDK 已过时：

- 客户端：[Azure AppService SDK](http://www.nuget.org/packages/Microsoft.Azure.AppService)
- 服务器：[Microsoft.Azure.AppService.ApiApps.Service](http://www.nuget.org/packages/Microsoft.Azure.AppService.ApiApps.Service)

但是，具体对于 Azure Active Directory 而言，如果直接使用 AAD 令牌，则不需要任何特定于应用服务的 SDK。

### 内部访问
以前的 API 应用模型包含内置的内部访问级别。这需要使用 SDK 为请求签名。如前所述，在 API 应用模型中，可以使用 AAD 服务主体代替服务到服务身份验证，而不需要特定于应用服务的 SDK。在 [Service principal authentication for API Apps in Azure App Service](/documentation/articles/app-service-api-dotnet-service-principal-auth/)（Azure 应用服务中 API 应用的服务主体身份验证）中了解详细信息。

### 发现
以前的 API 应用模型提供相应的 API，用于在运行时发现同一网关后面同一资源组中的其他 API 应用。在实现微服务模式的体系结构中，此功能特别有用。可以使用许多选项（但这些选项不一定直接受支持）：

1. 使用 Azure Resource Manager API 进行发现。
2. 将 Azure API 管理放在应用服务托管 API 的前面。Azure API 管理充当门面，即使内部拓扑发生更改，也能提供稳定的对外 URL。
3. 构建自己的发现 API 应用，并让其他 API 应用在启动时注册发现应用。
4. 在部署时，使用其他 API 应用的终结点填充所有 API 应用（和客户端）的应用设置。这在模板部署中是可行的，因为 API 应用现在允许控制 URL。

## 后续步骤

有关详细信息，请参阅 [API 应用文档部分](/documentation/services/app-service/api/)中的文章。这些文章已经过更新，反映 API 应用的新模型。此外，请务必访问论坛，获取其他详细信息或迁移指导：

- [MSDN 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureAPIApps)
- [堆栈溢出](http://stackoverflow.com/questions/tagged/azure-api-apps)

<!---HONumber=Mooncake_0919_2016-->