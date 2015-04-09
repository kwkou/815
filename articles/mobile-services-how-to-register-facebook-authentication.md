<properties linkid="develop-mobile-how-to-guides-register-for-facebook-authentication" urlDisplayName="Register for Facebook Authentication" pageTitle="注册以进行 Facebook 身份验证 - 移动服务" metaKeywords="Azure Facebook, Azure Facebook, Azure authenticate Mobile Services" description="了解如何在 Azure 移动服务应用程序中使用 Facebook 身份验证。" metaCanonical="" services="mobile-services" documentationCenter="Mobile" title="Register your apps for Facebook authentication with Mobile Services" authors="glenga" solutions="" manager="" editor="" />

# 注册应用程序以使用 Facebook 在移动服务中进行身份验证

本主题说明如何注册你的应用程序，以便能够使用 Facebook 在 Azure 移动服务中进行身份验证。 

>[WACOM.NOTE] 本教程有关 [Azure 移动服务]，该解决方案可帮助你构建任意平台的可缩放移动应用程序。使用移动服务可以轻松地同步数据、对用户进行身份验证和发送推送通知。此页支持<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started-users/">身份验证入门</a>教程，该教程演示如何将用户登录到你的应用程序。如果这是你第一次体验移动服务，请先完成<a href="/zh-cn/documentation/articles/mobile-services-ios-get-started/">移动服务入门教程</a>。
	
若要完成本主题中的过程，你必须拥有一个包含已验证电子邮件地址的 Facebook 帐户和一个手机号码。若要创建新的 Facebook 帐户，请转至 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268285" target="_blank">facebook.com</a>。

1. 导航到 <a href="http://go.microsoft.com/fwlink/p/?LinkId=268286" target="_blank">Facebook 开发人员</a>网站，然后使用 Facebook 帐户凭据登录。

2. （可选）如果你尚未注册，请依次单击**"应用程序"**、**"以开发人员身份注册"**，接受用户政策，然后遵照注册步骤操作。 

   	![][0]

3. 单击**"应用程序"**，然后单击**"创建新应用程序"**。

   	![][1]

4. 为应用程序选择唯一的名称，选择**"页面应用程序"**，单击**"创建应用程序"**，然后完成质询问题。

   	![][2]

	随即会将应用程序注册到 Facebook 

5. 单击**"设置"**，在**"应用程序域"**中键入移动服务的域。另外，输入**联系人电子邮件**，然后单击**"添加平台"**并选择**"网站"**。

   	![][3]

6. 在**"站点 URL"**中键入移动服务的 URL，然后单击**"保存更改"**。

	![][4]

7. 单击**"显示"**，出现请求时请提供你的密码，然后记下**"应用程序 ID"**和**"应用程序密钥"**的值。 

   	![][5]

	> [AZUTE.NOTE] **安全说明**
	应用程序密钥是一个非常重要的安全凭据。请勿与任何人分享此密钥或将密钥随应用程序分发。


8. 单击**"高级"**选项卡，在**"有效的 OAuth 重定向 URI"**中键入附加了路径 /login/facebook 的移动服务 URL，然后单击**"保存更改"**。 

	> [AZURE.NOTE] 对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL，后接用作 .NET 服务的移动服务的路径 signin-facebook，例如 <code>https://todolist.azure-mobile.cn/signin-facebook</code>。  
	
	![][7]

9. 为其定义新应用程序的 Facebook 帐户是应用程序的管理员，有权以管理员身份访问应用程序。若要对其他 Facebook 帐户进行身份验证，他们需要访问应用程序。此步骤授予常规公共访问权限，以便应用程序可以对其他 Facebook 帐户进行身份验证。单击**"状态和复查"**。然后单击**"是"**以启用常规公共访问权限。

    ![][6]



现在，你可以通过向移动服务提供应用程序 ID 和应用程序密钥值，使用 Facebook 登录在应用程序中进行身份验证。  

<!-- Anchors. -->

<!-- Images. -->
[0]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-developer-register.png
[1]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-add-app.png
[2]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-new-app-dialog.png
[3]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app.png
[4]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app-2.png
[5]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-completed.png
[6]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app-general-public.png
[7]: ./media/mobile-services-how-to-register-facebook-authentication/mobile-services-facebook-configure-app-3.png

<!-- URLs. -->
[Facebook 开发人员]: http://go.microsoft.com/fwlink/p/?LinkId=268286
[身份验证入门]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-get-started-users/
[Azure 管理门户]: https://manage.windowsazure.cn/
[Azure 移动服务]: /zh-cn/services/mobile-services/
