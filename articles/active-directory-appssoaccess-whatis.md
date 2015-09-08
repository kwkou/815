<properties
	pageTitle="什么是使用 Azure Active Directory 进行应用程序访问和单一登录？| Windows Azure"
	description="使用 Azure Active Directory 以对开展业务所需的全部 SaaS 和 Web 应用程序启用单一登录。"
	services="active-directory"
	documentationCenter=""
	authors="asmalser-msft"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/01/2015"
	wacn.date="08/29/2015"/>

#什么是使用 Azure Active Directory 进行应用程序访问和单一登录？

单一登录是指只需使用一个用户帐户登录一次即可访问开展业务所需的全部应用程序和资源。一旦登录，即可访问所需的全部应用程序，而无需再次进行身份验证（例如，输入密码）。

许多组织都依靠软件即服务 (SaaS) 应用程序（如 Office 365、Box 和 Salesforce）来提高最终用户的工作效率。过去，IT 员工需要单独在每个 SaaS 应用程序中创建和更新用户帐户，而用户必须记住每个 SaaS 应用程序的密码。

Azure Active Directory 将本地 Active Directory 扩展到云端，让用户不仅能够使用他们的主组织帐户登录其已加入域的设备和公司资源，也能登录到开展其工作所需的全部 Web 和 SaaS 应用程序。

因此，不仅用户无需管理多组用户名和密码，也可基于用户的组织组成员以及其员工状态自动设置或取消设置其应用程序访问。Azure Active Directory 引入了安全和访问管理控件，通过这些控件，你可以集中管理用户对所有 SaaS 应用程序的访问。

Azure AD 可以轻松集成当今的许多热门 SaaS 应用程序，它提供身份认证和访问管理，使用户可以直接单一登录到应用程序，或从一个门户（如 Office 365 或 Azure AD 访问面板）发现并启动这些应用程序。

集成的体系结构由以下四个主要构建基块组成：

* 单一登录使用户可以基于他们在 Azure AD 中的组织帐户访问其 SaaS 应用程序。单一登录使用户可以使用他们的单一组织帐户向应用程序进行身份验证。

* 用户设置可以基于 Windows Server Active Directory 和/或 Azure AD 中所做的更改实现到目标 SaaS 的用户设置和取消设置。已设置的帐户使用户可以在通过单一登录进行身份验证后获得使用应用程序的授权。

* Azure 管理门户中的集中应用程序访问管理支持对 SaaS 应用程序进行单点访问和管理，并能够将应用程序访问决策和批准委托给组织的任何成员

* Azure AD 中用户活动的统一报告和监视

##如何使用 Azure Active Directory 进行单一登录？

当用户“登录”一个应用程序，他们将通过身份验证过程，该过程要求用户证明其身份。如不采用单一登录方式，通常将通过输入应用程序中存储的密码来完成身份验证，这要求用户知道该密码。

Azure AD 支持以三种不同的方式登录应用程序：

*	**联合单一登录**，此登录方式使应用程序重定向到 Azure AD 以进行用户身份验证，而不是提示用户输入该应用程序的密码。凡支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 等协议的应用程序都支持此登录方式，这也是单一登录最丰富的模式。

*	**基于密码的单一登录**，此登录方式使用 Web 浏览器扩展或移动应用来实现安全的应用程序密码存储和重用。此登录方式利用的是应用程序提供的现有登录过程，但管理员可以管理密码且不要求用户知道密码。

*	**现有单一登录**，此登录方式使 Azure AD 可以利用已为应用程序设置的任何现有单一登录，但这些应用程序可以链接到 Office 365 或 Azure AD 访问面板门户，并且在 Azure AD 中启动应用程序还可以提供额外的报告。

一旦用户向应用程序验证完身份，他们还需要在应用程序中设置一条帐户记录，以区分内部存在访问权限和级别的应用程序。此帐户记录的设置可自动完成，也可在为用户提供单一登录访问权限之前由管理员手动完成。

 以下是关于上述单一登录模式和设置的更多详细信息。

###联合单一登录

联合单一登录可使组织中的用户使用 Azure AD 中的用户帐户信息自动登录到 Azure AD 中的第三方 SaaS 应用程序。

在这种情况下，如果你已经登录到 Azure AD 中，并且希望访问由第三方 SaaS 应用程序控制的资源，那么采用联合单一登录方式可使用户免于再次进行身份验证。

Azure AD 可在支持 SAML 2.0、WS 联合身份验证或 OpenID connect 协议的应用程序上支持联合单一登录。

另请参阅：[管理联合单一登录的证书](/documentation/articles/active-directory-sso-certs)

###基于密码的单一登录

配置基于密码的单一登录可使组织中的用户使用第三方 SaaS 应用程序中的用户帐户信息自动登录到 Azure AD 中的第三方 SaaS 应用程序。启用此功能时，Azure AD 将收集并安全地存储用户帐户信息及相关密码。

Azure AD 可在基于云且具有 HTML 登录页的应用程序上支持基于密码的单一登录。使用自定义浏览器插件时，AAD 通过安全地从目录中检索应用程序凭据（例如用户名和密码），然后以用户的名义将这些凭据输入应用程序登录页，自动完成用户的登录过程。以下是两个用例：

1.	**管理员管理凭据** – 管理员可以创建和管理应用程序凭据，并将这些凭据分配给需要访问应用程序的用户或组。在这种情况下，最终用户不需要知道凭据，但仍需要通过在其访问面板中进行单击或通过单击提供的链接获取对应用程序的单一登录访问权限。这种方式既允许管理员进行凭据的生命周期管理，也可以方便最终用户，使他们不必去记住或管理应用程序特定的密码。在自动登录过程中，对于最终用户来说，凭据是模糊的；但从技术上来说，用户可以使用 Web 调试工具发现这些凭据，并且用户和管理员都应遵守与直接向用户展示凭据相同的安全策略。在提供许多用户共享的帐户访问权限时，管理员提供的凭据非常有用，如社交媒体或文档分享应用程序。

2.	**用户管理凭据** – 管理员可将应用程序分配给最终用户或组，并允许最终用户首次在其访问面板中访问应用程序时直接输入他们自己的凭据。这种方式可为最终用户创造便利，他们将无需在每次访问应用程序时频繁地输入应用程序特定的密码。此用例也可用作管理员管理凭据的敲门砖，管理员可以在将来某个日期为应用程序设置新的凭据，而这并不会对最终用户的应用程序访问体验产生影响。

在这两种用例中，凭据都以加密状态保存在目录中，并且仅在自动登录过程中通过 HTTPS 进行传递。通过使用基于密码的单一登录，Azure AD 为不支持联合协议的应用程序提供方便的身份认证和访问管理解决方案。

基于密码的 SSO 依靠浏览器扩展安全地从 Azure AD 检索应用程序和用户特定信息，并将这些信息应用到服务中。Azure AD 支持的大多数第三方 SaaS 应用程序都支持此功能。

对于基于密码的 SSO，最终用户浏览器可以是：*IE 8、IE9 和 IE10（在 Windows 7 或更高版本中）；*Chrome（在 Windows 7 或更高版本中，或者在 MacOS X 或更高版本中）

###退出单一登录

为应用程序配置单一登录时，Azure 管理门户提供第三个选项“对出单一登录”。此选项只允许管理员创建一个指向应用程序的链接，并将该链接放在选定用户的访问面板上。

例如，如果有一个应用程序配置为使用 Active Directory 联合身份验证服务 2.0 对用户进行身份验证，则管理员可以使用“退出单一登录”选项在访问面板上创建一个指向该应用程序的链接。当用户访问该链接时，将使用 Active Directory 联合身份验证服务 2.0 或该应用程序提供的任何现有单一登录解决方案对他们进行身份验证。

###用户设置

对于选定的应用程序，Azure AD 可使用 Windows Server Active Directory 或 Azure AD 身份信息在 Azure 管理门户内对第三方 SaaS 应用程序中的帐户实现自动用户设置和取消设置。在 Azure AD 中，如果用户获得了其中一个应用程序的权限，则可以在目标 SaaS 应用程序中自动创建（设置）帐户。

在 Azure AD 中删除用户或者用户的信息发生更改时，这些更改也会反映在该 SaaS 应用程序中。这就意味着，配置自动身份生命周期管理使管理员可以控制并从 SaaS 应用程序提供自动设置和取消设置。在 Azure AD 中，此身份生命周期管理的自动化由用户设置实现。

若要了解详细信息，请参阅 [SaaS 应用程序的自动用户设置和取消设置](/documentation/articles/active-directory-saas-app-provisioning)

##Azure AD 应用程序库入门

准备好开始了吗？ 若要在你的组织使用的 Azure AD 与 SaaS 应用程序之间部署单一登录，请遵循以下指导。

###使用 Azure AD 应用程序库

[Azure Active Directory 应用程序库](http://azure.microsoft.com/zh-cn/marketplace/active-directory/all/)提供已知的支持使用 Azure Active Directory 进行单一登录的应用程序列表。

![][1]

以下是按应用程序支持的功能查找应用程序的一些小技巧：

*	Azure AD 支持 [Azure Active Directory 应用程序库](http://azure.microsoft.com/zh-cn/marketplace/active-directory/all/)中所有“精选”应用程序的自动设置和取消设置。

*	有关专门支持使用 SAML、WS 联合身份验证或 OpenID Connect 协议的联合单一登录的联合应用程序列表，可在[此处](http://social.technet.microsoft.com/wiki/contents/articles/20235.azure-active-directory-application-gallery-federated-saas-apps.aspx)找到。

一旦找到你的应用程序，即可遵循应用程序库中的分步说明开始操作，并在 Azure 管理门户中实现单一登录。

###库中没有应用程序？

如果在 Azure AD 应用程序库中没有找到你的应用程序，可以选择执行以下操作：

*	**添加你当前使用的未列出的应用程序** – 在 Azure 管理门户中使用应用程序库中的“自定义”类别连接你的组织当前使用的未列出的应用程序。你可以将支持 SAML 2.0 的任何应用程序添加为联合应用程序，也可以将具有 HTML 登录页的任何应用程序添加为密码 SSO 应用程序。有关更多详细信息，请参阅关于[添加你自己的应用程序](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)一文。


*	**添加你当前开发的自有应用程序** – 如果你自己开发了应用程序，请遵循 Azure AD 开发人员文档中的指导实现联合单一登录或使用 Azure AD 图形 API 进行设置。有关更多详细信息，请参阅以下资源：
  * https://msdn.microsoft.com/zh-cn/library/azure/dn499820.aspx
  * https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet
  * https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet
  * https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore

*	**请求应用程序集成** – 使用 [Azure AD 反馈论坛](http://feedback.azure.com/forums/169401-azure-active-directory)请求对你所需的应用程序的支持。

###使用 Azure 管理门户

你可以使用 Azure 管理门户中的 Active Directory 扩展来配置应用程序单一登录。第一步，你需要从门户中的 Active Directory 部分选择一个目录：

![][2]

若要管理你的第三方 SaaS 应用程序，你可以切换到选定目录的“应用程序”选项卡。此视图允许管理员进行以下操作：

* 从 Azure AD 库添加新应用程序，以及你正在开发的应用程序
* 删除集成的应用程序
* 管理已集成的应用程序

第三方 SaaS 应用程序的典型管理任务如下：

* 使用 Azure AD、密码 SSO 或联合 SSO（如果适用于目标 SaaS）实现单一登录
* （可选）启用用户设置实现用户设置和取消设置（身份生命周期管理）
* 对于启用用户设置的应用程序，选择对该应用程序具有访问权限的用户

对于支持联合单一登录的库应用程序，配置通常要求你提供额外的配置设置（如证书和元数据）以在第三方应用程序与 Azure AD 之间创建联合信任。配置向导将向你展示详细信息并让你轻松访问 SaaS 应用程序特定的数据和说明。

对于支持自动用户设置的库应用程序，要求你向 Azure AD 授予权限以管理你在 SaaS 应用程序中的帐户。向目标应用程序进行身份验证时，你至少需要提供 Azure AD 应使用的凭据。是否需要提供额外的配置设置视应用程序的要求而定。

##向用户部署集成 Azure AD 的应用程序

Azure AD 提供若干种自定义方式来向你组织中的最终用户部署应用程序：

* Azure AD 访问面板
* Office 365 应用程序启动器
* 直接单一登录到联合应用程序
* 到联合应用程序、基于密码的应用程序或现有应用程序的深层链接

选择要在组织中部署的方式由你自己决定。

###Azure AD 访问面板

https://myapps.microsoft.com 的访问面板是基于 Web 的门户，该门户允许在 Azure Active Directory 中具有组织帐户的最终用户查看并启动基于云的应用程序，Azure AD 管理员已授予访问这些应用程序的权限。如果你是使用 [Azure Active Directory Premium](https://msdn.microsoft.com/zh-cn/library/azure/dn532272.aspx) 的最终用户，你也可以通过访问面板使用自助服务组管理功能。

![][3]

访问面板与 Azure 管理门户相互分离，且不要求用户具有 Azure 订阅或 Office 365 订阅。

有关 Azure AD 访问面板的详细信息，请参阅[访问面板介绍](https://msdn.microsoft.com/zh-cn/library/azure/dn308586.aspx)。

###Office 365 应用程序启动器

对于已部署 Office 365 的组织，通过 Azure AD 分配给用户的应用程序也将出现在 Office 365 门户 (https://portal.office.com/myapps) 中。这样可使组织中的用户更加轻松方便地启动应用程序，而无需使用第二个门户，推荐使用 Office 365 的组织采用这种应用程序启动解决方案。

![][4]

有关 Office 365 应用程序启动器的详细信息，请参阅[让你的应用程序出现在 Office 365 应用程序启动器中](https://msdn.microsoft.com/office/office365/howto/connect-your-app-to-o365-app-launcher)。

###直接单一登录到联合应用程序

大多数支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 的联合应用程序也支持用户在应用程序中开始操作的功能，然后通过 Azure AD 登录到此类应用程序，登录方式采用自动重定向或单击登录链接。这种方式被称为服务提供程序启动登录，Azure AD 应用程序库中的大多数联合应用程序都支持此方式（请参阅 Azure 管理门户中应用程序的单一登录配置向导所链接的文档以获取详细信息）。

![][5]

###联合应用程序、基于密码的应用程序或现有应用程序的直接登录链接

Azure AD 还支持指向单个应用程序的直接单一登录链接，这些应用程序支持基于密码的单一登录、现有单一登录以及任何形式的联合单一登录。

这些链接是专门精心设计的 URL，它们通过 Azure AD 登录过程为特定应用程序发送用户，而不要求用户从 Azure AD 访问面板或 Office 365 启动。这些单一登录 URL 可以在 Azure 管理门户 Active Directory 部分中任何预集成应用程序的“仪表板”选项卡下找到，如下面的截图所示。

![][6]

你可以复制这些链接并将其粘贴到你想要提供选定应用程序登录链接的任何地方。你可以在电子邮件或基于 Web 的任何自定义门户中设置用户的应用程序访问权限。以下是用于 Twitter 的 Azure AD 直接单一登录 URL 的示例：

https://myapps.microsoft.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced

与访问面板的组织特定 URL 类似，你可以通过在 myapps.microsoft.com 域后添加你目录的其中一个活动的或已验证的域进一步自定义此 URL。这样可确保在登录页上立即加载任何组织品牌，而无需用户先输入其用户 ID：

https://myapps.microsoft.com/contosobuild.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced

当授权用户单击其中一个应用程序特定链接时，首先他们将看到其组织登录页（假设他们尚未登录），登录后他们将被重定向到其应用程序，而无需先停止访问面板。如果用户缺少访问应用程序所需的先决条件（例如基于密码的单一登录浏览器扩展），那么单击该链接时将提示用户安装缺失的扩展。如果应用程序的单一登录配置更改，该链接 URL 仍保持不变。

这些链接使用与访问面板和 Office 365 相同的访问控制机制，只有在 Azure 管理门户中分配到该应用程序的用户或组才能够成功通过身份验证。但是，未获得授权的所有用户都将看到一条消息，该消息说明他们未获得访问权限并提供一个链接，单击该链接将加载访问面板以查看他们有权访问的可用应用程序。

<!--Image references-->
[1]: ./media/active-directory-appssoaccess-whatis/onlineappgallery.png
[2]: ./media/active-directory-appssoaccess-whatis/azuremgmtportal.png
[3]: ./media/active-directory-appssoaccess-whatis/accesspanel.png
[4]: ./media/active-directory-appssoaccess-whatis/officeapphub.png
[5]: ./media/active-directory-appssoaccess-whatis/workdaymobile.png
[6]: ./media/active-directory-appssoaccess-whatis/deeplink.png

<!---HONumber=67-->