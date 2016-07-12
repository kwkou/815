<properties
	pageTitle="在 Azure Active Directory 中添加用户或更改用户信息 | Azure"
	description="介绍如何在 Azure Active Directory 中添加用户或更改用户信息，包括外部用户和来宾用户。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="03/31/2016"
	wacn.date="06/14/2016"/>

# 在 Azure Active Directory 中添加或更改用户

需要在租户目录中为每个访问 Microsoft 云服务的用户添加一个帐户。默认情况下，添加的用户没有管理员权限，但你随时可以向他们分配角色。

## 添加用户

1. 使用充当目录全局管理员的帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 选择“Active Directory”，然后选择组织目录的名称。
3. 选择“用户”选项卡，然后在命令栏中选择“添加用户”。
4. 在“告诉我们有关此用户的信息”页上的“用户类型”下，选择下列其中一项：

	- **组织中的新用户** – 在目录中添加新的用户帐户。
	- **现有 Microsoft 帐户的用户** – 将现有 Microsoft 使用者帐户添加到你的目录（例如 Outlook 帐户）
	- **另一个 Azure AD 目录中的用户** - 将源自另一个 Azure AD 目录的用户帐户添加到你的目录。仅当你也是另一目录的成员时，才可以选择该目录中的用户。
5. 根据“用户类型”输入用户名、电子邮件地址，或者上载指定了电子邮件地址的 CSV 文件。
6. 在该用户的“配置文件”页上，提供名字和姓氏、用户友好名称，并从“角色”列表中选择用户角色。有关用户和管理员角色的详细信息，请参阅 [Assigning administrator roles in Azure AD（在 Azure AD 中分配管理员角色）](/documentation/articles/active-directory-assign-admin-roles/)。指定是否要**启用 Multi-Factor Authentication**。
7. 在“获取临时密码”页上，选择“创建”。

> [AZURE.IMPORTANT]
> - 如果你所在的组织使用多个域，在添加用户帐户时你应知道以下问题：
> - 若要跨域使用同一个用户主体名称 (UPN) 来创建用户帐户，例如，你可以**先**创建 geoffgrisso@contoso.partner.onmschina.cn，**再**创建 geoffgrisso@contoso.com。
> - **不要**在添加 geoffgrisso@contoso.partner.onmschina.cn之前添加 geoffgrisso@contoso.com。此顺序非常重要，事后想要撤消操作将很麻烦。

## 更改用户信息

可以更改任何用户属性，但对象 ID 除外。

1. 打开你的目录。
2. 选择“用户”选项卡，然后选择要更改的用户的显示名称。
3. 完成更改，然后单击“保存”。

如果要更改的用户已与本地 Active Directory 服务同步，则无法使用此过程来更改用户信息。若要更改该用户，请使用本地 Active Directory 管理工具。

## 重置用户密码

1. 打开你的目录。
2. 选择“用户”选项卡，然后选择要更改的用户的显示名称。
3. 在命令栏中，选择“重置密码”。
4. 在重置密码对话框中，单击“重置”。
5. 选中相应的复选标记以完成重置密码。

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

## 来宾用户管理和限制

来宾帐户是来自其他目录的用户，他们已受邀加入你的目录，可以访问 SharePoint 文档、应用程序或其他 Azure 资源。目录中来宾帐户的基础 UserType 属性设置为“Guest”。 普通用户（具体而言，指目录的成员）的 UserType 属性设置为“Member”。

来宾在目录中的权利有限。这些权利限制了客户发现有关目录中其他用户的信息。但是，来宾用户仍可与其使用的资源相关联的用户和组交互。来宾用户可以：

- 查看与他们被分配到的 Azure 订阅相关联的用户和组
- 查看他们所属的组的成员
- 如果他们已经知道用户的完整电子邮件地址，则可以查找目录中的其他用户
- 仅能查看他们查找的用户的有限属性集 - 仅限于显示名称、电子邮件地址、用户主体名称 (UPN) 和照片缩略图
- 获取租户目录中已验证的域列表
- 同意应用程序，授予它们与在目录中相同的成员访问权限

## 设置用户访问策略

目录的**“配置”**选项卡包含用于控制外部用户访问权限的选项。这些选项只能由目录全局管理员在 Azure 经典管理门户中更改。目前不支持 PowerShell 或 API 方法。

若要在 Azure 经典管理门户中打开“配置”选项卡，请选择“Active Directory”，然后选择目录的名称。

![Azure Active Directory 中的“配置”选项卡][1]

然后，便可以编辑用于控制外部用户访问权限的选项。

![][2]


## 后续步骤

- [管理 Azure AD](/documentation/articles/active-directory-administer/)
- [在 Azure AD 中管理密码](/documentation/articles/active-directory-manage-passwords/)
- [在 Azure AD 中管理组](/documentation/articles/active-directory-manage-groups/)

<!--Image references-->
[1]: ./media/active-directory-create-users/RBACDirConfigTab.png
[2]: ./media/active-directory-create-users/RBACGuestAccessControls.png

<!---HONumber=Mooncake_0516_2016-->