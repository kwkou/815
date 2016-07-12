<properties 
   pageTitle="在 Azure Automation 中计划 Runbook"
   description="介绍如何在 Azure Automation 中创建一个计划，以便在特定的时间或按重复计划自动启动 Runbook。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags
	ms.service="automation"
	ms.date="02/03/2016"
	wacn.date="06/29/2016"/>

# 在 Azure 自动化中计划 Runbook

若要将 Azure 自动化中的 Runbook 计划为在指定的时间启动，可以将它链接到一个或多个计划。可以将某个计划配置为运行一次，或者每隔指定的小时数或天数重复运行。可将一个 Runbook 链接到多个计划，一个计划可以链接多个 Runbook。

## 创建计划

可以使用 Azure 经典管理门户或 Windows PowerShell 创建新的计划。也可以在使用 Azure 经典管理门户将 Runbook 链接到计划时，选择创建新的计划。

### 使用 Azure 经典管理门户创建新计划

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 选择“资产”选项卡。
1. 在窗口底部，单击“添加设置”。
1. 单击“添加计划”。
1. 键入新计划的“名称”和（可选）“说明”。
1. 选择计划的运行频率：“一次”、“每小时”或“每日”。
1. 指定“开始时间”，并根据选择的计划类型指定其他选项。

### 使用 Windows PowerShell 创建新计划

你可以使用 [New-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/azure/dn690271.aspx) cmdlet 在 Azure 自动化中创建新计划。必须指定计划的开始时间，以及它是要运行一次、每小时还是每天运行。

以下示例命令演示了如何创建一个从 2015 年 1 月 20 日开始在每天下午 3:30 运行的新计划。

	$automationAccountName = "MyAutomationAccount"
	$scheduleName = "Sample-DailySchedule"
	New-AzureAutomationSchedule –AutomationAccountName $automationAccountName –Name $scheduleName –StartTime "1/20/2015 15:30:00" –DayInterval 1

## 将计划链接到 Runbook

可将一个 Runbook 链接到多个计划，一个计划可以链接多个 Runbook。如果 Runbook 包含参数，则你可以提供这些参数的值。必须为所有必需参数提供值，并可以为任何可选参数提供值。此计划每次启动 Runbook 时，将使用这些值。你可以将同一个 Runbook 附加到另一个计划，并指定不同的参数值。

### 使用 Azure 经典管理门户将计划链接到 Runbook

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 选择“Runbook”选项卡。
1. 单击要计划的 Runbook 的名称。
1. 单击“计划”选项卡。
2. 如果 Runbook 当前未链接到计划，则系统会提供“链接到新计划”或“链接到现有计划”选项。如果 Runbook 当前已链接到计划，请单击窗口底部的“链接”来访问这些选项。
1. 如果 Runbook 包含参数，系统将提示你输入其值。  

### 使用 Windows PowerShell 将计划链接到 Runbook

你可以使用 [Register-AzureAutomationScheduledRunbook](http://msdn.microsoft.com/zh-cn/library/azure/dn690265.aspx) 将计划链接到 Runbook。可以使用 Parameters 参数指定 Runbook 参数的值。有关指定参数值的详细信息，请参阅[在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook/)。

以下示例命令演示了如何将计划链接到包含参数的 Runbook。

	$automationAccountName = "MyAutomationAccount"
	$runbookName = "Test-Runbook"
	$scheduleName = "Sample-DailySchedule"
	$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
	Register-AzureAutomationScheduledRunbook –AutomationAccountName $automationAccountName –Name $runbookName –ScheduleName $scheduleName –Parameters $params

## 禁用计划

禁用某个计划后，链接到该计划的所有 Runbook 将不再按该计划运行。可以手动禁用计划，也可以在创建每小时和每日计划时设置其过期时间。到达过期时间时，将会禁用该计划。

### 使用 Azure 经典管理门户禁用计划

可以在 Azure 经典管理门户中通过某个计划的“计划详细信息”页禁用该计划。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 选择“资产”选项卡。
1. 单击某个计划的名称以打开该计划的详细信息页。
2. 将“已启用”更改为“否”。

### 使用 Windows PowerShell 禁用计划

你可以使用 [Set-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/azure/dn690270.aspx) cmdlet 更改现有计划的属性。若要禁用计划，请为 **IsEnabled** 参数指定 **false**。

以下示例命令演示了如何禁用计划。

	$automationAccountName = "MyAutomationAccount"
	$scheduleName = "Sample-DailySchedule"
	Set-AzureAutomationSchedule –AutomationAccountName $automationAccountName –Name $scheduleName –IsEnabled $false

## 相关文章

- [在 Azure 自动化中计划资产](/documentation/articles/automation-schedules/)
- [在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook/) 

<!---HONumber=79-->