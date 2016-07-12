<properties
	pageTitle="管理 Azure Active Directory 中的自定义域名 | Microsoft Azure"
	description="用于管理 Azure Active Directory 中的自定义域的管理概念和操作指南"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="04/20/2016"
	wacn.date="06/23/2016"/>

# 管理 Azure Active Directory 中的自定义域名

域名是许多目录资源标识符的重要部分：它可能是用户的用户名或电子邮件地址的一部分、组地址的一部分，也可能是应用程序的应用 ID URI 的一部分。Azure Active Directory (Azure AD) 中的资源可包含已验证为目录（包含该资源）所拥有的域名。只有全局管理员才能在 Azure AD 中执行域管理任务。

## 设置 Azure AD 目录的主域名

创建目录后，初始域名（例如“contoso.onmicrosoft.com”）也是目录的主域名。当你在 [Azure 经典管理门户](https://manage.windowsazure.cn/)或其他门户（如 Office 365 管理门户）中创建新用户时，主域是新用户的默认域名。这简化了管理员在门户中创建新用户的过程。

若要更改目录的主域名，请执行以下操作：

1.  使用充当 Azure AD 目录全局管理员的用户帐户登录到 [Azure 经典管理门户](https://manage.windowsazure.cn/)。

2.  在左侧导航栏上选择“Active Directory”。

3.  打开你的目录。

4.  选择“域”选项卡。

5.  在命令栏上选择“更改主域名”按钮。

6.  选择你想要其作为目录的新主域的域。

你可以将目录的主域名更改为任何未联合的已验证自定义域。更改目录的主域不会更改任何现有用户的用户名。

## 将自定义域名添加到你的 Azure AD

可以将最多 900 个自定义域名添加到每个 Azure AD 目录。[添加其他自定义域名](/documentation/articles/active-directory-add-domain/)的过程与第一个自定义域名的相同。

## 添加自定义域的子域

如果想要将第三级域名（如 “europe.contoso.com”）添加到目录，则应首先添加并验证第二级域，例如 contoso.com。子域将由 Azure AD 自动验证。若要查看刚添加的子域是否已经过验证，请刷新浏览器中的页面，其中列出了目录中的域。

## 更改自定义域名的 DNS 注册机构会发生什么情况

如果你更改自定义域名的 DNS 注册机构，则可以继续将你的自定义域名用于 Azure AD 本身，而不会发生中断，也不需要执行其他配置任务。如果你在 Office 365、Intune 或其他依赖于 Azure AD 中的自定义域名的服务中使用自定义域名，请参阅这些服务的文档。

## 删除自定义域名

如果你的组织不再使用某个自定义域名，或者需要在另一个 Azure AD 中使用该域名，你可以从 Azure AD 中删除该域名。

若要删除自定义域名，则必须先确保你的目录中没有任何资源依赖域名。在以下情况中，你无法从目录删除域名：

-   任何用户都有包含域名的用户名、电子邮件地址或代理地址。

-   任何组都有包含域名的电子邮件地址或代理地址。

-   Azure AD 中的任何应用程序都具有包含域名的应用 ID URI。

必须更改或删除 Azure AD 目录中的任何此类资源，然后才能删除自定义域名。

## 使用 PowerShell 或图形 API 管理域名

针对 Azure Active Directory 中域名的大多数管理任务也可以使用 Microsoft PowerShell 或者使用 Azure AD 图形 API（公共预览版）以编程方式来完成。

-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)

-   [使用图形 API 管理 Azure AD 中的域名](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/domains-operations)

## 后续步骤

-   [了解 Azure AD 中的域名](/documentation/articles/active-directory-add-domain-concepts/)

-   [管理自定义域名](/documentation/articles/active-directory-add-manage-domain-names/)

<!---HONumber=Mooncake_0613_2016-->