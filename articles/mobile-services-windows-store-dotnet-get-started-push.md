<properties pageTitle="推送通知入门（Windows 应用商店）| 移动开发人员中心" metaKeywords="" description="了解如何使用 Azure 移动服务向 Windows 应用商店应用程序发送推送通知。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with push notifications in Mobile Services" authors="glenga" solutions="" manager="" editor="" />


# 将推送通知添加到移动服务应用程序（传统推送）

<div class="dev-center-tutorial-selector sublanding">
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push" title="Windows Store C#" class="current">Windows 应用商店 C#</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-push" title="Windows Store JavaScript">Windows 应用商店 JavaScript</a>
    <a href="/zh-cn/documentation/articles/mobile-services-windows-phone-get-started-push" title="Windows Phone">Windows Phone</a>
    <a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-push" title="iOS">iOS</a>
    <a href="/zh-cn/documentation/articles/mobile-services-android-get-started-push" title="Android">Android</a>
<!--    <a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-ios-get-started-push" title="Xamarin.iOS">Xamarin.iOS</a>
    <a href="/zh-cn/documentation/articles/partner-xamarin-mobile-services-android-get-started-push" title="Xamarin.Android">Xamarin.Android</a> -->
	<a href="/zh-cn/documentation/articles/partner-appcelerator-mobile-services-javascript-backend-appcelerator-get-started-push" title="Appcelerator">Appcelerator</a>
</div>

<div class="dev-center-tutorial-subselector"><a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-push/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push/"  title="JavaScript backend" class="current">JavaScript 后端</a></div>	

本主题说明如何通过 Visual Studio 2013 使用 Azure 移动服务向 Windows 应用商店应用程序发送推送通知。在本教程中，你将直接通过 Visual Studio 使用 Windows 推送通知服务 (WNS) 向快速入门项目添加推送通知。完成本教程后，每次插入一条记录时，你的移动服务就会发送一条推送通知。

>[WACOM.NOTE]本主题支持 <em>现有的</em> 尚未升级的 <em>移动服务</em> 使用通知中心集成。在创建 <em>新的</em> 移动服务时，会自动启用此集成功能。有关新建移动服务，请参阅[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/)。
>
>移动服务将与 Azure 通知中心集成，以支持附加的推送通知功能，如模板、多个平台和改进的规模。 <em>您应升级现有的移动服务以便在可能的情况下使用通知中心</em>。升级之后，请参阅此版本的[推送通知入门](/zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/)。

本教程将指导你完成启用推送通知的以下基本步骤：

1. [注册用于推送通知的应用程序并配置移动服务]
2. [更新生成的推送通知代码]
3. [插入数据以接收通知]

本教程基于移动服务快速入门。在开始学习本教程之前，必须先完成[移动服务入门]或[数据处理入门]，以将项目连接到移动服务。如果尚未连接移动服务，"添加推送通知"向导将为你创建此连接。 

<h2><a name="register"></a>在应用程序中添加和配置推送通知</h2>

[WACOM.INCLUDE [mobile-services-create-new-push-vs2013](../includes/mobile-services-create-new-push-vs2013.md)]

<ol start="6">
<li><p>展开 <strong>"服务"</strong>、 <strong>移动服务</strong>、您的服务名称，打开生成的代码文件，然后检查 <strong>UploadChannel</strong> 方法获取安装 ID 和设备的通道，并将此数据插入到新的通道表中。</p> 

<p>此外，向导还会在 App.xaml.cs 代码文件中将对此方法的一个调用添加到 <strong>OnLaunched</strong> 事件处理程序。这可确保每次启动应用程序时都尝试注册设备。</p></li> 
<li><p>在服务器资源管理器中，展开 <strong>Azure</strong>、 <strong>移动服务</strong>、您的服务名称和 <strong>通道</strong>，然后打开 insert.js 文件。</p> 

<p>此文件（存储在你的移动服务中）包含 JavaScript 代码，客户端通过将数据插入到 channels 表中来发送设备注册请求时，将执行该代码。</p> 

<div class="dev-callout"><b>注意</b>
	<p>此文件的初始版本包含检查设备的现有注册的代码。它还包含当新的注册添加到 channels 表时发送推送通知的代码。发送推送通知的代码可以包含在任何已注册的脚本文件中。此脚本的位置取决于通知的触发方式。可以针对对表进行的插入、更新、删除或读取操作注册脚本，还可将脚本注册为计划的作业或自定义 API。有关详细信息，请参阅 <a href="http://go.microsoft.com/fwlink/p/?LinkID=287178">"在移动服务中使用服务器脚本"</a>。</p>
</div>
</li> 
<li><p>按 F5 键以运行应用程序，并验证是否立刻从移动服务收到了通知。</p>
<p>此通知是通过在新的 channels 表中插入一行（即设备注册信息）生成的。</p>
</li>
</ol>
虽然使用生成的代码可在应用程序运行时轻松演示通知，但这通常不是有意义的方案。下一步，你将会从 channels 表中删除通知代码，并将其替换为在 TodoItem 表中进行的一些更改。 

<h2><a name="update-scripts"></a>更新生成的推送通知代码</h2>

[WACOM.INCLUDE [mobile-services-create-new-push-vs2013-2](../includes/mobile-services-create-new-push-vs2013-2.md)]

<h2><a name="test"></a>在应用程序中测试推送通知</h2>

1. 在 Visual Studio 中，按 F5 键运行应用程序。

2. 在应用中的"插入 TodoItem"内键入文本，然后单击"保存"。

   	![][13]

   	请注意，完成插入后，应用将会接收来自 WNS 的推送通知。

   	![][14]

## <a name="next-steps"> </a>后续步骤

本教程演示了有关如何使 Windows 应用商店应用程序处理移动服务中的数据的基础知识。接下来，建议您完成下列教程之一，这些教程是基于本教程中创建的 GetStartedWithData 应用程序制作的：

+ [通知中心入门]
  <br/>了解如何在 Windows 应用商店应用程序中利用通知中心。

+ [向订阅者发送通知]
	<br/>了解用户如何注册并接收其感兴趣的类别的推送通知。

+ [向用户发送通知]
	<br/>了解如何通过移动服务向任何设备上的特定用户发送推送通知。

+ [向用户发送跨平台通知]
	<br/>了解如何使用模板通过移动服务发送推送通知，而无需在后端处理特定于平台的负载。

建议进一步了解以下移动服务主题：

* [数据处理入门]
  <br/>了解有关使用移动服务存储和查询数据的详细信息。

* [身份验证入门]
  <br/>了解如何使用 Windows 帐户对应用程序的用户进行身份验证。

* [移动服务服务器脚本参考]
  <br/>了解有关注册和使用服务器脚本的详细信息。

* [移动服务 .NET 操作方法概念性参考]
  <br/>了解有关如何将移动服务与 .NET 一起使用的详细信息。

<!-- Anchors. -->
[注册用于推送通知的应用程序并配置移动服务]: #register
[更新生成的推送通知代码]: #update-scripts
[插入数据以接收通知]: #test
[后续步骤]:#next-steps

<!-- Images. -->











[13]: ./media/mobile-services-windows-store-dotnet-get-started-push/mobile-quickstart-push1.png
[14]: ./media/mobile-services-windows-store-dotnet-get-started-push/mobile-quickstart-push2.png




<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push
[向应用程序用户推送通知]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts
[JavaScript 和 HTML]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-push-js

[Azure 管理门户]: https://manage.windowsazure.cn/
[wns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library/
[使用脚本验证和修改数据]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-validate-modify-data-server-scripts
[使用分页优化查询]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-add-paging-data
[通知中心入门]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-get-started
[什么是通知中心？?]: /zh-cn/documentation/articles/notification-hubs-overview/
[向订阅者发送通知]: /zh-cn/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/
[向用户发送通知]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users/
[向用户发送跨平台通知]: /zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-push-notifications-app-users-xplat-mobile-services/
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
