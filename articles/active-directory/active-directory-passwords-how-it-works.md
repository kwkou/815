<properties
    pageTitle="工作原理：Azure AD 密码管理 | Azure"
    description="了解 Azure AD 密码管理的各个组件，包括用户可在何处注册、重置和更改其密码，以及管理员可在何处配置、报告和启用本地 Active Directory 密码管理。"
    services="active-directory"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila"
    editor="curtand" />
<tags
    ms.assetid="618c5908-5bf6-4f0d-bf88-5168dfb28a88"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/28/2017"
    wacn.date="04/05/2017"
    ms.author="joflore" />

# Azure Active Directory 中的密码管理的工作原理
> [AZURE.IMPORTANT]
**你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。

Azure Active Directory (Azure AD) 中的密码管理包含以下逻辑组件：

- **密码管理配置门户**：你可以通过导航到 [Azure 经典管理门户](https://manage.windowsazure.cn)中你的租户所在目录的“配置”选项卡来控制如何在你的租户中管理密码的各个方面。
- **用户注册门户**：用户可以通过此 Web 门户自行注册以进行密码重置。
- **用户密码重置门户**：用户可以使用符合管理员控制的密码重置策略的各种质询来重置其自己的密码。
- **用户密码更改门户**：用户可以通过使用此 Web 门户输入旧密码并选择一个新密码来随时更改其自己的密码。
- **Azure AD Connect 的密码写回组件**：在安装 Azure AD Connect 时你可以选择启用“密码写回”功能，以便从云中管理联合或密码同步用户的密码。

## <a name="password-management-configuration-portal"></a>密码管理配置门户
可以在 [Azure 经典管理门户](https://manage.windowsazure.cn)中为特定目录配置密码管理策略，可通过导航到目录的“配置”选项卡中的“用户密码重置策略”部分来完成此操作。从此配置页面，可以控制如何在组织中管理密码的许多方面，其中包括：

- 针对目录中的所有用户启用和禁用密码重置。
- 设置用户要重置其密码必须通过的质询数（一个或两个）。
- 通过以下选择设置你希望为你的组织中的用户启用的特定类型的质询：
  - 移动电话（通过短信或语音呼叫接收验证码）
  - 办公电话（语音呼叫）
  - 备用电子邮件（通过电子邮件接收验证码）
  - 安全问题（基于认知的身份验证）
- 设置用户为了使用安全问题身份验证方法而必须注册的问题数（仅在启用了安全问题时可见）。
- 设置用户为了使用安全问题身份验证方法而在重置期间必须提供的问题数（仅在启用了安全问题时可见）。
- 使用用户在针对密码重置进行注册时可以选择使用的预先编写的、已本地化的安全问题（仅在启用了安全问题时可见）。
- 定义用户在针对密码重置进行注册时可以选择使用的自定义安全问题（仅在启用了安全问题时可见）。
- 要求用户在转到应用程序[访问面板](https://login.partner.microsoftonline.cn)时针对密码重置进行注册。
- 要求用户在经过可配置的天数后再次确认其之前注册的数据（仅在启用了强制注册时可见）。
- 提供当用户重置密码期间遇到问题时将向其显示的自定义技术支持电子邮件或 URL。
- 启用或禁用密码写回功能（使用 Azure AD Connect 部署了写回时）。
- 查看密码写回代理的状态（使用 Azure AD Connect 部署了写回时）。
- 用户自己的密码被重置时向用户发送电子邮件通知（可在 [Azure 经典管理门户](https://manage.windowsazure.cn)的“通知”部分找到）。
- 其他管理员重置他们自己的密码时向管理员发送电子邮件通知（可在 [Azure 经典管理门户](https://manage.windowsazure.cn)的“通知”部分找到）。
- 通过使用租户品牌自定义功能在用户密码重置门户和密码重置电子邮件中标注所在组织的徽标和名称（可在 [Azure 经典管理门户](https://manage.windowsazure.cn)的“目录属性”部分找到）。

若要详细了解有关在组织中配置密码管理的信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)。

## <a name="user-registration-portal"></a>用户注册门户
在用户可以使用密码重置之前，必须使用正确的身份验证数据更新其云用户帐户，以确保他们可以通过其管理员定义的适当数量的密码重置质询。管理员还可以使用 Azure 或 Office 网络门户、DirSync/Azure AD Connect 或 Windows PowerShell 代表其用户定义此身份验证信息。

但是，如果你希望让用户注册其自己的数据，那么我们还提供了一个网页，用户可以转至该网页来添加此信息。利用此页面，用户可以根据在其组织中启用的密码重置策略指定身份验证信息。此数据通过验证后将存储在其云用户帐户中，以便日后用于帐户恢复。

注册门户的外观如下：

  ![][001]

有关详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)和[最佳做法：Azure AD 密码管理](/documentation/articles/active-directory-passwords-best-practices/)。

## <a name="user-password-reset-portal"></a> 用户密码重置门户
当你启用了自助密码重置、设置了你的组织的自助密码重置策略并确保了用户在目录中具有适当的联系人数据后，你的组织中的用户将可以从使用工作或学校帐户进行登录的任何网页（例如 [portal.partner.microsoftonline.cn](https://portal.partner.microsoftonline.cn)）自动重置他们自己的密码。在这些网页上，用户会看到“无法访问你的帐户?”链接。

  ![][002]

单击此链接将打开自助服务密码重置门户。

  ![][003]

若要详细了解有关用户如何重置其密码的信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)。

## <a name="user-password-change-portal"></a>用户密码更改门户
如果用户想要更改其密码，随时可以使用密码更改门户执行此操作。用户可以通过访问面板配置文件页来访问密码更改门户，也可以通过单击 Office 365 应用程序内的“更改密码”链接进行访问。如果用户的密码过期，则在用户登录时也会自动要求用户更改其密码。

  ![][004]

在这两种情况下，如果已启用密码写回且用户为联合用户或者密码已同步，则这些更改的密码都将写回到本地 Active Directory。

密码更改门户的外观如下：

  ![][005]

若要详细了解用户可以如何更改自己的本地 Active Directory 密码，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)。


## Azure AD Connect 的密码写回组件 <a name="password-writeback-component-of-azure-ad-connect"></a>
如果组织中用户的密码源自你的本地环境（通过联合或密码同步），则可以安装 Azure AD Connect 的最新版本以便直接从云中更新这些密码。这意味着，当用户忘记或要修改其 Azure AD 密码时，他们可以直接从 Web 执行此操作。密码写回在 Azure AD Connect 安装向导中的位置如下：

  ![][007]

有关 Azure AD Connect 的详细信息，请参阅[入门：Azure AD Connect](/documentation/articles/active-directory-aadconnect/)。有关密码写回的详细信息，请参阅[入门：Azure AD 密码管理](/documentation/articles/active-directory-passwords-getting-started/)。


## 后续步骤

若要了解详细信息，请参阅下面的 Azure AD 密码重置页面：

- **你是否因登录时遇到问题而浏览至此？** 如果是这样，请了解如何[更改和重置你自己的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
- [**入门**](/documentation/articles/active-directory-passwords-getting-started/)：了解如何允许用户重置和更改其云密码或本地密码。
- [**自定义**](/documentation/articles/active-directory-passwords-customize/)：了解如何根据组织的需求自定义服务的外观和行为。
- [**最佳做法**](/documentation/articles/active-directory-passwords-best-practices/)：了解如何快速部署且有效地管理你的组织的密码。
- [**常见问题**](/documentation/articles/active-directory-passwords-faq/)：获取常见问题的解答。
- [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/)：了解如何快速排查服务的问题。
- [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/)：深入探索服务工作原理的技术细节。

[001]: ./media/active-directory-passwords-how-it-works/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-how-it-works/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-how-it-works/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-how-it-works/004.jpg "Image_004.jpg"
[005]: ./media/active-directory-passwords-how-it-works/005.jpg "Image_005.jpg"
[006]: ./media/active-directory-passwords-how-it-works/006.jpg "Image_006.jpg"
[007]: ./media/active-directory-passwords-how-it-works/007.jpg "Image_007.jpg"

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->