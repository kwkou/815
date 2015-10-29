<properties
	pageTitle="身份验证入门 (JavaScript) | Windows Azure"
	description="了解如何使用移动服务通过提供各种标识提供程序（包括 Google、Facebook、Twitter 和 Microsoft）对 Windows 应用商店 JavaScript 应用程序的用户进行身份验证。"
	services="mobile-services"
	documentationCenter="windows"
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags 
	ms.service="mobile-services"
	ms.date="06/16/2015"
	wacn.date="10/22/2015"/>

# 向移动服务应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-selector-get-started-users-legacy](../includes/mobile-services-selector-get-started-users-legacy.md)]

本主题说明如何通过应用程序对 Azure 移动服务中的用户进行身份验证。在本教程中，你将要使用移动服务支持的标识提供程序向快速入门项目添加身份验证。在移动服务成功完成身份验证和授权后，将显示用户 ID 值。

本教程将指导你完成在应用程序中启用身份验证的以下基本步骤：

1. [注册应用程序以进行身份验证并配置移动服务]
2. [将表权限限制给已经过身份验证的用户]
3. [向应用程序添加身份验证]
4. [在客户端上存储身份验证令牌]

本教程基于移动服务快速入门。此外，还必须先完成[移动服务入门]教程。

>[AZURE.NOTE]本教程演示如何管理移动服务使用各种标识提供程序的身份验证流。此方法易于配置，并支持多个提供者。若要改用 Live Connect 进行客户端托管的身份验证并在 Windows Phone 应用中提供单一登录体验，请参阅主题[使用 Live Connect 实现对 Windows 应用商店应用的单一登录]。通过使用客户端管理身份验证，你的应用程序有权访问所维护的标识提供程序的其他用户数据。你可以通过移动服务中获取相同的用户数据，通过调用 **user.getIdentities()** 服务器脚本中的函数。有关详细信息，请参阅[此文章](http://go.microsoft.com/fwlink/p/?LinkId=506605)。

##<a name="register"></a>注册应用程序以进行身份验证并配置移动服务

[AZURE.INCLUDE [mobile-services-register-authentication](../includes/mobile-services-register-authentication.md)]

<ol start="5">
<li><p>（可选）完成<a href="/documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication/">注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证</a>中的步骤。</p>

    
	<p>请注意，由于此步骤只适用于 Microsoft 帐户登录提供程序，因此是可选的。将 Windows 应用商店应用程序包信息注册到移动服务后，客户端可以重复使用 Microsoft 帐户登录凭据来提供单一登录体验。如果你不执行此操作，则每次调用 login 方法时，系统都会向 Microsoft 帐户登录用户显示登录提示。如果你打算使用 Microsoft 帐户标识提供者，请完成此步骤。</p>
</li>
</ol>
你的移动服务和应用现已配置为使用你选择的身份验证提供程序。

##<a name="permissions"></a>将权限限制给已经过身份验证的用户

[AZURE.INCLUDE [mobile-services-restrict-permissions-javascript-backend](../includes/mobile-services-restrict-permissions-javascript-backend.md)]

<ol start="3">
<li><p>在 Visual Studio 2012 Express for Windows 8 中，打开你在完成教程<a href="/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started/">移动服务入门</a>时创建的项目。</p></li> 
<li><p>按 F5 键运行这个基于快速入门的应用程序；验证启动该应用程序后，是否会引发状态代码为 401（“未授权”）的未处理异常。</p>
   
   	<p>发生此异常的原因是应用程序尝试以未经身份验证的用户身份访问移动服务，但 <em>TodoItem</em> 表现在要求身份验证。</p></li>
</ol>

接下来，你需要更新应用程序，以便在从移动服务请求资源之前对用户进行身份验证。

##<a name="add-authentication"></a>向应用程序添加身份验证

[AZURE.INCLUDE [mobile-services-windows-store-javascript-authenticate-app](../includes/mobile-services-windows-store-javascript-authenticate-app.md)]

##<a name="tokens"></a>在客户端上存储授权令牌

[AZURE.INCLUDE [mobile-services-windows-store-javascript-authenticate-app-with-token](../includes/mobile-services-windows-store-javascript-authenticate-app-with-token.md)]

## <a name="next-steps"></a>后续步骤

在下一教程[移动服务用户的服务端授权][Authorize users with scripts]中，你将使用移动服务基于已进行身份验证的用户提供的用户 ID 值来筛选移动服务返回的数据。


<!-- Anchors. -->
[注册应用程序以进行身份验证并配置移动服务]: #register
[将表权限限制给已经过身份验证的用户]: #permissions
[向应用程序添加身份验证]: #add-authentication
[在客户端上存储身份验证令牌]: #tokens
[Next Steps]: #next-steps


<!-- URLs. -->
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[使用 Live Connect 实现对 Windows 应用商店应用的单一登录]: /documentation/articles/mobile-services-windows-store-javascript-single-sign-on
[移动服务入门]: /documentation/articles/mobile-services-windows-store-get-started
[Get started with data]: /documentation/articles/mobile-services-windows-store-javascript-get-started-data
[Get started with authentication]: /documentation/articles/mobile-services-windows-store-javascript-get-started-users
[Get started with push notifications]: /documentation/articles/mobile-services-windows-store-javascript-get-started-push
[Authorize users with scripts]: /documentation/articles/mobile-services-windows-store-javascript-authorize-users-in-scripts

[Azure Management Portal]: https://manage.windowsazure.cn/
[Register your Windows Store app package for Microsoft authentication]: /documentation/articles/mobile-services-how-to-register-store-app-package-microsoft-authentication

<!---HONumber=74-->