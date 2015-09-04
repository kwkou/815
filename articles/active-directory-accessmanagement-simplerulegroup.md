<properties
	pageTitle="创建用于为组配置动态成员资格的简单规则 | Windows Azure"
	description="说明如何创建用于为组配置动态成员资格的简单规则。"
	services="active-directory"
	documentationCenter=""
	authors="femila"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory" 
	ms.date="07/13/2015" 
	wacn.date="08/29/2015"/>


# 创建用于为组配置动态成员资格的简单规则

**若要为特定组启用动态成员资格，请执行以下步骤：**

1. 在 Azure 管理门户中，在“组”选项卡下选择需编辑的组，然后在此组的“配置”选项卡上将“启用动态成员资格”开关设为“是”。


2. 现在可以为组设置一条简单的规则，该规则将控制此组的动态成员资格的工作方式。请确保选中“添加用户位置”单选按钮，然后从下拉菜单（如“部门”、“jobTitle”等）中选择用户属性，

3. 接下来，选择一个条件（“不等于”、“等于”、“开头不是”、“开头为”、“不包含”、“包含”、“不匹配”、“匹配”），最后为选定用户属性指定一个值。例如，如果将组分配到 SaaS 应用程序，并通过设置规则启用此组的动态成员资格，且通过此规则将“添加用户位置”设置为 Equals(-eq)Sales Rep 的 jobTitle，那么 Azure AD 目录内职称设置为“销售代表”的所有用户都将有权访问此 SaaS 应用程序。

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->