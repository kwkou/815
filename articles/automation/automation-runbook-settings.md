<properties
    pageTitle="Runbook 设置 | Azure"
    description="介绍 Azure 自动化中 Runbook 的配置设置，以及如何使用 Azure 经典管理门户和 Windows PowerShell 更改这些设置。"
    services="automation"
    documentationcenter=""
    author="mgoedtel"
    manager="stevenka"
    editor="tysonn" />  

<tags
    ms.assetid="a726f20c-a952-48b8-88ee-36d76aa3ac61"
    ms.service="automation"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/11/2016"
    wacn.date="01/09/2017"
    ms.author="bwren" />  


# Runbook 设置
Azure 自动化中的每个 Runbook 都提供了多个设置用于帮助标识自身，以及更改它的日志记录行为。下面将会描述其中的每个设置，然后再介绍修改设置的过程。

## 设置
### 名称和说明
创建 Runbook 后，无法更改它的名称。“说明”是可选的，最多可包含 512 个字符。

### 标记
使用标记可以指定不同的单词和短语用于帮助标识 Runbook。例如，在向 [Runbook 库](/documentation/articles/automation-runbook-gallery/)提交 Runbook 时，可以指定特定的标记来标识应将该 Runbook 列入的类别。可为一个 Runbook 指定多个标记并用逗号分隔各个标记。

### 日志记录
默认情况下，“详细”和“进度”记录不会写入作业历史记录。你可以更改特定 Runbook 的设置以记录这些记录。有关这些记录的详细信息，请参阅 [Runbook 输出和消息](/documentation/articles/automation-runbook-output-and-messages/)。

## 更改 Runbook 设置

### 使用 Azure 经典管理门户更改 Runbook 设置
可以在 Azure 经典管理门户中通过 Runbook 的“配置”页更改 Runbook 的设置。

1. 在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
1. 选择“Runbook”选项卡。
1. 单击 Runbook 的名称。
1. 选择“配置”选项卡。

### 使用 Windows PowerShell 更改 Runbook 设置
可以使用 [Set-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690275.aspx) cmdlet 更改 Runbook 的设置。如果想要指定多个标记，可以向 Tags 参数提供一个数组，或者一个包含逗号分隔值的字符串。可以使用 [Get-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690278.aspx) 获取当前标记。

以下示例命令演示了如何设置 Runbook 的属性。此示例向现有标记添加了三个标记，并指定应该记录详细记录。

    $automationAccountName = "MyAutomationAccount"
    $runbookName = "Sample-TestRunbook"
    $tags = (Get-AzureAutomationRunbook -AutomationAccountName $automationAccountName -Name $runbookName).Tags
    $tags += "Tag1,Tag2,Tag3"
    Set-AzureAutomationRunbook -AutomationAccountName $automationAccountName -Name $runbookName -LogVerbose $true -Tags $tags

## 后续步骤
* 若要学习如何创建输出和错误消息以及从 Runbook 检索此类消息，请参阅 [Runbook 输出和消息](/documentation/articles/automation-runbook-output-and-messages/)
* 若要了解如何添加已由社区或其他源开发的 Runbook，或创建自己的 Runbook，请参阅[创建或导入 Runbook](/documentation/articles/automation-creating-importing-runbook/)

<!---HONumber=Mooncake_Quality_Review_0104_2017-->