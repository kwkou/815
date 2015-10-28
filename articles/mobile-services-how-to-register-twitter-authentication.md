<properties 
	pageTitle="注册以进行 Twitter 身份验证 - 移动服务" 
	description="了解如何在 Azure 移动服务应用程序中使用 Twitter 身份验证。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="Azure 管理门户"/>

#注册应用以便在移动服务中进行 Twitter 登录

[AZURE.INCLUDE [mobile-services-selector-register-identity-provider](../includes/mobile-services-selector-register-identity-provider.md)]

本主题说明如何注册你的应用程序，以便能够使用 Twitter 在 Azure 移动服务中进行身份验证。

>[AZURE.NOTE]本教程与 [Azure 移动服务](http://azure.microsoft.com/services/mobile-services/)有关，该解决方案可帮助你构建任意平台的可缩放移动应用程序。使用移动服务可以轻松地同步数据、对用户进行身份验证和发送推送通知。此页支持<a href="http://azure.microsoft.com/documentation/articles/mobile-services-ios-get-started-users/">身份验证入门</a>教程，该教程演示如何将用户登录到你的应用程序。如果这是你第一次体验移动服务，请先完成<a href="http://azure.microsoft.com/documentation/articles/mobile-services-ios-get-started/">移动服务入门</a>教程。

若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Twitter 帐户。若要创建新的 Twitter 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkID=268287" target="_blank">twitter.com</a>。

1. 导航到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268300" target="_blank">Twitter 开发人员</a>网站，使用你的 Twitter 帐户凭据登录，然后单击“创建新应用程序”。

   	![][1]

2. 输入应用的“名称”、“描述”和“网站”值，然后键入在“回调 URL”中附加了路径“/login/twitter”的移动服务的 URL。

	>[AZURE.NOTE]对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL，后接移动服务用作 .NET 服务的路径 _signin-twitter_，例如 <code>https://todolist.azure-mobile.net/signin-twitter</code>。

   	![][2]

3.  在页面的底部，阅读并接受条款，然后单击“创建 Twitter 应用程序”。



   	随即将会注册应用程序并显示应用程序详细信息。

6. 在应用仪表板中单击“密钥和访问令牌”选项卡，并记下“使用者密钥”和“使用者机密”的值。



    > [AZURE.NOTE]使用者机密是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用分发。

7. 单击“设置”选项卡，向下滚动并选中“允许使用此应用程序进行 Twitter 登录”，然后单击“更新此 Twitter 应用程序的设置”。



现在，你可以通过向移动服务提供使用者密钥和使用者机密值，使用 Twitter 登录在应用程序中进行身份验证。

<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-developers.png
[2]: ./media/mobile-services-how-to-register-twitter-authentication/mobile-services-twitter-register-app1.png


<!-- URLs. -->

[Twitter Developers]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[Get started with authentication]: /documentation/articles/mobile-services-javascript-backend-windows-universal-dotnet-get-started-users
[Azure Management Portal]: https://manage.windowsazure.cn
 

<!---HONumber=71-->