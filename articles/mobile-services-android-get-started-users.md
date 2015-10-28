<properties 
	pageTitle="身份验证入门 (Android) | 移动开发人员中心" 
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Android 应用程序的用户进行身份验证。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/01/2015" 
	wacn.date="10/03/2015"/>

# 向移动服务 Android 应用添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users](../includes/mobile-services-selector-get-started-users.md)]


## 摘要

<!--
<div class="dev-onpage-video-clear clearfix">
<div class="dev-onpage-left-content">



<p>本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。</p>

</div>

<div class="dev-onpage-video-wrapper"><a href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Android-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="label">观看教程</a> <a style="background-image: url('/media/devcenter/mobile/videos/mobile-android-get-started-authentication-180x120.png') !important;" href="http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Android-Getting-Started-with-Authentication-in-Windows-Azure-Mobile-Services" target="_blank" class="dev-onpage-video"><span class="icon">播放视频</span></a><span class="time">10:42</span></div>
</div>
-->


本教程将指导你完成在应用程序中启用身份验证的基本步骤。


##先决条件

[AZURE.INCLUDE [mobile-services-android-prerequisites](../includes/mobile-services-android-prerequisites.md)]

## 注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

## 将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)]

3. 在 Android Studio 中，打开你在完成[移动服务入门]教程后创建的项目。 

4. 然后，从“运行”菜单中单击“运行应用程序”；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。

	 发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 _TodoItem_ 表现在要求身份验证。

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

## 向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-android-authenticate-app](../includes/mobile-services-android-authenticate-app.md)]

## <a name="cache-tokens"></a>在客户端上缓存身份验证令牌

[AZURE.INCLUDE [mobile-services-android-authenticate-app-with-token](../includes/mobile-services-android-authenticate-app-with-token.md)]

## <a name="refresh-tokens"></a>刷新令牌缓存

[AZURE.INCLUDE [mobile-services-android-authenticate-app-refresh-token](../includes/mobile-services-android-authenticate-app-refresh-token.md)]



## <a name="next-steps"></a>后续步骤

在下一教程[使用脚本为用户授权]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。

<!-- Anchors. -->
[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Store authentication tokens on the client]: #cache-tokens
[Refresh expired tokens]: #refresh-tokens
[Next Steps]: #next-steps

<!-- Images. -->




[4]: ./media/mobile-services-android-get-started-users/mobile-services-selection.png
[5]: ./media/mobile-services-android-get-started-users/mobile-service-uri.png

[13]: ./media/mobile-services-android-get-started-users/mobile-identity-tab.png
[14]: ./media/mobile-services-android-get-started-users/mobile-portal-data-tables.png
[15]: ./media/mobile-services-android-get-started-users/mobile-portal-change-table-perms.png


<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Single sign-on for Windows Store apps by using Live Connect]: /documentation/articles/mobile-services-windows-store-dotnet-single-sign-on
[移动服务入门]: /documentation/articles/mobile-services-android-get-started
[Add Mobile Services to an existing app]: /documentation/articles/mobile-services-android-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-android-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push
[使用脚本为用户授权]: /documentation/articles/mobile-services-javascript-backend-service-side-authorization
[Azure Management Portal]: https://manage.windowsazure.cn/

<!---HONumber=71-->