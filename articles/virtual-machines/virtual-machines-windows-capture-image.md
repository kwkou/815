<properties
	pageTitle="基于 Azure VM 创建 VM 映像 | Azure"
	description="了解如何基于 Resource Manager 部署模型中创建的现有 Azure VM 创建通用化 VM 映像"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="08/04/2016"
	wacn.date="09/12/2016"/>

# 如何基于现有 Azure VM 创建 VM 映像


本文说明如何使用 Azure PowerShell 创建现有 Azure VM 的通用化映像。然后可以使用该映像来创建另一个 VM。此映像包含 OS 磁盘和附加到虚拟机的数据磁盘。该映像不包含虚拟网络资源，因此，使用该映像创建 VM 时需要设置这些资源。此过程将创建[通用化 Windows 映像](https://technet.microsoft.com/zh-cn/library/hh824938.aspx)。


## 先决条件

- 这些步骤假设在 Resource Manager 部署模型中已有要用于创建映像的 Azure 虚拟机。需要 VM 名称和资源组名称。可以键入 PowerShell cmdlet `Get-AzureRmResourceGroup` 获取订阅中资源组的列表。可以键入 `Get-AzureRMVM` 获取订阅中 VM 的列表。

- 需要安装 Azure PowerShell 1.0.x 版。如果尚未安装 PowerShell，请参阅 [How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）了解安装步骤。

- 确保 Sysprep 支持计算机上运行的服务器角色。有关详细信息，请参阅 [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)（Sysprep 对服务器角色的支持）

## <a name="prepare-the-vm-for-image-capture"></a>准备源 VM 

本部分说明如何通用化 Windows 虚拟机，使其可以用作映像。

> [AZURE.WARNING] 通用化 VM 后，无法通过 RDP 登录到该 VM，因为通用化过程会删除所有用户帐户。这些更改是不可逆的。

1. 登录到 Windows 虚拟机。在 [Azure 门户预览](https://portal.azure.cn)中，导航到“浏览”>“虚拟机”> Windows 虚拟机 >“连接”。

2. 以管理员身份打开“命令提示符”窗口。

3. 将目录更改为 `%windir%\system32\sysprep`，然后运行 sysprep.exe。

4. 在“系统准备工具”对话框中，执行以下操作：

	- 在**“系统清理操作”**中，选择**“进入系统全新体验(OOBE)”**，并确保选中**“一般化”**。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

	- 在**“关机选项”**中选择**“关机”**。

	- 单击**“确定”**。

	![运行 Sysprep](./media/virtual-machines-windows-capture-image/SysprepGeneral.png)

   Sysprep 关闭虚拟机。它在 Azure 门户预览中的状态将变为“已停止”。


## 登录到 Azure PowerShell

1. 打开 Azure PowerShell 并登录到 Azure 帐户。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	此时将打开一个弹出窗口让输入 Azure 帐户凭据。

2. 获取可用订阅的订阅 ID。

		Get-AzureRmSubscription

3. 使用订阅 ID 设置正确的订阅。

		Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"


## 解除分配 VM 并将状态设置为通用化		

1. 解除分配 VM 资源。

		Stop-AzureRmVM -ResourceGroupName <resourceGroup> -Name <vmName>

	Azure 门户预览中该 VM 的“状态”将从“已停止”更改为“已停止(已解除分配)”。

2. 将虚拟机的状态设置为“通用化”。

		Set-AzureRmVm -ResourceGroupName <resourceGroup> -Name <vmName> -Generalized

3. 检查 VM 的状态。VM 的“OSState/通用化”部分中的“DisplayStatus”应设置为“VM 已通用化”。
		
		$vm = Get-AzureRmVM -ResourceGroupName <resourceGroup> -Name <vmName> -status
		$vm.Statuses

		
## <a name="capture-the-vm"></a>创建映像 

1. 使用以下命令将虚拟机映像复制到目标存储容器。该映像在创建时所在的存储帐户与原始虚拟机的相同。`-Path` 变量在本地保存 JSON 模板的副本。`-DestinationContainerName` 变量是要在其中保存映像的容器的名称。如果该容器不存在，系统将自动创建。

		Save-AzureRmVMImage -ResourceGroupName YourResourceGroup -VMName YourWindowsVM -DestinationContainerName YourImagesContainer -VHDNamePrefix YourTemplatePrefix -Path Yourlocalfilepath\Filename.json

	可以从 JSON 文件模板获取映像的 URL。转到“资源”>“storageProfile”>“osDisk”>“映像”>“URI”部分即可查找映像的完整路径。映像的 URL 如下所示：`https://<storageAccountName>.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/<imagesContainer>/<templatePrefix-osDisk>.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`。
	
	也可以在门户中验证 URI。映像将复制到存储帐户中名为 **system** 的 Blob。

2. 创建映像路径的变量。

		$imageURI = "<https://<storageAccountName>.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/<imagesContainer>/<templatePrefix-osDisk>.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd>"


## 创建虚拟网络

创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。
		

1. 将变量值替换为自己的信息。以 CIDR 格式提供子网的地址前缀。创建变量和子网。

    	$rgName = "<resourceGroup>"
		$location = "<location>"
        $subnetName = "<subNetName>"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix <0.0.0.0/0>
        
2. 将 **$vnetName** 的值替换为虚拟网络的名称。以 CIDR 格式提供虚拟网络的地址前缀。创建变量和具有子网的虚拟网络。

        $vnetName = "<vnetName>"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $locName -AddressPrefix <0.0.0.0/0> -Subnet $singleSubnet
        
            
## 创建公共 IP 地址和网络接口

若要与虚拟网络中的虚拟机通信，需要一个[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 将 **$ipName** 的值替换为公共 IP 地址的名称。创建变量和公共 IP 地址。

        $ipName = "<ipName>"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
        
2. 将 **$nicName** 的值替换为网络接口的名称。创建变量和网络接口。

        $nicName = "<nicName>"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id


## 创建 VM

以下 PowerShell 脚本演示如何设置虚拟机配置和使用已上载的 VM 映像作为新安装的源。

>[AZURE.NOTE] VM 需要与原始 VHD 位于同一个存储帐户中。

</br>

	
	
	
	#Create variables
	# Enter a new user name and password to use as the local administrator account for the remotely accessing the VM
	$cred = Get-Credential
	
	# Name of the storage account 
	$storageAccName = "<storageAccountName>"
	
	# Name of the virtual machine
	$vmName = "<vmName>"
	
	# Size of the virtual machine. See the VM sizes documentation for more information: /documentation/articles/virtual-machines-windows-sizes/
	$vmSize = "<vmSize>"
	
	# Computer name for the VM
	$computerName = "<computerName>"
	
	# Name of the disk that holds the OS
	$osDiskName = "<osDiskName>"
	
	# Assign a SKU name
	# Valid values for -SkuName are: **Standard_LRS** - locally redundant storage, **Standard_ZRS** - zone redundant storage, **Standard_GRS** - geo redundant storage, **Standard_RAGRS** - read access geo redundant storage, **Premium_LRS** - premium locally redundant storage. 
	$skuName = "<skuName>"
	
	# Create a new storage account for the VM
	New-AzureRmStorageAccount -ResourceGroupName $rgName -Name $storageAccName -Location $location -SkuName $skuName -Kind "Storage"

	#Get the storage account where the uploaded image is stored
	$storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $rgName -AccountName $storageAccName

	#Set the VM name and size
	#Use "Get-Help New-AzureRmVMConfig" to know the available options for -VMsize
	$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

	#Set the Windows operating system configuration and add the NIC
	$vm = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate

	$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

	#Create the OS disk URI
	$osDiskUri = '{0}vhds/{1}-{2}.vhd' -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName

	#Configure the OS disk to be created from the image (-CreateOption fromImage), and give the URL of the uploaded image VHD for the -SourceImageUri parameter
	#You set this variable when you uploaded the VHD
	$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri $imageURI -Windows

	#Create the new VM
	New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm



完成后，应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

	$vmList = Get-AzureRmVM -ResourceGroupName $rgName
	$vmList.Name


## 后续步骤

若要使用 Azure PowerShell 管理新虚拟机，请参阅 [Manage virtual machines using Azure Resource Manager and PowerShell](/documentation/articles/virtual-machines-windows-ps-manage/)（使用 Azure Resource Manager 与 PowerShell 来管理虚拟机）。

<!---HONumber=Mooncake_0905_2016-->