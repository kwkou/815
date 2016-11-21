<properties
	pageTitle="适用于 Xamarin iOS 应用的 Azure 应用服务移动应用入门 | Azure"
	description="按照本教程进行操作，开始使用移动应用进行 Xamarin.iOS 开发。"
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="adrianhall"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="na"
	ms.tgt_pltfrm="mobile-xamarin-ios"
	ms.devlang="dotnet"
	ms.topic="hero-article"
	ms.date="10/01/2016"
	wacn.date="11/21/2016"
	ms.author="adrianha"/>


# 创建 Xamarin iOS 应用

[AZURE.INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## 概述

本教程说明如何使用 Azure 移动应用后端向 Xamarin.iOS 移动应用添加基于云的后端服务。创建一个新的移动应用后端以及一个简单的_待办事项列表_ Xamarin.iOS 应用，此应用将应用数据存储在 Azure 中。

只有在完成本教程后，才可以学习有关使用 Azure 应用服务中的移动应用功能的所有其他 Xamarin.iOS 教程。

## 先决条件

若要完成本教程，需要满足以下先决条件：

* 有效的 Azure 帐户。如果没有帐户，可以注册 Azure 试用版并获取多达 10 个免费的移动应用，即使在试用期结束之后仍可继续使用这些应用。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

* Visual Studio with Xamarin。有关说明，请参阅[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx)。

* 安装了 Xcode v7.0 版或更高版本以及 Xamarin Studio Community 的 Mac。请参阅[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx) 和 [Mac 用户的设置、安装和验证](https://msdn.microsoft.com/zh-cn/library/mt488770.aspx) (MSDN)。

>[AZURE.NOTE] 如果要在注册 Azure 帐户之前就开始使用 Azure 应用服务，请转到 [Try App Service](https://tryappservice.azure.com/?appServiceName=mobile)（试用应用服务）。可以立即在应用服务中创建短期的入门级移动应用，无需信用卡，也无需做出承诺。

## 创建 Azure 移动应用后端

按照下列步骤创建移动应用后端。

[AZURE.INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

## 配置服务器项目

现已预配可供移动客户端应用程序使用的 Azure 移动应用后端。接下来，为简单的“待办事项列表”后端下载服务器项目并将其发布到 Azure。

按照下列步骤将服务器项目配置为使用 Node.js 或 .NET 后端。

[AZURE.INCLUDE [app-service-mobile-configure-new-backend](../../includes/app-service-mobile-configure-new-backend.md)]

## 下载并运行 Xamarin.iOS 应用

1. 在浏览器窗口中，打开 [Azure 门户预览]。

2. 在移动应用的设置边栏选项卡上，单击“开始使用”>“Xamarin.iOS”。在步骤 3 下，单击“创建新应用”（如果尚未选择它）。接下来，单击“下载”按钮。

  	即可下载连接到移动后端的客户端应用程序。将压缩的项目文件保存到本地计算机，并记下保存位置。

3. 解压缩下载的项目，然后在 Xamarin Studio（或 Visual Studio）中打开它。

	![][9]

	![][8]

4. 按 F5 键生成项目，并在 iPhone 模拟器中启动应用。

5. 在应用中键入有意义的文本（例如 _Learn Xamarin_），然后单击“+”按钮。

	![][10]  


	来自请求的数据被插入到 TodoItem 表。移动应用后端返回存储在表中的项，数据显示在列表中。

>[AZURE.NOTE]可以在 QSTodoService.cs C# 文件中查看用于访问移动应用后端以查询和插入数据的代码。

## 后续步骤

* [向应用添加脱机同步功能](/documentation/articles/app-service-mobile-xamarin-ios-get-started-offline-data/)
* [向应用程序添加身份验证](/documentation/articles/app-service-mobile-xamarin-ios-get-started-users/)
* [向 Xamarin.Android 应用添加推送通知](/documentation/articles/app-service-mobile-xamarin-ios-get-started-push/)
* [如何使用 Azure 移动应用的托管客户端](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/)

<!-- Anchors. -->

[Getting started with mobile app backends]: #getting-started
[Create a new mobile app backend]: #create-new-service
[Next Steps]: #next-steps



<!-- Images. -->
[6]: ./media/app-service-mobile-xamarin-ios-get-started/xamarin-ios-quickstart.png
[8]: ./media/app-service-mobile-xamarin-ios-get-started/mobile-xamarin-project-ios-vs.png
[9]: ./media/app-service-mobile-xamarin-ios-get-started/mobile-xamarin-project-ios-xs.png
[10]: ./media/app-service-mobile-xamarin-ios-get-started/mobile-quickstart-startup-ios.png

<!-- URLs. -->
[Azure 门户预览]: https://portal.azure.cn/

<!---HONumber=Mooncake_1114_2016-->