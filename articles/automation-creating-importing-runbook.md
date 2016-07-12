<properties
	pageTitle="在 Azure 自动化中创建或导入 Runbook"
	description="本文介绍了如何在 Azure 自动化中创建新的 Runbook，或如何从文件中导入 Runbook。"
	services="automation"
	documentationCenter=""
	authors="mgoedtel"
	manager="stevenka"
	editor="tysonn" />
<tags
	ms.service="automation"
	ms.date="02/29/2016"
	wacn.date="06/30/2016" />

# 在 Azure 自动化中创建或导入 Runbook

你可以通过以下方法将 Runbook 添加到 Azure 自动化：[创建新的 Runbook](#creating-a-new-runbook)；从文件或 [Runbook 库](/documentation/articles/automation-runbook-gallery/)导入现有 Runbook。本文介绍如何通过文件创建和导入 Runbook。你可以在 [Azure 自动化的 Runbook 和模块库](/documentation/articles/automation-runbook-gallery/)中获取有关如何访问社区 Runbook 和模块的所有详细信息。

##<a name="creating-a-new-runbook"></a> 创建新的 Runbook

你可以使用其中一个 Azure 经典管理门户或 Windows PowerShell 在 Azure 自动化中创建一个新的 Runbook。一旦创建 Runbook，你就可以利用[了解 PowerShell 工作流](/documentation/articles/automation-powershell-workflow/)中的信息对其进行编辑。

### 使用 Azure 经典管理门户创建新的 Azure 自动化 Runbook

你只能使用 Azure 经典管理门户中的 PowerShell 工作流 Runbook。

1. 在 Azure 经典管理门户中，依次单击“新建”、“应用程序服务”、“自动化”、“Runbook”、“快速创建”。
2. 输入所需的信息，然后单击“创建”。Runbook 名称必须以字母开头，可以使用字母、数字、下划线和短划线。
3. 若要立即编辑 Runbook，则请单击“编辑 Runbook”。否则，请单击“确定”。
4. 新的 Runbook 将出现在“Runbook”选项卡中。

### 使用 Windows PowerShell 创建新的 Azure 自动化 Runbook

你可以使用 [New-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690272.aspx) cmdlet 创建空的 PowerShell 工作流 Runbook。你可以通过指定 **Name** 参数来创建空的可以随后编辑的 Runbook，也可以通过指定 **Path** 参数来导入脚本文件。

以下示例命令演示了如何创建新的空 Runbook。

    $automationAccountName = "MyAutomationAccount"
    $runbookName = "Sample-TestRunbook"
    New-AzureAutomationRunbook –AutomationAccountName $automationAccountName –Name $runbookName

##<a name="ImportRunbook"></a> 将 Runbook 从文件导入 Azure 自动化

你可以在 Azure 自动化中创建新的 Runbook，方法是导入 PowerShell 工作流（扩展名为 .ps1）。

如果该文件包含多个 PowerShell 工作流，导入将失败。必须将每个工作流保存到各自的文件中，并分别导入每个工作流。

### 使用 Azure 经典管理门户通过文件导入 Runbook
可通过以下过程将脚本文件导入 Azure 自动化。请注意，你只能通过此经典管理门户将 .ps1 文件导入 PowerShell 工作流 Runbook。

1. 在 Azure 经典管理门户中，选择“自动化”，然后选择一个自动化帐户。
2. 单击“导入”。
3. 单击“浏览文件”，找到要导入的脚本文件。
4. 若要立即编辑 Runbook，则请单击“编辑 Runbook”。否则，请单击“确定”。
5. 新的 Runbook 将出现在自动化帐户的“Runbook”选项卡中。
6. 必须先[发布 Runbook](#publishing-a-runbook)，然后才能运行它。



###<a name="ImportRunbookScriptPS"></a> 使用 Windows PowerShell 从脚本文件中导入 Runbook

可以使用 [Set-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/zh-cn/library/dn690267.aspx) cmdlet 将脚本文件导入到现有 Runbook 的草稿版本中。脚本文件必须包含单个 Windows PowerShell 工作流。如果 Runbook 已有草稿版本，则除非你使用 Overwrite 参数，否则导入将失败。导入 Runbook 后，可以使用 [Publish-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690266.aspx) 来发布它。

下面的示例命令演示了如何将脚本文件导入到现有 Runbook 中，然后将其发布。

    $automationAccountName = "MyAutomationAccount"
    $runbookName = "Sample-TestRunbook"
    $scriptPath = "c:\runbooks\Sample-TestRunbook.ps1"

    Set-AzureAutomationRunbookDefinition –AutomationAccountName $automationAccountName –Name $runbookName –Path $ scriptPath -Overwrite
    Publish-AzureAutomationRunbook –AutomationAccountName $automationAccountName –Name $runbookName


##<a name="publishing-a-runbook"></a> 发布 Runbook

创建或导入新的 Runbook 时，必须先将其发布，然后才能导入。Azure 自动化中的每个 Runbook 都有草稿版和已发布版。只有已发布版才能用来运行，只有草稿版才能用来编辑。已发布版不受对草稿版所做的任何更改的影响。当草稿版可以使用时，你可以发布草稿版，这样草稿版就会覆盖已发布版。

## 使用 Azure 经典管理门户发布 Runbook

1. 在 Azure 经典管理门户中打开 Runbook。
1. 在屏幕顶部，单击“创作”。
1. 在屏幕底部，单击“发布”，然后在出现验证消息时单击“是”。


## 使用 Windows PowerShell 发布 Runbook

可以使用 Windows PowerShell，通过 [Publish-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690266.aspx) cmdlet 来发布 Runbook。以下示例命令显示了如何发布示例 Runbook。

	$automationAccountName = "MyAutomationAccount"
	$runbookName = "Sample-TestRunbook"
	
	Publish-AzureAutomationRunbook –AutomationAccountName $automationAccountName –Name $runbookName



## 相关文章

- [Azure 自动化的 Runbook 和模块库](/documentation/articles/automation-runbook-gallery/)
- [在 Azure 自动化中编辑文本 Runbook](/documentation/articles/automation-edit-textual-runbook/)

<!---HONumber=Mooncake_0411_2016-->