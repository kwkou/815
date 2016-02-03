<properties 
	pageTitle="使用 PowerShell 在 Azure 云服务中启用诊断 | Windows Azure" 
	description="了解如何使用 PowerShell 为云服务启用诊断" 
	services="cloud-services" 
	documentationCenter=".net" 
	authors="sbtron" 
	manager="" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.date="11/16/2015" 
	wacn.date="01/15/2016"/>


# 使用 PowerShell 在 Azure 云服务中启用诊断

你可以使用 Azure 诊断扩展从云服务收集诊断数据（例如应用程序日志、性能计数器等）。本文介绍如何使用 PowerShell 为云服务启用 Azure 诊断扩展。有关本文所需的先决条件，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

## 在部署云服务过程中启用诊断扩展

此方法适用于可启用诊断扩展的连续集成类型的方案。你可以在部署云服务过程中，通过将 *ExtensionConfiguration* 参数传递到 [New-AzureDeployment](https://msdn.microsoft.com/zh-cn/library/azure/mt589089.aspx) cmdlet，来启用诊断扩展。*ExtensionConfiguration* 参数取值为可通过 [New-AzureServiceDiagnosticsExtensionConfig](https://msdn.microsoft.com/zh-cn/library/azure/mt589168.aspx) cmdlet 创建的诊断配置数组。

以下示例演示如何为某个云服务（其中的 WebRole 和 WorkerRole 拥有不同的诊断配置）启用诊断。

	$service_name = "MyService"
	$service_package = "CloudService.cspkg"
	$service_config = "ServiceConfiguration.Cloud.cscfg"
	$diagnostics_storagename = "myservicediagnostics"
	$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
	$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"

	$primary_storagekey = (Get-AzureStorageKey -StorageAccountName "$diagnostics_storagename").Primary
	$storage_context = New-AzureStorageContext -StorageAccountName $diagnostics_storagename -StorageAccountKey $primary_storagekey

	$webrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WebRole" -Storage_context $storageContext -DiagnosticsConfigurationPath $webrole_diagconfigpath
	$workerrole_diagconfig = New-AzureServiceDiagnosticsExtensionConfig -Role "WorkerRole" -StorageContext $storage_context -DiagnosticsConfigurationPath $workerrole_diagconfigpath
	  
	 
	New-AzureDeployment -ServiceName $service_name -Slot Production -Package $service_package -Configuration $service_config -ExtensionConfiguration @($webrole_diagconfig,$workerrole_diagconfig) 



## 在现有的云服务上启用诊断扩展

可以使用 [Set-AzureServiceDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/azure/mt589140.aspx) cmdlet 在已运行的云服务上启用诊断。


	$service_name = "MyService"
	$diagnostics_storagename = "myservicediagnostics"
	$webrole_diagconfigpath = "MyService.WebRole.PubConfig.xml" 
	$workerrole_diagconfigpath = "MyService.WorkerRole.PubConfig.xml"
	$primary_storagekey = (Get-AzureStorageKey -StorageAccountName "$diagnostics_storagename").Primary
	$storage_context = New-AzureStorageContext -StorageAccountName $diagnostics_storagename -StorageAccountKey $primary_storagekey
 
	Set-AzureServiceDiagnosticsExtension -StorageContext $storage_context -DiagnosticsConfigurationPath $webrole_diagconfigpath -ServiceName $service_name -Slot Production -Role "WebRole" 
	Set-AzureServiceDiagnosticsExtension -StorageContext $storage_context -DiagnosticsConfigurationPath $workerrole_diagconfigpath -ServiceName $service_name -Slot Production -Role "WorkerRole"
 

## 获取当前诊断扩展配置
使用 [Get AzureServiceDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/azure/mt589204.aspx) cmdlet 可以获取云服务的当前诊断配置。
	
	Get-AzureServiceDiagnosticsExtension -ServiceName "MyService"

## 删除诊断扩展
若要在云服务上关闭诊断，可以使用 [Remove-AzureServiceDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/azure/mt589183.aspx) cmdlet。

	Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService"

如果你使用不带 *Role* 参数的 *Set-AzureServiceDiagnosticsExtension* 或 *New-AzureServiceDiagnosticsExtensionConfig* 启用了诊断扩展，则可以使用不带 *Role* 参数的 *Remove-AzureServiceDiagnosticsExtension* 删除该扩展。如果启用扩展时使用了 *Role* 参数，则删除扩展时也必须使用该参数。

若要从单个角色中删除诊断扩展，请使用以下命令：

	Remove-AzureServiceDiagnosticsExtension -ServiceName "MyService" -Role "WebRole"


## 后续步骤

- 有关使用 Azure 诊断和其他方法排查问题的详细说明，请参阅[在 Azure 云服务和虚拟机中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics)。
- [诊断配置架构](https://msdn.microsoft.com/zh-cn/library/azure/dn782207.aspx)说明了诊断扩展的各种 XML 配置选项。
- 若要了解如何为虚拟机启用诊断扩展，请参阅[使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机](/documentation/articles/virtual-machines-extensions-diagnostics-windows-template)  

<!---HONumber=Mooncake_0104_2016-->
