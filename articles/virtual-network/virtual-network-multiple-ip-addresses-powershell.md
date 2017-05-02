<properties
    pageTitle="Azure 虚拟机的多个 IP 地址 - PowerShell | Azure"
    description="了解如何使用 PowerShell 将多个 IP 地址分配给虚拟机 | Resource Manager。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="c44ea62f-7e54-4e3b-81ef-0b132111f1f8"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/24/2017"
    wacn.date="05/02/2017"
    ms.author="jdial;annahar"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="c13ff082785efe483776734174d75148a2cbbe66"
    ms.lasthandoff="04/22/2017" />

# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-powershell"></a>使用 PowerShell 将多个 IP 地址分配到虚拟机

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文介绍如何使用 PowerShell 通过 Azure Resource Manager 部署模型创建虚拟机 (VM)。 无法将多个 IP 地址分配到通过经典部署模型创建的资源。 若要详细了解 Azure 部署模型，请阅读[了解部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-template-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>创建具有多个 IP 地址的 VM

下面的步骤说明如何根据方案中所述，创建具有多个 IP 地址的示例 VM。 根据实现的需要，更改变量值。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。 如果尚未安装并配置 PowerShell，请先完成[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 一文中所述的步骤。
2. 使用 `login-azurermaccount` 命令登录你的帐户。
3. 将 myResourceGroup 和 chinanorth 替换为所选名称和位置。 创建资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。

        $RgName   = "MyResourceGroup"
        $Location = "chinanorth"

        New-AzureRmResourceGroup `
        -Name $RgName `
        -Location $Location

4. 在与资源组相同的位置创建虚拟网络 (VNet) 和子网：

        # Create a subnet configuration
        $SubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
        -Name MySubnet `
        -AddressPrefix 10.0.0.0/24

        # Create a virtual network
        $VNet = New-AzureRmVirtualNetwork `
        -ResourceGroupName $RgName `
        -Location $Location `
        -Name MyVNet `
        -AddressPrefix 10.0.0.0/16 `
        -Subnet $subnetConfig

        # Get the subnet object
        $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet

5. 创建网络安全组 (NSG) 和规则。 NSG 使用入站和出站规则保护 VM。 在本例中，将为端口 3389 创建一个入站规则，该规则允许传入的远程桌面连接。

        # Create an inbound network security group rule for port 3389

        $NSGRule = New-AzureRmNetworkSecurityRuleConfig `
        -Name MyNsgRuleRDP `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 1000 `
        -SourceAddressPrefix * `
        -SourcePortRange * `
        -DestinationAddressPrefix * `
        -DestinationPortRange 3389 -Access Allow

        # Create a network security group
        $NSG = New-AzureRmNetworkSecurityGroup `
        -ResourceGroupName $RgName `
        -Location $Location `
        -Name MyNetworkSecurityGroup `
        -SecurityRules $NSGRule

6. 定义 NIC 的主 IP 配置。 如果你没有使用以前定义的值，请将 10.0.0.4 更改为你创建的子网中的有效地址。 分配静态 IP 地址前，建议先确认它未被占用。 输入命令 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.4 -VirtualNetwork $VNet`。 如果该地址可用，输出将返回 *True*。 如果该地址不可用，输出将返回 *False* 以及可用的地址列表。 

    在以下命令中，**使用要用的唯一 DNS 名称替换 <replace-with-your-unique-name>。** 该名称在 Azure 区域内的所有公共 IP 地址中必须唯一。 这是一个可选参数。 如果只想使用公共 IP 地址连接到 VM，则可删除该名称。

        # Create a public IP address
        $PublicIP1 = New-AzureRmPublicIpAddress `
        -Name "MyPublicIP1" `
        -ResourceGroupName $RgName `
        -Location $Location `
        -DomainNameLabel <replace-with-your-unique-name> `
        -AllocationMethod Static

        #Create an IP configuration with a static private IP address and assign the public IP ddress to it
        $IpConfigName1 = "IPConfig-1"
        $IpConfig1     = New-AzureRmNetworkInterfaceIpConfig `
        -Name $IpConfigName1 `
        -Subnet $Subnet `
        -PrivateIpAddress 10.0.0.4 `
        -PublicIpAddress $PublicIP1 `
        -Primary

    向 NIC 分配多个 IP 配置时，必须将一个配置指定为 Primary。

    > [AZURE.NOTE]
    > 公共 IP 地址会产生少许费用。 有关 IP 地址定价的详细信息，请阅读 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。 可在一个订阅中使用的公共 IP 地址数有限制。 有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。

7. 定义 NIC 的辅助 IP 配置。 可以根据需要添加或删除配置。 每个 IP 配置必须分配有专用 IP 地址。 每个配置可选择性分配有一个公共 IP 地址。

        # Create a public IP address
        $PublicIP2 = New-AzureRmPublicIpAddress `
        -Name "MyPublicIP2" `
        -ResourceGroupName $RgName `
        -Location $Location `
        -AllocationMethod Static

        #Create an IP configuration with a static private IP address and assign the public IP ddress to it
        $IpConfigName2 = "IPConfig-2"
        $IpConfig2     = New-AzureRmNetworkInterfaceIpConfig `
        -Name $IpConfigName2 `
        -Subnet $Subnet `
        -PrivateIpAddress 10.0.0.5 `
        -PublicIpAddress $PublicIP2

        $IpConfigName3 = "IpConfig-3"
        $IpConfig3 = New-AzureRmNetworkInterfaceIpConfig `
        -Name $IPConfigName3 `
        -Subnet $Subnet `
        -PrivateIpAddress 10.0.0.6

8. 创建 NIC 并将以下三个 IP 配置与之关联：

        $NIC = New-AzureRmNetworkInterface `
        -Name MyNIC `
        -ResourceGroupName $RgName `
        -Location $Location `
        -NetworkSecurityGroupId $NSG.Id `
        -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3

    >[AZURE.NOTE]
    >尽管本文中的所有配置均分配给了一个 NIC，但可将多个 IP 配置分配给附加到 VM 的每个 NIC。 若要了解如何创建具有多个 NIC 的 VM，请阅读[创建具有多个 NIC 的 VM](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)一文。

9. 通过输入以下命令创建 VM：

        # Define a credential object. When you run these commands, you're prompted to enter a sername and password for the VM you're reating.
        $cred = Get-Credential

        # Create a virtual machine configuration
        $VmConfig = New-AzureRmVMConfig `
        -VMName MyVM `
        -VMSize Standard_DS1_v2 | `
        Set-AzureRmVMOperatingSystem -Windows `
        -ComputerName MyVM `
        -Credential $cred | `
        Set-AzureRmVMSourceImage `
        -PublisherName MicrosoftWindowsServer `
        -Offer WindowsServer `
        -Skus 2016-Datacenter `
        -Version latest | `
        Add-AzureRmVMNetworkInterface `
        -Id $NIC.Id

        # Create the VM
        New-AzureRmVM `
        -ResourceGroupName $RgName `
        -Location $Location `
        -VM $VmConfig

10. 将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。 请勿向操作系统添加公共 IP 地址。

## <a name="add"></a>将 IP 地址添加到 VM

完成以下步骤即可将专用和公共 IP 地址添加到 NIC。 以下部分的示例假定用户的 VM 已完成本文 [方案](#Scenario) 中描述的三项 IP 配置，但这不是必需的。

1. 打开 PowerShell 命令提示符，在单个 PowerShell 会话中完成本部分余下的步骤。 如果尚未安装并配置 PowerShell，请先完成[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 一文中所述的步骤。
2. 将以下 $Variable 的“值”分别更改为要向其添加 IP 地址的 NIC 名称，以及 NIC 所在的资源组和位置：

        $NicName  = "MyNIC"
        $RgName   = "MyResourceGroup"
        $Location = "chinanorth"

    如果不知道要更改的 NIC 名称，请输入以下命令，然后更改上述变量的值：

        Get-AzureRmNetworkInterface | Format-Table Name, ResourceGroupName, Location

3. 键入以下命令创建变量，并将它设置为现有的 NIC：

        $MyNIC = Get-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $RgName

4. 在以下命令中，请将 MyVNet 和 MySubnet 更改为 NIC 连接到的 VNet 和子网的名称。 输入以下命令，检索 NIC 连接到的 VNet 和子网对象：

        $MyVNet = Get-AzureRMVirtualnetwork -Name MyVNet -ResourceGroupName $RgName
        $Subnet = $MyVnet.Subnets | Where-Object { $_.Name -eq "MySubnet" }

    如果不知道 NIC 所连接到的 VNet 或子网名称，请输入以下命令：

        $MyNIC.IpConfigurations

    在输出中查找类似于如下示例输出的文本：

        "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet"

    在此输出中，MyVnet 是 NIC 连接到的 VNet，MySubnet 是其连接到的子网。

5. 根据要求完成以下部分之一中的步骤：

    **添加专用 IP 地址**

    若要将专用 IP 地址添加到 NIC，必须创建 IP 配置。 以下命令创建具有静态 IP 地址 10.0.0.7 的配置。 指定静态 IP 地址时，该地址必须是未使用的子网地址。 建议首先输入 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet` 命令测试地址，确保地址可用。 如果 IP 地址可用，输出会返回 *True*。 如果该地址不可用，输出将返回 *False* 以及可用的地址列表。

        Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
        $MyNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7

    使用唯一的配置名称和专用 IP 地址创建任意数目的配置（适用于具有静态 IP 地址的配置）。

    将专用 IP 地址添加到 VM 操作系统，只需完成本文[将 IP 地址添加到 VM 操作系统](#os-config)部分针对操作系统的步骤即可。

    **添加公共 IP 地址**

    将公共 IP 地址资源关联到新 IP 配置或现有 IP 配置即可添加公共 IP 地址。 根据需要，完成以下任一部分中的步骤。

    > [AZURE.NOTE]
    > 公共 IP 地址会产生少许费用。 有关 IP 地址定价的详细信息，请阅读 [IP 地址定价](/pricing/details/reserved-ip-addresses/)页。 可在一个订阅中使用的公共 IP 地址数有限制。 有关限制的详细信息，请阅读 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)一文。
    >

    - **将公共 IP 地址资源关联到新 IP 配置**

        每次在新 IP 配置中添加公共 IP 地址时，还必须添加专用 IP 地址，因为所有 IP 配置都必须具有专用 IP 地址。 可添加现有公共 IP 地址资源，也可创建新的公共 IP 地址资源。 若要新建此类资源，请输入以下命令：

            $myPublicIp3 = New-AzureRmPublicIpAddress `
            -Name "myPublicIp3" `
            -ResourceGroupName $RgName `
            -Location $Location `
            -AllocationMethod Static

         若要新建具有静态专用 IP 地址和关联的 myPublicIp3 公共 IP 地址资源的 IP 配置，请输入下面的命令：

            Add-AzureRmNetworkInterfaceIpConfig `
            -Name IPConfig-4 `
            -NetworkInterface $myNIC `
            -Subnet $Subnet `
            -PrivateIpAddress 10.0.0.7 `
            -PublicIpAddress $myPublicIp3

    - **将公共 IP 地址资源关联到现有 IP 配置**

        公共 IP 地址资源仅可关联到尚未关联公共 IP 地址资源的 IP 配置。 输入以下命令即可确定某个 IP 配置是否具有关联的公共 IP 地址：

            $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

        将显示类似于下面的输出：

            Name       PrivateIpAddress PublicIpAddress                                           Primary

            IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
            IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
            IpConfig-3 10.0.0.6                                                                     False

        *IpConfig-3* 的 **PublicIpAddress** 列为空，这表示该 IP 配置当前没有任何关联的公共 IP 地址资源。 可将现有公共 IP 地址资源添加到 IpConfig-3，或输入以下命令进行创建：

            $MyPublicIp3 = New-AzureRmPublicIpAddress `
            -Name "MyPublicIp3" `
            -ResourceGroupName $RgName `
            -Location $Location -AllocationMethod Static

        输入以下命令，将公共 IP 地址资源关联到名为 *IPConfig-3*的现有 IP 配置：

            Set-AzureRmNetworkInterfaceIpConfig `
            -Name IpConfig-3 `
            -NetworkInterface $mynic `
            -Subnet $Subnet `
            -PublicIpAddress $myPublicIp3

6. 输入以下命令，为 NIC 设置新 IP 配置：

        Set-AzureRmNetworkInterface -NetworkInterface $MyNIC

7. 输入以下命令，查看分配给 NIC 的专用 IP 地址和公共 IP 地址资源：

        $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary

8. 将专用 IP 地址添加到 VM 操作系统，只需完成本文 [将 IP 地址添加到 VM 操作系统](#os-config) 部分针对操作系统的步骤即可。 请勿向操作系统添加公共 IP 地址。

[AZURE.INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]

<!--Update_Description: wording update-->