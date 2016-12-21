<!-- need to be verified -->

<properties
    pageTitle="使用 PowerShell 打开 VM 的端口 | Azure"
    description="了解如何使用 Azure Resource Manager 部署模型和 Azure PowerShell 在 Windows VM 上打开端口/创建终结点"
    services="virtual-machines-windows"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags 
    ms.assetid="cf45f7d8-451a-48ab-8419-730366d54f1e"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="10/27/2016"
    wacn.date="12/20/2016"
    ms.author="iainfou" />

# 使用 PowerShell 打开 Azure 中 VM 的端口和终结点
[AZURE.INCLUDE [virtual-machines-common-nsg-quickstart](../../includes/virtual-machines-common-nsg-quickstart.md)]

## 快速命令
若要创建网络安全组和 ACL 规则，需要[安装最新版本的 Azure PowerShell](/documentation/articles/powershell-install-configure/)。用户也可以[使用 Azure 门户预览执行这些步骤](/documentation/articles/virtual-machines-windows-nsg-quickstart-portal/)。

登录 Azure 帐户：

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`myNetworkSecurityGroup` 和 `myVnet`。

创建规则。以下示例创建名为 `myNetworkSecurityGroupRule` 的规则，以便允许在端口 80 上传输 TCP 流量：

    $httprule = New-AzureRmNetworkSecurityRuleConfig -Name "myNetworkSecurityGroupRule" `
        -Description "Allow HTTP" -Access "Allow" -Protocol "Tcp" -Direction "Inbound" `
        -Priority "100" -SourceAddressPrefix "Internet" -SourcePortRange * `
        -DestinationAddressPrefix * -DestinationPortRange 80

接下来，创建网络安全组，并按如下所示分配刚刚创建的 HTTP 规则。以下示例创建名为 `myNetworkSecurityGroup` 的网络安全组：

    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName "myResourceGroup" `
        -Location "ChinaNorth" -Name "myNetworkSecurityGroup" -SecurityRules $httprule

现在将网络安全组分配给子网。以下示例将名为 `myVnet` 的现有虚拟网络分配给变量 `$vnet`：

    $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Name "myVnet"

将网络安全组与子网相关联。以下示例将名为 `mySubnet` 的子网与网络安全组相关联：

    $subnetPrefix = $vnet.Subnets|?{$_.Name -eq 'mySubnet'}

    Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "mySubnet" `
        -AddressPrefix $subnetPrefix.AddressPrefix `
        -NetworkSecurityGroup $nsg

最后，更新虚拟网络以使做出的更改生效：

    Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

## <a name="more-information-on-network-security-groups"></a> 有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。网络安全组提供许多出色的功能和粒度来控制资源的访问。请参阅[创建网络安全组和 ACL 规则](/documentation/articles/virtual-networks-create-nsg-arm-ps/)了解更多信息。

可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。深入了解[使用模板创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-template/)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，请使用负载均衡器和网络地址转换 (NAT) 规则。例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。了解如何[创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)。

## 后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。下列文章更介绍了有关创建更详细环境的信息：

* [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)
* [什么是网络安全组 (NSG)？](/documentation/articles/virtual-networks-nsg/)
* [Azure Resource Manager 中负载均衡器的概述](/documentation/articles/load-balancer-arm/)

<!---HONumber=Mooncake_1212_2016-->