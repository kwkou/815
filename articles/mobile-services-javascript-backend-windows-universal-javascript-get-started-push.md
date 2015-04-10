<properties pageTitle="通过 JavaScript 后端移动服务开始使用推送通知" metaKeywords="" description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到您的通用 Windows 应用程序。" metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="mobile-services,notification-hubs" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="javascript" ms.topic="article" ms.date="09/15/2014" ms.author="glenga" />


# 向移动服务应用程序添加推送通知

[WACOM.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本主题说明如何结合使用 Azure 移动服务和 JavaScript 后端向通用 Windows 应用程序发送推送通知。在本教程中，你将在通用 Windows 应用程序项目中使用 Azure 通知中心来启用推送通知。完成后，当每次在 TodoList 表中插入一条记录时，你的移动服务将从 JavaScript 后端向所有已注册的 Windows 应用商店和 Windows Phone 应用商店应用发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

>[AZURE.NOTE]本主题演示如何在 Visual Studio 2013 Update 3 中使用此工具，以便从移动服务向通用 Windows 应用程序添加推送通知的支持。可以使用相同的步骤将推送通知从移动服务添加到 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序。如果您无法升级到 Visual Studio 2013 Update 3，或者您更喜欢手动将您的移动服务项目添加到 Windows 应用商店应用程序解决方案，请参阅该主题的[此版本](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push)。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [为推送通知注册应用程序](#register)
2. [更新服务以发送推送通知](#update-service)
3. [在应用程序中测试推送通知](#test)

若要完成本教程，您需要以下各项：

* 活动[Microsoft 应用商店帐户](http://go.microsoft.com/fwlink/p/?LinkId=280045)。
* [Visual Studio 2013 Express for Windows](http://go.microsoft.com/fwlink/?LinkId=257546)，Update 3 或更高版本 

##<a id="register"></a>为推送通知注册应用程序

[WACOM.INCLUDE [mobile-services-create-new-push-vs2013](../includes/mobile-services-create-new-push-vs2013.md)]

<ol start="6">
<li><p>浏览到 <code>\services\mobileServices\scripts</code> 项目文件夹，将生成的&lt;<em>your_service_name</em>&gt;.push.register.js 脚本文件复制到共享 <code>\js</code> 文件夹，然后从这两个单独的 Windows 和 WindowsPhone 应用程序项目中删除此文件。</p></li> 
<li><p>在共享 <code>\js</code> 项目文件夹中打开该脚本文件，找到 <em>激活</em> 事件侦听器中的代码，该侦听器将设备通道 URL 注册到通知中心，并删除 <strong>done</strong> promise 函数。</p>
<p>本教程将在插入新项（而不是在调用自定义 API）时发送通知。</p></li>
<li><p>在 Windows 应用程序项目中，打开 default.html 文件并更改共享 <code>\js</code> 项目文件夹的脚本文件引用路径，以便它看起来如下所示：</p><pre><code>&lt;script src="/js/your_service_name.push.register.js"&gt;&lt;/script&gt;</code></pre></li>
<li><p>针对 WindowsPhone 应用程序项目重复此步骤。</p>
<p>现在，这两个项目使用推送注册脚本的共享版本。</p></li>
</ol>

现在可以在应用程序中启用推送通知，你必须更新移动服务以发送推送通知。 

##<a id="update-service"></a>更新服务以发送推送通知

以下步骤更新注册到 TodoItem 表的插入脚本。在任何服务器脚本中或在后端服务中的其他任何地方，您都可以实施类似的代码。 

[WACOM.INCLUDE [mobile-services-javascript-update-script-notification-hubs](../includes/mobile-services-javascript-update-script-notification-hubs.md)]

##<a id="test"></a> 在应用程序中测试推送通知

[WACOM.INCLUDE [mobile-services-javascript-backend-windows-universal-test-push](../includes/mobile-services-javascript-backend-windows-universal-test-push.md)]

## <a name="next-steps"> </a>后续步骤

本教程演示了启用 Windows 应用商店应用程序以便使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成下一个教程，[向已经过身份验证的用户发送推送通知]，它显示如何使用标记将推送通知从移动服务发送到只有经过身份验证的用户。

下面的主题介绍了有关移动服务和通知中心的详细信息：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何通过不同帐户类型（使用移动服务）对应用程序的用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心跨所有主要的客户端平台向您的应用程序交付通知的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取故障排除和调试通知中心解决方案的指南。 

* [如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端]
  <br/>了解有关如何通过 HTML 和 JavaScript 应用程序使用移动服务的详细信息。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-universal-javascript-get-started-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-universal-javascript-get-started-users

[向已经过身份验证的用户发送推送通知]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-javascript-push-notifications-app-users/

[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/

[如何使用适用于 Azure 移动服务的 HTML/JavaScript 客户端]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library
