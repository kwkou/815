<properties
    pageTitle="Azure 虚拟机的多个 IP 地址 - PowerShell | Azure"
    description="了解如何使用 PowerShell 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="c44ea62f-7e54-4e3b-81ef-0b132111f1f8"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/30/2016"
    wacn.date="03/31/2017"
    ms.author="jdial;annahar" />  


# 使用 PowerShell 将多个 IP 地址分配给虚拟机

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文介绍如何使用 PowerShell 通过 Azure Resource Manager 部署模型创建虚拟机 (VM)。无法将多个 IP 地址分配给通过经典部署模型创建的资源。若要了解有关 Azure 部署模型的详细信息，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-preview](../../includes/virtual-network-preview.md)]

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-template-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>创建具有多个 IP 地址的 VM

以下步骤说明如何根据本方案所述，创建具有多个 IP 地址的示例 VM。根据实现的需要，更改变量名称和 IP 地址类型。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）一文中所述的步骤。
2. 通过登录后在 PowerShell 中运行以下命令并选择相应的订阅来注册预览版：

        Register-AzureRmProviderFeature -FeatureName AllowMultipleIpConfigurationsPerNic -ProviderNamespace Microsoft.Network

        Register-AzureRmProviderFeature -FeatureName AllowLoadBalancingonSecondaryIpconfigs -ProviderNamespace Microsoft.Network
    
        Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network

    请不要尝试完成剩余步骤，直至运行 ```Get-AzureRmProviderFeature``` 命令时看到以下输出：

        FeatureName                            ProviderName      RegistrationState
        -----------                            ------------      -----------------      
        AllowLoadBalancingOnSecondaryIpConfigs Microsoft.Network Registered       
        AllowMultipleIpConfigurationsPerNic    Microsoft.Network Registered       

    >[AZURE.NOTE] 
    这可能需要几分钟的时间。

3. 完成[创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/) 一文中的步骤 1-4。请不要完成步骤 5（创建公共 IP 资源和网络接口）。如果更改该文中使用的任何变量的名称，请同样更改剩余步骤中的变量名称。若要创建 Linux VM，请选择 Linux 操作系统，而不要选择 Windows。
4. 键入以下命令，创建一个变量，用于存储“创建 Windows VM”一文的步骤 4（创建 VNet）中创建的子网对象：

        $SubnetName = $mySubnet.Name
        $Subnet = $myVnet.Subnets | Where-Object { $_.Name -eq $SubnetName }

5. 定义要分配给 NIC 的 IP 配置。可根据需要添加、删除或更改配置。方案所述的配置如下所示：

    **IPConfig-1**

    输入以下命令，创建：
    - 一个具有静态公共 IP 地址的公共 IP 地址资源
    - 一个具有公共 IP 地址资源和动态专用 IP 地址的 IP 配置

        $myPublicIp1     = New-AzureRmPublicIpAddress -Name "myPublicIp1" -ResourceGroupName $myResourceGroup -Location $location -AllocationMethod Static
        $IpConfigName1  = "IPConfig-1"
        $IpConfig1      = New-AzureRmNetworkInterfaceIpConfig -Name $IpConfigName1 -Subnet $Subnet -PublicIpAddress $myPublicIp1 -Primary

    请注意以上命令中的 `-Primary` 开关。为 NIC 分配多个 IP 配置时，必须将一个配置指定为 *Primary*。

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
    >

    **IPConfig-2**

    更改所创建子网上的可用有效地址后的 **$IPAddress** 变量的值。若要检查地址 10.0.0.5 在子网上是否可用，请输入命令 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.5 -VirtualNetwork $myVnet`。如果该地址可用，输出会返回 *True*。如果不可用，输出会返回 *False* 以及可用地址的列表。输入以下命令，创建具有一个静态公共 IP 地址和一个静态专用 IP 地址的新公共 IP 地址资源和新 IP 配置：

        $IpConfigName2 = "IPConfig-2"
        $IPAddress     = "10.0.0.5"
        $myPublicIp2   = New-AzureRmPublicIpAddress -Name "myPublicIp2" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static
        $IpConfig2     = New-AzureRmNetworkInterfaceIpConfig -Name $IpConfigName2 `
        -Subnet $Subnet -PrivateIpAddress $IPAddress -PublicIpAddress $myPublicIp2

    **IPConfig-3**

    输入以下命令，创建具有一个动态专用 IP 地址且没有公共 IP 地址的 IP 配置：

        $IpConfigName3 = "IpConfig-3"
        $IpConfig3 = New-AzureRmNetworkInterfaceIpConfig -Name $IPConfigName3 -Subnet $Subnet

6. 输入以下命令，使用上一步骤中定义的 IP 配置创建 NIC：

        $myNIC = New-AzureRmNetworkInterface -Name myNIC -ResourceGroupName $myResourceGroup `
        -Location $location -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3

    > [AZURE.NOTE]
    尽管本文将所有 IP 配置都分配给一个 NIC，但还可将多个 IP 配置分配给 VM 中的任何 NIC。若要了解如何创建具有多个 NIC 的 VM，请阅读[创建具有多个 NIC 的 VM](/documentation/articles/virtual-network-deploy-multinic-arm-ps/) 一文。

7. 完成[创建 VM](/documentation/articles/virtual-machines-windows-ps-create/) 一文的步骤 6。

    > [AZURE.WARNING]
    在以下情况中，“创建 VM”一文中的步骤 6 会失败：
    > - 在本文的步骤 6 中将名为 $myNIC 的变量更改为其他名称。
    > - 尚未完成本文以及“创建 VM”一文中的之前步骤。
    >
8. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

9. 将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。请勿向操作系统添加公共 IP 地址。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤即可将专用和公共 IP 地址添加到 NIC。以下部分的示例假定用户的 VM 已完成本文[方案](#Scenario)中描述的三项 IP 配置，但这不是必需的。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。如果尚未安装并配置 PowerShell，请先完成[How to install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell）一文中所述的步骤。
2. 按照**创建具有多个 IP 地址的 VM** 部分中的步骤 2，注册公共预览版。
3. 将以下 $Variable 的“值”更改为要向其添加 IP 地址的 NIC 的名称，以及 NIC 所在的资源组和位置：

        $NICname         = "myNIC"
        $myResourceGroup = "myResourceGroup"
        $location        = "chinaeast"

    如果不知道要更改的 NIC 名称，请输入以下命令，然后更改上述变量的值：

        Get-AzureRmNetworkInterface | Format-Table Name, ResourceGroupName, Location

4. 键入以下命令创建变量，并将它设置为现有的 NIC：

        $myNIC = Get-AzureRmNetworkInterface -Name $NICname -ResourceGroupName $myResourceGroup

5. 在以下命令中，将 *myVNet* 和 *mySubnet* 更改为 NIC 连接到的 VNet 和子网的名称。输入以下命令，检索 NIC 连接到的 VNet 和子网对象：

        $myVnet = Get-AzureRMVirtualnetwork -Name myVNet -ResourceGroupName $myResourceGroup
        $Subnet = $myVnet.Subnets | Where-Object { $_.Name -eq "mySubnet" }

    如果不知道 NIC 连接到的 VNet 或子网的名称，请输入以下命令：

        $mynic.IpConfigurations

    在返回的输出中查找类似如下所示的文本：

        Subnet   : {
                     "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"

    在此输出中，*myVnet* 和 *mySubnet* 分别是 NIC 连接到的 VNet 和子网。

6. 根据要求，完成以下任一部分中的步骤：

    **添加专用 IP 地址**

    若要将专用 IP 地址添加到 NIC，必须创建 IP 配置。以下命令创建具有静态 IP 地址 10.0.0.7 的配置。如果想要添加动态专用 IP 地址，请在输入命令前删除 `-PrivateIpAddress 10.0.0.7`。指定的静态 IP 地址必须是子网的未使用地址。建议首先输入 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet` 命令测试地址，确保地址可用。如果 IP 地址可用，输出会返回 *True*。如果不可用，输出会返回 *False* 以及可用地址的列表。

        Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
         $myNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7

    使用唯一配置名称和专用 IP 地址（用于具有静态 IP 地址的配置），根据需要创建多个配置。

    将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。

    **添加公共 IP 地址**

    将公共 IP 地址资源关联到新 IP 配置或现有 IP 配置即可添加公共 IP 地址。根据需要，完成以下任一部分中的步骤。

    > [AZURE.NOTE]
    公共 IP 地址会产生少许费用。有关 IP 地址定价的详细信息，请阅读 [IP address pricing](/pricing/details/reserved-ip-addresses/)（IP 地址定价）页。可在一个订阅中使用的公共 IP 地址数有限制。若要了解有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
    >

    **将公共 IP 地址资源关联到新 IP 配置**
	
    每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。若要新建，请输入以下命令：

        $myPublicIp3   = New-AzureRmPublicIpAddress -Name "myPublicIp3" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static

    若要创建具有动态专用 IP 地址和关联的 *myPublicIP3* 公共 IP 地址资源的新 IP 配置，请输入以下命令：

        Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
         $myNIC -Subnet $Subnet -PublicIpAddress $myPublicIp3

    **将公共 IP 地址资源关联到现有 IP 配置**

    公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。可输入以下命令，确定某个 IP 配置是否具有关联的公共 IP 地址：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

    在返回的输出中查找类似如下所示的行：

        Name       PrivateIpAddress PublicIpAddress                                           Primary
        ----       ---------------- ---------------                                           -------
        IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
        IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
        IpConfig-3 10.0.0.6                                                                     False

    *IpConfig-3* 的 **PublicIpAddress** 列为空白，因此，当前没有公共 IP 地址资源与其关联。可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

        $myPublicIp3   = New-AzureRmPublicIpAddress -Name "myPublicIp3" -ResourceGroupName $myResourceGroup `
        -Location $location -AllocationMethod Static

    输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3* 的现有 IP 配置：

        Set-AzureRmNetworkInterfaceIpConfig -Name IpConfig-3 -NetworkInterface $mynic -Subnet $Subnet -PublicIpAddress $myPublicIp3

7. 输入以下命令，为 NIC 设置新 IP 配置：

        Set-AzureRmNetworkInterface -NetworkInterface $myNIC

8. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        $myNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

9. 将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。请勿向操作系统添加公共 IP 地址。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->