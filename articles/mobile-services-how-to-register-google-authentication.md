<properties 
	pageTitle="注册以进行 Google 身份验证 - 移动服务" 
	description="了解如何注册你的应用程序，以便使用 Google 在 Azure 移动服务中进行身份验证。" 
	services="mobile-services" 
	documentationCenter="android" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="06/11/2015" 
	wacn.date="10/03/2015"/>

# 注册应用程序以便在移动服务中进行 Google 登录

[AZURE.INCLUDE [mobile-services-selector-register-identity-provider](../includes/mobile-services-selector-register-identity-provider.md)]

本主题说明如何注册你的应用程序，以便能够使用 Google 在 Azure 移动服务中进行身份验证。

>[AZURE.NOTE]本教程与 [Azure 移动服务](/home/features/identity/)有关，该解决方案可帮助你构建任意平台的可缩放移动应用程序。使用移动服务可以轻松地同步数据、对用户进行身份验证和发送推送通知。此页支持[身份验证入门](/documentation/articles/mobile-services-ios-get-started-users)教程，该教程演示如何将用户登录到你的应用程序。<br/>如果这是你第一次体验移动服务，请先完成[移动服务入门](/documentation/articles/mobile-services-ios-get-started)教程。

若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Google 帐户。若要新建一个 Google 帐户，请转到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268302" target="_blank">accounts.google.com</a>。

1. 导航到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268303" target="_blank">Google API</a> 网站，使用 Google 帐户凭据登录，单击“创建项目”，提供“项目名称”，然后单击“创建”。

   	![Google API 新项目](./media/mobile-services-how-to-register-google-authentication/mobile-services-google-new-project.png)

2. 单击“许可屏幕”，选择你的“电子邮件地址”，输入“产品名称”，然后单击“保存”。

3. 单击“API 和身份验证”>“凭据”>“创建新的客户端 ID”。

   	![创建新的客户端 ID](./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client.png)

4. 选择“Web 应用程序”，在“授权的 JavaScript 源”中键入移动服务 URL，将“授权的重定向 URI”中生成的 URL 替换为你的移动服务 URL 后接路径 `/login/google`，然后单击“创建客户端 ID”。

	>[AZURE.NOTE]对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL 后接用作 .NET 服务的移动服务的路径 _signin-google_，例如 `https://todolist.azure-mobile.net/signin-google`。

   	![](./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client2.png)

5. 在“Web 应用程序的客户端 ID”下面，记下“客户端 ID”和“客户端密钥”的值。

   	![客户端凭据](./media/mobile-services-how-to-register-google-authentication/mobile-services-google-create-client3.png)

    >[AZURE.IMPORTANT]客户端密钥是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用分发。

现在，你可以将移动服务配置为使用 Google 登录名在应用中进行身份验证。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

[Google API]: http://go.microsoft.com/fwlink/p/?LinkId=268303
[身份验证入门]: /documentation/articles/mobile-services-windows-store-dotnet-get-started-users
[Azure 管理门户]: https://manage.windowsazure.cn/
<!---HONumber=71-->