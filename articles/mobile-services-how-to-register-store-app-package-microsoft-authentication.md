<properties linkid="develop-mobile-how-to-guides-register-windows-store-app-server-auth" urlDisplayName="Shared Access Signature Part 1" pageTitle="注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证" metaKeywords="" description="了解如何在 Azure 移动服务应用程序中注册 Windows 应用商店应用程序以进行 Microsoft 身份验证" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your Windows Store app package for Microsoft authentication" authors="glenga" solutions="" manager="" editor="" />

# 注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证

Azure 移动服务支持客户端驱动的和服务器驱动的身份验证方法。服务器驱动的身份验证使用标识提供程序（包括 Microsoft 帐户）。如果你将 Microsoft 帐户用于服务器驱动的身份验证但未将应用程序注册到移动服务，则每次请求身份验证时，系统都将提示用户提供凭据。如果你注册了应用程序，则 Microsoft 帐户登录凭据将被缓存并可用于身份验证，并且系统不会再次提示用户提供凭据。本主题说明如何注册你的 Windows 应用商店应用程序包，以便在使用 Azure 移动服务进行身份验证时获得更好的 Microsoft 帐户登录体验。 

>[WACOM.NOTE]使用 Visual Studio 2013 可以轻松地将 Windows 应用商店应用程序包注册到移动服务。有关详细信息，请参阅<a href="http://msdn.microsoft.com/library/windows/apps/xaml/dn263182.aspx">快速入门：为移动服务添加推送通知</a>（位于 Windows 开发中心）。

使用客户端管理的身份验证可以通过 Live Connect 在 Windows 设备上提供单一登录体验。如果你使用 Live Connect API，则不需要完成本主题中的步骤。有关详细信息，请参阅[使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证]。   

[WACOM.INCLUDE [mobile-services-register-windows-store-app](../includes/mobile-services-register-windows-store-app.md)]

注册应用程序包后，在调用 <a href="http://go.microsoft.com/fwlink/p/?LinkId=311594" target="_blank">LoginAsync</a> 方法时，请记得为 <em>useSingleSignOn</em> 提供 <strong>true</strong> 值。这样，你的用户在使用 Microsoft 帐户时可以获得更好的登录体验。

<!-- Anchors. -->
<!-- Images. -->


<!-- URLs. -->
[推送通知入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-push/
[使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-single-sign-on
[Get started with users C#]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[用户 JavaScript 入门]: /zh-cn/documentation/articles/mobile-services-javascript-backend-windows-store-dotnet-get-started-with-users-js/
