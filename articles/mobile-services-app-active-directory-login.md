<properties linkid="develop-mobile-how-to-guides-register-for-microsoft-waad-authentication" urlDisplayName="Register for Azure Active Directory Authentication" pageTitle="Register for Azure Active Directory authentication - Mobile Services" metaKeywords="Azure registering application, Azure Active Directory authentication, application authenticate, authenticate mobile services" description="Learn how to register for Azure Active Directory authentication in your Azure Mobile Services application." metaCanonical="" disqusComments="0" umbracoNaviHide="1" title="Register your apps to use an Azure Active Directory Account login" authors="" />


# 注册应用程序以使用 Azure Active Directory 帐户登录

本主题说明如何注册你的应用程序，以便能够使用 Azure Active Directory 作为 Azure 移动服务的身份验证提供程序。

<div class="dev-callout"><b>说明</b>

<p>如果你还希望从 Windows 应用商店应用程序提供用于单一登录 (SSO) 或推送通知的客户端驱动的身份验证，请考虑同时将你的应用程序注册到 Windows 应用商店。有关详细信息，请参阅<a href="/zh-cn/develop/mobile/how-to-guides/register-for-single-sign-on">注册 Windows 应用商店应用程序以进行 Windows Live Connect 身份验证</a>。</p>
</div>

1. 登录到 [Azure 管理门户][]。

2. 导航到管理门户中的“Active Directory”，然后单击你的目录 。

   ![][1] 

3. 单击“应用程序” 选项卡，然后单击“添加应用程序” 。

   ![][2]


4. 遵照新建应用程序向导中的说明，为 XXX 选择“Web 应用程序和/或 Web API” 。启用“单一登录”。当系统提示你输入“应用程序 URL”时，请粘贴移动服务应用程序 URL 。


5. *** 即将补充更多内容 ***

现在，你可以通过向移动服务提供客户端 ID 和客户端密钥值，使用 Azure Active Directory 在应用程序中进行身份验证。

  [注册 Windows 应用商店应用程序以进行 Windows Live Connect 身份验证]: /zh-cn/develop/mobile/how-to-guides/register-for-single-sign-on
  [Azure 管理门户]: https://manage.windowsazure.cn/
[1]: ./media/mobile-services-app-active-directory-login/mobile-services-live-connect-add-app.png
[2]: ./media/mobile-services-app-active-directory-login/mobile-live-connect-app-api-settings.png
