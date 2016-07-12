<properties
	pageTitle="Azure Active Directory 的应用程序访问与单一登录是什么？| Azure"
	description="使用 Azure Active Directory 启用单一登录，以访问完成业务所需的全部 SaaS 和 Web 应用程序。"
	services="active-directory"
	documentationCenter=""
	authors="asmalser-msft"
	manager="terrylan"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/09/2016"
	wacn.date="06/24/2016"/>

#Azure Active Directory 的应用程序访问与单一登录是什么？

单一登录是指只需使用单个用户帐户登录一次，就能访问展开业务所需的全部应用程序和资源。登录之后，你可以访问全部所需的应用程序，而无需再次进行身份验证（例如键入密码）。

许多组织依赖软件即服务 (SaaS) 应用程序（如 Office 365、Box 和 Salesforce）来提高最终用户生产力。从历史上看，IT 人员需要在每个 SaaS 应用程序中单独创建和更新用户帐户，而用户需要记住每个 SaaS 应用程序的密码。

Azure Active Directory 将本地 Active Directory 扩展到云，让用户不仅能够使用主要组织帐户登录到已加入域的设备和公司资源，而且还能登录到完成作业所需的全部 Web 和 SaaS 应用程序。

因此，优势是不仅用户无需管理多组用户名和密码，而且还可根据其组织的组成员以及其身为员工的状态，自动预配或取消预配其应用程序的访问权限。Azure Active Directory 引入了安全和访问管理控制，使你能够跨 SaaS 应用程序集中管理用户的访问权限。

Azure AD 能轻松集成到许多现今热门的 SaaS 应用程序。它提供标识与访问管理，并可让用户直接单一登录到应用程序，或者从门户（例如 Office 365 或 Azure AD 访问面板）发现和启动应用程序。

集成的体系结构由以下四个主要构建基块组成：

* 单一登录使用户能够基于他们在 Azure AD 中的组织帐户访问他们的 SaaS 应用程序。单一登录使用户能够使用其单个组织帐户通过应用程序的身份验证

* 用户预配使用户能够基于 Windows Server Active Directory 和/或 Azure AD 中所做的更改，对目标 SaaS 进行预配和取消预配操作。预配的帐户通过单一登录身份验证之后，可授权用户使用应用程序。

* Azure 经典管理门户中的集中式应用程序访问管理实现了 SaaS 应用程序的单点访问和管理，并且可以将应用程序访问决策和审批委派给组织中的任何人。

* 在 Azure AD 中对异常用户活动进行统一报告和监视

##Azure Active Directory 中单一登录的工作原理是什么？

当用户“登录”应用程序时，需要经历一个身份验证过程，以证明他们的身份。如果不使用单一登录，这通常是通过输入存储在应用程序中的密码进行，而用户必须知道此密码。

Azure AD 支持通过三种不同的方式登录应用程序：

*	**联合单一登录**可让应用程序重定向到 Azure AD 进行用户身份验证，而不是提示用户输入自己的密码。支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 等协议的应用程序都支持此方式，而且这是最丰富的单一登录模式。

*	**基于密码的单一登录**允许使用 Web 浏览器扩展或移动应用安全存储应用程序的密码和重放。此方式使用应用程序提供的现有登录过程，但允许管理员管理密码，并且用户无需记住密码。

*	**现有单一登录**可让 Azure AD 利用针对应用程序设置的任何现有单一登录，但可让这些应用程序链接到 Office 365 或 Azure AD 访问面板门户，当其中有应用程序启动时，Azure AD 中还会生成额外的报告。

在用户通过应用程序的身份验证后，他们还需要在应用程序中预配的帐户记录，以告知应用程序他们对应用程序的哪些部分具有权限和访问级别。此帐户记录的预配可以自动完成，也可以在为用户提供单一登录访问之前由管理员手动完成。

 以下是有关这些单一登录模式和预配的详细信息。

###联合单一登录

联合单一登录可使组织中的用户能够使用 Azure AD 中的用户帐户信息自动登录到第三方 SaaS 应用程序。

在此方案中，当用户已登录到 Azure AD 中，并且想要访问由第三方 SaaS 应用程序控制的资源时，使用联合的用户无需重新进行身份验证。

Azure AD 允许对支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 协议的应用程序使用联合单一登录。

另请参阅：[管理用于联合单一登录的证书](/documentation/articles/active-directory-sso-certs/)

###基于密码的单一登录

配置基于密码的单一登录可使组织中的用户能够使用第三方 SaaS 应用程序中的用户帐户信息通过 Azure AD 自动登录到第三方 SaaS 应用程序。当你启用此功能时，Azure AD 将收集并安全地存储用户帐户信息和相关密码。

对于提供了基于 HTML 的登录页的任何基于云的应用程序，Azure AD 都可以支持基于密码的单一登录。使用自定义浏览器插件时，AAD 可以通过安全地检索应用程序凭据（例如目录中的用户名和密码）来自动化用户的登录过程，并代表用户将这些凭据输入到应用程序的登录页中。下面是两个用例：

1.	**管理员管理凭据** – 管理员可以创建和管理应用程序凭据，并将这些凭据分配给需要访问应用程序的用户或组。在这种情况下，最终用户不需要知道凭据，但仍然能够通过单一登录方式来访问应用程序，只需通过其访问面板或提供的链接单击它即可。这既可以让管理员对凭据进行生命周期管理，又为最终用户提供了方便，这些最终用户不需要记住或管理特定于应用的密码。在自动登录过程中，最终用户提供的凭据会进行模糊化处理；不过在技术上，用户使用 Web 调试工具还是可以找到这些凭据，因此用户和管理员仍应遵循相同的安全策略，就像这些凭据是直接由用户提供一样。需要提供在多个用户之间共享的帐户访问权限时（例如在使用社交媒体或文档共享应用程序的情况下），由管理员提供凭据很有用。

2.	**用户管理凭据** – 管理员可以将应用程序分配给最终用户或组，并允许最终用户在第一次通过访问面板访问应用程序时直接输入自己的凭据。这将为最终用户创造方便，这样他们就不需要每次访问应用程序时不断输入特定于应用的密码。对凭据进行管理时也可以借鉴这个用例，管理员可以在未来某个日期为应用程序设置新的凭据，而不会改变最终用户的应用访问体验。

在这两种情况下，凭据都存储在目录中，处于加密状态，仅在自动登录期间通过 HTTPS 进行传递。通过基于密码的单一登录，Azure AD 为不能支持联合身份验证协议的应用程序提供了方便的标识访问管理解决方案。

基于密码的 SSO 可通过浏览器扩展安全地从 Azure AD 检索应用程序和用户特定的信息并将这些信息应用于服务。Azure AD 支持的大多数第三方 SaaS 应用程序支持此功能。

对于基于密码的 SSO，最终用户的浏览器可以是：

- Windows 7 或更高版本上的 Internet Explorer 8、9、10 和 11（另请参阅 [IE Extension Deployment Guide（IE 扩展部署指南）](/documentation/articles/active-directory-saas-ie-group-policy/)）
- Chrome -- 在 Windows 7 或更高版本上，以及在 MacOS X 或更高版本上
- Firefox 26.0 或更高版本 -- 在 Windows XP SP2 或更高版本上，以及在 Mac OS X 10.6 或更高版本上

**注意：**如果浏览器扩展支持 Edge，则基于密码的 SSO 扩展将可供 Windows 10 中的 Edge 使用。
###现有的单一登录

配置应用程序的单一登录时，Azure 经典管理门户提供了第三个选项，即“现有的单一登录”。管理员使用此选项即可创建指向应用程序的链接，并将其放置在所选用户的访问面板上。

例如，如果有一个应用程序配置为使用 Active Directory 联合身份验证服务 2.0 验证用户身份，则管理员可以使用“现有的单一登录”选项在访问面板上创建指向该应用程序的链接。当用户访问该链接时，将使用 Active Directory 联合身份验证服务 2.0 或者由该应用程序提供的任何现有的单一登录解决方案来验证这些用户的身份。

###用户预配

对于某些应用程序，Azure AD 允许从 Azure 经典管理门户中使用 Windows Server Active Directory 或 Azure AD 标识信息，对第三方 SaaS 应用程序中的帐户进行自动用户预配和取消预配操作。在 Azure AD 中向用户授予这些应用程序之一的权限后，即可在目标 SaaS 应用程序中自动创建（预配）帐户。

在 Azure AD 中删除用户或更改其信息时，这些更改也会反映在 SaaS 应用程序中。这意味着，配置自动身份生命周期管理可使管理员能够从 SaaS 应用程序控制并提供自动预配和取消预配。在 Azure AD 中，这种身份生命周期管理的自动化通过用户预配启用。

有关详细信息，请参阅[在 SaaS 应用程序中自动预配和取消预配用户](/documentation/articles/active-directory-saas-app-provisioning/)

##Azure 应用程序库入门

已准备就绪？ 若要在 Azure AD 和组织所用的 SaaS 应用程序之间部署单一登录，请遵循这些指导原则。

###使用 Azure 应用程序库

[Azure Active Directory 应用程序库](http://azure.microsoft.com/zh-cn/marketplace/active-directory/all/)提供了一份已知能够支持 Azure Active Directory 单一登录的应用程序列表。

![][1]

以下是有关了解应用支持哪些功能的一些提示：

*	在[此处](http://social.technet.microsoft.com/wiki/contents/articles/20235.azure-active-directory-application-gallery-federated-saas-apps.aspx)可以找到专门支持使用 SAML，WS 联合身份验证或 OpenID Connect 协议的联合单一登录的联合应用程序列表。

找到你的应用程序后，可以遵循应用程序库和 Azure 经典管理门户中显示的分步说明启用单一登录。

###应用程序不在库中怎么办？

如果在 Azure AD 应用程序库中找不到你的应用程序，你可以选择：

*	**添加你使用的但未列出的应用** - 使用 Azure 经典管理门户内应用库中的“自定义”类别来连接组织正在使用但未列出的应用程序。你可以添加支持 SAML 2.0 的任何应用程序作为联合应用，或者添加具有 HTML 登录页的任何应用程序作为密码 SSO 应用。有关详细信息，请参阅[添加自己的应用程序](/documentation/articles/active-directory-saas-custom-apps/)一文。


*	**添加正在开发的自有应用** - 如果你自己开发了应用程序，请遵照 Azure AD 开发人员文档中的指导原则使用 Azure AD 图形 API 来实施联合单一登录或预配。有关详细信息，请参阅以下资源：
  * [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)
  * [https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet)
  * [https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet](https://github.com/AzureADSamples/WebApp-WebAPI-MultiTenant-OpenIdConnect-DotNet)
  * [https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore](https://github.com/AzureADSamples/NativeClient-WebAPI-MultiTenant-WindowsStore)

*	**请求应用集成** - 使用 [Azure AD 反馈论坛](https://feedback.azure.com/forums/169401-azure-active-directory/)请求所需应用程序的支持。

###使用 Azure 经典管理门户

你可以使用 Azure 经典管理门户中的 Active Directory 扩展来配置应用程序单一登录。首先，你需要从该门户的“Active Directory”部分中选择一个目录：

![][2]

若要管理第三方 SaaS 应用程序，可切换到所选目录的“应用程序”选项卡。管理员使用此视图可执行以下操作：

* 从 Azure AD 库添加新应用程序，以及添加你正在开发的应用
* 删除集成的应用程序
* 管理已集成的应用程序

第三方 SaaS 应用程序的典型管理任务包括：

* 使用密码 SSO 或联合 SSO（如果可用于目标 SaaS）通过 Azure AD 启用单一登录
* （可选）为用户预配和取消预配启用用户预配（标识生命周期管理）
* 对于启用了用户预配的应用程序，选择哪些用户可以访问该应用程序

对于支持联合单一登录的库应用，配置通常要求你提供附加配置设置（例如证书和元数据），以便在第三方应用程序与 Azure AD 之间建立联合信任。配置向导将指导你完成详细操作，并允许你轻松访问 SaaS 应用程序特定的数据和说明。

对于支持自动用户预配的库应用，需要为 Azure AD 提供管理 SaaS 应用程序中帐户的权限。至少需要提供向目标应用程序进行身份验证时 Azure AD 应使用的凭据。是否需要提供附加配置设置取决于应用程序的要求。

##为用户部署 Azure AD 集成的应用程序

Azure AD 提供多种可自定义的方式来向组织中的用户部署应用程序：

* Azure AD 访问面板
* Office 365 应用程序启动器
* 直接登录联合应用
* 联合、基于密码或现有的应用的深层链接

请自行决定要选择哪种方法在组织中进行部署。

###Azure AD 访问面板

https://myapps.microsoft.com 上的访问面板是一个基于 Web 的门户，它允许在 Azure Active Directory 中拥有组织帐户的最终用户查看和启动 Azure AD 管理员已向他们授予其访问权限的基于云的应用程序。如果你是使用 [Azure Active Directory Premium](https://azure.microsoft.com/pricing/details/active-directory/) 的最终用户，则还可以通过访问面板利用自助服务组管理功能。

![][3]

访问面板是与 Azure 经典管理门户分开的，因此不要求用户拥有 Azure 订阅或 Office 365 订阅。

有关 Azure AD 访问面板的详细信息，请参阅[访问面板简介](/documentation/articles/active-directory-saas-access-panel-introduction/)。

###Office 365 应用程序启动器

对于已部署 Office 365 的组织，通过 Azure AD 分配给用户的应用程序也会出现在位于 https://portal.office.com/myapps 上的 Office 365 门户中。组织中的用户可以使用此方式便捷地启动应用程序且无需使用另一个门户，建议使用 Office 365 的组织采用这种应用程序启动解决方案。

![][4]

有关 Office 365 应用程序启动器的详细信息，请参阅[让你的应用出现在 Office 365 应用启动器中](https://msdn.microsoft.com/office/office365/howto/connect-your-app-to-o365-app-launcher)。

###直接登录联合应用

大多数支持 SAML 2.0、WS 联合身份验证或 OpenID Connect 的联合应用程序也支持用户在应用程序启动，然后再通过 Azure AD 的自动重定向或单击链接进行登录。这称为服务提供者发起的登录，Azure AD 应用程序库中的大多数联合应用程序都支持这种方式（请参阅 Azure 经典管理门户上应用的单一登录配置向导中的文档链接来了解详细信息）。

![][5]

###联合、基于密码或现有应用的直接登录链接

Azure AD 还为支持基于密码单一登录、现有单一登录以及任何形式的联合单一登录的各个应用程序提供直接单一登录链接。

这些链接是专门编写的 URL，通过 Azure AD 登录过程针对特定应用程序发送给用户，用户无需从 Azure AD 访问面板或 Office 365 启动。可以在 Azure 经典管理门户上“Active Directory”部分中任何预先集成的应用程序的“仪表板”选项卡下找到这些单一登录 URL，如以下屏幕截图所示。

![][6]

可以复制这些链接，然后粘贴到任何想要提供选定应用程序登录链接的位置。该位置可以是在电子邮件，或者任何已设置用户应用程序访问权限的自定义 Web 门户。以下是 Twitter 的 Azure AD 直接单一登录 URL 示例：

https://myapps.microsoft.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced

类似于访问面板的组织特定 URL，你可以在 myapps.microsoft.com 域之后添加一个活动的或经过验证的域作为目录，以进一步自定义此 URL。这样可以确保用户无需事先输入其用户 ID，登录页就可以立即加载任何组织品牌：

https://myapps.microsoft.com/contosobuild.com/signin/Twitter/230848d52c8745d4b05a60d29a40fced

当已授权的用户单击其中一个应用程序特定的链接时，他们会先看到其组织登录页（假设他们尚未登录），登录之后重定向到该应用程序，而不会先停留在访问面板。如果用户未安装访问应用程序所需的必备组件（例如基于密码的单一登录浏览器扩展），链接将会提示用户安装缺少的扩展。如果应用程序的单一登录配置发生更改，链接 URL 也会保持不变。

这些链接使用与访问面板和 Office 365 相同的访问控制机制，只有在 Azure 经典管理门户中已分配到应用程序的用户或组能够成功通过身份验证。不过，任何未经授权的用户都会看到一条消息说明他们未获得访问权限，并会获得一个加载访问面板的链接用于查看他们有权访问的应用程序。

##相关文章

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)
- [有关如何将 SaaS 应用与 Azure Active Directory 集成的教程列表](/documentation/articles/active-directory-saas-tutorial-list/)
- [使用 Cloud App Discovery 查找未经认可的云应用程序](/documentation/articles/active-directory-cloudappdiscovery-whatis/)
- [Introduction to Managing Access to Apps（管理对应用的访问简介）](/documentation/articles/active-directory-managing-access-to-apps/)
- [比较 Azure AD 中用于管理外部标识的功能](/documentation/articles/active-directory-b2b-compare-external-identities/)

<!--Image references-->
[1]: ./media/active-directory-appssoaccess-whatis/onlineappgallery.png
[2]: ./media/active-directory-appssoaccess-whatis/azuremgmtportal.png
[3]: ./media/active-directory-appssoaccess-whatis/accesspanel.png
[4]: ./media/active-directory-appssoaccess-whatis/officeapphub.png
[5]: ./media/active-directory-appssoaccess-whatis/workdaymobile.png
[6]: ./media/active-directory-appssoaccess-whatis/deeplink.png

<!---HONumber=Mooncake_0613_2016-->