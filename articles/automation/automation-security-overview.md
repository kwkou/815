<properties
    pageTitle="Azure 自动化安全性 | Azure"
    description="本文概述了 Azure 自动化中自动化帐户的自动化安全性以及可供使用的不同身份验证方法。"
    services="automation"
    documentationcenter=""
    author="MGoedtel"
    manager="jwhit"
    editor="tysonn"
    keywords="自动化安全性, 安全的自动化" />  

<tags
    ms.assetid="4a6bc2f5-c5a2-4dfb-b10d-7950d750dee8"
    ms.service="automation"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/01/2017"
    wacn.date="04/27/2017"
    ms.author="magoedte" />

# Azure 自动化安全性
Azure 自动化可让你自动针对 Azure 和本地中的资源执行任务。为了使 Runbook 执行所需操作，Runbook 必须有权使用订阅中所需的最小权限来安全地访问资源。本文将介绍 Azure 自动化支持的各种身份验证方案，并介绍如何根据你需要管理的单个或多个环境来入门。

## 自动化帐户概述
首次启动 Azure 自动化时，你必须创建至少一个自动化帐户。使用自动化帐户，可以将你的自动化资源（Runbook、资产、配置）与其他自动化帐户中包含的资源相隔离。可以使用自动化帐户将资源隔离到独立的逻辑环境中。例如，你可以在开发环境中使用一个帐户，在生产环境中使用另一个帐户，并在本地环境中使用另一个账户。Azure 自动化帐户不同于 Azure 帐户或在你的 Azure 订阅中创建的帐户。

每个自动化帐户的自动化资源都与单个 Azure 区域关联，但自动化帐户可以管理订阅中的所有资源。在不同区域中创建自动化帐户的主要原因是，你的策略要求数据和资源隔离到特定的区域。

所有使用 Azure Resource Manager 和 Azure 自动化中的 Azure cmdlet 对资源执行的任务必须使用 Azure Active Directory 组织标识基于凭据的身份验证向 Azure 进行身份验证。基于证书的身份验证是使用 Azure 服务管理模式的原始身份验证方法，但是安装很复杂。在 2014 年引入了使用 Azure AD 用户向 Azure 进行身份验证，不仅简化了配置身份验证帐户的过程，也支持使用在 Azure Resource Manager 和经典资源模式下均可使用的单个用户帐户向 Azure 进行非交互式身份验证的功能。

基于角色的访问控制在 Azure Resource Manager 中可用，向 Azure AD 用户帐户和运行方式帐户授予允许的操作，并对该服务主体进行身份验证。

## 身份验证方法
下表总结了适用于 Azure 自动化所支持的每个环境的不同身份验证方法，文章描述了如何为你的 Runbook 设置身份验证。

| 方法 | 环境 | 文章 |
| --- | --- | --- |
| Azure AD 用户帐户 |Azure 资源管理器和 Azure 服务管理 |[Authenticate Runbooks with Azure AD User account（使用 Azure AD 用户帐户进行 Runbook 身份验证）](/documentation/articles/automation-create-aduser-account/) |
| Azure 运行方式帐户 |Azure 资源管理器 |无 |
| Azure 经典运行方式帐户 |Azure 服务管理 |无 |

<!---HONumber=Mooncake_Quality_Review_0104_2017-->