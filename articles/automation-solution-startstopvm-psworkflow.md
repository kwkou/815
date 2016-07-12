<properties 
	pageTitle="通过 Azure 自动化启动和停止虚拟机 - PowerShell 工作流 | Azure"
	description="Azure 自动化解决方案的图形版本，包括启动和停止经典虚拟机所需的 Runbook。"
	services="automation"
	documentationCenter=""
	authors="bwren"
	manager="stevenka"
	editor="tysonn" />
<tags 
	ms.service="automation"
	ms.date="01/27/2016"
	wacn.date="06/30/2016" />

# Azure 自动化解决方案 - 启动和停止虚拟机

此 Azure 自动化解决方案包括启动和停止经典虚拟机所需的 Runbook。你可以将此解决方案用于以下任何情形：

- 在你自己的环境中使用 Runbook，无需修改。 
- 修改 Runbook 以执行自定义功能。  
- 从另一作为整体解决方案一部分的 Runbook 调用 Runbook。 
- 使用 Runbook 作为教程来学习 Runbook 创作概念。 


## 获取解决方案

此解决方案包含两个可通过以下链接下载的 PowerShell 工作流 Runbook。

| Runbook | 链接 | 类型 | 说明 |
|:---|:---|:---|:---|
| Start-AzureVMs | [启动 Azure 经典 VM](https://gallery.technet.microsoft.com/Start-Azure-Classic-VMs-86ef746b) | PowerShell 工作流 | 启动 Azure 订阅者中的所有经典虚拟机，或者启动所有具有特定服务名称的虚拟机。 |
| Stop-AzureVMs | [停止 Azure 经典 VM](https://gallery.technet.microsoft.com/Stop-Azure-Classic-VMs-7a4ae43e) | PowerShell 工作流 | 停止自动化帐户中的所有虚拟机，或者停止所有具有特定服务名称的虚拟机。 |

[AZURE.INCLUDE [automation-azurechinacloud-environment-parameter](../includes/automation-azurechinacloud-environment-parameter.md)]

##<a id="installing-the-solution"></a> 安装解决方案

### 1\.安装 Runbook

下载 Runbook 之后，你可以使用[导入 Runbook](/documentation/articles/automation-creating-importing-runbook/#ImportRunbook) 中的流程来导入它们。

### 2\.查看说明和要求
Runbook 包括带注释的帮助文本，其中包括说明和所需的资产。你也可以从本文中获取相同的信息。

### 3\.配置资产
Runbook 需要以下资产，你必须创建这些资产并在其中填充适当的值。

| 资产类型 | 资产名称 | 说明 |
|:---|:---|:---|:---|
| 凭据 | AzureCredential | 包含帐户凭据，该帐户有权在 Azure 订阅中启动和停止虚拟机。此外，你也可以在 **Add-AzureAccount -Environment AzureChinaCloud** 活动的 **Credential** 参数中指定其他凭据。 |
| 变量 | AzureSubscriptionId | 包含你的 Azure 订阅的订阅 ID。 |

##<a id="using-the-solution"></a> 使用解决方案

### 参数

每个 Runbook 具有以下参数。必需参数必须提供值，其他参数则可根据要求选择性地提供值。

| 参数 | 类型 | 必需 | 说明 |
|:---|:---|:---|:---|
| ServiceName | 字符串 | 否 | 如果提供了值，则会启动或停止具有该服务名称的所有虚拟机。如果没有提供值，则会启动或停止 Azure 订阅中的所有经典虚拟机。 |
| AzureSubscriptionIdAssetName | 字符串 | 否 | 包含[变量资产](#installing-the-solution)的名称，该资产中包含 Azure 订阅的订阅 ID。如果不指定值，则会使用 *AzureSubscriptionId*。 |
| AzureCredentialAssetName | 字符串 | 否 | 包含[凭据资产](#installing-the-solution)的名称，该资产中包含要使用的 Runbook 的凭据。如果不指定值，则会使用 *AzureCredential*。 |

### 启动 Runbook

你可以使用[在 Azure 自动化中启动 Runbook](/documentation/articles/automation-starting-a-runbook/) 中的任何方法来启动此解决方案中的任一 Runbook。

以下示例命令使用 Windows PowerShell 来运行 **StartAzureVMs**，以便启动服务名称为 *MyVMService* 的所有虚拟机。

	$params = @{"ServiceName"="MyVMService"}
	Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Start-AzureVMs" –Parameters $params

### 输出

这些 Runbook 会为每个虚拟机[输出一条消息](/documentation/articles/automation-runbook-output-and-messages/)，指示是否已成功提交启动或停止指令。你可以在输出中查找特定字符串，以确定每个 Runbook 的结果。可能的输出字符串列在下表中。

| Runbook | 条件 | 消息 |
|:---|:---|:---|
| Start-AzureVMs | 虚拟机已运行 | MyVM 已运行 |
| Start-AzureVMs | 已成功提交虚拟机启动请求 | MyVM 已启动 |
| Start-AzureVMs | 虚拟机启动请求失败 | MyVM 无法启动 |
| Stop-AzureVMs | 虚拟机已运行 | MyVM 已停止 |
| Stop-AzureVMs | 已成功提交虚拟机启动请求 | MyVM 已启动 |
| Stop-AzureVMs | 虚拟机启动请求失败 | MyVM 无法启动 |

例如，Runbook 中的以下代码段尝试启动服务名称为 *MyServiceName* 的所有虚拟机。如果任何启动请求失败，则可能会执行错误操作。

	$results = Start-AzureVMs -ServiceName "MyServiceName"
	foreach ($result in $results) {
		if ($result -like "* has been started" ) {
			# Action to take in case of success.
		}
		else {
			# Action to take in case of error.
		}
	}


## 明细

下面是此解决方案中 Runbook 的明细。你可以利用此信息来自定义 Runbook，或者从中了解某些情况。

### 参数

    param (
        [Parameter(Mandatory=$false)] 
        [String]  $AzureCredentialAssetName = 'AzureCredential',
        
        [Parameter(Mandatory=$false)]
        [String] $AzureSubscriptionIdAssetName = 'AzureSubscriptionId',

        [Parameter(Mandatory=$false)] 
        [String] $ServiceName
    )

工作流开始时，会获取[输入参数](#using-the-solution)的值。如果未提供资产名称，则使用默认名称。

### 输出

    # Returns strings with status messages
    [OutputType([String])]

此行声明 Runbook 的输出将是一个字符串。这不是必需的，但在将 Runbook 用作[子 Runbook](/documentation/articles/automation-child-runbooks/) 的情况下，这是一种最佳做法，可以让父 Runbook 了解应该期望哪种输出类型。

### 身份验证

	# Connect to Azure and select the subscription to work against
	$Cred = Get-AutomationPSCredential -Name $AzureCredentialAssetName
	$null = Add-AzureAccount -Environment AzureChinaCloud -Credential $Cred -ErrorAction Stop
	$SubId = Get-AutomationVariable -Name $AzureSubscriptionIdAssetName
    $null = Select-AzureSubscription -SubscriptionId $SubId -ErrorAction Stop

后续的行设置将要用于剩余 Runbook 的凭据和 Azure 订阅。首先，我们使用 **Get-AutomationPSCredential** 来获取用于存储凭据的资产，这些凭据具有相应的访问权限，可用于启动和停止 Azure 订阅中的虚拟机。**Add-AzureAccount -Environment AzureChinaCloud** 然后就会使用此资产来设置凭据。该输出已分配给一个虚拟变量，因此不会包括在 Runbook 输出中。

然后会使用 **Get-AutomationVariable** 来检索包含订阅 ID 的变量资产，并使用 **Select-AzureSubscription** 来设置订阅。

### 获取 VM

	# If there is a specific cloud service, then get all VMs in the service,
    # otherwise get all VMs in the subscription.
    if ($ServiceName) 
	{ 
		$VMs = Get-AzureVM -ServiceName $ServiceName
	}
    else 
	{ 
		$VMs = Get-AzureVM
	}

**Get-AzureVM** 用于检索兼容 Runbook 的虚拟机。如果在 **ServiceName** 输入变量中提供了值，则仅检索使用该服务名称的虚拟机。如果 **ServiceName** 为空，则检索所有虚拟机。

### 启动/停止虚拟机并发送输出。

    # Start each of the stopped VMs
    foreach ($VM in $VMs)
    {
		if ($VM.PowerState -eq "Started")
		{
			# The VM is already started, so send notice
			Write-Output ($VM.InstanceName + " is already running")
		}
		else
		{
			# The VM needs to be started
        	$StartRtn = Start-AzureVM -Name $VM.Name -ServiceName $VM.ServiceName -ErrorAction Continue

	        if ($StartRtn.OperationStatus -ne 'Succeeded')
	        {
				# The VM failed to start, so send notice
                Write-Output ($VM.InstanceName + " failed to start")
	        }
			else
			{
				# The VM started, so send notice
				Write-Output ($VM.InstanceName + " has been started")
			}
		}
    }

后续行将在每个虚拟机中执行。首先会检查虚拟机的 **PowerState**，看其是处于运行还是停止状态，具体取决于 Runbook。如果该虚拟机已处于目标状态，则会将消息发送到输出中，Runbook 结束。如果该虚拟机尚未处于目标状态，则会使用 **Start-AzureVM** 或 **Stop-AzureVM** 来尝试启动或停止虚拟机，其结果就是将请求存储到某个变量中。然后会将一条消息发送到输出，指出是否已成功提交启动或停止请求。


## 相关文章

- [Azure 自动化中的子 Runbook](/documentation/articles/automation-child-runbooks/) 
- [Azure 自动化中的 Runbook 输出和消息](/documentation/articles/automation-runbook-output-and-messages/)

<!---HONumber=Mooncake_0307_2016-->