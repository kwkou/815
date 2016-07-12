<!-- ARM: tested -->

<properties 
   pageTitle="在 Resource Manager 中使用 PowerShell 通过静态公共 IP 部署 VM | Azure"
   description="了解如何在 Resource Manager 中使用 PowerShell 通过静态公共 IP 部署 VM"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="03/15/2016"
	wacn.date="06/06/2016"/>

# 使用 PowerShell 通过静态公共 IP 部署 VM

[AZURE.INCLUDE [virtual-network-deploy-static-pip-arm-selectors-include.md](../includes/virtual-network-deploy-static-pip-arm-selectors-include.md)]

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

[AZURE.INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../includes/virtual-network-deploy-static-pip-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../includes/azure-ps-prerequisites-include.md)]

## 步骤 1 - 启动脚本

你可以在[此处](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/virtual-network-deploy-static-pip-arm-ps.ps1)下载所用的完整 PowerShell 脚本。请按照以下步骤更改要在你的环境中使用的脚本。

1. 根据需要用于部署的值更改以下变量的值。以下值映射到本文档中使用的方案。

		# Set variables resource group
		$rgName                = "IaaSStory"
		$location              = "China North"
		
		# Set variables for VNet
		$vnetName              = "WTestVNet"
		$vnetPrefix            = "192.168.0.0/16"
		$subnetName            = "FrontEnd"
		$subnetPrefix          = "192.168.1.0/24"
		
		# Set variables for storage
		$stdStorageAccountName = "iaasstorystorage"
		
		# Set variables for VM
		$vmSize                = "Standard_A1"
		$diskSize              = 127
		$publisher             = "MicrosoftWindowsServer"
		$offer                 = "WindowsServer"
		$sku                   = "2012-R2-Datacenter"
		$version               = "latest"
		$vmName                = "WEB1"
		$osDiskName            = "osdisk"
		$nicName               = "NICWEB1"
		$privateIPAddress      = "192.168.1.101"
		$pipName               = "PIPWEB1"
		$dnsName               = "iaasstoryws1"

## 步骤 2 - 为你的 VM 创建必要的资源

在创建 VM 之前，你需要可供 VM 使用的资源组、VNet、公共 IP 和 NIC。

1. 创建新的资源组。

		New-AzureRmResourceGroup -Name $rgName -Location $location
		
2. 创建 VNet 和子网。

		$vnet = New-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName `
		    -AddressPrefix $vnetPrefix -Location $location   
		
		Add-AzureRmVirtualNetworkSubnetConfig -Name $subnetName `
		    -VirtualNetwork $vnet -AddressPrefix $subnetPrefix
		
		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet 

3. 创建公共 IP 资源。

		$pip = New-AzureRmPublicIpAddress -Name $pipName -ResourceGroupName $rgName `
		    -AllocationMethod Static -DomainNameLabel $dnsName -Location $location

4. 使用公共 IP 为上面创建的子网中的 VM 创建网络接口 (NIC)。请注意第一个用于从 Azure 检索 VNet 的 cmdlet。该 cmdlet 是必需的，因为是通过执行 **Set-AzureRmVirtualNetwork** 来更改现有 VNet 的。

		$vnet = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $subnetName
		$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
		    -Subnet $subnet -Location $location -PrivateIpAddress $privateIPAddress `
		    -PublicIpAddress $pip

5. 创建存储帐户，以托管 VM OS 驱动器。

		$stdStorageAccount = New-AzureRmStorageAccount -Name $stdStorageAccountName `
		    -ResourceGroupName $rgName -Type Standard_LRS -Location $location

## 步骤 3 - 创建 VM 

现在，所有必需的资源均已就绪，你可以创建新的 VM 了。

1. 创建 VM 配置对象。

		$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize 

1. 获取 VM 本地管理员帐户的凭据。

		$cred = Get-Credential -Message "Type the name and password for the local administrator account."

2. 创建 VM 配置对象。

		$vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName `
    		-Credential $cred -ProvisionVMAgent -EnableAutoUpdate

3. 设置 VM 的操作系统映像。

		$vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName $publisher `
    		-Offer $offer -Skus $sku -Version $version

4. 配置 OS 磁盘。

		$osVhdUri = $stdStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $osDiskName + ".vhd"
		$vmConfig = Set-AzureRmVMOSDisk -VM $vmConfig -Name $osDiskName -VhdUri $osVhdUri -CreateOption fromImage

5. 将 NIC 添加到 VM。

		$vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id -Primary

6. 创建 VM。

		New-AzureRmVM -VM $vmConfig -ResourceGroupName $rgName -Location $location

7. 保存脚本文件。

## 步骤 4 - 运行脚本

进行必要的更改并理解上面显示的脚本以后，可运行该脚本。

1. 在 PowerShell 控制台或 PowerShell ISE 中，运行上述脚本。
2. 几分钟后，将会显示以下输出。

		ResourceGroupName : IaaSStory
		Location          : chinanorth
		ProvisioningState : Succeeded
		Tags              : 
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory
				
		AddressSpace      : Microsoft.Azure.Commands.Network.Models.PSAddressSpace
		DhcpOptions       : Microsoft.Azure.Commands.Network.Models.PSDhcpOptions
		Subnets           : {FrontEnd}
		ProvisioningState : Succeeded
		AddressSpaceText  : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptionsText   : {}
		SubnetsText       : [
		                      {
		                        "Name": "FrontEnd",
		                        "AddressPrefix": "192.168.1.0/24"
		                      }
		                    ]
		ResourceGroupName : IaaSStory
		Location          : chinanorth
		ResourceGuid      : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		Tag               : {}
		TagsTable         : 
		Name              : WTestVNet
		Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
		Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet
		
		AddressSpace      : Microsoft.Azure.Commands.Network.Models.PSAddressSpace
		DhcpOptions       : Microsoft.Azure.Commands.Network.Models.PSDhcpOptions
		Subnets           : {FrontEnd}
		ProvisioningState : Succeeded
		AddressSpaceText  : {
		                      "AddressPrefixes": [
		                        "192.168.0.0/16"
		                      ]
		                    }
		DhcpOptionsText   : {
		                      "DnsServers": []
		                    }
		SubnetsText       : [
		                      {
		                        "Name": "FrontEnd",
		                        "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
		                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet/subnets/FrontEnd",
		                        "AddressPrefix": "192.168.1.0/24",
		                        "IpConfigurations": [],
		                        "ProvisioningState": "Succeeded"
		                      }
		                    ]
		ResourceGroupName : IaaSStory
		Location          : chinanorth
		ResourceGuid      : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		Tag               : {}
		TagsTable         : 
		Name              : WTestVNet
		Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
		Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/IaaSStory/providers/Microsoft.Network/virtualNetworks/WTestVNet
				
		TrackingOperationId : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		RequestId           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		Status              : Succeeded
		StatusCode          : OK
		Output              : 
		StartTime           : 1/12/2016 12:57:56 PM -08:00
		EndTime             : 1/12/2016 12:59:13 PM -08:00
		Error               : 
		ErrorText           : 

<!---HONumber=Mooncake_0418_2016-->