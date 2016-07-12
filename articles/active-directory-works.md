<properties 
	pageTitle="Azure Active Directory 的工作原理 | Azure"
	description="Azure Active Directory 可在云中创建属于你的标识布局。它可以连接到你的本地标识系统，你也可以单独使用它。"
	services="active-directory" 
	documentationCenter="" 
	authors="curtand" 
	manager="terrylan" 
	editor=""/>

<tags 
	ms.service="active-directory" 
	ms.date="01/05/2016"
	wacn.date="06/03/2016"/>



# Azure Active Directory 的工作原理

###本主题的其他相关文章

> [AZURE.SELECTOR]
- [什么是 Azure AD](/documentation/articles/active-directory-whatis/)
- [它的工作原理是怎样的？](/documentation/articles/active-directory-works/)
- [入门](/documentation/articles/active-directory-get-started/)
- [后续步骤](/documentation/articles/active-directory-next-steps/)
- [了解详细信息](/documentation/articles/active-directory-learn-map/)


Azure Active Directory (Azure AD) 可在云中创建属于你的标识布局。它可以连接到你的本地标识系统，你也可以单独使用它。

你可以将 Azure AD 中的帐户看作是云中的驾照：它是你用来访问联机服务的唯一 ID。从这种意义上讲，Azure AD 类似于云中颁发这些驾照的私用注册机构。它可以实现在云中任何位置使用相应的标识，并可提高访问本地资源的用户的移动性。 

> [AZURE.NOTE] 若要使用 Azure Active Directory，你需要一个 Azure 帐户。如果你没有帐户，可以[注册 Azure 试用帐户](/pricing/1rmb-trial)。

## Azure AD 如何为 Office 365、Microsoft Intune 和其他 Azure 服务提供支持？
Azure 门户、Office 365 管理中心、Microsoft Intune 帐户门户和 Azure AD PowerShell 模块中的 cmdlet 都在与你目录关联的 Azure AD 的单个共享实例中读取和写入数据。门户（或 cmdlet）充当前端接口，它可以输入或更改目录信息。
## Azure AD 如何为第三方应用程序提供支持？
Azure AD 提供标识即服务，并提供针对不同平台的开放源代码库来帮助你快速编程，从而简化了开发人员的身份验证。[了解有关 Azure AD 的身份验证方案的详细信息](/documentation/articles/active-directory-authentication-scenarios/)。


## Azure AD 如何扩展我的本地目录？
Azure AD 支持多个最广泛使用的身份验证和授权协议。[了解有关 Azure Active Directory 身份验证协议的详细信息](https://msdn.microsoft.com/zh-CN/library/azure/dn151124.aspx)。 

## Azure 如何帮助我管理标识？
想要了解有关如何管理 Azure AD 的详细信息？如何获取目录？如何删除目录？如何管理目录数据？了解有关管理 Azure AD 目录的详细信息。[了解有关如何管理 Azure AD（主题待定）]。

## 其他资源

* [以组织身份注册 Azure](/documentation/articles/sign-up-organization/)

* [Azure 标识](/documentation/articles/fundamentals-identity/)


<!---HONumber=Mooncake_0620_2016-->