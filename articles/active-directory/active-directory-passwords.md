<properties
    pageTitle="Azure AD 自助服务密码重置概述 | Azure"
    description="Azure AD 中的自助服务密码重置对组织有什么用处？"
    services="active-directory"
    documentationcenter=""
    author="MicrosoftGuyJFlo"
    manager="femila" />
<tags
    ms.assetid=""
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/05/2017"
    wacn.date="06/12/2017"
    ms.author="joflore"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="899054fed6e201a07bb9aa8908e05639854f56c0"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-ad-self-service-password-reset-for-the-it-professional"></a>面向 IT 专业人员的 Azure AD 自助服务密码重置

“自助服务”是在世界各地的众多 IT 部门中兴起的一个流行用语，其含义因情况而异。 当前市场上有大量产品都允许从云或本地管理本地组、密码或用户配置文件。

凭借着易于使用和部署，Azure Active Directory (Azure AD) 自助服务密码重置 (SSPR) 脱颖而出。 Azure AD 自助服务密码重置组合了一系列功能，它们：

- 允许用户通过以下方式管理其自己的密码
  - 从任何设备
  - 在任何位置
  - 在任何时间
- 遵循你以管理员身份定义的策略

## <a name="what-is-possible"></a>可以实现的功能

- **自助密码更改**允许最终用户或管理员更改其密码，不需要管理员协助
- **自助服务帐户解锁**允许最终用户解锁其自己的帐户，不需要管理员协助
- **自助服务密码重置**允许最终用户或管理员自动重置其密码，不需要管理员协助。 自助服务密码重置需要 Azure AD 高级版或基本版 - [Azure Active Directory 版本](/documentation/articles/active-directory-editions/)。
- **管理员启动的密码重置**允许管理员通过 Azure 门户预览重置最终用户或其他管理员的密码

## <a name="why-choose-azure-ad-self-service-password-reset"></a>为何选择 Azure AD 自助服务密码重置

- **降低成本**，因为由支持人员协助的密码重置通常占 IT 组织 20% 的费用
- 通过允许最终用户立即解决其自己的密码问题而不需要呼叫支持人员或发出支持请求，**改进了最终用户体验**并**减小了支持人员规模**。
- **促进移动**，因为用户可以从任何位置重置其密码。

## <a name="azure-ad-self-service-password-reset-availability"></a>Azure AD 自助服务密码重置可用性

Azure AD 自助服务密码重置在三个层中可用，具体取决于订阅。

- Azure AD 免费版 - 仅限云的管理员可以重置自己的密码
- Azure AD 基本版或任何付费版 Office 365 订阅 - 仅限云的用户和仅限云的管理员可以重置其自己的密码
- Azure AD 高级版 - 任何用户或管理员（包括仅限云、联合或密码同步的用户）都可以重置其自己的密码。 本地密码要求启用密码写回。

## <a name="azure-ad-self-service-password-reset-a-sum-of-the-parts"></a>Azure AD 自助服务密码重置，一系列部件的组合

Azure AD 中的自助服务密码重置由以下组件构成：

- 密码管理配置门户，可以在其中控制有关如何通过 Azure 门户预览在租户中管理密码的选项
- 密码重置注册门户，用户可以在其中自助注册密码重置
- 密码重置门户，用户可以在其中使用由管理员定义的质询和用户已提供的答案来重置其密码
- 用户密码更改门户，用户可以在其中通过输入旧密码并提供新密码来更改其自己的密码
- 密码管理报表，用户可以在其中查看和分析其在 Azure 门户预览中的租户的密码活动报表
- 使用 Azure AD Connect 将密码写回到本地，用于通过云实现对本地、联合或密码同步用户的管理

## <a name="azure-ad-pricing-sla-updates-and-roadmap"></a>Azure AD 定价、SLA、更新和路线图

可以在以下页面找到有关这些主题的更多详细信息

- [**Azure AD 定价详细信息**](/pricing/details/identity/)
- [**Office 365 定价**](https://products.office.com/compare-all-microsoft-office-products?tab=2)
- [**Azure 服务级别协议**](/support/legal/sla/)
- [**Microsoft Online Services 的服务级别协议**](http://go.microsoft.com/fwlink/?LinkID=272026&clcid=0x409)
- [**Azure 更新**](https://azure.microsoft.com/updates/)
- [**Azure 路线图**](https://www.microsoft.com/cloud-platform/roadmap-recently-available)

<!---Update_Description: wording update -->