<properties
	pageTitle="上载 Windows VHD 用于资源管理器 | Azure"
	description="了解如何上载 Windows 虚拟机映像以用于资源管理器部署模型。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="08/02/2016"
	wacn.date="09/12/2016"/>

# 将 Windows VM 映像上载到 Azure 以进行资源管理器部署


本文说明如何创建和上载 Windows 虚拟硬盘 (VHD) 映像，以便快速创建 VM。有关 Azure 中的磁盘和 VHD 的更多详细信息，请参阅 [About disks and VHDs for virtual machines](/documentation/articles/virtual-machines-linux-about-disks-vhds/)（关于虚拟机的磁盘和 VHD）。


## 先决条件

本文假设你具备以下条件：

- **Azure 订阅** - 如果没有 Azure 订阅，请[注册 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

- **Azure PowerShell 1.4 或更高版本** - 如果还没有安装它，请阅读 [How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）。

- **运行 Windows 的虚拟机** - 有许多工具可在本地创建虚拟机。有关示例，请参阅 [Install the Hyper-V Role and configure a virtual machine](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)（安装 Hyper-V 角色和配置虚拟机）。有关 Azure 支持哪些 Windows 操作系统的信息，请参阅 [Microsoft server software support for Azure virtual machines](https://support.microsoft.com/kb/2721672)（Azure 虚拟机支持的 Microsoft 服务器软件）。

- 确保 VM 上运行的服务器角色支持 sysprep。有关详细信息，请参阅 [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)（Sysprep 对服务器角色的支持）。


## 确保 VM 使用正确的文件格式

在 Azure 中，只能使用采用 VHD 文件格式的[第 1 代虚拟机](http://blogs.technet.com/b/ausoemteam/archive/2015/04/21/deciding-when-to-use-generation-1-or-generation-2-virtual-machines-with-hyper-v.aspx)。VHD 必须是固定大小，且为整数 MB，也就是可被 8 整除。VHD 允许的最大大小为 1,023 GB。

- 如果 Windows VM 映像为 VHDX 格式，请使用以下方法之一将它转换成 VHD：

	- Hyper-V：打开 Hyper-V，在左侧选择本地计算机。然后在它上方的菜单中，单击“操作”>“编辑磁盘...”。通过单击“下一步”在屏幕之间导航并输入以下选项：VHDX 文件的路径 >“转换”>“VHD”>“固定大小”> 新的 VHD 文件的路径。单击“完成”以关闭。

	- [Convert-VHD PowerShell cmdlet](http://technet.microsoft.com/zh-cn/library/hh848454.aspx)：有关详细信息，请阅读博客文章 [Converting Hyper-V .vhdx to .vhd file formats](https://blogs.technet.microsoft.com/cbernier/2013/08/29/converting-hyper-v-vhdx-to-vhd-file-formats-for-use-in-windows-azure/)（将 Hyper-V .vhdx 转换为 .vhd 文件格式）。

- 如果你有 [VMDK 文件格式](https://en.wikipedia.org/wiki/VMDK)的 Windows VM 映像，可以使用 [Microsoft 虚拟机转换器](https://www.microsoft.com/download/details.aspx?id=42497)将其转换为 VHD。有关详细信息，请阅读博客 [How to Convert a VMware VMDK to Hyper-V VHD](http://blogs.msdn.com/b/timomta/archive/2015/06/11/how-to-convert-a-vmware-vmdk-to-hyper-v-vhd.aspx)（如何将 VMware VMDK 转换为 Hyper-V VHD）。


## 准备要上载的 VHD

本部分说明如何使 Windows 虚拟机通用化。Sysprep 将删除所有个人帐户信息及其他某些数据。有关 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

1. 登录到 Windows 虚拟机。

2. 以管理员身份打开“命令提示符”窗口。将目录更改为 **%windir%\\system32\\sysprep**，然后运行 `sysprep.exe`。

3. 在“系统准备工具”对话框中，执行以下操作：

	1. 在“系统清理操作”中，选择“进入系统全新体验(OOBE)”，并确保选中“通用化”复选框。

	2. 在**“关机选项”**中选择**“关机”**。

	3. 单击**“确定”**。

	![启动 Sysprep](./media/virtual-machines-windows-upload-image/sysprepgeneral.png)

</br>  



## 登录 Azure

1. 打开 Azure PowerShell 并登录到 Azure 帐户。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

	此时将打开一个弹出窗口让输入 Azure 帐户凭据。

2. 获取可用订阅的订阅 ID。

		Get-AzureRmSubscription

3. 使用订阅 ID 设置正确的订阅。

		Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"

	
## <a name="createstorage"></a> 获取存储帐户

需要在 Azure 中创建一个存储帐户用于容装上载的 VM 映像。你可以使用现有存储帐户，也可以创建新的存储帐户。

显示可用的存储帐户。

		Get-AzureRmStorageAccount

如果要使用现有存储帐户，请转到[上载 VM 映像](#upload-the-vm-image-to-your-storage-account)部分。

若要创建存储帐户，请执行以下步骤：

1. 请确保你拥有此存储帐户的资源组。可以使用以下命令列出订阅中的所有资源组：

		Get-AzureRmResourceGroup

2. 若要创建资源组，请使用以下命令：

		New-AzureRmResourceGroup -Name <resourceGroupName> -Location <location>

3. 使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet 在此资源组中创建存储帐户：

		New-AzureRmStorageAccount -ResourceGroupName <resourceGroupName> -Name <storageAccountName> -Location "<location>" -SkuName "<skuName>" -Kind "Storage"
			
-SkuName 的有效值为：

- **Standard\_LRS** - 本地冗余存储。
- **Standard\_ZRS** - 区域冗余存储。
- **Standard\_GRS** - 异地冗余存储。
- **Standard\_RAGRS** - 读取访问权限异地冗余存储。
- **Premium\_LRS** - 高级本地冗余存储。



## <a name="upload-the-vm-image-to-your-storage-account"></a>将 VM 映像上载到你的存储帐户

使用 [Add-AzureRmVhd](https://msdn.microsoft.com/zh-cn/library/mt603554.aspx) cmdlet 将映像上载到存储帐户中的容器：

		$rgName = "<resourceGroupName>"
		$urlOfUploadedImageVhd = "<storageAccount>/<blobContainer>/<targetVHDName>.vhd"
		Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd -LocalFilePath <localPathOfVHDFile>

其中：

- **storageAccount** 是映像的存储帐户名。

- **blobContainer** 是用于存储映像的 Blob 容器。如果找不到具有此名称的现有 Blob 容器，系统会自动创建此容器。

- **targetVHDName** 是上载的 VHD 文件使用的名称。

- **localPathOfVHDFile** 是 .vhd 文件在本地计算机上的完整路径和名称。


如果成功，将显示类似于下面的响应：

		C:\> Add-AzureRmVhd -ResourceGroupName testUpldRG -Destination https://testupldstore2.blob.core.chinacloudapi.cn/testblobs/WinServer12.vhd -LocalFilePath "C:\temp\WinServer12.vhd"
		MD5 hash is being calculated for the file C:\temp\WinServer12.vhd.
		MD5 hash calculation is completed.
		Elapsed time for the operation: 00:03:35
		Creating new page blob of size 53687091712...
		Elapsed time for upload: 01:12:49

		LocalFilePath           DestinationUri
		-------------           --------------
		C:\temp\WinServer12.vhd https://testupldstore2.blob.core.chinacloudapi.cn/testblobs/WinServer12.vhd

此命令可能需要一段时间才能完成，具体取决于网络连接速度和 VHD 文件的大小。




## 创建虚拟网络

创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。

1. 将变量值替换为自己的信息。以 CIDR 格式提供子网的地址前缀。创建变量和子网。

    	$rgName = "<resourceGroup>"
		$location = "<location>"
        $subnetName = "<subNetName>"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix <0.0.0.0/0>
        
2. 将 **$vnetName** 的值替换为虚拟网络的名称。以 CIDR 格式提供虚拟网络的地址前缀。创建变量和具有子网的虚拟网络。

        $vnetName = "<vnetName>"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location -AddressPrefix <0.0.0.0/0> -Subnet $singleSubnet
        
            
## 创建公共 IP 地址和网络接口

若要与虚拟网络中的虚拟机通信，需要一个[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 将 **$ipName** 的值替换为公共 IP 地址的名称。创建变量和公共 IP 地址。

        $ipName = "<ipName>"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location -AllocationMethod Dynamic
        
2. 将 **$nicName** 的值替换为网络接口的名称。创建变量和网络接口。

        $nicName = "<nicName>"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

		

## 创建 VM

以下 PowerShell 脚本演示如何设置虚拟机配置和使用已上载的 VM 映像作为新安装的源。

>[AZURE.NOTE] VM 需要与上载的 VHD 位于同一个存储帐户中。

</br>  


	
	
	#Create variables
	# Enter a new user name and password to use as the local administrator account for the remotely accessing the VM
	$cred = Get-Credential
	
	# Name of the storage account where the VHD file is and where the OS disk will be created
	$storageAccName = "<storageAccountName>"
	
	# Name of the virtual machine
	$vmName = "<vmName>"
	
	# Size of the virtual machine. See the VM sizes documentation for more information: /documentation/articles/virtual-machines-windows-sizes/
	$vmSize = "<vmSize>"
	
	# Computer name for the VM
	$computerName = "<computerName>"
	
	# Name of the disk that holds the OS
	$osDiskName = "<osDiskName>"

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
	$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri $urlOfUploadedImageVhd -Windows

	#Create the new VM
	New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm



完成后，应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

	$vmList = Get-AzureRmVM -ResourceGroupName $rgName
	$vmList.Name


## 后续步骤

若要使用 Azure PowerShell 管理新虚拟机，请阅读 [Manage virtual machines using Azure Resource Manager and PowerShell](/documentation/articles/virtual-machines-windows-ps-manage/)（使用 Azure Resource Manager 与 PowerShell 来管理虚拟机）。

<!---HONumber=Mooncake_0905_2016-->