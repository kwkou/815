<properties
	pageTitle="在 Azure Active Directory 预览版中更改企业应用的名称或徽标 | Azure"
	description="如何在 Azure Active Directory 中更改自定义企业应用的名称或徽标"
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

# 在 Azure Active Directory 预览版中更改企业应用的名称或徽标

在 Azure Active Directory (Azure AD) 预览版中更改自定义企业应用程序的名称或徽标很容易。[预览版内容](/documentation/articles/active-directory-preview-explainer/) 用户必须具有相应的权限才能进行这些更改。在当前预览版中，用户必须是自定义应用的创建者。

## 如何更改企业应用的名称或徽标？

1. 使用充当目录全局管理员的帐户登录到 [Azure 门户](https://portal.azure.cn)。

2. 选择"更多服务"，在文本框中输入 Azure Active Directory，然后选择 **Enter**。

3. 在"Azure Active Directory - *directoryname* "边栏选项卡（即所管理目录的 Azure AD 边栏选项卡）中，选择"企业应用程序"。

	![打开企业应用](./media/active-directory-coreapps-change-app-logo-azure-portal/open-enterprise-apps.png)

4. 在"企业应用程序"边栏选项卡中，选择"所有应用程序"。此时会显示可管理应用的列表。

5. 在在"企业应用程序 - 所有应用程序"边栏选项卡中，选择一个应用。

6. 在" *appname* "边栏选项卡（即标题中包含所选应用名称的边栏选项卡）中，选择"属性"。

	![选择属性命令](./media/active-directory-coreapps-change-app-logo-azure-portal/select-app.png)

7. 在" *appname* - 属性"边栏选项卡中，浏览到可用作新徽标的文件，并/或编辑应用名称。

	![更改应用徽标或 nameproperties 命令](./media/active-directory-coreapps-change-app-logo-azure-portal/change-logo.png)

8. 选择"保存"命令。

## 后续步骤

- [Assign a user or group to an enterprise app](/documentation/articles/active-directory-coreapps-assign-user-azure-portal/)（向企业应用分配用户或组）
- [Remove a user or group assignment from an enterprise app](/documentation/articles/active-directory-coreapps-remove-assignment-user-azure-portal/)（删除企业应用中的用户或组分配）
- [Disable user sign-ins for an enterprise app](/documentation/articles/active-directory-coreapps-disable-app-azure-portal/)（禁用企业应用的用户登录）

<!---HONumber=Mooncake_0926_2016-->
