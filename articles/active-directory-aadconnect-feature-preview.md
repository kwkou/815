<properties
   pageTitle="Azure AD Connect 中的预览版功能 | Azure"
   description="本主题详细介绍 Azure AD Connect 中以预览版形式提供的功能。"
   services="active-directory"
   documentationCenter=""
   authors="andkjell"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"  
   ms.date="10/13/2015"
   wacn.date="11/02/2015"/>

# 有关预览版功能的详细信息
本主题介绍如何使用预览版中当前提供的功能。

## 用户写回
> [AZURE.IMPORTANT]AAD Connect 的 8 月更新版中临时删除了用户写回预览版功能。如果你已启用此功能，现在应将它禁用。

用户写回目前以先行预览版的形式提供。仅当 Azure AD 是所有用户对象的源，并且在启用该功能之前本地 Active Directory 为空时，才可以使用该功能。

> [AZURE.IMPORTANT]只能在测试环境中测试此功能，而不能在用于生产用途的 Azure AD 目录中使用它。

## 组写回
可选功能中的组写回选项可让你将“Office 365 组”写回到装有 Exchange 的林。这是始终在云中受控制的新组类型。你可以在 outlook.office365.com 或 myapps.microsoft.com 中找到此组类型，如下所示：


![同步筛选](./media/active-directory-aadconnect-feature-preview/office365.png)

![同步筛选](./media/active-directory-aadconnect-feature-preview/myapps.png)

此组将在本地 AD DS 中显示为分发组。你的本地 Exchange 服务器必须是 Exchange 2013 累积更新 8（2015 年 3 月发行），才能识别这个新的组类型。

**Note**

- 预览版中当前不会填充通讯簿属性。若要这么做，最简单的方法是使用 Exchange PowerShell cmdlet update-recipient。
- 只有使用 Exchange 架构的林才是组的有效目标。如果没有检测到 Exchange，则无法启用组写回功能。
- 组写回功能当前无法处理安全组或分发组。

可以在[此处](http://aka.ms/O365g)找到有关 Office 365 组的详细信息。

## 目录扩展
目录扩展可让你使用本地 Active Directory 中自己的属性扩展 Azure AD 中的架构。

仅支持单值属性，并且属性中的值不能超过 250 个字符。将使用选择的属性来扩展 Metaverse 和 Azure AD 架构。在 Azure AD 中，将添加具有这些属性的新应用程序。

![同步筛选](./media/active-directory-aadconnect-feature-preview/extension3.png)

现在可以通过图形提供这些属性：

![同步筛选](./media/active-directory-aadconnect-feature-preview/extension4.png)

## 后续步骤
配置 [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom)。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)的详细信息。

<!---HONumber=79-->