<properties 
	pageTitle="未注册 Resource Provider 异常" 
	description="通过注册 Resource Provicder 解决使用 PowerShell 出现模块未注册的异常" 
	service=""
	resource=""
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="PowerShell, Reource Provider"​
    cloudEnvironments="MoonCake" 
/>
<tags 
	ms.service="na-aog"
	ms.date="" 
	wacn.date="01/12/2017"
/>
# 未注册 Resource Provider 异常
## **问题描述**

当执行 PowerShell 命令有时会遇到此异常信息：

	cmdlet Set-AzureRmRedisCacheDiagnostics at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	ResourceGroupName: redispms01
	Name: redisstpms01
	StorageAccountId: "/subscriptions/8e1c9497-3e47-4f59-ab04-943aa4661a7b/resourceGroups/redispms01/providers/Microsoft.Storage/storageAccounts/redis01storagearm"
	Set-AzureRmRedisCacheDiagnostics : RdfeClientError: Resource not found at specified location.

## **问题分析**

该类型错误是因为没有注册执行 Azure PowerShell 命令对应的模块导致的。

## **解决方法**

执行以下命令注册相应的模块 : 

	Register-AzureRmResourceProvider -ProviderNamespace microsoft.insights

>[AZURE.NOTE] 该命令中 microsoft.insights 为示例，实际情况需要根据自己使用的命令来判断具体的名称。可以使用 `Get-AzureRmResourceProvider` 命令查看所有的 ResourceProvider 信息。
