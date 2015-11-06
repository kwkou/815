<properties 
   pageTitle="配置 Azure 自动化"
   description="介绍在配置初次使用的 Azure 自动化时必须执行的步骤。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="07/10/2015"
   wacn.date="09/15/2015" />

# 配置 Azure 自动化

本文介绍在首次开始使用 Azure 自动化时必须执行的操作。

## 自动化帐户

首次启动 Azure 自动化时，你必须创建至少一个自动化帐户。使用自动化帐户，可以将你的自动化资源（Runbook、资产）与其他自动化帐户中包含的自动化资源相隔离。可以使用自动化帐户将自动化资源隔离到独立的逻辑环境中。例如，你可以在开发环境中使用一个帐户，在生产环境中使用另一个帐户。

每个自动化帐户的自动化资源与单个 Azure 区域相关联，但自动化帐户可以管理任何区域中的 Azure 服务。在不同区域中创建自动化帐户的主要原因是，你的策略要求数据和资源隔离到特定的区域。

如果你的 Azure 帐户存在问题（例如逾期付款），则自动化帐户可能会挂起。在这种情况下，你无法访问帐户，任何正在运行的作业都将挂起，并且所有计划都将被禁用。你将可以查看帐户，但无法看到其中的任何资源。在纠正问题并启用自动化帐户后，你必须启用你的计划并重新启动已挂起的任何 Runbook。


## 配置对 Azure 资源的身份验证

使用 [Azure cmdlet](https://msdn.microsoft.com/zh-CN/library/azure/jj554330.aspx) 访问 Azure 资源时，需要针对你的 Azure 订阅提供身份验证。在 Azure 自动化中，这是使用你在 Azure Active Directory 中配置为订阅管理员的组织帐户完成的。然后，可以为此用户帐户创建[凭据](https://msdn.microsoft.com/zh-CN/library/dn940015.aspx)，并将它与 Runbook 中的 [Add-AzureAccount](https://msdn.microsoft.com/zh-CN/library/azure/dn722528.aspx) 结合使用。

>[AZURE.NOTE]Microsoft 帐户（以前称为 LiveID）不能用于 Azure 自动化。

## 创建新的 Azure Active Directory 用户用于管理 Azure 订阅

1. 以你要管理的 Azure 订阅的服务管理员身份登录到 Azure 门户。
2. 选择“Active Directory”
3. 选择与你的 Azure 订阅关联的目录名称。如有必要，可以从“设置”>“订阅”>“编辑目录”更改此关联。
4. [创建新的 Active Directory 用户](https://msdn.microsoft.com/zh-CN/library/azure/hh967632.aspx)。为“用户类型”选择“你的组织中的新用户”，并且不要选择“启用多重身份验证”。
5. 记下该用户的完整名称和临时密码。
7. 选择“设置”>“管理员”>“添加”。
8. 键入你创建的用户的完整用户名。
9. 选择你希望该用户管理的订阅。
10. 从 Azure 注销，然后使用你刚创建的帐户重新登录。系统将提示你更改用户密码。
11. 为创建的用户帐户创建新的 [Azure 自动化凭据资产](https://msdn.microsoft.com/zh-CN/library/dn940015.aspx)。“凭据类型”应该是“Windows PowerShell 凭据”。


## 在 Runbook 中使用凭据

可以使用 [Get-AutomationPSCredential](https://msdn.microsoft.com/zh-CN/library/dn940015.aspx) 活动检索 Runbook 中的凭据，然后将它与 [Add-AzureAccount](https://msdn.microsoft.com/zh-CN/library/azure/dn722528.aspx) 结合使用以连接到你的 Azure 订阅。如果该凭据是多个 Azure 订阅的管理员，则你还应使用 [Select-AzureSubscription](https://msdn.microsoft.com/zh-CN/library/dn495203.aspx) 来指定正确的订阅。下面所示的 Windows PowerShell 通常出现在大多数 Azure 自动化 Runbook 的顶部。

    $cred = Get-AutomationPSCredential –Name "myuseraccount.partner.onmschina.cn"
	Add-AzureAccount -Environment AzureChinaCloud –Credential $cred
	Select-AzureSubscription –SubscriptionName "My Subscription"

应该在 Runbook 中任何[检查点](/documentation/articles/automation-runbook-execution#checkpoints)的后面重复这些行。如果 Runbook 已挂起，然后在另一个工作线程上恢复，则它需要再次执行身份验证。

## 相关文章
- [Azure 自动化：使用 Azure Active Directory 向 Azure 进行身份验证](http://azure.microsoft.com/blog/2014/08/27/azure-automation-authenticating-to-azure-using-azure-active-directory/)
 

<!---HONumber=69-->