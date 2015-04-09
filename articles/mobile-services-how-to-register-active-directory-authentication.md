<properties linkid="develop-mobile-how-to-guides-register-for-active-directory-authentication" urlDisplayName="Register for Azure Active Directory Authentication" pageTitle="注册以进行 Azure Active Directory 身份验证 - 移动服务" metaKeywords="Azure registering application, Azure Active Directory authentication, application authenticate, authenticate mobile services" description="了解如何在移动服务应用程序中注册以进行 Azure Active Directory 身份验证." title="Register your account to use an Azure Active Directory account login" authors="wesmc" services="mobile-services" documentationCenter="Mobile" />

# 注册应用程序以使用 Azure Active Directory 帐户登录

本主题说明如何注册你的应用程序，以便能够使用 Azure Active Directory 作为 Azure 移动服务的身份验证提供程序。 


>[AZURE.NOTE] 本主题中所述的步骤应在你想要对应用程序使用[服务定向的登录操作](http://msdn.microsoft.com/library/azure/dn283952.aspx)时与[向移动服务应用程序添加身份验证](/zh-cn/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users/)教程一起使用。此外，如果你的应用程序对于 Azure Active Directory 需要[客户端定向的登录操作](http://msdn.microsoft.com/library/azure/jj710106.aspx)和 .NET 后端移动服务，应首先阅读[使用 Active Directory 身份验证库单一登录对应用程序进行身份验证](/zh-cn/documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication/)教程。


1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的移动服务。

    ![][1]

2. 单击移动服务的**"标识"**选项卡。 

    ![][2]

3. 向下滚动到**"Azure Active Directory"**标识提供程序部分，并复制其中列出的**"应用程序 URL"**。

    ![][3]

4. 导航到管理门户中的**"Active Directory"**，然后单击你的目录。

    ![][4] 

5. 单击顶部的**"应用程序"**选项卡，然后单击**"添加"**以添加应用程序。 

    ![][10]

6. 单击**"添加我的组织正在开发的应用程序"**。

7. 在"添加应用程序"向导中，为应用程序输入**"名称"**，并单击**"Web 应用程序和/或 Web API"**类型。然后单击以继续。

    ![][5]

8. 在**"登录 URL"**框中，粘贴你从移动服务 Active Directory 标识提供程序设置中复制的应用程序 ID。在**"应用程序 ID URI"**框中输入相同的唯一资源标识符。然后单击以继续。
 
    ![][6]


9. 添加应用程序后，请单击**"配置"**选项卡。然后单击相应的按钮以复制应用程序的**"客户端 ID"**。

    如果你将移动服务创建为使用 .Net 后端提供移动服务，则还需要将**"单一登录"**下的**"答复 URL"**编辑为移动服务的 URL 后接路径"signin-aad"。例如  `https://todolist.azure-mobile.cn/signin-aad`

    ![][8]


10. 返回到移动服务的**"标识"**选项卡。在底部粘贴 Azure Active Directory 标识提供程序的**"客户端 ID"**设置。

  
11. 在**"允许的租户"**列表中，需要添加已注册该应用程序的目录的域（例如 contoso.onmicrosoft.com）。可以通过单击 Active Directory 上的**"域"**选项卡找到默认域名。

    ![][11]
 
    将域名添加到**"允许的租户"**列表中，然后单击**"保存"**。    


    ![][9]



现在，你可以使用 Azure Active Directory 在应用程序中进行身份验证。 



<!-- Anchors. -->

<!-- Images. -->
[1]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-selection.png
[2]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-identity-tab.png
[3]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-copy-app-url-waad-auth.png
[4]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-select-ad-waad-auth.png
[5]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-1-waad-auth.png
[6]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-2-waad-auth.png
[7]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-3-waad-auth.png
[8]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-clientid-waad-auth.png
[9]: ./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-clientid-pasted-waad-auth.png
[10]:./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-waad-idenity-tab-selection.png
[11]:./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-default-domain.png

<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.cn/

