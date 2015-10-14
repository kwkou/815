<properties 
	pageTitle="注册以进行 Microsoft 身份验证 - 移动服务" 
	description="了解如何在 Azure 移动服务应用程序中注册以进行 Microsoft 身份验证。" 
	authors="ggailey777" 
	services="mobile-services" 
	documentationCenter="Mobile" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.date="08/08/2015" 
	wacn.date="10/03/2015"/>

# 注册应用程序以使用 Microsoft 帐户进行身份验证

[AZURE.INCLUDE [mobile-services-selector-register-identity-provider](../includes/mobile-services-selector-register-identity-provider.md)]

## 概述 

本主题说明如何注册你的移动应用程序，以便能够使用 Microsoft 帐户作为 Azure 移动服务的标识提供程序。以下步骤同样适用于使用 Live SDK 的服务导向型身份验证和客户端导向型身份验证。

##在 Windows 开发人员中心注册 Windows 应用商店应用程序

首先必须在 Windows 开发人员中心注册 Windows 应用商店应用程序。

>[AZURE.NOTE]Windows Phone 8、Windows Phone 8.1 Silverlight 和非 Windows 应用程序可以跳过本部分。

1. 如果尚未注册应用程序，请在开发人员中心内导航到 Windows 应用商店应用的“提交应用程序”页，用 Microsoft 帐户登录，然后单击“应用程序名称”[]。

   	![](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-submit-win8-app.png)

2. 选择“通过保留唯一名称来创建新应用程序”，单击“继续”，在“应用程序名称”中输入应用程序的名称，单击“保留应用程序名称”，然后单击“保存”。

   	![](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-name.png)

   	此操作为应用创建一个新的 Windows 应用商店注册。

3. 在 Visual Studio 中，打开你在完成[移动服务入门](/documentation/articles/mobile-services-dotnet-backend-windows-store-dotnet-get-started)教程后创建的项目。

4. 在“解决方案资源管理器”中，右键单击 Windows 应用商店应用程序项目，单击“应用商店”，然后单击“将应用与应用商店关联...”。

  	![](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-store-association.png)

   	此时将显示“将应用与 Windows 应用商店关联”向导。

5. 在向导中，单击“登录”，以你的 Microsoft 帐户登录，选择你在步骤 2 中保留的应用程序名称，然后单击“下一步”>“关联”。

   	这会将所需的 Windows 应用商店注册信息添加到应用程序清单中。

6. （可选）对于 Windows 通用应用程序，请对 Windows Phone 应用商店项目重复执行步骤 4 和 5。

7. 回到新应用的“Windows 开发人员中心”页，单击“服务”>“推送通知”。

8. 在“推送通知”页中，在“Windows 推送通知服务(WNS)和 Microsoft Azure 移动服务”下面单击“Live Services 站点”。


此时将显示应用程序的 Microsoft 帐户页。

## 配置你的 Microsoft 帐户注册，然后连接到移动服务

本部分中的第一个步骤只适用于 Windows Phone 8、Windows Phone 8.1 Silverlight 以及非 Windows 应用商店应用程序。对于这些应用程序，你也可以忽略包安全性标识符 (SID)，因为该标识符只适用于 Windows 应用商店应用程序。

1. 对于非 Windows 应用商店应用程序，请浏览到 Microsoft 帐户开发人员中心的<a href="http://go.microsoft.com/fwlink/p/?LinkId=262039" target="_blank">我的应用程序</a>页，以你的 Microsoft 帐户登录（如果需要），单击“创建应用程序”，键入应用程序名称，然后单击“我接受”。

   	这将会通过 Microsoft 帐户为你保留应用程序名称，并显示应用程序的 Microsoft 帐户页。

2. 在应用程序的 Microsoft 帐户页上，单击“API 设置”，选择启用“移动或桌面客户端应用程序”，将移动服务 URL 设为“目标域”，在“重定向 URL”中提供 `https://<mobile_service>.azure-mobile.net/login/microsoftaccount/` 的值，然后单击“保存”。

	 >[AZURE.NOTE]对于使用 Visual Studio 发布到 Azure 的 .NET 后端移动服务，重定向 URL 是移动服务的 URL 后接用作 .NET 服务的移动服务的路径 _signin-microsoft_，例如 `https://todolist.azure-mobile.net/signin-microsoft`。

    ![Microsoft 帐户 API 设置](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth-2.png)

	“根域”应该会自动填入。

3. 单击“应用程序设置”，并记下“客户端 ID”、“客户端机密”和“包 SID”的值。

   	![Microsoft 帐户应用程序设置](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-win8-app-push-auth.png)

    > [AZURE.NOTE]客户端密钥是一个非常重要的安全凭据。请勿与任何人分享客户端密钥或将密钥随应用程序分发。只有 Windows 应用商店应用程序注册才能看到“包 SID”字段。

4. 在 [Azure 管理门户]中，单击移动服务的“标识”选项卡，输入从标识提供程序获取的客户端 ID、客户端机密和包 SID，然后单击“保存”。

 	![移动服务“标识”选项卡](./media/mobile-services-how-to-register-microsoft-authentication/mobile-services-identity-tab.png)
	
	>[AZURE.NOTE]对于 Windows Phone 8、Windows Phone 应用商店 8.1 Silverlight 或非 Windows 应用程序，不需提供包 SID 值。
	
现在，你的移动服务和应用程序都已配置为使用 Microsoft 帐户。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039

[Azure 管理门户]: https://manage.windowsazure.cn/

<!---HONumber=71-->