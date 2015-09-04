
<properties 
	pageTitle="使用组来管理对 SaaS 应用程序的访问 | Windows Azure" 
	description="本主题介绍如何使用 Azure AD Premium 中的组来分配对集成 Azure AD 的 SaaS 应用程序的访问权限。" 
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


# 使用组来管理对 SaaS 应用程序的访问

如果你安装了 Azure AD 高级版，则可以使用组来分配对集成 Azure AD 的 SaaS 应用程序的访问权限。例如，如果你想要分配市场营销部门的访问权限以使用五个不同的 SaaS 应用程序，你可以创建一个包含市场营销部门中用户的组，然后将该组分配到市场营销部门所需的这五个 SaaS 应用程序。这样，通过在一个位置管理市场营销部门的成员身份，可以节省时间。然后，当他们被添加为市场营销组的成员时，用户就被分配到应用程序，当将他们从市场营销组中删除时，就将从应用程序中删除其分配。

此功能可用于数百个你可从 Azure AD 应用程序库内添加的应用程序。

**将组的访问权限分配给 SaaS 应用程序**


1. 打开所选择的浏览器并转到 Azure 管理门户。在 Azure 管理门户的左侧导航栏上，找到 Active Directory 扩展。在“目录”选项卡下，单击你要在其中将组的访问权限分配给 SaaS 应用程序的目录。


2. 单击你目录的“应用程序”选项卡。单击一个从应用程序库添加的应用程序，然后单击“用户和组”选项卡。

3. 在“用户和组”选项卡上，在“开头”字段中，输入你要向其分配访问权限的组的名称，然后单击右上角的复选标记。你只需键入组名称的第一部分。然后，单击该组以突出显示它，再单击“分配访问权限”按钮并在看到确认消息时单击“是”。


4. 你还可以查看直接或通过使用组成员身份分配到该应用程序的用户。若要执行此操作，请将“显示下拉列表”从“组”更改为“所有用户”。该列表显示目录中的用户，而无论是否每个用户都分配到该应用程序。此列表还显示已分配的用户是直接（分配类型显示为“直接”）还是通过组成员身份（分配类型显示为“继承”）分配到应用程序。


> [AZURE.NOTE]一旦启用了 Azure AD Premium，将只看到“用户和组”选项卡。

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->