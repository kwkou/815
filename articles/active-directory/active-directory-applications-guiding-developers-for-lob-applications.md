<properties
    pageTitle="为 Azure AD 开发 LOB 应用 | Azure"
    description="本文专门为 IT 专业人员编写，提供有关将 Azure 应用程序与 Active Directory 集成的指导。"
    services="active-directory"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="" />
<tags
    ms.assetid="dd69f2bc-37c5-457c-857d-27acb84267fb"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="03/07/2017"
    ms.author="kgremban" />  


# Azure AD 和应用程序：开发业务线应用
本指南概要介绍如何开发适用于 Azure Active Directory (AD) 的业务线 (LoB) 应用程序，目标受众为 Active Directory/Office 365 全局管理员。

## 概述
构建集成 Azure AD 的应用程序可让组织的用户使用 Office 365 进行单一登录。将应用程序置于 Azure AD 中即可控制应用程序的身份验证策略。

注册应用程序以使用 Azure Active Directory。注册应用程序意味着开发人员可以使用 Azure AD 对用户进行身份验证并请求访问用户资源（如电子邮件、日历和文档）。

目录的任何成员（不是来宾）都可以注册应用程序，也称为*创建应用程序对象*。

注册应用程序后，可让任何用户执行以下操作：

- 获取 Azure AD 识别的应用程序标识
- 获取应用程序可用于向 AD 验证其身份的一个或多个机密/密钥
- 在 Azure 门户预览中使用自定义名称、徽标等指定应用程序的品牌
- 对应用应用 Azure AD 授权功能，包括：

  - 基于角色的访问控制 (RBAC)
  - 使用 Azure Active Directory 作为 oAuth 授权服务器（保护应用程序公开的 API）
- 声明让应用程序按预期运行所需的权限，包括：

      - 应用权限（仅限全局管理员）。例如：另一个 Azure AD 应用程序中的角色成员身份，或相对于 Azure 资源、资源组或订阅的角色成员身份
      - 委派的权限（任何用户）。例如：Azure AD、登录和读取配置文件

> [AZURE.NOTE]
默认情况下，任何成员都可以注册应用程序。若要了解如何限定只有特定成员拥有注册应用程序的权限，请参阅 [How applications are added to Azure AD](/documentation/articles/active-directory-how-applications-are-added/#who-has-permission-to-add-applications-to-my-azure-ad-instance/)（如何将应用程序添加到 Azure AD）。
>
>

下面是全局管理员帮助开发人员将应用程序为生产做好准备所需执行的操作：

- 配置访问规则（访问策略/MFA）
- 将应用配置为要求用户分配，并分配用户
- 抑制默认的用户同意体验


## 将应用配置为要求用户分配，并分配用户
默认情况下，用户无需分配即可访问应用程序。不过，如果应用程序公开角色或者你希望应用程序显示在用户的访问面板上，则应该要求用户分配。

[要求用户分配](/documentation/articles/active-directory-applications-guiding-developers-requiring-user-assignment/)

如果你是 Azure AD Premium 或 Enterprise Mobility Suite (EMS) 的订阅者，我们强烈建议使用组。将组分配到应用程序可让你将持续进行的访问管理委派给组所有者。你可以创建组，或使用组管理功能请求组织中负责人创建组。

[将用户分配到应用程序](/documentation/articles/active-directory-applications-guiding-developers-assigning-users/)

## 抑制用户同意
默认情况下，每个用户通过同意体验进行登录。对于不太熟悉如何做出此类决定的用户而言，同意体验（要求用户授予对应用程序的权限）可能会令其不安。

对于你信任的应用程序，可以代表组织来同意应用程序，以简化用户体验。

有关 Azure 中的用户同意和同意体验的详细信息，请参阅 [Integrating Applications with Azure Active Directory](/documentation/articles/active-directory-integrating-applications/)（将应用程序与 Azure Active Directory 集成）。

## 相关文章
- [使用 Azure AD 管理对应用的访问](/documentation/articles/active-directory-managing-access-to-apps/)
- [有关 Azure Active Directory 中应用程序管理的文章索引](/documentation/articles/active-directory-apps-index/)

<!---HONumber=Mooncake_0227_2017-->
<!---Update_Description: wording update -->