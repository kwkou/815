<properties
	pageTitle="使用自定义域名来简化用户登录体验 | Microsoft Azure"
	description="介绍如何将自己的域名添加到 Azure Active Directory，并提供其他相关信息。"
	services="active-directory"
	documentationCenter=""
	authors="jeffsta"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="02/08/2016"
	wacn.date="04/28/2016"/>

# 使用自定义域名来简化用户登录体验

你可以使用自己的自定义域来改善及简化 Azure Active Directory 中的登录和其他用户体验。例如，如果你的组织拥有域名“contoso.com”，则用户可以使用他们熟悉的用户名（例如“joe@contoso.com”）来登录。

Azure Active Directory 中的每个目录随附内置域名，格式为“contoso.onmicrosoft.com”，可让你开始使用 Azure 或其他 Microsoft 服务。

## 在 Azure AD 中使用自定义域名

如果你的组织拥有用户熟悉的自定义域名，则最佳做法是在 Azure Active Directory 中使用该自定义域名。若要在 Azure Active Directory 中使用自定义域名，请执行以下操作：



## 在 Office 365 和其他服务中使用自定义域名

与 Azure AD 中的其他资源一样，已添加并验证的自定义域名可以在 Office 365、Intune 以及使用 Azure AD 的其他应用程序中使用。例如，在 Exchange Online 中使用自定义域名可让用户以熟悉的电子邮件地址（例如 joe@contoso.com）发送和接收电子邮件。若要让其他这些应用程序使用自定义域，你需要根据应用程序所述，在 DNS 注册机构添加其他 DNS 条目。

-   [与 Office 365 中使用自定义域](https://support.office.com/article/Add-your-users-and-domain-to-Office-365-6383f56d-3d09-4dcb-9b41-b5f5a5efd611?ui=en-US&rs=en-US&ad=US)

-   [在 Intune 中使用自定义域](https://technet.microsoft.com/library/dn646966.aspx#BKMK_DomainNames)

## 在 Azure AD 中管理域名

在 Azure Active Directory 中，域名一般无需持续进行管理。


## Azure Active directory 的内置域名

分辨 Azure Active Directory (Azure AD) 中的内置域和添加的自定义域之间的差异。

-   内置：Azure AD 目录随附的域，格式为 contoso.onmicrosoft.com

-   自定义：组织拥有的域名，例如 contoso.com

## 使用 PowerShell 或图形 API 管理域名

针对 Azure Active Directory 中域名的大多数管理任务也可以使用 Microsoft PowerShell 或者使用图形 API 以编程方式来完成。

-   [使用 PowerShell 管理 Azure AD 中的域名](https://msdn.microsoft.com/library/azure/e1ef403f-3347-4409-8f46-d72dafa116e0#BKMK_ManageDomains)




## 后续步骤

如果你需要额外的资源来了解 Azure Active Directory 中的域名用法，请尝试：

- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding)



<!---HONumber=Mooncake_0411_2016-->