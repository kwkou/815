<properties 
	pageTitle="从 Windows Phone 客户端调用自定义 API — 移动服务" 
	description="了解如何定义自定义 API 以及如何从使用 Azure 移动服务的 Windows Phone 应用程序调用它。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/16/2015"
	wacn.date="10/22/2015"/>

#  从客户端调用自定义 API

[AZURE.INCLUDE [mobile-services-selector-call-custom-api](../includes/mobile-services-selector-call-custom-api.md)]

本主题说明如何从 Windows Phone 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

本主题中创建的自定义 API 使你能够发送单个 POST 请求，将已完成的标记设置为 `true` 的表中的所有 todo 项。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

将此功能添加到你在完成[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-windows-phone-get-started-data)教程后创建的应用程序。本教程是在 GetStartedWithData 示例（一个简单的 TodoList 应用程序）基础上制作的。在开始本教程之前，必须先完成[将移动服务添加到现有应用程序](/documentation/articles/mobile-services-windows-phone-get-started-data)。

##  <a name="define-custom-api"></a>定义自定义 API

[AZURE.INCLUDE [mobile-services-create-custom-api](../includes/mobile-services-create-custom-api.md)]

[AZURE.INCLUDE [mobile-services-windows-phone-call-custom-api](../includes/mobile-services-windows-phone-call-custom-api.md)]

##  后续步骤

本主题说明了如何使用 **InvokeApiAsync** 方法从 Windows Phone 应用程序调用一个相当简单的自定义 API。若要了解有关使用 **InvokeApiAsync** 方法的详细信息，请参阅文章 [Azure 移动服务中的自定义 API](http://blogs.msdn.com/b/carlosfigueira/archive/2013/06/19/custom-api-in-azure-mobile-services-client-sdks.aspx)。

另外，建议进一步了解以下移动服务主题：

* [移动服务服务器脚本参考]<br/>了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本]<br/>了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->
[Define the custom API]: #define-custom-api
[Update the app to call the custom API]: #update-app
[Test the app]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->


<!-- URLs. -->
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[Get started with Mobile Services]: /documentation/articles/mobile-services-windows-phone-get-started/
[Get started with data]: /documentation/articles/mobile-services-windows-phone-get-started-data/
[Get started with authentication]: /documentation/articles/mobile-services-windows-phone-get-started-users/
[Get started with push notifications]: /documentation/articles/mobile-services-windows-phone-get-started-push/

[在源代码管理中存储服务器脚本]: /documentation/articles/mobile-services-store-scripts-source-control

<!---HONumber=74-->