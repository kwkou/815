<properties
	pageTitle="Azure 监视器中的角色、权限和安全性入门 | Azure"
	description="了解如何使用 Azure 监视器的内置角色和权限来限制对监视资源的访问。"
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  


<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/26/2016"
	wacn.date="11/14/2016"
	ms.author="johnkem"/>  


# Azure 监视器中的角色、权限和安全性入门

许多团队需要严格管制对监视数据与设置的访问。例如，如果你的团队成员专门从事监视工作（技术支持工程师、开发运营工程师），或者你要与托管服务提供商合作，则可以只授予他们监视数据的访问权限，同时限制他们创建、修改或删除资源。本文说明如何在 Azure 中快速将内置监视 RBAC 角色应用到用户，或针对需要有限监视权限的用户构建自己的自定义角色。然后，本文介绍 Azure 监视器相关资源的安全注意事项，以及如何限制对这些资源中的数据的访问。

## 内置监视角色

Azure 监视器的内置角色旨在帮助限制对订阅中资源的访问，同时仍然允许负责监视基础结构的人员获取和配置他们所需的数据。Azure 监视器提供两个现成的角色：“监视读取者”和“监视参与者”。

### 监视读取者

拥有“监视读取者”角色的人员可以查看订阅中的所有监视数据，但无法修改任何资源或编辑与监视资源相关的任何设置。此角色适用于组织中的用户，例如技术支持工程师或运营工程师，这些人员必须能够：

- 在门户中查看监视仪表板，以及创建自己的专用监视仪表板。
- 使用 [Azure 监视器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931930.aspx)、[PowerShell cmdlet](/documentation/articles/insights-powershell-samples/) 或[跨平台 CLI](/documentation/articles/insights-cli-samples/) 查询指标。
- 使用门户、Azure 监视器 REST API、PowerShell cmdlets 或跨平台 CLI 查询活动日志。
- 查看资源的[诊断设置](/documentation/articles/monitoring-overview-of-diagnostic-logs/#diagnostic-settings)。
- 查看订阅的[日志配置文件](/documentation/articles/monitoring-overview-activity-logs/#export-the-activity-log-with-log-profiles)。
- 查看自动缩放设置。
- 查看警报活动和设置。
- 访问 Application Insights 数据，查看 AI Analytics 中的数据。
- 搜索 Log Analytics (OMS) 工作区数据，包括工作区的使用情况数据。
- 查看 Log Analytics (OMS) 管理组。
- 检索 Log Analytics (OMS) 搜索架构。
- 列出 Log Analytics (OMS) 智能包。
- 检索和执行 Log Analytics (OMS) 的已保存搜索。
- 检索 Log Analytics (OMS) 存储配置。

> [AZURE.NOTE] 此角色无法对已流式传输到事件中心或存储在存储帐户中的日志数据授予读取访问权限。请[参阅下文](#security-considerations-for-monitoring-data)，了解如何配置对这些资源的访问权限。

### 监视参与者

拥有“监视参与者”角色的人员可以查看订阅中的所有监视数据，以及创建或修改监视设置，但无法修改其他任何资源。此角色是“监视读取者”角色的超集，适用于组织中的监视团队成员或托管服务提供商，这些人员除了上述权限外，还必须能够：

- 将监视仪表板发布为共享仪表板。
- 设置资源的[诊断设置](/documentation/articles/monitoring-overview-of-diagnostic-logs/#diagnostic-settings)。*
- 设置订阅的[日志配置文件](/documentation/articles/monitoring-overview-activity-logs/#export-the-activity-log-with-log-profiles)。*
- 设置警报活动。
- 创建 Application Insights Web 测试和组件。
- 列出 Log Analytics (OMS) 工作区共享密钥。
- 启用或禁用 Log Analytics (OMS) 智能包。
- 创建、删除和执行 Log Analytics (OMS) 的已保存搜索。
- 创建和删除 Log Analytics (OMS) 存储配置。

*还必须单独向用户授予对目标资源（存储帐户或事件中心命名空间）的 ListKeys 权限，以便他们能够设置日志配置文件或诊断设置。

> [AZURE.NOTE] 此角色无法对已流式传输到事件中心或存储在存储帐户中的日志数据授予读取访问权限。请[参阅下文](#security-considerations-for-monitoring-data)，了解如何配置对这些资源的访问权限。

## 监视权限和自定义 RBAC 角色
如果上述内置角色不能满足团队的确切需求，可以使用更精细的权限[创建自定义的 RBAC 角色](/documentation/articles/role-based-access-control-custom-roles/)。下面是常见的 Azure 监视器 RBAC 操作及其说明。

| 操作 | 说明 |
|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| Microsoft.Insights/AlertRules/[Read, Write, Delete] | 读取/写入/删除警报规则。 |
| Microsoft.Insights/AlertRules/Incidents/Read | 列出警报规则的事件（触发的警报规则历史记录）。仅适用于门户。 |
| Microsoft.Insights/AutoscaleSettings/[Read, Write, Delete] | 读取/写入/删除自动缩放设置。 |
| Microsoft.Insights/DiagnosticSettings/[Read, Write, Delete] | 读取/写入/删除诊断设置。 |
| Microsoft.Insights/eventtypes/digestevents/Read | 需要通过门户访问活动日志的用户必须拥有此权限。 |
| Microsoft.Insights/eventtypes/values/Read | 列出订阅中的活动日志事件（管理事件）。此权限适用于以编程方式和通过门户访问活动日志。 |
| Microsoft.Insights/LogDefinitions/Read | 需要通过门户访问活动日志的用户必须拥有此权限。 |
| Microsoft.Insights/MetricDefinitions/Read | 读取指标定义（资源的可用指标类型列表）。 |
| Microsoft.Insights/Metrics/Read | 读取资源的指标。 |

> [AZURE.NOTE] 若要访问警报、诊断设置和资源的指标，用户必须对资源类型和该资源的范围拥有读取访问权限。若要创建（“写入”）可在存储帐户中存档或流式传输到事件中心的诊断设置或日志配置文件，用户还必须对目标资源拥有 ListKeys 权限。

例如，可以使用上表，针对“活动日志读取者”创建类似于下面的自定义 RBAC 角色：

```powershell
$role = Get-AzureRmRoleDefinition "Reader"
$role.Id = $null
$role.Name = "Activity Log Reader"
$role.Description = "Can view activity logs."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Insights/eventtypes/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/mySubscription")
New-AzureRmRoleDefinition -Role $role 
```

## <a id="security-considerations-for-monitoring-data"></a>监视数据的安全注意事项
监视数据（尤其是日志文件）可能包含敏感信息，例如 IP 地址或用户名。Azure 中的监视数据采用三种基本形式：
1. 活动日志，描述 Azure 订阅中的所有控制面操作。
2. 资源发出的诊断日志。
3. 资源发出的指标。

这三种类型的数据都可以存储在存储帐户中或流式传输到事件中心，存储帐户和事件中心属于通用 Azure 资源。由于它们是通用资源，因此其创建、删除和访问权限往往保留给管理员。我们建议对监视相关的资源采取以下做法，防止不当使用：

- 专门为监视数据使用一个存储帐户。如果需要将监视数据分割到多个存储帐户中，切勿在监视数据与非监视数据之间共享一个存储帐户，否则可能会意外地让只需访问监视数据的用户（例如，第三方 SIEM）得以访问非监视数据。
- 为所有诊断设置专门使用一个服务总线或事件中心命名空间，原因同上。
- 通过将监视相关的存储帐户或事件中心保留在不同的资源组中来限制对它们的访问，并对监视角色[使用范围](/documentation/articles/role-based-access-control-what-is/#basics-of-access-management-in-azure)，将访问权限限定于该资源组。
- 如果用户只需访问监视数据，切勿在订阅范围向其授予对存储帐户或事件中心的 ListKeys 权限。应该在资源或资源组（如果有专用的监视资源组）范围向用户授予这些权限。

### 限制对监控相关的存储帐户的访问
当用户或应用程序需要访问存储帐户中的监视数据时，应该在包含监视数据、对 Blob 存储具有服务级只读访问权限的存储帐户中[生成帐户 SAS](https://msdn.microsoft.com/zh-cn/library/azure/mt584140.aspx)。在 PowerShell 中，相应的命令如下所示：

```powershell
$context = New-AzureStorageContext -ConnectionString "[connection string for your monitoring Storage Account]"
$token = New-AzureStorageAccountSASToken -ResourceType Service -Service Blob -Permission "rl" -Context $context
```

可将令牌提供给需要读取该存储帐户的实体，然后，该实体即可列出和读取该存储帐户的所有 Blob 中的数据。

或者，如果需要使用 RBAC 控制此权限，可以向该实体授予对该特定存储帐户的 Microsoft.Storage/storageAccounts/listkeys/action 权限。需要指定诊断设置或要设置可存档到存储帐户的日志配置文件的用户必须拥有此权限。例如，对于只需读取一个存储帐户的用户或应用程序，可以创建以下自定义 RBAC 角色：

```powershell
$role = Get-AzureRmRoleDefinition "Reader"
$role.Id = $null
$role.Name = "Monitoring Storage Account Reader"
$role.Description = "Can get the storage account keys for a monitoring storage account."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/storageAccounts/listkeys/action")
$role.Actions.Add("Microsoft.Storage/storageAccounts/Read")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/mySubscription/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/myMonitoringStorageAccount")
New-AzureRmRoleDefinition -Role $role 
```

> [AZURE.WARNING] ListKeys 权限可让用户列出主要和辅助存储帐户密钥。这些密钥可向用户授予对该存储帐户中所有已签名服务（Blob、队列、表、文件）的所有签名权限（读取、写入、创建 Blob、删除 Blob，等等）。建议尽可能地使用上述帐户 SAS。

### 限制对监控相关的事件中心的访问
可对事件中心采用类似的模式，但首先需要创建专用的侦听授权规则。如果要向只需侦听监视相关事件中心的应用程序授予访问权限，请执行以下操作：

1. 在针对只有侦听声明的流式处理监视数据创建的事件中心上创建共享访问策略。可以在门户中完成此操作。例如，可以将此策略称为“monitoringReadOnly”。 如果可能，可以直接将该密钥提供给使用者，并跳过下一步骤。
2. 如果使用者必须能够临时获取密钥，请向用户授予对该事件中心执行 ListKeys 操作的权限。需要指定诊断设置或者要设置可流式传输到事件中心的日志配置文件的用户必须拥有此权限。例如，可以创建一条 RBAC 规则：

   ```powershell
   $role = Get-AzureRmRoleDefinition "Reader"
   $role.Id = $null
   $role.Name = "Monitoring Event Hub Listener"
   $role.Description = "Can get the key to listen to an event hub streaming monitoring data."
   $role.Actions.Clear()
   $role.Actions.Add("Microsoft.ServiceBus/namespaces/authorizationrules/listkeys/action")
   $role.Actions.Add("Microsoft.ServiceBus/namespaces/Read")
   $role.AssignableScopes.Clear()
   $role.AssignableScopes.Add("/subscriptions/mySubscription/resourceGroups/myResourceGroup/providers/Microsoft.ServiceBus/namespaces/mySBNameSpace")
   New-AzureRmRoleDefinition -Role $role 
   ```


## 后续步骤
- [了解 Resource Manager 中的 RBAC 和权限](/documentation/articles/role-based-access-control-what-is/)
- [阅读 Azure 中的监视概述](/documentation/articles/monitoring-overview/)

<!---HONumber=Mooncake_1107_2016-->