<properties linkid="develop-mobile-how-to-guides-register-for-active-directory-authentication" urlDisplayName="Register for Azure Active Directory Authentication" pageTitle="Register for Azure Active Directory authentication - Mobile Services" metaKeywords="Azure registering application, Azure Active Directory authentication, application authenticate, authenticate mobile services" description="Learn how to register for Azure Active Directory authentication in your Mobile Services application." title="Register your account to use an Azure Active Directory account login" authors="wesmc" services="mobile-services" documentationCenter="Mobile" />

# 注册应用程序以使用 Azure Active Directory 帐户登录

本主题说明如何注册你的应用程序，以便能够使用 Azure Active Directory 作为 Azure 移动服务的身份验证提供程序。

> [WACOM.NOTE] 如果你要使用 Azure Active Directory 为单一登录 (SSO) 提供客户端驱动的身份验证，请参阅[使用 Active Directory 身份验证库单一登录对应用程序进行身份验证][]教程。

1.  登录到 [Azure 管理门户][]，单击“移动服务”，然后单击你的移动服务 。

    ![][]

2.  单击移动服务的“标识”选项卡 。

    ![][1]

3.  向下滚动到“Azure Active Directory”标识提供者部分，并复制其中列出的“应用程序 URL” 。

    ![][2]

4.  导航到管理门户中的“Active Directory”，然后单击你的目录 。

    ![][3]

5.  单击顶部的“应用程序” 选项卡，然后单击“添加” 以添加应用程序。

    ![][4]

6.  单击“添加我的组织正在开发的应用程序” 。

7.  在“添加应用程序”向导中，为应用程序输入“名称”，并单击“Web 应用程序和/或 Web API”类型 。然后单击以继续。

    ![][5]

8.  在“登录 URL”框中，粘贴你从移动服务 Active Directory 标识提供者设置中复制的应用程序 ID 。另外，请在“应用程序 ID URI”框中输入唯一的资源标识符 。应用程序将使用该 URI 向 Azure Active Directory 提交单一登录请求。然后单击以继续。

    ![][6]

9.  添加应用程序后，请单击“配置” 选项卡。然后单击相应的按钮以复制应用程序的“客户端 ID” 。

    ![][7]

10. 返回到移动服务的“标识”选项卡 。在底部粘贴 Azure Active Directory 标识提供者的“客户端 ID”设置 。然后单击“保存” 。

    ![][8]

现在，你可以使用 Azure Active Directory 在应用程序中进行身份验证。

  [使用 Active Directory 身份验证库单一登录对应用程序进行身份验证]: /zh-cn/documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  []: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-selection.png
  [1]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-identity-tab.png
  [2]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-copy-app-url-waad-auth.png
  [3]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-select-ad-waad-auth.png
  [4]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-waad-idenity-tab-selection.png
  [5]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-1-waad-auth.png
  [6]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-2-waad-auth.png
  [7]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-clientid-waad-auth.png
  [8]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-clientid-pasted-waad-auth.png
