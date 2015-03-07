<properties linkid="mobile-services-call-custom-api-wp8" urlDisplayName="从客户端调用自定义 API" pageTitle="从 Windows Phone 客户端调用自定义 API - 移动服务" metaKeywords="" description="了解如何定义自定义 API 以及如何从使用 Azure 移动服务的 Windows Phone 应用程序调用它。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Call a custom API from the client" authors="glenga" solutions="" manager="" editor="" />




# 从客户端调用自定义 API

[WACOM.INCLUDE [mobile-services-selector-call-custom-api](../includes/mobile-services-selector-call-custom-api.md)]

本主题说明如何从 Windows Phone 应用程序调用自定义 API。可使用自定义 API 定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

使用本主题中创建的自定义 API，你可以发送单个 POST 请求，用于将表中所有 todo 项的 completed 标志设置为 `true`。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

将此功能添加到您在完成 [将移动服务添加到现有应用程序](/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/)教程后创建的应用程序。为此，需要完成以下步骤：

1. [定义自定义 API]
2. [更新应用以调用自定义 API]
3. [测试应用] 

本教程是在 GetStartedWithData 示例的基础上制作的，后者是一个简单的 TodoList 应用程序。在开始本教程之前，您必须先完成 [将移动服务添加到现有应用程序](/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/)。

## <a name="define-custom-api"></a>定义自定义 API

[WACOM.INCLUDE [mobile-services-create-custom-api](../includes/mobile-services-create-custom-api.md)]

[WACOM.INCLUDE [mobile-services-windows-phone-call-custom-api](../includes/mobile-services-windows-phone-call-custom-api.md)]

## 后续步骤

创建自定义 API 并从 Windows Phone 应用程序调用该 API 后，建议您了解有关以下移动服务主题的详细信息：

* [移动服务服务器脚本参考]
  <br/>了解有关创建自定义 API 的详细信息。

* [在源代码管理中存储服务器脚本]
  <br/> 了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. -->
[定义自定义 API]: #define-custom-api
[更新应用以调用自定义 API]: #update-app
[测试应用]: #test-app
[后续步骤]: #next-steps

<!-- Images. -->


<!-- URLs. -->
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push/

[在源代码管理中存储服务器脚本]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control
