<!-- ARM: tested -->

<properties
   pageTitle="适用于 Windows Server 的 Azure Hybrid Use Benefit | Azure"
   description="了解如何充分利用 Windows Server 软件保障权益，以将本地许可证带入 Azure"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/03/2016"
	wacn.date="06/20/2016"/>

# 适用于 Windows Server 的 Azure Hybrid Use Benefit

对于配合软件保证使用 Windows Server 的客户，可将本地 Windows Server 许可证带到 Azure，并以较低的成本在 Azure 中运行 Windows Server VM。Azure Hybrid Use Benefit 可让你在 Azure 中运行 Windows Server VM，且只需支付基本计算费用。

> [AZURE.NOTE] 如果要利用 Azure Hybrid Use Benefit，就不能使用 Azure 库映像来部署 Windows Server VM。必须使用 PowerShell 或 Resource Manager 模板部署 VM，以将 VM 正确注册为符合基本计算费率折扣资格。

## 先决条件
若要使 Azure 中的 Windows Server VM 利用 Azure Hybrid Use Benefit，必须符合以下几项先决条件：

- 已安装 Azure PowerShell 模块
- 已将 Windows Server VHD 上载到 Azure 存储空间

### 安装 Azure PowerShell

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

有关如何安装最新版 Azure PowerShell 的信息，请参阅 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/)。选择要使用的订阅，然后登录到你的 Azure 帐户。即使你要使用 Resource Manager 模板部署 VM，仍需要安装 Azure PowerShell 才能上载 Windows Server VHD（请参阅下一步骤）。

### 上载 Windows Server VHD

若要在 Azure 中部署 Windows Server VM，必须先创建包含基本 Windows Server 版本的 VHD。必须先通过 Sysprep 妥善准备此 VHD，再将其上载到 Azure。请[详细了解 VHD 要求和 Sysprep 进程](/documentation/articles/virtual-machines-windows-upload-image/)。准备好 VHD 之后，即可使用 `Add-AzureRmVhd` cmdlet 将 VHD 上载到 Azure 存储帐户，如下所示：

```
Add-AzureRmVhd -ResourceGroupName MyResourceGroup -Destination "https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myvhd.vhd" -LocalFilePath 'C:\Path\To\myvhd.vhd'
```

你还可以进一步了解[将 VHD 上载到 Azure 的过程](/documentation/articles/virtual-machines-windows-upload-image/#upload-the-vm-image-to-your-storage-account)。

> [AZURE.TIP] 本文重点说明 Windows Server VM 的部署，不过你也可以使用相同的方式来部署 Windows 客户端 VM。在以下示例中，请将 `Server` 相应替换为 `Client`。

## 通过 PowerShell 快速入门部署 VM
通过 PowerShell 部署 Windows Server VM 时，可以使用 `-LicenseType` 的附加参数。将 VHD 上载到 Azure 之后，可以使用 `New-AzureRmVM` 创建新 VM 并指定许可类型，如下所示：

	New-AzureRmVM -ResourceGroupName MyResourceGroup -Location "China North" -VM $vm
	    -LicenseType Windows_Server

你可以在下面进一步[阅读有关如何通过 PowerShell 在 Azure 中部署 VM 的详细演练](/documentation/articles/virtual-machines-windows-hybrid-use-benefit-licensing/#deploy-windows-server-vm-via-powershell-detailed-walkthrough)，或阅读有关[使用 Resource Manager 和 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/) 的不同步骤的描述说明。

## 通过 Resource Manager 部署 VM
在 Resource Manager 模板中，可以指定 `licenseType` 的附加参数。请了解有关[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)的详细信息。将 VHD 上载到 Azure 之后，请编辑 Resource Manager 模板以将许可证类型包含为计算提供程序的一部分，然后照常部署模板：

	"properties": {  
	   "licenseType": "Windows_Server",
	   "hardwareProfile": {
	        "vmSize": "[variables('vmSize')]"
	   },
 
## 验证 VM 是否正在利用许可权益
通过 PowerShell 或 Resource Manager 部署方法部署 VM 之后，请使用 `Get-AzureRmVM` 验证许可证类型，如下所示：

	Get-AzureRmVM -ResourceGroup MyResourceGroup -Name MyVM

你将看到与下面类似的输出：

	Type                     : Microsoft.Compute/virtualMachines
	Location                 : chinanorth
	LicenseType              : Windows_Server

以下 VM 在部署时未提供 Azure Hybrid Use Benefit 许可（例如直接从 Azure 库部署 VM），可看出其中的差异：

	Type                     : Microsoft.Compute/virtualMachines
	Location                 : chinanorth
	LicenseType              : 

##<a name="deploy-windows-server-vm-via-powershell-detailed-walkthrough"></a> PowerShell 详细演练

以下面详细 PowerShell 步骤演示了 VM 的完整部署。可以在 [Create a Windows VM using Resource Manager and PowerShell（使用 Resource Manager 和 PowerShell 创建 Windows VM）](/documentation/articles/virtual-machines-windows-ps-create/)中了解更多上下文，包括实际的 cmdlet 和所创建的不同组件。你将逐步执行创建资源组、存储帐户和虚拟网络，然后定义 VM 并完成创建 VM 的整个过程。
 
首先请安全获取凭据，并设置位置和资源组名称：

	$cred = Get-Credential
	$location = "China North"
	$resourceGroupName = "TestLicensing"

创建一个公共 IP：

	$publicIPName = "testlicensingpublicip"
	$publicIP = New-AzureRmPublicIpAddress -Name $publicIPName -ResourceGroupName $resourceGroupName -Location $location -AllocationMethod Dynamic

定义子网、NIC 和 VNET：

	$subnetName = "testlicensingsubnet"
	$nicName = "testlicensingnic"
	$vnetName = "testlicensingvnet"
	$subnetconfig = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/8
	$vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix 10.0.0.0/8 -Subnet $subnetconfig
	$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $resourceGroupName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $publicIP.Id

为 VM 命名并创建 VM 配置：

	$vmName = "testlicensing"
	$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A1"

定义 OS：

	$computerName = "testlicensing"
	$vm = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate

将 NIC 添加到 VM：

	$vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

定义要使用的存储帐户：

	$storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -AccountName testlicensing


上载适当准备的 VHD，并附加到 VM 以供使用：

	$osDiskName = "licensing.vhd"
	$osDiskUri = '{0}vhds/{1}{2}.vhd' -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName
	$urlOfUploadedImageVhd = "https://testlicensing.blob.core.chinacloudapi.cn/vhd/licensing.vhd"
	$vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri $urlOfUploadedImageVhd -Windows

最后，创建 VM 并定义许可类型，以利用 Azure Hybrid Use Benefit：

	New-AzureRmVM -ResourceGroupName $resourceGroupName -Location $location -VM $vm -LicenseType Windows_Server

## 后续步骤

了解有关[使用 Azure Resource Manager 模板](/documentation/articles/resource-group-overview/)的详细信息。
<!---HONumber=Mooncake_0613_2016-->