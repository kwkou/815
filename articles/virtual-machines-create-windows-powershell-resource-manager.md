<properties pageTitle="使用 Azure 资源管理器和 PowerShell 创建 Windows 虚拟机" description="使用 Azure PowerShell 的资源管理模式轻松创建新的 Windows 虚拟机。" services="virtual-machines" documentationCenter="" authors="JoeDavies-MSFT" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.date="06/02/2015" wacn.date="08/29/2015"/>

# 使用 Azure 资源管理器和 PowerShell 创建 Windows 虚拟机

本主题介绍如何使用 Azure 资源管理器和 PowerShell 快速创建基于 Windows 的 Azure 虚拟机。

[AZURE.INCLUDE [resource-manager-pointer-to-service-management](../includes/resource-manager-pointer-to-service-management)]

- [使用 PowerShell 和 Azure 服务管理创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-service-manager)

## 创建 Windows 虚拟机

如果你已安装 Azure PowerShell，必须确保安装的是 Azure PowerShell 版本 0.9.0 或更高版本。可以使用此命令在 Azure PowerShell 命令提示符下查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

如果你尚未这样做，请按[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell) 中的说明在本地计算机上安装 Azure PowerShell。然后，打开 Azure PowerShell 命令提示符。

首先，必须使用以下命令登录到 Azure。

	Add-AzureAccount -Environment AzureChinaCloud

在 Windows Azure 登录对话框中指定 Azure 帐户的电子邮件地址及其密码。

接下来，如果你有多个 Azure 订阅，则需要设置你的 Azure 订阅。若要查看当前订阅的列表，请运行以下命令。

	Get-AzureSubscription | sort SubscriptionName | Select SubscriptionName

现在，请将引号内的所有内容（包括 < and > 字符）替换为正确的订阅名称，然后运行以下命令。

	$subscrName="<subscription name>"
	Select-AzureSubscription -SubscriptionName $subscrName –Current

接下来，需要创建存储帐户。必须选择只包含小写字母和数字的唯一名称。你可以使用此命令测试存储帐户名称的唯一性。

	Test-AzureName -Storage <Proposed storage account name>

如果此命令返回“False”，则你建议的名称是唯一的。

需要指定 Azure 数据中心的位置。若要获取 Azure 数据中心的列表，请运行以下命令。

	Get-AzureLocation | sort Name | Select Name

接下来，需要将 Azure PowerShell 的模式切换为资源管理器。运行以下命令。

	Switch-AzureMode AzureResourceManager

现在，请将以下 PowerShell 命令块复制到文本编辑器。填写所选的存储帐户和位置，替换引号内的所有内容（包括 < and > 字符）。

	$stName="<chosen storage account name>"	
	$locName="<chosen Azure location name>"
	$rgName="TestRG"
	New-AzureResourceGroup -Name $rgName -Location $locName
	$storageAcc=New-AzureStorageAccount -ResourceGroupName $rgName -Name $stName -Type "Standard_GRS" -Location $locName
	$singleSubnet=New-AzureVirtualNetworkSubnetConfig -Name singleSubnet -AddressPrefix 10.0.0.0/24
	$vnet=New-AzurevirtualNetwork -Name TestNet -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
	$pip = New-AzurePublicIpAddress -Name TestNIC -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureNetworkInterface -Name TestNIC -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
	$cred = Get-Credential -Message "Type the name and password of the local administrator account."
	$vm = New-AzureVMConfig -VMName WindowsVM -VMSize "Standard_A1"
	$vm = Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName MyWindowsVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm = Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm = Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	$osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/WindowsVMosDisk.vhd"
	$vm = Set-AzureVMOSDisk -VM $vm -Name "windowsvmosdisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm 

最后，请将上述命令集复制到剪贴板，然后右键单击打开的 Azure PowerShell 命令提示符。这将发出包含一系列 PowerShell 命令的命令集，提示你输入本地管理员帐户的名称和密码并创建 Azure 虚拟机。

下面是你可能会看到的示例：

	PS C:\> $stName="contosost"
	PS C:\> $locName="China North"
	PS C:\> $rgName="TestRG"
	PS C:\> New-AzureResourceGroup -Name $rgName -Location $locName
	VERBOSE: 12:45:15 PM - Created resource group 'TestRG' in location 'westus'
	
	
	ResourceGroupName : TestRG
	Location          : westus
	ProvisioningState : Succeeded
	Tags              :
	Permissions       :
	                    Actions  NotActions
	                    =======  ==========
	                    *
	
	ResourceId        : /subscriptions/fd92919d-eeca-4f5b-840a-e45c6770d92e/resourceGroups/TestRG
	
	
	PS C:\> $storageAcc=New-AzureStorageAccount -ResourceGroupName $rgName -Name $stName -Type "Standard_GRS" -Location $locName
	PS C:\> $singleSubnet=New-AzureVirtualNetworkSubnetConfig -Name singleSubnet -AddressPrefix 10.0.0.0/24
	PS C:\> $vnet=New-AzurevirtualNetwork -Name TestNet3 -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
	PS C:\> $pip = New-AzurePublicIpAddress -Name TestNIC -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	PS C:\> $nic = New-AzureNetworkInterface -Name TestNIC -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
	PS C:\> $cred = Get-Credential -Message "Type the name and password of the local administrator account."
	PS C:\> $vm = New-AzureVMConfig -VMName WindowsVM -VMSize "Standard_A1"
	PS C:\> $vm = Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName MyWindowsVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	PS C:\> $vm = Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	PS C:\> $vm = Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	PS C:\> $osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/MyWindowsVMosDisk.vhd"
	PS C:\> $vm = Set-AzureVMOSDisk -VM $vm -Name "windowsvmosdisk" -VhdUri $osDiskUri -CreateOption fromImage
	PS C:\> New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	
	EndTime             : 4/28/2015 1:00:05 PM -07:00
	Error               :
	Output              :
	StartTime           : 4/28/2015 12:52:52 PM -07:00
	Status              : Succeeded
	TrackingOperationId : 45035a90-ea12-4e1e-87e7-2a5e9ed12c93
	RequestId           : 98c7b4fb-b26e-4a58-b17a-b0983d896aae
	StatusCode          : OK

## 其他资源

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

[Azure 资源管理器概述](/documentation/articles/resource-group-overview)

[使用资源管理器模板和 PowerShell 创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-resource-manager-template-simple)

[使用 PowerShell 和 Azure 服务管理创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-service-manager)

[虚拟机文档](http://www.windowsazure.cn/documentation/services/virtual-machines/)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

<!---HONumber=67-->