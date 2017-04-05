<properties
    pageTitle="为 VM 配置专用 IP 地址 - Azure PowerShell | Azure"
    description="了解如何使用 PowerShell 为虚拟机配置专用 IP 地址。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="d5f18929-15e3-40a2-9ee3-8188bc248ed8"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/23/2016"
    wacn.date="03/31/2017"
    ms.author="jdial"
    ms.custom="H1Hack27Feb2017" />  


# 使用 PowerShell 为虚拟机配置专用 IP 地址

[AZURE.INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

Azure 有两个部署模型：Azure Resource Manager 和经典模型。Azure 建议通过 Resource Manager 部署模型创建资源。若要深入了解这两个模型之间的差异，请阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。本文介绍 Resource Manager 部署模型。你还可以[管理经典部署模型中的静态专用 IP 地址](/documentation/articles/virtual-networks-static-private-ip-classic-ps/)。

[AZURE.INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

以下示例 PowerShell 命令需要基于以上方案创建的简单环境。若要按本文档所示运行命令，首先需要构建[创建 VNet](/documentation/articles/virtual-networks-create-vnet-arm-ps/) 中所述的测试环境。

## 使用静态专用 IP 地址创建 VM
若要在名为 *TestVNet* 的 VNet 的 *FrontEnd* 子网中使用静态专用 IP *192.168.1.101* 创建名为 *DNS01* 的 VM，请按照以下步骤进行操作：

1. 为要使用的存储帐户、位置、资源组和凭据设置变量。需要为 VM 输入用户名和密码。存储帐户和资源组必须已存在。

        $stName  = "vnetstorage"
        $locName = "China North"
        $rgName  = "TestRG"
        $cred    = Get-Credential -Message "Type the name and password of the local administrator account."

2. 检索要在其中创建 VM 的虚拟网络和子网。

        $vnet   = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
        $subnet = $vnet.Subnets[0].Id

3. 如有必要，创建用于从 Internet 访问 VM 的公共 IP 地址。

        $pip = New-AzureRmPublicIpAddress -Name TestPIP -ResourceGroupName $rgName `
        -Location $locName -AllocationMethod Dynamic

4. 使用要分配给 VM 的静态专用 IP 地址创建 NIC。确保 IP 处于要将 VM 添加到的子网范围中。这是本文的主要步骤，其中将专用 IP 设为静态。

        $nic = New-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName $rgName `
        -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id `
        -PrivateIpAddress 192.168.1.101

5. 使用前面创建的 NIC 创建 VM。

        $vm = New-AzureRmVMConfig -VMName DNS01 -VMSize "Standard_A1"
        $vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName DNS01 `
        -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
        $vm = Set-AzureRmVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer `
        -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
        $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
        $osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/WindowsVMosDisk.vhd"
        $vm = Set-AzureRmVMOSDisk -VM $vm -Name "windowsvmosdisk" -VhdUri $osDiskUri `
        -CreateOption fromImage
        New-AzureRmVM -ResourceGroupName $rgName -Location $locName -VM $vm 

    预期输出：
	
        EndTime             : [Date and time]
        Error               : 
        Output              : 
        StartTime           : [Date and time]
        Status              : Succeeded
        TrackingOperationId : [Id]
        RequestId           : [Id]
        StatusCode          : OK 

## 检索网络接口的静态专用 IP 地址信息
若要查看使用上述脚本创建的 VM 的静态专用 IP 地址信息，请运行以下 PowerShell 命令并观察 *PrivateIpAddress* 和 *PrivateIpAllocationMethod* 的值：

    Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG

预期输出：

    Name                 : TestNIC
    ResourceGroupName    : TestRG
    Location             : chinaeast
    Id                   : /subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
    Etag                 : W/"[Id]"
    ProvisioningState    : Succeeded
    Tags                 : 
    VirtualMachine       : {
                             "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01"
                           }
    IpConfigurations     : [
                             {
                               "Name": "ipconfig1",
                               "Etag": "W/\"[Id]\"",
                               "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                               "PrivateIpAddress": "192.168.1.101",
                               "PrivateIpAllocationMethod": "Static",
                               "Subnet": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
                               },
                               "PublicIpAddress": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP"
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

## 删除网络接口的静态专用 IP 地址
若要删除使用上述脚本添加到 VM 的静态专用 IP 地址，请运行以下 PowerShell 命令：

    $nic=Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG
    $nic.IpConfigurations[0].PrivateIpAllocationMethod = "Dynamic"
    Set-AzureRmNetworkInterface -NetworkInterface $nic

预期输出：

    Name                 : TestNIC
    ResourceGroupName    : TestRG
    Location             : chinaeast
    Id                   : /subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
    Etag                 : W/"[Id]"
    ProvisioningState    : Succeeded
    Tags                 : 
    VirtualMachine       : {
                             "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/WindowsVM"
                           }
    IpConfigurations     : [
                             {
                               "Name": "ipconfig1",
                               "Etag": "W/\"[Id]\"",
                               "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                               "PrivateIpAddress": "192.168.1.101",
                               "PrivateIpAllocationMethod": "Dynamic",
                               "Subnet": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
                               },
                               "PublicIpAddress": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP"
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

## 将静态专用 IP 地址添加到网络接口
若要向使用上述脚本创建的 VM 添加静态专用 IP 地址，请运行以下命令：

    $nic=Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG
    $nic.IpConfigurations[0].PrivateIpAllocationMethod = "Static"
    $nic.IpConfigurations[0].PrivateIpAddress = "192.168.1.101"
    Set-AzureRmNetworkInterface -NetworkInterface $nic

## 更改分配给网络接口的专用 IP 地址的分配方法

使用静态或动态分配方法将专用 IP 地址分配给 NIC。启动原先处于停止（已释放）状态的 VM 之后，动态 IP 地址可能出现更改。如果 VM 要托管需要相同 IP 地址的服务，则在停止（已释放）状态下重启之后，这仍可能导致问题。将保留静态 IP 地址，直到 VM 已删除。若要更改 IP 地址的分配方法，请运行以下脚本，将分配方法从动态更改为静态。如果当前专用 IP 地址采用静态分配方法，请在执行脚本前将*静态*更改为*动态*。

    $RG = "TestRG"
    $NIC_name = "testnic1"

    $nic = Get-AzureRmNetworkInterface -ResourceGroupName $RG -Name $NIC_name
    $nic.IpConfigurations[0].PrivateIpAllocationMethod = 'Static'
    Set-AzureRmNetworkInterface -NetworkInterface $nic 
    $IP = $nic.IpConfigurations[0].PrivateIpAddress

    Write-Host "The allocation method is now set to"$nic.IpConfigurations[0].PrivateIpAllocationMethod"for the IP address" $IP"." -NoNewline

如果不知道 NIC 的名称，可输入以下命令来查看资源组中 NIC 的列表：

    Get-AzureRmNetworkInterface -ResourceGroupName $RG | Where-Object {$_.ProvisioningState -eq 'Succeeded'} 

## 后续步骤
* 了解[保留公共 IP](/documentation/articles/virtual-networks-reserved-public-ip/) 地址。
* 了解[实例层级公共 IP (ILPIP)](/documentation/articles/virtual-networks-instance-level-public-ip/) 地址。
* 查阅[保留 IP REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn722420.aspx)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: add "Change the allocation method for a private IP address assigned to a network interface"-->