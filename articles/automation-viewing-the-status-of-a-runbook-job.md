<properties 
   pageTitle="在 Azure Automation 中查看 Runbook 作业的状态"
   description="在 Azure Automation 中启动 Runbook 时，将会创建一个作业。本文提供有关如何跟踪每个作业及查看其当前状态的信息。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
wacn.date="05/15/2015"
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="03/30/2015"
   ms.author="bwren" />

# 在 Azure Automation 中查看 Runbook 作业的状态


在 Azure Automation 中启动 Runbook 时，将会创建一个作业。作业是 Runbook 的单一执行实例。单个 Runbook 可以包含多个作业，每个作业具有自身的 Runbook 参数值集。可通过多种方法来检查一个或多个 Runbook 的特定作业和所有作业的状态。

## 作业状态

下表描述了作业的各种可能状态。

| 状态| 说明|
|:---|:---|
|已完成|作业已成功完成。|
|失败|作业已结束但出现错误。|
|失败，正在等待资源|作业失败，因为它已达到[公平份额](https://msdn.microsoft.com/zh-CN/library/azure/3179b655-ab39-407a-9169-33571f958325#fairshare)限制三次，并且每次都已从同一个检查点或 Runbook 开始处启动。|
|已排队|作业正在等待提供 Automation 工作线程的资源，以便能够启动。|
|正在启动|作业已分配给工作线程，并且系统正在将它启动。|
|正在恢复|系统正在恢复已暂停的作业。|
|正在运行|作业正在运行。|
|正在运行，正在等待资源|作业已卸载，因为它已达到[公平份额](https://msdn.microsoft.com/zh-CN/library/azure/3179b655-ab39-407a-9169-33571f958325#fairshare)限制。片刻之后，它将从其上一个检查点恢复。|
|已停止|作业在完成之前已被用户停止。|
|正在停止|系统正在停止作业。|
|已暂停|作业已被用户、系统或 Runbook 中的命令暂停。挂起的作业可以重新启动，并且将从其上一个检查点恢复，如果没有检查点，则从 Runbook 的开始处恢复。只有在出现异常时，系统才会挂起 Runbook。默认情况下，ErrorActionPreference 设置为 **Continue**，表示出错时作业将保持运行。如果此首选项变量设置为 **Stop**，则出错时作业将会挂起。|
|正在暂停|系统正在尝试按用户请求暂停作业。Runbook 只有在达到其下一个检查点后才能挂起。如果 Runbook 越过了最后一个检查点，则只有在完成后才能挂起。|

## 使用 Azure 管理门户查看作业状态

### Automation 仪表板

Automation 仪表板显示特定 Automation 帐户的所有 Runbook 的摘要。此外，它还包含帐户的"使用概览"。摘要图表显示在给定的天数或小时数内，进入每种状态的所有 Runbook 的作业总数。可以在图表右上角选择时间范围。图表的时间轴会根据选择的时间范围类型发生变化。可以通过单击屏幕顶部的特定状态来选择是否显示该状态的行。

可以使用以下步骤显示 Automation 仪表板。

1. 在 Azure 管理门户中，选择"自动化"，然后单击 Automation 帐户的名称。
1. 选择"仪表板"选项卡。

### Runbook 仪表板

Runbook 仪表板显示单个 Runbook 的摘要。摘要图表显示在给定的天数或小时数内，进入每种状态的 Runbook 的作业总数。可以在图表右上角选择时间范围。图表的时间轴会根据选择的时间范围类型发生变化。可以通过单击屏幕顶部的特定状态来选择是否显示该状态的行。

可以使用以下步骤显示 Runbook 仪表板。

1. 在 Azure 管理门户中，选择"自动化"，然后单击 Automation 帐户的名称。
1. 单击 Runbook 的名称。
1. 选择"仪表板"选项卡。

### 作业摘要

可以查看为特定 Runbook 创建的所有作业的列表及其最新状态。你可以按作业状态以及上次对作业进行更改的日期范围筛选此列表。单击作业的名称可以查看其详细信息和输出。作业的详细视图包括提供给该作业的 Runbook 参数值。

可以使用以下步骤查看 Runbook 的作业。

1. 在 Azure 管理门户中，选择"自动化"，然后单击 Automation 帐户的名称。
1. 单击 Runbook 的名称。
1. 选择"作业"选项卡。
1. 单击作业的"已创建作业"列可查看其详细信息和输出。

## 使用 Windows PowerShell 检索作业状态

可以使用 [Get-AzureAutomationJob](https://msdn.microsoft.com/zh-CN/library/azure/dn690263.aspx) 检索为 Runbook 创建的作业和特定作业的详细信息。如果在 Windows PowerShell 中使用 [Start-AzureAutomationRunbook](https://msdn.microsoft.com/zh-CN/library/azure/dn690259.aspx) 启动了 Runbook，则会返回生成的作业。使用 [Get-AzureAutomationJob](https://msdn.microsoft.com/zh-CN/library/azure/dn690263.aspx)Output 可以获取作业的输出。

以下示例命令将检索示例 Runbook 的最后一个作业，并显示其状态、为 Runbook 参数提供的值以及作业的输出。

	$job = (Get-AzureAutomationJob -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" | sort LastModifiedDate -desc)[0]
	$job.Status
	$job.JobParameters
	Get-AzureAutomationJobOutput -AutomationAccountName "MyAutomationAccount" -Id $job.Id -Stream Output

## 相关文章

- [在 Azure Automation 中启动 Runbook](/documentation/articles/automation-starting-a-runbook)

<!--HONumber=53-->