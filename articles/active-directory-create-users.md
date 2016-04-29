<properties 
	pageTitle="在 Azure AD 中创建或编辑用户" 
	description="本主题说明如何在 Azure AD 中创建或编辑用户帐户。" 
	services="active-directory" 
	documentationCenter="" 
	authors="Justinha" 
	manager="TerryLan" 
	editor="LisaToft"/>

<tags 
	ms.service="active-directory" 
	ms.date="03/03/2016"
	wacn.date="04/28/2016"/>  

# 在 Azure AD 中创建或编辑用户

你必须在你的租户目录中为每个访问 Microsoft 云服务的用户添加帐户。你还可以更改用户帐户或删除不再需要的用户帐户。默认情况下，用户不具有管理员权限，但你可以给他们分配这种权限。

## 创建用户

1. 单击“Active Directory”，然后选择你所在组织的目录的名称。
2. 在“用户”页上，单击“添加用户”。
3. 在“告诉我们有关此用户的信息”页上，对于“用户类型”，请选择下列其中一项：

	- **组织中的新用户** – 在目录中创建新的用户帐户
	- **现有 Microsoft 帐户的用户** – 将现有 Microsoft 使用者帐户添加到你的目录（例如 Outlook 帐户）
	- **另一个 Azure AD 目录中的用户** – 将源自另一个 Azure AD 目录的用户帐户添加到目录（注意：必须是其他目录的成员才能选择其中的用户）
	- **合作伙伴公司的用户** - 邀请并授权合作伙伴公司用户使用目录（请参阅 [Azure Active Directory B2B collaboration](active-directory-b2b-what-is-azure-ad-b2b.md)（Azure Active Directory B2B 协作））


4. 根据选择的用户类型，输入用户名、电子邮件地址或上载 CSV 文件，其中包括登录用户的电子邮件地址。
5. 在该用户的“配置文件”页上，提供用户的名字和姓氏、用户友好名称，并从“角色”下拉菜单中选择用户角色。有关用户和管理员角色的详细信息，请参阅 [Assigning administrator roles in Azure AD](active-directory-assign-admin-roles.md)（在 Azure AD 中分配管理员角色）。指定是否要**启用 Multi-Factor Authentication**。
6. 在“获取临时密码”页上，单击“创建”。

如果你所在的组织使用多个域，在创建用户帐户时你应知道以下问题：

- 你可以跨域使用同一个用户主体名称 (UPN) 来创建用户帐户，例如，你可以先创建 geoffgrisso@contoso.onmicrosoft.com，再创建 geoffgrisso@contoso.com。
- 不能先创建 geoffgrisso@contoso.com，再创建 geoffgrisso@contoso.onmicrosoft.com。

## 编辑用户

在 Azure 经典门户中编辑用户：

1. 单击“Active Directory”，然后单击你所在组织的目录的名称。
2. 在“用户”页上，单击你想要编辑的用户的显示名称。
3. 完成更改，然后单击“保存”。

如果你尝试编辑的用户与你的本地 Active Directory 服务同步，将出现一条错误消息，你将无法通过此过程编辑用户。若要编辑用户，请使用你的本地 Active Directory 管理工具。

## 重置用户密码

1. 单击“Active Directory”，然后单击你所在组织的目录的名称。
2. 在“用户”页上，单击你想要编辑的用户的显示名称。
3. 在门户底部，单击“重置密码”。
4. 在重置密码对话框中，单击“重置”。
5. 单击复选标记以确认密码已重置。

## 创建外部用户

在 Azure AD 中，你还可以通过上载 CSV 文件，将来自所属的另一个 Azure AD 目录且具有 Microsoft 帐户的用户，或来自合作伙伴公司的用户，添加到 Azure AD 目录。若要创建外部用户，请在门户中添加用户，针对“用户类型”选择“另一个 Microsoft Azure AD 目录中的用户”或“合作伙伴公司的用户”。

任一类型的用户源自另一个目录，并且将添加为**外部用户**。外部用户可以使用单个帐户与已存在于目录中的用户协作，而不需要添加帐户和凭据。当外部用户登录时，系统会根据这些用户的主目录对其进行身份验证，这种身份验证适用于这些用户添加到的其他所有目录。

## 外部用户管理和限制

将另一个目录中的用户添加到你的目录时，该用户是新目录中的外部用户。最初，从用户的“主目录”复制显示名称和用户名，并将其标记在你目录中的外部用户上。从那时起，这些属性与外部用户帐户的其他属性是完全不相关的：如果你对主目录中的用户进行任何更改（如更改用户的名称、添加职务等），这些更改不会传播到你目录中的外部用户帐户。

这两个帐户之间的唯一联系是用户始终针对主目录或使用它们的 Microsoft 帐户进行身份验证。这就是为什么你看不到“重置密码”或“为外部用户启用多重身份验证”选项的原因：目前，主目录的身份验证或 Microsoft 帐户策略是用户登录时唯一需要进行评估的策略。

> [AZURE.NOTE]
你仍然可以禁用目录中的外部用户，这将阻止其对目录访问。

如果在用户的主目录中删除了用户或取消 Microsoft 帐户，外部用户仍然存在于你的目录中。但是，用户将无法访问你目录中的资源，因为用户不能针对该目录或 Microsoft 帐户进行身份验证。

下面是目前支持 Azure AD 外部用户访问的服务：

- **Azure 经典门户**：允许身为多个目录的管理员的外部用户管理这些目录
- **SharePoint Online**：如果启用外部共享，则允许外部用户访问 SharePoint Online 的已授权资源
- **Dynamics CRM**：如果用户通过 PowerShell 获得许可，则允许外部用户访问 Dynamics CRM 中的已授权资源

下面是 Azure AD 外部用户的已知限制：

- 身为管理员的外部用户无法将来自合作伙伴公司的用户添加到其主目录以外的目录（B2B 协作）
- 外部用户无法同意在其主目录以外的目录中的多租户应用程序
- Visual Studio Online 目前不支持外部用户访问*
- PowerBI 目前不支持外部用户访问
- Office 门户不支持向外部用户提供许可

Visual Studio Online 不允许使用 Microsoft 帐户进行身份验证的外部用户访问，但使用工作或学校帐户进行身份验证的外部用户没有此限制。

## 来宾用户管理和限制

**来宾**是目录中 UserType 属性设置为“Guest”的用户帐户。普通用户的 UserType 属性为“Member”，表示他们是目录中的成员。来宾帐户代表来自其他目录的用户，其已受邀到目录以访问特定资源，例如 SharePoint Online 文档、应用程序或 Azure 资源。

来宾在目录中的权利有限。这些权利限制了来宾发现有关目录中其他用户的信息，但仍然可以用户和组进行交互，这些用户和组与其正在处理的资源关联。来宾用户可以：

- 查看与他们被分配到的 Azure 订阅相关联的用户和组
- 查看其所属组的成员
- 如果他们已经知道用户的完整电子邮件地址，则可以查找目录中的其他用户
- 仅能查看他们查找的用户的有限属性集 - 仅限于显示名称、电子邮件地址、用户主体名称 (UPN) 和照片缩略图
- 获取租户目录中已验证的域列表
- 同意应用程序，授予它们与在目录中相同的成员访问权限

## 配置用户访问策略

目录的“配置”选项卡包含用于控制外部用户访问权限的选项。只能在 Azure 经典门户 UI 中由目录全局管理员更改这些选项（不支持 Windows PowerShell 或 API 方法）。
若要在 Azure 经典门户中打开“配置”选项卡，请单击“Active Directory”，然后单击目录的名称。

![Azure Active Directory 中的“配置”选项卡][1]

然后，便可以编辑用于控制外部用户访问权限的选项。

![][2]


## 后续步骤

- [管理 Azure AD](/documentation/articles/active-directory-administer)
- [在 Azure AD 中管理密码](/documentation/articles/active-directory-manage-passwords)
- [在 Azure AD 中管理组](/documentation/articles/active-directory-manage-groups)

<!--Image references-->
[1]: ./media/active-directory-create-users/RBACDirConfigTab.png
[2]: ./media/active-directory-create-users/RBACGuestAccessControls.png

<!---HONumber=Mooncake_0411_2016-->