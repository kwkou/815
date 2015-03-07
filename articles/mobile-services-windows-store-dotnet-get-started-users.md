<properties pageTitle="身份验证入门（Windows 应用商店）| 移动开发人员中心" metaKeywords="authentication, FAcebook, GOogle, Twitter, Microsoft Account, login" description="了解如何使用移动服务通过各种身份提供商（包括 Google、Facebook、 Twitter 和 Microsoft）验证 Windows 应用商店应用程序用户的身份。" metaCanonical="" services="mobile" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="Glenn Gailey" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-store" ms.devlang="dotnet" ms.topic="article" ms.date="09/23/2014" ms.author="glenga" />

# 向移动服务应用程序添加身份验证 

[WACOM.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]		

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">
<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>
<p>单击右侧的剪辑可观看本教程的视频版本。</p>
</div>
<div class="dev-onpage-video-wrapper" style="display:none"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Introduction-to-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/get-started-with-users-windows-store-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:04</span></div>
</div> 

>[WACOM.NOTE]使用移动服务 JavaScript 客户端库时，Windows Phone 应用商店 8.1 应用程序当前不支持身份验证。如果您有一个使用 JavaScript 客户端的通用 Windows 应用程序项目，则目前将无法对 Windows Phone 上用户进行身份验证。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]
5. [在客户端上存储身份验证令牌]

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。 

>[WACOM.NOTE]本教程演示由移动服务管理的使用各种身份提供商的身份验证流程。此方法易于配置，并且支持多个提供商。若要改用 Live Connect 和客户端管理的身份验证并在 Windows 应用商店应用程序中提供单一登录体验，请参阅主题[使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录]。通过使用客户端管理的身份验证，您的应用程序可以访问身份提供商维护的其他用户数据。通过在服务器脚本中调用 **user.getIdentities()** 函数，可以在移动服务中获取同样的用户数据。有关详细信息，请参阅[这篇博文](http://go.microsoft.com/fwlink/p/?LinkId=506605)。

## <a name="register"></a> 注册应用程序以进行身份验证并配置移动服务

[WACOM.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)] 

<ol start="5">
<li><p>（可选）完成 <a href="/zh-cn/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication/">注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证</a>。</p>

    <div class="dev-callout"><b>注意</b>
	<p>由于此步骤只适用于 Microsoft 帐户登录提供程序，因此是可选的。将 Windows 应用商店应用程序包信息注册到移动服务后，客户端可以重复使用 Microsoft 帐户登录凭据来提供单一登录体验。如果你不执行此操作，则每次调用 login 方法时，系统都会向 Microsoft 帐户登录用户显示登录提示。如果你打算使用 Microsoft 帐户标识提供者，请完成此步骤。</p>
    </div>
</li>
</ol>

## <a name="permissions"></a> 将权限限制给已经过身份验证的用户

[WACOM.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)] 

<ol start="3">
<li><p>在 Visual Studio 2012 Express for Windows 8 中，打开在完成教程 <a href="/zh-cn/documentation/articles/mobile-services-windows-store-get-started">移动服务入门</a>。</p></li> 
<li><p>按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（"未授权"）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a> 向应用程序添加身份验证

[WACOM.INCLUDE [mobile-services-windows-dotnet-authenticate-app](../includes/mobile-services-windows-dotnet-authenticate-app.md)] 

## <a name="tokens"></a>在客户端上存储授权令牌

[WACOM.INCLUDE [mobile-services-windows-store-dotnet-authenticate-app-with-token](../includes/mobile-services-windows-store-dotnet-authenticate-app-with-token.md)] 

## <a name="next-steps"> </a>后续步骤

在下一教程[移动服务用户的服务端授权][使用脚本为用户授权]中，您将使用移动服务根据已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。请在[移动服务 .NET 操作方法概念性参考]中了解有关如何将移动服务与 .NET 一起使用的详细信息。

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[在客户端上存储身份验证令牌]: #tokens
[后续步骤]:#next-steps


<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[使用 Live Connect 实现对 Windows 应用商店应用程序的单一登录]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-store-get-started/
[数据处理入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-data/
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-push/
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-authorize-users-in-scripts
[JavaScript 和 HTML]: /zh-cn/documentation/articles/mobile-services-windows-store-javascript-get-started-users/

[Azure 管理门户]: https://manage.windowsazure.cn/
[移动服务 .NET 操作方法概念性参考]: /zh-cn/documentation/articles/mobile-services-windows-dotnet-how-to-use-client-library
[注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证]: /zh-cn/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication
