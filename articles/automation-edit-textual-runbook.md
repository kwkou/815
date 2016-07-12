<properties 
	pageTitle="在 Azure 自动化中编辑文本 Runbook"
	description="本文提供的不同过程适用于在 Azure 自动化中通过文本编辑器来处理 PowerShell 工作流 Runbook。"
	services="automation"
	documentationCenter=""
	authors="mgoedtel"
	manager="stevenka"
	editor="tysonn" />
<tags 
	ms.service="automation"
	ms.date="02/23/2016"
	wacn.date="06/30/2016" />

# 在 Azure 自动化中编辑文本 Runbook

Azure 自动化中的文本编辑器可以用来编辑 PowerShell 工作流 Runbook。该编辑器具有其他代码编辑器的典型功能（例如智能感知和颜色编码），并提供其他特殊功能来帮助你访问 Runbook 的常用资源。本文提供了使用该编辑器执行不同功能的详细步骤。

该文本编辑器包含的一项功能是将活动、资产和子 Runbook 的代码插入 Runbook 中。你不需要亲自键入代码，只需从可用资源列表中进行选择，然后即可将相应代码插入 Runbook 中。

Azure 自动化中的每个 Runbook 都有两个版本：草稿版和已发布版。你先对 Runbook 的草稿版进行编辑，然后将其发布，这样便可以执行了。无法编辑已发布版本。有关详细信息，请参阅[发布 Runbook](/documentation/articles/automation-creating-importing-runbook/#publishing-a-runbook)。


## 使用 Azure 经典管理门户编辑 Runbook

通过以下过程打开 Runbook 即可在文本编辑器中进行编辑。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
2. 选择“Runbook”选项卡。
3. 单击你想要编辑的 Runbook 的名称，然后选择“创作”选项卡。
5. 单击屏幕底部的“编辑”按钮。
6. 执行所需的编辑。
7. 完成编辑后，单击“保存”。
8. 若要发布最新的 Runbook 草稿版，请单击“发布”。

### 将活动插入 Runbook

1. 在文本编辑器的“画布”中，将光标置于要放置该活动的地方。
1. 在屏幕底部，单击“插入”，然后单击“活动”。
1. 在“集成模块”列中，选择包含活动的模块。
1. 在“活动”窗格中，选择一个活动。
1. 在“说明”列中，记下活动说明。你也可以选择单击“查看详细帮助”，以便在浏览器中启动活动的帮助。
1. 单击右箭头。如果该活动具有参数，则会将其列出供你参考。
1. 单击勾选按钮。此时会将运行活动的代码插入 Runbook 中。
1. 如果活动需要参数，请提供合适的值来替换大括号 <> 中的数据类型。

### 将子 Runbook 的代码插入 Runbook 中

1. 在文本编辑器的“画布”中，将光标置于要放置[子 Runbook](/documentation/articles/automation-child-runbooks/) 的地方。
2. 在屏幕底部，单击“插入”，然后单击“Runbook”。
3. 选择要从中心列插入的 Runbook，然后单击右箭头。
4. 如果该 Runbook 具有参数，则会将其列出供你参考。
5. 单击勾选按钮。此时会将运行所选 Runbook 的代码插入当前 Runbook 中。
7. 如果 Runbook 需要参数，请提供合适的值来替换大括号 <> 中的数据类型。

### 将资产插入 Runbook

1. 在文本编辑器的“画布”中，将光标置于要放置该活动以检索资产的地方。
1. 在屏幕底部，单击“插入”，然后单击“设置”。
1. 在“设置操作”列中，选择所需操作。
1. 在中心列的可用资产中进行选择。
1. 单击勾选按钮。此时会将获取或设置资产的代码插入 Runbook 中。



## 使用 Windows PowerShell 编辑 Azure 自动化 Runbook

若要使用 Windows PowerShell 来编辑 Runbook，可使用所选编辑器进行操作，然后将其保存到 .ps1 文件。你可以先使用 [Get-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/zh-cn/library/dn690269.aspx) cmdlet 来检索 Runbook 的内容，然后使用 [Set-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/zh-cn/library/dn690267.aspx) cmdlet 将现有的草稿 Runbook 替换为修改的 Runbook。

### 使用 Windows PowerShell 检索 Runbook 的内容

以下示例命令演示了如何检索 Runbook 的脚本并将其保存到脚本文件。在此示例中，检索的是草稿版本。也可以检索 Runbook 的已发布版本，不过该版本不能进行更改。

    $automationAccountName = "MyAutomationAccount"
    $runbookName = "Sample-TestRunbook"
    $scriptPath = "c:\runbooks\Sample-TestRunbook.ps1"
    
    $runbookDefinition = Get-AzureAutomationRunbookDefinition -AutomationAccountName $automationAccountName -Name $runbookName -Slot Draft
    $runbookContent = $runbookDefinition.Content

    Out-File -InputObject $runbookContent -FilePath $scriptPath

### 使用 Windows PowerShell 更改 Runbook 的内容

以下示例命令演示了如何使用脚本文件的内容替换 Runbook 的现有内容。请注意，此示例过程与[使用 Windows PowerShell 从脚本文件中导入 Runbook](/documentation/articles/automation-creating-importing-runbook/#ImportRunbookScriptPS) 中的相同。

    $automationAccountName = "MyAutomationAccount"
    $runbookName = "Sample-TestRunbook"
    $scriptPath = "c:\runbooks\Sample-TestRunbook.ps1"

    Set-AzureAutomationRunbookDefinition -AutomationAccountName $automationAccountName -Name $runbookName -Path $scriptPath -Overwrite
    Publish-AzureAutomationRunbook –AutomationAccountName $automationAccountName –Name $runbookName

## 相关文章

- [在 Azure 自动化中创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)
- [了解 PowerShell 工作流](/documentation/articles/automation-powershell-workflow/)
- [证书](/documentation/articles/automation-certificates/)
- [连接](/documentation/articles/automation-connections/)
- [凭据](/documentation/articles/automation-credentials/)
- [计划](/documentation/articles/automation-schedules/)
- [变量](/documentation/articles/automation-variables/)

<!---HONumber=Mooncake_0307_2016-->