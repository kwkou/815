<properties 
   pageTitle="在 Azure Automation 中执行 Runbook"
   description="详细介绍如何处理 Azure Automation 中的 Runbook。"
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
   ms.date="04/13/2015"
   ms.author="bwren" />

# 在 Azure Automation 中执行 Runbook


在 Azure Automation 中启动 Runbook 时，将会创建一个作业。作业是 Runbook 的单一执行实例。将分配一个 Azure Automation 工作线程来运行每个作业。尽管工作线程由多个 Azure 帐户共享，但不同 Automation 帐户中的作业是相互独立的。你无法控制要由哪个工作线程为作业的请求提供服务。一个 Runbook 可以同时运行多个作业。当你在 Azure 门户中查看 Runbook 列表时，列表中会列出上次为每个 Runbook 启动的作业的状态。可以查看每个 Runbook 的作业列表以跟踪每个作业的状态。有关不同作业状态的说明，请参阅[作业状态](/documentation/articles/automation-viewing-the-status-of-a-runbook-job#job-statuses)。

![Job Statuses](./media/automation-runbook-execution/job-statuses.png)


作业可以通过与 Azure 订阅建立连接来访问 Azure 资源。仅当数据中心内的资源可从公有云访问时，作业才能访问这些资源。

## 权限

Azure Automation 中的 Runbook 通常需要访问 Azure 订阅中的资源。[使用 Azure Active Directory 向 Azure 进行身份验证](http://aka.ms/runbookauthor/authentication)介绍了如何在 Azure Automation 中配置 Azure Active Directory 和[凭据]()，以便对 Azure 资源进行身份验证。此主题还包含有关使用 [Add-AzureAccount cmdlet](http://aka.ms/runbookauthor/azurecmdlets/addazureaccount) 访问你创建的 Runbook 中的 Azure 资源的信息。请参阅你导入的各个 Runbook 的文档，以了解其安全要求。

## 公平份额

为了在云中的所有 Runbook 之间共享资源，Azure Automation 在运行 3 小时后将暂时卸载所有作业，然后从其上一个[检查点](http://aka.ms/runbookauthor/checkpoints)将其重启。在此期间，该作业将显示"正在运行，正在等待资源"状态。如果该 Runbook 没有检查点或者作业在卸载之前尚未达到第一个检查点，则会从开始处重启。

如果 Runbook 连续三次从同一个检查点或者从 Runbook 的开始处重启，则会终止并显示状态"失败，正在等待资源"。这是为了防止 Runbook 无限期运行而无法完成，因为在不重新卸载的情况下，它们无法到达下一个检查点。在此情况下，你将会收到以下异常和失败。

该作业无法继续运行，因为它已反复从同一个检查点逐出。请确保你的 Runbook 在未保持其状态的情况下没有执行冗长的操作。

在创建 Runbook 时，应确保在两个检查点之间运行任何活动的时间不超过 30 分钟。你可能需要向 Runbook 添加检查点，以确保它不会达到此 30 分钟限制。还可能需要中断长时间运行的操作。例如，你的 Runbook 可能对大型 SQL 数据库执行了重新编制索引。如果这一项操作未在公平份额限制内完成，则作业将会卸载并从开始处重启。在此情况下，你应该将重新编制索引操作拆分成多个步骤（例如，一次重新编制一个表的索引），然后在每项操作的后面插入一个检查点，使作业能够在上次操作后恢复并得以完成。

## 相关文章

- [在 Azure Automation 中启动 Runbook](/documentation/articles/automation-starting-a-runbook)
- [在 Azure Automation 中查看 Runbook 作业的状态](/documentation/articles/automation-viewing-the-status-of-a-runbook-job)

<!--HONumber=53-->