<properties
   pageTitle="允许从外部访问 Linux VM | Azure"
   description="了解如何打开端口/创建终结点，以便允许使用 Resource Manager 部署模型和 Azure CLI 从外部访问 Linux VM"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/24/2016"
	wacn.date="07/11/2016"/>

# 允许从外部访问 VM
[AZURE.INCLUDE [virtual-machines-common-nsg-quickstart](../includes/virtual-machines-common-nsg-quickstart.md)]

## 快速命令
若要创建网络安全组和规则，需要将 [Azure CLI](/documentation/articles/xplat-cli-install/) 置于 Resource Manager 模式 (`azure config mode arm`)。

适当地输入你自己的名称和位置，以创建网络安全组：

	azure network nsg create --resource-group TestRG --name TestNSG --location chinanorth

添加规则以允许 HTTP 流量流向 Web 服务器（你可以根据自己的情况（例如 SSH 访问或数据库连接）来调整此规则）：

	azure network nsg rule create --protocol tcp --direction inbound --priority 1000 \
	    --destination-port-range 80 --access allow --resource-group TestRG --nsg-name TestNSG --name AllowHTTP

将网络安全组与 VM 网络接口相关联：

	azure network nic set --resource-group TestRG --name TestNIC --network-security-group-name TestNSG

或者，你也可以将网络安全组与虚拟网络的子网相关联，而不是只与单个 VM 上的网络接口相关联：

	azure network vnet subnet set --resource-group TestRG --name TestSubnet --network-security-group-name TestNSG

##<a name="more-information-on-network-security-groups"></a> 有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。网络安全组提供许多出色的功能和粒度来控制资源访问。你可以深入了解此处的[创建网络安全组和 ACL 规则](/documentation/articles/virtual-networks-create-nsg-arm-cli/)。

你也可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。深入了解[使用模板创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-template/)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，则需要使用负载平衡器和网络地址转换 (NAT) 规则。例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。

## 后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。你可以从下列文章中，找到有关创建更详细环境的信息：

- [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)
- [什么是网络安全组 (NSG)？](/documentation/articles/virtual-networks-nsg/)

<!---HONumber=Mooncake_0704_2016-->