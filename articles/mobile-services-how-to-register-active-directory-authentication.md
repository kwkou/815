<properties 
	pageTitle="注册以进行 Azure Active Directory 身份验证 | Microsoft Azure" 
	description="了解如何在移动服务应用程序中注册以进行 Azure Active Directory 身份验证。" 
	authors="wesmc7777" 
	services="mobile-services" 
	documentationCenter="" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="02/05/2016"
	wacn.date="03/28/2016"/>

# 注册应用程序以使用 Azure Active Directory 帐户登录

[AZURE.INCLUDE [mobile-services-selector-register-identity-provider](../includes/mobile-services-selector-register-identity-provider.md)]

##概述

本主题说明如何注册你的应用程序，以便能够使用 Azure Active Directory 作为移动服务的身份验证提供程序。

##注册你的应用程序

>[AZURE.NOTE]本主题中所述的步骤应在你想要对应用程序使用[服务定向的登录操作](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started-users/)时与[向移动服务应用程序添加身份验证](http://msdn.microsoft.com/zh-cn/library/azure/dn283952.aspx)教程一起使用。此外，如果你的应用程序对于 Azure Active Directory 需要[客户端定向的登录操作](http://msdn.microsoft.com/zh-cn/library/azure/jj710106.aspx)和 .NET 后端移动服务，应首先阅读[使用 Active Directory 身份验证库单一登录对应用程序进行身份验证](/documentation/articles/mobile-services-windows-store-dotnet-adal-sso-authentication/)教程。

1. 登录到 [Azure 管理门户]，导航到你的移动服务，单击“身份”选项卡，然后向下滚动到“Azure Active Directory”标识提供者部分，并复制显示的“应用 URL”。

    ![AAD 的移动服务应用 URL](./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-copy-app-url-waad-auth.png)

2. 在[Azure 管理门户]中，导航到“Active Directory”，依次单击你的目录和“域”，然后记下目录的默认域。

3. 单击“应用程序”>“添加”。

4. 在“添加应用程序向导”中，为应用程序输入“名称”，并单击“Web 应用程序和/或 Web API”类型。

    ![为 AAD 应用命名](./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-1-waad-auth.png)

5. 在“登录 URL”框中，粘贴你从移动服务中复制的应用 ID 值。在“应用程序 ID URI”框中输入相同的唯一值，然后单击以继续。
 
    ![设置 AAD 应用属性](./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-add-app-wizard-2-waad-auth.png)

6. 添加应用程序后，单击“配置”选项卡并复制应用的“客户端 ID”。

    >[AZURE.NOTE]对于 .Net 后端移动服务，还必须将“单一登录”下的“回复 URL”值编辑为移动服务的 URL 后接路径“signin-aad”。例如 `https://todolist.azure-mobile.cn/signin-aad`

7. 返回到移动服务的“身份”选项卡，然后粘贴复制的 Azure Active Directory 标识提供者“客户端 ID”值。
 
    ![](./media/mobile-services-how-to-register-active-directory-authentication/mobile-services-clientid-pasted-waad-auth.png)

8.  在“允许的租户数”列表中，键入已注册该应用程序的目录的域（例如 `contoso.onmicrosoft.com`），然后单击“保存”。


现在，你可以使用 Azure Active Directory 在应用程序中进行身份验证。



<!-- Anchors. -->

<!-- Images. -->


<!-- URLs. -->
[Azure 管理门户]: https://manage.windowsazure.cn/


<!---HONumber=Mooncake_0118_2016-->