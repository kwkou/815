<properties
	pageTitle="向 Azure Active Directory 添加新用户 | Azure"
	description="说明如何在 Azure Active Directory 中添加新用户或更改用户信息。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>  


<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/22/2016"
	wacn.date="12/16/2016"
	ms.author="curtand"/>  


# 向 Azure Active Directory 添加新用户或具有 Microsoft 帐户的用户

请添加用户以填充你的目录。本文说明如何在组织中添加新用户，以及如何添加具有 Microsoft 帐户的用户。有关在 Azure Active Directory 中添加来自其他目录的用户或添加来自合作伙伴公司的用户的详细信息，请参阅[在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户](/documentation/articles/active-directory-create-users-external/)。默认情况下，添加的用户没有管理员权限，但你随时可以向他们分配角色。

## 添加用户

1. 使用充当目录全局管理员的帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 选择“Active Directory”，然后选择组织目录的名称。
3. 选择“用户”选项卡，然后在命令栏中选择“添加用户”。
4. 在“告诉我们有关此用户的信息”页上的“用户类型”下，选择下列其中一项：

	- **组织中的新用户** - 在目录中添加新的用户帐户。
	- **具有现有 Microsoft 帐户的用户** - 将现有 Microsoft 使用者帐户添加到你的目录（例如 Outlook 帐户）

5. 根据“用户类型”输入用户名（适用于新用户）或电子邮件地址（适用于具有 Microsoft 帐户的用户）。
6. 在用户的“配置文件”页上，提供名字和姓氏、用户友好名称，并从“角色”列表中选择用户角色。有关用户和管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。指定是否要为用户**启用多重身份验证**。
7. 在“获取临时密码”页上，选择“创建”。

> [AZURE.IMPORTANT] 如果你所在的组织使用多个域，在添加用户帐户时你应知道以下问题：
>
> - 若要跨域添加具有相同用户主体名称 (UPN) 的用户帐户，例如，你可以**先**添加 geoffgrisso@contoso.partner.onmschina.cn，**再**添加 geoffgrisso@contoso.com。
> - **不要**在添加 geoffgrisso@contoso.partner.onmschina.cn 之前添加 geoffgrisso@contoso.com。此顺序非常重要，事后想要撤消操作将很麻烦。

## 更改用户信息

可以更改任何用户属性，但对象 ID 除外。

1. 打开你的目录。
2. 选择“用户”选项卡，然后选择要更改的用户的显示名称。
3. 完成更改，然后单击“保存”。

如果要更改的用户已与本地 Active Directory 服务同步，则无法使用此过程来更改用户信息。若要更改该用户，请使用本地 Active Directory 管理工具。

## 来宾用户管理和限制

来宾帐户是来自其他目录的用户，他们已受邀加入你的目录，可以访问 SharePoint 文档、应用程序或其他 Azure 资源。目录中来宾帐户的基础 UserType 属性设置为“Guest”。 普通用户（具体而言，指目录的成员）的 UserType 属性设置为“Member”。

来宾在目录中的权利有限。这些权利限制了客户发现有关目录中其他用户的信息。但是，来宾用户仍可与其使用的资源相关联的用户和组交互。来宾用户可以：

- 查看与他们被分配到的 Azure 订阅相关联的用户和组
- 查看他们所属的组的成员
- 如果他们已经知道用户的完整电子邮件地址，则可以查找目录中的其他用户
- 仅能查看他们查找的用户的有限属性集 - 仅限于显示名称、电子邮件地址、用户主体名称 (UPN) 和照片缩略图
- 获取目录中已验证域的列表
- 同意应用程序，授予它们与在目录中相同的成员访问权限

## 设置来宾用户访问策略

目录的“配置”选项卡包含用于控制来宾用户访问权限的选项。这些选项只能由目录全局管理员在 Azure 经典管理门户中更改。目前不支持 PowerShell 或 API 方法。

若要在 Azure 经典管理门户中打开“配置”选项卡，请选择“Active Directory”，然后选择目录的名称。

![Azure Active Directory 中的“配置”选项卡][1]  


然后，便可以编辑用于控制来宾用户访问权限的选项。

![来宾用户的访问控制选项][2]


## 后续步骤

- [在 Azure Active Directory 中添加来自其他目录或合作伙伴公司的用户](/documentation/articles/active-directory-create-users-external/)
- [管理 Azure AD](/documentation/articles/active-directory-administer/)
- [在 Azure AD 中管理密码](/documentation/articles/active-directory-manage-passwords/)

<!--Image references-->
[1]: ./media/active-directory-create-users/RBACDirConfigTab.png
[2]: ./media/active-directory-create-users/RBACGuestAccessControls.png

<!---HONumber=Mooncake_1024_2016-->
