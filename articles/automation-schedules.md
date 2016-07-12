<properties
   pageTitle="Azure 自动化中的计划 | Azure"
   description="自动化计划用于安排自动启动 Azure 自动化中的 Runbook。本文介绍如何创建计划。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="03/18/2016"
   wacn.date="04/20/2016" />

# Azure 自动化中的计划

自动化计划用于安排自动运行 Runbook。可能是在单一日期和时间运行一次 Runbook。也可以是用于多次启动 Runbook 的重复执行的每日或每小时计划。通常不从 Runbook 中访问计划。

## Windows PowerShell Cmdlet

下表中的 cmdlet 用于在 Azure 自动化中通过 Windows PowerShell 创建和管理计划。它们作为 [Azure PowerShell 模块](/documentation/articles/powershell-install-configure/)的一部分提供。

|Cmdlet|说明|
|:---|:---|
|[Get-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/dn690274.aspx)|检索计划。|
|[New-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/dn690271.aspx)|创建新计划。|
|[Remove-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/dn690279.aspx)|删除计划。|
|[Set-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/dn690270.aspx)|设置现有计划的属性。|
|[Get-AzureAutomationScheduledRunbook](http://msdn.microsoft.com/zh-cn/library/dn913778.aspx)|检索计划 Runbook。|
|[Register-AzureAutomationScheduledRunbook](http://msdn.microsoft.com/zh-cn/library/dn690265.aspx)|将 Runbook 与计划相关联。|
|[Unregister-AzureAutomationScheduledRunbook](http://msdn.microsoft.com/zh-cn/library/dn690273.aspx)|将 Runbook 与计划取消关联。|

## 创建新计划

### 使用 Azure 经典管理门户创建新计划


1. 在你的自动化帐户中，单击窗口顶部的“资产”。
1. 在窗口底部，单击“添加设置”。
1. 单击“添加计划”。
1. 完成向导并单击复选框以保存新计划。

### 使用 Windows PowerShell 创建新计划

[New-AzureAutomationSchedule](http://msdn.microsoft.com/zh-cn/library/dn690271.aspx) cmdlet 创建新计划并为现有计划设置值。下面的示例 Windows PowerShell 命令创建名为 My Daily Schedule 的新计划，该计划于明天中午上启动并在一年中的每一天启动：

	$automationAccountName = "MyAutomationAccount"
	$scheduleName = "My Daily Schedule"
	$startTime = (Get-Date).Date.AddDays(1).AddHours(12)
	$expiryTime = $startTime.AddYears(1)
	
	New-AzureAutomationSchedule –AutomationAccountName $automationAccountName –Name $scheduleName –StartTime $startTime –ExpiryTime $expiryTime –DayInterval 1


## 另请参阅
- [在 Azure 自动化中计划 Runbook](/documentation/articles/automation-scheduling-a-runbook/)
 

<!---HONumber=Mooncake_0411_2016-->