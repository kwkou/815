<properties
   pageTitle="Azure 自动化安全"
   description="本文提供了对 Azure 自动化中的 Automation 帐户安全性和可用的不同身份验证方法的概述。"
   services="automation"
   documentationCenter=""
   authors="MGoedtel"
   manager="jwhit"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="04/08/2016"
   wacn.date="05/30/2016" />

# Azure 自动化安全
Azure 自动化让你可以通过其他云提供程序（如 Amazon Web Services (AWS)）针对 Azure、本地中的资源来自动执行任务。为了使 Runbook 执行其所需的操作，Runbook 必须有权安全访问它所处理的资源，并且仅使用订阅中所需的最小权限。  
本文将介绍 Azure 自动化支持的各种安全方案，并介绍如何基于你需要管理的环境入门。

## 自动化帐户概述
首次启动 Azure 自动化时，你必须创建至少一个自动化帐户。使用自动化帐户，可以将你的自动化资源（Runbook、资产、配置）与其他自动化帐户中包含的资源相隔离。可以使用自动化帐户将资源隔离到独立的逻辑环境中。例如，你可以在开发环境中使用一个帐户，在生产环境中使用另一个帐户，并在本地环境中使用另一个账户。Azure 自动化帐户不同于 Microsoft 帐户或在你的 Azure 订阅中创建的帐户。

每个自动化帐户的自动化资源与单个 Azure 区域相关联，但自动化帐户可以管理任何区域中的资源。在不同区域中创建自动化帐户的主要原因是，你的策略要求数据和资源隔离到特定的区域。

所有使用 Azure Service Manager (ASM) 和 Azure 自动化中的 Azure cmdlet 对资源执行的任务必须使用 Azure Active Directory 组织标识基于凭据的身份验证向 Azure 进行身份验证。基于证书的身份验证是使用 Azure 服务管理 (ASM) 的原始身份验证方法，但是安装很复杂。在 2014 年引入了使用 Azure AD 用户向 Azure 进行身份验证，简化了配置身份验证帐户的过程。

## 身份验证方法

下表总结了适用于 Azure 自动化所支持的身份验证方法，文章描述了如何为你的 Runbook 设置身份验证。

方法 | 环境 | 文章
----------|----------|----------
Azure AD 用户帐户 | Azure 服务管理 | [Authenticate Runbooks with Azure AD User account（使用 Azure AD 用户帐户进行 Runbook 身份验证）](/documentation/articles/automation-sec-configure-aduser-account)
 |   |
 |   | 

<!---HONumber=Mooncake_0516_2016-->