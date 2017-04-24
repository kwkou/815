<properties
    pageTitle="Azure Active Directory 密码重置 | Azure"
    description="介绍 Azure AD 中的密码管理功能，包括密码重置、更改、密码管理报告，以及将密码写回到本地 Active Directory。"
    services="active-directory"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila"
    editor="curtand" />
<tags
    ms.assetid="be6164fc-bae1-49df-af76-761329ba70a1"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/28/2017"
    wacn.date="04/11/2017"
    ms.author="joflore" />  


# IT 管理员的 Azure Active Directory 密码重置
> [AZURE.IMPORTANT]
**你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。
>
>

利用自助服务降低成本和节省人力，一直以来都是世界各地 IT 部门追求的主要目标。事实上，市场中充斥着各种产品，使得能够从云或本地管理本地组、密码或用户配置文件。Azure Active Directory (Azure AD) 因提供如今最容易使用且最强大的一些自助服务功能而从其他产品中脱颖而出。

**Azure AD 密码管理**提供一组让用户随时随地从任何设备管理密码的功能，同时还能遵循你定义的安全策略。

## 管理员：了解如何开始使用 Azure AD 密码重置
对于想要启用 Azure AD 密码重置或只是想详细了解此功能的管理员，可使用以下链接阅读感兴趣的内容。

| 主题 | |
| --- | --- |
| 支持的方案 |[Azure AD 密码重置有何用途？](#what-is-possible-with-azure-ad-password-reset) |
| 为何要使用它？ |[为何使用 Azure AD 密码重置？](#why-use-azure-ad-password-reset) |
| 定价和可用性 |[定价和可用性](#pricing-and-availability) |
| 重置用户的密码 |[管理用户的密码](#manage-your-users-passwords) |
| 设置组织的密码策略 |[设置密码策略](#set-password-policies) |
| 新发布的功能 |[最新服务更新](#recent-service-updates) |
| 其他文档的链接 |[密码重置文档的链接](#links-to-password-reset-documentation) |

### Azure AD 密码重置有何用途？ <a name="what-is-possible-with-azure-ad-password-reset"></a>
以下是使用 Azure AD 密码管理功能可实现的某些用途。

- **自助密码更改**：可让最终用户或管理员就可以更改过期的或未过期的密码，而无需请求管理员或帮助台提供支持。
- **自助密码重置**：最终用户或管理员可以自行重置密码，而无需请求管理员或帮助台提供支持。自助密码重置功能需要 Azure AD Premium 或 Basic。有关详细信息，请参阅“Azure Active Directory 版本”。
- **管理员启动的密码重置**：管理员可以通过 [Azure 管理门户](https://manage.windowsazure.cn)重置某个最终用户的或其他管理员的密码。
- **密码写回**：从云管理本地密码，因此，所有上述方案都可以由经过联合身份验证的或密码同步的用户本人或其代表来执行。密码写回功能需要 Azure AD Premium。有关详细信息，请参阅“Azure AD Premium 入门”。

### <a name="why-use-azure-ad-password-reset"></a>为何使用 Azure AD 密码重置？
以下是应该使用 Azure AD 密码管理功能的一些原因

- **降低成本** - 支持人员辅助的密码重置通常占组织 IT 部门 20% 的费用
- **改善用户体验** - 用户不希望每次忘记密码时，都必须致电技术支持并花费大量时间在通话上
- **减少技术支持人力** - 密码管理是大部分组织设立技术支持的最重要因素
- **实现移动** - 用户可随时随地重置密码

### <a name="pricing-and-availability"></a>定价和可用性
Azure AD 密码重置根据所拥有的订阅提供 3 个级别：

- **Azure AD 免费版** - 仅限云的管理员可以重置自己的密码
- **Azure AD 基本版或任何付费 O365 订阅** - 仅限云的用户及仅限云的管理员可以重置自己的密码
- **Azure AD Premium** — 任何用户或管理员（包括仅限云、联合或密码同步的用户）都可以重置自己的密码

有关关于 Azure AD 高级版或基本版定价的详细信息，请访问 [Active Directory 定价详细信息](/pricing/details/identity/)页。

## <a name="manage-your-users-passwords"></a>管理用户的密码
| 主题 | |
| --- | --- |
| 如何从 O365 管理门户重置用户的密码？ |[在 Office 365 中重置用户的密码](https://support.office.com/article/Reset-a-user-s-password-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C) |
| 如何使用 PowerShell 重置用户的密码？ |[使用 Set-MsolUserPassword 重置用户的密码](https://msdn.microsoft.com/zh-cn/library/azure/dn194140.aspx) |


## <a name="set-password-policies"></a>设置密码策略
| 主题 | |
| --- | --- |
| 如何从 Office 365 设置组织密码过期策略？ |[设置密码过期策略](https://support.office.com/article/Set-a-user-s-password-expiration-policy-0f54736f-eb22-414c-8273-498a0918678f) |
| 如何使用 PowerShell 将特定用户的密码设置为永不过期？ |[使用 PowerShell 将单个用户的密码设置为永不过期](https://support.office.com/article/Set-an-individual-user-s-password-to-never-expire-f493e3af-e1d8-4668-9211-230c245a0466) |
| 如何使用 PowerShell 确定用户的密码是否设置为永不过期 |[使用 PowerShell 检查单个用户的密码过期状态](https://support.office.com/article/Set-an-individual-user-s-password-to-never-expire-f493e3af-e1d8-4668-9211-230c245a0466#__toc378845827) |

## <a name="recent-service-updates"></a>最新服务更新
#### 在登录 Office 365 应用时强制密码重置注册 - 2015 年 11 月
- 现在，在启用强制注册功能后，用户需要使用工作或学校帐户从他们登录的任何位置进行注册。这可以大幅提高许多组织推行密码重置的速度。我们发现，使用此新增功能的大型组织在短短 2 周内便完成了推行工作！

#### 支持在不重置密码的情况下解锁本地 AD 帐户 - 2015 年 11 月
- 只解锁而不重置，是近来技术支持部门繁重的任务之一。事实上，许多组织有高达 70% 的密码重置预算都花在解锁帐户上。 为了满足此要求，现在可以使用 Azure AD 密码重置启用特定功能，让用户直接解锁本地 AD 帐户，而不需要重置密码。

#### 注册页的可用性更新 - 2015 年 10 月
- 现在，在用户注册数据之后，他（她）只需单击“看起来不错”即可更新数据，而无需重新发送电子邮件或拨打电话。

#### 改进了密码写回的可靠性 - 2015 年 9 月
- 从九月发布的 Azure AD Connect 开始，密码写回代理将更积极地重试连接，以及提供更稳健的故障转移功能。

#### 用于检索密码重置报告数据的 API - 2015 年 8 月
- 现在，可以直接通过 [Azure AD 报告和事件 API](https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/howto/azure-ad-reports-and-events-preview) 来检索密码重置报告幕后的数据。

#### 支持在云域加入期间执行 Azure AD 密码重置 - 2015 年 8 月
- 现在，任何云用户可以在云域加入载入体验过程中，直接从 Windows 10 登录屏幕重置其密码。请注意，此功能尚未在 Windows 10 登录屏幕上公开。

#### 在登录 Azure 和联合应用时强制密码重置注册 - 2015 年 7 月
- 除了在登录 myapps.microsoft.com 时强制注册以外，我们现在支持在登录 Azure 管理门户和任何联合单一登录应用程序期间强制注册

#### 安全问题本地化支持 - 2015 年 5 月
- 现在，可以在配置密码重置的安全问题时，选择以完整 O365 语言集本地化的预定义安全问题。

#### 密码重置期间的帐户解锁支持 - 2015 年 6 月
- 如果正在使用密码写回并在帐户已锁定时重置密码，我们将自动解锁此 Active Directory 帐户！

#### 安全问题 - 2015 年 3 月
- 我们已将安全问题发布到 GA！

#### 帐户解锁 - 2015 年 3 月
- 用户现在可以在重置密码后解锁帐户

## 即将推出
下面是我们目前正在完善的一些优异功能！

**支持在登录期间提醒用户更新其注册的数据** - 正在开发

- 目前，我们支持在访问 myapps.microsoft.com 时提醒用户更新其注册的数据，但我们正努力实现对所有登录使用此功能。

## 后续步骤 <a name="links-to-password-reset-documentation"></a>
以下是所有 Azure AD 密码重置文档页的链接：

- **你是否因登录时遇到问题而浏览至此？** 如果是这样，[可按以下方式更改和重置你的密码](/documentation/articles/active-directory-passwords-update-your-own-password/#how-to-reset-your-password/)。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->