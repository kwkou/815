<properties
	pageTitle="Azure AD Connect 同步：理解和自定义同步 | Azure"
	description="介绍 Azure AD Connect 同步的工作原理以及如何自定义。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags 
	ms.service="active-directory"
	ms.date="05/11/2016"
	wacn.date="06/14/2016"/>


# Azure AD Connect 同步：理解和自定义同步
Azure Active Directory Connect 同步服务（Azure AD Connect 同步）是 Azure AD Connect 的一个主要组件，负责与本地环境和云中 Azure AD 之间同步标识数据相关的所有操作。Azure AD Connect 同步是 DirSync、Azure AD Sync 和 Forefront Identity Manager 的后继版本，同时配置了 Azure Active Directory 连接器。

本主题是 **Azure AD Connect 同步**（也称为**同步引擎**）的主页，其中列出了与其相关的所有其他主题的链接。有关 Azure AD Connect 的链接，请参阅[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)。

同步服务包括两个组件，本地“Azure AD Connect 同步”组件和 Azure AD 中称为“Azure AD Connect 同步服务”的服务端组件。该服务是 DirSync、Azure AD Sync 和 Azure AD Connect 的常见服务。

## Azure AD Connect 同步主题

| 主题 | 涵盖的内容和阅读时机 |
| ----- | ----- |
| **Azure AD Connect 同步基础知识** ||
| [技术概念](/documentation/articles/active-directory-aadconnectsync-technical-concepts/) | 精简版的体系结构主题，简要解释所用的术语。 |
| [Azure AD Connect 的拓扑](/documentation/articles/active-directory-aadconnect-topologies/) | 介绍同步引擎支持的各种拓扑和方案。 |
| **自定义配置** ||
| [了解用户和联系人](/documentation/articles/active-directory-aadconnectsync-understanding-users-and-contacts/) | 延续前一个主题，并说明用户和联系人的配置如何一起工作（尤其是在多林环境中）。 |
| [了解声明性设置表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions/) | 深入了解配置模型的工作原理和表达式语言的语法。 |
| [更改默认配置的最佳做法](/documentation/articles/active-directory-aadconnectsync-best-practices-changing-default-configuration/) | 如果你了解上述主题的详细信息并需要更改现成的配置，以配合方案或要求使用时，可以阅读此主题。 |
| [配置筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/) | 介绍有关如何限制哪些对象同步到 Azure AD 的各种选项，并逐步说明如何配置这些选项。 |
| **功能** ||
| [实现密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/) | 介绍密码同步的工作原理、实现方式，及其操作与故障排除方法。 |
| [防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes/) | 介绍防止意外删除功能以及如何对其进行配置。 |
| **操作** ||
| [操作任务和注意事项](/documentation/articles/active-directory-aadconnectsync-operations/) | 描述操作注意事项，例如灾难恢复。 |
| **详细信息和参考** ||
| [端口](/documentation/articles/active-directory-aadconnect-ports/) | 列出需要在同步引擎以及本地目录与 Azure AD 之间打开的端口。 |
| [与 Azure Active Directory 同步的属性](/documentation/articles/active-directory-aadconnectsync-attributes-synchronized/) | 列出在本地 AD 与 Azure AD 之间同步的所有属性。 |
| [函数引用](/documentation/articles/active-directory-aadconnectsync-functions-reference/) | 列出声明性预配中可用的所有函数。 |

## 其他资源

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---HONumber=Mooncake_0606_2016-->