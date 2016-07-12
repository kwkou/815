<properties
   pageTitle="在 Azure 自动化中启动 Runbook | Azure"
   description="汇总了可用于在 Azure 自动化中启动 Runbook 的不同方法，并提供有关使用 Azure 经典管理门户和 Windows PowerShell 的详细信息。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn" />

 <tags
   ms.service="automation"
   ms.date="02/23/2016"
   wacn.date="06/30/2016"/>

# 在 Azure 自动化中启动 Runbook

下表将帮助你确定如何在 Azure 自动化中以最适合你方案的方法启动 Runbook。本文包含有关使用 Azure 经典管理门户和 Windows PowerShell 启动 Runbook 的详细信息。有关其他方法的详细信息将在其他文档中提供，你可以通过以下链接来访问。

| **方法** | **特征** |
|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Azure 经典管理门户](#starting-a-runbook-with-the-azure-portal) | <li>使用交互式用户界面的最简单方法。<br> <li>用于提供简单参数值的窗体。<br> <li>轻松跟踪作业状态。<br> <li>使用 Azure 登录对访问进行身份验证。 |
| [Windows PowerShell](https://msdn.microsoft.com/zh-cn/library/dn690259.aspx) | <li>使用 Windows PowerShell cmdlet 从命令行调用。<br> <li>可以使用多个步骤包含在自动化解决方案中。<br> <li>使用证书或 OAuth 用户主体/服务主体对请求进行身份验证。<br> <li>提供简单和复杂的参数值。<br> <li>跟踪作业状态。<br> <li>支持 PowerShell cmdlet 所需的客户端。 |
| [Azure 自动化 API](http://msdn.microsoft.com/zh-cn/library/azure/mt163849.aspx) | <li>最有弹性的方法，但也最复杂。<br> <li>从任何可发出 HTTP 请求的自定义代码调用。<br> <li>使用证书或 OAuth 用户主体/服务主体对请求进行身份验证。<br> <li>提供简单和复杂的参数值。<br> <li>跟踪作业状态。 |
| [Webhook](/documentation/articles/automation-webhooks/) | <li>从单个 HTTP 请求启动 Runbook。<br> <li>使用 URL 中的安全令牌进行身份验证。<br> <li>客户端无法覆盖创建 Webhook 时指定的参数值。Runbook 可以定义填入了 HTTP 请求详细信息的单个参数。<br> <li>无法通过 Webhook URL 跟踪作业状态。 |
| [响应 Azure 警报](/documentation/articles/automation-webhooks/) | <li>启动 Runbook 以响应 Azure 警报。<br> <li>为 Runbook 配置 Webhook 并链接到警报。<br> <li>使用 URL 中的安全令牌进行身份验证。<br> <li>当前仅对度量值支持警报。 |
| [计划](/documentation/articles/automation-scheduling-a-runbook/) | <li>按每小时、每天或每周计划自动启动 Runbook。<br> <li>通过 Azure 经典管理门户、PowerShell cmdlet 或 Azure API 来操作计划。<br> <li>提供与计划配置使用的参数值。 |
| [从另一个 Runbook](/documentation/articles/automation-child-runbooks/) | <li>使用一个 Runbook 作为另一个 Runbook 中的活动。<br> <li>对多个 Runbook 使用的功能很有用。<br> <li>为子 Runbook 提供参数值，并使用父 Runbook 中的输出。 |

下图演示了 Runbook 生命周期的详细分步过程。其中包括在 Azure 自动化中启动 Runbook 的不同方式、本地计算机执行 Azure 自动化 Runbook 时所需的组件，以及不同组件之间的交互。

![Runbook 体系结构](./media/automation-starting-runbook/runbooks-architecture.png)

##<a name="starting-a-runbook-with-the-azure-portal"></a> 使用 Azure 经典管理门户启动 Runbook

1.	在 Azure 经典管理门户中，选择“自动化”，然后单击自动化帐户的名称。
2.	选择“Runbook”选项卡。
3.	选择一个 Runbook，然后单击“启动”。
4.	如果 Runbook 包含参数，则系统会提示你在文本框中提供每个参数的值。请参阅下面的 [Runbook 参数](#Runbook-parameters)，以获取有关参数的更多详细信息。
5.	选择“启动 Runbook”消息旁边的“查看作业”，或选择 Runbook 的“作业”选项卡以查看 Runbook 作业的状态。


##<a name="starting-a-runbook-with-windows-powershell"></a> 使用 Windows PowerShell 启动 Runbook

可以在 Windows PowerShell 中使用 [Start-AzureAutomationRunbook](http://msdn.microsoft.com/zh-cn/library/azure/dn690259.aspx) 启动 Runbook。以下示例代码将启动名为 Test-Runbook 的 Runbook。

	Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook"

Start-AzureAutomationRunbook 将返回一个作业对象，启动 Runbook 后，你可以使用该对象来跟踪 Runbook 的状态。然后可以将此作业对象与 [Get-AzureAutomationJob](http://msdn.microsoft.com/zh-cn/library/azure/dn690263.aspx) 结合使用来确定作业的状态，并将它与 [Get-AzureAutomationJobOutput](http://msdn.microsoft.com/zh-cn/library/azure/dn690268.aspx) 结合使用以获取作业的输出。以下示例代码将启动名为 Test-Runbook 的 Runbook，等待它完成，然后显示其输出。

	$job = Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook"
	
	$doLoop = $true
	While ($doLoop) {
	   $job = Get-AzureAutomationJob –AutomationAccountName "MyAutomationAccount" -Id $job.Id
	   $status = $job.Status
	   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped") 
	}
	
	Get-AzureAutomationJobOutput –AutomationAccountName "MyAutomationAccount" -Id $job.Id –Stream Output

如果 Runbook 需要参数，则你必须以[哈希表](http://technet.microsoft.com/zh-cn/library/hh847780.aspx)的形式提供参数，其中，哈希表的密钥与参数名称匹配，值为参数值。以下示例演示如何启动包含两个名称分别为 FirstName 和 LastName 的字符串参数、一个名为 RepeatCount 的整数和一个名为 Show 的布尔参数的 Runbook。有关参数的其他信息，请参阅下面的 [Runbook 参数](#Runbook-parameters)。

	$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
	Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" –Parameters $params

##<a name="runbook-parameters"></a> Runbook 参数

当你使用 Azure 经典管理门户或 Windows PowerShell 启动 Runbook 时，系统将通过 Azure 自动化 Web 服务发送指令。此服务不支持复杂数据类型的参数。如果需要提供复杂参数的值，则必须根据 [Azure 自动化中的子 Runbook](/documentation/articles/automation-child-runbooks/) 中所述，以内联方式从另一个 Runbook 调用该参数值。

Azure 自动化 Web 服务将为使用特定数据类型的参数提供特殊功能，如以下部分中所述。

### 命名值

如果参数的数据类型为 object，则你可以使用以下 JSON 格式向它发送命名值列表：*{"Name1":Value1, "Name2":Value2, "Name3":Value3}*。这些值必须使用简单类型。Runbook 将以 [PSCustomObject](http://msdn.microsoft.com/zh-cn/library/azure/system.management.automation.pscustomobject(v=vs.85).aspx) 的形式接收参数，该对象的属性对应于每个命名值。

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

如果参数是数组（如 array 或 string），则你可以使用以下 JSON 格式向它发送值列表：*[Value1,Value2,Value3]*。这些值必须使用简单类型。

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

如果参数的数据类型为 **PSCredential**，则你可以提供 Azure 自动化[凭据资产](/documentation/articles/automation-credentials/)的名称。Runbook 将检索具有指定名称的凭据。

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

## 后续步骤

-	本文中的 Runbook 体系结构提供了有关子 Runbook 的概括说明，若要了解详细信息，请参阅 [Child runbooks in Azure Automation（Azure 自动化中的子 Runbook）](/documentation/articles/automation-child-runbooks/)

<!---HONumber=Mooncake_0411_2016-->