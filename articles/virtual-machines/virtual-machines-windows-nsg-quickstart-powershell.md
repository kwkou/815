<properties
   pageTitle="使用 PowerShell 对 VM 实现外部访问 | Azure"
   description="了解如何打开端口/创建终结点，以便允许使用 Resource Manager 部署模型和 Azure PowerShell 从外部访问 Windows VM"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/24/2016"
	wacn.date="07/11/2016"/>

# 使用 PowerShell 对 VM 实现外部访问
[AZURE.INCLUDE [virtual-machines-common-nsg-quickstart](../includes/virtual-machines-common-nsg-quickstart.md)]

## 快速命令
若要创建网络安全组和 ACL 规则，你需要 [ 安装最新版本的 Azure PowerShell](/documentation/articles/powershell-install-configure/)。你也可以 [ 使用 Azure 门户预览执行这些步骤](/documentation/articles/virtual-machines-windows-nsg-quickstart-portal/)。

首先，你需要创建规则以允许 TCP 端口 80 上的 HTTP 流量输入你自己的名称和描述：

	$httprule = New-AzureRmNetworkSecurityRuleConfig -Name http-rule -Description "Allow HTTP" `
	    -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
	    -SourceAddressPrefix Internet -SourcePortRange * `
	    -DestinationAddressPrefix * -DestinationPortRange 80

接下来，创建网络安全组并按以下步骤分配刚刚创建的 HTTP 规则，输入你自己的资源组名称和位置：

	$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location chinanorth `
	    -Name "TestNSG" -SecurityRules $httprule

现在我们将网络安全组分配给子网。首先，选择虚拟网络：

	$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet

将网络安全组与子网相关联：

	Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name TestSubnet `
	    -NetworkSecurityGroup $nsg

最后，更新虚拟网络以使做出的更改生效：

	Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

##<a name="more-information-on-network-security-groups"></a> 有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。网络安全组提供许多出色的功能和粒度来控制资源访问。你可以深入了解此处的[创建网络安全组和 ACL 规则](/documentation/articles/virtual-networks-create-nsg-arm-ps/)。

你也可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。深入了解[使用模板创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-template/)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，则需要使用负载平衡器和网络地址转换 (NAT) 规则。例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。

## 后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。你可以从下列文章中，找到有关创建更详细环境的信息：

- [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)
- [什么是网络安全组 (NSG)？](/documentation/articles/virtual-networks-nsg/)

<!---HONumber=Mooncake_0704_2016-->