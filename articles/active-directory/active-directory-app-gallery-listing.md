<properties
    pageTitle="列出 Azure Active Directory 应用程序库中的应用程序"
    description="如何列出 Azure Active Directory 库中支持单一登录的应用程序 | Azure"
    services="active-directory"
    documentationcenter="dev-center-name"
    author="bryanla"
    manager="mbaldwin"
    editor="" />
<tags
    ms.assetid="820acdb7-d316-4c3b-8de9-79df48ba3b06"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="01/07/2017"
    wacn.date="02/07/2017"
    ms.author="mbaldwin" />

# 列出 Azure Active Directory 应用程序库中的应用程序
若要列出 Azure AD 库中支持使用 Azure Active Directory 进行单一登录的应用程序，该应用程序首先需要实现下列集成模式之一：

- **OpenID Connect** - 与 Azure AD 直接集成，使用 OpenID Connect 进行身份验证，并使用 Azure AD 许可 API 进行配置。如果你刚刚开始集成并且你的应用程序不支持 SAML，则这是建议的模式。
- **SAML** - 你的应用程序已经能够使用 SAML 协议配置第三方标识提供程序。

下面显示了每种模式的列出要求。

## OpenID Connect 集成
若要将应用程序与 Azure AD 集成，请遵循[开发人员说明](/documentation/articles/active-directory-authentication-scenarios/)。然后回答以下问题并发送到 waadpartners@microsoft.com。

- 提供应用程序测试租户或帐户的凭据，以便 Azure AD 团队用来测试集成。
- 为 Azure AD 团队提供有关如何使用 [Azure AD 许可框架](/documentation/articles/active-directory-integrating-applications/#overview-of-the-consent-framework/)登录并将 Azure AD 实例连接到应用程序的说明。
- 为 Azure AD 团队提供在应用程序上测试单一登录所需的其他说明。
- 提供以下信息：

	> 公司名称：
	> 
	> 公司网站：
	> 
	> 应用程序名称：
	> 
	> 应用程序说明（不超过 256 个字符）：
	> 
	> 应用程序网站（信息性）：
	> 
	> 应用程序技术支持网站或联系信息：
	> 
	> 应用程序的客户端 ID，在 https://manage.windowsazure.cn 上的应用程序详细信息中显示：
	> 
	> 客户注册和/或购买应用程序时所用的应用程序注册 URL：
	> 
	> 选择应用程序所要列入到的最多三个类别（有关可用类别，请参阅“Azure Active Directory 应用商店”）：
	> 
	> 附加应用程序小图标（PNG 文件，45 x 45 像素，实色背景）：
	> 
	> 附加应用程序大图标（PNG 文件，215 x 215 像素，实色背景）：
	> 
	> 附加应用程序徽标（PNG 文件，150 x 122 像素，透明背景色）：
	> 
	> 

## SAML 集成
如果使用这些说明添加自定义应用程序，支持 SAML 2.0 的任何应用程序都可直接与 Azure AD 租户集成。应用程序经测试可与 Azure AD 集成后，请将以下信息发送到 <mailto:waadpartners@microsoft.com>。

- 提供应用程序测试租户或帐户的凭据，以便 Azure AD 团队用来测试集成。
- 提供应用程序的 SAML 登录 URL、颁发者 URL（实体 ID）和回复 URL（断言使用者服务）的值。如果你通常在 SAML 元数据文件中提供了这些值，请随附该文件。
- 提供有关如何在使用 SAML 2.0 的应用程序中配置 Azure AD 作为标识提供者的简短描述。如果应用程序支持通过自助服务管理门户将 Azure AD 配置为标识提供者，请确保上面提供的凭据可用于进行此项设置。
- 提供以下信息：

	> 公司名称：
	> 
	> 公司网站：
	> 
	> 应用程序名称：
	> 
	> 应用程序说明（不超过 256 个字符）：
	> 
	> 应用程序网站（信息性）：
	> 
	> 应用程序技术支持网站或联系信息：
	> 
	> 客户注册和/或购买应用程序时所用的应用程序注册 URL：
	> 
	> 最多为应用程序选择三个要列入的类别：
	> 
	> 附加应用程序小图标（PNG 文件，45 x 45 像素，实色背景）：
	> 
	> 附加应用程序大图标（PNG 文件，215 x 215 像素，实色背景）：
	> 
	> 附加应用程序徽标（PNG 文件，150 x 122 像素，透明背景色）：
	> 
	> 

<!---HONumber=Mooncake_0120_2017-->
<!---Update_Description: wording update -->