<!-- ARM: tested -->

<properties 
	pageTitle="使用 PowerShell 在运行 Windows 的 Azure 虚拟机中启用诊断 | Microsoft Azure"
	description="了解如何使用 PowerShell 在运行 Windows 的 Azure 虚拟机中启用诊断"
	services="virtual-machines-windows" 
	documentationCenter=""
	authors="sbtron"
	manager=""
	editor=""/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/15/2015"
	wacn.date="06/07/2016"/>


# 使用 PowerShell 在运行 Windows 的 Azure 虚拟机中启用诊断

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

你可以使用 Azure 诊断扩展从运行 Windows 的 Azure 虚拟机收集诊断数据（例如应用程序日志、性能计数器等）。本文介绍如何使用 PowerShell 为虚拟机启用 Azure 诊断扩展。有关本文所需的先决条件，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 使用资源管理器部署模型在虚拟机上启用 Azure 诊断扩展

你可以使用资源管理器部署模型在创建 Windows 虚拟机的同时启用诊断扩展，只需将扩展配置添加到资源管理器模板即可。请参阅[使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/)。

若要在通过资源管理器部署模型创建的现有虚拟机上启用 Azure 诊断扩展，可以使用 [Set-AzureRMVMDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/mt603499.aspx) powershell cmdlet，如下所示。


	$vm_resourcegroup = "myvmresourcegroup"
	$vm_name = "myvm"
	$diagnosticsconfig_path = "DiagnosticsPubConfig.xml"
	
	Set-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path 


*$diagnosticsconfig\_path* 是一个路径，指向内含 XML 格式诊断配置的文件，如以下[示例](#sample-diagnostics-configuration)所示。

如果诊断配置文件使用某个存储帐户名称指定了 **StorageAccount** 元素，则 *Set-AzureRMVMDiagnosticsExtension* 脚本会自动将诊断扩展设置为将诊断数据发送到该存储帐户。存储帐户必须与虚拟机属于同一订阅，此功能才会起作用。

如果没有在诊断配置中指定 **StorageAccount**，则必须将 *StorageAccountName* 参数传递给 cmdlet。如果指定了 *StorageAccountName* 参数，则 cmdlet 会始终使用该参数中指定的存储帐户，而不使用诊断配置文件中指定的。

如果诊断存储帐户与虚拟机属于不同的订阅，则必须将 *StorageAccountName* 和 *StorageAccountKey* 参数显式传递给 cmdlet。当诊断存储帐户属于同一订阅时，不需要 *StorageAccountKey* 参数，因为在启用诊断扩展的情况下，cmdlet 可以自动查询和设置密钥值。但是，如果诊断存储帐户属于不同的订阅，则 cmdlet 可能无法自动获取密钥，你必须通过 *StorageAccountKey* 参数显式指定该密钥。

	Set-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName $diagnosticsstorage_name -StorageAccountKey $diagnosticsstorage_key	

在 VM 上启用 Azure 诊断扩展以后，即可使用 [Get-AzureRMVmDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/mt603678.aspx) cmdlet 获取当前设置。

	Get-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name

该 cmdlet 返回的 *PublicSettings* 包含 base64 编码格式的 XML 配置。若要读取该 XML，需对其解码。
	
	$publicsettings = (Get-AzureRmVMDiagnosticsExtension -ResourceGroupName $vm_resourcegroup -VMName $vm_name).PublicSettings
	$encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
	$xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
	Write-Host $xmlconfig
 
可以使用 [Remove-AzureRMVmDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/mt603782.aspx) cmdlet 从 VM 中删除该诊断扩展。
  
## 使用经典部署模型在虚拟机上启用 Azure 诊断扩展

可以使用 [Set-AzureVMDiagnosticsExtension](https://msdn.microsoft.com/zh-cn/library/mt589189.aspx) cmdlet 在通过经典部署模型创建的虚拟机上启用 Azure 诊断扩展。下面的示例演示了在启用 Azure 诊断扩展的情况下，如何通过经典部署模型创建新的虚拟机。

	$VM = New-AzureVMConfig -Name $VM -InstanceSize Small -ImageName $VMImage
	$VM = Add-AzureProvisioningConfig -VM $VM -AdminUsername $Username -Password $Password -Windows
	$VM = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
	New-AzureVM -Location $Location -ServiceName $Service_Name -VM $VM

若要在现有虚拟机（经典）上启用 Azure 诊断扩展，请先使用 [Get-AzureVM](https://msdn.microsoft.com/zh-cn/library/mt589152.aspx) cmdlet 获取虚拟机配置。然后，使用 [Set-AzureVMDiagnosticsExtension](https://msdn.microsoft.com/library/mt589189.aspx) cmdlet 更新虚拟机配置，使之包括 Azure 诊断扩展。最后，通过 [Update-AzureVM](https://msdn.microsoft.com/library/mt589121.aspx) 将更新的配置应用于虚拟机。

	$VM = Get-AzureVM -ServiceName $Service_Name -Name $VM_Name
	$VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $Config_Path -VM $VM -StorageContext $Storage_Context
	Update-AzureVM -ServiceName $Service_Name -Name $VM_Name -VM $VM_Update.VM

##<a name="sample-diagnostics-configuration"></a> 诊断配置示例

以下 XML 可用于上述脚本的诊断公共配置。此示例配置会将各种性能计数器传输给诊断存储帐户，同时还会传输 Windows 事件日志的应用程序、安全和系统通道中的错误，以及诊断基础结构日志中的任何错误。

需要对该配置进行更新，以便包括以下内容：

- **Metrics** 元素的 *resourceID* 属性需要使用虚拟机的资源 ID 进行更新。 
	- 资源 ID 可以使用以下模式构造：“/subscriptions/{*VM 订阅的订阅 ID*}/resourceGroups/{*VM 的资源组名称*}/providers/Microsoft.Compute/virtualMachines/{*VM 名称*}”。 
	- 例如，如果在其中运行 VM 的订阅的订阅 ID 为 **11111111-1111-1111-1111-111111111111**，资源组的资源组名称为 **MyResourceGroup**，VM 名称为 **MyWindowsVM**，则 *resourceID* 的值为：
	 
		```
		<Metrics resourceId="/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/MyWindowsVM" >
		```
	- 有关如何根据性能计数器和度量值配置生成度量值的详细信息，请参阅[存储中的 WAD 度量值表](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/#wadmetrics-tables-in-storage)

- 需要使用诊断存储帐户的名称对 **StorageAccount** 元素进行更新。
 
	```
	<?xml version="1.0" encoding="utf-8"?>
	<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <WadCfg>
	      <DiagnosticMonitorConfiguration overallQuotaInMB="4096">
	        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error"/>
	        <PerformanceCounters scheduledTransferPeriod="PT1M">
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU utilization" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Privileged Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU privileged time" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% User Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="CPU user time" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Processor Information(_Total)\Processor Frequency" sampleRate="PT15S" unit="Count">
	        <annotation displayName="CPU frequency" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\System\Processes" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Processes" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Thread Count" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Threads" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Process(_Total)\Handle Count" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Handles" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes In Use" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Memory usage" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory available" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory committed" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Commit Limit" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory commit limit" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Paged Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory paged pool" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\Memory\Pool Nonpaged Bytes" sampleRate="PT15S" unit="Bytes">
	        <annotation displayName="Memory non-paged pool" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active time" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Read Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active read time" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\% Disk Write Time" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk active write time" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Transfers/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk operations" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Reads/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk read operations" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Writes/sec" sampleRate="PT15S" unit="CountPerSecond">
	        <annotation displayName="Disk write operations" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk speed" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Read Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk read speed" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Disk Write Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond">
	        <annotation displayName="Disk write speed" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average queue length" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Read Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average read queue length" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\PhysicalDisk(_Total)\Avg. Disk Write Queue Length" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk average write queue length" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\% Free Space" sampleRate="PT15S" unit="Percent">
	        <annotation displayName="Disk free space (percentage)" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	      <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(_Total)\Free Megabytes" sampleRate="PT15S" unit="Count">
	        <annotation displayName="Disk free space (MB)" locale="en-us"/>
	      </PerformanceCounterConfiguration>
	    </PerformanceCounters>
		<Metrics resourceId="(Update with resource ID for the VM)" >
	        <MetricAggregation scheduledTransferPeriod="PT1H"/>
	        <MetricAggregation scheduledTransferPeriod="PT1M"/>
	    </Metrics>
	    <WindowsEventLog scheduledTransferPeriod="PT1M">
	      <DataSource name="Application!*[System[(Level = 1 or Level = 2)]]"/>
	      <DataSource name="Security!*[System[(Level = 1 or Level = 2)]"/>
	      <DataSource name="System!*[System[(Level = 1 or Level = 2)]]"/>
	    </WindowsEventLog>
	      </DiagnosticMonitorConfiguration>
	    </WadCfg>
	    <StorageAccount>(Update with diagnostics storage account name)</StorageAccount>
	</PublicConfig>
	```

## 后续步骤 
- 有关使用 Azure 诊断和其他方法排查问题的详细说明，请参阅[在 Azure 云服务和虚拟机中启用诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)。
- [诊断配置架构](https://msdn.microsoft.com/zh-cn/library/azure/mt634524.aspx)说明了诊断扩展的各种 XML 配置选项。

<!---HONumber=Mooncake_0118_2016-->