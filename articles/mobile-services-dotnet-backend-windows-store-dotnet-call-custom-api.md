<properties 
	pageTitle="从 Windows 应用商店客户端调用自定义 API - 移动服务" 
	description="了解如何定义自定义 API，然后从使用 Azure 移动服务的 Windows 应用商店应用程序调用它。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	writer="glenga" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/04/2015" 
	wacn.date="10/03/2015"/>

# 从客户端调用自定义 API

[AZURE.INCLUDE [mobile-services-selector-call-custom-api](../includes/mobile-services-selector-call-custom-api.md)]

本主题说明如何从 Windows 应用商店应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

本主题中创建的自定义 API 使你能够发送单个 POST 请求，将已完成的标记设置为 `true` 的表中的所有 todo 项。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门]或 [将移动服务添加到现有应用]。

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-custom-api](../includes/mobile-services-dotnet-backend-create-custom-api.md)]

## <a name="update-app"></a>更新应用程序以调用自定义 API

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-call-custom-api](../includes/mobile-services-windows-store-dotnet-call-custom-api.md)]


## 后续步骤

本主题说明了如何使用 **InvokeApiAsync** 方法从 Windows 应用商店应用调用一个相当简单的自定义 API。若要了解有关使用 **InvokeApiAsync** 方法的详细信息，请参阅文章 [Azure 移动服务中的自定义 API](http://blogs.msdn.com/b/carlosfigueira/archive/2013/06/19/custom-api-in-azure-mobile-services-client-sdks.aspx)。

另外，建议进一步了解以下移动服务主题：

* [定义支持定期通知的自定义 API]<br/>了解如何使用自定义 API 来支持 Windows 应用商店应用程序中的定期通知。启用定期通知后，Windows 将定期访问你的自定义 API 终结点，并使用返回的、采用磁贴特定格式的 XML 来更新开始菜单中的应用程序磁贴。

* [移动服务服务器脚本参考]<br/>了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本]<br/>了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->
[Define the custom API]: #define-custom-api
[Update the app to call the custom API]: #update-app
[Test the app]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts
[移动服务入门]: /documentation/articles/mobile-services-windows-store-get-started
[Get started with data]: /documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-data
[定义支持定期通知的自定义 API]: /documentation/articles/mobile-services-windows-store-dotnet-create-pull-notifications
[在源代码管理中存储服务器脚本]: /documentation/articles/mobile-services-store-scripts-source-control
<!---HONumber=71-->