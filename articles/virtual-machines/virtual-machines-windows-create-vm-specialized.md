<properties
	pageTitle="创建 Windows VM 的副本 | Azure"
	description="了解如何在 Resource Manager 部署模型中，为运行 Windows 的专用 Azure VM 创建副本。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/21/2016"
	wacn.date="01/05/2017"
	ms.author="cynthn"/>  


# 从专用 VHD 创建 VM

通过使用 Powershell 将专用 VHD 附加为 OS 磁盘来创建新 VM。专用 VHD 保留原始 VM 中的用户帐户、应用程序和其他状态数据。

如果想要从通用 VHD 创建 VM，请参阅[从通用 VHD 映像创建 VM](/documentation/articles/virtual-machines-windows-create-vm-generalized/)。

## 创建子网和 vNet

创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。

1. 创建子网。此示例在资源组 **myResourceGroup** 中创建名为 **mySubNet** 的子网，并将子网地址前缀设置为 **10.0.0.0/24**。

		$rgName = "myResourceGroup"
		$subnetName = "mySubNet"
		$singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

2. 创建 vNet。此示例将虚拟网络名称设置为 **myVnetName**，将位置设置为**中国北部**，将虚拟网络的地址前缀设置为 **10.0.0.0/16**。

		$location = "China North"
		$vnetName = "myVnetName"
		$vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
		    -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

## 创建公共 IP 地址和 NIC

若要与虚拟网络中的虚拟机通信，需要一个[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 创建公共 IP。在此示例中，公共 IP 地址名称设置为 **myIP**。

		$ipName = "myIP"
		$pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
		    -AllocationMethod Dynamic

2. 创建 NIC。在此示例中，NIC 名称设置为 **myNicName**。

		$nicName = "myNicName"
		$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location `
		    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

## 创建网络安全组和 RDP 规则

若要使用 RDP 登录到 VM，需要创建一个允许在端口 3389 上进行 RDP 访问的安全规则。由于新 VM 的 VHD 是从现有专用 VM 创建的，因此在创建 VM 后，可以使用源虚拟机中有权通过 RDP 登录的现有帐户。

此示例将 NSG 名称设置为 **myNsg**，将 RDP 规则名称设置为 **myRdpRule**。

	$nsgName = "myNsg"

	$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
		-Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
		-SourceAddressPrefix Internet -SourcePortRange * `
		-DestinationAddressPrefix * -DestinationPortRange 3389

	$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
		-Name $nsgName -SecurityRules $rdpRule

有关终结点和 NSG 规则的详细信息，请参阅 [Opening ports to a VM in Azure using PowerShell](/documentation/articles/virtual-machines-windows-nsg-quickstart-powershell/)（使用 PowerShell 在 Azure 中打开 VM 端口）。

## 创建 VM 配置

设置 VM 配置，以便将复制的 VHD 附加为 OS VHD。

	# Set the URI for the VHD that you want to use. In this example, the VHD file named "myOsDisk.vhd" is kept
	# in a storage account named "myStorageAccount" in a container named "myContainer".
	$osDiskUri = "https://myStorageAccount.blob.core.chinacloudapi.cn/myContainer/myOsDisk.vhd"
	
	# Set the VM name and size. This example sets the VM name to "myVM" and the VM size to "Standard_A2".
	$vmName = "myVM"
	$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"

	# Add the NIC
	$vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id

	# Add the OS disk by using the URL of the copied OS VHD. In this example, when the OS disk is created, the
	# term "osDisk" is appened to the VM name to create the OS disk name. This example also specifies that this
	# Windows-based VHD should be attached to the VM as the OS disk.
	$osDiskName = $vmName + "osDisk"
	$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption attach -Windows

如果需要将数据磁盘附加到 VM，还应该添加以下代码：

	# Optional: Add data disks by using the URLs of the copied data VHDs at the appropriate Logical Unit
	# Number (Lun).
	$dataDiskName = $vmName + "dataDisk"
	$vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -VhdUri $dataDiskUri -Lun 0 -CreateOption attach

数据磁盘和操作系统磁盘的 URL 类似于：`https://StorageAccountName.blob.core.chinacloudapi.cn/BlobContainerName/DiskName.vhd`。可通过以下方法在门户上找到此信息：浏览到目标存储容器，单击复制的操作系统或数据 VHD，然后复制 URL 的内容。


## 创建 VM

使用刚刚创建的配置创建 VM。

	#Create the new VM
	New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm

如果此命令成功，你将看到类似于下面的输出：

	RequestId IsSuccessStatusCode StatusCode ReasonPhrase
	--------- ------------------- ---------- ------------
							True         OK OK   

## 验证是否已创建 VM 
 
应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

	$vmList = Get-AzureRmVM -ResourceGroupName $rgName
	$vmList.Name

## 后续步骤

若要登录到新虚拟机，请在[门户](https://portal.azure.cn)中浏览到该 VM，单击“连接”，然后打开远程桌面 RDP 文件。使用原始虚拟机的帐户凭据登录到新虚拟机。有关详细信息，请参阅 [How to connect and log on to an Azure virtual machine running Windows](/documentation/articles/virtual-machines-windows-connect-logon/)（如何连接并登录到运行 Windows 的 Azure 虚拟机）。

<!---HONumber=Mooncake_1114_2016-->