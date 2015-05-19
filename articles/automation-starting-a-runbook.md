<properties 
   pageTitle="在 Azure Automation 中启动 Runbook"
   description="介绍如何使用 Azure 管理门户和 Windows PowerShell 在 Azure Automation 中启动 Runbook。包括有关提供参数值的信息。"
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

# 在 Azure Automation 中启动 Runbook

本文介绍如何使用 Azure 管理门户和 Windows PowerShell 在 Azure Automation 中启动 Runbook。[从一个 Runbook 启动另一个 Runbook](https://msdn.microsoft.com/zh-CN/library/azure/dn857355.aspx) 中介绍了如何从一个 Runbook 启动另一个 Runbook。

## 使用 Azure 管理门户启动 Runbook

1. 在 Azure 管理门户中，选择"自动化"，然后单击 Automation 帐户的名称。
1. 选择"Runbook"选项卡。
1. 选择一个 Runbook，然后单击"启动"。
1. 如果 Runbook 包含参数，则系统会提示你在文本框中提供每个参数的值。参数类型及其默认值（如果有）显示在文本框的下面。对于布尔值和日期时间参数，将提供特殊的选择器而不是标准的文本框。有关参数的更多详细信息，请参阅下面的 [Runbook 参数](#Runbook-parameters) 。
1. 选择"启动 Runbook"消息旁边的"查看作业"，或选择 Runbook 的"作业"选项卡以查看该作业的状态。

## 使用 Windows PowerShell 启动 Runbook

可以在 Windows PowerShell 中使用 [Start-AzureAutomationRunbook](https://msdn.microsoft.com/zh-CN/library/azure/dn690259.aspx) 启动 Runbook。以下示例代码将启动名为 Test-Runbook 的 Runbook。

	Start-AzureAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook"

Start-AzureAutomationRunbook 将返回一个作业对象，启动 Runbook 后，你可以使用该对象来跟踪 Runbook 的状态。然后可以将此作业对象与 [Get-AzureAutomationJob](https://msdn.microsoft.com/zh-CN/library/azure/dn690263.aspx) 结合使用来确定作业的状态，并将它与 [Get-AzureAutomationJobOutput](https://msdn.microsoft.com/zh-CN/library/azure/dn690268.aspx) 结合使用以获取作业的输出。以下示例代码将启动名为 Test-Runbook 的 Runbook，等待它完成，然后显示其输出。

	$job = Start-AzureAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook"
	
	$doLoop = $true
	While ($doLoop) {
	   $job = Get-AzureAutomationJob -AutomationAccountName "MyAutomationAccount" -Id $job.Id
	   $status = $job.Status
	   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped") 
	}
	
	Get-AzureAutomationJobOutput -AutomationAccountName "MyAutomationAccount" -Id $job.Id -Stream Output

如果 Runbook 需要参数，则你必须以[哈希表](https://technet.microsoft.com/zh-CN/library/hh847780.aspx)的形式提供参数，其中，哈希表的密钥与参数名称匹配，值为参数值。以下示例演示如何启动包含两个名称分别为 FirstName 和 LastName 的字符串参数、一个名为 RepeatCount 的整数和一个名为 Show 的布尔参数的 Runbook。有关参数的其他信息，请参阅下面的 [Runbook 参数](#Runbook-parameters) 。

	$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
	Start-AzureAutomationRunbook -AutomationAccountName "MyAutomationAccount" -Name "Test-Runbook" -Parameters $params

## Runbook 参数

当你使用 Azure 管理门户或 Windows PowerShell 启动 Runbook 时，系统将通过 Azure Automation Web 服务发送指令。此服务不支持复杂数据类型的参数。如果需要提供复杂参数的值，则必须根据[从一个 Runbook 启动另一个 Runbook](https://msdn.microsoft.com/zh-CN/library/azure/dn857355.aspx) 中所述，以内联方式从另一个 Runbook 调用该参数值。

Azure Automation Web 服务将为使用特定数据类型的参数提供特殊功能，如以下部分中所述。

### 命名值

如果参数的数据类型为 [object]，则你可以使用以下 JSON 格式向它发送命名值列表：*{"Name1":Value1, "Name2":Value2, "Name3":Value3}*. 这些值必须使用简单类型。Runbook 将以 [PSCustomObject](https://msdn.microsoft.com/zh-CN/library/azure/system.management.automation.pscustomobject(v=vs.85).aspx) 的形式接收参数，该对象的属性对应于每个命名值。

请考虑以下接受名为 user 的参数的测试 Runbook。

	Workflow Test-Parameters
	{
	   param ( 
	      [Parameter(Mandatory=$true)][object]$user
	   )
	    if ($user.Show) {
	        foreach ($i in 1..$user.RepeatCount) {
	            $user.FirstName
	            $user.LastName
	        }
	    } 
	}

可为 user 参数使用以下文本。

	{"FirstName":"Joe","LastName":"Smith","RepeatCount":2,"Show":true}

这会导致生成以下输出。

	Joe
	Smith
	Joe
	Smith

### 数组

如果参数是数组（如 [array] 或 [string[]]），则你可以使用以下 JSON 格式向它发送值列表：*[Value1,Value2,Value3]*。这些值必须使用简单类型。

请考虑以下接受名为 *user* 的参数的测试 Runbook。

	Workflow Test-Parameters
	{
	   param ( 
	      [Parameter(Mandatory=$true)][array]$user
	   )
	    if ($user[3]) {
	        foreach ($i in 1..$user[2]) {
	            $ user[0]
	            $ user[1]
	        }
	    } 
	}

可为 user 参数使用以下文本。

	["Joe","Smith",2,true]

这会导致生成以下输出。

	Joe
	Smith
	Joe
	Smith

### 凭据

如果参数的数据类型为 **PSCredential**，则你可以提供 Azure Automation [凭据资产](https://msdn.microsoft.com/zh-CN/library/azure/dn940015.aspx)的名称。Runbook 将检索具有指定名称的凭据。

请考虑以下接受名为 credential 的参数的测试 Runbook。

	Workflow Test-Parameters
	{
	   param ( 
	      [Parameter(Mandatory=$true)][PSCredential]$credential
	   )
	   $credential.UserName
	}

假设存在名为 *My Credential* 的凭据资产，则可为 user 参数使用以下文本。

	My Credential

假设凭据中的用户名为 *jsmith*，则会导致生成以下输出。

	jsmith

## 相关文章

- [从一个 Runbook 启动另一个 Runbook](https://msdn.microsoft.com/zh-CN/library/azure/dn857355.aspx)

<!--HONumber=53-->