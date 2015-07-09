<properties 
	writer="ricksal" 
	pageTitle="从 Android 客户端调用自定义 API | 移动开发人员中心" 
	description="了解如何定义自定义 API，然后从使用 Azure 移动服务的 Android 应用程序调用它。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/03/2015" 
	wacn.date="06/26/2015"/>

# 从客户端调用自定义 API


本主题说明如何从 Android 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

使用本主题中创建的自定义 API，你可以发送单个 POST 请求，用于将移动服务表中所有 todo 项的 *completed* 标志设置为 `true`。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

需要将此功能添加到你在完成[移动服务入门]或[数据处理入门]教程后创建的应用程序。


>[AZURE.NOTE]如果要查看已完成应用程序的源代码，请转到<a href="https://github.com/RickSaling/mobile-services-samples/tree/futures/CallCustomApi/Android" target="_blank">此处</a>。

##先决条件

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites.md)]

## <a name="define-custom-api"></a>定义自定义 API

[AZURE.INCLUDE [mobile-services-create-custom-api](../includes/mobile-services-create-custom-api.md)]


[AZURE.INCLUDE [mobile-services-android-call-custom-api](../includes/mobile-services-android-call-custom-api.md)]


## 后续步骤

创建自定义 API 并从 Android 应用程序调用该 API 后，建议你了解有关以下移动服务主题的详细信息：



* [移动服务服务器脚本参考] 
  了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本] 
  了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->

[Define the custom API]: #define-custom-api
[Update the app to call the custom API]: #update-app
[Test the app]: #test-app
[Next Steps]: #next-steps

<!-- URLs. -->

[Mobile Services Android SDK]: http://go.microsoft.com/fwlink/p/?LinkID=280126
[移动服务服务器脚本参考]: /documentation/articles/mobile-services-how-to-use-server-scripts/
[My Apps dashboard]: http://go.microsoft.com/fwlink/?LinkId=262039
[移动服务入门]: mobile-services-android-get-started
[数据处理入门]: mobile-services-android-get-started-data
[Get started with authentication]: mobile-services-android-get-started-users
[Get started with push notifications]: mobile-services-android-get-started-push

[在源代码管理中存储服务器脚本]: mobile-services-store-scripts-source-control

<!---HONumber=61-->