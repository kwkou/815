<!-- ARM: tested -->

<properties 
   pageTitle="在资源管理器中使用 PowerShell 部署多 NIC VM | Azure"
   description="了解如何在资源管理器中使用 PowerShell 部署多 NIC VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="06/29/2016"/>

#使用 PowerShell 部署多 NIC VM

[AZURE.INCLUDE [virtual-network-deploy-multinic-arm-selectors-include.md](../includes/virtual-network-deploy-multinic-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/virtual-network-deploy-multinic-classic-ps/)。

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../includes/virtual-network-deploy-multinic-scenario-include.md)]

由于目前你不能使用具有单个 NIC 的 VM 以及具有同一个资源组中的多个 NIC 的 VM，因此你将在一个资源组中实现后端服务器，而在其他资源组中实现所有其他组件。以下步骤使用名为 *IaaSStory* 的资源组作为主资源组，并在名为 *IaaSStory-BackEnd* 的资源组中实现后端服务器。

##<a name="Prerequisites"></a> 先决条件

在部署后端服务器之前，你需要先使用此方案的所有必需资源部署主资源组。若要部署这些资源，请按照以下步骤操作。

1. 导航到[模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/IaaS-Story/11-MultiNIC)。
2. 在模板页中“父资源组”的右侧，单击“部署到 Azure”。
3. 如果需要，更改参数值，然后按照 Azure 门户预览中的步骤部署资源组。

> [AZURE.IMPORTANT]请确保你的存储帐户名称是唯一的。不能在 Azure 中有重复的存储帐户名称。

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]

## 部署后端 VM

后端 VM 取决于下面列出的资源的创建。

- **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保你部署的 Azure 位置支持高级存储。
- **NIC**。每个 VM 都将具有两个 NIC，一个用于数据库访问，另一个用于管理。
- **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。  

### 步骤 1 - 启动脚本

你可以在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/arm/virtual-network-deploy-multinic-arm-ps.ps1)下载所用的完整 PowerShell 脚本。请按照以下步骤更改要在你的环境中使用的脚本。

1. 基于你在上面[先决条件](#Prerequisites)中部署的现有资源组更改以下变量的值。

		$existingRGName        = "IaaSStory"
		$location              = "China North"
		$vnetName              = "WTestVNet"
		$backendSubnetName     = "BackEnd"
		$remoteAccessNSGName   = "NSG-RemoteAccess"
		$stdStorageAccountName = "wtestvnetstoragestd"

2. 基于要用于后端部署的值更改以下变量的值。

		$backendRGName         = "IaaSStory-Backend"
		$prmStorageAccountName = "wtestvnetstorageprm"
		$avSetName             = "ASDB"
		$vmSize                = "Standard_DS3"
		$publisher             = "MicrosoftSQLServer"
		$offer                 = "SQL2014SP1-WS2012R2"
		$sku                   = "Standard"
		$version               = "latest"
		$vmNamePrefix          = "DB"
		$osDiskPrefix          = "osdiskdb"
		$dataDiskPrefix        = "datadisk"
		$diskSize	           = "120"	
		$nicNamePrefix         = "NICDB"
		$ipAddressPrefix       = "192.168.2."
		$numberOfVMs           = 2

3. 检索你的部署所需的现有资源。

		$vnet                  = Get-AzureVirtualNetwork -Name $vnetName -ResourceGroupName $existingRGName
		$backendSubnet         = $vnet.Subnets|?{$_.Name -eq $backendSubnetName}
		$remoteAccessNSG       = Get-AzureNetworkSecurityGroup -Name $remoteAccessNSGName -ResourceGroupName $existingRGName
		$stdStorageAccount     = Get-AzureStorageAccount -Name $stdStorageAccountName -ResourceGroupName $existingRGName

### 步骤 2 - 为你的 VM 创建必要的资源

你需要为数据磁盘创建一个新的资源组和存储帐户，并为所有 VM 创建一个可用性集。你还需要每个 VM 的本地管理员帐户凭据。若要创建这些资源，请执行以下步骤。

1. 创建新的资源组。

		New-AzureResourceGroup -Name $backendRGName -Location $location

2. 在上面创建的资源组中创建新的高级存储帐户。

		$prmStorageAccount = New-AzureStorageAccount -Name $prmStorageAccountName `
			-ResourceGroupName $backendRGName -Type Premium_LRS -Location $location

3. 创建新的可用性集

		$avSet = New-AzureAvailabilitySet -Name $avSetName -ResourceGroupName $backendRGName -Location $location

4. 获取要用于每个 VM 的本地管理员帐户凭据。

		$cred = Get-Credential -Message "Type the name and password for the local administrator account."

### 步骤 3 - 创建 NIC 和后端 VM

你需要使用循环来根据需要创建任意多的 VM，并在循环中创建所需的 NIC 和 VM。若要创建 NIC 和 VM，请执行以下步骤。

1. 启动 `for` 循环以基于 `$numberOfVMs` 变量的值重复执行命令以根据需要创建任意多次一个 VM 和两个 NIC。

		for ($suffixNumber = 1; $suffixNumber -le $numberOfVMs; $suffixNumber++){

2. 创建用于数据库访问的 NIC。
		
		    $nic1Name = $nicNamePrefix + $suffixNumber + "-DA"
		    $ipAddress1 = $ipAddressPrefix + ($suffixNumber + 3)
		    $nic1 = New-AzureNetworkInterface -Name $nic1Name -ResourceGroupName $backendRGName `
				-Location $location -SubnetId $backendSubnet.Id -PrivateIpAddress $ipAddress1

3. 创建用于远程访问的 NIC。请注意此 NIC 的 NSG 关联到它的方式。

		    $nic2Name = $nicNamePrefix + $suffixNumber + "-RA"
		    $ipAddress2 = $ipAddressPrefix + (53 + $suffixNumber)
		    $nic2 = New-AzureNetworkInterface -Name $nic2Name -ResourceGroupName $backendRGName `
				-Location $location -SubnetId $backendSubnet.Id -PrivateIpAddress $ipAddress2 `
				-NetworkSecurityGroupId $remoteAccessNSG.Id

4. 创建 `vmConfig` 对象。

		    $vmName = $vmNamePrefix + $suffixNumber
		    $vmConfig = New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avSet.Id

5. 为每个 VM 创建两个数据磁盘。请注意，数据磁盘位于前面创建的高级存储帐户中。

		    $dataDisk1Name = $vmName + "-" + $osDiskPrefix + "-1"    
		    $data1VhdUri = $prmStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $dataDisk1Name + ".vhd"
		    Add-AzureRmVMDataDisk -VM $vmConfig -Name $dataDisk1Name -DiskSizeInGB $diskSize `
				-VhdUri $data1VhdUri -CreateOption empty -Lun 0

		    $dataDisk2Name = $vmName + "-" + $dataDiskPrefix + "-2"    
		    $data2VhdUri = $prmStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $dataDisk2Name + ".vhd"
		    Add-AzureRmVMDataDisk -VM $vmConfig -Name $dataDisk2Name -DiskSizeInGB $diskSize `
				-VhdUri $data2VhdUri -CreateOption empty -Lun 1

6. 配置要用于 VM 的操作系统和映像。
		    
		    $vmConfig = Set-AzureVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
		    $vmConfig = Set-AzureVMSourceImage -VM $vmConfig -PublisherName $publisher -Offer $offer -Skus $sku -Version $version

7. 将上面创建的两个 NIC 添加到 `vmConfig` 对象。

		    $vmConfig = Add-AzureVMNetworkInterface -VM $vmConfig -Id $nic1.Id -Primary
		    $vmConfig = Add-AzureVMNetworkInterface -VM $vmConfig -Id $nic2.Id

8. 创建 OS 磁盘，并创建 VM。请注意以 `}` 结束 `for` 循环。

		    $osDiskName = $vmName + "-" + $osDiskSuffix
		    $osVhdUri = $stdStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $osDiskName + ".vhd"
		    $vmConfig = Set-AzureVMOSDisk -VM $vmConfig -Name $osDiskName -VhdUri $osVhdUri -CreateOption fromImage
		    New-AzureVM -VM $vmConfig -ResourceGroupName $backendRGName -Location $location
		}

### 步骤 4 - 运行脚本

现在，你已根据需要下载并更改了脚本，请运行该脚本以创建具有多个 NIC 的后端数据库 VM。

1. 保存脚本并从 **PowerShell** 命令提示符或 **PowerShell ISE** 运行它。你将看到最初的输出，如下所示。

		ResourceGroupName : IaaSStory-Backend
		Location          : chinanorth
		ProvisioningState : Succeeded
		Tags              : 
		Permissions       : 
		                    Actions  NotActions
		                    =======  ==========
		                    *                  
		                    
		ResourceId        : /subscriptions/628dad04-b5d1-4f10-b3a4-dc61d88cf97c/resourceGroups/IaaSStory-Backend

2. 几分钟后，填写提示的凭据，然后单击“确定”。下面的输出表示单个 VM。请注意，整个过程需要 8 分钟才能完成。

		ResourceGroupName            : 
		Id                           : 
		Name                         : DB2
		Type                         : 
		Location                     : 
		Tags                         : 
		TagsText                     : null
		AvailabilitySetReference     : Microsoft.Azure.Management.Compute.Models.AvailabilitySetReference
		AvailabilitySetReferenceText : {
		                                 "ReferenceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/
		                               Microsoft.Compute/availabilitySets/ASDB"
		                               }
		Extensions                   : 
		ExtensionsText               : null
		HardwareProfile              : Microsoft.Azure.Management.Compute.Models.HardwareProfile
		HardwareProfileText          : {
		                                 "VirtualMachineSize": "Standard_DS3"
		                               }
		InstanceView                 : 
		InstanceViewText             : null
		NetworkProfile               : 
		NetworkProfileText           : null
		OSProfile                    : 
		OSProfileText                : null
		Plan                         : 
		PlanText                     : null
		ProvisioningState            : 
		StorageProfile               : Microsoft.Azure.Management.Compute.Models.StorageProfile
		StorageProfileText           : {
		                                 "DataDisks": [
		                                   {
		                                     "Lun": 0,
		                                     "Caching": null,
		                                     "CreateOption": "empty",
		                                     "DiskSizeGB": 127,
		                                     "Name": "DB2-datadisk-1",
		                                     "SourceImage": null,
		                                     "VirtualHardDisk": {
		                                       "Uri": "https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/DB2-datadisk-1.vhd"
		                                     }
		                                   }
		                                 ],
		                                 "ImageReference": null,
		                                 "OSDisk": null
		                               }
		DataDiskNames                : {DB2-datadisk-1}
		NetworkInterfaceIDs          : 
		RequestId                    : 
		StatusCode                   : 0
		
		
		ResourceGroupName            : 
		Id                           : 
		Name                         : DB2
		Type                         : 
		Location                     : 
		Tags                         : 
		TagsText                     : null
		AvailabilitySetReference     : Microsoft.Azure.Management.Compute.Models.AvailabilitySetReference
		AvailabilitySetReferenceText : {
		                                 "ReferenceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/
		                               Microsoft.Compute/availabilitySets/ASDB"
		                               }
		Extensions                   : 
		ExtensionsText               : null
		HardwareProfile              : Microsoft.Azure.Management.Compute.Models.HardwareProfile
		HardwareProfileText          : {
		                                 "VirtualMachineSize": "Standard_DS3"
		                               }
		InstanceView                 : 
		InstanceViewText             : null
		NetworkProfile               : 
		NetworkProfileText           : null
		OSProfile                    : 
		OSProfileText                : null
		Plan                         : 
		PlanText                     : null
		ProvisioningState            : 
		StorageProfile               : Microsoft.Azure.Management.Compute.Models.StorageProfile
		StorageProfileText           : {
		                                 "DataDisks": [
		                                   {
		                                     "Lun": 0,
		                                     "Caching": null,
		                                     "CreateOption": "empty",
		                                     "DiskSizeGB": 127,
		                                     "Name": "DB2-datadisk-1",
		                                     "SourceImage": null,
		                                     "VirtualHardDisk": {
		                                       "Uri": "https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/DB2-datadisk-1.vhd"
		                                     }
		                                   },
		                                   {
		                                     "Lun": 1,
		                                     "Caching": null,
		                                     "CreateOption": "empty",
		                                     "DiskSizeGB": 127,
		                                     "Name": "DB2-datadisk-2",
		                                     "SourceImage": null,
		                                     "VirtualHardDisk": {
		                                       "Uri": "https://wtestvnetstorageprm.blob.core.chinacloudapi.cn/vhds/DB2-datadisk-2.vhd"
		                                     }
		                                   }
		                                 ],
		                                 "ImageReference": null,
		                                 "OSDisk": null
		                               }
		DataDiskNames                : {DB2-datadisk-1, DB2-datadisk-2}
		NetworkInterfaceIDs          : 
		RequestId                    : 
		StatusCode                   : 0
		
		
		EndTime             : 10/30/2015 9:30:03 AM -08:00
		Error               : 
		Output              : 
		StartTime           : 10/30/2015 9:22:54 AM -08:00
		Status              : Succeeded
		TrackingOperationId : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		RequestId           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		StatusCode          : OK

<!---HONumber=Mooncake_1221_2015-->