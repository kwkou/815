<properties
    pageTitle="Azure Active Directory PoC 演练手册成分 | Azure"
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
    ms.openlocfilehash="48894228773f84072664c9b790e1c6367834aa4d"
    ms.lasthandoff="04/22/2017" />

# <a name="azure-active-directory-proof-of-concept-playbook-ingredients"></a>Azure Active Directory 概念证明演练手册成分 

## <a name="theme"></a>主题
Azure AD 为企业中的诸多方面提供标识和访问解决方案。 我们根据以下方面将方案分类： 

- [ 多个应用，一个标识](/documentation/articles/active-directory-playbook-implementation/#theme---lots-of-apps-one-identity/) 
- [提高安全性](/documentation/articles/active-directory-playbook-implementation/#theme---increase-your-security/) 
- [使用自助服务进行缩放](/documentation/articles/active-directory-playbook-implementation/#theme---scale-with-self-service/) 

定义一个用于限定 PoC 范围的主题有助于将工作重心放在如何与业务目标产生共鸣上，这通常是概念证明的前导需求触发器。 

## <a name="environment"></a>环境

必须确定要在其中提供 PoC 的环境的详细信息。 理想情况下，完成 PoC 后，可以基于该环境构建解决方案。 目标环境至关重要，你既要让它尽量真实，同时又要考虑约束产生的开销或者其他因素。 PoC 的典型环境包括：

- **生产：**方案将在已部署 Microsoft 云服务（生产 AD、Office 365、Azure AD 租户/SSO 解决方案）的实时环境中实施。 
- **用户验收测试 (UAT)/开发环境：**已部署包含测试数据的、可模拟生产环境的基础结构（并行 AD，可能还包括 Azure AD 租户/SSO 解决方案）。 通常，测试环境在企业中的多个项目之间共享。

本指南中的大多数方案在性质上是累加的。 因此，可在不影响 PoC 外部用户的情况下，将这些方案部署在生产租户中。 整篇文档将会概述哪些方案会产生租户范围的影响。 在这种情况下，你可能需要考虑非生产环境。 


## <a name="target-users"></a>目标用户

必须确定一组要演练这些方案的目标用户，尤其是在生产或测试环境中。 PoC 的目标用户类别包括：

- **试点用户：**环境中要通过其日常工作时所用的帐户使用解决方案的真实用户
- **测试用户：**在环境中创建的测试帐户 

本指南中的大多数方案可由试点用户演练。 整篇文档将会根据需要概述目标用户注意事项。


[AZURE.INCLUDE [active-directory-playbook-toc](../../includes/active-directory-playbook-steps.md)]

