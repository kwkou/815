<properties
	pageTitle="使用 Azure 移动服务开发 iOS 应用程序入门"
	description="遵照本教程开始使用 Azure 移动服务进行 iOS 开发。"
	services="mobile-services"
	documentationCenter="ios"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="07/21/2016"
	wacn.date="09/26/2016"
	ms.author="krisragh"/>

# <a name="getting-started"></a>移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../../includes/mobile-services-selector-get-started.md)]


本教程说明如何使用 Azure 移动服务向 iOS 应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单“待办事项列表”应用程序。移动服务为服务器端业务逻辑使用 .NET 和 Visual Studio。若要以 JavaScript 创建包含服务器端业务逻辑的移动服务，请参阅本主题的 [JavaScript 后端版本]。

> [AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。如果你没有帐户，可以注册 Azure 试用版并获取[免费的移动服务，即使在试用期结束之后仍可继续使用这些服务](/pricing/details/mobile-services/)。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

## <a name="create-new-service"></a>创建新的移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-new-service](../../includes/mobile-services-dotnet-backend-create-new-service.md)]

## 将移动服务和应用程序下载到本地计算机

创建移动服务后，请下载可在本地运行的项目。

1. 单击刚刚创建的移动服务，在“快速启动”选项卡中单击“选择平台”下的“iOS”，然后展开“创建新的 iOS 应用程序”。

2. 在 Windows 电脑上，在“下载你的服务并将其发布到云”下面单击“下载”。这样可以下载实现你的移动服务的 Visual Studio 项目。将压缩的项目文件保存到本地计算机，并记下保存位置。

3. 在 Mac 上，在“下载并运行应用程序”下面单击“下载”。随即将会下载已连接到移动服务的示例_待办事项列表_应用程序的项目，以及移动服务 iOS SDK。将压缩的项目文件保存到本地计算机，并记下保存位置。

## 测试移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-test-local-service](../../includes/mobile-services-dotnet-backend-test-local-service.md)]

## 发布移动服务

[AZURE.INCLUDE [mobile-services-dotnet-backend-publish-service](../../includes/mobile-services-dotnet-backend-publish-service.md)]


## 运行新的 iOS 应用程序

[AZURE.INCLUDE [mobile-services-ios-run-app](../../includes/mobile-services-ios-run-app.md)]


## <a name="next-steps"></a>后续步骤

本主题说明了如何针对 Azure 中运行的移动服务运行新的客户端应用程序。在对本地计算机上运行的移动服务测试 iOS 应用程序之前，必须配置 Web 服务器和防火墙，以允许从 iOS 开发计算机进行访问。有关详细信息，请参阅[配置本地 Web 服务器以允许连接到本地移动服务](/documentation/articles/mobile-services-dotnet-backend-how-to-configure-iis-express/)。

了解如何在移动服务中执行其他重要任务：

* [脱机数据同步入门]
<br/>了解如何使用脱机数据同步来使应用程序保持较高的响应能力和稳健性。

* [向现有应用程序添加身份验证]
<br/>了解如何使用标识提供程序对应用程序的用户进行身份验证。

* [向现有应用程序添加推送通知]
<br/>了解如何向应用程序发送一条很基本的推送通知。

* [移动服务 .NET 后端故障排除]
<br/>了解如何诊断和修复移动服务 .NET 后端可能会出现的问题。

[AZURE.INCLUDE [app-service-disqus-feedback-slug](../../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-completed-ios.png
[1]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-steps-vs.png

[6]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-portal-quickstart-ios.png
[7]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-steps-ios.png
[8]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-xcode-project.png

[10]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-dotnet-backend-ios-get-started/mobile-data-browse.png


<!-- URLs. -->
[脱机数据同步入门]: /documentation/articles/mobile-services-ios-get-started-offline-data/
[向现有应用程序添加身份验证]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-users/
[向现有应用程序添加推送通知]: /documentation/articles/mobile-services-dotnet-backend-ios-get-started-push/
[移动服务 .NET 后端故障排除]: /documentation/articles/mobile-services-dotnet-backend-how-to-troubleshoot/

[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533

[XCode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[JavaScript 后端版本]: /documentation/articles/mobile-services-ios-get-started/

<!---HONumber=Mooncake_0118_2016-->