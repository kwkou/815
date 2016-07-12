<properties
   pageTitle="将 Azure Active Directory 与应用程序集成入门指南 | Azure"
   description="本文是一篇入门指南，介绍如何将 Azure Active Directory (AD) 与本地应用程序和云应用程序集成。"
   services="active-directory"
   documentationCenter=""
   authors="ihenkel"
   manager="stevenpo"
   editor=""/>

<tags
      ms.service="active-directory"
      ms.date="02/09/2016"
      wacn.date="06/23/2016"/>


# 将 Azure Active Directory 与应用程序集成入门指南
## 概述
本主题旨在提供有关将应用程序与 Azure Active Directory (AD) 集成的路线图。下面每个部分包含更详细主题的简短摘要，使你可以认识本入门指南的哪些部分与你相关。请单击链接深入学习每个主题。

## 开始前的盘点工作
在开始将应用程序与 Azure AD 集成之前，你必须知道你所在的位置，以及你要前往的位置。以下问题旨在帮助你思考 Azure AD 应用程序集成项目。

### 应用程序盘点
- 所有应用程序的所在位置？ 其所有者是谁？
- 应用程序需要哪种身份验证？
- 谁需要访问哪些应用程序？
- 是否要部署新应用程序？
  - 是否要在公司内部构建应用程序并将其部署在 Azure 计算实例上？
  - 是否要使用 Azure 应用程序库中提供的某个应用程序？

### 用户和组盘点
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
- 是否需要在集成之前清理用户/组数据库？ （垃圾的进出是一个相当重要的问题。）

### 访问管理盘点
- 你当前如何管理用户对应用程序的访问？ 是否需要做出更改？ 你是否考虑过使用其他方式来管理访问，例如使用 [RBAC](/documentation/articles/role-based-access-control-configure/)？
- 谁需要访问哪些应用程序？

也许你暂时无法回答所有这些问题，但没有关系。本指南可帮助你回答其中一些问题并做出明智的决策。

## 先决条件
- 一个 Azure 订阅和一个 Azure Active Directory 目录。如果你还没有 Azure 订阅，可以尝试注册 Azure 30 天试用版。[试试看！](/pricing/1rmb-trial-full/?v=c&form-type=waitinglist)

## 将应用程序与 Azure AD 集成
### 使用 Cloud App Discovery 查找未经认可的云应用程序
如上所述，可能有些应用程序到目前为止仍不受你组织的管理。在盘点过程中，你可以查找未经认可的云应用程序。请参阅[使用 Cloud App Discovery 查找未经认可的云应用程序](/documentation/articles/active-directory-cloudappdiscovery-whatis/)。

### 身份验证类型
每个应用程序可能有不同的身份验证要求。借助 Azure AD，可对使用 SAML 2.0、WS 联合身份验证或 OpenID Connect 协议以及密码单一登录的应用程序使用签名证书。有关可用于 Azure AD 的应用程序身份验证类型的详细信息，请参阅[在 Azure Active Directory 中管理用于联合单一登录的证书](/documentation/articles/active-directory-sso-certs/)和[基于密码的单一登录](/documentation/articles/active-directory-appssoaccess-whatis/)。

### 使用 Azure AD 应用代理启用 SSO
使用 Microsoft Azure AD 应用程序代理，可以从任何位置和任何设备安全访问专用网络中的应用程序。在环境中安装应用程序代理连接器后，可以使用 Azure AD 轻松配置该连接器。

### 将应用程序与 Azure AD 集成
以下文章介绍了将应用程序与 Azure AD 集成的不同方法，并提供了一些指导。

- [确定要使用的 Active Directory](/documentation/articles/active-directory-administer/)
- [使用 Azure 应用程序库中的应用程序](/documentation/articles/active-directory-appssoaccess-whatis/)
- [集成 SaaS 应用程序教程列表](/documentation/articles/active-directory-saas-tutorial-list/)


## 管理对应用程序的访问
以下文章介绍了在使用 Azure AD 连接器和 Azure AD 将应用程序与 Azure AD 集成之后，如何管理对应用程序的访问。

- [使用 Azure AD 管理对应用的访问](/documentation/articles/active-directory-managing-access-to-apps/)
- [使用 Azure AD 连接器自动化](/documentation/articles/active-directory-saas-app-provisioning/)
- 使用 Azure 经典管理门户[将用户分配到应用程序](/documentation/articles/active-directory-applications-guiding-developers-assigning-users/)。
- 使用 Azure 经典管理门户[将组分配到应用程序](/documentation/articles/active-directory-applications-guiding-developers-assigning-groups/)。
- [共享帐户](/documentation/articles/active-directory-sharing-accounts/)

## 集成自定义应用程序
如果你正在编写新应用程序，并想要协助开发人员利用 Azure AD 的强大功能，请参阅[指导开发人员](/documentation/articles/active-directory-applications-guiding-developers-for-lob-applications/)。

如果你想要将自定义应用程序添加到 Azure 应用程序库，请参阅[使用 Azure AD 自助 SAML 配置加入自己的应用](http://blogs.technet.com/b/ad/archive/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-gt-now-in-preview.aspx)。

## 另请参阅

- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)


<!---HONumber=Mooncake_0613_2016-->