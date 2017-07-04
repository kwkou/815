<properties
    pageTitle="使用 Azure Active Directory 管理应用程序 | Microsoft 文档"
    description="本文介绍将 Azure Active Directory 与本地、云和 SaaS 应用程序集成的好处。"
    services="active-directory"
    documentationcenter=""
    author="MarkusVi"
    manager="femila"
    translationtype="Human Translation" />
<tags
    ms.assetid="95b96f10-2d5c-4b78-8af8-d3657a24140f"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="04/06/2017"
    wacn.date="05/08/2017"
    ms.author="markvi"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="de6fa04134ad2d297a223b221ca70b29ddba0677"
    ms.lasthandoff="04/28/2017" />

# <a name="managing-applications-with-azure-active-directory"></a>使用 Azure Active Directory 管理应用程序
除了实际工作流或内容以外，企业对所有应用程序还有另外两个基本要求：

1. 为了提高工作效率，应用程序应该易于发现和访问
2. 为了支持安全性和监管，组织需要控制和监督可以访问每个应用程序的人以及正在实施访问的人

在云应用程序的语境中，使用标识来控制“*谁有权执行什么操作*”是实现此目的的最佳方式。

在计算术语中：

- “*谁*”称为“*标识*” - 用于管理用户和组
- *什么*被称为*访问管理* - 用于管理对受保护资源的访问

这两个组成部分统称为“*标识和访问管理 (IAM)*”，[Gartner](http://www.gartner.com/it-glossary/identity-and-access-management-iam) 小组将其定义为“*允许适当的人员在适当的时间出于适当的理由访问适当的资源的安全策略*”。

那么，这有什么问题呢？ 如果 *不* 使用集成式解决方案在一个位置管理 IAM：

- 标识管理员必须单独在所有应用程序中逐个创建和更新用户帐户，这是一项繁琐而耗时的活动。
- 用户必须记住多个凭据，才能访问他们需要使用的应用程序。 因此，用户往往会写下其密码或使用其他密码管理解决方案，这又带来了其他数据安全风险。
- 繁琐而耗时的活动减少了用户和管理员在可增加业务收益的业务活动上投入的时间。

那么，哪些因素通常会阻止企业采用集成式 IAM 解决方案呢？

- 每家组织都需要针对其自身的应用程序部署和改编大多数技术解决方案所基于的软件平台。
- 云应用程序的部署费用通常高于 IT 组织集成现有 IAM 解决方案的费用。
- 安全和监视工具需要额外的定制和集成才能实现全面的 E2E 方案。

## <a name="azure-active-directory-integrated-with-applications"></a>Azure Active Directory 与应用程序集成
Azure Active Directory 是 Microsoft 的综合性标识即服务 (IDaaS) 解决方案，可以：

- 启用 IAM 作为云服务 
- 提供中心访问管理、单一登录 (SSO) 及报告功能 
- 支持应用程序库中[数千个应用程序](https://azure.microsoft.com/marketplace/active-directory/)（包括 Salesforce、Google Apps、Box、Concur 等）的集成访问管理。 

借助 Azure Active Directory，为合作伙伴与客户（企业或消费者）发布的所有应用程序都具有相同的标识和访问管理功能。<br> 这可让你大幅降低运营成本。

如果需要实施未在应用程序库中列出的应用程序怎么办？ 尽管这要比为应用程序库中的应用程序配置 SSO 更耗时，但 Azure AD 提供的向导可以帮助你完成配置。

Azure AD 的价值“不只”体现在云应用程序方面。 你也可以通过提供安全的远程访问，将它用于本地应用程序。 有了安全的远程访问，你就不再需要 VPN 或其他传统的远程访问管理实施。

Azure AD 可为所有应用程序提供中心访问管理和单一登录 (SSO)，从而解决主要数据的安全和工作效率问题。

- 用户只需登录一次就能访问多个应用程序，从而腾出更多的时间来增加收入或者完成业务运营活动。
- 标识管理员可在一个位置管理对应用程序的访问。

这为用户和公司带来的好处十分明显。 接下来，我们具体了解一下它为标识管理员和组织带来的好处。

## <a name="integrated-application-benefits"></a>集成应用程序的好处
SSO 过程分为两个步骤：

- 身份验证：验证用户标识的过程。
- 授权：根据访问策略来决定是允许还是阻止对某个资源的访问。

使用 Azure AD 来管理应用程序和启用 SSO 时：

- 身份验证在用户本地（例如 AD）或 Azure AD 帐户中完成。
- 根据 Azure AD 分配和保护策略执行授权，确保一致的最终用户体验，不管应用程序有何内部功能，都让你能够在任何应用程序中添加分配、位置和 MFA 条件。

你必须知道，对目标应用程序实施授权的方式根据应用程序与 Azure AD 集成的方式而有所不同。

- **服务提供商预先集成的应用程序** ：与 Office 365 和 Azure 一样，这些应用程序直接构建在 Azure AD 的基础之上，并依赖于 Azure AD 来实现其全面的标识和访问管理功能。 通过目录信息和令牌颁发来实现对这些应用程序的访问。
- **Microsoft 预先集成的应用程序和自定义应用程序** ：这些应用程序是独立的云应用程序，它们依赖于一个内部应用程序目录，可以独立于 Azure AD 运行。 可以通过颁发映射到应用程序帐户的应用程序特定凭据来实现对这些应用程序的访问。 根据应用程序的功能，凭据可以是联合身份验证令牌，也可以是先前在应用程序中预配的帐户的用户名和密码。
- **本地应用程序** ：通过 Azure AD 应用程序代理发布的应用程序，该代理主要用于实现对本地应用程序的访问。 这些应用程序主要依赖于 Windows Server Active Directory 等本地目录。 若要访问这些应用程序，可以触发代理以便将应用程序内容传送给最终用户，同时遵守本地登录要求。

例如，如果某个用户加入了你的组织，你需要在 Azure AD 中为该用户创建一个帐户，使其能够执行主要的登录操作。 如果此用户需要访问托管应用程序（如 Salesforce），则你还需要在 Salesforce 中为此用户创建一个帐户并将该帐户链接到 Azure 帐户，这样才能使 SSO 正常工作。 当该用户离开你的组织时，建议你删除该用户以前所访问应用程序的 IAM 存储中的 Azure AD 帐户和所有对应帐户。

## <a name="access-detection"></a>访问检测
在现代企业中，IT 部门不一定了解正在使用的所有云应用程序。 Cloud App Discovery 与 Azure AD 的组合为你提供了检测这些应用程序的解决方案。

## <a name="account-management"></a>帐户管理
管理不同应用程序中的帐户一直以来都是由组织中 IT 或支持人员手动执行的过程。 Azure AD 可以全自动管理服务提供商集成的所有应用程序，以及 Microsoft 预先集成且支持自动用户预配或 SAML JIT 的应用程序中的帐户。

## <a name="automated-user-provisioning"></a>自动用户预配
某些应用程序提供了用于创建和删除（或停用）帐户的自动化接口。 如果提供商提供了这种接口，Azure AD 就会利用该接口。 由于管理任务会自动进行，因此可以降低你的运营成本；此外，由于减少了未授权访问的可能性，因此可以提高环境的安全性。

## <a name="access-management"></a>访问管理
借助 Azure AD，你可以使用个人或规则驱动的分配来管理对应用程序的访问。 你还可以将访问管理权委派给组织中的适当人员，确保以最适当的方式监督访问，减轻支持人员的负担。

## <a name="on-premises-applications"></a>本地应用程序
使用内置的应用程序代理，可以将本地应用程序发布给用户，使现代云应用程序提供一致的访问体验，同时受益于 Azure AD 监视、报告和安全功能。

## <a name="reporting-and-monitoring"></a>报告和监视
Azure AD 提供预先集成的报告和监视功能，使你能够知道谁有权访问应用程序，以及他们使用应用程序的实际时间。

## <a name="related-capabilities"></a>相关功能
借助 Azure AD，可以使用精细的访问策略和预先集成的 MFA 来保护应用程序。 若要了解有关 Azure MFA 的详细信息，请参阅 [Azure MFA](/home/features/multi-factor-authentication/)。

## <a name="getting-started"></a>入门
若要开始将应用程序与 Azure AD 集成，请参阅[将 Azure Active Directory 与应用程序集成入门指南](/documentation/articles/active-directory-integrating-applications-getting-started/)。

## <a name="see-also"></a>另请参阅
[有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)

<!---Update_Description: wording update -->