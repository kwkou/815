<properties
	pageTitle="在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户 | Azure"
	description="介绍如何在 Azure Active Directory 中添加用户或更改用户信息，包括外部用户和来宾用户。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="03/31/2016"
	/documentation/articles//>

# 在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户

请添加用户以填充你的目录。本文介绍了如何从 Azure Active Directory 中的其他目录添加用户或添加合作伙伴公司中的用户。有关添加你组织中的新用户和添加具有 Microsoft 帐户的用户的信息，请参阅[将新用户添加到 Azure Active Directory](/documentation/articles/active-directory-create-users/)。默认情况下，添加的用户没有管理员权限，但你随时可以向他们分配角色。

## 添加用户

1. 使用充当目录全局管理员的帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 选择 **Active Directory**，然后打开你的目录。

3. 选择“用户”选项卡，然后在命令栏中选择“添加用户”。

4. 在“告诉我们有关此用户的信息”页上的“用户类型”下，选择下列其中一项：

	- **另一个 Azure AD 目录中的用户** - 将源自另一个 Azure AD 目录的用户帐户添加到你的目录。仅当你也是另一目录的成员时，才可以选择该目录中的用户。
	- **合作伙伴公司的用户** - 邀请并授权合作伙伴公司用户使用目录（请参阅 [Azure Active Directory B2B collaboration（Azure Active Directory B2B 协作）](/documentation/articles/active-directory-b2b-what-is-azure-ad-b2b/)）。你需要[上传指定电子邮件地址的 CSV 文件](/documentation/articles/active-directory-b2b-references-csv-file-format/)。

6. 在用户的“配置文件”页上，提供名字和姓氏、用户友好名称，并从“角色”列表中选择用户角色。有关用户和管理员角色的详细信息，请参阅 [Assigning administrator roles in Azure AD（在 Azure AD 中分配管理员角色）](/documentation/articles/active-directory-assign-admin-roles/)。指定是否要为用户**启用 Multi-Factor Authentication**。

7. 在“获取临时密码”页上，选择“创建”。

> [AZURE.IMPORTANT] 如果你所在的组织使用多个域，在添加用户帐户时你应知道以下问题：
>
> - 若要跨域添加具有相同用户主体名称 (UPN) 的用户帐户，例如，你可以**先**添加 geoffgrisso@contoso.onmschina.cn，**再**添加 geoffgrisso@contoso.com。
> - **不要**在添加 geoffgrisso@contoso.onmschina.cn 之前添加 geoffgrisso@contoso.com。此顺序非常重要，事后想要撤消操作将很麻烦。

如果你更改身份与本地 Active Directory 服务同步的用户的信息，则无法更改 Azure 经典管理门户中的用户信息。若要更改该用户信息，请使用本地 Active Directory 管理工具。

## 创建外部用户

你还可以从你所属的 Azure AD 目录添加用户，或者通过上载 CSV 文件来添加合作伙伴公司的用户。若要添加外部用户，请针对“用户类型”指定“另一个 Microsoft Azure AD 目录中的用户”或“合作伙伴公司的用户”。

任一类型的用户源自另一个目录，并且将添加为**外部用户**。外部用户可与目录中的其他用户协作，而无需添加新帐户和凭据。当外部用户登录时，系统会使用这些用户的主目录对其进行身份验证，这种身份验证适用于这些用户添加到的其他所有目录。

## 外部用户管理和限制

将另一个目录中的用户添加到你的目录时，该用户是新目录中的外部用户。显示名称和用户名是从用户的主目录复制的，将用于目录中的外部用户。此后，外部用户帐户的属性是完全独立的。如果你对主目录中的用户进行属性更改，这些更改不会传播到你目录中的外部用户帐户。

这两个帐户之间的唯一联系是用户始终针对主目录或使用它们的 Microsoft 帐户进行身份验证。这就是为什么你看不到重置密码或为外部用户启用多重身份验证选项的原因。目前，主目录的身份验证或 Microsoft 帐户策略是用户登录时唯一需要进行评估的策略。

> [AZURE.NOTE]
你仍然可以禁用目录中的外部用户，这将阻止其对目录访问。

如果在用户的主目录中删除了用户或取消 Microsoft 帐户，外部用户仍然存在于你的目录中。但是，目录中的用户无法访问资源，因为他们无法使用主目录或 Microsoft 帐户进行身份验证。

### 目前支持 Azure AD 外部用户访问的服务

- **Azure 经典管理门户**：允许身为多个目录的管理员的外部用户管理这些目录。
- **SharePoint Online**：如果启用外部共享，则允许外部用户访问 SharePoint Online 的已授权资源。
- **Dynamics CRM**：如果用户通过 PowerShell 获得许可，则允许外部用户访问 Dynamics CRM 中的已授权资源。
- **Dynamics AX**：如果用户通过 PowerShell 获得许可，则允许外部用户访问 Dynamics AX 中的已授权资源。适用于 [Azure AD 外部用户](#known-limitations-of-azure-ad-external-users)和[来宾用户](#guest-user-management-and-limitations)的限制同样适用于 Dynamics AX 中的外部用户。

### Azure AD 外部用户的已知限制

- 身为管理员的外部用户无法将来自合作伙伴公司的用户添加到其主目录以外的目录（B2B 协作）
- 外部用户无法同意在其主目录以外的目录中的多租户应用程序
- PowerBI 目前不支持外部用户访问
- Office 门户不支持向外部用户提供许可


## 后续步骤

- [向 Azure Active Directory 添加新用户](/documentation/articles/active-directory-create-users/)
- [管理 Azure AD](/documentation/articles/active-directory-administer/)
- [在 Azure AD 中管理密码](/documentation/articles/active-directory-manage-passwords/)
- [在 Azure AD 中管理组](/documentation/articles/active-directory-manage-groups/)

<!---HONumber=Mooncake_0613_2016-->