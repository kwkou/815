<properties
	pageTitle="用于 API 发现和代码生成的应用服务 API 应用元数据 | Azure"
	description="了解 Azure 应用服务中的 API 应用如何使用 Swagger 元数据来简化 API 发现和代码生成。"
	services="app-service\api"
	documentationCenter=".net"
	authors="tdykstra"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-api"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/30/2016"
	wacn.date="11/21/2016"
	ms.author="rachelap"/>

# 用于 API 恢复和代码生成的应用服务 API 应用元数据 

应用服务 API 应用内置 [Swagger 2.0](http://swagger.io/) API 元数据支持。不一定要使用 Swagger，但如果使用 Swagger 的话，就能利用可让发现和使用变得更轻松的 API 应用功能。

## Swagger 终结点

可以指定一个终结点，为 API 应用提供 Swagger 2.0 JSON 元数据作为该 API 应用的属性。终结点可以相对于 API 应用的基 URL 或绝对 URL。绝对 URL 可以指向 API 应用外部。

对于许多下游客户端（例如，Visual Studio 代码生成和 PowerApps 的“添加 API”流）而言，此 URL 必须可公开访问（不受用户或服务身份验证的保护）。这意味着，如果使用了应用服务身份验证并想要从应用本身内部公开 API 定义，则需要使用身份验证选项，允许匿名流量进入 API。有关详细信息，请参阅 [Authentication and authorization for App Service API Apps](/documentation/articles/app-service-api-authentication/)（应用服务 API 应用的身份验证和授权）。

### 门户边栏选项卡

可以在[Azure 门户预览的](https://portal.azure.cn/)“API 定义”边栏选项卡中查看和更改终结点 URL。

![](./media/app-service-api-metadata/apidefblade.png)

### Azure Resource Manager 属性

也可以在 [Azure PowerShell](/documentation/articles/powershell-install-configure/) 和 [Azure CLI](/documentation/articles/xplat-cli-install/) 等命令行工具中使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)配置 API 应用的 API 定义 URL。

有关可设置 `apiDefinition` 属性的 Azure Resource Manager 模板的示例，请打开[待办事项列表示例应用程序中的 azuredeploy.json 文件](https://github.com/azure-samples/app-service-api-dotnet-todo-list/blob/master/azuredeploy.json)。找到如以下 JSON 示例所示的模板部分。

### 默认值

使用 Visual Studio 创建 API 应用时，API 定义终结点将自动设置为 API 应用的基 URL 加上 `/swagger/docs/v1`。这是 [Swashbuckle](https://www.nuget.org/packages/Swashbuckle) NuGet 包用来为 ASP.NET Web API 项目提供动态生成的 Swagger 元数据的默认 URL。

## 代码生成

将 Swagger 集成到 Azure API 应用的好处之一是自动生成代码。使用生成的客户端类可以更轻松地编写调用 API 应用的代码。

可以使用 Visual Studio 或命令行为 API 应用生成客户端代码。有关如何在 Visual Studio 中为 ASP.NET Web API 项目生成客户端类的信息，请参阅 [Get started with API Apps and ASP.NET](/documentation/articles/app-service-api-dotnet-get-started/#codegen)（API 应用和 ASP.NET 入门）。有关如何通过命令行为所有支持的语言执行此操作的信息，请参阅 GitHub.com 上 [Azure/autorest](https://github.com/azure/autorest) 存储库的自述文件。
 
## 后续步骤

有关指导如何创建、部署和使用 API 应用的分步教程，请参阅 [Get started with API Apps in Azure App Service](/documentation/articles/app-service-api-dotnet-get-started/)（Azure 应用服务中的 API 应用入门）。

<!---HONumber=Mooncake_0919_2016-->