<properties
	pageTitle="管理 Azure 资源管理器 VM | Windows Azure"
	description="使用 Azure 资源管理器、模板与 PowerShell 来管理虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="10/08/2015"
	wacn.date="11/12/2015"/>

# 使用 Azure 资源管理器与 PowerShell 来管理虚拟机

> [AZURE.SELECTOR]
- [Preview Portal](/documentation/articles/virtual-machines-windows-tutorial)
- [PowerShell - Windows](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)
- [Azure CLI](/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli)

管理 Windows Azure 中的资源时，使用 Azure PowerShell 和资源管理器模板可为你提供大量强大功能和灵活性。你可以使用本文中的任务创建和管理虚拟机资源。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)] [classic deployment model](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)。

这些任务使用资源管理器模板和 PowerShell：

- [创建虚拟机](#windowsvm)
- [使用专用磁盘创建虚拟机](#customvm)
- [在一个使用外部负载平衡器的虚拟网络中创建多个虚拟机](#multivm)

这些任务仅使用 PowerShell：

- [删除资源组](#removerg)
- [显示有关虚拟机的信息](#displayvm)
- [启动虚拟机](#start)
- [停止虚拟机](#stop)
- [重新启动虚拟机](#restart)
- [删除虚拟机](#delete)

[AZURE.INCLUDE [powershell 预览](../includes/powershell-preview-inline-include.md)]

## Azure 资源管理器模板和资源组

本文中的一些任务说明如何使用 Azure 资源管理器模板和 PowerShell 来自动部署和管理 Azure 虚拟机。

大多数 Windows Azure 中运行的应用程序是通过不同云资源类型的组合（例如，一个或多个虚拟机和存储帐户、一个 SQL 数据库或一个虚拟网络）构建的。Azure 资源管理器模板使你能够集中管理这些不同的资源，只需对资源和关联的配置及部署参数进行 JSON 描述即可。

定义基于 JSON 的资源模板之后，你就可以对它使用 PowerShell 命令将定义的资源部署到 Azure 中。你可以在 PowerShell 命令 shell 中单独运行这些命令，也可以将其集成到包含其他自动化逻辑的脚本中。

将使用 Azure 资源管理器模板创建的资源部署到新的或现有的 *Azure 资源组*。资源组可让你将部署的多个资源作为一个逻辑组统一进行管理；这样，你就可以管理组/应用程序的整个生命周期。

如果你想要了解如何创作模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。

### 创建资源组

有关创建资源的任务，如果你还没有一个资源组，则你需要创建一个资源组。

在下面的命令中，将*资源组名称*替换为新的资源组名称，将 *Azure 位置*替换为 Azure 数据中心位置（你想存储资源的位置），然后运行该命令：

	New-AzureRmResourceGroup -Name "resource group name" -Location "Azure location"

## <a id="windowsvm"></a>任务：创建虚拟机

此任务使用模板库的模板。若要了解有关该模板的详细信息，请参阅[在美国西部部署简单的 Windows VM](https://azure.microsoft.com/documentation/templates/101-simple-windows-vm/)。

![](./media/virtual-machines-deploy-rmtemplates-powershell/windowsvm.png)

在下面的命令中，将*部署名称*替换为你想要使用的部署名称，将*资源组名称*替换为现有资源组的名称，然后运行该命令：

	New-AzureRmResourceGroupDeployment -Name "deployment name" -ResourceGroupName "resource group name" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-windows-vm/azuredeploy.json"

下面是一个示例：

	New-AzureRmResourceGroupDeployment -Name "TestDeployment" -ResourceGroupName "TestRG" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-simple-windows-vm/azuredeploy.json"

系统会提示你提供 JSON 文件的**参数**部分中的参数值：

	cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	newStorageAccountName: saacct
	adminUsername: WinAdmin1
	adminPassword: *********
	dnsNameForPublicIP: contoso

它会返回类似于下面的内容：

	VERBOSE: 10:56:59 AM - Template is valid.
	VERBOSE: 10:56:59 AM - Create template deployment 'TestDeployment'.
	VERBOSE: 10:57:08 AM - Resource Microsoft.Network/virtualNetworks 'MyVNET' provisioning status is succeeded
	VERBOSE: 10:57:11 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is running
	VERBOSE: 10:57:11 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is running
	VERBOSE: 10:57:38 AM - Resource Microsoft.Storage/storageAccounts 'newsaacct' provisioning status is succeeded
	VERBOSE: 10:57:40 AM - Resource Microsoft.Network/publicIPAddresses 'myPublicIP' provisioning status is succeeded
	VERBOSE: 10:57:45 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is running
	VERBOSE: 10:57:45 AM - Resource Microsoft.Network/networkInterfaces 'myVMNic' provisioning status is succeeded
	VERBOSE: 11:01:59 AM - Resource Microsoft.Compute/virtualMachines 'MyWindowsVM' provisioning status is succeeded


	DeploymentName    : TestDeployment
	ResourceGroupName : TestRG
	ProvisioningState : Succeeded
	Timestamp         : 4/28/2015 6:02:13 PM
	Mode              : Incremental
	TemplateLink      :
	Parameters        :
                    	Name             Type                       Value
	                    ===============  =========================  ==========
	                    newStorageAccountName  String                     saacct
	                    adminUsername    String                     WinAdmin1
	                    adminPassword    SecureString
	                    dnsNameForPublicIP  String                     contoso9875
	                    windowsOSVersion  String                     2012-R2-Datacenter

	Outputs           :





## <a id="customvm"></a>任务：使用专用磁盘创建虚拟机

此任务使用模板库的模板。若要了解有关该模板的详细信息，请参阅[从专用 VHD 磁盘创建 VM](https://www.windowsazure.cn/documentation/templates/201-vm-from-specialized-vhd/)。

在下面的命令中，将*部署名称*替换为你想要使用的部署名称，将*资源组名称*替换为现有资源组的名称，然后运行该命令：

	New-AzureRmResourceGroupDeployment -Name "deployment name" -ResourceGroupName "resource group name" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-from-specialized-vhd/azuredeploy.json"

下面是一个示例：

	New-AzureRmResourceGroupDeployment -Name "TestDeployment" -ResourceGroupName "TestRG" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-from-specialized-vhd/azuredeploy.json"

系统会提示你提供 JSON 文件的**参数**部分中的参数值：

	cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	osDiskVhdUri: http://saacct.blob.core.chinacloudapi.cn/vhds/osdiskforwindows.vhd
	osType: windows
	location: China North
	vmSize: Standard_A3
	...

> [AZURE.NOTE]上述示例使用 saacct 存储帐户中存在的一个 vhd 文件。磁盘名称作为参数向模板提供。
## <a id="multivm"></a>任务：在一个使用外部负载平衡器的虚拟网络中创建多个虚拟机

此任务使用模板库的模板。若要了解有关该模板的详细信息，请参阅[从专用 VHD 磁盘创建 VM](https://www.windowsazure.cn/documentation/templates/201-2-vms-loadbalancer-lbrules/)。

![](./media/virtual-machines-deploy-rmtemplates-powershell/multivmextlb.png)

在下面的命令中，将*部署名称*替换为你想要使用的部署名称，将*资源组名称*替换为现有资源组的名称，然后运行该命令：

	New-AzureRmResourceGroupDeployment -Name "deployment name" -ResourceGroupName "resource group name" -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-loadbalancer-lbrules/azuredeploy.json"

系统会提示你提供 JSON 文件的**参数**部分中的参数值：

	cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	newStorageAccountName: saTest
	adminUserName: WebAdmin1
	adminPassword: *******
	dnsNameforLBIP: web07
	backendPort: 80
	vmNamePrefix: WEBFARM
	...



## <a id="removerg"></a>任务：删除资源组

在下面的命令中，将*资源组名称*替换为你想要删除的资源组名称，然后运行该命令：

	Remove-AzureRmResourceGroup  -Name "resource group name"

> [AZURE.NOTE]可以使用 **-Force** 参数跳过确认提示。

如果你没有使用 -Force 参数，系统会提示你进行确认：

	Confirm
	Are you sure you want to remove resource group 'BuildRG'
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

如果你想要观看完成此任务的视频，请看一看此视频：

[AZURE.VIDEO removing-a-resource-group-in-azure]

## <a id="displayvm"></a>任务：显示有关虚拟机的信息

在下面的命令中，将*资源组名称*替换为包含该虚拟机的资源组名称，将 *VM 名称*替换为该虚拟机名称，然后运行该命令：

	Get-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：

	AvailabilitySetReference : null
	Extensions               : []
	HardwareProfile          : {
	                             "VirtualMachineSize": "Standard_D1"
	                           }
	Id                       : /subscriptions/{subscription-id}/resourceGroups/BuildRG/providers/Microso
	                           ft.Compute/virtualMachines/MyWindowsVM
	InstanceView             : null
	Location                 : westus
	Name                     : MyWindowsVM
	NetworkProfile           : {
	                             "NetworkInterfaces": [
	                               {
	                                 "Primary": null,
	                                 "ReferenceUri": "/subscriptions/{subscription-id}/resourceGroups/Bu
	                           ildRG/providers/Microsoft.Network/networkInterfaces/myVMNic"
	                               }
	                             ]
	                           }
	OSProfile                : {
	                             "AdminPassword": null,
	                             "AdminUsername": "WinAdmin1",
	                             "ComputerName": "MyWindowsVM",
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
	StorageProfile           : {
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
	                                 "Uri": "http://buildsaacct.blob.core.chinacloudapi.cn/vhds/osdiskforwindowssimple.vhd"
	                               }
	                             },
	                             "SourceImage": null
	                           }
	Tags                     : {}
	Type                     : Microsoft.Compute/virtualMachines


## <a id="start"></a>任务：启动虚拟机

在下面的命令中，将*资源组名称*替换为包含该虚拟机的资源组名称，将 *VM 名称*替换为该虚拟机名称，然后运行该命令：

	Start-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：

	EndTime             : 4/28/2015 11:11:41 AM -07:00
	Error               :
	Output              :
	StartTime           : 4/28/2015 11:10:35 AM -07:00
	Status              : Succeeded
	TrackingOperationId : e1705973-d266-467e-8655-920016145347
	RequestId           : aac41de1-b85d-4429-9a3d-040b922d2e6d
	StatusCode          : OK

## <a id="stop"></a>任务：停止虚拟机

在下面的命令中，将*资源组名称*替换为包含该虚拟机的资源组名称，将 *VM 名称*替换为该虚拟机名称，然后运行该命令：

	Stop-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

系统会提示你进行确认：

	Virtual machine stopping operation
	This cmdlet will stop the specified virtual machine. Do you want to continue?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

它会返回类似于下面的内容：

	EndTime             : 4/28/2015 11:09:08 AM -07:00
	Error               :
	Output              :
	StartTime           : 4/28/2015 11:06:55 AM -07:00
	Status              : Succeeded
	TrackingOperationId : 0c94dc74-c553-412c-a187-108bdb29657e
	RequestId           : 5cc9ddba-0643-4b5e-82b6-287b321394ee
	StatusCode          : OK

## <a id="restart"></a>任务：重新启动虚拟机

在下面的命令中，将*资源组名称*替换为包含该虚拟机的资源组名称，将 *VM 名称*替换为该虚拟机名称，然后运行该命令：

	Restart-AzureRmVM -ResourceGroupName "resource group name" -Name "VM name"

它会返回类似于下面的内容：

	EndTime             : 4/28/2015 11:16:26 AM -07:00
	Error               :
	Output              :
	StartTime           : 4/28/2015 11:16:25 AM -07:00
	Status              : Succeeded
	TrackingOperationId : 390571e0-c804-43ce-88c5-f98e0feb588e
	RequestId           : 7dac33e3-0164-4a08-be33-96205284cb0b
	StatusCode          : OK

## <a id="delete"></a>任务：删除虚拟机

在下面的命令中，将*资源组名称*替换为包含该虚拟机的资源组名称，将 *VM 名称*替换为该虚拟机名称，然后运行该命令：

	Remove-AzureRmVM -ResourceGroupName "resource group name" –Name "VM name"

> [AZURE.NOTE]可以使用 **-Force** 参数跳过确认提示。

如果你没有使用 -Force 参数，系统会提示你进行确认：

	Virtual machine removal operation
	This cmdlet will remove the specified virtual machine. Do you want to continue?
	[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):

它会返回类似于下面的内容：

	EndTime             : 4/28/2015 11:21:55 AM -07:00
	Error               :
	Output              :
	StartTime           : 4/28/2015 11:20:13 AM -07:00
	Status              : Succeeded
	TrackingOperationId : f74fad9e-f6bc-46ae-82b1-bfad3952aa44
	RequestId           : 6a30d2e0-63ca-43cf-975b-058631e048e7
	StatusCode          : OK

## 其他资源
[Azure 快速入门模板](http://www.windowsazure.cn/documentation/templates/)和[应用程序框架](/documentation/articles/virtual-machines-app-frameworks)

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

[Azure 资源管理器概述](/documentation/articles/resource-group-overview)

[虚拟机文档](http://www.windowsazure.cn/documentation/services/virtual-machines/)

<!---HONumber=79-->