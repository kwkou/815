<properties
	pageTitle="在 Azure 应用服务移动应用中创建 iOS 应用 | Azure"
	description="遵循本教程开始使用 Azure 移动应用后端以 Objective-C 或 Swift 进行 iOS 开发"
	services="app-service\mobile"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="na"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="hero-article"
	ms.date="10/01/2016"
	wacn.date="10/17/2016"
	ms.author="yuaxu"/>

#创建 iOS 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## 概述

本教程说明如何将云后端服务 [Azure 移动应用](/documentation/articles/app-service-mobile-value-prop/)添加到 iOS 应用。首先将创建新的移动后端。然后，使用一个简单的_待办事项列表_ iOS 应用在 Azure 中存储数据。

## 先决条件

若要完成本教程，您需要以下各项：

* [有效的 Azure 帐户](/pricing/1rmb-trial/)
* 装有 [Visual Studio Community 2013] 或更高版本的电脑
* 装有 Xcode 7.3 或更高版本的 Mac

## 步骤 I：创建新的 Azure 移动应用后端

[AZURE.INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

## 步骤 II：配置后端项目

[AZURE.INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## 步骤 III：下载并运行 iOS 应用

[AZURE.INCLUDE [app-service-mobile-ios-run-app](../../includes/app-service-mobile-ios-run-app.md)]


<!-- Images. -->

<!-- URLs -->
[Azure portal]: https://portal.azure.cn/
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203

<!---HONumber=Mooncake_0919_2016-->