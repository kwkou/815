<!-- ARM: tested -->

<properties
	pageTitle="创建并配置 VM | Microsoft Azure"
	description="使用 PowerShell 和资源管理器部署模型来创建并配置 Azure 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="10/08/2015"
	wacn.date="06/07/2016"/>

# 使用资源管理器和 Azure PowerShell 创建并配置 Windows 虚拟机

> [AZURE.SELECTOR]
- [门户 - Windows](/documentation/articles/virtual-machines-windows-hero-tutorial)
- [PowerShell](/documentation/articles/virtual-machines-windows-create-powershell)
- [PowerShell - Template](/documentation/articles/virtual-machines-windows-ps-template)
- [门户 - Linux](/documentation/articles/virtual-machines-linux-portal-create)
- [CLI](/documentation/articles/virtual-machines-linux-quick-create-cli)

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代[经典部署模型](/documentation/articles/virtual-machines-windows-classic-create-powershell)。

以下步骤演示如何构建一组 Azure PowerShell 命令，以创建和配置 Azure 虚拟机。你可以使用此构建基块过程快速创建用于新的基于 Windows 的虚拟机的命令集并扩展现有部署。你还可以使用该过程创建多个命令集以快速构建出自定义开发/测试或 IT 专业环境。

这些步骤采用填空方法来创建 Azure PowerShell 命令集。如果你不熟悉 PowerShell 或只想知道为成功的配置指定什么值，则此方法很有用。如果你是高级 PowerShell 用户，可以使用命令并将变量（以“$”开头的行）替换为你自己的值

## 步骤 1：安装 Azure PowerShell

[AZURE.INCLUDE [powershell 预览](../includes/powershell-preview-inline-include.md)]

## 步骤 2：设置订阅

首先，请启动 PowerShell 提示符。

登录到你的帐户。

	Login-AzureRmAccount

使用以下命令获取你的订阅名称。

	Get-AzureRmSubscription | Sort SubscriptionName | Select SubscriptionName

设置你的 Azure 订阅。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<subscription name>"
	Get-AzureRmSubscription -SubscriptionName $subscr | Select-AzureRmSubscription


## 步骤 3：创建资源

在本部分中，我们将说明如何为新虚拟机创建每个资源。

### 资源组


使用资源管理器部署模型创建的 VM 需要一个资源组。如果需要，请为新虚拟机创建新资源组。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	New-AzureRmResourceGroup -Name $rgName -Location $locName

可以使用此命令列出现有的资源组。

	Get-AzureRmResourceGroup | Sort ResourceGroupName | Select ResourceGroupName

### 存储帐户


使用资源管理器部署模型创建的 VM 需要基于资源管理器的存储帐户。如果需要，请使用这些命令创建新虚拟机的新存储帐户。

	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	$saName="<storage account name>"
	$saType="<storage account type, specify one: Standard_LRS, Standard_GRS, Standard_RAGRS, or Premium_LRS>"
	New-AzureRmStorageAccount -Name $saName -ResourceGroupName $rgName –Type $saType -Location $locName

必须为存储帐户选择只包含小写字母和数字的全局唯一名称。可以使用以下命令列出现有的存储帐户。

	Get-AzureRmStorageAccount

若要测试所选存储帐户名称是否为全局唯一名称，需运行 **Test-AzureName** 命令。

	Test-AzureName -Storage <Proposed storage account name>

如果 Test-AzureName 命令显示“False”，则建议的名称是唯一的。


### 公共域名标签


使用资源管理器部署模型创建的 VM 可以使用公共域名标签。该标签只能包含字母、数字和连字符。第一个和最后一个字符必须是字母或数字。

若要测试所选的域名标签是否全局唯一，请使用这些命令。

	$domName="<domain name label to test>"
	$loc="<short name of an Azure location, for example, for West US, the short name is westus>"
	Test-AzureRmDnsAvailability -DomainQualifiedName $domName -Location $loc

如果 DNSNameAvailability 为“True”，则表示提供的名称是全局唯一的。

### 可用性集


如果需要，请使用这些命令创建新虚拟机的新可用性集。

	$avName="<availability set name>"
	$rgName="<resource group name>"
	$locName="<location name, such as West US>"
	New-AzureRmAvailabilitySet –Name $avName –ResourceGroupName $rgName -Location $locName

使用此命令列出现有的可用性集。

	Get-AzureRmAvailabilitySet –ResourceGroupName $rgName | Sort Name | Select Name

### NAT 规则

可以为基于资源管理器的虚拟机配置入站 NAT 规则，以允许来自 Internet 的传入流量并将其放入负载平衡集。在这两种情况下，都必须指定负载平衡器实例和其他设置。

使用资源管理器部署模型创建的 VM 需要资源管理器虚拟网络。如果需要，请使用新虚拟机的至少一个子网创建基于资源管理器的新虚拟网络。以下示例显示了名为 **TestNet** 的新虚拟网络，其中包含名为 **frontendSubnet** 和 **backendSubnet** 的两个子网。

	$rgName="LOBServers"
	$locName="West US"
	$frontendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name frontendSubnet -AddressPrefix 10.0.1.0/24
	$backendSubnet=New-AzureRmVirtualNetworkSubnetConfig -Name backendSubnet -AddressPrefix 10.0.2.0/24
	New-AzureRmVirtualNetwork -Name TestNet -ResourceGroupName $rgName -Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $frontendSubnet,$backendSubnet

请使用这些命令列出现有的虚拟网络。

	$rgName="<resource group name>"
	Get-AzureRmVirtualNetwork -ResourceGroupName $rgName | Sort Name | Select Name

## 步骤 4：生成命令集

打开所选文本编辑器的新实例或 PowerShell 集成脚本环境 (ISE)，并复制以下行以启动你的命令集。指定此新虚拟机的资源组名称、Azure 位置和存储帐户。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$rgName="<resource group name>"
	$locName="<Azure location, such as West US>"
	$saName="<storage account name>"

在虚拟网络中，必须指定基于资源管理器的虚拟网络的名称和子网的索引编号。使用这些命令列出虚拟网络的子网。

	$rgName="<resource group name>"
	$vnetName="<virtual network name>"
	Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName | Select Subnets

子网索引是此命令显示的子网编号，遵循从左到右的顺序并从 0 开始。

对于此示例：

	PS C:\> Get-AzureRmVirtualNetwork -Name TestNet -ResourceGroupName LOBServers | Select Subnets

	Subnets
	-------
	{frontendSubnet, backendSubnet}

frontendSubnet 的子网索引为 0，backendSubnet 的子网索引为 1。

将这几行复制到命令集，并指定现有虚拟网络的名称和虚拟机的子网索引。

	$vnetName="<name of an existing virtual network>"
	$subnetIndex=<index of the subnet on which to create the NIC for the virtual machine>
	$vnet=Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

接下来，创建网络接口卡 (NIC)。将下列其中一个选项复制到命令集，并填入所需的信息。

### 选项 1：指定 NIC 名称并分配公共 IP 地址

将以下几行复制到命令集，并指定 NIC 的名称。

	$nicName="<name of the NIC of the VM>"
	$pip = New-AzureRmPublicIpAddress -Name $nicName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

### 选项 2：指定 NIC 名称和 DNS 域名标签

将以下几行复制到命令集，并指定 NIC 的名称与全局唯一的域名标签。

	$nicName="<name of the NIC of the VM>"
	$domName="<domain name label>"
	$pip = New-AzureRmPublicIpAddress -Name $nicName -ResourceGroupName $rgName -DomainNameLabel $domName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

### 选项 3：指定 NIC 名称并分配静态专用 IP 地址

将以下几行复制到命令集，并指定 NIC 的名称。

	$nicName="<name of the NIC of the VM>"
	$staticIP="<available static IP address on the subnet>"
	$pip = New-AzureRmPublicIpAddress -Name $nicName -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
	$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id -PrivateIpAddress $staticIP

### 选项 4：指定 NIC 名称和入站 NAT 规则的负载平衡器实例。

若要创建 NIC 并将其添加到入站 NAT 规则的负载平衡器实例，你需要：

- 前面创建的、针对转发到虚拟机的流量创建了入站 NAT 规则的负载平衡器实例的名称。
- 要分配给 NIC 的负载平衡器实例的后端地址池的索引编号。
- 要分配给 NIC 的入站 NAT 规则的索引编号。

将以下几行复制到命令集，并指定所需的名称和索引编号。

	$nicName="<name of the NIC of the VM>"
	$lbName="<name of the load balancer instance>"
	$bePoolIndex=<index of the back end pool, starting at 0>
	$natRuleIndex=<index of the inbound NAT rule, starting at 0>
	$lb=Get-AzureRmLoadBalancer -Name $lbName -ResourceGroupName $rgName
	$nic=New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -Subnet $vnet.Subnets[$subnetIndex].Id -LoadBalancerBackendAddressPool $lb.BackendAddressPools[$bePoolIndex] -LoadBalancerInboundNatRule $lb.InboundNatRules[$natRuleIndex]

$nicName 字符串必须是资源组中唯一的字符串。最佳做法是将虚拟机名称合并在字符串中，例如“LOB07-NIC”。

### 选项 5：为负载平衡集指定 NIC 名称和负载平衡器实例。

若要创建 NIC 并将其添加到负载平衡集的负载平衡器实例，你需要：

- 前面创建的、针对负载平衡流量创建了规则的负载平衡器实例的名称。
- 要分配给 NIC 的负载平衡器实例的后端地址池的索引编号。

将以下几行复制到命令集，并指定所需的名称和索引编号。

	$nicName="<name of the NIC of the VM>"
	$lbName="<name of the load balancer instance>"
	$bePoolIndex=<index of the back end pool, starting at 0>
	$lb=Get-AzureRmLoadBalancer -Name $lbName -ResourceGroupName $rgName
	$nic=New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -Subnet $vnet.Subnets[$subnetIndex].Id -LoadBalancerBackendAddressPool $lb.BackendAddressPools[$bePoolIndex]

接下来，创建本地 VM 对象，并选择性地将它添加到可用性集。将以下两个选项中的一个复制到命令集，并填入名称、大小和可用性集名称。

选项 1：指定虚拟机名称和大小。

	$vmName="<VM name>"
	$vmSize="<VM size string>"
	$vm=New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

若要确定选项 1 的 VM 大小字符串的可能值，请使用这些命令。

	$locName="<Azure location of your resource group>"
	Get-AzureRmVMSize -Location $locName | Select Name

选项 2：指定虚拟机名称和大小，并将其添加到可用性集。

	$vmName="<VM name>"
	$vmSize="<VM size string>"
	$avName="<availability set name>"
	$avSet=Get-AzureRmAvailabilitySet –Name $avName –ResourceGroupName $rgName
	$vm=New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id

若要确定选项 2 的 VM 大小字符串的可能值，请使用这些命令。

	$rgName="<resource group name>"
	$avName="<availability set name>"
	Get-AzureRmVMSize -ResourceGroupName $rgName -AvailabilitySetName $avName | Select Name

> [AZURE.NOTE]如果使用资源管理器，目前只能在创建虚拟机期间将其添加到可用性集。

若要将其他数据磁盘添加到 VM，请将这几行复制到命令集，并指定磁盘设置。

	$diskSize=<size of the disk in GB>
	$diskLabel="<the label on the disk>"
	$diskName="<name identifier for the disk in Azure storage, such as 21050529-DISK02>"
	$storageAcc=Get-AzureRmStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	Add-AzureRmVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI  -CreateOption empty

接下来，必须确定虚拟机的发布者、产品名称以及映像的 SKU。以下是基于 Windows 的常用映像表。

|发布者名称 | 产品名称 | SKU 名称
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

如果未列出你需要的虚拟机映像，请按照[此处](/documentation/articles/virtual-machines-linux-cli-ps-findimage#powershell)的说明来确定发布者、产品和 SKU 名称。

将这些命令复制到命令集，并填入发布者、产品和 SKU 名称。

	$pubName="<Image publisher name>"
	$offerName="<Image offer name>"
	$skuName="<Image SKU name>"
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm=Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRmVMSourceImage -VM $vm -PublisherName $pubName -Offer $offerName -Skus $skuName -Version "latest"
	$vm=Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

最后，将这些命令复制到命令集，并填入 VM 操作系统磁盘的名称标识符。

	$diskName="<name identifier for the disk in Azure storage, such as OSDisk>"
	$storageAcc=Get-AzureRmStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"
	$vm=Set-AzureRmVMOSDisk -VM $vm -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRmVM -ResourceGroupName $rgName -Location $locName -VM $vm

## 步骤 5：运行命令集

复查你在步骤 4 中使用文本编辑器或 PowerShell ISE 生成的 Azure PowerShell 命令集。确保指定了所有变量，并且这些变量具有正确的值。另请确保你已删除所有 < and > 字符。

如果你的命令是在文本编辑器中生成的，请将命令集复制到剪贴板，然后右键单击 Azure PowerShell 提示符。这将以 PowerShell 命令序列的形式发出命令集，并创建 Azure 虚拟机。或者，可以通过 Azure PowerShell ISE 运行命令集。

如果你要重复使用这些信息来创建其他 VM，可以将此命令集保存为 PowerShell 脚本文件 (*.ps1)。。

## 示例

我需要 PowerShell 命令集来为基于 Web 的业务线工作负荷创建其他虚拟机，该虚拟机：

- 位于现有的 LOBServers 资源组中
- 使用 Windows Server 2012 R2 Datacenter 映像
- 名称为 LOB07，且位于现有的 WEB\_AS 可用性集中
- 在现有 AZDatacenter 虚拟网络的前端子网（子网索引 0）中具有采用公共 IP 地址的 NIC
- 具有 200 GB 的附加数据磁盘

下面是用于创建此虚拟机的 Azure PowerShell 命令集。

	# Set values for existing resource group and storage account names
	$rgName="LOBServers"
	$locName="West US"
	$saName="contosolobserverssa"

	# Set the existing virtual network and subnet index
	$vnetName="AZDatacenter"
	$subnetIndex=0
	$vnet=Get-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	# Create the NIC
	$nicName="LOB07-NIC"
	$domName="contoso-vm-lob07"
	$pip=New-AzureRmPublicIpAddress -Name $nicName -ResourceGroupName $rgName -DomainNameLabel $domName -Location $locName -AllocationMethod Dynamic
	$nic=New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[$subnetIndex].Id -PublicIpAddressId $pip.Id

	# Specify the name, size, and existing availability set
	$vmName="LOB07"
	$vmSize="Standard_A3"
	$avName="WEB_AS"
	$avSet=Get-AzureRmAvailabilitySet –Name $avName –ResourceGroupName $rgName
	$vm=New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id

	# Add a 200 GB additional data disk
	$diskSize=200
	$diskLabel="APPStorage"
	$diskName="21050529-DISK02"
	$storageAcc=Get-AzureRmStorageAccount -ResourceGroupName $rgName -Name $saName
	$vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	Add-AzureRmVMDataDisk -VM $vm -Name $diskLabel -DiskSizeInGB $diskSize -VhdUri $vhdURI -CreateOption empty

	# Specify the image and local administrator account, and then add the NIC
	$pubName="MicrosoftWindowsServer"
	$offerName="WindowsServer"
	$skuName="2012-R2-Datacenter"
	$cred=Get-Credential -Message "Type the name and password of the local administrator account."
	$vm=Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureRmVMSourceImage -VM $vm -PublisherName $pubName -Offer $offerName -Skus $skuName -Version "latest"
	$vm=Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

	# Specify the OS disk name and create the VM
	$diskName="OSDisk"
	$storageAcc=Get-AzureRmStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + $diskName  + ".vhd"
	$vm=Set-AzureRmVMOSDisk -VM $vm -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureRmVM -ResourceGroupName $rgName -Location $locName -VM $vm

## 其他资源

[Azure 资源管理器中的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-windows-compare-deployment-models)

[Azure 资源管理器概述](/documentation/articles/resource-group-overview)

[使用资源管理器模板与 PowerShell 来部署和管理 Azure 虚拟机](/documentation/articles/virtual-machines-windows-ps-manage)

[使用资源管理器模板和 PowerShell 创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template)

[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)

<!---HONumber=Mooncake_1221_2015-->