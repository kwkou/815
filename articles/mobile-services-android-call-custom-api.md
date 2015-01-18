<properties writer="ricksal" pageTitle="Call a custom API from an Android client | Mobile Dev Center" metaKeywords="" description="Learn how to define a custom API and then call it from an Android app that uses Azure Mobile Services." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="mobile-services" documentationCenter="Mobile" title="Call a custom API from the client" authors="glenga"  />

# 从客户端调用自定义 API

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api" title="Windows 应用商店 C#">Windows 应用商店 C#</a><a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-call-custom-api" title="Windows 应用商店 JavaScript">Windows 应用商店 JavaScript</a><a href="/zh-cn/documentation/articles/mobile-services-windows-phone-call-custom-api" title="Windows Phone">Windows Phone</a><a href="/zh-cn/documentation/articles/mobile-services-ios-call-custom-api" title="iOS">iOS</a><a href="/zh-cn/documentation/articles/mobile-services-android-call-custom-api" title="Android" class="current">Android</a>
</div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-android-call-custom-api" title=".NET 后端">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-android-call-custom-api"  title="JavaScript 后端" class="current">JavaScript 后端</a></div>

本主题说明如何从 Android 应用程序调用自定义 API。自定义 API 可让你定义自定义终结点，这些终结点将会公开不映射到插入、更新、删除或读取操作的服务器功能。使用自定义 API 能够以更大的力度控制消息传送，包括读取和设置 HTTP 消息标头，以及定义除 JSON 以外的消息正文格式。

使用本主题中创建的自定义 API，你可以发送单个 POST 请求，以便将移动服务表中所有 todo 项的 *completed* 标志设置为`true` 。如果没有此自定义 API，客户端必须逐个地发送请求，以更新表中每个 todo 项的该标志。

需要将此功能添加到你在完成[移动服务入门][移动服务入门]或[数据处理入门][数据处理入门]教程后创建的应用程序。为此，你需要完成以下步骤：

1.  [定义自定义 API][定义自定义 API]
2.  [更新应用程序以调用自定义 API][更新应用程序以调用自定义 API]
3.  [测试应用程序][测试应用程序]

本教程基于移动服务快速入门。在开始本教程之前，必须先完成[移动服务入门][移动服务入门]或[数据处理入门][数据处理入门]。

## <a name="define-custom-api"></a>定义自定义 API

[WACOM.INCLUDE [mobile-services-create-custom-api][mobile-services-create-custom-api]]

[WACOM.INCLUDE [mobile-services-android-call-custom-api][mobile-services-android-call-custom-api]]

## 后续步骤

创建自定义 API 并从 Android 应用程序调用该 API 后，建议你了解有关以下移动服务主题的详细信息：

-   [移动服务服务器脚本参考][移动服务服务器脚本参考]

    了解有关创建自定义 API 的详细信息。

-   [在源代码管理中存储服务器脚本][在源代码管理中存储服务器脚本]

    了解如何使用源代码管理功能来更方便、更安全地开发和发布自定义 API 脚本代码。

<!-- Anchors. --> 
<!-- URLs. -->

  [Windows 应用商店 C\#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-call-custom-api "Windows 应用商店 C#"
  [Windows 应用商店 JavaScript]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-call-custom-api "Windows 应用商店 JavaScript"
  [Windows Phone]: /zh-cn/documentation/articles/mobile-services-windows-phone-call-custom-api "Windows Phone"
  [iOS]: /zh-cn/documentation/articles/mobile-services-ios-call-custom-api "iOS"
  [Android]: /zh-cn/documentation/articles/mobile-services-android-call-custom-api "Android"
  [.NET 后端]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-android-call-custom-api ".NET 后端"
  [JavaScript 后端]: /zh-cn/documentation/articles/mobile-services-android-call-custom-api "JavaScript 后端"
  [移动服务入门]: /zh-cn/documentation/articles/mobile-services-android-get-started/
  [数据处理入门]: /zh-cn/documentation/articles/mobile-services-android-get-started-data/
  [定义自定义 API]: #define-custom-api
  [更新应用程序以调用自定义 API]: #update-app
  [测试应用程序]: #test-app
  [mobile-services-create-custom-api]: ../includes/mobile-services-create-custom-api.md
  [mobile-services-android-call-custom-api]: ../includes/mobile-services-android-call-custom-api.md
  [移动服务服务器脚本参考]: http://go.microsoft.com/fwlink/?LinkId=262293
  [在源代码管理中存储服务器脚本]: /zh-cn/documentation/articles/mobile-services-store-scripts-source-control
