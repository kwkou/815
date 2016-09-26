<properties
	pageTitle="如何为应用服务应用程序配置 Azure Active Directory 身份验证"
	description="了解如何为应用服务应用程序配置 Azure Active Directory 身份验证。"
	authors="mattchenderson"
	services="app-service"
	documentationCenter=""
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.date="05/04/2016"
	wacn.date="09/26/2016"/>

# 如何将应用服务应用程序配置为使用 Azure Active Directory 登录

[AZURE.INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主题说明如何将 Azure 应用程序服务配置为使用 Azure Active Directory 作为身份验证提供程序。

## <a name="express"></a>使用快速设置配置 Azure Active Directory

13. 在 [Azure 门户预览]中，导航到应用程序。依次单击“设置”和“身份验证/授权”。

14. 如果“身份验证/授权”功能未启用，请切换为“打开”。

15. 单击“Azure Active Directory”，然后单击“管理模式”下面的“快速”。

16. 单击“确定”，在 Azure Active Directory 中注册应用程序。随后将会创建新注册。如果想要选择现有注册，请单击“选择现有应用”，然后在租户中搜索以前创建的注册的名称。单击注册将它选中，然后单击“确定”。在 Azure Active Directory 设置边栏选项卡中单击“确定”。

    ![][0]

	默认情况下，应用服务提供身份验证但不限制对站点内容和 API 的已授权访问。必须在应用代码中为用户授权。

17. （可选）若要限制只有通过 Azure Active Directory 身份验证的用户可以访问站点，请将“请求未经身份验证时需执行的操作”设置为“使用 Azure Active Directory 登录”。这会要求对所有请求进行身份验证，所有未经身份验证的请求将重定向到 Azure Active Directory 以进行身份验证。

17. 单击“保存”。

现在，可以使用 Azure Active Directory 在应用中进行身份验证。

## <a name="advanced"></a>（备选方法）使用高级设置手动配置 Azure Active Directory
也可以选择手动提供配置设置。如果要使用的 AAD 租户不同于登录 Azure 所用的租户，这是较好的解决方案。若要完成配置，必须先在 Azure Active Directory 中创建注册，然后向应用服务提供一些注册详细信息。

### <a name="register"></a>将应用程序注册到 Azure Active Directory

1. 登录到 [Azure 门户预览]并导航到应用程序。复制 **URL**。稍后要使用此信息配置 Azure Active Directory 应用。

3. 登录到 [Azure 经典管理门户]并导航到“Active Directory”。

    ![][2]

4. 选择目录，然后选择顶部的“应用程序”选项卡。单击底部的“添加”，创建新的应用注册。

5. 单击“添加我的组织正在开发的应用程序”。

6. 在“添加应用程序向导”中，为应用程序输入“名称”，并单击“Web 应用程序和/或 Web API”类型。然后单击以继续。

7. 在“登录 URL”框中，粘贴前面复制的应用程序 URL。在“应用 ID URI”框中输入同一个 URL。然后单击以继续。

8. 添加应用程序后，请单击“配置”选项卡。将“单一登录”下面的“回复 URL”编辑为应用程序的 URL，并追加 _/.auth/login/aad/callback_ 路径。例如，`https://contoso.chinacloudsites.cn/.auth/login/aad/callback`。请务必使用 HTTPS 方案。

    ![][3]

9. 单击“保存”。然后，复制应用的**客户端 ID**。稍后配置应用程序时要用到此信息。

10. 在底部命令行中，单击“查看终结点”，然后复制“联合元数据文档”URL 并下载该文档，或者在浏览器中导航到该文档。

11. 在根 **EntityDescriptor** 元素中，应该有一个 `https://sts.chinacloudapi.cn/` 格式的 **entityID** 属性，其后接租户特定的 GUID（称为“租户 ID”）。复制此值 - 它将用作**颁发者 URL**。稍后配置应用程序时要用到此信息。

### <a name="secrets"></a>将 Azure Active Directory 信息添加到应用程序

13. 返回 [Azure 门户预览]，导航到应用程序。依次单击“设置”和“身份验证/授权”。

14. 如果“身份验证/授权”功能未启用，请切换为“打开”。

15. 单击“Azure Active Directory”，然后单击“管理模式”下面的“高级”。粘贴前面获取的客户端 ID 和颁发者 URL 值。然后，单击“确定”。

    ![][1]

	默认情况下，应用服务提供身份验证但不限制对站点内容和 API 的已授权访问。必须在应用代码中为用户授权。

17. （可选）若要限制只有通过 Azure Active Directory 身份验证的用户可以访问站点，请将“请求未经身份验证时需执行的操作”设置为“使用 Azure Active Directory 登录”。这会要求对所有请求进行身份验证，所有未经身份验证的请求将重定向到 Azure Active Directory 以进行身份验证。

17. 单击“保存”。

现在，可以使用 Azure Active Directory 在应用中进行身份验证。

## （可选）配置本机客户端应用程序

Azure Active Directory 还允许注册权限映射控制度更高的本机客户端。如果想要使用 **Active Directory 身份验证库**等库执行登录，则需要这种注册。

1. 在 [Azure 经典管理门户]中导航到“Active Directory”。

2. 选择目录，然后选择顶部的“应用程序”选项卡。单击底部的“添加”，创建新的应用注册。

3. 单击“添加我的组织正在开发的应用程序”。

4. 在“添加应用程序”向导中，为应用程序输入**名称**，并单击“本机客户端应用程序”类型。然后单击以继续。

5. 在“重定向 URI”框中，使用 HTTPS 方案输入站点的 _/.auth/login/done_ 终结点。此值应类似于 \_https://contoso.chinacloudsites.cn/.auth/login/done_。如果要创建 Windows 应用程序，请改用[包 SID](/documentation/articles/app-service-mobile-dotnet-how-to-use-client-library/#package-sid) 作为 URI。

6. 添加本机应用程序后，单击“配置”选项卡。找到**客户端 ID**并记下此值。

7. 向下滚动到“对其他应用程序的权限”部分，然后单击“添加应用程序”。

8. 搜索以前注册的 Web 应用程序并单击加号图标。然后，单击勾选图标关闭对话框。如果找不到 Web 应用程序，请导航到其注册并添加新的回复 URL（例如，当前 URL 的 HTTP 版本），单击“保存”，然后重复这些步骤 - 应用程序应会显示在列表中。

9. 针对刚刚添加的新项，打开“委托的权限”下拉列表，然后选择“访问(appName)”。然后单击“保存”。

现已配置可以访问应用服务应用程序的本机客户端应用程序。

## <a name="related-content"></a>相关内容

[AZURE.INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/mobile-app-aad-express-settings.png
[1]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/mobile-app-aad-advanced-settings.png
[2]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-navigate-aad.png
[3]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-app-configure.png

<!-- URLs. -->

[Azure 门户预览]: https://portal.azure.cn/
[Azure 经典管理门户]: https://manage.windowsazure.cn/
[alternative method]: #advanced

<!---HONumber=Mooncake_0919_2016-->