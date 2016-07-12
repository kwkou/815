<!-- ARM: tested -->

<properties
	pageTitle="使用 PowerShell 创建 Azure VM | Azure"
	description="使用 Azure PowerShell 和 Azure Resource Manager 轻松创建运行 Windows Server 的新 VM。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/02/2016"
	wacn.date="06/27/2016"/>

# 使用 Resource Manager 和 PowerShell 创建 Windows VM

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

本文将说明如何使用 [Resource Manager](/documentation/articles/resource-group-overview/) 和 PowerShell 快速创建运行 Windows Server 的 Azure 虚拟机及其所需的资源。

若要创建虚拟机，必须执行本文中的所有步骤，这需要花费大约 30 分钟时间。

## 步骤 1：安装 Azure PowerShell

有关如何安装最新版 Azure PowerShell 的信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。选择要使用的订阅，然后登录到你的 Azure 帐户。
        
## 步骤 2：创建资源组

首先，请创建资源组。

1. 获取可以创建资源的可用位置列表。

	    Get-AzureLocation | sort Name | Select Name
        
    您应看到与下面类似的内容：
    
        Name
        ----
        China North
        China East

2. 将 **$locName** 的值替换为列表中的某个位置。创建变量。

        $locName = "China North"
        
3. 将 **$rgName** 的值替换为新资源组的名称。创建变量和资源组。

        $rgName = "mygroup1"
        New-AzureRmResourceGroup -Name $rgName -Location $locName
    
## 步骤 3：创建存储帐户

需要一个[存储帐户](/documentation/articles/storage-introduction/)来存储与要创建的虚拟机使用的虚拟硬盘。

1. 将 **$stName** 的值替换为存储帐户的名称。测试名称的唯一性。

        $stName = "mystorage1"
        Test-AzureName -Storage $stName

    如果此命令返回 **False**，则表示建议的名称在 Azure 中是唯一的。存储帐户名称必须为 3 到 24 个字符，并且只能包含数字和小写字母。
    
2. 现在，请运行以下命令来创建存储帐户。
    
        $storageAcc = New-AzureRmStorageAccount -ResourceGroupName $rgName -Name $stName -Type "Standard_LRS" -Location $locName
        
## 步骤 4：创建虚拟网络

所有虚拟机都是[虚拟网络](/documentation/articles/virtual-networks-overview/)的一部分。

1. 将 **$subnetName** 的值替换为子网的名称。创建变量和子网。
    	
        $subnetName = "mysubnet1"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
        
2. 将 **$vnetName** 的值替换为虚拟网络的名称。创建变量和具有子网的虚拟网络。

        $vnetName = "myvnet1"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
        
    应使用对应用程序和环境有意义的值。
        
## 步骤 5：创建公共 IP 地址和网络接口

若要与虚拟网络中的虚拟机通信，需要一个[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 将 **$ipName** 的值替换为公共 IP 地址的名称。创建变量和公共 IP 地址。

        $ipName = "myIPaddress1"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
        
2. 将 **$nicName** 的值替换为网络接口的名称。创建变量和网络接口。

        $nicName = "mynic1"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
        
## 步骤 6：创建虚拟机

现已做好所有的准备，接下来可以开始创建虚拟机。

1. 运行以下命令以设置虚拟机的管理员帐户名和密码。

        $cred = Get-Credential -Message "Type the name and password of the local administrator account."
        
    密码的长度必须为 8 到 123 个字符，并且必须至少包含 3 个以下字符：1 个小写字符、1 个大写字符、1 个数字和 1 个特殊字符。
        
2. 将 **$vmName** 的值替换为虚拟机的名称。创建变量和虚拟机配置。

        $vmName = "myvm1"
        $vm = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A1"
        
    有关虚拟机的可用大小列表，请参阅 [Sizes for virtual machines in Azure（Azure 中的虚拟机大小）](/documentation/articles/virtual-machines-windows-sizes/)。
    
3. 将 **$compName** 的值替换为虚拟机的计算机名称。创建变量并将操作系统信息添加到配置。

        $compName = "myvm1"
        $vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName $compName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
        
4. 定义要用于预配虚拟机的映像。

        $vm = Set-AzureRmVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
        
    有关选择要使用的映像的详细信息，请参阅 [Navigate and select Windows virtual machine images in Azure with PowerShell or the CLI（使用 PowerShell 或 CLI 在 Azure 中浏览和选择 Windows 虚拟机映像）](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)。
        
5. 将创建的网络接口添加到配置。

        $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
        
6. 将 **$blobPath** 的值替换为虚拟硬盘要使用的存储中的路径和文件名。虚拟硬盘文件通常存储在容器中，例如 **vhds/WindowsVMosDisk.vhd**。创建变量。

        $blobPath = "vhds/WindowsVMosDisk.vhd"
        $osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + $blobPath
        
7. 将 **$diskName** 的值替换为操作系统磁盘的名称。创建变量并将磁盘信息添加到配置。

        $diskName = "windowsvmosdisk"
        $vm = Set-AzureRmVMOSDisk -VM $vm -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage
        
8. 最后，创建虚拟机。

        New-AzureRmVM -ResourceGroupName $rgName -Location $locName -VM $vm

    你应会在 Azure 门户预览中看到资源组及其所有资源，并会在 PowerShell 窗口中看到成功状态：

        RequestId  IsSuccessStatusCode  StatusCode  ReasonPhrase
        ---------  -------------------  ----------  ------------
                                  True          OK  OK
                                  
## 后续步骤

- 查看 [Manage virtual machines using Azure Resource Manager and PowerShell（使用 Azure Resource Manager 和 PowerShell 管理虚拟机）](/documentation/articles/virtual-machines-windows-ps-manage/)，了解如何管理刚创建的虚拟机。
- 参考 [Create a Windows virtual machine with a Resource Manager template（使用 Resource Manager 模板创建 Windows 虚拟机）](/documentation/articles/virtual-machines-windows-ps-template/)中的信息，通过模板创建虚拟机

<!---HONumber=Mooncake_0620_2016-->