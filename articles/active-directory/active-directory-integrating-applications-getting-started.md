<properties
    pageTitle="将 Azure AD 与应用集成入门 | Azure"
    description="本文是一篇入门指南，介绍如何将 Azure Active Directory (AD) 与本地应用程序和云应用程序集成。"
    services="active-directory"
    documentationcenter=""
    author="MarkusVi"
    manager="femila"
    editor="" />
<tags
    ms.assetid="db6d210d-c970-49e9-bd20-36d984bcd1c3"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="05/04/2017"
    wacn.date="06/12/2017"
    ms.author="markvi"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="685f3e274c63dbc792e86d6930705777a597e7b7"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="integrating-azure-active-directory-with-applications-getting-started-guide"></a>将 Azure Active Directory 与应用程序集成入门指南
## <a name="overview"></a>概述
本主题旨在提供有关将应用程序与 Azure Active Directory (AD) 集成的路线图。 下面每个部分包含更详细主题的简短摘要，以便你识别本入门指南中与你相关的各个部分。  请单击链接深入学习每个主题。

## <a name="before-you-begin-take-inventory"></a>开始前的盘点工作
在开始将应用程序与 Azure AD 集成之前，你必须知道所在的位置，以及要前往的位置。  以下问题旨在帮助你思考 Azure AD 应用程序集成项目。

### <a name="application-inventory"></a>应用程序盘点
- 所有应用程序的所在位置？ 其所有者是谁？
- 应用程序需要哪种身份验证？
- 谁需要访问哪些应用程序？
- 是否要部署新应用程序？
  - 是否要在公司内部构建应用程序并将其部署在 Azure 计算实例上？
  - 是否要使用 Azure 应用程序库中提供的某个应用程序？

### <a name="user-and-group-inventory"></a>用户和组盘点
- 用户帐户位于哪个位置？
  - 本地 Active Directory
  - Azure AD
  - 在你拥有的独立应用程序数据库中
  - 在未经认可的应用程序中
  - 以上都是
- 各个用户当前有哪些权限和角色分配？ 是否需要审查其访问权限，或者你是否确定用户当前的访问和角色分配适当？
- 是否已在本地 Active Directory 中建立组？
  - 组的组织方式如何？
  - 有哪些组成员？
  - 组当前有哪些权限/角色分配？
- 是否需要在集成之前清理用户/组数据库？  （垃圾的进出是一个 相当重要的问题。）

### <a name="access-management-inventory"></a>访问管理盘点
- 你当前如何管理用户对应用程序的访问？ 是否需要做出更改？  是否考虑过使用其他方式来管理访问，例如使用 [RBAC](/documentation/articles/role-based-access-control-configure/)？
- 谁需要访问哪些应用程序？

也许你暂时无法回答所有这些问题，但没有关系。  本指南可帮助你回答其中一些问题并做出明智的决策。

## <a name="prerequisites"></a>先决条件
- 一个 Azure 订阅和一个 Azure Active Directory 目录。  如果还没有 Azure 订阅，可以尝试 Azure 1RMB 试用版。 [立即试用！](/pricing/1rmb-trial/)

## <a name="application-integration-with-azure-ad"></a>将应用程序与 Azure AD 集成
### <a name="finding-unsanctioned-cloud-applications-with-cloud-app-discovery"></a>使用 Cloud App Discovery 查找未经认可的云应用程序
如上所述，可能有些应用程序到目前为止仍不受你组织的管理。在盘点过程中，你可以查找未经认可的云应用程序。

### <a name="authentication-types"></a>身份验证类型
每个应用程序可能有不同的身份验证要求。 借助 Azure AD，可对使用 SAML 2.0、WS 联合身份验证或 OpenID Connect 协议以及密码单一登录的应用程序使用签名证书。 有关可用于 Azure AD 的应用程序身份验证类型的详细信息，请参阅[在 Azure Active Directory 中管理用于联合单一登录的证书](/documentation/articles/active-directory-sso-certs/)和[基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/)。

### <a name="integrating-applications-with-azure-ad"></a>将应用程序与 Azure AD 集成
以下文章介绍了将应用程序与 Azure AD 集成的不同方法，并提供了一些指导。

- [确定要使用的 Active Directory](/documentation/articles/active-directory-administer/)
- [使用 Azure 应用程序库中的应用程序](/documentation/articles/active-directory-appssoaccess-whatis/)

## <a name="managing-access-to-applications"></a>管理对应用程序的访问
以下文章介绍了在使用 Azure AD 连接器和 Azure AD 将应用程序与 Azure AD 集成之后，如何管理对应用程序的访问。

- [使用 Azure AD 管理对应用的访问](/documentation/articles/active-directory-managing-access-to-apps/)
- [将用户分配到应用程序](/documentation/articles/active-directory-applications-guiding-developers-assigning-users/)
- [共享帐户](/documentation/articles/active-directory-sharing-accounts/)

## <a name="integrating-custom-applications"></a>集成自定义应用程序
如果正在编写新应用程序，并想要协助开发人员利用 Azure AD 的强大功能，请参阅[指导开发人员](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)。

如果你想要将自定义应用程序添加到 Azure 应用程序库，请参阅 [使用 Azure AD 自助 SAML 配置加入自己的应用](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)。

## <a name="see-also"></a>另请参阅
- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)

<!---Update_Description: wording update -->