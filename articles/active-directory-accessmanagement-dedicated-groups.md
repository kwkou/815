<properties 
	pageTitle="Azure Active Directory 中的专用组 | Windows Azure" 
	description="本主题介绍如何在 Azure AD 中管理组。" 
	services="active-directory" 
	documentationCenter="" 
	authors="femila" 
	manager="swadhwa" 
	editor=""
	tags="azure-classic-portal"/>

<tags 
	ms.service="active-directory" 
	ms.date="07/13/2015" 
	wacn.date="08/29/2015"/>

# Azure Active Directory 中的专用组

在 Azure Active Directory 中，专用组是自动创建的，并且专用组的组成员身份也是自动的。你不能通过 Azure 管理门户、Windows PowerShell cmdlet 或以编程方式向专用组添加成员，也不能从专用组删除成员。若要启用专用组，请在 Azure 管理门户中的“配置”选项卡上，将“启用专用组”开关设置为“是”。

在当前公共预览版本中，一旦“启用专用组”开关设置为“是”，你就可以通过将“启用‘所有用户’组”开关设置为“是”来进一步使目录能够自动创建“所有用户”专用组。然后，你还可以编辑此专用组的名称，即在“‘所有用户’组的显示名称”字段中键入名称。

如果你想要将同一权限分配给你目录中的所有用户，则使用“所有用户”专用组会很有帮助。例如，通过将对“所有用户”专用组的访问权限分配给 SaaS 应用程序，你可以授予目录中的所有用户对此应用程序的访问权限。

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->