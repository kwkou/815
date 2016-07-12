<properties
   pageTitle="在 Azure 自动化中执行 Runbook"
   description="详细介绍如何处理 Azure 自动化中的 Runbook。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="03/21/2016"
   wacn.date="06/30/2016" />

# 在 Azure 自动化中执行 Runbook


在 Azure 自动化中启动 Runbook 时，将会创建一个作业。作业是 Runbook 的单一执行实例。将分配一个 Azure 自动化工作线程来运行每个作业。尽管工作线程由多个 Azure 帐户共享，但不同自动化帐户中的作业是相互独立的。你无法控制要由哪个工作线程为作业的请求提供服务。一个 Runbook 可以同时运行多个作业。当你在 Azure 经典管理门户中查看 Runbook 列表时，列表中会列出上次为每个 Runbook 启动的作业的状态。可以查看每个 Runbook 的作业列表以跟踪每个作业的状态。有关不同作业状态的说明，请参阅[作业状态](#job-statuses)。

下图显示 PowerShell 工作流 Runbook 的 Runbook 作业生命周期。

![作业状态 - PowerShell 工作流](./media/automation-runbook-execution/job-statuses.png)

作业可以通过与 Azure 订阅建立连接来访问 Azure 资源。仅当数据中心内的资源可从公有云访问时，作业才能访问这些资源。

## 作业状态

下表描述了作业的各种可能状态。

| 状态| 说明|
|:---|:---|
|已完成|作业已成功完成。|
|已失败| 对于 PowerShell 工作流的 Runbook，Runbook 无法编译。 |
|失败，正在等待资源|作业失败，因为它已达到[公平份额](#fairshare)限制三次，并且每次都已从同一个检查点或 Runbook 开始处启动。|
|已排队|作业正在等待提供自动化工作线程的资源，以便能够启动。|
|正在启动|作业已分配给工作线程，并且系统正在将它启动。|
|正在恢复|系统正在恢复已暂停的作业。|
|正在运行|作业正在运行。|
|正在运行，正在等待资源|作业已卸载，因为它已达到[公平份额](#fairshare)限制。片刻之后，它将从其上一个检查点恢复。|
|已停止|作业在完成之前已被用户停止。|
|正在停止|系统正在停止作业。|
|已挂起|作业已被用户、系统或 Runbook 中的命令暂停。挂起的作业可以重新启动，并且将从其上一个检查点恢复，如果没有检查点，则从 Runbook 的开始处恢复。只有在出现异常时，系统才会挂起 Runbook。默认情况下，ErrorActionPreference 设置为 **Continue**，表示出错时作业将保持运行。如果此首选项变量设置为 **Stop**，则出错时作业将会挂起。仅适用于 PowerShell 工作流 Runbook。|
|正在暂停|系统正在尝试按用户请求暂停作业。Runbook 只有在达到其下一个检查点后才能挂起。如果 Runbook 越过了最后一个检查点，则只有在完成后才能挂起。|

## 使用 Azure 经典管理门户查看作业状态

### 自动化仪表板

自动化仪表板显示特定自动化帐户的所有 Runbook 的摘要。此外，它还包含帐户的“使用概览”。摘要图表显示在给定的天数或小时数内，进入每种状态的所有 Runbook 的作业总数。可以在图表右上角选择时间范围。图表的时间轴会根据选择的时间范围类型发生变化。可以通过单击屏幕顶部的特定状态来选择是否显示该状态的行。

可以使用以下步骤显示自动化仪表板。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 选择“仪表板”选项卡。

### Runbook 仪表板

Runbook 仪表板显示单个 Runbook 的摘要。摘要图表显示在给定的天数或小时数内，进入每种状态的 Runbook 的作业总数。可以在图表右上角选择时间范围。图表的时间轴会根据选择的时间范围类型发生变化。可以通过单击屏幕顶部的特定状态来选择是否显示该状态的行。

可以使用以下步骤显示 Runbook 仪表板。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 单击 Runbook 的名称。
1. 选择“仪表板”选项卡。

### 作业摘要

可以查看为特定 Runbook 创建的所有作业的列表及其最新状态。你可以按作业状态以及上次对作业进行更改的日期范围筛选此列表。单击作业的名称可以查看其详细信息和输出。作业的详细视图包括提供给该作业的 Runbook 参数值。

可以使用以下步骤查看 Runbook 的作业。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 单击 Runbook 的名称。
1. 选择“作业”选项卡。
1. 单击作业的“已创建作业”列可查看其详细信息和输出。

## 使用 Windows PowerShell 检索作业状态

可以使用 [Get-AzureAutomationJob](http://msdn.microsoft.com/zh-cn/library/azure/dn690263.aspx) 检索为 Runbook 创建的作业和特定作业的详细信息。如果在 Windows PowerShell 中使用 [Start-AzureAutomationRunbook](http://msdn.microsoft.com/zh-cn/library/azure/dn690259.aspx) 启动了 Runbook，则会返回生成的作业。使用 [Get-AzureAutomationJobOutput](http://msdn.microsoft.com/zh-cn/library/azure/dn690263.aspx) 可以获取作业的输出。

以下示例命令将检索示例 Runbook 的最后一个作业，并显示其状态、为 Runbook 参数提供的值以及作业的输出。

	$job = (Get-AzureAutomationJob -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" | sort LastModifiedDate –desc)[0]
	$job.Status
	$job.JobParameters
	Get-AzureAutomationJobOutput -AutomationAccountName "MyAutomationAccount" -Id $job.Id –Stream Output

## 公平份额

为了在云中的所有 Runbook 之间共享资源，Azure 自动化在任何作业运行 3 小时后都会将其暂时卸载。PowerShell 工作流 Runbook 将会从上一个[检查点](http://technet.microsoft.com/zh-cn/library/dn469257.aspx#bk_Checkpoints)进行恢复。在此期间，该作业将显示“正在运行，正在等待资源”状态。如果该 Runbook 没有检查点或者作业在卸载之前尚未达到第一个检查点，则会从开始处重启。

如果 Runbook 连续三次从同一个检查点或者从 Runbook 的开始处重启，则会终止并显示状态“失败，正在等待资源”。这是为了防止 Runbook 无限期运行而无法完成，因为在不重新卸载的情况下，它们无法到达下一个检查点。在此情况下，你将会收到以下异常和失败。

该作业无法继续运行，因为它已反复被系统从同一个检查点逐出。请确保你的 Runbook 在未保持其状态的情况下没有执行冗长的操作。

在创建 Runbook 时，应确保在两个检查点之间运行任何活动的时间不超过 3 小时。你可能需要向 Runbook 添加检查点以确保它不会达到此 3 小时限制，或者需要将长时间运行的操作进行分解。例如，你的 Runbook 可能对大型 SQL 数据库执行了重新编制索引。如果这一项操作未在公平份额限制内完成，则作业将会卸载并从开始处重启。在此情况下，你应该将重新编制索引操作拆分成多个步骤（例如，一次重新编制一个表的索引），然后在每项操作的后面插入一个检查点，使作业能够在上次操作后恢复并得以完成。



## 后续步骤

- [在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook/)

<!---HONumber=Mooncake_0411_2016-->