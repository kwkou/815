<properties
    pageTitle="在 Azure 应用服务移动应用中创建 Android 应用 | Azure"
    description="遵循本教程开始使用 Azure 移动应用后端进行 Android 开发"
    services="app-service\mobile"
    documentationCenter="android"
    authors="yuaxu"
    manager="erikre"
    editor=""/>

<tags
    ms.service="app-service-mobile"
    ms.workload="na"
    ms.tgt_pltfrm="mobile-android"
    ms.devlang="java"
    ms.topic="hero-article"
    ms.date="10/01/2016"
    wacn.date="11/21/2016"
    ms.author="yuaxu"/>

#创建 Android 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## 概述

本教程说明如何使用 Azure 移动应用后端向 Android 移动应用添加基于云的后端服务。将创建一个新的移动应用后端以及一个简单的_待办事项列表_ Android 应用，此应用将应用数据存储在 Azure 中。

只有在完成本教程后，才可以学习有关使用 Azure 应用服务中的移动应用功能的所有其他 Android 教程。

## 先决条件

若要完成本教程，您需要以下各项：

* [Android 开发人员工具](https://developer.android.com/sdk/index.html)，其中包含 Android Studio 集成开发环境和最新的 Android 平台。
* Azure Mobile Android SDK，下载的快速入门项目中会自动引用它。
* 装有 [Visual Studio Community 2013] 或更高版本的电脑 &mdash; 在 Node.js 后端中不需要。
* [有效的 Azure 帐户](/pricing/1rmb-trial/)。

## <a name="create-a-new-azure-mobile-app-backend"></a>创建新的 Azure 移动应用后端

[AZURE.INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

## 配置服务器项目

[AZURE.INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## 下载并运行 Android 应用

[AZURE.INCLUDE [app-service-mobile-android-run-app](../../includes/app-service-mobile-android-run-app.md)]


<!-- Images. -->

<!-- URLs -->
[Azure portal]: https://portal.azure.cn/
[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203

<!---HONumber=Mooncake_0919_2016-->