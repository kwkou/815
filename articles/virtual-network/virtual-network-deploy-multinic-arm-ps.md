<properties
    pageTitle="使用 PowerShell 创建具有多个 NIC 的 VM | Azure"
    description="了解如何使用 PowerShell 通过 Azure Resource Manager 创建具有多个 NIC 的 VM。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="88880483-8f9e-4eeb-b783-64b8613407d9"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/02/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 PowerShell 创建具有多个 NIC 的 VM
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)
- [Azure CLI](/documentation/articles/virtual-network-deploy-multinic-arm-cli/)
- [模板](/documentation/articles/virtual-network-deploy-multinic-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-network-deploy-multinic-classic-ps/)
- [Azure CLI（经典）](/documentation/articles/virtual-network-deploy-multinic-classic-cli/)

[AZURE.INCLUDE [virtual-network-deploy-multinic-intro-include.md](../../includes/virtual-network-deploy-multinic-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是[经典部署模型](/documentation/articles/virtual-network-deploy-multinic-classic-ps/)。
>

[AZURE.INCLUDE [virtual-network-deploy-multinic-scenario-include.md](../../includes/virtual-network-deploy-multinic-scenario-include.md)]

以下步骤将名为 *IaaSStory* 的资源组用于 Web 服务器，并将名为 *IaaSStory-BackEnd* 的资源组用于数据库服务器。

## <a name="Prerequisites"></a> 先决条件
需要先创建具有此方案需要的所有资源的 *IaaSStory* 资源组，然后才能创建数据库服务器。若要创建这些资源，请完成以下步骤：

1. 导航到[模板页](https://github.com/Azure/azure-quickstart-templates/tree/master/IaaS-Story/11-MultiNIC)。
2. 在模板页中“父资源组”的右侧，单击“部署到 Azure”。
3. 如果需要，更改参数值，然后按照 Azure 门户中的步骤部署资源组。

> [AZURE.IMPORTANT]
请确保存储帐户名称是唯一的。Azure 中不能存在重复的存储帐户名称。
> 

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## 创建后端 VM
后端 VM 取决于以下资源的创建：

* **数据磁盘的存储帐户**。为了提高性能，数据库服务器上的数据磁盘将使用固态驱动器 (SSD) 技术，这需要高级存储帐户。请确保部署到的 Azure 位置支持高级存储。
* **NIC**。每个 VM 都将具有两个 NIC，一个用于数据库访问，另一个用于管理。
* **可用性集**。所有数据库服务器都将添加到单个可用性集，以确保在维护期间至少有一个 VM 已启动且正在运行。

### 步骤 1 - 启动脚本
可在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/11-MultiNIC/arm/virtual-network-deploy-multinic-arm-ps.ps1)下载所用的完整 PowerShell 脚本。按照以下步骤更改脚本，以便用于具体环境。

1. 根据在上述[先决条件](#Prerequisites)中部署的现有资源组，更改以下变量的值。

        $existingRGName        = "IaaSStory"
        $location              = "China North"
        $vnetName              = "WTestVNet"
        $backendSubnetName     = "BackEnd"
        $remoteAccessNSGName   = "NSG-RemoteAccess"
        $stdStorageAccountName = "wtestvnetstoragestd"

2. 根据要用于后端部署的值，更改以下变量的值。

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
        $diskSize               = "120"    
        $nicNamePrefix         = "NICDB"
        $ipAddressPrefix       = "192.168.2."
        $numberOfVMs           = 2

3. 检索部署所需的现有资源。

        $vnet                  = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $existingRGName
        $backendSubnet         = $vnet.Subnets|?{$_.Name -eq $backendSubnetName}
        $remoteAccessNSG       = Get-AzureRmNetworkSecurityGroup -Name $remoteAccessNSGName -ResourceGroupName $existingRGName
        $stdStorageAccount     = Get-AzureRmStorageAccount -Name $stdStorageAccountName -ResourceGroupName $existingRGName

### 步骤 2 - 为 VM 创建必要的资源
需要创建新的资源组，为数据磁盘创建存储帐户，并为所有 VM 创建可用性集。还需要每个 VM 的本地管理员帐户凭据。若要创建这些资源，请执行以下步骤。

1. 创建新的资源组。

        New-AzureRmResourceGroup -Name $backendRGName -Location $location

2. 在上面创建的资源组中创建新的高级存储帐户。

        $prmStorageAccount = New-AzureRmStorageAccount -Name $prmStorageAccountName `
        -ResourceGroupName $backendRGName -Type Premium_LRS -Location $location

3. 创建新的可用性集

        $avSet = New-AzureRmAvailabilitySet -Name $avSetName -ResourceGroupName $backendRGName -Location $location

4. 获取要用于每个 VM 的本地管理员帐户凭据。

        $cred = Get-Credential -Message "Type the name and password for the local administrator account."

### 步骤 3 - 创建 NIC 和后端 VM
需要使用循环创建所需数量的 VM，并在循环中创建所需的 NIC 和 VM。若要创建 NIC 和 VM，请执行以下步骤。

1. 启动 `for` 循环，基于 `$numberOfVMs` 变量的值重复执行命令，从而以所需次数创建一个 VM 和两个 NIC。

        for ($suffixNumber = 1; $suffixNumber -le $numberOfVMs; $suffixNumber++){

2. 创建用于数据库访问的 NIC。

        $nic1Name = $nicNamePrefix + $suffixNumber + "-DA"
        $ipAddress1 = $ipAddressPrefix + ($suffixNumber + 3)
        $nic1 = New-AzureRmNetworkInterface -Name $nic1Name -ResourceGroupName $backendRGName `
        -Location $location -SubnetId $backendSubnet.Id -PrivateIpAddress $ipAddress1

3. 创建用于远程访问的 NIC。请注意，此 NIC 有关联的 NSG。

        $nic2Name = $nicNamePrefix + $suffixNumber + "-RA"
        $ipAddress2 = $ipAddressPrefix + (53 + $suffixNumber)
        $nic2 = New-AzureRmNetworkInterface -Name $nic2Name -ResourceGroupName $backendRGName `
        -Location $location -SubnetId $backendSubnet.Id -PrivateIpAddress $ipAddress2 `
        -NetworkSecurityGroupId $remoteAccessNSG.Id

4. 创建 `vmConfig` 对象。

        $vmName = $vmNamePrefix + $suffixNumber
        $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avSet.Id

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

        $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
        $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName $publisher -Offer $offer -Skus $sku -Version $version

7. 将上面创建的两个 NIC 添加到 `vmConfig` 对象。

        $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic1.Id -Primary
        $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic2.Id

8. 创建 OS 磁盘，并创建 VM。请注意以 `}` 结束 `for` 循环。

        $osDiskName = $vmName + "-" + $osDiskSuffix
        $osVhdUri = $stdStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $osDiskName + ".vhd"
        $vmConfig = Set-AzureRmVMOSDisk -VM $vmConfig -Name $osDiskName -VhdUri $osVhdUri -CreateOption fromImage
        New-AzureRmVM -VM $vmConfig -ResourceGroupName $backendRGName -Location $location
        }

### 步骤 4 - 运行脚本
既已根据需要下载并更改了脚本，可运行该脚本以创建具有多个 NIC 的后端数据库 VM。

1. 保存脚本并从 **PowerShell** 命令提示符或 **PowerShell ISE** 运行它。最初的输出会如下所示：

		ResourceGroupName : IaaSStory-Backend
		Location          : chinanorth
		ProvisioningState : Succeeded
		Tags              :
		Permissions       :
			Actions  NotActions
			=======  ==========
				*                  

		ResourceId        : /subscriptions/[Subscription ID]/resourceGroups/IaaSStory-Backend

2. 几分钟后，填写提示的凭据，然后单击“确定”。下面的输出表示单个 VM。请注意，整个过程需要 8 分钟才能完成。

		ResourceGroupName            :
		Id                           :
		Name                         : DB2
		Type                         :
		Location                     :
		Tags                         :
		TagsText                     : null
		AvailabilitySetReference     : Microsoft.Azure.Management.Compute.Models.AvailabilitySetReference
		AvailabilitySetReferenceText : 	{
 									"ReferenceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory-Backend/providers/Microsoft.Compute/availabilitySets/ASDB"
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
        EndTime             : [Date] [Time]
        Error               :
        Output              :
        StartTime           : [Date] [Time]
        Status              : Succeeded
        TrackingOperationId : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
        RequestId           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
        StatusCode          : OK

<!---HONumber=Mooncake_1219_2016-->