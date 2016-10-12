<properties
	pageTitle="在 Azure Active Directory 预览版中添加来自其他目录或合作伙伴公司的用户 | Azure"
	description="介绍如何在 Azure Active Directory 中添加用户或更改用户信息，包括外部用户和来宾用户。"
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
	ms.topic="article"
	ms.date="09/12/2016"
	ms.author="curtand"
   wacn.date="10/11/2016"/>
   	wacn.date="10/11/2016"/>

# 在 Azure Active Directory 预览版中添加来自其他目录或合作伙伴公司的用户

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/active-directory-users-create-external-azure-portal/)
- [Azure 经典管理门户](/documentation/articles/active-directory-create-users-external/)

本文介绍如何从 Azure Active Directory (Azure AD) 预览版中的其他目录或合作伙伴公司添加用户。[预览版内容](/documentation/articles/active-directory-preview-explainer/) 有关添加你组织中的新用户和添加具有 Microsoft 帐户的用户的信息，请参阅[将新用户添加到 Azure Active Directory](/documentation/articles/active-directory-users-create-azure-portal/)。默认情况下，添加的用户没有管理员权限，但你随时可以向他们分配角色。

## 添加用户

1.  使用目录全局管理员的帐户登录到 [Azure 门户](https://portal.azure.cn)。

2.  选择"更多服务"，在文本框中输入"用户和组"，然后选择 **Enter**。

    ![打开"用户管理"](./media/active-directory-users-create-external-azure-portal/create-users-user-management.png)

3.  在"用户和组"边栏选项卡中，选择"用户"，然后选择"添加"。

    ![选择"添加"命令](./media/active-directory-users-create-external-azure-portal/create-users-add-command.png)

4. 在"用户"边栏选项卡中，在**名称**中提供显示名称，在**用户名**中提供用户的登录名。

5. 复制或以其他方式记下生成的用户密码，以便在此过程完成后可以提供给用户。

6. （可选）选择"个人资料"，添加用户的名字和姓氏、职务和部门名称。

	![Opening the user profile](./media/active-directory-users-create-external-azure-portal/create-users-user-profile.png)

	- 选择"组"，将用户添加到一个或多个组。

		![将用户添加到组](./media/active-directory-users-create-external-azure-portal/create-users-user-groups.png)

	- 选择"组织角色"，为用户分配"角色"列表中角色。有关用户和管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。

		![为用户分配角色](./media/active-directory-users-create-external-azure-portal/create-users-assign-role.png)

7. 选择"创建"。

8. 以安全方式将生成的密码分发给新用户，以便用户可以登录。

> [AZURE.IMPORTANT] 如果你所在的组织使用多个域，在添加用户帐户时你应知道以下问题：
>
> - 若要跨域添加具有相同用户主体名称 (UPN) 的用户帐户，例如，你可以**先**添加 geoffgrisso@contoso.partner.onmschina.cn，**再**添加 geoffgrisso@contoso.com。
> - **不要**在添加 geoffgrisso@contoso.partner.onmschina.cn 之前添加 geoffgrisso@contoso.com。此顺序非常重要，事后想要撤消操作将很麻烦。

如果更改身份与本地 Active Directory 服务同步的用户的信息，则无法更改 Azure 经典管理门户中的用户信息。若要更改该用户信息，请使用本地 Active Directory 管理工具。


## 后续步骤

- [添加用户](/documentation/articles/active-directory-users-create-azure-portal/)
- [在新的 Azure 门户中重置用户的密码](/documentation/articles/active-directory-users-reset-password-azure-portal/)
- [为用户分配 Azure AD 中的角色](/documentation/articles/active-directory-users-assign-role-azure-portal/)
- [更改用户的工作信息](/documentation/articles/active-directory-users-work-info-azure-portal/)
- [管理用户个人资料](/documentation/articles/active-directory-users-profile-azure-portal/)
- [在 Azure AD 中删除用户](/documentation/articles/active-directory-users-delete-user-azure-portal/)

<!---HONumber=Mooncake_0926_2016-->
