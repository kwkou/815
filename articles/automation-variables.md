<properties 
   pageTitle="Azure 自动化中的变量资产 | Windows Azure"
   description="变量资产是可供 Azure 自动化中的所有 Runbook 使用的值。本文介绍了变量的详细信息，以及如何在文本创作中使用变量。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.date="08/18/2015"
   wacn.date="09/15/2015" />

# Azure 自动化中的变量资产

变量资产是可供自动化帐户中的所有 Runbook 使用的值。可以从 Azure 门户、Windows PowerShell 和 Runbook 中创建、修改和检索变量资产。自动化变量可用于以下方案：

- 在多个 Runbook 之间共享某个值。

- 在同一 Runbook 中的多个作业之间共享某个值。

- 从门户或通过 Runbook 使用的 Windows PowerShell 命令行管理某个值。

自动化变量将保留，以便在 Runbook 失败时它们仍然继续可用。这也允许一个 Runbook 设置的值随后由另一个 Runbook 使用，或由同一 Runbook 在下次运行时使用。

创建变量时，可以指定将其加密存储。当变量加密后，它将安全地存储在 Azure 自动化中并且不能从 Azure PowerShell 模块随附的 [Get-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913772.aspx) cmdlet 检索变量值。可以检索加密值的唯一方法是从 Runbook 中的 **Get-AutomationVariable** 活动检索。

>[AZURE.NOTE]Azure 自动化中的安全资产包括凭据、证书、连接和加密的变量。这些资产已使用针对每个自动化帐户生成的唯一密钥加密并存储在 Azure 自动化中。此密钥由主证书加密，并存储在 Azure 自动化中。在存储安全资产之前，会先使用主证书来解密自动化帐户的密钥，然后使用该密钥来加密资产。

## <a id="variable-types"></a> 变量类型

当使用 Azure 门户创建变量时，您必须通过下拉列表指定一个数据类型，以便门户可以显示用于输入变量值的相应控件。该变量并不局限于此数据类型，但如果您想要指定不同类型的值，则必须使用 Windows PowerShell 设置该变量。如果指定为“未定义”，则该变量的值将设置为 **$null**，并且你必须使用 [Set-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913767.aspx) cmdlet 或 **Set-AutomationVariable** 活动来设置该值。无法在该门户中创建或更改复杂变量类型的值，但您可以使用 Windows PowerShell 提供任何类型的值。复杂类型将作为 [PSCustomObject](http://msdn.microsoft.com/zh-cn/library/system.management.automation.pscustomobject.aspx) 返回。

您可以通过创建一个数组或哈希表并将其保存到变量，来将多个值存储到单一变量。

## Cmdlet 和工作流活动

下表中的 cmdlet 用于通过 Windows PowerShell 创建和管理自动化变量。可在自动化 Runbook 中使用的 [Azure PowerShell 模块](/documentation/articles/powershell-install-configure)已随附了这些 cmdlet。

|Cmdlet|说明|
|:---|:---|
|[Get-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913772.aspx)|检索现有变量的值。|
|[New-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913771.aspx)|创建新变量并设置变量值。|
|[Remove-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913775.aspx)|删除现有变量。|
|[Set-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913767.aspx)|设置现有变量的值。|

下表中的工作流活动用于在 Runbook 中访问自动化变量。它们仅可在 Runbook 中使用，而不作为“Azure PowerShell 模块”的一部分提供。

|工作流活动|说明|
|:---|:---|
|Get-AutomationVariable|检索现有变量的值。|
|Set-AutomationVariable|设置现有变量的值。|

>[AZURE.NOTE]应避免在 **Get-AutomationVariable** 的 –Name 参数中使用变量，因为这可能会使设计时发现 Runbook 与自动化变量之间的依赖关系变得复杂化。

## 创建新的自动化变量

### 使用 Azure 门户创建新变量

1. 在你的自动化帐户中，单击窗口顶部的“资产”。
1. 在窗口底部，单击“添加设置”。
1. 单击“添加变量”。
1. 完成向导并单击复选框以保存新变量。

<!--
### 使用 Azure 预览门户创建新变量

1. 在你的自动化帐户中，单击“资产”部分打开“资产”边栏选项卡。
1. 单击“变量”部分以打开“变量”边栏选项卡。
1. 单击边栏选项卡顶部的“添加变量”。
1. 完成表单，然后单击“创建”以保存新变量。
-->

### 使用 Windows PowerShell 创建新变量

[New-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913771.aspx) cmdlet 创建一个新的变量并设置其初始值。你可以使用 [Get-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913772.aspx) 检索该值。如果该值为简单类型，则返回相同的类型。如果其为复杂类型，则返回 **PSCustomObject**。

下面的示例命令演示如何创建字符串类型的变量，然后返回其值。


	New-AzureAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' –Encrypted $false –Value 'My String'
	$string = (Get-AzureAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value

下面的示例命令演示如何创建复杂类型的变量，然后返回其属性。在这种情况下，会使用来自 **Get-AzureVM** 的虚拟机对象。

	$vm = Get-AzureVM –ServiceName "MyVM" –Name "MyVM"
	New-AzureAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm
	
	$vmValue = (Get-AzureAutomationVariable –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable").Value
	$vmName = $ vmValue.Name
	$vmIpAddress = $ vmValue.IpAddress



## 在 Runbook 中使用变量

使用 **Set-AutomationVariable** 活动设置 Runbook 中的自动化变量的值，并使用 **Get-AutomationVariable** 来检索该值。不应在 Runbook 中使用 **Set-AzureAutomationVariable** 或 **Get-AzureAutomationVariable** cmdlet，因为它们的效率低于工作流活动。你也无法使用 **Get-AzureAutomationVariable** 检索安全变量的值。从 Runbook 中创建新变量的唯一方法是使用 [New-AzureAutomationVariable](http://msdn.microsoft.com/zh-cn/library/dn913771.aspx) cmdlet。


### 文本 Runbook 示例

#### 设置和检索变量中的一个简单值

下面的示例命令演示如何设置和检索文本 Runbook 中的变量。在此示例中，假定已经创建了名为 *NumberOfIterations* 和 *NumberOfRunnings* 的整数类型变量和名为 *SampleMessage* 的字符串类型变量。

	$NumberOfIterations = Get-AutomationVariable -Name 'NumberOfIterations'
	$NumberOfRunnings = Get-AutomationVariable -Name 'NumberOfRunnings'
	$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'
	
	Write-Output "Runbook has been run $NumberOfRunnings times."
	
	for ($i = 1; $i -le $NumberOfIterations; $i++) {
	   Write-Output "$i`: $SampleMessage"
	}
	Set-AutomationVariable –Name NumberOfRunnings –Value (NumberOfRunngs += 1)


#### 设置和检索变量中的复杂对象

下面的示例代码演示如何更新文本 Runbook 中具有复杂值的变量。在此示例中，使用 **Get-AzureVM** 检索到一个 Azure 虚拟机并保存到一个现有的自动化变量。如[变量类型](#variable-types)中所述，该对象存储为一个 PSCustomObject。

	$vm = Get-AzureVM -ServiceName "MyVM" -Name "MyVM"
	Set-AutomationVariable -Name "MyComplexVariable" -Value $vm


在下面的代码中，从该变量检索值并将其用于启动虚拟机。

	$vmObject = Get-AutomationVariable -Name "MyComplexVariable"
	if ($vmObject.PowerState -eq 'Stopped') {
	   Start-AzureVM -ServiceName $vmObject.ServiceName -Name $vmObject.Name
	}


#### 设置和检索变量中的集合

下面的示例代码演示如何使用文本 Runbook 中包含一组复杂值的变量。在此示例中，使用 **Get-AzureVM** 检索到多个 Azure 虚拟机并保存到一个现有的自动化变量。如[变量类型](#variable-types)中所述，变量存储为一组 PSCustomObject。

	$vms = Get-AzureVM | Where -FilterScript {$_.Name -match "my"}     
    Set-AutomationVariable -Name 'MyComplexVariable' -Value $vms

在下面的代码中，从该变量检索此集合并用于启动每个虚拟机。

	$vmValues = Get-AutomationVariable -Name "MyComplexVariable"
	ForEach ($vmValue in $vmValues)
	{
	   if ($vmValue.PowerState -eq 'Stopped') {
	      Start-AzureVM -ServiceName $vmValue.ServiceName -Name $vmValue.Name
	   }
	}
<!--
### 图形 Runbook 示例

在图形 Runbook 中，你通过在图形编辑器的库窗格中的变量上右键单击并选择所需的活动添加 **Get-AutomationVariable** 或 **Set-AutomationVariable**。

![将变量添加到画布](./media/automation-variables/variable-add-canvas.png)

#### 设置变量中的值

下图显示了在图形 Runbook 中用于更新具有简单值的一个变量的示例活动。在此示例中，使用 **Get-AzureVM** 检索到一个 Azure 虚拟机并将计算机名称保存到一个字符串类型的现有自动化变量。[链接是管道还是序列](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)并不重要，因为我们仅预期输出中的单个对象。

![设置简单变量](./media/automation-variables/set-simple-variable.png)

下图显示了在图形 Runbook 中用于更新一个具有简单值的变量的示例活动。与前面的示例唯一不同之处在于不指定 **Set-AutomationVariable** 活动的“活动输出”的“字段路径”，因此将存储对象，而非对象的属性。如[变量类型](#variable-types)中所述，该对象存储为一个 PSCustomObject。

![设置复杂变量](./media/automation-variables/set-complex-variable.png)

下图显示与前面的示例类型的功能，将多个虚拟机保存到变量。此处必须使用[序列链接](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)，以便 **Set-AutomationVariable** 活动将整组虚拟机作为一个集合接收。如果使用[管道链接](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)，则会对每个对象单独运行 **Set-AutomationVariable** 活动，结果是只会保存该集合中的最后一个虚拟机。如[变量类型](#variable-types)中所述，变量存储为一组 PSCustomObject。

![设置复杂集合变量](./media/automation-variables/set-complex-variable-collection.png)

#### 从变量中检索值

下图显示了在图形 Runbook 中用于检索和使用变量的示例活动。第一个活动检索在前面的示例中保存至变量的虚拟机。该链接需要是[管道](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)链接，以便 **Start-AzureVM** 活动对发自 **Get-AutomationVariable** 活动的每个对象运行一次。无论存储在变量中的是单个对象还是多个对象，将执行的操作相同。**Start-AzureVM** 活动使用表示每个虚拟机的 PSCustomObject 的属性。

![获取复杂变量](./media/automation-variables/get-complex-variable.png)

下图显示了如何筛选存储到图形 Runbook 中的一个变量的对象。向前面示例中的链接添加了一个[条件](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)来仅筛选出设置该变量时已停止的那些虚拟机。

![获取筛选出的复杂变量](./media/automation-variables/get-complex-variable-filter.png)


## 相关文章

- [图形创作中的链接](/documentation/articles/automation-graphical-authoring-intro#links-and-workflow)-->
 

<!---HONumber=69-->