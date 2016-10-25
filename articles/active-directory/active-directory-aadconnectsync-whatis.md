<properties
	pageTitle="Azure AD Connect 同步：理解和自定义同步 | Azure"
	description="介绍 Azure AD Connect 同步的工作原理以及如何自定义。"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/29/2016"
	ms.author="markusvi;andkjell"
	wacn.date="10/11/2016"/>


# Azure AD Connect 同步：理解和自定义同步
Azure Active Directory Connect 同步服务（Azure AD Connect 同步）是 Azure AD Connect 的一个主要组件。它负责在本地环境与 Azure AD 之间同步标识数据的所有相关操作。Azure AD Connect 同步是 DirSync、Azure AD Sync 和 Forefront Identity Manager 的后继版本，同时配置了 Azure Active Directory 连接器。

本主题是 **Azure AD Connect 同步**（也称为**同步引擎**）的主页，其中列出了与其相关的所有其他主题的链接。有关 Azure AD Connect 的链接，请参阅 [Integrating your on-premises identities with Azure Active Directory](/documentation/articles/active-directory-aadconnect/)（将本地标识与 Azure Active Directory 集成）。

同步服务包括两个组件，本地"Azure AD Connect 同步"组件和 Azure AD 中称为"Azure AD Connect 同步服务"的服务端组件。该服务是 DirSync、Azure AD Sync 和 Azure AD Connect 的常见服务。

## Azure AD Connect 同步主题

主题 | 涵盖的内容和阅读时机
----- | -----
**Azure AD Connect 同步基础知识** |
[了解体系结构](/documentation/articles/active-directory-aadconnectsync-understanding-architecture/) | 适合不熟悉同步引擎并想要深入了解所用体系结构和术语的人员。
[技术概念](/documentation/articles/active-directory-aadconnectsync-technical-concepts/) | 精简版的体系结构主题，简要解释所用的术语。
[Azure AD Connect 的拓扑](/documentation/articles/active-directory-aadconnect-topologies/) | 介绍同步引擎支持的各种拓扑和方案。
**自定义配置** |
[再次运行安装向导](/documentation/articles/active-directory-aadconnectsync-installation-wizard/) | 介绍再次运行 Azure AD Connect 安装向导时可以使用的选项。
[了解声明性预配](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning/)| 介绍称为"声明性预配"的配置模型。
[了解声明性设置表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions/) | 介绍声明性预配中使用的表达式语言的语法。
[了解默认配置](/documentation/articles/active-directory-aadconnectsync-understanding-default-configuration/)| 描述现成的规则和默认配置。此外还描述规则如何一起工作，以供现成的方案使用。
[了解用户和联系人](/documentation/articles/active-directory-aadconnectsync-understanding-users-and-contacts/) | 延续前一个主题，说明用户和联系人的配置如何一起工作（尤其是在多林环境中）。
[如何更改默认配置](/documentation/articles/active-directory-aadconnectsync-change-the-configuration/) | 逐步讲解如何对属性流做出常见的配置更改。
[更改默认配置的最佳做法](/documentation/articles/active-directory-aadconnectsync-best-practices-changing-default-configuration/) | 支持方面的限制，以及如何对现成配置进行更改。
[配置筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering/) | 介绍用于限制哪些对象要同步到 Azure AD 的各种选项，并逐步说明如何配置这些选项。
**功能和方案** |
[防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes/) | 介绍 *防止意外删除* 功能以及如何配置该功能。
[计划程序](/documentation/articles/active-directory-aadconnectsync-feature-scheduler/) | 介绍导入、同步和导出数据的内置计划程序。
[实现密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/) | 介绍密码同步的工作原理、实现方式，及其操作与故障排除方法。
[设备写回](/documentation/articles/active-directory-aadconnect-feature-device-writeback/) | 介绍设备写回在 Azure AD Connect 中的工作原理。
[目录扩展](/documentation/articles/active-directory-aadconnectsync-feature-directory-extensions/) | 介绍如何使用你自己的自定义属性扩展 Azure AD 架构。
**同步服务** |
[Azure AD Connect 同步服务功能](/documentation/articles/active-directory-aadconnectsyncservice-features/) | 介绍同步服务端，以及如何在 Azure AD 中更改同步设置。
[重复属性修复](/documentation/articles/active-directory-aadconnectsyncservice-duplicate-attribute-resiliency/) | 介绍如何启用并使用 **userPrincipalName** 和 **proxyAddresses** 重复属性值复原。
**操作和 UI** |
[Synchronization Service Manager](/documentation/articles/active-directory-aadconnectsync-service-manager-ui/) | 介绍 Synchronization Service Manager UI，包括["操作"](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-operations/)、["连接器"](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-connectors/)、["Metaverse 设计器"](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-mvdesigner/)和["Metaverse 搜索"](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-mvsearch/)选项卡。
[操作任务和注意事项](/documentation/articles/active-directory-aadconnectsync-operations/) | 描述操作注意事项，例如灾难恢复。
**如何...** |
[重置 Azure AD 帐户](/documentation/articles/active-directory-aadconnectsync-howto-azureadaccount/) | 如何重置用于从 Azure AD Connect 同步连接到 Azure AD 的服务帐户凭据。
**详细信息和参考** |
[端口](/documentation/articles/active-directory-aadconnect-ports/) | 列出需要在同步引擎以及本地目录与 Azure AD 之间打开的端口。
[与 Azure Active Directory 同步的属性](/documentation/articles/active-directory-aadconnectsync-attributes-synchronized/) | 列出在本地 AD 与 Azure AD 之间同步的所有属性。
[函数引用](/documentation/articles/active-directory-aadconnectsync-functions-reference/) | 列出声明性预配中可用的所有函数。

## 其他资源

* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

<!---HONumber=Mooncake_0926_2016-->
