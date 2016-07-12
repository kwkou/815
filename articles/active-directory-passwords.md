<properties 
	pageTitle="Azure AD 密码重置 | Microsoft Azure"
	description="介绍 Azure AD 中的密码管理功能，包括密码重置、更改、密码管理报告，以及将密码写回到本地 Active Directory。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="02/16/2016" 
	wacn.date="05/13/2016"/>


# 用户和管理员的 Azure AD 密码重置

  >[AZURE.IMPORTANT] 你是因为想要重置 Azure 或 O365 密码而来到这里吗？ 如果是这样，请[跳到此部分](#users-how-to-manage-your-own-password)。
  
利用自助服务降低成本和节省人力，一直以来都是世界各地 IT 部门追求的主要目标。事实上，市场中充斥着各种产品，让你能够从云或本地管理本地组、密码或用户配置文件。独树一帜的 Azure AD 提供一些现今市场上最容易使用且最强大的自助服务功能。

**Azure AD 密码管理**提供一组让用户随时随地从任何设备管理密码的功能，同时还能遵循你定义的安全策略。

## <a name="users-how-to-manage-your-own-password"></a>用户：如何管理自己的密码
如果你是组织中使用 Office 365 或 Microsoft 帐户访问工作资源的用户（不是管理员），请单击以下链接，了解如何解决常见密码问题。

| 主题 | |
| --------- | --------- |
| 我要注册密码重置 | [如何注册密码重置](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-register-for-password-reset) |
| 我要从 O365 更改密码 | [如何从 Office365 更改密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-change-your-password-from-o365) |
| 我要从 myapps.microsoft.com 更改密码 | [如何从访问面板更改密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-change-your-password-from-the-access-panel) |
| 我忘记了密码，想要重置密码 | [如何重置密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password) |
| 我无法登录，想要解锁我的帐户 | [如何解锁本地帐户](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-unlock-your-account) |
| 我希望你们帮助我排查密码重置失败的原因 | [常见问题及其解决方法](/documentation/articles/active-directory-passwords-update-your-own-password/#common-problems-and-their-solutions) |

##管理员：了解如何开始使用 Azure AD 密码重置
如果你是一名管理员，想要启用 Azure AD 密码重置或只是想详细了解此功能，请使用以下链接阅读你感兴趣的内容。

| 主题 | |
| --------- | --------- |
| 支持的方案 | [Azure AD 密码重置有何用途？](#what-is-possible-with-azure-ad-password-reset) |
| 为何要使用它？ | [为何使用 Azure AD 密码重置？](#why-use-azure-ad-password-reset) |
| 定价和可用性 | [定价和可用性](#pricing-and-availability) |
| 启用密码重置 | [为用户启用密码重置](#enable-password-reset-for-your-users) |
| 自定义密码重置的工作方式 | [自定义密码重置行为](#customize-password-reset-behavior) |
| 为用户启用密码重置 | [将用户配置为使用密码重置](#configure-your-users-to-use-password-reset) |
| 查看报告 | [使用集成式报告查看密码重置活动](#view-password-reset-activity-with-integrated-reports) |
| 重置用户密码 | [管理用户的密码](#manage-your-users-passwords) |
| 设置组织的密码策略 | [设置密码策略](#set-password-policies) |
| 排查问题 | [排查问题](#troubleshoot-a-problem) |
| 常见问题 | [阅读常见问题](#read-a-faq) |
| 技术详细信息 | [了解技术详细信息](#understand-the-technical-details) |
| 新发布的功能 | [最新服务更新](#recent-service-updates) |
| 其他文档的链接 | [密码重置文档的链接](#links-to-password-reset-documentation) |

### Azure AD 密码重置有何用途？
以下是使用 Azure AD 密码管理功能可实现的某些目的。

- **自助密码更改**：可让最终用户或管理员就可以更改过期的或未过期的密码，而无需请求管理员或帮助台提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或帮助台提供支持。自助密码重置功能需要 Azure AD 高级或基本版。有关详细信息，请参阅“Azure Active Directory 版本”。
- **管理员启动的密码重置**：管理员可以通过 [Azure 经典管理门户](https://manage.windowsazure.cn)重置某个最终用户的或其他管理员的密码。
- **密码管理活动报告**：管理员可以深入了解发生在其组织中的密码重置和注册活动。 
- **密码写回**：从云管理本地密码，因此，所有上述方案都可以由经过联合身份验证的或密码同步的用户本人或其代表来执行。密码写回功能需要 Azure AD Premium。有关详细信息，请参阅“Azure AD Premium 入门”。

## <a name="why-use-azure-ad-password-reset"></a>为何使用 Azure AD 密码重置？
以下是应该使用 Azure AD 密码管理功能的某些原因

- **降低成本** - 支持人员辅助的密码重置通常占组织 IT 部门 20% 的费用
- **改善用户体验** - 用户不希望每次忘记密码时，都必须致电技术支持并花费大量时间在通话上
- **减少技术支持人力** - 密码管理是大部分组织设立技术支持的最重要因素
- **实现移动** - 用户可随时随地重置密码

### <a name="pricing-and-availability"></a>定价和可用性
Azure AD 密码重置根据用户使用的订阅提供 3 个级别：

- **Azure AD 免费版** - 仅限云的管理员可以重置自己的密码
- **Azure AD 基本版或任何付费 O365 订阅** - 仅限云的用户及仅限云的管理员可以重置自己的密码
- **Azure AD 高级版** - 任何用户或管理员（包括仅限云、联合或密码同步的用户）可以重置自己的密码（需要[启用密码写回](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords)）

有关关于 Azure AD 高级版或基本版定价的详细信息，请访问 [Active Directory 定价详细信息](/pricing/details/active-directory/)页。

## <a name="enable-password-reset-for-your-users"></a>为用户启用密码重置
| 主题 | |
| --------- | --------- |
| 如何为云用户启用密码重置？ | [让用户重置其云 Azure Active Directory 密码](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-their-azure-ad-passwords) |
| 如何为本地用户启用密码重置和更改？ | [让用户重置或更改其本地 Active Directory 密码](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords) |
| 如何将密码重置的范围限定为一组特定的用户？ | [将密码重置限定于特定用户](/documentation/articles/active-directory-passwords-customize/#restrict-access-to-password-reset) |
| 如何测试云密码重置？ | [以用户身份重置 Azure AD 密码](/documentation/articles/active-directory-passwords-getting-started/#step-3-reset-your-azure-ad-password-as-a-user) |
| 如何测试本地密码重置？ | [以用户身份重置本地 AD 密码](/documentation/articles/active-directory-passwords-getting-started/#step-5-reset-your-ad-password-as-a-user) |
| 以后如何禁用密码重置？ | [设置：为用户启用了密码重置](/documentation/articles/active-directory-passwords-customize/#users-enabled-for-password-reset) |


## <a name="customize-password-reset-behavior"></a>自定义密码重置行为
| 主题 | |
| --------- | --------- |
| 如何更改支持的身份验证方法？ | [设置：用户可用的身份验证方法](/documentation/articles/active-directory-passwords-customize/#authentication-methods-available-to-users) |
| 如何更改所需身份验证方法的数量？ | [设置：所需身份验证方法的数量](/documentation/articles/active-directory-passwords-customize/#number-of-authentication-methods-required) |
| 如何设置自定义安全提问？ | [设置：自定义安全提问](/documentation/articles/active-directory-passwords-customize/#custom-security-questions) |
| 如何设置预先编写的本地化安全提问？ | [设置：基于知识的安全提问](/documentation/articles/active-directory-passwords-customize/#knowledge-based-security-questions) |
| 如何更改所需的安全提问数？ | [设置：注册或重置的安全提问数](/documentation/articles/active-directory-passwords-customize/#number-of-questions-required-to-register) |
| 如何自定义用户联系管理员的方式？ | [设置：自定义“联系管理员”链接](/documentation/articles/active-directory-passwords-customize/#customize-the-contact-your-administrator-link) |
| 如何让用户直接解锁 AD 帐户而不必重置密码？ | [设置：让用户直接解锁 AD 帐户而不必重置密码](/documentation/articles/active-directory-passwords-customize/#allow-users-to-unlock-accounts-without-resetting-their-password) |
| 如何为用户启用密码重置通知？ | [设置：在用户的密码重置时通知用户](/documentation/articles/active-directory-passwords-customize/#notify-users-and-admins-when-their-own-password-has-been-reset) |
| 如何为管理员启用密码重置通知？ | [设置：在管理员重置其密码时通知其他管理员](/documentation/articles/active-directory-passwords-customize/#notify-admins-when-other-admins-reset-their-own-passwords) |
| 如何自定义密码重置的外观？ | [设置：公司名称、品牌和徽标](/documentation/articles/active-directory-passwords-customize/#password-managment-look-and-feel) |


## <a name="configure-your-users-to-use-password-reset"></a>将用户配置为使用密码重置
| 主题 | |
| --------- | --------- |
| 如何知道某个帐户是否已设置密码重置？ | [如何为密码重置配置帐户？](/documentation/articles/active-directory-passwords-best-practices/#what-makes-an-account-configured) |
| 如何为用户配置密码重置？ | [为用户填充密码重置身份验证数据的方式](/documentation/articles/active-directory-passwords-best-practices/#ways-to-populate-authentication-data) |
| 如何手动上载用户的数据？ | [自行上载密码重置数据](/documentation/articles/active-directory-passwords-best-practices/#uploading-data-yourself) |
| 如何使用 PowerShell 读取或设置用户的数据？ | [如何访问用户的密码重置数据](/documentation/articles/active-directory-passwords-learn-more/#how-to-access-password-reset-data-for-your-users) |
| 如何从本地同步密码重置数据？ | [密码重置使用哪些数据？](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset) |
| 如何使用电子邮件营销活动促使我的用户注册并使用密码重置？ | [基于电子邮件的密码重置推广](/documentation/articles/active-directory-passwords-best-practices/#email-based-rollout) |
| 如何强制用户在登录时注册？ | [基于强制注册的密码重置推广](/documentation/articles/active-directory-passwords-customize/#require-users-to-register-when-signing-in) |
| 如何强制用户定期重新确认注册？ | [设置：用户必须在几天后重新确认其身份验证数据](/documentation/articles/active-directory-passwords-customize/#number-of-days-before-users-must-confirm-their-contact-data) |
| 将密码重置传达给最终用户的最佳做法是什么？ | [创建自己的密码门户供用户使用](/documentation/articles/active-directory-passwords-best-practices/#creating-your-own-password-portal) |


## <a name="view-password-reset-activity-with-integrated-reports"></a>使用集成式报告查看密码重置活动
| 主题 | |
| --------- | --------- |
| 可以在何处查看密码重置报告？ | [密码管理报告概述](/documentation/articles/active-directory-passwords-get-insights/#overview-of-password-management-reports) |
| 可以在何处查看用户在我的组织中使用密码重置的方式？ | [查看密码重置活动](/documentation/articles/active-directory-passwords-get-insights/#view-password-reset-activity) |
| 可以在何处查看有多少用户注册及其注册的项目？ | [查看密码重置注册活动](/documentation/articles/active-directory-passwords-get-insights/#view-password-reset-registration-activity) |
| 通过 API 可获取哪种密码重置报告信息？ | [报告 API 提供的密码重置和注册事件](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-reports-and-events-preview#SsprActivityEvent) |


## <a name="manage-your-users-passwords"></a>管理用户的密码
| 主题 | |
| --------- | --------- |
| 如何从 O365 经典管理门户重置用户的密码？ | [在 Office 365 中重置用户的密码](https://support.office.com/article/Reset-a-user-s-password-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C) |
| 如何使用 PowerShell 重置用户的密码？ | [使用 Set-MsolUserPassword 重置用户的密码](https://msdn.microsoft.com/library/azure/dn194140.aspx) |


## <a name="set-password-policies"></a>设置密码策略
| 主题 | |
| --------- | --------- |
| 如何从 Office 365 设置组织密码过期策略？ | [设置密码过期策略](https://support.office.com/article/Set-a-user-s-password-expiration-policy-0f54736f-eb22-414c-8273-498a0918678f) |
| 如何使用 PowerShell 将特定用户的密码设置为永不过期？ | [使用 PowerShell 将单个用户的密码设置为永不过期](https://support.office.com/article/Set-an-individual-user-s-password-to-never-expire-f493e3af-e1d8-4668-9211-230c245a0466) |
| 如何使用 PowerShell 确定用户的密码是否设置为永不过期 | [使用 PowerShell 检查单个用户的密码过期状态](https://support.office.com/article/Set-an-individual-user-s-password-to-never-expire-f493e3af-e1d8-4668-9211-230c245a0466#__toc378845827) |


## <a name="troubleshoot-a-problem"></a>排查问题
| 主题 | |
| --------- | --------- |
| 我需要帮助时应向支持人员提供哪些信息？ | [你需要帮助时应包含的信息](/documentation/articles/active-directory-passwords-troubleshoot/#information-to-include-when-you-need-help) |
| 如何解决密码重置的问题 | [排查密码重置门户问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-the-password-reset-portal) |
| 如何解决密码写回问题 | [排查密码写回问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback) |
| 如何解决密码写回连接问题 | [排查密码写回连接问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-writeback-connectivity) |
| 如何解决密码重置配置问题 | [在 Azure 经典管理门户中排查密码重置配置问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-reset-configuration-in-the-azure-management-portal) |
| 如何解决密码重置报告问题 | [在 Azure 经典管理门户中排查密码管理报告问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-password-management-reports-in-the-azure-management-portal) |
| 如何解决密码重置注册问题 | [排查密码重置注册门户问题](/documentation/articles/active-directory-passwords-troubleshoot/#troubleshoot-the-password-reset-registration-portal) |
| 密码写回事件日志错误代码 | [密码写回事件日志错误代码](/documentation/articles/active-directory-passwords-troubleshoot/#password-writeback-event-log-error-codes) |


## <a name="read-a-faq"></a>阅读常见问题
| 主题 | |
| --------- | --------- |
| 我想要阅读关于密码重置注册的常见问题 | [密码重置注册常见问题](/documentation/articles/active-directory-passwords-faq/#password-reset-registration) |
| 我想要阅读关于密码重置的常见问题 | [密码重置常见问题](/documentation/articles/active-directory-passwords-faq/#password-reset) |
| 我想要阅读关于密码重置报告的常见问题 | [密码管理报告常见问题](/documentation/articles/active-directory-passwords-faq/#password-management-reports) |
| 我想要阅读关于密码写回的常见问题 | [密码写回常见问题](/documentation/articles/active-directory-passwords-faq/#password-writeback) |


## <a name="understand-the-technical-details"></a>了解技术详细信息

| 主题 | |
| --------- | --------- |
| 我想要了解什么是密码写回 | [密码写回概述](/documentation/articles/active-directory-passwords-learn-more/#password-writeback-overview) |
| 我想要了解密码写回的工作原理 | [密码写回的工作原理](/documentation/articles/active-directory-passwords-learn-more/#how-password-writeback-works) |
| 我想要了解密码写回支持的方案 | [密码写回支持的方案](/documentation/articles/active-directory-passwords-learn-more/#scenarios-supported-for-password-writeback) |
| 我想要了解密码写回如何受到保护 | [密码写回安全模型](/documentation/articles/active-directory-passwords-learn-more/#password-writeback-security-model) |
| 我想要了解密码重置门户的工作原理 | [密码重置门户的工作原理](/documentation/articles/active-directory-passwords-learn-more/#how-does-the-password-reset-portal-work) |
| 我想要了解密码重置使用的数据 | [密码重置使用哪些数据？](/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset) |

## <a name="recent-service-updates"></a>最新服务更新

####在登录 Office 365 应用时强制密码重置注册 - 2015 年 11 月

- 现在，在启用[强制注册](/documentation/articles/active-directory-passwords-customize/#require-users-to-register-when-signing-in)功能后，用户必须使用工作或学校帐户从他们登录的任何位置进行注册。这可以大幅提高许多组织登记密码重置的速度。我们发现，使用此新增功能的组织在短短 2 周内即可完成登记！

####支持在不重置密码的情况下解锁 Active Directory 帐户 - 2015 年 11 月

- 只解锁而不重置，是近来客服中心繁重的任务之一。事实上，许多组织有高达 70% 的密码重置预算都花在解锁帐户上。 为了满足此要求，现在你可以使用 Azure AD 密码重置启用特定功能，让用户直接解锁 AD 帐户，而不需要重置密码。请在此处了解如何启用此功能：[设置：让用户直接解锁 AD 帐户而不必重置密码](/documentation/articles/active-directory-passwords-customize/#allow-users-to-unlock-accounts-without-resetting-their-password)。

####注册页的可用性更新 - 2015 年 11 月

- 现在，在用户注册数据之后，他（她）只需单击“看起来不错”即可更新数据，而无需重新发送电子邮件或拨打电话。

####改进了密码写回的可靠性 - 2015 年 9 月

- 从 Azure AD Connect 九月版开始，密码写回代理现在将更积极地重试连接。另外，该版本还提供其他更稳健的故障转移功能。

####用于检索密码重置报告数据的 API - 2015 年 8 月

- 现在，可以直接通过 [Azure AD 报告和事件 API](https://msdn.microsoft.com/library/azure/mt126081.aspx#BKMK_SsprActivityEvent) 来检索密码重置报告幕后的数据。

####支持在云域加入期间执行 Azure AD 密码重置 - 2015 年 8 月

- 现在，任何云用户可以在云域加入登记体验过程中，直接从 Windows 10 登录屏幕重置其密码。请注意，此功能尚未在 Windows 10 登录屏幕上公开。

####在登录 Azure 和联合应用时强制密码重置注册 - 2015 年 7 月

- 除了在登录 myapps.microsoft.com 时强制注册以外，我们现在支持在登录 Azure 经典管理门户和任何联合单一登录应用程序期间强制注册

####安全提问本地化支持 - 2015 年 5 月

- 现在，你可以在配置密码重置的安全问题时，选择以完整 O365 语言集本地化的预定义安全问题。

####密码重置期间的帐户解锁支持 - 2015 年 6 月

- 如果你正在使用密码写回并在帐户已锁定时重置密码，我们将自动解锁你的 Active Directory 帐户！

####带品牌的 SSPR 注册 - 2015 年 4 月

- 密码重置注册页面现在已加上你的公司徽标！

####安全提问 - 2015 年 3 月

- 我们已将安全问题发布到 GA！

####帐户解锁 - 2015 年 3 月

- 用户现在可以在重置密码后解锁帐户

## 即将支持

下面是我们目前正在完善的一些优异功能！

**支持在登录期间提醒用户更新其注册的数据** - 正在开发

- 目前，我们支持在访问 myapps.microsoft.com 时提醒用户更新其注册的数据，但我们正努力实现对所有登录使用此功能。

## <a name="links-to-password-reset-documentation"></a>密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**重置自己的密码**](/documentation/articles/active-directory-passwords-update-your-own-password/) - 了解如何以系统用户的身份重置或更改自己的密码
* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights/) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节
<!---HONumber=Mooncake_0620_2016-->