<properties

	pageTitle="在 Azure Active Directory 中管理安全组 | Windows Azure"
	description="介绍如何注册 Azure，以及可以使用 Azure AD 尝试的前几个步骤。"
	services="active-directory"
	documentationCenter=""
	authors="femila"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory" 
	ms.date="07/13/2015" 
	wacn.date="08/29/2015"/>


#在 Azure Active Directory 中管理安全组


##如何创建和管理安全组

**从 Azure 管理门户创建组**

1. 在管理门户中，单击“Active Directory”，然后单击组织目录的名称。
2. 单击“组”选项卡。
3. 在“组”页上，单击“添加组”。
4. 在“添加组”窗口中，指定组的名称和说明。
5. 可使用 Office 365 帐户门户、Windows Intune 帐户门户或 Azure 管理门户（具体取决于组织订阅的服务）来完成此任务。有关如何使用门户来管理 Azure Active Directory 的详细信息，请参阅“管理 Azure AD 目录”。

## 如何分配或删除安全组中的用户

**在 Azure 管理门户中向组添加成员**

1. 在管理门户中，单击“Active Directory”，然后单击组织目录的名称。
2. 单击“组”选项卡。
3. 在“组”页上，单击要向其添加成员的组的名称。默认情况下，这将显示所选组的“成员”选项卡。
4. 在该组的页面上，单击“添加成员”。
5. 在“添加成员”页面，单击要添加为此组成员的用户或组的名称，并确保将此名称添加到“选定”窗格中。


**在 Azure 管理门户中从组中删除成员**

1. 在管理门户中，单击“Active Directory”，然后单击组织目录的名称。
2. 单击“组”选项卡。
3. 在“组”页中，单击要从中删除成员的组的名称。
4. 在该组的页面上，单击“成员”选项卡。
5. 在该组的页面上，单击要从此组中删除的成员的名称，然后单击“删除”。
6. 单击“是”作为操作验证问题的答案，确认要从组中删除此成员。


##如何使用规则动态地管理安全组成员
**若要为特定组启用动态成员资格，请执行以下步骤：**

1. 在 Azure 管理门户中，在“组”选项卡下选择需编辑的组，然后在此组的“配置”选项卡上将“启用动态成员资格”开关设为“是”。
2. 现在可以为组设置一条简单的规则，该规则将控制此组的动态成员资格的工作方式。确保选中“添加用户位置”单选按钮，然后从下拉菜单（如“部门”、“jobTitle”等）中选择用户属性， 
3. 接下来，选择一个条件（“不等于”、“等于”、“开头不是”、“开头为”、“不包含”、“包含”、“不匹配”、“匹配”），最后为选定用户属性指定一个值。
4. 例如，如果将组分配给 SaaS 应用程序（有关详细信息，请参阅“在 Azure AD 中将组的访问权限分配给 SaaS 应用程序”），并且通过设置规则来为此组启用动态成员资格（通过此规则将“添加用户位置”设置为 Equals(-eq)Sales Rep 的 jobTitle），那么 Azure AD 目录内职称设置为“销售代表”的所有用户都将有权访问 SaaS 应用程序。

下面的主题将提供有关 Azure Active Directory 的一些其他信息

* [使用 Azure Active Directory 组管理对资源的访问](/documentation/articles/active-directory-manage-groups)

* [什么是 Azure Active Directory？](/documentation/articles/active-directory-whatis)

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

<!---HONumber=67-->