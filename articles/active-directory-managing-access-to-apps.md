<properties
  pageTitle="使用 Azure AD 管理对应用的访问 | Azure"
  description="介绍 Azure Active Directory 如何使组织能够指定每个用户有权访问的应用。"
  services="active-directory"
  documentationCenter=""
  authors="msStevenPo"
  manager="stevenpo"
  editor=""/>

 <tags
  ms.service="active-directory"
  ms.date="02/26/2016"
  wacn.date="06/24/2016"/>


# 管理对应用的访问

将应用集成到组织的标识系统之后，进行中的访问管理、使用评估和报告将持续成为一项挑战。在许多情况下，IT 管理员或支持人员需要对应用访问管理持续采取主动措施。有时，分配由一般或分部 IT 小组执行。分配决策往往由业务决策人委派，在 IT 人员进行分配之前需要其批准。其他组织会投资来与现有自动化标识与访问管理系统集成，例如基于角色的访问控制 (RBAC) 或基于属性的访问控制 (ABAC)。集成与规则开发往往是针对性的且费用高昂。对任一管理方式进行监视或报告本身是一项独立、昂贵且复杂的投资。

## Azure Active Directory 有何帮助？

 Azure AD 针对已配置的应用程序支持广泛的访问管理，使组织能够轻松实现正确的访问策略，范围包括自动的基于分配的策略（ABAC 或 RBAC 方案）到委派和纳入管理员管理。有了 Azure AD，你可以轻松地实现复杂策略，结合单个应用程序的多个管理模型，甚至在具有相同对象的应用程序之间重复使用管理规则。

 - [添加新的或现有的应用程序](/documentation/articles/active-directory-sso-integrate-saas-apps/)


 Azure AD 的应用程序分配着重于两种主要分配模式：

- **单独分配**：具有目录全局管理员权限的 IT 管理员可以选择单个用户帐户并授予这些用户访问应用程序的权限。
- **基于组的分配（仅限付费版 Azure AD）**：具有目录全局管理员权限的 IT 管理员可以将组分配给应用程序。特定用户的访问权限取决于用户尝试访问应用程序时是否为组的成员。换而言之，管理员可以有效地创建分配规则，指明“分配组的任何现有成员都有权访问应用程序”。使用此分配选项，管理员可以受益于 Azure AD 组管理选项，外部系统组（例如，本地 Active Directory 或 Workday）、管理员管理的组或自助管理组。可以轻松地将单个组分配给多个应用程序，确保具有分配关联性的应用程序可以共享分配规则，从而降低总体管理复杂性。请注意，目前不支持使用嵌套的组成员身份来对应用程序进行基于组的分配。

管理员可以使用这两种分配模式实现任何所需的分配管理方案。

借助 Azure AD，使用和分配报告可完全集成，使管理员能够轻松报告分配状态、分配错误甚至使用情况。

## 使用 Azure AD 进行复杂应用程序分配

以 Salesforce 之类的应用程序为例。在许多组织中，Salesforce 主要由营销和销售部门使用。通常，营销团队成员对 Salesforce 拥有较高的访问权限，而销售团队的访问权限则受限。在许多情况下，广泛的信息工作者对应用程序的访问权限会受到限制。这些规则存在的例外使情况变得复杂。营销或销售领导团队通常有特权授予用户访问权限，或独立于这些常规规则单独更改用户角色。

使用 Azure AD 可将 Salesforce 等应用程序预先配置为支持单一登录 (SSO) 和自动化预配。配置应用程序后，管理员可以执行一次性的操作来创建和分配相应的组。在本示例中，管理员可以执行以下分配：


- 若要启用例外机制，可为每个角色创建自助组。例如，可以将“Salesforce 营销例外”组创建为自助服务组。可以为该组分配 Salesforce 营销角色，而营销领导团队可成为所有者。营销领导团队的成员可以添加或删除用户、设置加入策略甚至批准或拒绝单个用户的加入请求。信息工作者只需有相应的经验，而无需经过专门的所有者或成员培训就能做到这一点。

在此情况下，所有分配的用户将自动预配到 Salesforce，因为当他们添加到不同组时，他们的角色分配会在 Salesforce 中更新。用户可以通过 Microsoft 应用程序访问面板、Office Web 客户端甚至通过浏览到其组织的 Salesforce 登录页来发现和访问 Salesforce。管理员可以使用 Azure AD 报告轻松查看使用情况和分配状态。

 
## 如何入门？

首先，如果你未用过 Azure AD 并且你是 IT 管理员：

 - [立即试用！](/pricing/1rmb-trial-full/?v=c&form-type=waitinglist)- 你现在就可以使用此链接注册 30 天免费试用版，然后在不到 5 分钟内部署第一个云解决方案

支持帐户共享的 Azure AD 功能包括：

- [组分配](/documentation/articles/active-directory-accessmanagement-self-service-group-management/)
- 将应用程序添加到 Azure AD
- 分配入门
- 应用程序分配常见问题
- [应用使用情况仪表板/报告](/documentation/articles/active-directory-passwords-get-insights/)

## 可以从何处了解详细信息？

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)


<!---HONumber=Mooncake_0620_2016-->