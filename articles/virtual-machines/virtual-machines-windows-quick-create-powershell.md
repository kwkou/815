<properties
    pageTitle="Azure 快速入门 - 创建 Windows VM PowerShell | Azure"
    description="快速了解如何使用 PowerShell 创建 Windows 虚拟机"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="03/14/2017"
    wacn.date="04/17/2017"
    ms.author="nepeters"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="3dbc944f4f7e407166c0f9a046022cd33076695a"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-windows-virtual-machine-with-powershell"></a>使用 PowerShell 创建 Windows 虚拟机

Azure PowerShell 模块用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 本指南详细说明了如何使用 PowerShell 创建运行 Windows Server 2016 的 Azure 虚拟机。 

在开始之前，请确保已安装了 Azure PowerShell 模块的最新版本。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

## <a name="log-in-to-azure"></a>登录 Azure

使用 `Login-AzureRmAccount` 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="create-resource-group"></a>创建资源组

创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

    New-AzureRmResourceGroup -Name myResourceGroup -Location chinanorth

## <a name="create-networking-resources"></a>创建网络资源

创建虚拟网络、子网和公共 IP 地址。 这些资源用来与虚拟机建立网络连接，以及连接到 Internet。

    # Create a subnet configuration
    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

    # Create a virtual network
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

    # Create a public IP address and specify a DNS name
    $pip = New-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup -Location chinanorth `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"

创建网络安全组和网络安全组规则。 网络安全组使用入站和出站规则保护虚拟机。 在本例中，将为端口 3389 创建一个入站规则，该规则允许传入的远程桌面连接。

    # Create an inbound network security group rule for port 3389
    $nsgRuleRDP = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

    # Create a network security group
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP

为虚拟机创建网卡。 网卡将虚拟机连接到子网、网络安全组和公共 IP 地址。

    # Create a virtual network card and associate with public IP address and NSG
    $nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location chinanorth `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

## <a name="create-virtual-machine"></a>创建虚拟机

创建虚拟机配置。 此配置包括部署虚拟机时使用的设置，例如虚拟机映像、大小和身份验证配置。 运行此步骤时，会提示你输入凭据。 你输入的值将配置为用于虚拟机的用户名和密码。

    # Define a credential object
    $cred = Get-Credential

    # Create a virtual machine configuration
    $vmConfig = New-AzureRmVMConfig -VMName myVM -VMSize Standard_D1 | `
    Set-AzureRmVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzureRmVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
    Add-AzureRmVMNetworkInterface -Id $nic.Id

创建虚拟机。

    New-AzureRmVM -ResourceGroupName myResourceGroup -Location chinanorth -VM $vmConfig

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

在部署完成后，创建到虚拟机的远程桌面连接。

运行以下命令，以返回虚拟机的公共 IP 地址。

    Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress

使用以下命令创建到虚拟机的远程桌面连接。 将 IP 地址替换为你的虚拟机的公共 IP 地址。 出现提示时，输入创建虚拟机时使用的凭据。

    mstsc /v:<Public IP Address>

## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    Remove-AzureRmResourceGroup -Name myResourceGroup

## <a name="next-steps"></a>后续步骤

[安装角色和配置防火墙教程](/documentation/articles/virtual-machines-windows-hero-role/)

[浏览 VM 部署 PowerShell 示例](/documentation/articles/virtual-machines-windows-powershell-samples/)