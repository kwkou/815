<properties 
   pageTitle="将 Azure Automation Runbook 添加到恢复计划" 
   description="本指南介绍了如何借助 Azure Site Recovery，在恢复到 Azure 期间使用 Azure Automation 完成复杂任务，从而扩展恢复计划" 
   services="site-recovery" 
   documentationCenter="" 
   authors="ruturaj" 
   manager="mkjain" 
   editor=""/>

<tags
   ms.service="site-recovery"
   ms.devlang="powershell"
   ms.tgt_pltfrm="na"
   ms.topic="article"
   ms.workload="required" 
   ms.date="03/31/2015"
   wacn.date="05/15/2015"
   ms.author="ruturajd@microsoft.com"/>

  
   

# 将 Azure Automation Runbook 添加到恢复计划


本教程介绍 Azure Site Recovery 如何与 Azure Automation 集成以便为恢复计划提供可扩展性。恢复计划可以协调使用 Azure Site Recovery 保护的虚拟机的恢复，以便复制到辅助云和 Azure 方案。它们还可帮助实现恢复的**一致准确性**、**可重复性**和**自动化**。如果你要将虚拟机故障转移到 Azure，则与 Azure Automation 集成可扩展恢复计划，并使你能够执行 Runbook，从而可以执行强大的自动化任务。

如果你从未听说过 Azure Automation，请在[此处](/home/features/automation/)注册并下载其示例脚本。阅读有关 [Azure Site Recovery](/home/features/site-recovery/) 的详细信息

在本教程中，我们将了解如何将 Azure Automation Runbook 集成到恢复计划。我们将自动执行以前需要手动干预的简单任务，并了解如何将多步骤恢复转换成单击恢复操作。我们还将了解如何解决已出错的简单脚本。

## 在 Azure 中保护应用程序

让我们先从一个包括两个虚拟机的简单应用程序开始。在这里，我们已有一个 HRweb 应用程序 Fabrikam。Fabrikam-HRweb-frontend 和 Fabrikam-Hrweb-backend 是使用 Azure Site Recovery 在 Azure 中保护的两个虚拟机。若要使用 Azure Site Recovery 保护虚拟机，请遵循以下步骤。 

1.  为虚拟机启用保护。

2.  确保虚拟机已完成初始复制，并且正在复制。 

3.  等到初始复制完成，并且复制状态显示为"受保护"。

![](./media/site-recovery-runbook-automation/01.png)
---------------------

在本教程中，我们将为 Fabrikam HRweb 应用程序创建一个恢复计划，以将应用程序故障转移到 Azure。然后，我们会将它与某个 Runbook 集成，该 Runbook 将在故障转移的 Azure 虚拟机上创建一个终结点，以便为端口 80 上的网页提供服务。

首先，让我们为应用程序创建恢复计划。

## 创建恢复计划

若要将应用程序恢复到 Azure，你需要创建一个恢复计划。使用恢复计划可以指定虚拟机的恢复顺序。首先会恢复并启动组 1 中的虚拟机，接着是组 2 中的虚拟机。

创建类似于下面的恢复计划。

![](./media/site-recovery-runbook-automation/12.png)

若要了解有关恢复计划的详细信息，请阅读[此处](https://msdn.microsoft.com/zh-CN/library/azure/dn788799.aspx "here")的文档。 

接下来，让我们在 Azure Automation 中创建必要的项目。

## 创建 Automation 帐户及其资产

你需要使用一个 Azure Automation 帐户来创建 Runbook。如果你还没有帐户，请导航到 Azure Automation 选项卡（由 ![](./media/site-recovery-runbook-automation/02.png)表示）并创建一个新帐户。

1.  为该帐户指定一个标识名称。

2.  指定要将该帐户放置到的地理区域。

建议将该帐户放在与 ASR 保管库所在的同一个区域。

![](./media/site-recovery-runbook-automation/03.png)

接下来，在该帐户中创建以下资产。

### 添加订阅名称作为资产

1.  在 Azure Automation"资产"中添加新设置 ![](./media/site-recovery-runbook-automation/04.png) 并选择 ![](./media/site-recovery-runbook-automation/05.png)

2.  选择"字符串"作为变量类型

3.  指定 **AzureSubscriptionName** 作为变量名称

    ![](./media/site-recovery-runbook-automation/06.png)

4.  指定你的实际 Azure 订阅名称作为变量值。

	![](./media/site-recovery-runbook-automation/07_1.png)

你可以从 Azure 门户上的帐户设置页中找到你的订阅名称。 

### 添加 Azure 登录凭据作为资产

Azure Automation 使用 Azure PowerShell 连接到订阅以及对那里的项目进行操作。为此，你需要使用自己的 Microsoft 帐户、工作帐户或学校帐户进行身份验证。
可以将帐户凭据存储在资产中，以供 Runbook 安全使用。

1.  在 Azure Automation"资产"中添加新设置 ![](./media/site-recovery-runbook-automation/04.png) 并选择 ![](./media/site-recovery-runbook-automation/09.png)

2.  选择"Windows PowerShell 凭据"作为凭据类型

3.  指定 **AzureCredential** 作为名称

    ![](./media/site-recovery-runbook-automation/10.png)

4.  指定用于登录的用户名和密码。

现在，你的资产中会出现这两项设置。

![](./media/site-recovery-runbook-automation/11.png)

[此处](/documentation/articles/install-configure-powershell)提供了有关如何通过 powershell 连接到订阅的详细信息。

接下来，你将要在 Azure Automation 中创建一个 Runbook，用于在故障转移后为前端虚拟机添加终结点。

## Azure Automation 上下文

ASR 会将上下文变量传递给 Runbook，以帮助你编写确定性的脚本。有人可能会认为云服务和虚拟机的名称是可预测的，但在某些情况下并不总是这样，例如，可能会由于 Azure 中不支持的字符而导致虚拟机名称发生变化。因此，此信息将作为 *context*的一部分传递到 ASR 恢复计划。

下面是上下文变量形式的示例。 

        {"RecoveryPlanName":"hrweb-recovery",

        "FailoverType":"Test",

        "FailoverDirection":"PrimaryToSecondary",

        "GroupId":"1",

        "VmMap":{"7a1069c6-c1d6-49c5-8c5d-33bfce8dd183":

                {"CloudServiceName":"pod02hrweb-Chicago-test",

                "RoleName":"Fabrikam-Hrweb-frontend-test"}

                }

        }


下表包含上下文中每个变量的名称和说明。
  
<table border="1">
  <tr><th>变量名称</th><th>变量说明</th></tr>
  <tr><td>RecoveryPlanName</td><td>要执行的恢复计划的名称。 <p> 此变量可帮助你根据恢复计划的名称，使用同一个脚本执行不同的操作。</td></tr>
  <tr><td>FailoverType</td><td>指定执行是"测试"、"计划内"还是"计划外"。 <p> 此变量可帮助你根据故障转移的类型执行不同的操作。 </td></tr>
  <tr><td>FailoverDirection</td><td>指定恢复是从主端到辅助端，还是相反。 <p>它使用的两个值为 **PrimaryToSecondary** 和 **SecondaryToPrimary** </td></tr>
  <tr><td>GroupId</td><td> 标识执行 Runbook 的恢复计划中的组编号。 <p> 例如，如果 Runbook 是发布组 2，则 GroupId 将是 2。 </td></tr>
  <tr><td>VmMap</td><td> 这是组中所有虚拟机的数组。 </td></tr>
  <tr><td>VmMap 键</td><td>每个虚拟机都有一个由 GUID 标识的唯一键。此 GUID 与虚拟机的 VMM ID 相同。 <p> 你可以使用此 GUID 明确地指定想要对哪个虚拟机执行操作。 </td></tr>
  <tr><td>RoleName</td><td>指定要恢复的 Azure 虚拟机的名称。</td></tr>
  <tr><td>CloudServiceName</td><td> 指定要在其下创建虚拟机的 Azure 云服务名称。 </td></tr>
  </table><br />

若要在上下文中标识 VmMap 键，你也可以转到 ASR 中的 VM 属性页，并查看 VM GUID 属性。

![](./media/site-recovery-runbook-automation/13.png)

## 创作 Automation Runbook

现在，请创建用于在前端虚拟机上打开端口 80 的 Runbook。

1.  在 Azure Automation 帐户中使用名称 **OpenPort80** 创建一个新的 Runbook

![](./media/site-recovery-runbook-automation/14.png)

2.  导航到 Runbook 的"创作"视图，并进入草稿模式。

3.  首先指定要用作恢复计划上下文的变量

```
	param (
		[Object]$RecoveryPlanContext
	)

```
  

4.  接下来，使用凭据和订阅名称连接到订阅

```
	$Cred = Get-AutomationPSCredential -Name 'AzureCredential'
	
	# Connect to Azure
	$AzureAccount = Add-AzureAccount -Credential $Cred
	$AzureSubscriptionName = Get-AutomationVariable -Name 'AzureSubscriptionName'
	Select-AzureSubscription -SubscriptionName $AzureSubscriptionName
```

> 请注意，此处使用了 Azure 资产 - **AzureCredential** 和 **AzureSubscriptionName**。

5.  现在，请指定终结点详细信息和你要公开其终结点（在本例中为前端虚拟机）的虚拟机的 GUID。

```
	# Specify the parameters to be used by the script
	$AEProtocol = "TCP"
	$AELocalPort = 80
	$AEPublicPort = 80
	$AEName = "Port 80 for HTTP"
	$VMGUID = "7a1069c6-c1d6-49c5-8c5d-33bfce8dd183"
```

这将指定 Azure 终结点协议、VM 上的本地端口及其映射的公共端口。这些变量是向 VM 添加终结点的 Azure 命令所需的参数。VMGUID 包含你要对其执行操作的虚拟机的 GUID。 

6.  现在，脚本提取给定 VM GUID 的上下文，并在它引用的虚拟机上创建终结点。

```
	#Read the VM GUID from the context
	$VM = $RecoveryPlanContext.VmMap.$VMGUID

	if ($VM -ne $null)
	{
		# Invoke pipeline commands within an InlineScript

		$EndpointStatus = InlineScript {
			# Invoke the necessary pipeline commands to add a Azure Endpoint to a specified Virtual Machine
			# This set of commands includes: Get-AzureVM | Add-AzureEndpoint | Update-AzureVM (including necessary parameters)

			$Status = Get-AzureVM -ServiceName $Using:VM.CloudServiceName -Name $Using:VM.RoleName | `
				Add-AzureEndpoint -Name $Using:AEName -Protocol $Using:AEProtocol -PublicPort $Using:AEPublicPort -LocalPort $Using:AELocalPort | `
				Update-AzureVM
			Write-Output $Status
		}
	}
```

7. 完成此操作后，点击"发布" ![](./media/site-recovery-runbook-automation/20.png) 使脚本可执行。 

下面提供了完整脚本供你参考

```
  workflow OpenPort80
  {
	param (
		[Object]$RecoveryPlanContext
	)

	$Cred = Get-AutomationPSCredential -Name 'AzureCredential'
	
	# Connect to Azure
	$AzureAccount = Add-AzureAccount -Credential $Cred
	$AzureSubscriptionName = Get-AutomationVariable -Name 'AzureSubscriptionName'
	Select-AzureSubscription -SubscriptionName $AzureSubscriptionName

	# Specify the parameters to be used by the script
	$AEProtocol = "TCP"
	$AELocalPort = 80
	$AEPublicPort = 80
	$AEName = "Port 80 for HTTP"
	$VMGUID = "7a1069c6-c1d6-49c5-8c5d-33bfce8dd183"
	
	#Read the VM GUID from the context
	$VM = $RecoveryPlanContext.VmMap.$VMGUID

	if ($VM -ne $null)
	{
		# Invoke pipeline commands within an InlineScript

		$EndpointStatus = InlineScript {
			# Invoke the necessary pipeline commands to add an Azure Endpoint to a specified Virtual Machine
			# This set of commands includes: Get-AzureVM | Add-AzureEndpoint | Update-AzureVM (including necessary parameters)

			$Status = Get-AzureVM -ServiceName $Using:VM.CloudServiceName -Name $Using:VM.RoleName | `
				Add-AzureEndpoint -Name $Using:AEName -Protocol $Using:AEProtocol -PublicPort $Using:AEPublicPort -LocalPort $Using:AELocalPort | `
				Update-AzureVM
			Write-Output $Status
		}
	}
  }
```

## 将脚本添加到恢复计划

脚本准备就绪后，你可以将它添加到先前创建的恢复计划。

1.  在创建的恢复计划中，选择在组 2 后面添加脚本。 ![](./media/site-recovery-runbook-automation/15.png)

2.  指定脚本名称。这只是此恢复计划的友好名称，将在恢复计划中显示。

3.  在故障转移到 Azure 脚本中，选择 Azure Automation 帐户名。

4.  在 Azure Runbook 中，选择你创作的 Runbook。

![](./media/site-recovery-runbook-automation/16.png)

## 测试恢复计划

将 Runbook 添加到计划后，你可以启动测试故障转移并查看其运行情况。我们始终建议运行测试故障转移来测试你的应用程序和恢复计划，以确保不会出现任何错误。

1.  选择恢复计划并启动测试故障转移。

2.  在执行计划期间，你可以通过状态了解 Runbook 是否已执行。

    ![](./media/site-recovery-runbook-automation/17.png)

3.  你也可以在 Runbook 的 Azure Automation 作业页上查看详细的 Runbook 执行状态。

    ![](./media/site-recovery-runbook-automation/18.png)

4.  在故障转移完成后，除了 Runbook 执行结果以外，你还可以通过访问 Azure 虚拟机页并查看终结点，来了解执行是否成功。 

![](./media/site-recovery-runbook-automation/19.png)

## 示例脚本

尽管我们在本教程中演练的是一个常见任务，那就是向 Azure 虚拟机添加终结点，但是，你可以使用 Azure Automation 完成其他许多功能强大的自动化任务。Microsoft 和 Azure Automation 社区提供了示例 Runbook，可帮助你开始创建自己的解决方案和实用 Runbook，可用作更大自动化任务的构建基块。你可以从库中使用这些 Runbook，通过 Azure Site Recovery 为应用程序生成强大的单击式恢复计划。

## 其他资源

[Azure Automation 概述](https://msdn.microsoft.com/zh-CN/library/azure/dn643629.aspx "Azure Automation Overview")

[Azure Automation 示例脚本](http://gallery.technet.microsoft.com/scriptcenter/site/search?f[0].Type=User&f[0].Value=SC%20Automation%20Product%20Team&f[0].Text=SC%20Automation%20Product%20Team "Sample Azure Automation Scripts")


<!--HONumber=53-->