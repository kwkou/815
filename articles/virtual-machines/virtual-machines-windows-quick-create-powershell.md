<properties
    pageTitle="Azure 快速入门 - 创建 Windows VM PowerShell | Azure"
    description="快速了解如何使用 PowerShell 创建 Windows 虚拟机"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="04/03/2017"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="da44851f58b46e8c635823ca436b1dbda9cb8b91"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-windows-virtual-machine-with-powershell"></a>使用 PowerShell 创建 Windows 虚拟机

Azure PowerShell 模块用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 本指南详细说明了如何使用 PowerShell 创建运行 Windows Server 2016 的 Azure 虚拟机。  部署完成后，我们将连接到服务器并安装 IIS。  

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](/pricing/1rmb-trial/)。

另请确保已安装了 Azure PowerShell 模块的最新版本。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

## <a name="log-in-to-azure"></a>登录 Azure

使用 `Login-AzureRmAccount` 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="create-resource-group"></a>创建资源组

创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

    New-AzureRmResourceGroup -Name myResourceGroup -Location chinanorth

## <a name="create-networking-resources"></a>创建网络资源

### <a name="create-a-virtual-network-subnet-and-a-public-ip-address"></a>创建虚拟网络、子网和公共 IP 地址。 
这些资源用来与虚拟机建立网络连接，以及连接到 Internet。

    # Create a subnet configuration
    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

    # Create a virtual network
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

    # Create a public IP address and specify a DNS name
    $pip = New-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup -Location chinanorth `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>创建网络安全组和网络安全组规则。 
网络安全组使用入站和出站规则保护虚拟机。 在本例中，将为端口 3389 创建一个入站规则，该规则允许传入的远程桌面连接。  我们还需要为端口 80 创建入站规则，以允许传入的 Web 流量。

    # Create an inbound network security group rule for port 3389
    $nsgRuleRDP = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

    # Create an inbound network security group rule for port 80
    $nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

    # Create a network security group
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP,$nsgRuleWeb

### <a name="create-a-network-card-for-the-virtual-machine"></a>为虚拟机创建网卡。 
网卡将虚拟机连接到子网、网络安全组和公共 IP 地址。

    # Create a virtual network card and associate with public IP address and NSG
    $nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location chinanorth `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

## <a name="create-virtual-machine"></a>创建虚拟机

创建虚拟机配置。 此配置包括部署虚拟机时使用的设置，例如虚拟机映像、大小和身份验证配置。 运行此步骤时，会提示你输入凭据。 你输入的值将配置为用于虚拟机的用户名和密码。

    # Define a credential object
    $cred = Get-Credential

    # Create a virtual machine configuration
    $vmConfig = New-AzureRmVMConfig -VMName myVM -VMSize Standard_DS2 | `
    Set-AzureRmVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzureRmVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
    Add-AzureRmVMNetworkInterface -Id $nic.Id

创建虚拟机。

    New-AzureRmVM -ResourceGroupName myResourceGroup -Location chinanorth -VM $vmConfig

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

在部署完成后，创建到虚拟机的远程桌面连接。

运行以下命令，以返回虚拟机的公共 IP 地址。  需记下此 IP 地址，以便在后续步骤中使用浏览器连接到它测试 Web 连接。

    Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress

使用以下命令创建与虚拟机的远程桌面会话。 将 IP 地址替换为你的虚拟机的 `publicIPAddress`。 出现提示时，输入创建虚拟机时使用的凭据。

    mstsc /v:<publicIpAddress>

## <a name="install-iis-via-powershell"></a>通过 PowerShell 安装 IIS

现在，你已登录到 Azure VM，可以使用单行 PowerShell 安装 IIS，并启用本地防火墙规则以允许 Web 流量。  打开 PowerShell 提示符并运行以下命令：

    Install-WindowsFeature -name Web-Server -IncludeManagementTools

## <a name="view-the-iis-welcome-page"></a>查看 IIS 欢迎页

IIS 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 你可以使用所选的 Web 浏览器查看默认的 IIS 欢迎页。 请务必使用前面记录的 `publicIpAddress` 访问默认页面。 

![IIS 默认站点](./media/virtual-machines-windows-quick-create-powershell/default-iis-website.png) 

## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    Remove-AzureRmResourceGroup -Name myResourceGroup

## <a name="next-steps"></a>后续步骤

[安装角色和配置防火墙教程](/documentation/articles/virtual-machines-windows-hero-role/)

[浏览 VM 部署 PowerShell 示例](/documentation/articles/virtual-machines-windows-powershell-samples/)