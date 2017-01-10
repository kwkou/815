<properties 
	pageTitle="在 Azure 自动化中测试 Runbook | Azure"
	description="在 Azure 自动化中发布某个 Runbook 之前，你可以对它进行测试，以确保它按预期工作。本文介绍如何测试 Runbook 并查看其输出。"
	services="automation"
	documentationCenter=""
	authors="mgoedtel"
	manager="jwhit"
	editor="tysonn" />
<tags 
	ms.service="automation"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="infrastructure-services"
	ms.date="09/12/2016"
	wacn.date="01/09/2017"
	ms.author="magoedte;bwren" />

# 在 Azure 自动化中测试 Runbook
测试 Runbook 时，将执行[草稿版](/documentation/articles/automation-creating-importing-runbook/#publishing-a-runbook)，并会完成其所执行的任何操作。不会创建作业历史记录，但会在测试输出窗格中显示“[输出](/documentation/articles/automation-runbook-output-and-messages/#output-stream)”与“[警告和错误](/documentation/articles/automation-runbook-output-and-messages/#message-streams)”。仅当 [$VerbosePreference 变量](/documentation/articles/automation-runbook-output-and-messages/#preference-variables)设置为 Continue 时，才会在输出窗格中显示发送到“[详细流](/documentation/articles/automation-runbook-output-and-messages/#message-streams)”的消息。

即使草稿版正在运行，该 Runbook 也仍会正常执行工作流，并针对环境中的资源执行任何操作。因此，你只能在非生产资源中测试 Runbook。

## 在 Azure 经典管理门户中测试 Runbook

1. [打开 Runbook 的草稿版](/documentation/articles/automation-edit-textual-runbook/#to-edit-a-runbook-with-the-azure-portal)。
2. 单击“测试”按钮以开始测试。如果 Runbook 包含参数，则会显示一个对话框让你提供用于测试的值。
6. 可以使用输出窗格下方的按钮停止或暂停正在测试中的 Runbook。当你暂停 Runbook 时，该 Runbook 会完成它在被暂停之前正在进行的活动。暂停 Runbook 后，你可以将它停止或重启。
7. 在输出窗格中检查 Runbook 的输出。

## 后续步骤

- 若要了解如何创建或导入 Runbook，请参阅[在 Azure 自动化中创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)
- 若要开始使用 PowerShell 工作流 Runbook，请参阅[我的第一个 PowerShell 工作流 Runbook](/documentation/articles/automation-first-runbook-textual/)
- 若要详细了解如何配置 Runbook 以返回状态消息和错误（包括建议的做法），请参阅 [Runbook output and messages in Azure Automation](/documentation/articles/automation-runbook-output-and-messages/)（Azure 自动化中的 Runbook 输出和消息）

<!---HONumber=Mooncake_Quality_Review_0104_2017-->