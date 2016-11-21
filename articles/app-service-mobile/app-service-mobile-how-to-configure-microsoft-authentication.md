<properties
	pageTitle="如何为应用服务应用程序配置 Microsoft 帐户身份验证"
	description="了解如何为应用服务应用程序配置 Microsoft 帐户身份验证。"
	authors="mattchenderson"
	services="app-service"
	documentationCenter=""
	manager="erikre"
	editor=""/>  

<tags
	ms.service="app-service"
	ms.workload="mobile"
	ms.tgt_pltfrm="na"
	ms.devlang="multiple"
	ms.topic="article"
	ms.date="10/01/2016"
	wacn.date="11/21/2016"
	ms.author="mahender"/>

# 如何将应用服务应用程序配置为使用 Microsoft 帐户登录

[AZURE.INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主题说明如何将 Azure 应用服务配置为使用 Microsoft 帐户作为身份验证提供程序。

## <a name="register-microsoft-account"></a>将应用注册到 Microsoft 帐户

1. 登录到 [Azure 门户预览]并导航到应用程序。复制 **URL**，随后会用于使用 Microsoft 帐户来配置应用。

2. 在 Microsoft 帐户开发人员中心内导航到“[我的应用程序]”页，然后根据需要使用 Microsoft 帐户登录。

3. 单击“添加应用”，键入应用程序名称，然后单击“创建应用程序”。

4. 记下“应用程序 ID”，因为稍后将要用到。

5. 在“平台”下，单击“添加平台”，然后选择“Web”。

6. 在“重定向 URI”下，提供应用程序的终结点，然后单击“保存”。
 
	>[AZURE.NOTE] 重定向 URI 是应用程序 URL 加上路径 _/.auth/login/microsoftaccount/callback_。例如，`https://contoso.chinacloudsites.cn/.auth/login/microsoftaccount/callback`。
	>请务必使用 HTTPS 方案。

7. 在“应用程序机密”下，单击“生成新密码”。请记下显示的值。关闭页面后，就不再显示该值。


    > [AZURE.IMPORTANT] 密码是一个非常重要的安全凭据。请不要与任何人共享密码或者在客户端应用程序中分发它。

## <a name="secrets"></a>将 Microsoft 帐户信息添加到应用服务应用程序

1. 返回 [Azure 门户预览]，导航到应用程序，然后单击“设置”>“身份验证/授权”。

2. 如果“身份验证/授权”功能未启用，请将它切换为“打开”。

3. 单击“Microsoft 帐户”。粘贴前面获取的应用程序 ID 和密码值，启用应用程序所需的任何范围（可选）。然后，单击“确定”。

    ![][1]

	默认情况下，应用服务提供身份验证但不限制对站点内容和 API 的已授权访问。必须在应用代码中为用户授权。

4. （可选）若要限制只有通过 Microsoft 帐户身份验证的用户可以访问站点，请将“请求未经身份验证时需执行的操作”设置为“Microsoft 帐户”。这会要求对所有请求进行身份验证，所有未经身份验证的请求将重定向到 Microsoft 帐户进行身份验证。

5. 单击“保存”。

现在，可以使用 Microsoft 帐户在应用中进行身份验证。

## <a name="related-content"></a>相关内容

[AZURE.INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]


<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/app-service-microsoftaccount-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/mobile-app-microsoftaccount-settings.png

<!-- URLs. -->

[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure 门户预览]: https://portal.azure.cn/

<!---HONumber=Mooncake_0919_2016-->