<properties
	pageTitle="Azure Active Directory 中自定义域名的概念性概述 | Azure"
	description="介绍了在 Azure Active Directory 中使用自定义域名的概念性框架，其中包括用于实现单一登录的联盟"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/21/2016"
	wacn.date="06/24/2016"/>

# Azure Active Directory 中自定义域名的概念性概述

域名是许多目录资源标识符的重要部分：它可能是用户的用户名或电子邮件地址的一部分、组地址的一部分，也可能是应用程序的应用 ID URI 的一部分。Azure Active Directory (Azure AD) 中的资源可包含已验证为目录（包含该资源）所拥有的域名。只有全局管理员才能在 Azure AD 中执行域管理任务。

Azure AD 中的域名是全局唯一的。一个域名只可供一个 Azure AD 使用。如果某个 Azure AD 目录已验证域名，则其他 Azure AD 目录就不能验证或使用该相同的域名。

## 初始域名和自定义域名

Azure AD 中的每个域名不是初始域名就是自定义域名。

每个 Azure AD 都随附形式为 contoso.onmicrosoft.com 的初始域名。在创建目录时，会建立此三级域名（通常由创建目录的管理员建立），在此示例中为“contoso.onmicrosoft.com”。目录的初始域名无法更改或删除。完全正常运行的初始域名主要用作启动机制，直到验证了自定义域名为止。

在大部分生产环境中，目录具有至少一个已验证的自定义域，例如“contoso.com”，而且是可让最终用户看到的自定义域。自定义域名是由该组织拥有和使用的域名，例如“contoso.com”，用途包括托管其网站等等。员工很熟悉此域名，因为它是用来登录公司网络或发送和检索电子邮件的用户名的一部分。

自定义域名必须先添加到目录中并经过验证，才可供 Azure AD 使用。

## 已验证和未验证的域名

目录的初始域名会由 Azure AD 隐式评估为已验证。管理员在 Azure AD 中添加自定义域名时，其最初是未验证的状态。Azure AD 不允许任何目录资源使用未验证的域名。如此可确保只有一个目录可以使用特定的域名，而且使用域名的组织实际拥有该域名。

Azure AD 通过在域名的域名服务 (DNS) 区域文件中查找特定的条目，来验证域名的所有权。若要验证域名的所有权，管理员可以从 Azure AD 获取 Azure AD 将查找的 DNS 条目，并将该条目添加到域名的 DNS 区域文件。该域的域名注册机构负责维护 DNS 区域文件。域验证步骤可在[将自定义域添加到 Azure AD 目录](/documentation/articles/active-directory-add-domain/)一文中找到。

将 DNS 条目添加到域名的区域文件中，并不会影响其他域服务，例如电子邮件或 Web 托管。

## 联盟域名和托管域名

你可以配置 Azure AD 中的自定义域名，让用户在本地 Active Directory 与 Azure AD 之间获得联合登录体验。为联盟配置域除了需要更新 Azure AD 中的特权资源，还需要更新 Windows Server Active Directory。配置联盟域的操作必须在 Azure AD Connect 中或使用 PowerShell 来完成。无法从 Azure 经典管理门户启动自定义域联盟操作。[观看此视频以了解如何配置 AD FS，让用户能使用 Azure AD Connect 登录](http://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Configuring-AD-FS-for-user-sign-in-with-Azure-AD-Connect)。

未联盟的域有时也称为托管域。Azure AD 目录的初始域会隐式评估为托管域。

## 主要域名

目录的主要域名是管理员在 [Azure 经典管理门户](https://manage.windowsazure.cn/)或其他门户（例如 Office 365 管理门户）中创建新用户时，预先选择作为用户名“域”部分的默认值的域名。一个目录只能有一个主要域名。管理员可以将主要域名更改为任何未联盟的已验证自定义域，或更改为初始域。

## Azure AD 和其他 Microsoft Online Services 中的域名

必须先在 Azure AD 中验证域名，才可将其供其他 Microsoft Online Service（例如 Exchange Online、SharePoint Online 和 Intune）使用。其他这些服务通常要求管理员添加一或多个特定于服务的 DNS 条目。

Azure Web 应用使用其自身的机制来验证域的所有权。必须验证域是否可用于 Azure AD，即使先前已验证其可供依赖该 Azure AD 的订阅中的 Azure Web 应用使用也是如此。Azure Web 应用可使用在保护 Web 应用的目录以外的目录中进行过验证的域名。

## 管理域名

可以在 Azure 经典管理门户和 PowerShell 中完成域管理任务。许多任务可使用 Azure AD 图形 API（公共预览版）来完成。

-   [添加和验证自定义域名](/documentation/articles/active-directory-add-domain/)

-   [在 Azure 经典管理门户中管理域](/documentation/articles/active-directory-add-manage-domain-names/)

-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)

-   [使用 Azure AD 图形 API 管理 Azure AD 中的域名](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/domains-operations)

<!---HONumber=Mooncake_0613_2016-->