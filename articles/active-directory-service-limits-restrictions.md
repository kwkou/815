<properties 
	pageTitle="Azure AD 服务限制和局限性" 
	description="Azure Active Directory 服务的使用限制和其他服务限制。" 
	services="active-directory" 
	documentationCenter="" 
	authors="Justinha" 
	writer="Justinha" 
	manager="TerryLan" 
	editor="LisaToft"/>

<tags 
	ms.service="active-directory"  
	ms.date="04/27/2015" 
	wacn.date="06/16/2015"/>

# Azure AD 服务限制和局限性

以下是 Azure Active Directory 服务的使用限制和其他服务限制。如果你正查找 Windows Azure 服务限制全集，请参阅 <!--[-->Azure 订阅和服务限制、配额和约束<!--](/documentation/articles/azure-subscription-service-limits)-->。

## 目录

单个用户只能与最多 20 个 Azure Active Directory 目录相关联。以下示例中可能达到此限制：

- 单个用户创建了 20 个目录。
- 单个用户以成员身份添加到 20 个目录。
- 单个用户创建了 10 个目录，之后其他人又将该用户添加到 10 个不同的目录。

## 对象

- 对 Azure Active Directory Premium 或 Azure Active Directory Basic、企业移动套件、Office 365、Windows Intune 或其他依赖 Azure Active Directory 目录服务的 Microsoft 联机服务的订户没有限制。
- 在使用 Azure Active Directory 免费版的单一目录中最多可以使用 500,000 个对象。
- 非管理员用户最多可以创建 250 个用户。

##架构扩展

当前“用户”、“组”、“TenantDetail”、“设备”、“应用程序”和“ServicePrincipal” 实体可以用“String”类型或 “Binary” 类型单一值属性进行扩展。其中包括以下限制：

- String 类型扩展最多只能有 256 个字符。
- Binary 类型扩展限制在 256 字节以内。
- 100 扩展值（在 ALL 类型和 ALL 应用程序中）可以编写到任何单一对象中。
- 架构扩展仅在 Graph API 1.21 预览版中可用。必须授予应用程序编写访问注册扩展的权限。

## 应用程序

最多有 10 位用户可以是单一应用程序的所有者。

## 组 

- 最多有 10 位用户可以是单一组的所有者。
- Azure Active Directory 中的单个组可以具有任意数量的对象。


> [AZURE.NOTE]
> 
可以从本地 Active Directory 同步到 Azure Active Directory 的对象数存在限制。
> - 如果你使用的是 DirSync，该限制为 15000 个用户。
> - 如果你使用的是 Azure AD Connect，该限制则为 50000 个用户。

## 访问面板

- 对应用程序的数量没有限制，应用程序可在每位订阅 Azure AD Premium 或 Azure AD Basic 或企业移动套件的最终用户“访问面板”中查看。
- 最多 10 个预先集成的 SaaS 应用程序（例如：Box、Salesforce）可在每位使用 Azure Active Directory 免费版的最终用户的访问面板中查看。如果组织开发的应用程序在后来与 Azure Active Directory 集成，则最终用户可在访问面板中查看 10 个以上应用程序。此限制不适用于管理员帐户。

## 报告

在报告中最多可查看或下载 1,000 行。系统会截断其他任何数据。

## 后续步骤
- [以组织身份注册 Azure](/documentation/articles/sign-up-organization)
- [Azure 订阅与 Azure AD 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory)
- [Azure AD 术语](/documentation/articles/active-directory-terminology)

<!---HONumber=60-->