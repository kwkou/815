<properties
	pageTitle="Azure Monitor PowerShell 快速启动示例 | Azure"
	description="使用 PowerShell 访问 Azure Monitor 功能，例如自动缩放、警报、webhook 和搜索活动日志。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  


<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/06/2017"
	ms.author="ashwink"
	wacn.date="05/02/2017"/>  


# Azure Monitor PowerShell 快速启动示例
本文说明可帮助访问 Azure Monitor 功能的示例 PowerShell 命令。Azure Monitor 允许基于配置的遥测数据值自动缩放云服务、虚拟机和 Web 应用，以及发送警报通知或调用 Web URL。

> [AZURE.NOTE]
“Azure Insights”在 2016 年 9 月 25 日后称为 Azure Monitor。但是，命名空间和以下命令中仍然包含“insights”。
> 
> 

## 设置 PowerShell
如果尚未安装，请在你的计算机上安装要运行的 PowerShell。有关详细信息，请参阅[如何安装和配置 PowerShell](/documentation/articles/powershell-install-configure/)。

## 本文中的示例
本文中的示例演示如何使用 Azure Monitor cmdlet。还可以在 [Azure Monitor (Insights) Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt282452#40v=azure.200#41.aspx) 中查看 Azure Monitor PowerShell cmdlet 的完整列表。


## 登录并使用订阅

首先，登录 Azure 订阅。

	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

这要求你进行登录。执行此操作后，会显示帐户、TenantId 和默认的订阅 ID。所有 Azure cmdlet 都可用于默认订阅的上下文。若要查看有权访问的订阅的列表，请使用以下命令。

	Get-AzureRmSubscription

若要将工作上下文更改为其他订阅，请使用以下命令。

	Set-AzureRmContext -SubscriptionId <subscriptionid>


## 检索订阅的活动日志
使用 `Get-AzureRmLog` cmdlet。下面是一些常用示例。

从此时间/日期中获取要显示的日志条目︰

	Get-AzureRmLog -StartTime 2016-03-01T10:30

在一个时间/日期范围中获取日志条目︰

	Get-AzureRmLog -StartTime 2015-01-01T10:30 -EndTime 2015-01-01T11:30

从特定资源组中获取日志条目︰

	Get-AzureRmLog -ResourceGroup 'myrg1'

在一个时间/日期范围中从特定资源提供程序获取日志条目︰

	Get-AzureRmLog -ResourceProvider 'Microsoft.Web' -StartTime 2015-01-01T10:30 -EndTime 2015-01-01T11:30

获取特定调用方的所有日志项︰

    Get-AzureRmLog -Caller 'myname@company.com'

以下命令从活动日志中检索最后 1000 个事件：

	Get-AzureRmLog -MaxEvents 1000

`Get-AzureRmLog` 支持许多其他参数。有关详细信息，请参阅 `Get-AzureRmLog` 参考文档。

>[AZURE.NOTE] `Get-AzureRmLog` 仅提供 15 天的历史记录。使用 **-MaxEvents** 参数可查询 15 天之外的最后 N 个事件。要访问超过 15 天的事件，请使用 REST API 或 SDK （使用 SDK 的 C# 示例）。如果不包括 **StartTime**，则默认值为 **EndTime** 减去一小时。如果不包括 **EndTime**，则默认值为当前时间。所有时间均是 UTC 时间。

## 检索警报历史记录
若要查看所有警报事件，可以使用以下示例查询 Azure Resource Manager 日志。

	Get-AzureRmLog -Caller "Microsoft.Insights/alertRules" -DetailedOutput -StartTime 2015-03-01

若要查看特定警报规则的历史记录，可以使用 `Get-AzureRmAlertHistory` cmdlet，同时会传入警报规则的资源 ID。

		Get-AzureRmAlertHistory -ResourceId /subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/alertrules/myalert -StartTime 2016-03-1 -Status Activated

`Get-AzureRmAlertHistory` Cmdlet 支持各种参数。有关详细信息，请参阅 [Get-AlertHistory](https://msdn.microsoft.com/zh-cn/library/mt282453.aspx)。


## 检索关于警报规则的信息
下面的所有命令可用于名为“montest”的资源组。

查看警报规则的所有属性︰

	Get-AzureRmAlertRule -Name simpletestCPU -ResourceGroup montest -DetailedOutput

检索某个资源组的所有警报︰

	Get-AzureRmAlertRule -ResourceGroup montest

检索目标资源的所有警报规则设置。例如，虚拟机上的所有警报规则设置。

	Get-AzureRmAlertRule -ResourceGroup montest -TargetResourceId /subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig

`Get-AzureRmAlertRule` 支持其他参数。有关详细信息，请参阅 [Get-AlertRule](https://msdn.microsoft.com/zh-cn/library/mt282459.aspx)。

## <a name="create-alert-rules"></a>创建警报规则
可以使用 `Add-AlertRule` cmdlet 来创建、更新或禁用警报规则。

可以分别使用 `New-AzureRmAlertRuleEmail` 和 `New-AzureRmAlertRuleWebhook` 创建电子邮件和 webhook 属性。在警报规则 cmdlet 中，将这些作为操作分配给警报规则的 **Actions** 属性。

下一节中的示例演示了如何创建具有多个参数的警报规则。

### 指标的警报规则
下表描述了用于使用指标创建警报的参数和值。


|参数|value|
|---|---|
|名称|	simpletestdiskwrite|
|此警报规则的位置|	中国东部|
|resourceGroup|	montest|
|TargetResourceId|	/subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig|
|创建的警报的 MetricName|	\\PhysicalDisk(\_Total)\\Disk Writes/sec。请参阅以下 `Get-MetricDefinitions` cmdlet，了解如何检索确切的指标名称|
|operator|	GreaterThan|
|阈值（对于此指标，单位为计数/秒）|	1|
|WindowSize（格式为 hh:mm:ss）|	00:05:00|
|聚合器（指标的统计信息，在此情况下使用平均值计数）|	平均值|
|自定义电子邮件（字符串数组）|“foo@example.com”、“bar@example.com”|
|将电子邮件发送给所有者、参与者和读者|	-SendToServiceOwners|

创建电子邮件操作
		$actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com

创建 Webhook 操作

	$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://example.com?token=mytoken

在经典虚拟机上创建关于 CPU %指标的警报规则

	Add-AzureRmMetricAlertRule -Name vmcpu_gt_1 -Location "chinanorth" -ResourceGroup myrg1 -TargetResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.ClassicCompute/virtualMachines/my_vm1 -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 1 -WindowSize 00:05:00 -TimeAggregationOperator Average -Actions $actionEmail, $actionWebhook -Description "alert on CPU > 1%"

检索警报规则

	Get-AzureRmAlertRule -Name vmcpu_gt_1 -ResourceGroup myrg1 -DetailedOutput

如果给定属性已存在警报规则，则添加警报 cmdlet 还会更新该规则。若要禁用警报规则，请包括 **-DisableRule** 参数。

### 活动日志事件警报

>[AZURE.NOTE] 此功能目前处于预览状态，将在以后删除（即将替换）。

在此方案中，在资源组 *abhingrgtest123* 中的我的订阅中成功启动网站时，会发送电子邮件。

设置电子邮件规则

	$actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com

设置 webhook 规则

	$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://example.com?token=mytoken

针对事件创建规则


	Add-AzureRmLogAlertRule -Name superalert1 -Location "China East" -ResourceGroup myrg1 -OperationName microsoft.web/sites/start/action -Status Succeeded -TargetResourceGroup abhingrgtest123 -Actions $actionEmail, $actionWebhook


检索警报规则


	Get-AzureRmAlertRule -Name superalert1 -ResourceGroup myrg1 -DetailedOutput


`Add-AlertRule` Cmdlet 允许使用多个其他参数。有关详细信息，请参阅 [Add-AlertRule](https://msdn.microsoft.com/zh-cn/library/mt282468.aspx)。

## 获取警报的可用指标的列表
可以使用 `Get-AzureRmMetricDefinition` cmdlet 来查看针对特定资源的所有指标的列表。


	Get-AzureRmMetricDefinition -ResourceId <resource_id>


以下示例会生成一张表，其中包含指标名称和单位。


	Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit


`Get-AzureRmMetricDefinition` 的可用选项的完整列表位于 [Get MetricDefinitions](https://msdn.microsoft.com/zh-cn/library/mt282458.aspx) 中。


## <a name="create-and-manage-autoscale-settings"></a> 创建和管理自动缩放设置
资源（例如 Web 应用、虚拟机、云服务或 VM 缩放设置）只能有一种为其配置的自动缩放设置。但是，每个自动缩放设置可具有多个配置文件。例如，一个用于基于性能的缩放配置文件，另一个用于基于计划的配置文件。每个配置文件可以为其配置多个规则。有关自动缩放的详细信息，请参阅[如何自动缩放应用程序](/documentation/articles/cloud-services-how-to-scale/)。

下面列出了要使用的步骤：

1. 创建规则。
2. 创建配置文件，将之前创建的规则映射到该配置文件。
3. 可选︰通过配置 webhook 和电子邮件属性，创建自动缩放通知。
4. 通过映射在前面步骤中创建的配置文件和通知，创建自动缩放设置，并使用目标资源上的名称。

以下示例演示了如何使用 CPU 利用率指标为基于 Windows 操作系统的 VM 缩放设置创建自动缩放设置。

首先，创建向外扩展规则，实例计数增加。


	$rule1 = New-AzureRmAutoscaleRule -MetricName "\Processor(_Total)\% Processor Time" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 0.01 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Increase -ScaleActionScaleType ChangeCount -ScaleActionValue 1
		

接下来，创建向内扩展规则，实例计数减少。


	$rule2 = New-AzureRmAutoscaleRule -MetricName "\Processor(_Total)\% Processor Time" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 2 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Decrease -ScaleActionScaleType ChangeCount -ScaleActionValue 1


然后，为规则创建配置文件。


	$profile1 = New-AzureRmAutoscaleProfile -DefaultCapacity 2 -MaximumCapacity 10 -MinimumCapacity 2 -Rules $rule1,$rule2 -Name "My_Profile"


创建 webhook 属性。


	$webhook_scale = New-AzureRmAutoscaleWebhook -ServiceUri "https://example.com?mytoken=mytokenvalue"


创建自动缩放设置的通知属性，包括电子邮件和之前创建的 webhook。


	$notification1= New-AzureRmAutoscaleNotification -CustomEmails ashwink@microsoft.com -SendEmailToSubscriptionAdministrators SendEmailToSubscriptionCoAdministrators -Webhooks $webhook_scale


最后，创建自动缩放设置以添加上面创建的配置文件。


	Add-AzureRmAutoscaleSetting -Location "China East" -Name "MyScaleVMSSSetting" -ResourceGroup big2 -TargetResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -AutoscaleProfiles $profile1 -Notifications $notification1


有关管理自动缩放设置的详细信息，请参阅 [Get AutoscaleSetting](https://msdn.microsoft.com/zh-cn/library/mt282461.aspx)。

## 自动缩放历史记录
以下示例演示了如何查看近期的自动缩放和警报事件。使用活动日志搜索来查看自动缩放历史记录。


	Get-AzureRmLog -Caller "Microsoft.Insights/autoscaleSettings" -DetailedOutput -StartTime 2015-03-01


可以使用 `Get-AzureRmAutoScaleHistory` cmdlet 来检索自动缩放历史记录。


	Get-AzureRmAutoScaleHistory -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/microsoft.insights/autoscalesettings/myScaleSetting -StartTime 2016-03-15 -DetailedOutput


有关详细信息，请参阅 [Get-AutoscaleHistory](https://msdn.microsoft.com/zh-cn/library/mt282464.aspx)。

### 查看自动缩放设置的详细信息
可以使用 `Get-Autoscalesetting` cmdlet 来检索有关自动缩放设置的详细信息。

以下示例显示了关于资源组 myrg1 中所有自动缩放设置的详细信息。


	Get-AzureRmAutoscalesetting -ResourceGroup myrg1 -DetailedOutput


以下示例显示了关于资源组 myrg1 中所有自动缩放设置的详细信息，特别是名为 MyScaleVMSSSetting 的自动缩放设置的详细信息。


	Get-AzureRmAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting -DetailedOutput


### 删除自动缩放设置
可以使用 `Remove-Autoscalesetting` cmdlet 来删除自动缩放设置。


	Remove-AzureRmAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting


## 管理活动日志的日志配置文件
可以创建*日志配置文件*并从活动日志中将数据导出到存储帐户，并且可以为其配置数据保留期。也可以选择将数据流式传输到事件中心。请注意，此功能目前以预览版提供，每个订阅只能创建一个日志配置文件。可以通过当前订阅使用以下 cmdlet 来创建和管理日志配置文件。也可以选择一个特定订阅。虽然 PowerShell 默认使用当前订阅，但可以使用 `Set-AzureRmContext` 随时更改。可以配置活动日志将数据路由到该订阅中的任何存储帐户或事件中心。以 JSON 格式将数据写为 blob 文件。

### 获取日志配置文件
若要提取现有日志配置文件，请使用 `Get-AzureRmLogProfile` cmdlet。

### 添加没有数据保留期的日志配置文件


	Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Locations chinaeast,chinanorth


### 删除日志配置文件


	Remove-AzureRmLogProfile -name my_log_profile_s1


### 添加有数据保留期的日志配置文件

可以用天数将 **-RetentionInDays** 属性指定为一个正整数，会在此期间保留数据。


	Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Locations chinaeast,chinanorth


### 添加具有保留期和 EventHub 的日志配置文件
除了将数据路由到存储帐户，还可以流式传输到事件中心。请注意，在此预览版本中，存储帐户配置是必需的，但事件中心配置是可选的。


	Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Locations chinaeast,chinanorth


## 配置诊断日志
许多 Azure 服务都提供其他日志和遥测，这些日志和遥测可以配置为在 Azure 存储帐户中保存数据，并将数据发送到事件中心。该操作只能在资源级别执行，并且存储帐户或事件中心应与配置诊断设置的目标资源处于相同的区域中。

### 获取诊断设置


	Get-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp


禁用诊断设置


	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-chinaeast/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $false


启用没有保留期的诊断设置


	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-chinaeast/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true


启用有保留期的诊断设置


	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-chinaeast/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true -RetentionEnabled $true -RetentionInDays 90


为特定日志类别启用有保留期的诊断设置

	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/sakteststorage -Categories NetworkSecurityGroupEvent -Enable $true -RetentionEnabled $true -RetentionInDays 90

启用事件中心的诊断设置


	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-chinaeast/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Enable $true


启用 OMS 的诊断设置


	Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -WorkspaceId 76d785fd-d1ce-4f50-8ca3-858fc819ca0f -Enabled $true



<!---HONumber=Mooncake_1226_2016-->