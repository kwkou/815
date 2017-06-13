<properties
    pageTitle="策略：Azure AD SSPR | Azure"
    description="Azure AD 自助密码重置策略选项"
    services="active-directory"
    keywords="Active Directory 密码管理, 密码管理, Azure AD 自助密码重置"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/26/2017"
    wacn.date="06/12/2017"
    ms.author="joflore"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="926c9504ebe2e14e51bd3230deaa794a5db5a7da"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="password-policies-and-restrictions-in-azure-active-directory"></a>Azure Active Directory 中的密码策略和限制

本文介绍与 Azure AD 租户中存储的用户帐户关联的密码策略和复杂性要求。


## <a name="userprincipalname-policies-that-apply-to-all-user-accounts"></a>适用于所有用户帐户的 UserPrincipalName 策略

需登录到 Azure AD 的每个用户帐户都必须有唯一的与其帐户关联的用户主体名称 (UPN) 属性值。 下表概括了既适用于同步到云的本地 Active Directory 用户帐户，又适用于仅限云的用户帐户的策略。

| 属性 | UserPrincipalName 要求 |
| --- | --- |
| 允许的字符 |<ul> <li>A - Z</li> <li>a - z</li><li>0 - 9</li> <li> 。 - \_ ! \# ^ \~</li></ul> |
| 不允许的字符 |<ul> <li>任何不分隔用户名和域的“@”字符。</li> <li>不能紧靠在“@”符号前面添加句点字符“.”</li></ul> |
| 长度约束 |<ul> <li>总长度不能超过 113 个字符</li><li>“@”符号前为 64 个字符</li><li>“@”符号后为 48 个字符</li></ul> |

## <a name="password-policies-that-apply-only-to-cloud-user-accounts"></a>仅适用于云用户帐户的密码策略

下表描述了可用于在 Azure AD 中创建和管理的用户帐户的密码策略设置。

| 属性 | 要求 |
| --- | --- |
| 允许的字符 |<ul><li>A - Z</li><li>a - z</li><li>0 - 9</li> <li>@ # $ % ^ & * - _ ! + = [ ] { } &#124; \ : ‘ , . ? / ` ~ “ ( ) ;</li></ul> |
| 不允许的字符 |<ul><li>Unicode 字符</li><li>空格</li><li> **仅限强密码**：不能紧靠在“@”符号前面添加点字符“.”</li></ul> |
| 密码限制 |<ul><li>最少 8 个字符，最多 16 个字符</li><li>**仅限强密码**：需满足以下 4 条中的 3 条：<ul><li>小写字符</li><li>大写字符</li><li>数字 (0-9)</li><li>符号（请参阅上面的密码限制）</li></ul></li></ul> |
| 密码过期期限 |<ul><li>默认值：**90** 天 </li><li>可通过 Windows PowerShell 的 Azure Active Directory 模块中的 Set-MsolPasswordPolicy cmdlet 来配置该值。</li></ul> |
| 密码过期通知 |<ul><li>默认值：**14** 天（密码到期前）</li><li>可使用 Set-MsolPasswordPolicy cmdlet 配置值。</li></ul> |
| 密码过期 |<ul><li>默认值：**false** 天（表示已启用密码到期） </li><li>可以使用 Set-MsolUser cmdlet 为单个用户帐户配置该值。 </li></ul> |
| 密码 **更改** 历史记录 |**更改**密码后将**无法**再次使用上一个密码。 |
| 密码 **重置** 历史记录 | 由于忘记密码而**重置**后，**可能**还可再次使用上一个密码。 |
| 帐户锁定 |10 次登录尝试失败（错误密码）之后，用户会被锁定一分钟。 后续的错误登录尝试会增加用户被锁定的时间。 |

## <a name="set-password-expiration-policies-in-azure-active-directory"></a>在 Azure Active Directory 中设置密码过期策略

Microsoft 云服务的全局管理员可使用用于 Windows PowerShell 的 Azure Active Directory 模块将用户密码设置为永不过期。 你还可以使用 Windows PowerShell cmdlet 删除永不过期配置，或者查看已将哪些用户密码设置为永不过期。 本指南适用于其他提供程序（如 Microsoft Intune 和 Office 365），这些提供程序也依赖于 Azure Active Directory 提供标识和目录服务。

> [AZURE.NOTE]
> 只能将未通过目录同步进行同步的用户帐户的密码配置为永不过期。 有关目录同步的详细信息，请参阅[将 AD 与 Azure AD 连接](/documentation/articles/active-directory-aadconnect/)。
>
>

## <a name="set-or-check-password-policies-using-powershell"></a>使用 PowerShell 设置或检查密码策略

若要开始，需要[下载并安装 Azure AD PowerShell 模块](https://msdn.microsoft.com/zh-cn/library/azure/jj151815.aspx#bkmk_installmodule)。 安装后，可以遵照以下步骤配置每个字段。

### <a name="how-to-check-expiration-policy-for-a-password"></a>如何检查密码过期策略
1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   - 若要查看单个用户的密码是否已设置为永不过期，请使用用户主体名称 (UPN)（例如 aprilr@contoso.partner.onmschina.cn）或要查看的用户的用户 ID 运行以下 cmdlet：`Get-MSOLUser -UserPrincipalName <user ID> | Select PasswordNeverExpires`
   - 若要查看所有用户的“密码永不过期”设置，请运行以下 cmdlet： `Get-MSOLUser | Select UserPrincipalName, PasswordNeverExpires`

### <a name="set-a-password-to-expire"></a>设置密码过期

1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   - 若要将某一个用户的密码设置为过期，请使用用户主体名称 (UPN) 或该用户的用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $false`
   - 若要将组织中所有用户的密码设置为过期，请使用以下 cmdlet：`Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $false`

### <a name="set-a-password-to-never-expire"></a>将密码设置为永不过期

1. 使用公司管理员凭据连接到 Windows PowerShell。
2. 执行以下命令之一：

   - 若要将某一个用户的密码设置为永不过期，请使用用户主体名称 (UPN) 或该用户的用户 ID 运行以下 cmdlet：`Set-MsolUser -UserPrincipalName <user ID> -PasswordNeverExpires $true`
   - 若要将组织中所有用户的密码设置为永不过期，请运行以下 cmdlet： `Get-MSOLUser | Set-MsolUser -PasswordNeverExpires $true`

<!---Update_Description: wording update -->