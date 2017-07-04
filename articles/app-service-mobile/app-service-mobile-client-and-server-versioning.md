<properties
  pageTitle="移动应用和移动服务中的客户端与服务器 SDK 版本控制 | Azure 应用服务"
  description="客户端 SDK 列表，以及移动服务与 Azure 移动应用的服务器 SDK 版本兼容性"
  services="app-service\mobile"
  documentationCenter=""
  authors="adrianhall"
  manager="erikre"
  editor=""/>

<tags
  ms.service="app-service-mobile"
  ms.workload="mobile"
  ms.tgt_pltfrm="mobile-multiple"
  ms.devlang="dotnet"
  ms.topic="article"
  ms.date="10/01/2016"
  wacn.date="11/21/2016"
  ms.author="adrianha"/>

# 移动应用和移动服务中的客户端与服务器版本控制

Azure 移动服务的最新版本是 Azure 应用服务的**移动应用**功能。

移动应用客户端和服务器 SDK 最初基于移动服务中的 SDK，但它们彼此 *不* 兼容。也就是说，必须将 *移动应用* 客户端 SDK 与 *移动应用* 服务器 SDK 配合使用， *移动服务* 也是如此。可以通过客户端和服务器 SDK 使用的特殊标头值 `ZUMO-API-VERSION` 来强制实施此合约。

注意：每当本文档提到 *移动服务* 后端时，该后端不一定要托管在移动服务中。现在，可以将移动服务迁移到应用服务上运行，而无需更改任何代码，但服务还是使用 *移动服务* SDK 版本。

有关如何迁移到应用服务而不更改任何代码的详细信息，请参阅 [Migrate a Mobile Service to Azure App Service]（将移动服务迁移到 Azure 应用服务）一文。

## 标头规范

可以在 HTTP 标头或查询字符串中指定键 `ZUMO-API-VERSION`。值为版本字符串，格式为 **x.y.z**。

例如：

GET https://service.chinacloudsites.cn/tables/TodoItem

HEADERS: ZUMO-API-VERSION: 2.0.0

POST https://service.chinacloudsites.cn/tables/TodoItem?ZUMO-API-VERSION=2.0.0

## 选择不进行版本检查

可以将应用设置 **MS\_SkipVersionCheck** 的值设置为 **true**，选择不进行版本检查。在 web.config 中或在 Azure 门户的“应用程序设置”部分中可以指定此设置。

> [AZURE.NOTE] 移动服务和移动应用之间有许多行为发生了变化，尤其是在脱机同步、身份验证和推送通知方面。应该在完成测试之后才选择不要进行版本检查，确保这些行为的更改不会影响应用功能。

## 所有版本的兼容性摘要

下图显示了所有客户端与服务器类型之间的兼容性。后端根据使用的服务器 SDK 分类为移动**服务**或移动**应用**。

| | **移动服务** Node.js 或 .NET | **移动应用** Node.js 或 .NET |
| ----------                | -----------------------             |   ----------------              |
| [移动服务客户端] | 正常 | 错误* |
| [移动应用客户端] | 错误* | 正常 |

\*这可以通过指定 **MS\_SkipVersionCheck** 来控制。


<!-- IMPORTANT!  The anchors for Mobile Services and Mobile Apps MUST be 1.0.0 and 2.0.0 respectively, since there is an exception error message that uses those anchors. -->

<!-- NOTE: the fwlink to this document is http://go.microsoft.com/fwlink/?LinkID=690568 -->

## <a name="1.0.0"></a>移动服务客户端和服务器

下表中的客户端 SDK 与**移动服务**兼容。

注意：移动服务客户端 SDK *不* 发送 `ZUMO-API-VERSION` 的标头值。如果服务收到此标头或查询字符串值，将返回错误，除非已按上述明确选择不要进行检查。

### <a name="MobileServicesClients"></a>移动 *服务* 客户端 SDK

| 客户端平台 | 版本 | 版本标头值 |
| -------------------               | ------------------------                                                  | -------------------  |
| 托管客户端（Windows、Xamarin） | [1\.3.2](https://www.nuget.org/packages/WindowsAzure.MobileServices/1.3.2) | 不适用 |
| iOS | [2\.2.2](http://aka.ms/gc6fex) | 不适用 |
| Android | [2\.0.3](https://go.microsoft.com/fwLink/?LinkID=280126) | 不适用 |
| HTML | [1\.2.7](http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.7.min.js) | 不适用 |

### 移动 *服务* 服务器 SDK

| 服务器平台 | 版本 | 接受的版本标头 |
| ---------------- | ------------------------------------------------------------                                                   | ----------------------- |
| .NET | [WindowsAzure.MobileServices.Backend.* 版本 1.0.x](https://www.nuget.org/packages/WindowsAzure.MobileServices.Backend/) | **无版本标头** |
| Node.js | （即将支持） | **无版本标头** |

<!-- TODO: add Node npm version -->

### 移动服务后端的行为

| ZUMO-API-VERSION | MS\_SkipVersionCheck 的值 | 响应 |
| ---------------- | ---------------------------- | -------- |
| 未指定 | 任意 | 200 - 正常 |
| 任何值 | True | 200 - 正常 |
| 任何值 | False/未指定 | 400 - 错误的请求 |

## <a name="2.0.0"></a>Azure 移动应用客户端和服务器

### <a name="MobileAppsClients"></a>移动 *应用* 客户端 SDK

版本检查从 **Azure 移动应用**以下版本的客户端 SDK 开始引入：

| 客户端平台 | 版本 | 版本标头值 |
| -------------------               | ------------------------  | -----------------    |
| 托管客户端（Windows、Xamarin） | [2\.0.0](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/2.0.0) | 2\.0.0 |
| iOS | 3\.0.0 | 2\.0.0 |
| Android | [3\.0.0](http://go.microsoft.com/fwlink/?LinkID=717033&clcid=0x409) | 3\.0.0 |

<!-- TODO: add HTML version when released -->

### 移动*应用*服务器 SDK

以下服务器 SDK 版本包含版本检查：

| 服务器平台 | SDK 中 IsInRole 中的声明 | 接受的版本标头 |
| ---------------- | ------------------------------------------------------------                                                   | ----------------------- |
| .NET | [Microsoft.Azure.Mobile.Server](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/) | 2\.0.0 |
| Node.js | [azure-mobile-apps)](https://www.npmjs.com/package/azure-mobile-apps) | 2\.0.0 |

### 移动应用后端的行为

| ZUMO-API-VERSION | MS\_SkipVersionCheck 的值 | 响应 |
| ---------------- | ---------------------------- | -------- |
| x.y.z 或 Null | True | 200 - 正常 |
| Null | False/未指定 | 400 - 错误的请求 |
| 1\.x.y | False/未指定 | 400 - 错误的请求 |
| 2\.0.0-2.x.y | False/未指定 | 200 - 正常 |
| 3\.0.0-3.x.y | False/未指定 | 400 - 错误的请求 |


## 后续步骤

- [将移动服务迁移到 Azure 应用服务]


[移动服务客户端]: #MobileServicesClients
[移动应用客户端]: #MobileAppsClients


[Mobile App Server SDK]: http://www.nuget.org/packages/microsoft.azure.mobile.server
[Migrate a Mobile Service to Azure App Service]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/
[将移动服务迁移到 Azure 应用服务]: /documentation/articles/app-service-mobile-migrating-from-mobile-services/

<!---HONumber=Mooncake_0919_2016-->