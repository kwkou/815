<properties
    pageTitle="最佳实践：Azure AD 密码管理 | Azure"
    description="有关 Azure Active Directory 中密码管理的部署和使用最佳做法、示例最终用户文档和培训指南。"
    services="active-directory"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila"
    editor="curtand" />
<tags
    ms.assetid="f8cd7e68-2c8e-4f30-b326-b22b16de9787"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/28/2017"
    wacn.date="04/05/2017"
    ms.author="joflore" />  


# 部署密码管理并向用户培训其用法
> [AZURE.IMPORTANT]
**是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
>
>

启用密码重置之后，必须执行的下一个步骤就是让用户使用组织中的服务。为此，需要确保已正确将用户配置为使用服务，同时用户已完成需要的培训，可以顺利地管理自己的密码。本文将解释以下概念：

- [**如何为密码管理配置用户**](#how-to-get-users-configured-for-password-reset)
  - [如何为密码重置配置帐户](#what-makes-an-account-configured)
  - [自行填充身份验证数据的方式](#ways-to-populate-authentication-data)
- [**组织启用密码重置的最佳方式**](#what-is-the-best-way-to-roll-out-password-reset-for-users)
  - [基于电子邮件的启用方式与示例电子邮件通信](#email-based-rollout)
  - [为用户创建自己的自定义密码管理门户](#creating-your-own-password-portal)
  - [如何使用强制注册来强制用户在登录时注册](#using-enforced-registration)
  - [如何上载用户帐户的身份验证数据](#uploading-data-yourself)
- [**示例用户和支持培训材料（即将推出！）**](#sample-training-materials)

## 如何为密码重置配置用户 <a name="how-to-get-users-configured-for-password-reset"></a>
本部分介绍确保组织中的每个用户在忘记其密码时可以有效使用自助服务密码重置的各种方法。

### 如何配置帐户 <a name="what-makes-an-account-configured"></a>
只有满足以下**所有**条件，用户才能使用密码重置：

1. 必须在目录中启用密码重置。请阅读[让用户重置其 Azure AD 密码](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-their-azure-ad-passwords/)或[让用户重置或更改其 AD 密码](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords/)，了解如何启用密码重置
2. 用户必须获得许可。
   - 云用户必须分配有**任何付费型 Office 365 许可证**，或者 **AAD Basic 许可证**或 **AAD Premium 许可证**。
   - 本地用户（联合或哈希同步）**必须分配有 AAD Premium 许可证**。
3. 用户必须具有按照当前密码重置策略**定义的最小身份验证数据集**。
   - 如果目录中的对应字段包含的数据格式正确，则将身份验证数据视为已定义。
   - 如果配置了一个关口策略，则将最小身份验证数据集定义为**至少一个**启用的身份验证选项；如果配置了两个关口策略，则将其定义为**至少两个**启用的身份验证选项。
4. 如果用户使用的是本地帐户，则必须启用并打开[密码写回](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords/)

### 填充身份验证数据的方式 <a name="ways-to-populate-authentication-data"></a>
关于如何为所在组织中的用户指定用于密码重置的数据，有几种选择。

- 在 [Azure 管理门户](https://manage.windowsazure.cn)或 [Office 365 管理门户](https://portal.partner.microsoftonline.cn)中编辑用户
- 使用 Azure AD Sync 将用户属性从本地 Active Directory 域同步到 Azure AD
- [遵循此处所述的步骤](/documentation/articles/active-directory-passwords-learn-more/#how-to-access-password-reset-data-for-your-users/)使用 Windows PowerShell 编辑用户属性。
- 通过将用户引导至位于 [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn) 的注册门户，使用户能够注册自己的数据
- 通过将“[要求用户在登录时注册?](/documentation/articles/active-directory-passwords-customize/#require-users-to-register-when-signing-in/)”配置选项设置为“是”，要求用户在登录其 Azure AD 帐户时注册密码重置。

用户无需注册密码重置，系统便能正常工作。例如，如果你的现有手机号码或办公电话号码位于本地目录中，则可以在 Azure AD 中同步它们，我们将自动使用它们进行密码重置。

你也可以阅读[密码重置使用数据的方式](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset/)和[如何使用 PowerShell 填充单个身份验证字段](/documentation/articles/active-directory-passwords-learn-more/#how-to-access-password-reset-data-for-your-users/)。

## 为用户启用密码重置的最佳方式是什么？ <a name="what-is-the-best-way-to-roll-out-password-reset-for-users"></a>
密码重置的常规启用步骤如下：

1. 转到你的目录
2. 点击“USERS"
3. 点击你要重置密码的用户
4. 点击页面下方的“RESET PASSWORD"
5. 在弹出的页面上点击“reset"

通知用户可以注册并在组织中使用密码重置的方法有多种。下面将详细介绍。

### 基于电子邮件的启用 <a name="email-based-rollout"></a>
或许，通知用户有关注册或使用密码重置事宜的最简单方法是向其发送一封电子邮件，指导他们如何操作。下面是可实现此目的的模板。可以将颜色/徽标随意替换为自己选择的自定义项，以符合具体需求。

  ![][001]

你可以[从此](https://1drv.ms/f/s!AharQMeRmrWggcNnPlk_7xHInYJzow)处下载电子邮件模板。

### 创建自己的密码门户 <a name="creating-your-own-password-portal"></a>
对于部署密码管理功能的大型客户而言，一种策略是创建用户可用来在单个位置管理所有密码相关事项的单一“密码门户”。

我们的许多大客户都选择创建根 DNS 条目，例如 https://passwords.contoso.com，其中包含 Azure AD 密码重置门户、密码重置注册门户和密码更改页面的链接。这样，你发出的任何电子邮件通信或传单都可以包含一个容易记住的，让用户在开始使用服务时随时可以访问的 URL。

为演示此功能，我们创建了一个使用最新响应性 UI 设计模式且能够在所有浏览器和移动设备上运行的简单页面。

  ![][007]

你可以[在此处下载网站模板](https://github.com/kenhoff/password-reset-page)。我们建议你根据组织的需求自定义徽标和颜色。

### 使用强制注册 <a name="using-enforced-registration"></a>
如果希望你的用户自行注册密码重置，还可以强制他们在登录到访问面板（网址为 [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn)）时注册。你可以从目录的“配置”选项卡启用“要求用户在登录到访问面板时注册”选项，从而启用此选项。

也可以通过将“用户必须确认其联系人数据前的天数”选项修改为一个不是零的值，来定义在可配置的一段时间后是否要求用户重新注册。有关详细信息，请参阅[自定义用户密码管理行为](/documentation/articles/active-directory-passwords-customize/#password-management-behavior/)。

  ![][002]

启用此选项后，当用户登录到访问面板时，他们将看到一个弹出窗口，通知他们管理员已要求其验证联系信息。他们可以使用它来重置其密码（如果他们曾失去对其帐户的访问权限）。

  ![][003]

单击“立即验证”可将用户带到**密码重置注册门户**（网址为 [https://login.partner.microsoftonline.cn）](https://login.partner.microsoftonline.cn)并要求他们注册。单击“取消”按钮或关闭窗口可撤消采用此方法进行的注册，但如果用户未注册，则每次登录时都会收到提醒。

  ![][004]

### 自行上载数据 <a name="uploading-data-yourself"></a>
如果要自行上载身份验证数据，用户无需注册密码重置便可重置其密码。只要用户在其帐户上定义的身份验证数据符合你定义的密码重置策略，便能重置其密码。

要了解可以通过 DirSync 或 Windows PowerShell 设置哪些属性，请参阅[密码重置使用的数据](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset/)。

你可以执行以下步骤，通过 [Azure 经典管理门户](https://manage.windowsazure.cn)上载身份验证数据：

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)的 **Active Directory 扩展**中导航到你的目录。
2. 单击“用户”选项卡。
3. 从列表中选择所需的用户。
4. 在第一个选项卡上，你将发现“备用电子邮件”，它可作为启用密码重置的属性使用。

   ![][005]
5. 单击“工作信息”选项卡。
6. 在此页面上，你将发现“办公电话”、“移动电话”、“身份验证电话”和“身份验证电子邮件”。也可以将这些属性设置为允许用户重置其密码。也可以将这些属性设置为允许用户重置其密码。

   ![][006]  


若要了解如何使用上述每个属性，请参阅[密码重置使用的数据](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset/)。

请参阅[如何从 PowerShell 访问用户的密码重置数据](/documentation/articles/active-directory-passwords-learn-more/#how-to-access-password-reset-data-for-your-users/)，以了解如何使用 PowerShell 读取和设置此数据。

## 示例培训材料 <a name="sample-training-materials"></a>
我们正在准备示例培训材料，以帮助你的 IT 部门和用户快速了解如何部署及使用密码重置。敬请期待！

<br/> 
<br/> 
<br/>

## 后续步骤
以下是所有 Azure AD 密码重置文档页面的链接：

- **你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
- [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
- [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
- [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
- [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
- [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
- [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节

[001]: ./media/active-directory-passwords-best-practices/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-best-practices/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-best-practices/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-best-practices/004.jpg "Image_004.jpg"
[005]: ./media/active-directory-passwords-best-practices/005.jpg "Image_005.jpg"
[006]: ./media/active-directory-passwords-best-practices/006.jpg "Image_006.jpg"
[007]: ./media/active-directory-passwords-best-practices/007.jpg "Image_007.jpg"

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->