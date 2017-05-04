<properties
    pageTitle="Azure Active Directory PoC 演练手册的构建基块 | Azure"
    description="研究并快速实现标识和访问管理方案"
    services="active-directory"
    keywords="azure active directory, 演练手册, 概念证明, PoC"
    documentationcenter=""
    author="dstefanMSFT"
    manager="asuthar"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="4/12/2017"
    wacn.date="05/02/2017"
    ms.author="dstefan"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="5198eb391cb441acf42f73756c8444e3a9d7476c"
    ms.lasthandoff="04/22/2017" />

# <a name="azure-active-directory-proof-of-concept-playbook-building-blocks"></a>Azure Active Directory 概念证明演练手册：构建基块

## <a name="catalog-of-actors"></a>执行组件目录

| 执行组件 | 说明 | PoC 责任 |
| --- | --- | --- |
| **标识体系结构/开发团队** | 此团队通常是设计解决方案、实现原型、推动审批以及最后将解决方案转交给运营部门的团队 | 他们提供环境并从管理角度评估不同的方案 |
| **本地标识运营团队** | 管理不同的本地标识源：Active Directory 林、LDAP 目录、HR 系统和联合标识提供程序。 | 提供 PoC 方案所需的本地资源的访问权限。<br/>应该尽量减少与此团队的交互|
| **应用程序技术所有者** | 要与 Azure AD 集成的不同云应用和服务的技术所有者 | 提供有关 SaaS 应用程序（可能是用于测试的实例）的详细信息 |
| **Azure AD 全局管理员** | 管理 Azure AD 配置 | 提供用于配置同步服务的凭据。 在 PoC 过程中，该团队通常就是标识体系结构团队，但在运营阶段，它们是两个不同的团队|
| **数据库团队** | 数据库基础结构的所有者 | 提供对 SQL 环境（ADFS 或 Azure AD Connect）的访问权限，以完成特定的方案准备工作。<br/>应该尽量减少与此团队的交互 |
| **网络团队** | 网络基础结构的所有者 | 在网络级别提供所需的访问权限，使同步服务器能够正常访问数据源和云服务（防火墙规则、打开的端口、IPSec 规则等。） |
| **安全团队** | 定义安全策略，分析来自各种源的安全报告，跟进结果。 | 提供目标安全性评估方案 |

## <a name="common-prerequisites-for-all-building-blocks"></a>所有构建基块的一般先决条件

| 先决条件 | 资源 |
| --- | --- |
| 使用有效 Azure 订阅定义的 Azure AD 租户 | [如何获取 Azure Active Directory 租户](/documentation/articles/active-directory-howto-tenant/)<br/>**注意：**如果你已创建一个具有 Azure AD Premium 许可证的环境，可以导航到 https://aka.ms/accessaad 获取一个没有上限的订阅 <br/>在以下网页中了解详细信息：https://blogs.technet.microsoft.com/enterprisemobility/2016/02/26/azure-ad-mailbag-azure-subscriptions-and-azure-ad-2/ 和 https://technet.microsoft.com/zh-cn/library/dn832618.aspx |
| Azure AD 全局管理员凭据 | 在 Azure Active Directory 中分配管理员角色 |
| 可选但强烈建议：用作应急措施的并行实验室环境 | [Azure AD Connect 的先决条件](/documentation/articles/active-directory-aadconnect-prerequisites/) |

## <a name="directory-synchronization---password-hash-sync-phs---new-installation"></a>目录同步 - 密码哈希同步 (PHS) - 新安装

大约完成时间：一小时（如果 PoC 用户数不超过 1,000 个）

### <a name="pre-requisites"></a>先决条件

| 先决条件 | 资源 |
| --- | --- |
| 用于运行 Azure AD Connect 的服务器 | [Azure AD Connect 的先决条件](/documentation/articles/active-directory-aadconnect-prerequisites/) |
| 在同一个域中并且属于某个安全组 POC 用户，以及 OU | [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/#domain-and-ou-filtering/) |
| 已确定 POC 所需的 Azure AD Connect 功能 | [将 Active Directory 连接到 Azure Active Directory - 配置同步功能](/documentation/articles/active-directory-aadconnect/#configure-sync-features/) |
| 已准备好本地和云环境所需的凭据  | [Azure AD Connect：帐户和权限](/documentation/articles/active-directory-aadconnect-accounts-permissions/) |

### <a name="steps"></a>步骤

| 步骤 | 资源 |
| --- | --- |
| 下载最新版本的 Azure AD Connect | [下载 Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594) |
| 使用最简单的路径安装 Azure AD Connect：快速安装 <br/>1.根据目标 OU 进行筛选，以最小化同步周期时间<br/>2.在本地组中选择一组目标用户。<br/>3.部署其他 POC 主题所需的功能 | [Azure AD Connect：自定义安装：域和 OU 筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/#domain-and-ou-filtering/) <br/>[Azure AD Connect：自定义安装：基于组的筛选](/documentation/articles/active-directory-aadconnect-get-started-custom/#sync-filtering-based-on-groups/)<br/>[Azure AD Connect：将本地标识与 Azure Active Directory 集成：配置同步功能](/documentation/articles/active-directory-aadconnect/#configure-sync-features/) |
| 打开 Azure AD Connect UI，查看正在运行的配置文件是否已完成（导入、同步和导出） | [Azure AD Connect 同步：计划程序](/documentation/articles/active-directory-aadconnectsync-feature-scheduler/) |

### <a name="considerations"></a>注意事项

1. 在[此处](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)查看密码哈希同步的安全注意事项。  如果试点生产用户的密码哈希同步明显不是一个选项，请考虑以下替代方法：
   - 在生产域中创建测试用户。 确保不要同步任何其他帐户
   - 移到 UAT 环境
2.    如果想要追求联合，有必要了解使用本地标识提供者关联联合解决方案以及 POC 的成本，并根据想要获得的优势权衡这种做法：
    - 该解决方案在关键路径中，因此必须采用高可用性设计
    - 它是需要规划容量的本地服务
    - 它是需要监视/维护/修补的本地服务

了解详细信息：[了解 Office 365 标识和 Azure Active Directory - 联合标识](https://support.office.com/article/Understanding-Office-365-identity-and-Azure-Active-Directory-06a189e7-5ec6-4af2-94bf-a22ea225a7a9#bk_federated)

## <a name="generic-ldap-connector-configuration"></a>通用 LDAP 连接器配置

大约完成时间：60 分钟

> [AZURE.IMPORTANT]
> 这是一项高级配置，需要对 FIM/MIM 有一定的了解。 如果在生产环境中使用，我们建议浏览[顶级支持](https://support.microsoft.com/zh-cn/premier)了解有关此配置的问题。

### <a name="pre-requisites"></a>先决条件

| 先决条件 | 资源 |
| --- | --- |
| 已安装并配置 Azure AD Connect | 构建基块：[目录同步 - 密码哈希同步](#directory-synchronization--password-hash-sync-phs--new-installation) |
| 符合要求的 ADLDS 实例 | [通用 LDAP 连接器技术参考：通用 LDAP 连接器概述](/documentation/articles/active-directory-aadconnectsync-connector-genericldap/#overview-of-the-generic-ldap-connector/) |
| 用户使用的工作负荷列表以及与这些工作负荷关联的属性 | [Azure AD Connect 同步：与 Azure Active Directory 同步的属性](/documentation/articles/active-directory-aadconnectsync-attributes-synchronized/) |


### <a name="steps"></a>步骤

| 步骤 | 资源 |
| --- | --- |
| 添加通用 LDAP 连接器 | [通用 LDAP 连接器技术参考：创建新连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericldap/#create-a-new-connector/) |
| 为创建的连接器创建运行配置文件（完整导入、增量导入、完整同步、增量同步、导出） | [创建管理代理运行配置文件](https://technet.microsoft.com/zh-cn/library/jj590219(v=ws.10).aspx)<br/> [将连接器与 Azure AD Connect Sync Service Manager 配合使用](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-connectors/)|
| 运行完整导入配置文件，并验证连接器空间中是否有对象 | [搜索连接器空间对象](https://technet.microsoft.com/zh-cn/library/jj590287(v=ws.10).aspx)<br/>[将连接器与 Azure AD Connect Sync Service Manager 配合使用：搜索连接器空间](/documentation/articles/active-directory-aadconnectsync-service-manager-ui-connectors/#search-connector-space/) |
| 创建同步规则，使 Metaverse 中的对象具有工作负荷的必要属性 | [Azure AD Connect 同步：有关更改默认配置的最佳做法：对同步规则的更改](/documentation/articles/active-directory-aadconnectsync-best-practices-changing-default-configuration/#changes-to-synchronization-rules/)<br/>[Azure AD Connect 同步：了解声明性预配](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning/)<br/>[Azure AD Connect 同步：了解声明性预配表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions/) |
| 启动完整同步周期 | [Azure AD Connect 同步：计划程序：启动计划程序](/documentation/articles/active-directory-aadconnectsync-feature-scheduler/#start-the-scheduler/) |
| 出现问题时执行故障排除 | [排查对象无法同步到 Azure AD 的问题](/documentation/articles/active-directory-aadconnectsync-troubleshoot-object-not-syncing/) |
| 验证 LDAP 用户是否可以登录及访问应用程序 | https://login.partner.microsoftonline.cn |

### <a name="considerations"></a>注意事项

> [AZURE.IMPORTANT]
> 这是一项高级配置，需要对 FIM/MIM 有一定的了解。 如果在生产环境中使用，我们建议浏览[顶级支持](https://support.microsoft.com/zh-cn/premier)了解有关此配置的问题。


## <a name="configuring-certificate-based-authentication"></a>配置基于证书的身份验证

大约完成时间：20 分钟

### <a name="pre-requisites"></a>先决条件

| 先决条件 | 资源 |
| --- | --- |
| 通过企业 PKI 预配的、包含用户证书的设备（Windows、iOS 或 Android） | [部署用户证书](https://msdn.microsoft.com/zh-cn/library/cc770857.aspx) |
| 与 ADFS 联合的 Azure AD 域 | [Azure AD Connect 和联合身份验证](/documentation/articles/active-directory-aadconnectfed-whatis/)<br/>[Active Directory 证书服务概述](https://technet.microsoft.com/zh-cn/library/hh831740.aspx)|
| 已经为 iOS 设备安装了 Microsoft 验证器应用 | [Microsoft 验证器应用入门](/documentation/articles/microsoft-authenticator-app-how-to/) |

### <a name="steps"></a>步骤

| 步骤 | 资源 |
| --- | --- |
| 在 ADFS 中启用“证书身份验证” | [配置身份验证策略：在 Windows Server 2012 R2 中全局配置主要身份验证](https://technet.microsoft.com/windows-server-docs/identity/ad-fs/operations/configure-authentication-policies#to-configure-primary-authentication-globally-in-windows-server-2012-r2) |
| 可选：在适用于 Exchange Active Sync 客户端的 Azure AD 中启用证书身份验证 | [Azure Active Directory 中基于证书的身份验证入门](/documentation/articles/active-directory-certificate-based-authentication-get-started/) |
| 导航到访问面板并使用用户证书进行身份验证 | https://login.partner.microsoftonline.cn |

### <a name="considerations"></a>注意事项

若要详细了解这种部署的注意事项，请访问：[ADFS：使用 Azure AD 和 Office 365 进行证书身份验证](https://blogs.msdn.microsoft.com/samueld/2016/07/19/adfs-certauth-aad-o365/)

> [AZURE.NOTE]
> 应该保护用户证书资产。 通过管理设备或者使用 PIN 码（如果在智能卡中存储）保护证书。



[AZURE.INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]

