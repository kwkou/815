<properties
    pageTitle="在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户 | Azure"
    description="介绍如何在 Azure Active Directory 中添加用户或更改用户信息，包括外部用户和来宾用户。"
    services="active-directory"
    documentationcenter=""
    author="curtand"
    manager="femila"
    editor="" />
<tags
    ms.assetid="564a04ec-53c1-470b-9ab9-f3db57da0a89"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="05/14/2017"
    wacn.date="06/12/2017"
    ms.author="curtand"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="80082407786d50d82e78fc9aa43384c042c29299"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="add-users-from-other-directories-or-partner-companies-in-azure-active-directory"></a>在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户

本文介绍了如何从 Azure Active Directory 中的其他目录添加用户或添加合作伙伴公司中的用户。 有关添加组织中的新用户和添加具有 Microsoft 帐户的用户的信息，请参阅[将新用户添加到 Azure Active Directory](/documentation/articles/active-directory-create-users/)。 默认情况下，添加的用户没有管理员权限，但你随时可以向他们分配角色。

## <a name="add-a-user"></a>添加用户
1. 使用充当目录全局管理员的帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn) 。
2. 选择“Active Directory” ，然后打开你的目录。
3. 选择“用户”选项卡，然后在命令栏中选择“添加用户”。
4. 在“告诉我们有关此用户的信息”页上的“用户类型”下，选择下列其中一项：

    - **另一个 Azure AD 目录中的用户** - 将源自另一个 Azure AD 目录的用户帐户添加到你的目录。 仅当你也是另一目录的成员时，才可以选择该目录中的用户。

5. 在用户的“配置文件”页上，提供名字和姓氏、用户友好名称，并从“角色”列表中选择用户角色。 有关用户和管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。 指定是否要为用户**启用多重身份验证**。

6. 在“获取临时密码”页上，选择“创建”。

> [AZURE.IMPORTANT]
> 如果你所在的组织使用多个域，在添加用户帐户时你应知道以下问题：
><p>
> * 例如，若要跨域添加具有相同用户主体名称 (UPN) 的用户帐户，请**先**添加 geoffgrisso@contoso.partner.onmschina.cn，**再**添加 geoffgrisso@contoso.com。
> <p>* **不要**在添加 geoffgrisso@contoso.partner.onmschina.cn 之前添加 geoffgrisso@contoso.com。
>

如果更改身份与本地 Active Directory 服务同步的用户的信息，则无法更改 Azure 经典管理门户中的用户信息。 若要更改该用户信息，请使用本地 Active Directory 管理工具。

## <a name="add-external-users"></a>添加外部用户
还可以从所属的另一个 Azure AD 目录添加用户，或者通过上传 CSV 文件来添加合作伙伴公司的用户。 若要添加外部用户，请针对“用户类型”指定“另一个 Azure AD 目录中的用户”或“合作伙伴公司的用户”。

两种类型的用户均源自另一个目录，并且添加为 **外部用户**。 外部用户可与目录中的其他用户协作，而无需添加新帐户和凭据。 当外部用户登录时，系统会使用这些用户的主目录对其进行身份验证，这种身份验证适用于这些用户添加到的其他所有目录。

## <a name="external-user-management-and-limitations"></a>外部用户管理和限制
将另一个目录中的用户添加到你的目录时，该用户是你目录中的外部用户。 显示名称和用户名是从用户的主目录复制的，将用于目录中的外部用户。 此后，外部用户帐户的属性是完全独立的。 如果你对主目录中的用户进行属性更改，这些更改不会传播到你目录中的外部用户帐户。

这两个帐户之间的唯一联系是用户始终针对主目录或使用他们的 Microsoft 帐户进行身份验证。 这就是你看不到重置密码或为外部用户启用多重身份验证的选项的原因。 目前，主目录或 Microsoft 帐户的身份验证策略是用户登录时唯一进行评估的策略。

> [AZURE.NOTE]
> 你仍然可以禁用目录中的外部用户，这将阻止其访问你的目录。
>
>

如果在用户的主目录中将其删除或取消其 Microsoft 帐户，外部用户仍然存在于你的目录中。 但是，目录中的用户无法访问资源，因为他们无法使用主目录或 Microsoft 帐户进行身份验证。

### <a name="services-that-currently-support-access-by-azure-ad-external-users"></a>目前支持 Azure AD 外部用户访问的服务
- **Azure 经典管理门户**：允许身为多个目录的管理员的外部用户管理这些目录。
- **SharePoint Online**：如果启用外部共享，则允许外部用户访问 SharePoint Online 的已授权资源。
- **Dynamics CRM**：如果用户通过 PowerShell 获得许可，则允许外部用户访问 Dynamics CRM 中的已授权资源。
- **Dynamics AX**：如果用户通过 PowerShell 获得许可，则允许外部用户访问 Dynamics AX 中的已授权资源。 适用于 [Azure AD 外部用户](#known-limitations-of-azure-ad-external-users) 的限制同样适用于 Dynamics AX 中的外部用户。

## <a name="next-steps"></a>后续步骤
- [向 Azure Active Directory 添加新用户](/documentation/articles/active-directory-create-users/)
- [管理 Azure AD](/documentation/articles/active-directory-administer/)
- [在 Azure AD 中管理密码](/documentation/articles/active-directory-manage-passwords/)

<!---Update_Description: wording update -->