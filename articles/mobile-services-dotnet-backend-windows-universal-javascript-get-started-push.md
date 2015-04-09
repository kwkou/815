<properties pageTitle="使用 .NET 后端移动服务的推送通知入门" metaKeywords="" description="了解如何使用 Azure 移动服务和通知中心将推送通知发送到通用 Windows 应用程序。" metaCanonical="" services="mobile-services,notification-hubs" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="mobile" manager="dwrede" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="javascript" ms.topic="article" ms.date="09/27/2014" ms.author="glenga" />


# 向移动服务应用程序添加推送通知

[WACOM.INCLUDE [mobile-services-selector-get-started-push](../includes/mobile-services-selector-get-started-push.md)]

本主题说明如何结合使用 Azure 移动服务和 .NET 后端向通用 Windows 应用程序发送推送通知。在本教程中，你将在通用 Windows 应用程序项目中使用 Azure 通知中心启用推送通知。完成后，每当在 TodoList 表中插入一条记录时，移动服务都会从 .NET 后端向所有已注册的 Windows 应用商店和 Windows Phone 应用商店应用程序发送一条推送通知。创建的通知中心可在移动服务中任意使用，可独立于移动服务进行管理，并可供其他应用程序和服务使用。

>[AZURE.NOTE]本主题演示如何使用 Visual Studio Professional 2013 Update 3 中的工具，为通用 Windows 应用程序添加从移动服务发送推送通知的支持。可以使用相同的步骤为 Windows 应用商店或 Windows Phone 应用商店 8.1 应用程序添加从移动服务发送推送通知的功能。如果你无法升级到 Visual Studio Professional 2013 Update 3，或者你想要手动将移动服务项目添加到 Windows 应用商店应用程序解决方案，请参阅[此版本](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started-push)的主题。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [为推送通知注册应用程序](#register)
2. [更新服务以发送推送通知](#update-service)
3. [为本地测试启用推送通知](#local-testing)
4. [在应用程序中测试推送通知](#test)

若要完成本教程，你需要以下项：

* 有效的 [Microsoft 商店帐户](http://go.microsoft.com/fwlink/p/?LinkId=280045)。
* <a href="https://go.microsoft.com/fwLink/p/?LinkID=391934" target="_blank">Visual Studio Professional 2013</a>（Update 3 或更高版本）。 <br/>可以使用免费试用版。 

## <a id="register"></a>为推送通知注册应用程序

[WACOM.INCLUDE [mobile-services-create-new-push-vs2013](../includes/mobile-services-create-new-push-vs2013.md)]

<ol start="6">
<li><p>浏览到 <code>\services\mobileServices\scripts</code> 项目文件夹，将生成的 &lt;<em>your_service_name</em>&gt;.push.register.js 脚本文件复制到共享的 <code>\js</code> 文件夹，然后从这两个单独的 Windows 和 WindowsPhone 应用程序项目中删除此文件。</p></li> 
<li><p>在共享的 <code>\js</code> 项目文件夹中打开此脚本文件，找到 <em>activated</em> 事件侦听器中将设备的通道 URL 注册到通知中心的代码，并删除 <strong>done</strong> 约定函数。</p>
<p>本教程在插入新项时（而不是在调用自定义 API 时）发送通知。</p></li>
<li><p>在 Windows 应用程序项目中，打开 default.html 文件并将脚本文件引用的路径更改为共享的 <code>\js</code> 项目文件夹，使它显示如下：</p><pre><code>&lt;script src="/js/your_service_name.push.register.js"&gt;&lt;/script&gt;</code></pre></li>
<li><p>对 WindowsPhone 应用程序项目重复执行此步骤。</p>
<p>现在，这两个项目使用的是推送注册脚本的共享版本。</p></li>
</ol>

现已在应用程序中启用推送通知，必须更新移动服务以发送推送通知。 

## <a id="update-service"></a>更新服务以发送推送通知

以下步骤将更新现有的 TodoItemController 类，以便在插入新项时将推送通知发送到所有已注册的设备。你可以在任何自定义的 [ApiController]、[TableController] 或后端服务中的其他任何位置实现类似的代码。 

[WACOM.INCLUDE [mobile-services-dotnet-backend-update-server-push](../includes/mobile-services-dotnet-backend-update-server-push.md)]

## <a id="local-testing"></a>为本地测试启用推送通知

[WACOM.INCLUDE [mobile-services-dotnet-backend-configure-local-push-vs2013](../includes/mobile-services-dotnet-backend-configure-local-push-vs2013.md)]

本节中的剩余步骤是可选的。使用这些步骤可以针对本地计算机上运行的移动服务测试你的应用程序。如果你计划使用 Azure 中运行的移动服务测试推送通知，则可以直接跳到最后一节。这是因为"添加推送通知"向导已将你的应用程序配置为连接到 Azure 中运行的服务。  

>[AZURE.NOTE]永远不要使用生产用移动服务进行测试和开发工作。始终将你的移动服务项目发布到单独的临时服务进行测试。

<ol start="5">
<li><p>浏览到 <code>\services\mobileServices\settings</code> 项目文件夹，将生成的 &lt;<em>your_service_name</em>&gt;.js 脚本文件复制到共享的 <code>\js</code> 项目文件夹，然后从这两个单独的 Windows 和 WindowsPhone 应用程序项目中删除此文件。同时，从每个应用程序项目的 <code>\services\mobileServices\scripts</code> 文件夹中删除此文件（如果在该处也存在）。</p></li> 
<li><p>在共享的 <code>\js</code> 项目文件夹中打开此脚本文件，然后注释掉定义用于访问 Azure 中运行的移动服务的 <a href="http://msdn.microsoft.com/library/azure/jj554219.aspx">MobileServiceClient 对象</a>的现有代码。</p></li>
<li><p>添加新的同名但在构造函数中使用本地主机 URL 的 <strong>MobileServiceClient</strong> 对象定义，类似于如下代码：</p>
<pre><code>// This MobileServiceClient has been configured to communicate with your local
// test project for debugging purposes.
var todolistClient = new WindowsAzure.MobileServiceClient(
	"http://localhost:4584");
</code></pre><p>使用此 <strong>MobileServiceClient</strong> 对象，应用程序将连接到本地服务（而不是 Azure 中托管的版本）。当你想要切换回来并针对 Azure 中托管的移动服务运行应用程序时，请更改回原始 <strong>MobileServiceClient</strong> 对象定义。</p></li>
</ol>

## <a id="test"></a>在应用程序中测试推送通知

[WACOM.INCLUDE [mobile-services-dotnet-backend-windows-universal-test-push](../includes/mobile-services-dotnet-backend-windows-universal-test-push.md)]

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序使用移动服务和通知中心发送推送通知的基础知识。接下来，请考虑完成下一个教程[向经过身份验证的用户发送推送通知]，该教程显示如何使用标记从移动服务将推送通知只发送给经过身份验证的用户。

在以下主题中详细了解移动服务和通知中心：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何使用移动服务对不同帐户类型的应用程序用户进行身份验证。

* [什么是通知中心？]
  <br/>了解有关通知中心将通知传递到所有主要客户端平台上的应用程序的工作方式的详细信息。

* [调试通知中心应用程序](http://go.microsoft.com/fwlink/p/?linkid=386630)
  </br>获取故障排除和调试通知中心解决方案的指南。 

* [如何将 HTML/JavaScript 客户端用于 Azure 移动服务]
  <br/>了解有关如何从 HTML 和 JavaScript 应用程序使用移动服务的详细信息。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
["提交应用"页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-get-started
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-universal-javascript-get-started-data
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-universal-javascript-get-started-users

[Send push notifications to authenticated users]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-javascript-push-notifications-app-users/

[什么是通知中心？]: /zh-cn/documentation/articles/notification-hubs-overview/

[如何将 HTML/JavaScript 客户端用于 Azure 移动服务]: /zh-cn/documentation/articles/mobile-services-html-how-to-use-client-library
