<properties
	pageTitle="将新用户添加到 Azure Active Directory 预览版 | Azure"
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
	ms.topic="article"
	ms.date="09/12/2016"
	ms.author="curtand"
   	wacn.date="10/11/2016"/>


# 将新用户添加到 Azure Active Directory 预览版

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/active-directory-users-create-azure-portal/)
- [Azure 经典管理门户](/documentation/articles/active-directory-create-users/)

本文介绍如何在 Azure Active Direstory (Azure AD) 预览版中添加组织中的新用户。[预览版内容](/documentation/articles/active-directory-preview-explainer/)

1.  使用目录全局管理员的帐户登录到 [Azure 门户](https://portal.azure.cn)。

2.  选择"更多服务"，在文本框中输入"用户和组"，然后选择 **Enter**。

    ![打开"用户管理"](./media/active-directory-users-create-azure-portal/create-users-user-management.png)

3.  在"用户和组"边栏选项卡中，选择"所有用户"，然后选择"添加"。

    ![选择"添加"命令](./media/active-directory-users-create-azure-portal/create-users-add-command.png)

4.  输入用户的详细信息，如**名称**和**用户名**。用户名的域名部分必须是初始默认域名"foo.partner.onmschina.cn"或已验证的非联合域名（例如"contoso.com"）。

5. 复制或以其他方式记下生成的用户密码，以便在此过程完成后可以提供给用户。

6. （可选）可以打开"个人资料"边栏选项卡、"组"边栏选项卡或"目录角色"边栏选项卡并在其中填写用户的信息。有关用户和管理员角色的详细信息，请参阅[在 Azure AD 中分配管理员角色](/documentation/articles/active-directory-assign-admin-roles/)。

7.  在"用户"边栏选项卡中，选择"创建"。

8. 以安全方式将生成的密码分发给新用户，以便用户可以登录。

## 后续步骤

- [添加外部用户](/documentation/articles/active-directory-users-create-external-azure-portal/)
- [在新的 Azure 门户中重置用户的密码](/documentation/articles/active-directory-users-reset-password-azure-portal/)
- [更改用户的工作信息](/documentation/articles/active-directory-users-work-info-azure-portal/)
- [管理用户个人资料](/documentation/articles/active-directory-users-profile-azure-portal/)
- [在 Azure AD 中删除用户](/documentation/articles/active-directory-users-delete-user-azure-portal/)
- [为用户分配 Azure AD 中的角色](/documentation/articles/active-directory-users-assign-role-azure-portal/)

<!---HONumber=Mooncake_0926_2016-->
