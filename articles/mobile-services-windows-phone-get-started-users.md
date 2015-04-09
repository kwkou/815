<properties linkid="develop-mobile-tutorials-get-started-with-users-wp8" urlDisplayName="身份验证入门" pageTitle="身份验证 （Windows Phone） 入门 |移动开发人员中心" metaKeywords="" description="了解如何使用移动服务通过各个身份提供平台（包括 Google、 Facebook、 Twitter 和 Microsoft）对 Windows Phone 应用程序的用户进行身份验证。" metaCanonical="" services="" documentationCenter="Mobile" title="Get started with authentication in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-windows-phone" ms.devlang="dotnet" ms.topic="article" ms.date="09/23/2014" ms.author="glenga" />

# 向移动服务应用程序添加身份验证

[WACOM.INCLUDE [mobile-services-selector-get-started-users-legacy](../includes/mobile-services-selector-get-started-users-legacy.md)]

<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">

<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>
</div>
<div class="dev-onpage-video-wrapper"><a href="http://go.microsoft.com/fwlink/?LinkId=298631" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-wp8-get-started-authentication-180x120.png') !important;" href="http://go.microsoft.com/fwlink/?LinkId=298631" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a> <span class="time">10:50</span></div>
</div>  

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。 

>[WACOM.NOTE]本教程演示由移动服务管理的使用各种身份提供商的身份验证流程。此方法易于配置，并且支持多个提供商。若要改用 Live Connect（提供客户端托管的身份验证），在 Windows Phone 应用程序中提供单一登录体验，请参阅主题[使用 Live Connect 实现对 Windows Phone 应用程序的单一登录]。通过使用客户端管理的身份验证，您的应用程序可以访问身份提供商维护的其他用户数据。通过在服务器脚本中调用 **user.getIdentities()** 函数，可以在移动服务中获取同样的用户数据。有关详细信息，请参阅[这篇博文](http://go.microsoft.com/fwlink/p/?LinkId=506605)。

## <a name="register"></a>注册应用程序以进行身份验证并配置移动服务


[WACOM.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)] 


## <a name="permissions"></a>将权限限制给已经过身份验证的用户


[WACOM.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)] 


3. 在 Visual Studio 2012 Express for Windows Phone 中，打开你在完成教程[移动服务入门]后创建的项目。 

4. 按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（"未授权"）的未处理异常。 
   
   	发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## <a name="add-authentication"></a>向应用程序添加身份验证

[WACOM.INCLUDE [mobile-services-windows-phone-authenticate-app](../includes/mobile-services-windows-phone-authenticate-app.md)]

## <a name="tokens"></a>在客户端上存储授权令牌

[WACOM.INCLUDE [mobile-services-windows-phone-authenticate-app-with-token](../includes/mobile-services-windows-phone-authenticate-app-with-token.md)] 

## <a name="next-steps"> </a>后续步骤

在下一教程[移动服务用户的服务端授权][使用脚本进行用户授权]中，您将使用移动服务根据已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。 

<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[后续步骤]:#next-steps

<!-- Images. -->
[1]: ./media/mobile-services-wp8-get-started-users/mobile-services-selection.png
[2]: ./media/mobile-services-wp8-get-started-users/mobile-service-uri.png
[3]: ./media/mobile-services-wp8-get-started-users/mobile-identity-tab.png
[4]: ./media/mobile-services-wp8-get-started-users/mobile-portal-data-tables.png
[5]: ./media/mobile-services-wp8-get-started-users/mobile-portal-change-table-perms.png

<!-- URLs. -->
[提交应用程序页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[移动服务入门]: /zh-cn/documentation/articles/mobile-services-windows-phone-get-started
[使用脚本为用户授权]: /zh-cn/documentation/articles/mobile-services-windows-phone-authorize-users-in-scripts
[Azure 管理门户]: https://manage.windowsazure.cn/
[使用 Live Connect 实现对 Windows Phone 应用程序的单一登录]: /zh-cn/documentation/articles/mobile-services-windows-phone-single-sign-on
