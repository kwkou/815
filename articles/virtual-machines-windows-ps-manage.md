<!-- ARM: tested -->

<properties
	pageTitle="管理 Azure Resource Manager VM | Azure"
	description="使用 Azure 资源管理器、模板与 PowerShell 来管理虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="01/07/2016"
	wacn.date=""/>

# 使用 Azure 资源管理器与 PowerShell 来管理虚拟机

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-machines-windows-ps-manage)
- [CLI](/documentation/articles/virtual-machines-linux-cli-deploy-templates)

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

<br>


管理 Azure 中的资源时，可充分利用 Azure PowerShell 和 Resource Manager 模板的强大功能和灵活性。你可以使用本文中的任务管理虚拟机资源。

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)。

这些任务仅使用 PowerShell：

- [显示有关虚拟机的信息](#displayvm)
- [启动虚拟机](#start)
- [停止虚拟机](#stop)
- [重新启动虚拟机](#restart)
- [删除虚拟机](#delete)

[AZURE.INCLUDE [powershell 预览](../includes/powershell-preview-inline-include.md)]

## <a id="displayvm"></a>显示有关虚拟机的信息

在下面的命令中，将“资源组名称”替换为包含该虚拟机的资源组名称，将 “VM 名称”替换为该虚拟机名称，然后运行该命令：

	Get-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：


	ResourceGroupName        : rg1
	Id                       : /subscriptions/{subscription-id}/resourceGroups/
															rg1/providers/Microsoft.Compute/virtualMachines/vm1
	Name                     : vm1
	Type                     : Microsoft.Azure.Management.Compute.Models.VirtualMachineGetResponse
	Location                 : chinanorth
	Tags                     : {}
	AvailabilitySetReference : null
	Extensions               : []
	HardwareProfile          :  {
																"VirtualMachineSize": "Standard_D1"
															}
	InstanceView             : null
	Location                 : chinanorth
	Name                     : vm1
	NetworkProfile           :  {
																"NetworkInterfaces": [
																	{
																		"Primary": null,
																		"ReferenceUri": "/subscriptions/{subscription-id}/resourceGroups/
																		rg1/providers/Microsoft.Network/networkInterfaces/nc1"
																	}
																]
															}
	OSProfile                :  {
																"AdminPassword": null,
																"AdminUsername": "WinAdmin1",
																"ComputerName": "vm1",
																"CustomData": null,
																"LinuxConfiguration": null,
																"Secrets": [],
																"WindowsConfiguration": {
																	"AdditionalUnattendContents": [],
																	"EnableAutomaticUpdates": true,
																	"ProvisionVMAgent": true,
																	"TimeZone": null,
																	"WinRMConfiguration": null
																}
															}
	Plan                     : null
	ProvisioningState        : Succeeded
	StorageProfile           : 	{
																"DataDisks": [],
																"ImageReference": {
																	"Offer": "WindowsServer",
																	"Publisher": "MicrosoftWindowsServer",
																	"Sku": "2012-R2-Datacenter",
																	"Version": "latest"
																},
																"OSDisk": {
																	"OperatingSystemType": "Windows",
																	"Caching": "ReadWrite",
																	"CreateOption": "FromImage",
																	"Name": "osdisk",
																	"SourceImage": null,
																	"VirtualHardDisk": {
																		"Uri": "http://sa1.blob.core.chinacloudapi.cn/vhds/osdisk1.vhd"
																	}
																}
															}
	DataDiskNames            :  {}
	NetworkInterfaceIDs      : { /subscriptions/{subscription-id}/resourceGroups/
																rg1/providers/Microsoft.Network/networkInterfaces/nc1}

如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO displaying-information-about-a-virtual-machine-in-microsoft-azure-with-powershell]

## <a id="start"></a>启动虚拟机

在下面的命令中，将“资源组名称”替换为包含该虚拟机的资源组名称，将 “VM 名称”替换为该虚拟机名称，然后运行该命令：

	Start-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：

	Status              : Succeeded
	StatusCode          : OK
	RequestId           : 06935ddf-6e89-48d2-b46a-229493e3e9d1
	Output              :
	Error               :
	StartTime           : 4/28/2015 11:10:35 AM -07:00
	EndTime             : 4/28/2015 11:11:41 AM -07:00
	TrackingOperationId : c1aa0a70-4f4f-4d6c-a8ac-7ea35c004ce0

如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="stop"></a>停止虚拟机

在下面的命令中，将“资源组名称”替换为包含该虚拟机的资源组名称，将 “VM 名称”替换为该虚拟机名称，然后运行该命令：

	Stop-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

系统会提示你进行确认：

	Virtual machine stopping operation
	This cmdlet will stop the specified virtual machine. Do you want to continue?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

它会返回类似于下面的内容：

	Status              : Succeeded
	StatusCode          : OK
	RequestId           : aac41de1-b85d-4429-9a3d-040b922d2e6d
	Output              :
	Error               :
	StartTime           : 4/28/2015 11:10:35 AM -07:00
	EndTime             : 4/28/2015 11:11:41 AM -07:00
	TrackingOperationId : e1705973-d266-467e-8655-920016145347

如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="restart"></a>重新启动虚拟机

在下面的命令中，将“资源组名称”替换为包含该虚拟机的资源组名称，将 “VM 名称”替换为该虚拟机名称，然后运行该命令：

	Restart-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：

	Status              : Succeeded
	StatusCode          : OK
	RequestId           : 4b05891c-fdff-4c9a-89ca-e4f1d7691aed
	Output              :
	Error               :
	StartTime           : 1/5/2016 12:06:53 PM -08:00
	EndTime             : 1/5/2016 12:06:54 PM -08:00
	TrackingOperationId : 5aeeab89-45ab-41b9-84ef-9e9a7e732207


如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

## <a id="delete"></a>删除虚拟机

在下面的命令中，将“资源组名称”替换为包含该虚拟机的资源组名称，将 “VM 名称”替换为该虚拟机名称，然后运行该命令：

	Remove-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

> [AZURE.NOTE] 可以使用 **-Force** 参数跳过确认提示。

如果你没有使用 -Force 参数，系统会提示你进行确认：

	Virtual machine removal operation
	This cmdlet will remove the specified virtual machine. Do you want to continue?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

它会返回类似于下面的内容：

	Status              : Succeeded
	StatusCode          : OK
	RequestId           : 2d723b40-ce1f-4b11-a603-dc659a13b6f0
	Output              :
	Error               :
	StartTime           : 1/5/2016 12:10:28 PM -08:00
	EndTime             : 1/5/2016 12:12:12 PM -08:00
	TrackingOperationId : d138ab29-83bf-4948-9d13-dab87db1a639

如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO start-stop-restart-and-delete-vms-in-microsoft-azure-with-powershell]

<!---HONumber=Mooncake_0411_2016-->