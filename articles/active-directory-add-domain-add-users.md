<properties
	pageTitle="将用户添加到 Azure Active Directory 中的自定义域 | Azure"
	description="如何使用用户帐户填充 Azure Active Directory 中的自定义域。"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/20/2016"
	wacn.date="06/24/2016"/>

# 将用户分配到自定义域

将自定义域添加到 Azure Active Directory 之后，你必须添加此域的用户帐户，才能开始对其身份验证。

## 从企业网络上的目录同步的用户

如果你已经设定本地 Active Directory 与 Azure Active Directory 之间的连接，则同步可以填充帐户。如需有关如何同步 Azure Active Directory 与本地 Active Directory 的详细信息，请参阅[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)。

## 在云中添加和管理的用户

若要更改现有用户帐户的域，请执行以下操作：

1.  使用全局管理员或用户管理员帐户打开 Azure 经典管理门户。

2.  打开你的目录。

3.  选择“用户”选项卡。

4.  从列表中选择用户。

5.  更改用户的域，然后选择“保存”。

也可以使用 [Microsoft PowerShell](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains) 或[图形 API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/domains-operations) 完成此操作。

## 创建新用户时选择自定义域

1.  使用全局管理员或用户管理员帐户打开 Azure 经典管理门户。

2.  打开你的目录。

3.  选择“用户”选项卡。

4.  在命令栏中，选择“添加”。

5.  添加用户名时，请从域列表中选择自定义域。

6.  选择“保存”。

## 后续步骤

-   [使用自定义域名来简化用户登录体验](/documentation/articles/active-directory-add-domain/)

-   [管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)

-   [了解 Azure AD 中的域管理概念](/documentation/articles/active-directory-add-domain-concepts/)

<!---HONumber=Mooncake_0606_2016-->