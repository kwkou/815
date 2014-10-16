<properties linkid="develop-mobile-how-to-guides-register-windows-store-app-server-auth" urlDisplayName="Shared Access Signature Part 1" pageTitle="Register your Windows Store app package for Microsoft authentication" metaKeywords="" description="Learn how to register your Windows Store app for Microsoft authentication in your Azure Mobile Services application" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your Windows Store app package for Microsoft authentication" authors="glenga" solutions="" manager="" editor="" />

# 注册 Windows 应用商店应用程序包以进行 Microsoft 身份验证

Azure 移动服务支持客户端驱动的和服务器驱动的身份验证方法。服务器驱动的身份验证使用标识提供者（包括 Microsoft 帐户）。如果你将 Microsoft 帐户用于服务器驱动的身份验证但未将应用程序注册到移动服务，则每次请求身份验证时，系统将提示用户提供凭据。如果你注册了应用程序，则 Microsoft 帐户登录凭据将被缓存并可用于身份验证，并且系统不会再次提示用户提供凭据。本主题说明如何注册你的 Windows 应用商店应用程序包，以便在使用 Azure 移动服务进行身份验证时获得更好的 Microsoft 帐户登录体验。

使用客户端驱动的身份验证可以通过 Live Connect 在 Windows 设备上提供单一登录体验。如果你使用了 Live Connect API，则不需要完成本主题中的步骤。有关详细信息，请参阅[使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证][]。

> [WACOM.NOTE] 使用 Visual Studio 2013 可以轻松地将 Windows 应用商店应用程序包注册到移动服务。有关详细信息，请参阅[快速入门：为移动服务添加推送通知][]（位于 Windows 开发中心）。

[WACOM.INCLUDE [mobile-services-register-windows-store-app](../includes/mobile-services-register-windows-store-app.md)]

注册应用程序包后，在调用 [LoginAsync][] 方法时，请记得为 *useSingleSignOn* 提供 "true" 值。这样，你的用户在使用 Microsoft 帐户时可以获得更好的登录体验。

  [使用 Live Connect 单一登录对 Windows 应用商店应用程序进行身份验证]: /zh-cn/develop/mobile/tutorials/single-sign-on-windows-8-dotnet
  [快速入门：为移动服务添加推送通知]: http://go.microsoft.com/fwlink/p/?LinkId=309101
  [mobile-services-register-windows-store-app]: ../includes/mobile-services-register-windows-store-app.md
  [LoginAsync]: http://go.microsoft.com/fwlink/p/?LinkId=311594
