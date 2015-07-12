<properties 
	pageTitle="从 Windows Phone 应用程序调用自定义 API - 移动服务" 
	description="了解如何定义自定义 API 以及如何从使用 Azure 移动服务的 Windows Phone 应用程序调用它。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	writer="glenga" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="04/24/2015" 
	wacn.date="06/26/2015"/>

# 从客户端调用自定义 API



本主题说明如何从 Windows Phone 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

本主题中创建的自定义 API 使你能够发送单个 POST 请求，将已完成的标记设置为 `true` 的表中的所有 todo 项。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

将此功能添加到你在完成[将移动服务添加到现有应用程序](mobile-services-dotnet-backend-windows-phone-get-started-data)教程后创建的应用程序。为此，需要完成以下步骤：

1. [定义自定义 API]
2. [更新应用程序以调用自定义 API]
3. [测试应用程序] 

本教程是在 GetStartedWithData 示例（一个简单的 TodoList 应用程序）基础上制作的。在开始本教程之前，必须先完成[将移动服务添加到现有应用程序](mobile-services-dotnet-backend-windows-phone-get-started-data)。

## <a name="define-custom-api"></a>定义自定义 API

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-custom-api](../includes/mobile-services-dotnet-backend-create-custom-api.md)]

[AZURE.INCLUDE [mobile-services-windows-phone-call-custom-api](../includes/mobile-services-windows-phone-call-custom-api.md)]


## 后续步骤

创建自定义 API 并从 Windows Phone 应用程序调用该 API 后，建议你了解有关以下移动服务主题的详细信息：

* [移动服务服务器脚本参考]<br/>了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本]<br/>了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->

[定义自定义 API]: #define-custom-api
[更新应用程序以调用自定义 API]: #update-app
[测试应用程序]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->

[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[Get started with Mobile Services]: mobile-services-windows-phone-get-started
[Get started with data]: mobile-services-dotnet-backend-windows-phone-get-started-data
[Get started with authentication]: mobile-services-dotnet-backend-windows-phone-get-started-users
[Get started with push notifications]: mobile-services-dotnet-backend-windows-phone-get-started-push

[在源代码管理中存储服务器脚本]: mobile-services-store-scripts-source-control

<!---HONumber=61-->