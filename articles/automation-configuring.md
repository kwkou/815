<properties
   pageTitle="配置 Azure 自动化"
   description="介绍在配置初次使用的 Azure 自动化时必须执行的步骤。"
   services="automation"
   documentationCenter=""
   authors="SnehaGunda"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="02/09/2016"
   wacn.date="03/22/2016" />

# 配置 Azure 自动化

本文介绍在首次开始使用 Azure 自动化时必须执行的操作。

## 自动化帐户

首次启动 Azure 自动化时，你必须创建至少一个自动化帐户。使用自动化帐户，可以将你的自动化资源（Runbook、资产、配置）与其他自动化帐户中包含的自动化资源相隔离。可以使用自动化帐户将自动化资源隔离到独立的逻辑环境中。例如，你可以在开发环境中使用一个帐户，在生产环境中使用另一个帐户。

每个自动化帐户的自动化资源与单个 Azure 区域相关联，但自动化帐户可以管理任何区域中的 Azure 服务。在不同区域中创建自动化帐户的主要原因是，你的策略要求数据和资源隔离到特定的区域。

>[AZURE.NOTE] 使用 Azure 经典门户创建的自动化帐户及其包含的资源无法在 Azure 门户中访问。如果你想要使用 Windows PowerShell 来管理这些帐户或其资源，必须使用 Azure 资源管理器模块。
>
>使用 Azure 门户创建的自动化帐户可以通过门户和 cmdlet 集进行管理。创建帐户后，在该帐户中创建和管理资源的方式没有差别。如果你打算继续使用 Azure 门户，则应该使用它而不是 Azure 经典门户来创建任何自动化帐户。


如果你的 Azure 帐户存在问题（例如逾期付款），则自动化帐户可能会挂起。在这种情况下，你无法访问帐户，任何正在运行的作业都将挂起，并且所有计划都将被禁用。你将可以查看帐户，但无法看到其中的任何资源。在纠正问题并启用自动化帐户后，你必须启用你的计划并重新启动已挂起的任何 Runbook。


## 配置对 Azure 资源的身份验证

使用 [Azure cmdlet](http://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx) 访问 Azure 资源时，需要针对你的 Azure 订阅提供身份验证。在 Azure 自动化中，这是使用你在 Azure Active Directory 中配置为订阅管理员的组织帐户完成的。然后，可以为此用户帐户创建[凭据](/documentation/articles/automation-credentials)，并将它与 Runbook 中的 [Add-AzureAccount](http://msdn.microsoft.com/zh-cn/library/azure/dn722528.aspx) 结合使用。

>[AZURE.NOTE] Microsoft 帐户（以前称为 LiveID）不能用于 Azure 自动化。

## 创建新的 Azure Active Directory 用户用于管理 Azure 订阅

1. 以你要管理的 Azure 订阅的服务管理员身份登录到 Azure 门户。
2. 选择“Active Directory”
3. 选择与你的 Azure 订阅关联的目录名称。如有必要，可以从“设置”>“订阅”>“编辑目录”更改此关联。
4. [创建新的 Active Directory 用户](http://msdn.microsoft.com/zh-cn/library/azure/hh967632.aspx)。为“用户类型”选择“你的组织中的新用户”，并且不要选择“启用 Multi-Factor Authentication”。
5. 记下该用户的完整名称和临时密码。
7. 选择“设置”>“管理员”>“添加”。
8. 键入你创建的用户的完整用户名。
9. 选择你希望该用户管理的订阅。
10. 从 Azure 注销，然后使用你刚创建的帐户重新登录。系统将提示你更改用户密码。
11. 为创建的用户帐户创建新的 [Azure 自动化凭据资产](/documentation/articles/automation-credentials)。“凭据类型”应该是“Windows PowerShell 凭据”。

## 创建自动化帐户

自动化帐户是 Azure 自动化资源的容器。它可以用来隔离环境，或者对工作流进行进一步的组织。如果你已创建自动化帐户，可以跳过此步骤。

1. 登录到 [Azure 经典门户](https://manage.windowsazure.cn)。

2. 在 Azure 经典门户中，单击“新建”>“管理”>“自动化帐户”

3. 在“添加自动化帐户”边栏选项卡中，配置自动化帐户详细信息。

>[AZURE.NOTE] 使用 Azure 经典门户创建自动化帐户时，不会让该帐户及其所有关联的资源回到经典管理门户中。

下面是要配置的参数的列表：

|参数 |说明 |
|:---|:---|
| Name | 自动化帐户的名称，必须是唯一值。 |
| 资源组 | 利用资源组可以轻松查看和管理相关的 Azure 资源。在 Azure 经典门户中，你可以为自动化帐户选择现有的资源组，也可以创建新的资源组，而在 Azure 管理门户中，所有的自动化帐户都会放置在默认资源组中。 |
| 订阅 | 从可用订阅的列表中选择订阅。 |
| 区域 | 该区域指定要将帐户中的自动化资源存储在哪个位置。你可以从列表中选择任何区域，这不会影响帐户的功能，但是，如果帐户所在区域靠近其他 Azure 资源的存储位置，则 Runbook 可以更快地执行。 |
| 帐户选项 | 此选项允许你选择在新的自动化帐户中创建哪些资源；选择“是”即可创建教程式的 Runbook。 |



>[AZURE.NOTE] 将通过经典管理门户创建的自动化帐户通过 Azure 经典门户[移到其他资源组](/documentation/articles/resource-group-move-resources)后，就再也不能在 Azure 经典门户中使用该自动化帐户，因为经典管理门户不支持 Azure 资源管理器 帐户。



## 在 Runbook 中使用凭据

可以使用 [Get-AutomationPSCredential](/documentation/articles/automation-credentials) 活动检索 Runbook 中的凭据，然后将它与 [Add-AzureAccount](http://msdn.microsoft.com/zh-cn/library/azure/dn722528.aspx) 结合使用以连接到你的 Azure 订阅。如果该凭据是多个 Azure 订阅的管理员，则你还应使用 [Select-AzureSubscription](http://msdn.microsoft.com/zh-cn/library/dn495203.aspx) 来指定正确的订阅。下面所示的 Windows PowerShell 通常出现在大多数 Azure 自动化 Runbook 的顶部。

    $cred = Get-AutomationPSCredential –Name "myuseraccount.partner.onmschina.cn"
	Add-AzureAccount -Environment AzureChinaCloud –Credential $cred
	Select-AzureSubscription –SubscriptionName "My Subscription"

应该在 Runbook 中任何[检查点](http://technet.microsoft.com/zh-cn/library/dn469257.aspx#bk_Checkpoints)的后面重复这些行。如果 Runbook 已挂起，然后在另一个工作线程上恢复，则它需要再次执行身份验证。

## 相关文章
- [Azure 自动化：使用 Azure Active Directory 向 Azure 进行身份验证](https://azure.microsoft.com/blog/2014/08/27/azure-automation-authenticating-to-azure-using-azure-active-directory)
 

<!---HONumber=Mooncake_0307_2016-->