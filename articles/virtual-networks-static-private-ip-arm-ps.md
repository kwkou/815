<!-- ARM: tested -->

<properties 
   pageTitle="如何在 Azure 资源管理器中使用 PowerShell 设置静态专用 IP 地址 | Azure"
   description="了解静态专用 IP 地址以及如何在 Azure 资源管理器中使用 PowerShell 对其进行管理"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/23/2016"
	wacn.date="06/06/2016"/>

# 如何在 Azure 资源管理器中使用 PowerShell 设置静态专用 IP 地址

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../includes/virtual-networks-static-private-ip-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍资源管理器部署模型。你还可以[管理经典部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../includes/virtual-networks-static-ip-scenario-include.md)]

下面的示例 PowerShell 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-arm-ps/) 中所述的测试环境。

## 如何在创建 VM 时指定静态专用 IP 地址
若要在名为 *TestVNet* 的 VNet 的 *FrontEnd* 子网中使用静态专用 IP *192.168.1.101* 创建名为 *DNS01* 的 VM，请按照以下步骤进行操作：

[AZURE.INCLUDE [powershell-preview-include.md](../includes/powershell-preview-include.md)]

1. 为要使用的存储帐户、位置、资源组和凭据设置变量。你将需要为 VM 输入用户名和密码。存储帐户和资源组必须已存在。

		$stName = "vnetstorage"
		$locName = "China North"
		$rgName = "TestRG"
	    $cred = Get-Credential -Message "Type the name and password of the local administrator account."

3. 检索你要在其中创建 VM 的虚拟网络和子网。

	    $vnet = Get-AzureVirtualNetwork -ResourceGroupName TestRG -Name TestVNet	
	    $subnet = $vnet.Subnets[0].Id

4. 如有必要，创建用于从 Internet 访问 VM 的公共 IP 地址。

		$pip = New-AzurePublicIpAddress -Name TestPIP -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic

5. 使用要分配给 VM 的静态专用 IP 地址创建 NIC。确保 IP 在你要将 VM 添加到的子网范围中。这是本文的主要步骤，在本文中将专用 IP 设为静态。

		$nic = New-AzureNetworkInterface -Name TestNIC -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -PrivateIpAddress 192.168.1.101

6. 使用前面创建的 NIC 创建 VM。

		$vm = New-AzureVMConfig -VMName DNS01 -VMSize "Standard_A1"
		$vm = Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName DNS01  -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
		$vm = Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
		$vm = Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
		$osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/WindowsVMosDisk.vhd"
		$vm = Set-AzureVMOSDisk -VM $vm -Name "windowsvmosdisk" -VhdUri $osDiskUri -CreateOption fromImage
		New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm 

	预期输出：

		EndTime             : 9/8/2015 2:32:09 PM -07:00
		Error               : 
		Output              : 
		StartTime           : 9/8/2015 2:27:42 PM -07:00
		Status              : Succeeded
		TrackingOperationId : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		RequestId           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
		StatusCode          : OK 


## 如何检索 VM 的静态专用 IP 地址信息
若要查看使用上述脚本创建的 VM 的静态专用 IP 地址信息，请运行以下 PowerShell 命令并观察 *PrivateIpAddress* 和 *PrivateIpAllocationMethod* 的值：

	Get-AzureNetworkInterface -Name TestNIC -ResourceGroupName TestRG

预期输出：

	Name                 : TestNIC
	ResourceGroupName    : TestRG
	Location             : centralus
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/Te
	                       stNIC
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ProvisioningState    : Succeeded
	Tags                 : 
	VirtualMachine       : {
	                         "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMach
	                       ines/DNS01"
	                       }
	IpConfigurations     : [
	                         {
	                           "Name": "ipconfig1",
	                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkIn
	                       terfaces/TestNIC/ipConfigurations/ipconfig1",
	                           "PrivateIpAddress": "192.168.1.101",
	                           "PrivateIpAllocationMethod": "Static",
	                           "Subnet": {
	                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtual
	                       Networks/TestVNet/subnets/FrontEnd"
	                           },
	                           "PublicIpAddress": {
	                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/publicI
	                       PAddresses/TestPIP"
	                           },
	                           "LoadBalancerBackendAddressPools": [],
	                           "LoadBalancerInboundNatRules": [],
	                           "ProvisioningState": "Succeeded"
	                         }
	                       ]
	DnsSettings          : {
	                         "DnsServers": [],
	                         "AppliedDnsServers": [],
	                         "InternalDnsNameLabel": null,
	                         "InternalFqdn": null
	                       }
	EnableIPForwarding   : False
	NetworkSecurityGroup : null
	Primary              : True

## 如何从 VM 中删除静态专用 IP 地址
若要删除使用上述脚本添加到 VM 的静态专用 IP 地址，请运行以下 PowerShell 命令：
	
	$nic=Get-AzureNetworkInterface -Name TestNIC -ResourceGroupName TestRG
	$nic.IpConfigurations[0].PrivateIpAllocationMethod = "Dynamic"
	Set-AzureNetworkInterface -NetworkInterface $nic

预期输出：

	Name                 : TestNIC
	ResourceGroupName    : TestRG
	Location             : centralus
	Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/Te
	                       stNIC
	Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
	ProvisioningState    : Succeeded
	Tags                 : 
	VirtualMachine       : {
	                         "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMach
	                       ines/WindowsVM"
	                       }
	IpConfigurations     : [
	                         {
	                           "Name": "ipconfig1",
	                           "Etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	                           "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkIn
	                       terfaces/TestNIC/ipConfigurations/ipconfig1",
	                           "PrivateIpAddress": "192.168.1.101",
	                           "PrivateIpAllocationMethod": "Dynamic",
	                           "Subnet": {
	                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtual
	                       Networks/TestVNet/subnets/FrontEnd"
	                           },
	                           "PublicIpAddress": {
	                             "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/publicI
	                       PAddresses/TestPIP"
	                           },
	                           "LoadBalancerBackendAddressPools": [],
	                           "LoadBalancerInboundNatRules": [],
	                           "ProvisioningState": "Succeeded"
	                         }
	                       ]
	DnsSettings          : {
	                         "DnsServers": [],
	                         "AppliedDnsServers": [],
	                         "InternalDnsNameLabel": null,
	                         "InternalFqdn": null
	                       }
	EnableIPForwarding   : False
	NetworkSecurityGroup : null
	Primary              : True

## 如何将静态专用 IP 地址添加到现有 VM
若要向使用上述脚本创建的 VM 添加静态专用 IP 地址，请运行以下命令：

	$nic=Get-AzureNetworkInterface -Name TestNIC -ResourceGroupName TestRG
	$nic.IpConfigurations[0].PrivateIpAllocationMethod = "Static"
	$nic.IpConfigurations[0].PrivateIpAddress = "192.168.1.101"
	Set-AzureNetworkInterface -NetworkInterface $nic

## 后续步骤

- 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
- 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
- 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=Mooncake_1221_2015-->