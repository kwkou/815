<properties
    pageTitle="在 Azure 应用服务移动应用中创建 Cordova 应用 | Azure"
    description="遵循本教程开始使用 Azure 移动应用后端进行 Apache Cordova 开发"
    services="app-service\mobile"
    documentationcenter="javascript"
    author="adrianhall"
    manager="erikre"
    editor=""
    tags=""
    keywords="cordova,javascript,移动,客户端" />
<tags
    ms.assetid="0b08fc12-0a80-42d3-9cc1-9b3f8d3e3a3f"
    ms.service="app-service-mobile"
    ms.workload="na"
    ms.tgt_pltfrm="mobile-html"
    ms.devlang="javascript"
    ms.topic="hero-article"
    ms.date="10/30/2016"
    wacn.date="01/23/2017"
    ms.author="adrianha"/>

# 创建 Apache Cordova 应用
[AZURE.INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## 概述
本教程说明如何使用 Azure 移动应用后端向 Apache Cordova 移动应用添加基于云的后端服务。创建一个新的移动应用后端以及一个简单的待办事项列表 Apache Cordova 应用，此应用将应用数据存储在 Azure 中。

只有在完成本教程后，才可以学习有关使用 Azure 应用服务中的移动应用功能的所有其他 Apache Cordova 教程。

## 先决条件
若要完成本教程，需要满足以下先决条件：

* 装有 [Visual Studio Community 2015] 或更高版本的电脑。
* [用于 Apache Cordova 的 Visual Studio 工具]
* [有效的 Azure 帐户](https://azure.microsoft.com/pricing/free-trial/)。

也可以绕过 Visual Studio，直接使用 Apache Cordova 命令行。在 Mac 计算机上学习本教程时，使用命令行相当有效。本教程不介绍如何使用命令行编译 Apache Cordova 客户端应用程序。

## 创建 Azure 移动应用后端
[AZURE.INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]


## 配置服务器项目

[AZURE.INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## 下载并运行 Apache Cordova 应用

[AZURE.INCLUDE [app-service-mobile-cordova-run-app](../../includes/app-service-mobile-cordova-run-app.md)]

## 后续步骤

完成本快速入门教程后，请继续学习下列教程之一：

* 将[脱机数据添加](/documentation/articles/app-service-mobile-cordova-get-started-offline-data/)到 Apache Cordova 应用。
* 将[身份验证添加](/documentation/articles/app-service-mobile-cordova-get-started-users/)到 Apache Cordova 应用。

详细了解 Azure 应用服务的重要概念。

* [脱机数据]
* [身份验证]
* [推送通知]

了解如何使用 SDK。

* [Apache Cordova SDK]
* [ASP.NET Server SDK]
* [Node.js Server SDK]

<!-- Images. -->

<!-- URLs -->
[Azure portal]: https://portal.azure.cn/
[Visual Studio Community 2015]: http://www.visualstudio.com/
[用于 Apache Cordova 的 Visual Studio 工具]: https://www.visualstudio.com/vs/cordova/
[脱机数据]: /documentation/articles/app-service-mobile-offline-data-sync/
[身份验证]: /documentation/articles/app-service-mobile-auth/
[推送通知]: /documentation/articles/notification-hubs-push-notification-overview/
[Apache Cordova SDK]: /documentation/articles/app-service-mobile-cordova-how-to-use-client-library/
[ASP.NET Server SDK]: /documentation/articles/app-service-mobile-dotnet-backend-how-to-use-server-sdk/
[Node.js Server SDK]: /documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description:update wording-->