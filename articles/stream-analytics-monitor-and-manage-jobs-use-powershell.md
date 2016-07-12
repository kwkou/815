<properties 
	pageTitle="使用 PowerShell 监视和管理流分析作业 | Azure" 
	description="了解如何使用 Azure PowerShell 和 cmdlet 监视和管理流分析作业。" 
	keywords="azure powershell、azure powershell cmdlet、powershell 命令、powershell 脚本"	
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>


# 使用 Azure PowerShell cmdlet 监视和管理流分析作业

了解如何使用可执行基本流分析任务的 Azure PowerShell cmdlet 和 PowerShell 脚本来监视和管理流分析资源。

## 针对流分析运行 Azure PowerShell cmdlet 的先决条件

 - 在订阅中创建 Azure 资源组。下面是 Azure PowerShell 脚本示例。有关 Azure PowerShell 的信息，请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)；  

Azure PowerShell 0.9.8：

 		# Log in to your Azure account
		Add-AzureAccount -Environment AzureChinaCloud

		# Select the Azure subscription you want to use to create the resource group if you have more than one subscription on your account.
		Select-AzureSubscription -SubscriptionName <subscription name>
 
		# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
		#Register-AzureProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'

		# Create an Azure resource group
		New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>

Azure PowerShell 1.0：

 		# Log in to your Azure account
		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

		# Select the Azure subscription you want to use to create the resource group.
		Get-AzureRmSubscription –SubscriptionName “your sub” | Select-AzureRmSubscription

		# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
		#Register-AzureRmResourceProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
		
		# Create an Azure resource group
		New-AzureRMResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
		


> [AZURE.NOTE] 以编程方式创建的流分析作业默认情况下并不启用监视功能。你可以在 Azure 门户中手动启用监视功能，只需导航到作业的“监视”页，然后单击“启用”按钮即可；你也可以通过编程方式来执行此操作，只需遵循 [Azure 流分析 - 以编程方式监视流分析作业](/documentation/articles/stream-analytics-monitor-jobs/)中的步骤即可

## 适用于流分析的 Azure PowerShell cmdlet
下面的 Azure PowerShell cmdlet 可用于监视和管理 Azure 流分析作业。请注意，Azure PowerShell 具有不同版本。**在所列示例中，第一个命令用于 Azure PowerShell 0.9.8，第二个命令用于 Azure PowerShell 1.0。** Azure PowerShell 1.0 命令将在命令中始终包含“AzureRM”。

### Get-AzureStreamAnalyticsJob | Get-AzureRMStreamAnalyticsJob
列出在 Azure 订阅或指定资源组中定义的所有流分析作业，或者获取某个资源组中特定作业的作业信息。

**示例 1**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsJob

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsJob

此 PowerShell 命令在 Azure 订阅中返回有关所有流分析作业的信息。

**示例 2**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East 

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East 

此 PowerShell 命令在资源组 StreamAnalytics-Default-China-East 中返回有关所有流分析作业的信息。

**示例 3**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East -Name StreamingJob

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East -Name StreamingJob

此 PowerShell 命令在资源组 StreamAnalytics-Default-China-East 中返回有关流分析作业 StreamingJob 的信息。

### Get-AzureStreamAnalyticsInput | Get-AzureRMStreamAnalyticsInput
列出在指定流分析作业中定义的所有输入，或获取有关特定输入的信息。

**示例 1**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob

此 PowerShell 命令返回在作业 StreamingJob 中定义的所有输入的相关信息。

**示例 2**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name EntryStream

此 PowerShell 命令返回在作业 StreamingJob 中定义的名为 EntryStream 的输入的相关信息。

### Get-AzureStreamAnalyticsOutput | Get-AzureRMStreamAnalyticsOutput
列出在指定流分析作业中定义的所有输出，或获取有关特定输出的信息。

**示例 1**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob

此 PowerShell 命令返回在作业 StreamingJob 中定义的输出的相关信息。

**示例 2**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name Output

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name Output

此 PowerShell 命令返回在作业 StreamingJob 中定义的名为 Output 的输出的相关信息。

### Get-AzureStreamAnalyticsQuota | Get-AzureRMStreamAnalyticsQuota
获取指定区域中流式处理单元配额的相关信息。

**示例 1**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsQuota –Location "China East" 

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsQuota –Location "China East" 

此 PowerShell 命令返回有关美中地区配额和流式处理单位使用情况的信息。

### Get-AzureStreamAnalyticsTransformation | GetAzureRMStreamAnalyticsTransformation
获取在流分析作业中定义的特定转换的相关信息。

**示例 1**

Azure PowerShell 0.9.8：

	Get-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name StreamingJob

Azure PowerShell 1.0：

	Get-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –Name StreamingJob

此 PowerShell 命令返回作业 StreamingJob 中名为 StreamingJob 的转换的相关信息。

### New-AzureStreamAnalyticsInput | New-AzureRMStreamAnalyticsInput
在流分析作业中创建新的输入，或者更新现有的指定输入。
  
输入的名称可以在 .json 文件中指定，也可以在命令行中指定。如果在两处同时指定，则命令行的名称必须与文件中的名称相同。

如果你所指定的输入已存在，而且你并未指定 –Force 参数，则该 cmdlet 会询问你是否替换现有输入。

如果你指定了 –Force 参数，同时又指定了一个现有的输入名称，则会在不进行确认的情况下替换该输入。

有关 JSON 文件结构和内容的详细信息，请参阅[流分析管理 REST API 参考库][stream.analytics.rest.api.reference]的[创建输入（Azure 流分析）][msdn-rest-api-create-stream-analytics-input]部分。

**示例 1**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" 

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" 

此 PowerShell 命令从文件 Input.json 创建新的输入。如果已定义了现有的输入，在输入定义文件中指定了名称，则该 cmdlet 会询问是否替换该输入。

**示例 2**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

此 PowerShell 命令在名为 EntryStream 的作业中创建新的输入。如果已定义了使用此名称的现有输入，则该 cmdlet 会询问是否替换该输入。

**示例 3**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

此 PowerShell 命令会使用文件中的定义来替换名为 EntryStream 的现有输入源的定义。

### New-AzureStreamAnalyticsJob | New-AzureRMStreamAnalyticsJob
在  Azure 中创建新的流分析作业，或者更新现有的指定作业的定义。

作业的名称可以在 .json 文件中指定，也可以在命令行中指定。如果在两处同时指定，则命令行的名称必须与文件中的名称相同。

如果你所指定的作业名称已存在，而且你并未指定 –Force 参数，则该 cmdlet 会询问你是否替换现有作业。

如果你指定了 –Force 参数，同时又指定了一个现有的作业名称，则会在不进行确认的情况下替换作业定义。

有关 JSON 文件结构和内容的详细信息，请参阅[流分析管理 REST API 参考库][stream.analytics.rest.api.reference]的[创建流分析作业][msdn-rest-api-create-stream-analytics-job]部分。

**示例 1**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\JobDefinition.json" 

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\JobDefinition.json" 

此 PowerShell 命令在 JobDefinition.json 的定义中创建新的作业。如果已定义了现有的作业，在作业定义文件中指定了名称，则该 cmdlet 会询问是否替换该作业。

**示例 2**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\JobDefinition.json" –Name StreamingJob -Force

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\JobDefinition.json" –Name StreamingJob -Force

此 PowerShell 命令替换 StreamingJob 的作业定义。

### New-AzureStreamAnalyticsOutput | New-AzureRMStreamAnalyticsOutput
在流分析作业中创建新的输出，或者更新现有的输出。

输出的名称可以在 .json 文件中指定，也可以在命令行中指定。如果在两处同时指定，则命令行的名称必须与文件中的名称相同。

如果你所指定的输出已存在，而且你并未指定 –Force 参数，则该 cmdlet 会询问你是否替换现有输出。

如果你指定了 –Force 参数，同时又指定了一个现有的输出名称，则会在不进行确认的情况下替换该输出。

有关 JSON 文件结构和内容的详细信息，请参阅[流分析管理 REST API 参考库][stream.analytics.rest.api.reference]的[创建输出（Azure 流分析）][msdn-rest-api-create-stream-analytics-output]部分。

**示例 1**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Output.json" –JobName StreamingJob –Name output

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Output.json" –JobName StreamingJob –Name output

此 PowerShell 命令在 StreamingJob 作业中创建新的名为“output”的输出。如果已定义了使用此名称的现有输出，则该 cmdlet 会询问是否替换该输出。

**示例 2**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Output.json" –JobName StreamingJob –Name output -Force

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Output.json" –JobName StreamingJob –Name output -Force

此 PowerShell 命令替换作业 StreamingJob 中“output”的定义。

### New-AzureStreamAnalyticsTransformation | New-AzureRMStreamAnalyticsTransformation
在流分析作业中创建新的转换，或者更新现有转换。
  
转换的名称可以在 .json 文件中指定，也可以在命令行中指定。如果在两处同时指定，则命令行的名称必须与文件中的名称相同。

如果你所指定的转换已存在，而且你并未指定 –Force 参数，则该 cmdlet 会询问你是否替换现有转换。

如果你指定了 –Force 参数，同时又指定了一个现有的转换名称，则会在不进行确认的情况下替换转换。

有关 JSON 文件结构和内容的详细信息，请参阅[流分析管理 REST API 参考库][stream.analytics.rest.api.reference]的[创建转换（Azure 流分析）][msdn-rest-api-create-stream-analytics-transformation]部分。

**示例 1**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

此 PowerShell 命令在作业 StreamingJob 中创建名为 StreamingJobTransform 的新转换。如果已定义了使用此名称的现有转换，则该 cmdlet 会询问是否替换该转换。

**示例 2**

Azure PowerShell 0.9.8：

	New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

Azure PowerShell 1.0：

	New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-China-East –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

 此 PowerShell 命令替换作业 StreamingJob 中 StreamingJobTransform 的定义。

### Remove-AzureStreamAnalyticsInput | Remove-AzureRMStreamAnalyticsInput
以异步方式从  Azure 的流分析作业中删除特定的输入。  
如果指定 -Force 参数，则会在不确认的情况下删除输入。

**示例 1**

Azure PowerShell 0.9.8：

	Remove-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name EventStream

Azure PowerShell 1.0：

	Remove-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name EventStream

此 PowerShell 命令删除作业 StreamingJob 中的输入 EventStream。

### Remove-AzureStreamAnalyticsJob | Remove-AzureRMStreamAnalyticsJob
以异步方式删除  Azure 中的特定流分析作业。  
如果指定 -Force 参数，则会在不确认的情况下删除作业。

**示例 1**

Azure PowerShell 0.9.8：

	Remove-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –Name StreamingJob 

Azure PowerShell 1.0：

	Remove-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –Name StreamingJob 

此 PowerShell 命令删除作业 StreamingJob。

### Remove-AzureStreamAnalyticsOutput | Remove-AzureRMStreamAnalyticsOutput
以异步方式从  Azure 的流分析作业中删除特定的输出。  
如果指定 -Force 参数，则会在不确认的情况下删除输出。

**示例 1**

Azure PowerShell 0.9.8：

	Remove-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name Output

Azure PowerShell 1.0：

	Remove-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name Output

此 PowerShell 命令删除作业 StreamingJob 中的输出 Output。

### Start-AzureStreamAnalyticsJob | Start-AzureRMStreamAnalyticsJob
以异步方式部署和启动  Azure 中的流分析作业。

**示例 1**

Azure PowerShell 0.9.8：

	Start-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

Azure PowerShell 1.0：

	Start-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

此 PowerShell 命令启动作业 StreamingJob，并将自定义输出开始时间设置为 2012 年 12 月 12 日 12:12:12（UTC）。


### Stop-AzureStreamAnalyticsJob | Stop-AzureRMStreamAnalyticsJob
以异步方式停止流分析作业在  Azure 中的运行，并取消分配所使用的资源。在订阅中，你将始终可以通过 Azure 门户和管理 API 来使用作业定义和元数据，方便你编辑和重新启动该作业。处于停止状态的作业不收费。

**示例 1**

Azure PowerShell 0.9.8：

	Stop-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –Name StreamingJob 

Azure PowerShell 1.0：

	Stop-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-China-East –Name StreamingJob 

此 PowerShell 命令停止作业 StreamingJob。

### Test-AzureStreamAnalyticsInput | Test-AzureRMStreamAnalyticsInput
测试流分析能否连接到指定的输入。

**示例 1**

Azure PowerShell 0.9.8：

	Test-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0：

	Test-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name EntryStream

此 PowerShell 命令测试 StreamingJob 中输入 EntryStream 的连接状态。

###Test-AzureStreamAnalyticsOutput | Test-AzureRMStreamAnalyticsOutput
测试流分析能否连接到指定的输出。

**示例 1**

Azure PowerShell 0.9.8：

	Test-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name Output

Azure PowerShell 1.0：

	Test-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-China-East –JobName StreamingJob –Name Output

此 PowerShell 命令测试 StreamingJob 中输出 Output 的连接状态。

## 获取支持
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStreamAnalytics)。


## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)
 



[msdn-switch-azuremode]: http://msdn.microsoft.com/zh-cn/library/dn722470.aspx
[powershell-install]: /documentation/articles/powershell-install-configure/
[msdn-rest-api-create-stream-analytics-job]: https://msdn.microsoft.com/zh-cn/library/dn834994.aspx
[msdn-rest-api-create-stream-analytics-input]: https://msdn.microsoft.com/zh-cn/library/dn835010.aspx
[msdn-rest-api-create-stream-analytics-output]: https://msdn.microsoft.com/zh-cn/library/dn835015.aspx
[msdn-rest-api-create-stream-analytics-transformation]: https://msdn.microsoft.com/zh-cn/library/dn835007.aspx

[stream.analytics.introduction]: /documentation/articles/stream-analytics-introduction/
[stream.analytics.get.started]: /documentation/articles/stream-analytics-get-started/
[stream.analytics.developer.guide]: /documentation/articles/stream-analytics-developer-guide/
[stream.analytics.scale.jobs]: /documentation/articles/stream-analytics-scale-jobs/
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://msdn.microsoft.com/zh-cn/library/dn835031.aspx
 

<!---HONumber=Mooncake_0307_2016-->