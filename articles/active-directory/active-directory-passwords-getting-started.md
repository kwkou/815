<properties
    pageTitle="入门：Azure AD 密码管理 | Azure"
    description="使用户能够重置其自己的密码和了解密码重置先决条件，并启用密码写回来管理 Active Directory 中的本地密码。"
    services="active-directory"
    keywords="Active Directory 密码管理、密码管理、重置 Azure AD 密码"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila"
    editor="curtand" />
<tags
    ms.assetid="bde8799f-0b42-446a-ad95-7ebb374c3bec"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/08/2017"
    wacn.date="04/05/2017"
    ms.author="joflore" />  


# 密码管理入门
> [AZURE.IMPORTANT]
**你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
>
>

只要完成几个简单的步骤，就能让你的用户管理他们自己的云 Azure Active Directory 密码或本地 Active Directory 密码。在确保符合几个简单的先决条件之后，你将在不知不觉中为整个组织启用密码更改和重置。本文将引导你完成以下操作思路：

- [**我们的客户提供的供你在开始之前阅读的重要提示**](#top-tips-from-our-customers-to-read-before-you-begin)
 - [**重要提示：文档导航** - 使用目录和浏览器的查找功能来查找答案](#top-tip-documentation-navigation---use-our-table-of-contents-and-your-browsers-find-feature-to-find-answers)
 - [**提示 1：许可** - 务必了解许可要求](#tip-1-licensing---make-sure-you-understand-the-licensing-requirements)
 - [**提示 2：测试** - 使用最终用户而非管理员进行测试，试运行时使用一小组用户](#tip-2-testing---test-with-an-end-user-not-an-administrator-and-pilot-with-a-small-set-of-users)
 - [**提示 3：部署** - 为用户预填充数据，这样用户就不需要注册](#tip-3-deployment---pre-populate-data-for-your-users-so-they-dont-have-to-register)
 - [**提示 4：部署** - 使用密码重置，这样就不需要向用户发送临时密码](#tip-4-deployment---use-password-reset-to-obviate-the-need-to-communicate-temporary-passwords)
 - [**提示 5：写回** - 查看 AAD Connect 计算机上的应用程序事件日志，排查密码写回问题](#tip-5-writeback---look-at-the-application-event-log-on-your-aad-connect-machine-to-troubleshoot-password-writeback)
 - [**提示 6：写回** - 确保为密码写回启用正确的权限、防火墙规则和连接设置](#tip-6-writeback---ensure-you-enable-the-correct-permissions-firewall-rules-and-connection-settings-for-password-writeback)
 - [**提示 7：报告** - 通过 Azure AD SSPR 审核日志了解谁正在注册或重置密码](#tip-7-reporting---see-who-is-registering-or-resetting-passwords-with-the-azure-ad-sspr-audit-logs)
 - [**提示 8：故障排除** - 请阅读我们的故障排除指南和常见问题解答，这样可以解决许多问题](#tip-8-troubleshoot---read-our-troubleshooting-guide-and-faq-to-solve-many-issues)
 - [**提示 9：故障排除** - 如果仍需帮助，请提供充足的信息，以便我们为你提供帮助](#tip-9-troubleshoot---if-you-still-need-help-include-enough-information-for-us-to-assist-you)
- [**如何让用户重置其 Azure Active Directory 密码**](#enable-users-to-reset-their-azure-ad-passwords)
 - [自助密码重置先决条件](#prerequisites)
 - [步骤 1：配置密码重置策略](#step-1-configure-password-reset-policy)
 - [步骤 2：为测试用户添加联系人数据](#step-2-add-contact-data-for-your-test-user)
 - [步骤 3：以用户身份重置密码](#step-3-reset-your-azure-ad-password-as-a-user)
- [**如何让用户重置或更改其本地 Active Directory 密码**](#enable-users-to-reset-or-change-their-ad-passwords)
 - [密码写回先决条件](#writeback-prerequisites)
 - [步骤 1：下载最新版本的 Azure AD Connect](#step-1-download-the-latest-version-of-azure-ad-connect)
 - [步骤 2：通过 UI 或 PowerShell 在 Azure AD Connect 中启用密码写回并验证](#step-2-enable-password-writeback-in-azure-ad-connect)
 - [步骤 3：配置防火墙](#step-3-configure-your-firewall)
 - [步骤 4：设置适当的权限](#step-4-set-up-the-appropriate-active-directory-permissions)
 - [步骤 5：以用户身份重置 AD 密码并验证](#step-5-reset-your-ad-password-as-a-user)

## 我们的客户提供的供你在开始之前阅读的重要提示
下面是一些重要提示，这些提示对于需要在其组织中部署密码管理的客户很有用。

- [**重要提示：文档导航** - 使用目录和浏览器的查找功能来查找答案](#top-tip-documentation-navigation---use-our-table-of-contents-and-your-browsers-find-feature-to-find-answers)
- [**提示 1：许可** - 务必了解许可要求](#tip-1-licensing---make-sure-you-understand-the-licensing-requirements)
- [**提示 2：测试** - 使用最终用户而非管理员进行测试，试运行时使用一小组用户](#tip-2-testing---test-with-an-end-user-not-an-administrator-and-pilot-with-a-small-set-of-users)
- [**提示 3：部署** - 为用户预填充数据，这样用户就不需要注册](#tip-3-deployment---pre-populate-data-for-your-users-so-they-dont-have-to-register)
- [**提示 4：部署** - 使用密码重置，这样就不需要向用户发送临时密码](#tip-4-deployment---use-password-reset-to-obviate-the-need-to-communicate-temporary-passwords)
- [**提示 5：写回** - 查看 AAD Connect 计算机上的应用程序事件日志，排查密码写回问题](#tip-5-writeback---look-at-the-application-event-log-on-your-aad-connect-machine-to-troubleshoot-password-writeback)
- [**提示 6：写回** - 确保为密码写回启用正确的权限、防火墙规则和连接设置](#tip-6-writeback---ensure-you-enable-the-correct-permissions-firewall-rules-and-connection-settings-for-password-writeback)
- [**提示 7：故障排除** - 请阅读我们的故障排除指南和常见问题解答，这样可以解决许多问题](#tip-8-troubleshoot---read-our-troubleshooting-guide-and-faq-to-solve-many-issues)
- [**提示 8：故障排除** - 如果仍需帮助，请提供充足的信息，以便我们为你提供帮助](#tip-9-troubleshoot---if-you-still-need-help-include-enough-information-for-us-to-assist-you)

### 重要提示：文档导航 - 使用目录和浏览器的查找功能来查找答案
如果你使用我们的文档，你会发现，我们会尽量在目录中提供所有相关内容的快速链接，方便管理员进行了解。

请查看以下目录：
- [Azure AD 密码重置：文档目录](/documentation/articles/active-directory-passwords/)

### 提示 1：许可 - 务必了解许可要求
为了启用 Azure AD 密码重置，必须在组织中至少分配一次许可证。我们不强制要求就密码重置体验本身进行单个用户的许可，但是，如果你在使用此功能时没有向用户分配许可证，则会被视为不符合 Microsoft 许可协议，因此仍需向相关用户分配许可证。

可以阅读下面这些文档，了解进行密码重置时需要哪些许可证。
- [常规密码重置许可信息](/documentation/articles/active-directory-passwords-customize#what-customization-options-are-available/)
- [按功能密码重置许可信息](/documentation/articles/active-directory-passwords#pricing-and-availability/)
- [密码写回支持的方案](/documentation/articles/active-directory-passwords-learn-more#scenarios-supported-for-password-writeback/)

### 提示 2：测试 - 使用最终用户而非管理员进行测试，试运行时使用一小组用户
使用管理员进行测试时，我们会强制实施管理员密码重置策略，该策略定义如下。这意味着，你不会看到我们为你的最终用户配置的策略的预期结果。

在管理性的 UX ONLY 中配置的策略适用于最终用户而非管理员。为了确保组织的安全，Microsoft 强制实施针对管理员的强默认密码重置策略，该策略可能不同于你为最终用户设置的策略。

#### 管理员密码重置策略
- **适用对象** - 任何管理员角色（全局管理员、支持管理员、密码管理员等）
- **单门策略适用条件...**
 - ...启动/创建试用版后头 30 天，**或**
 - ...虚域不存在**且** Azure AD Connect 未在同步标识
 - **_必需条件_**：“身份验证电子邮件”、“备用电子邮件”、“身份验证电话”、“移动电话”或“办公电话”中的任何**一个**有值存在
- **双门策略适用条件...**
 - ...头 30 天试用已过，**或**
 - ...虚域存在，**或**
 - ... 已允许 Azure AD Connect 从本地环境同步标识
 - _**必需条件**_：“身份验证电子邮件”、“备用电子邮件”、“身份验证电话”、“移动电话”或“办公电话”中的任何**两个**有值存在

### 提示 3：部署 - 为用户预填充数据，这样用户就不需要注册
许多人并未认识到，使用此功能不需要要让用户注册密码重置。预先为用户设置电话或电子邮件属性后，即可立刻将密码重置推广到整个组织，**不需要用户执行任何操作！**

若要了解如何通过 API、PowerShell 或 Azure AD Connect 执行该操作，请阅读以下文档：
- [在无需用户注册的情况下部署密码重置](/documentation/articles/active-directory-passwords-learn-more#deploying-password-reset-without-requiring-end-user-registration/)
- [密码重置使用哪些数据？](/documentation/articles/active-directory-passwords-learn-more#what-data-is-used-by-password-reset/)

### 提示 4：部署 - 使用密码重置，这样就不需要向用户发送临时密码
这是对提示 3 的补充。针对密码重置预配置用户以后，设想一下这样一个场景：一位雇员首次加入你的公司。你现在可以直接让其导航到 [Azure AD 密码重置门户](https://passwordreset.microsoftonline.com)来重置其密码，而不必向其发送临时密码。

若要了解如何通过 API、PowerShell 或 Azure AD Connect 执行该操作，请阅读以下文档。预填充该数据后，即可让用户重置其密码并立即访问其帐户：
- [在无需用户注册的情况下部署密码重置](/documentation/articles/active-directory-passwords-learn-more#deploying-password-reset-without-requiring-end-user-registration/)
- [密码重置使用哪些数据？](/documentation/articles/active-directory-passwords-learn-more#what-data-is-used-by-password-reset/)

### 提示 5：写回 - 查看 AAD Connect 计算机上的应用程序事件日志，排查密码写回问题
Azure AD Connect 应用程序事件日志包含大量的日志记录信息，这些信息主要描述实时发生的与密码写回服务相关的事件。若要访问该日志，请执行以下步骤：

1. 登录到 **Azure AD Connect** 计算机
2. 按“开始”并键入“事件查看器”，打开“Windows 事件查看器”
3. 打开“应用程序”事件日志
4. 查找来自 **PasswordResetService** 或 **ADSync** 源的事件，详细了解可能出现的问题

如需可能出现在此日志中的事件的完整列表以及更多有关密码写回的故障排除指南，请参阅：
- [故障排除：密码写回](/documentation/articles/active-directory-passwords-troubleshoot#troubleshoot-password-writeback/)
- [写回事件日志错误代码](/documentation/articles/active-directory-passwords-troubleshoot#password-writeback-event-log-error-codes/)
- [故障排除：密码写回连接](/documentation/articles/active-directory-passwords-troubleshoot#troubleshoot-password-writeback-connectivity/)
- [写回部署 - 步骤 3：配置防火墙](#step-3-configure-your-firewall)
- [写回部署 - 步骤 4：设置适当的权限](#step-4-set-up-the-appropriate-active-directory-permissions)

### 提示 6：写回 - 确保为密码写回启用正确的权限、防火墙规则和连接设置
为了使写回正常工作，必须确保：

1. 已为使用密码写回功能的用户设置适当的 **Active Directory 权限**，使之有权在 AD 中修改其密码和帐户解锁标志
2. 已打开适当的**防火墙端口**，因此密码写回服务可以通过出站连接安全地与外界通信
3. 已为密码重置服务 URL（例如服务总线）设置适当的**防火墙例外**
4. **代理和防火墙不终止空闲的出站连接**。建议将空闲时间设置为至少 10 分钟

针对密码写回配置权限和防火墙规则时，如需完整的故障排除指南列表和具体的准则，请参阅：
- [故障排除：密码写回](/documentation/articles/active-directory-passwords-troubleshoot#troubleshoot-password-writeback/)
- [写回事件日志错误代码](/documentation/articles/active-directory-passwords-troubleshoot#password-writeback-event-log-error-codes/)
- [故障排除：密码写回连接](/documentation/articles/active-directory-passwords-troubleshoot#troubleshoot-password-writeback-connectivity/)
- [写回部署 - 步骤 3：配置防火墙](#step-3-configure-your-firewall)
- [写回部署 - 步骤 4：设置适当的权限](#step-4-set-up-the-appropriate-active-directory-permissions)

### 提示 7：故障排除 - 请阅读我们的故障排除指南和常见问题解答，这样可以解决许多问题
你是否知道密码重置有大量故障排除指南和常见问题解答？ 如果你遇到问题，很有可能会在下面的链接中找到答案。

密码重置故障排除指南和常见问题解答的链接：
- [排查密码管理问题](/documentation/articles/active-directory-passwords-troubleshoot/)
- [密码管理常见问题](/documentation/articles/active-directory-passwords-faq/)

### 提示 8：故障排除 - 如果仍需帮助，请提供充足的信息，以便我们为你提供帮助
如果仍需故障排除帮助，我们很乐意为你提供帮助。你可以提交支持案例，也可以通过客户管理团队直接与我们联系。欢迎咨询！

但在联系我们之前，请**确保收集下面要求的所有信息**，以便我们为你提供快速帮助！
- [需要帮助时应包含的信息](/documentation/articles/active-directory-passwords-troubleshoot#information-to-include-when-you-need-help/)

#### 提供密码重置反馈的方式
- [功能请求或故障排除 - 在 Azure AD MSDN 论坛上发帖](https://social.msdn.microsoft.com/Forums/azure/home?forum=WindowsAzureAD)
- [功能请求或故障排除 - 在 StackOverflow 上发帖](http://stackoverflow.com/questions/tagged/azure-active-directory)
- [仅功能请求 - 在 UserVoice 上留言](https://feedback.azure.com/forums/169401-azure-active-directory)

## 让用户重置其 Azure AD 密码
本部分详细介绍如何为 AAD 云目录启用自助密码重置、注册用户进行自助密码重置，以及最终以用户身份执行测试性的自助密码重置。

- [自助密码重置先决条件](#prerequisites)
- [步骤 1：配置密码重置策略](#step-1-configure-password-reset-policy)
- [步骤 2：为测试用户添加联系人数据](#step-2-add-contact-data-for-your-test-user)
- [步骤 3：以用户身份重置密码](#step-3-reset-your-azure-ad-password-as-a-user)


### <a name="prerequisites"></a>先决条件
你必须先完成以下先决条件才能启用和使用自助密码重置：

- 创建 AAD 租户。有关详细信息，请参阅 [Azure AD 入门](/documentation/services/identity/)。
- 获取 Azure 订阅。有关详细信息，请参阅[什么是 Azure AD 租户？](/documentation/articles/active-directory-administer/#what-is-an-azure-ad-tenant/)
- 将你的 AAD 租户与 Azure 订阅相关联。有关详细信息，请参阅 [Azure 订阅与 Azure AD 的关联方式](https://msdn.microsoft.com/zh-cn/library/azure/dn629581.aspx)。
- 升级到 Azure AD Premium、Basic 或使用 O365 付费许可证。有关详细信息，请参阅 [Azure Active Directory 版本](/pricing/details/identity/)。

  > [AZURE.NOTE]
  若要为云用户启用自助密码重置，必须升级到 Azure AD Premium、Azure AD Basic 或 O365 付费许可证。若要为本地用户启用自助密码重置，必须升级到 Azure AD Premium。有关详细信息，请参阅 [Azure Active Directory 版本](/pricing/details/identity/)。这些信息包括以下任务的详细说明：如何注册 Azure AD Premium 或 Basic、如何激活你的许可计划并激活 Azure AD 访问权限，以及如何为管理员和用户帐户分配访问权限。
  >
  >
- 在 AAD 目录中创建至少一个管理员帐户和一个用户帐户。
- 为所创建的管理员和用户帐户分配 AAD Premium、Basic 或 O365 付费许可证。

### <a name="step-1-configure-password-reset-policy"></a>步骤 1：配置密码重置策略
若要配置用户密码重置策略，请完成以下步骤：

1. 打开选择的浏览器并转到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)的左侧导航栏上，找到 **Active Directory 扩展**。

	![Azure AD 中的密码管理][001]  

3. 在“目录”选项卡上，单击你想在其中配置用户密码重置策略的目录，例如 Wingtip Toys。

    ![][002]  

4. 单击“配置”选项卡。

	![][003]  


5. 在“配置”选项卡上，向下滚动到“用户密码重置策略”部分。你将在此为给定目录配置用户密码重置策略的各个方面。*如果未看见“配置”选项卡，请确保已注册了 Azure Active Directory Premium 或 Basic，并为要配置此功能的管理员帐户__分配了许可证__。*

	> [AZURE.NOTE]
	**设置的策略仅适用于组织中的最终用户，而不是管理员**。出于安全原因，Microsoft 控制了管理员的密码重置策略。针对管理员的当前策略要求两个质询 - 移动电话和电子邮件地址。

	![][004]  

6. 若要配置用户密码重置策略，请将“可进行密码重置的用户”开关滑动到“是”设置。这将显示更多控件，通过这些控件你可以配置此功能如何在你目录中工作。根据需要随意自定义密码重置。如果你想详细了解每个密码重置策略控件的作用，请参阅[自定义：Azure AD 密码管理](/documentation/articles/active-directory-passwords-customize/)。

	![][005]  

7. 根据租户需求配置用户密码重置策略后，请单击屏幕底部的“保存”按钮。

	> [AZURE.NOTE]
	建议使用两个质询用户密码重置策略，以便你能够了解该功能如何在最复杂的情况下工作。
	>
	>

	![][006]  


	> [AZURE.NOTE]
	**设置的策略仅适用于组织中的最终用户，而不是管理员**。出于安全原因，Microsoft 控制了管理员的密码重置策略。针对管理员的当前策略要求两个质询 - 移动电话和电子邮件地址。
	>
	>

### 步骤 2：为测试用户添加联系人数据
关于如何为所在组织中的用户指定用于密码重置的数据，有几种选择。

- 在 [Azure 经典管理门户](https://manage.windowsazure.cn)或 [Office 365 管理门户](https://portal.partner.microsoftonline.cn)中编辑用户
- 使用 AAD Connect 将用户属性从本地 Active Directory 域同步到 Azure AD
- 使用 Windows PowerShell 编辑用户属性
- 通过将用户引导至位于 [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn) 的注册门户，使用户能够注册其自己的数据
- 要求用户在登录到位于 [https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn) 的访问面板时注册以进行密码重置，方法是将“登录到访问面板时要求用户注册”SSPR 配置选项设置为“是”。

如果你想要详细了解密码重置使用哪些数据，以及此数据的任何格式要求，请参阅[密码重置使用哪些数据？](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset/)

#### 通过用户注册门户添加用户联系人数据
1. 为使用密码重置注册门户，你必须为你所在组织的用户提供指向此页面的链接 ([https://login.partner.microsoftonline.cn](https://login.partner.microsoftonline.cn))，或者启用要求用户自动注册的选项。一旦他们单击此链接，将要求他们使用组织帐户登录。这样做之后，他们将看到以下页面：

	![][007]  

2. 在此，用户可提供并验证其移动电话、备用电子邮件地址或安全提问。这是验证移动电话的界面。

	![][008]
3. 用户指定此信息后，页面将更新以指示此信息是否有效（下面已将信息模糊处理）。单击“完成”或“取消”按钮后，用户将回到访问面板。

	![][009]  

4. 一旦用户验证这两条信息后，将用其提供的数据更新其个人资料。本例中，已手动指定**办公电话号码**，因此用户可将其用作重置密码的联系方式。

	![][010]  


### 步骤 3：以用户身份重置密码
由于你已配置用户重置策略并为用户指定了联系人详细信息，此用户将可以进行自助密码重置。

#### 执行自助密码重置
1. 如果你转到诸如 [**portal.partner.microsoftonline.cn**](http://portal.partner.microsoftonline.cn) 的站点，你将看到如下登录屏幕。单击“无法访问你的帐户?”链接以测试密码重置 UI。

	![][011]  

2. 单击“无法访问你的帐户?”后，你将转到一个新的页面，此页面将会要求你提供希望为其重置密码的**用户 ID**。在此处输入你的测试**用户 ID**，输入验证码，然后单击“下一步”。

	![][012]  

3. 由于在此案例中用户已指定**办公电话**、**移动电话**和**备用电子邮件**，你将看到他（她）可以将所有这三个选项用于通过第一次质询。

	![][013]  

4. 在这种情况下，请选择首先**拨打****办公电话**。请注意，选择基于电话的方式时，将要求用户先**验证其电话号码**，然后才能重置密码。这是为了防止恶意个人向你组织中的用户拨打垃圾电话。

	![][014]  

5. 用户确认其电话号码后，单击“呼叫”会出现旋转图标，随后用户的电话铃声会响起。用户拿起电话后，将播放一条消息，指示**用户应按“#”**确认其帐户。按下此键将自动验证此用户是否通过第一次质询，并将使 UI 前进到第二个验证步骤。

	![][015]  

6. 通过第一次质询后，UI 自动更新，将此质询从用户的选择列表中删除。在本例中，由于首先使用的是**办公电话**，因此只有**移动电话**和**备用电子邮件**仍然是可用作第二个验证步骤的质询的有效选项。单击“向我的备用电子邮件发送电子邮件”选项。完成该操作后，按“电子邮件”将向存档的备用电子邮件发送电子邮件。

	![][016]  

7. 下面是用户将看到的电子邮件的示例 - 请注意租户品牌：

	![][017]  

8. 电子邮件送达后，页面将更新，你将能够在如下所示的输入框中输入电子邮件中找到的验证码。输入适当的代码后，“下一步”按钮亮起，你将能够通过第二个验证步骤。

	![][018]  

9. 满足组织策略的要求后，你可以选择一个新密码。验证密码是否符合 AAD“强”密码要求（参阅 [Azure AD 中的密码策略](https://msdn.microsoft.com/zh-cn/library/azure/jj943764.aspx)），显示强度验证器以指示用户输入的密码是否符合该策略。

	![][019]  

10. 提供符合组织策略的匹配密码后，你的密码将会重置，你可以立即用新密码登录。

	![][020]  


## 让用户重置或更改其 AD 密码
本部分将指导你配置密码重置，以便将密码写回到本地 Active Directory。

- [密码写回先决条件](#writeback-prerequisites)
- [步骤 1：下载最新版本的 Azure AD Connect](#step-1-download-the-latest-version-of-azure-ad-connect)
- [步骤 2：通过 UI 或 PowerShell 在 Azure AD Connect 中启用密码写回并验证](#step-2-enable-password-writeback-in-azure-ad-connect)
- [步骤 3：配置防火墙](#step-3-configure-your-firewall)
- [步骤 4：设置适当的权限](#step-4-set-up-the-appropriate-active-directory-permissions)
- [步骤 5：以用户身份重置 AD 密码并验证](#step-5-reset-your-ad-password-as-a-user)

### 写回先决条件
在能够启用和使用密码写回之前，你必须确保满足以下先决条件：

- 必须拥有已启用 Azure AD Premium 的 Azure AD 租户。有关详细信息，请参阅 [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。
- 必须将 Azure AD Connect 连接到主域控制器模拟器，以便密码写回能够工作。可以根据需要通过右键单击 Active Directory 同步连接器的“属性”，然后选择“配置目录分区”将 Azure AD Connect 配置为使用主域控制器。在这里，请查找“域控制器连接设置”部分，并选中标题为“仅使用首选的域控制器”的框。注意：如果首选的 DC 不是 PDC 模拟器，Azure AD Connect 将仍访问 PDC 以用于密码写回。
- 已在你的租户中配置和启用了密码重置。有关详细信息，请参阅[让用户重置其 Azure AD 密码](#enable-users-to-reset-their-azure-ad-passwords)
- 你拥有至少一个管理员帐户和一个测试用户帐户，还有一个可用于测试此功能的 Azure AD Premium 许可证。有关详细信息，请参阅 [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。

  > [AZURE.NOTE]
  确保用于启用密码回写的管理员帐户是云管理员帐户（在 Azure AD 中创建），而不是联合帐户（在本地 AD 中创建并同步到 Azure AD 中）。
  >
  >
- 拥有运行 Windows Server 2008、Windows Server 2008 R2、Windows Server 2012 或 Windows Server 2012 R2 并装有最新服务包的单林或多林 AD 本地部署。

  > [AZURE.NOTE]
  如果你运行的是旧版 Windows Server 2008 或 2008 R2，则你仍然可以使用此功能，但在云中实施本地 AD 密码策略之前，需要先[下载并安装 KB 2386717](https://support.microsoft.com/zh-cn/kb/2386717)。
  >
  >
- 已安装了 Azure AD 同步工具，并已准备好 AD 环境，可随时同步到云。有关详细信息，请参阅[在云中使用本地标识基础结构](/documentation/articles/active-directory-aadconnect/)。

  > [AZURE.NOTE]
  测试密码写回之前，请确保完成完整导入，并在 Azure AD Connect 中从本地 AD 与 Azure AD 执行完全同步。
  >
  >
- 如果使用 Azure AD Sync 或 Azure AD Connect，则需要打开 **TCP 443** 出站端口（在某些情况下需要打开 **TCP 9350-9354**）。有关详细信息，请参阅[步骤 3：配置防火墙](#step-3-configure-your-firewall)。不再支持对此方案使用 DirSync。如果你仍在使用 DirSync，请升级到最新版本的 Azure AD Connect，然后再部署密码写回。

  > [AZURE.NOTE]
  我们强烈建议使用 Azure AD Sync 或 DirSync 工具的任何用户升级到最新版本的 Azure AD Connect，以确保拥有最佳使用体验和发行的新功能。
  >
  >

### 步骤 1：下载最新版本的 Azure AD Connect
Azure AD Connect 发行版或版本号为 **1.0.0419.0911** 或更高的 Azure AD Sync 工具中提供了密码写回功能。Azure AD Connect 发行版或版本号为 **1.0.0485.0222** 或更高的 Azure AD Sync 工具中提供了支持自动帐户解锁的密码写回功能。如果你运行的是较旧版本，请在继续操作之前至少升级到此版本。[单击此处下载最新版本的 Azure AD Connect](/documentation/articles/active-directory-aadconnect/#install-azure-ad-connect/)。

#### 查看 Azure AD Sync 的版本
1. 导航到 **%ProgramFiles%\\Azure Active Directory Sync**。
2. 找到 **ConfigWizard.exe** 可执行文件。
3. 右键单击该可执行文件，然后从上下文菜单中选择“属性”选项。
4. 单击“详细信息”选项卡。
5. 找到“文件版本”字段。

   ![][021]  


如果此版本号大于或等于 **1.0.0419.0911**，或者想要安装 Azure AD Connect，则可以跳至[步骤 2：通过 UI 或 PowerShell 在 Azure AD Connect 中启用密码写回并验证](#step-2-enable-password-writeback-in-azure-ad-connect)。

> [AZURE.NOTE]
如果是首次安装 Azure AD Connect 工具，建议遵循一些最佳实践，为目录同步准备好环境。在安装 Azure AD Connect 工具之前，必须在 [Office 365 管理门户](https://portal.partner.microsoftonline.cn)或 [Azure 经典管理门户](https://manage.windowsazure.cn)中激活目录同步。有关详细信息，请参阅[管理 Azure AD Connect](/documentation/articles/active-directory-aadconnect-whats-next/)。
>
>

### 步骤 2：在 Azure AD Connect 中启用密码写回
现在已下载 Azure AD Connect 工具，为启用密码写回做好了准备。可通过两种方式之一启用密码写回。可以在 Azure AD Connect 设置向导的可选功能屏幕中启用密码写回，也可以通过 Windows PowerShell 启用。

#### 在 Azure AD Connect 设置向导中启用密码写回
1. 在你的**目录同步计算机**上，打开 **Azure AD Connect** 配置向导。
2. 单击各个步骤，直到出现**可选功能**配置屏幕。
3. 选中“密码写回”选项。

	![][022]  

4. 完成向导，最后一页将汇总更改并包括密码写回配置更改。

> [AZURE.NOTE]
你随时可以禁用密码写回，方法是重新运行此向导并取消选择该功能，或者在 [Azure 经典管理门户](https://manage.windowsazure.cn)中目录的“配置”选项卡的“用户密码重置策略”部分中，将“密码写回到本地目录”设置设为“否”。有关自定义密码重置体验的详细信息，请查看[自定义：Azure AD 密码管理](/documentation/articles/active-directory-passwords-customize/)。
>
>

#### 使用 Windows PowerShell 启用密码写回
1. 在你的**目录同步计算机**上，打开一个新的**权限提升的 Windows PowerShell 窗口**。
2. 如果尚未加载该模块，请键入 `import-module ADSync` 命令以将 Azure AD Connect cmdlet 载入当前会话。
3. 通过运行 `Get-ADSyncConnector` cmdlet 并将结果存储在 `$aadConnectorName` 中，以获取系统中 Azure AD 连接器的列表，例如 `$connectors = Get-ADSyncConnector|where-object {$\_.name -like "*AAD"}`
4. 运行以下 cmdlet 获得当前连接器的当前写回状态：`Get-ADSyncAADPasswordResetConfiguration -Connector $aadConnectorName.name`
5. 运行以下 cmdlet 启用密码写回：`Set-ADSyncAADPasswordResetConfiguration -Connector $aadConnectorName.name -Enable $true`

> [AZURE.NOTE]
如果系统提示你输入凭据，请确保你为 AzureADCredential 指定的管理员帐户是**云管理员帐户（在 Azure AD 中创建）**，而不是联合帐户（在本地 AD 中创建并同步到 Azure AD 中）。
>
> [AZURE.NOTE]
可以通过 PowerShell 禁用密码写回，方法是重复上面的相同的步骤，不同的是在步骤中传递 `$false`，或者在 [Azure 经典管理门户](https://manage.windowsazure.cn)的目录的“配置”选项卡的“用户密码重置策略”部分中，将“密码写回到本地目录”设置设为“否”。
>
>

#### 验证配置是否成功
配置成功之后，你将在 Windows PowerShell 窗口中看到消息“密码重置写回已启用”，或者在配置 UI 中看到成功消息。

你可以打开“事件查看器”，导航到应用程序事件日志，然后从源 **PasswordResetService** 查找事件 **31005 - OnboardingEventSuccess**，以验证服务是否正确安装。

  ![][023]  


### 步骤 3：配置防火墙
启用密码写回后，需要确保运行 Azure AD Connect 的计算机能够访问 Microsoft 云服务以接收密码写回请求。此步骤包括更新网络设备（代理服务器、防火墙等）中的连接规则，以允许通过特定网络端口与某些 [Microsoft 自有的 URL 和 IP 地址](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2?ui=en-us&rs=en-us&ad=US)进行出站连接。这些更改可能会根据 Azure AD Connect 工具的版本而有所不同。有关更多上下文，可阅读有关[密码写回的工作原理](/documentation/articles/active-directory-passwords-learn-more/#how-password-writeback-works/)和[密码写回安全模型](/documentation/articles/active-directory-passwords-learn-more/#password-writeback-security-model/)的详细信息。

#### 为什么需要执行此操作？

为了使密码写回正常运行，运行 Azure AD Connect 的计算机需要能够建立到 **.servicebus.chinacloudapi.cn* 的出站 HTTPS 连接以及 Azure 使用的特定 IP 地址，如 [Azure 数据中 IP 范围列表](https://www.microsoft.com/en-us/download/details.aspx?id=41653)中所述。

对于 Azure AD Connect 工具 **1.1.443.0**（最新）及更高版本：

- 最新版的 Azure AD Connect 工具将需要以下网站的**出站 HTTPS** 访问权限：
    - *passwordreset.microsoftonline.com*
    - *servicebus.chinacloudapi.cn*

对于 Azure AD Connect 工具版本 **1.0.8667.0** 到 **1.1.380.0**：

- **选项 1：**允许通过端口 443 使用 URL 或 IP 地址的所有出站 HTTPS 连接。
	- 何时使用此选项：
		- 如果希望最简单的配置不需要要随着 Azure 数据中心 IP 范围的更改而更新，请使用此选项。
	- 所需的步骤：
		- 允许通过端口 443 使用 URL 或 IP 地址建立所有出站 HTTPS 连接。<br><br>
- **选项 2：**允许与特定 IP 范围和 URL 建立出站 HTTPS 连接
	- 何时使用此选项：
		- 如果处于受限网络环境中，或者不愿意允许出站连接，请使用此选项。
		- 在此配置中，要使密码写回继续工作，需要确保网络设备使用 Azure 数据中心 IP 范围列表中的最新 IP 每周更新。这些 IP 范围可用作每周三（太平洋时间）更新的 XML 文件，并在下一个星期一（太平洋时间）生效。
	- 所需的步骤：
		- 允许与 *.servicebus.chinacloudapi.cn 建立所有出站 HTTPS 连接
		- 允许与 Azure 数据中心 IP 范围列表中的所有 IP 建立所有出站 HTTPS 连接，并使此配置每周更新。

> [AZURE.NOTE]
注意如果按照上述说明配置了密码写回，且在 Azure AD Connect 事件日志中没有看到任何错误，但是在测试时遇到连接错误，则可能是环境中的网络设备阻止了连接到 IP 地址的 HTTPS 连接。例如，虽然允许连接到 *https://*.servicebus.chinacloudapi.cn*，但可能会阻止连接到该范围内的特定 IP 地址。若要解决此问题，需要配置网络环境以允许通过端口 443 连接到任何 URL 或 IP 地址的所有出站 HTTPS 连接（上述的选项 1），或者与网络团队协作，明确允许 HTTPS 连接到特定 IP 地址（上述的选项 2）。

**对于较旧版本：**

- 允许通过端口 443、9350-9354 和端口 5671 建立出站 TCP 连接
- 允许与 *https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/* 建立出站连接

> [AZURE.NOTE]
如果使用的是 1.0.8667.0 之前的 Azure AD Connect 版本，Microsoft 强烈建议升级到[最新版本的 Azure AD Connect](https://www.microsoft.com/zh-cn/download/details.aspx?id=47594)，该版本包括许多写回网络增强功能，可简化配置。

配置网络设备后，重新启动运行 Azure AD Connect 工具的计算机。

#### Azure AD Connect（1.1.443.0 及更高版本）上的空闲连接
Azure AD Connect 工具会定期将 ping/keepalive 发送到 ServiceBus 终结点，确保连接始终处于活动状态。如果该工具检测到丢失的连接过多，则会自动增加发送到终结点的 ping 的频率。最低的“ping 间隔”将降至每 60 秒 ping 1 次，但是，**我们强烈建议将代理/防火墙设置为允许空闲连接保持至少 2-3 分钟**。*对于较旧的版本，我们建议让空闲连接保持 4 分钟或更长时间。

### 步骤 4：设置适当的 Active Directory 权限
对于包含密码将要重置的用户的每个林，如果 X 是（初始配置期间）在配置向导中为该林指定的帐户，则必须为 X 授予对 `lockoutTime` 的“重置密码”、“更改密码”和“写入”权限，对 `pwdLastSet` 的“写入”权限，以及对该林中每个域的根对象的扩展权限，或者对要添加到 SSPR 范围中的用户 OU 的扩展权限。如果要将重置权限限定于特定的用户对象集，而针对域的根目录这样做是不可接受的，则可使用后一选项。应当将权限标记为“由所有用户对象继承”。

如果你不确定上面指的是哪个帐户，请打开 Azure Active Directory Connect 配置 UI，然后单击“查看解决方案”选项。在下面的屏幕截图中，需要将权限添加到的帐户标有红色下划线。

**<font color="red">请确保为系统中每个林内的每个域设置此权限，否则密码写回功能将无法正常工作。</font>**

  ![][032]

  设置这些权限将允许每个林的 MA 服务帐户代表该林中的用户帐户管理密码。如果你忘记分配这些权限，即使写回看上去配置正确，用户在尝试从云中管理本地密码时也会遇到错误。下面是如何使用 **Active Directory 用户和计算机**管理单元实现此目的的详细步骤：

> [AZURE.NOTE]
将这些权限复制到目录中的所有对象可能需要一小时。
>
>

#### 设置正确的权限以执行写回
1. 使用具有适当域管理权限的帐户打开“Active Directory 用户和计算机”。
2. 在“视图菜单”选项中，确保打开“高级功能”。
3. 在左面板中，右键单击表示根级域的对象。
4. 单击“安全性”选项卡。
5. 然后单击“高级”。

   ![][024]
6. 在“权限”选项卡上，单击“添加”。

   ![][025]
7. 选择要对其授予权限的帐户（该帐户与为此林设置同步时指定的帐户相同）。
8. 在顶部的下拉列表中，选择“下级用户对象”。
9. 在出现的“权限条目”中，选中针对 `lockoutTime` 的“重置密码”、“更改密码”和“写入权限”框，以及针对 `pwdLastSet` 的“写入权限”框。

   ![][026] 
   ![][027] 
   ![][028]
10. 然后，单击所有打开对话框中的“应用/确定”。

### 步骤 5：以用户身份重置 AD 密码
现在密码写回功能已启用，你可以通过重置其帐户已同步到云租户的用户的密码，测试该功能是否能够正常工作。

#### 验证密码写回是否正常工作
1. 导航到 [http://passwordreset.microsoftonline.com](https://passwordreset.microsoftonline.com) 或转到任何组织 ID 登录屏幕并单击“无法访问你的帐户?”链接。

	![][029]  

2. 你现在应该看到一个新页面，它请求提供你要为其重置密码的用户 ID。输入测试用户 ID，并继续执行密码重置流程。
3. 重置密码后，你将看到与此类似的屏幕。这意味着你已成功重置了本地和/或云目录中的密码。

	![][030]  

4. 若要验证操作是否成功或要诊断任何错误，请转到**目录同步计算机**，打开“事件查看器”，导航到**应用程序事件日志**，并从测试用户的源 **PasswordResetService** 查找事件 **31002 - PasswordResetSuccess**。

	![][031]  




## 后续步骤
以下是所有 Azure AD 密码重置文档页面的链接：

- **你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
- [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
- [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
- [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
- [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
- [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
- [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节

[001]: ./media/active-directory-passwords-getting-started/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-getting-started/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-getting-started/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-getting-started/004.jpg "Image_004.jpg"
[005]: ./media/active-directory-passwords-getting-started/005.jpg "Image_005.jpg"
[006]: ./media/active-directory-passwords-getting-started/006.jpg "Image_006.jpg"
[007]: ./media/active-directory-passwords-getting-started/007.jpg "Image_007.jpg"
[008]: ./media/active-directory-passwords-getting-started/008.jpg "Image_008.jpg"
[009]: ./media/active-directory-passwords-getting-started/009.jpg "Image_009.jpg"
[010]: ./media/active-directory-passwords-getting-started/010.jpg "Image_010.jpg"
[011]: ./media/active-directory-passwords-getting-started/011.jpg "Image_011.jpg"
[012]: ./media/active-directory-passwords-getting-started/012.jpg "Image_012.jpg"
[013]: ./media/active-directory-passwords-getting-started/013.jpg "Image_013.jpg"
[014]: ./media/active-directory-passwords-getting-started/014.jpg "Image_014.jpg"
[015]: ./media/active-directory-passwords-getting-started/015.jpg "Image_015.jpg"
[016]: ./media/active-directory-passwords-getting-started/016.jpg "Image_016.jpg"
[017]: ./media/active-directory-passwords-getting-started/017.jpg "Image_017.jpg"
[018]: ./media/active-directory-passwords-getting-started/018.jpg "Image_018.jpg"
[019]: ./media/active-directory-passwords-getting-started/019.jpg "Image_019.jpg"
[020]: ./media/active-directory-passwords-getting-started/020.jpg "Image_020.jpg"
[021]: ./media/active-directory-passwords-getting-started/021.jpg "Image_021.jpg"
[022]: ./media/active-directory-passwords-getting-started/022.jpg "Image_022.jpg"
[023]: ./media/active-directory-passwords-getting-started/023.jpg "Image_023.jpg"
[024]: ./media/active-directory-passwords-getting-started/024.jpg "Image_024.jpg"
[025]: ./media/active-directory-passwords-getting-started/025.jpg "Image_025.jpg"
[026]: ./media/active-directory-passwords-getting-started/026.jpg "Image_026.jpg"
[027]: ./media/active-directory-passwords-getting-started/027.jpg "Image_027.jpg"
[028]: ./media/active-directory-passwords-getting-started/028.jpg "Image_028.jpg"
[029]: ./media/active-directory-passwords-getting-started/029.jpg "Image_029.jpg"
[030]: ./media/active-directory-passwords-getting-started/030.jpg "Image_030.jpg"
[031]: ./media/active-directory-passwords-getting-started/031.jpg "Image_031.jpg"
[032]: ./media/active-directory-passwords-getting-started/032.jpg "Image_032.jpg"

<!---HONumber=Mooncake_0327_2017-->
