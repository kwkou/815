<properties
	pageTitle="使用资源管理器和 Azure PowerShell 创建并预配置 Windows 虚拟机"
	description="了解如何使用 Azure PowerShell 在 Azure 中创建和预配置基于 Windows 和资源管理器的虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines"
	ms.date="07/22/2015"
	wacn.date="08/29/2015"/>

# 使用资源管理器和 Azure PowerShell 创建并预配置 Windows 虚拟机

这些步骤演示了如何在 Azure PowerShell 的资源管理器模式下构建一组命令来创建和预配置基于 Windows 的 Azure 虚拟机。此构建基块过程可用于为基于 Windows 的新虚拟机快速创建一个命令集以及扩展现有的部署。此外，还可用于创建多个命令集，以便快速增建自定义开发/测试或 IT 专业人员环境。

这些步骤采用填空方法来创建 Azure PowerShell 命令集。如果你不熟悉 PowerShell 或只想知道为成功的配置指定什么值，则此方法很有用。如果你是高级 PowerShell 用户，则可以使用命令并将变量（以“$”开头的行）替换为自己的值

[AZURE.INCLUDE [resource-manager-pointer-to-service-management](../includes/resource-manager-pointer-to-service-management.md)]

- [使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)

## 步骤 1：安装 Azure PowerShell

你还必须有 Azure PowerShell 0.9.0 或更高版本。如果你尚未安装和配置 Azure PowerShell，请单击[此处](/documentation/articles/powershell-install-configure)获取相关说明。

可以使用此命令在 Azure PowerShell 提示符下查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

下面是一个示例。

	Version
	-------
	0.9.0

如果没有版本 0.9.0 或更高版本，则必须使用“程序和功能”控制面板删除 Azure PowerShell，然后安装最新的版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

## 步骤 2：设置订阅

首先，运行 Azure PowerShell 提示符。

接着，通过在 Azure PowerShell 提示符下运行以下命令来设置 Azure 订阅。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<subscription name>"
	Select-AzureSubscription -SubscriptionName $subscr –Current

你可以从以下命令的显示内容中获取正确的订阅名称。

	Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName

接着，将 Azure PowerShell 切换到资源管理器模式。

	Switch-AzureMode AzureResourceManager

## 步骤 3：创建所需的资源

基于资源管理器的虚拟机需要一个资源组。如有需要，可使用以下命令为新虚拟机创建一个新资源组。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	New-AzureResourceGroup -Name $rgName -Location $locName

若要确定唯一的资源组名称，可使用以下命令列出现有的资源组。

	Get-AzureResourceGroup | Sort ResourceGroupName | Select ResourceGroupName

若要列出可在其中创建基于资源管理器的虚拟机的 Azure 位置，可使用以下命令。

	$loc=Get-AzureLocation | where { $_.Name –eq "Microsoft.Compute/virtualMachines" }
	$loc.Locations

基于资源管理器的虚拟机需要一个基于资源管理器的存储帐户。如有需要，可使用以下命令为新虚拟机创建一个新存储帐户。

	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	$saName="<storage account name>"
	$saType="<storage account type, specify one: Standard_LRS, Standard_GRS, Standard_RAGRS, or Premium_LRS>"
	New-AzureStorageAccount -Name $saName -ResourceGroupName $rgName –Type $saType -Location $locName

必须为存储帐户选择只包含小写字母和数字的全局唯一名称。可以使用以下命令列出现有的存储帐户。

	Get-AzureStorageAccount | Sort Name | Select Name

若要测试所选存储帐户名称是否为全局唯一名称，需在 PowerShell 的 Azure 服务管理模式下运行 **Test-AzureName** 命令。使用以下命令。

	Switch-AzureMode AzureServiceManagement
	Test-AzureName -Storage <Proposed storage account name>

如果 Test-AzureName 命令显示“False”，则建议的名称是唯一的。在确定唯一名称后，使用以下命令将 Azure PowerShell 切换回资源管理器模式。

	Switch-AzureMode AzureResourceManager

基于资源管理器的虚拟机可以使用公共域名标签，它只能包含字母、数字和连字符。该字段中的第一个和最后一个字符必须是字母或数字。

若要测试所选域名标签是否为全局唯一名称，可使用以下命令。

	$domName="<domain name label to test>"
	$loc="<short name of an Azure location, for example, for West US, the short name is westus>"
	Get-AzureCheckDnsAvailability -DomainQualifiedName $domName -Location $loc

如果 DNSNameAvailability 为“True”，则建议的名称是全局唯一名称。

基于资源管理器的虚拟机可以放在一个基于资源管理器的可用性集中。如有需要，可使用以下命令为新虚拟机创建一个新可用性集。

	$avName="<availability set name>"
	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	New-AzureAvailabilitySet –Name $avName –ResourceGroupName $rgName -Location $locName

使用以下命令列出现有的可用性集。

	Get-AzureAvailabilitySet –ResourceGroupName $rgName | Sort Name | Select Name

可以为基于资源管理器的虚拟机配置入站 NAT 规则以允许 Internet 的传入流量，并且可以将这些虚拟机放在一个负载平衡集中。在这两种情况中，都必须指定负载平衡器实例和其他设置。有关详细信息，请参阅 <!--[-->使用 Azure 资源管理器创建负载平衡器<!--](/documentation/articles/load-balancer-arm-powershell)-->。

基于资源管理器的虚拟机需要基于资源管理器的虚拟网络。如有需要，可以为新虚拟机创建一个基于资源管理器且包含至少一个子网的新虚拟网络。下面是一个包含两个名为 frontendSubnet 和 backendSubnet 的子网的新虚拟网络的示例。

	$rgName="LOBServers"
	$locName="West US"
	$frontendSubnet=New-AzureVirtualNetworkSubnetConfig -Name frontendSubnet -AddressPrefix 10.0.1.0/24
	$backendSubnet=New-AzureVirtualNetworkSubnetConfig -Name backendSubnet -AddressPrefix 10.0.2.0/24
	New-AzurevirtualNetwork -Name TestNet -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $frontendSubnet,$backendSubnet

使用以下命令列出现有的虚拟网络。

	$rgName="<resource group name>"
	Get-AzureVirtualNetwork -ResourceGroupName $rgName | Sort Name | Select Name

## 步骤 4：生成命令集

打开所选文本编辑器的一个新实例或 PowerShell 集成脚本环境 (ISE)，并复制以下行以启动命令集。指定此新虚拟机的资源组、Azure 位置和存储帐户的名称。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	Switch-AzureMode AzureResourceManager
	$rgName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$saName="<storage account name>"

必须指定基于资源管理器的虚拟网络的名称以及该虚拟网络中子网的索引号。使用以下命令列出虚拟网络的子网。

	$rgName="<resource group name>"
	$vnetName="<virtual network name>"
	Get-AzureVirtualNetwork -Name $vnetName -ResourceGroupName $rgName | Select Subnets

子网索引是指此命令的显示内容中子网的编号，将它们从 0 开始从左到右依次编号。

在本示例中：

	PS C:\> Get-AzureVirtualNetwork -Name TestNet -ResourceGroupName LOBServers | Select Subnets

	Subnets
	-------
	{frontendSubnet, backendSubnet}

frontendSubnet 的子网索引为 0，backendSubnet 的子网索引为 1。

将以下行复制到命令集中，并指定现有虚拟网络的名称和虚拟机的子网索引。

	$vnetName="<name of an existing virtual network>"
	$subnetIndex=<index of the subnet on which to create the NIC for the virtual machine>
	$vnet=Get-AzurevirtualNetwork -Name $vnetName -ResourceGroupName $rgName

接着，创建网络接口卡 (NIC)。将下列选项之一复制到命令集中，并填写所需的信息。

### 选项 1：指定 NIC 名称并分配一个公共 IP 地址

将以下行复制到命令集中，并指定 NIC 的名称。

	$nicName="<name of the NIC of the VM>"
	$pip = New-AzurePublicIpAddress -Name $nicName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

### 选项 2：指定 NIC 名称和 DNS 域名标签

将以下行复制到命令集中，并指定 NIC 的名称以及全局唯一域名标签。在 Azure PowerShell 的服务管理模式下创建虚拟机时，Azure 将自动完成这些步骤。

	$nicName="<name of the NIC of the VM>"
	$domName="<domain name label>"
	$pip = New-AzurePublicIpAddress -Name $nicName -ResourceGroupName $rgName -DomainNameLabel $domName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

### 选项 3：指定 NIC 名称并分配一个静态的专用 IP 地址

将以下行复制到命令集中，并指定 NIC 的名称。

	$nicName="<name of the NIC of the VM>"
	$staticIP="<available static IP address on the subnet>"
	$pip = New-AzurePublicIpAddress -Name $nicName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id -PrivateIpAddress $staticIP

### 选项 4：指定 NIC 名称以及入站 NAT 规则的负载平衡器实例。

若要创建 NIC 并其添加到入站 NAT 规则的负载平衡器实例中，你需要：

- 一个以前创建的负载平衡器实例的名称，该实例具有将流量转发到虚拟机的入站 NAT 规则。
- 要分配给 NIC 的负载平衡器实例的后端地址池的索引号。
- 要分配给 NIC 的入站 NAT 规则的索引号。

有关如何创建具有入站 NAT 规则的负载平衡器实例的信息，请参阅[使用 Azure 资源管理器创建负载平衡器](/documentation/articles/load-balancer-arm-powershell)。

将以下行复制到命令集中，并指定所需的名称和索引号。

	$nicName="<name of the NIC of the VM>"
	$lbName="<name of the load balancer instance>"
	$bePoolIndex=<index of the back end pool, starting at 0>
	$natRuleIndex=<index of the inbound NAT rule, starting at 0>
	$lb=Get-AzureLoadBalancer -Name $lbName -ResourceGroupName $rgName 
	$nic=New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -Subnet $vnet.Subnets[$subnetIndex].Id -LoadBalancerBackendAddressPool $lb.BackendAddressPools[$bePoolIndex] -LoadBalancerInboundNatRule $lb.InboundNatRules[$natRuleIndex]

$nicName 字符串对于资源组必须是唯一的。最佳做法是将虚拟机名称包含在该字符串中，比如“LOB07-NIC”。

### 选项 5：指定 NIC 名称以及负载平衡集的负载平衡器实例。

若要创建 NIC 并将其添加到负载平衡集的负载平衡器实例中，你需要：

- 一个以前创建的负载平衡器实例的名称，该实例具有针对负载平衡流量的规则。
- 要分配给 NIC 的负载平衡器实例的后端地址池的索引号。

有关如何创建具有针对负载平衡流量的规则的负载平衡器实例的信息，请参阅[使用 Azure 资源管理器创建负载平衡器](/documentation/articles/load-balancer-arm-powershell)。

将以下行复制到命令集中，并指定所需的名称和索引号。

	$nicName="<name of the NIC of the VM>"
	$lbName="<name of the load balancer instance>"
	$bePoolIndex=<index of the back end pool, starting at 0>
	$lb=Get-AzureLoadBalancer -Name $lbName -ResourceGroupName $rgName 
	$nic=New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -Subnet $vnet.Subnets[$subnetIndex].Id -LoadBalancerBackendAddressPool $lb.BackendAddressPools[$bePoolIndex]

接着，创建一个本地 VM 对象并根据需要将其添加到可用性集中。将下列两个选项中的一个复制到命令集中，并填写名称、大小和可用性集名称。

选项 1：指定虚拟机名称和大小。

	$vmName="<VM name>"
	$vmSize="<VM size string>"
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize

若要确定选项 1 的 VM 大小字符串的可能值，可使用以下命令。

	$locName="<Azure location of your resource group>"
	Get-AzureVMSize -Location $locName | Select Name

选项 2：指定虚拟机的名称和大小并将其添加到可用性集中。

	$vmName="<VM name>"
	$vmSize="<VM size string>"
	$avName="<availability set name>"
	$avSet=Get-AzureAvailabilitySet –Name $avName –ResourceGroupName $rgName
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id

若要确定选项 2 的 VM 大小字符串的可能值，可使用以下命令。

	$rgName="<resource group name>"
	$avName="<availability set name>"
	Get-AzureVMSize -ResourceGroupName $rgName -AvailabilitySetName $avName | Select Name

> [AZURE.NOTE]借助资源管理器，当前只能在虚拟机的创建过程中将其添加到可用性集中。

若要向 VM 添加附加数据磁盘，可将以下行复制到命令集中并指定磁盘设置。

	$diskSize=<size of the disk in GB>
	$diskLabel="<the label on the disk>"
	$diskName="<name identifier for the disk in Azure storage, such as 21050529-DISK02>"
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	Add-AzureVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty

接着，你需要确定虚拟机映像的发布者、产品/服务和 SKU。下面是一个包含基于 Windows 的常用映像的表。

|发布者名称 | 产品/服务名称 | SKU 名称
|:---------------------------------|:-------------------------------------------|:---------------------------------|
|MicrosoftWindowsServer | WindowsServer | 2008-R2-SP1 |
|MicrosoftWindowsServer | WindowsServer | 2012-Datacenter |
|MicrosoftWindowsServer | WindowsServer | 2012-R2-Datacenter |
|MicrosoftDynamicsNAV | DynamicsNAV | 2015 |
|MicrosoftSharePoint | MicrosoftSharePointServer | 2013 |
|MicrosoftSQLServer | SQL2014-WS2012R2 | Enterprise-Optimized-for-DW |
|MicrosoftSQLServer | SQL2014-WS2012R2 | Enterprise-Optimized-for-OLTP |
|MicrosoftWindowsServerEssentials | WindowsServerEssentials | WindowsServerEssentials |
|MicrosoftWindowsServerHPCPack | WindowsServerHPCPack | 2012R2 |

如果未列出你需要的虚拟机映像，请按照<!--[-->此处<!--](/documentation/articles/resource-groups-vm-searching#powershell)-->的说明来确定发布者、产品/服务和 SKU 名称。

将以下命令复制到命令集中，并填写发布者、产品/服务和 SKU 名称。

	$pubName="<Image publisher name>"
	$offerName="<Image offer name>"
	$skuName="<Image SKU name>"
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName $pubName -Offer $offerName -Skus $skuName -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id

最后，将以下命令复制到命令集中，并填写 VM 的操作系统磁盘的名称标识符。

	$diskName="<name identifier for the disk in Azure storage, such as OSDisk>"
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm

## 步骤 5：运行命令集

复查你在文本编辑器或 PowerShell ISE 中生成的包含步骤 4 中多个命令块的 Azure PowerShell 命令集。确保你指定了所需的所有变量，并且这些变量具有正确的值。另请确保你已删除所有 < and > 字符。

如果你的命令在文本编辑器中，则将命令集复制到剪贴板，然后右键单击打开的 Azure PowerShell 提示符。这将发出作为一系列 PowerShell 命令的命令集，并创建 Azure 虚拟机。也可以从 Azure PowerShell ISE 运行命令集。

如果你将再次创建此虚拟机或者创建一个类似的虚拟机，则可以将此命令集另存为 PowerShell 脚本文件 (*.ps1)。

## 示例

我需要 PowerShell 命令集来为基于 Web 的业务线工作负荷创建附加虚拟机，该虚拟机将：

- 放置在现有的 LOBServers 资源组中
- 使用 Windows Server 2012 R2 Datacenter 映像
- 名为 LOB07，并且位于现有的 WEB\_AS 可用性集中
- 在现有 AZDatacenter 虚拟网络的前端子网（子网索引 0）中有一个包含公共 IP 地址的 NIC
- 具有 200 GB 的附加数据磁盘

下面是用于根据步骤 4 所述的过程创建此虚拟机的相应 Azure PowerShell 命令集。

	# Switch to the Resource Manager mode
	Switch-AzureMode AzureResourceManager

	# Set values for existing resource group and storage account names
	$rgName="LOBServers"
	$locName="West US"
	$saName="contosolobserverssa"

	# Set the existing virtual network and subnet index
	$vnetName="AZDatacenter"
	$subnetIndex=0
	$vnet=Get-AzurevirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	# Create the NIC
	$nicName="LOB07-NIC"
	$domName="contoso-vm-lob07"
	$pip=New-AzurePublicIpAddress -Name $nicName -ResourceGroupName $rgName -DomainNameLabel $domName -Location $locName -AllocationMethod Dynamic
	$nic=New-AzureNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

	# Specify the name, size, and existing availability set
	$vmName="LOB07"
	$vmSize="Standard_A3"
	$avName="WEB_AS"
	$avSet=Get-AzureAvailabilitySet –Name $avName –ResourceGroupName $rgName
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id

	# Add a 200 GB additional data disk
	$diskSize=200
	$diskLabel="APPStorage"
	$diskName="21050529-DISK02"
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	Add-AzureVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI -CreateOption empty

	# Specify the image and local administrator account, and then add the NIC
	$pubName="MicrosoftWindowsServer"
	$offerName="WindowsServer"
	$skuName="2012-R2-Datacenter"
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName $pubName -Offer $offerName -Skus $skuName -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id

	# Specify the OS disk name and create the VM
	$diskName="OSDisk"
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm

## 其他资源

[Azure 资源管理器下的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-azurerm-versus-azuresm)

<!--[-->Azure 资源管理器概述<!--](/documentation/articles/resource-group-overview)-->

[使用资源管理器模板与 PowerShell 来部署和管理 Azure 虚拟机](/documentation/articles/virtual-machines-deploy-rmtemplates-powershell)

[使用资源管理器模板和 PowerShell 创建 Windows 虚拟机](/documentation/articles/virtual-machines-create-windows-powershell-resource-manager-template-simple)

[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell)

<!---HONumber=67-->