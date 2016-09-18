<properties
   pageTitle="打开 Linux VM 的端口或终结点 | Azure"
   description="了解如何打开端口/创建终结点，以便允许使用 Resource Manager 部署模型和 Azure CLI 从外部访问 Linux VM"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>  


<tags
	ms.service="virtual-machines-linux"
	ms.date="08/08/2016"
	wacn.date="09/12/2016"/>

# 打开端口和终结点
通过创建网络筛选器可在 Azure 中打开端口或创建终结点，以允许流量到达子网或虚拟机 (VM) 网络接口上的所选端口。将这些筛选器（控制入站和出站流量）放在网络安全组中，并附加到将接收流量的资源。让我们在端口 80 上使用 Web 流量的常见示例。

## 快速命令
若要创建网络安全组和规则，需要将 [Azure CLI](/documentation/articles/xplat-cli-install/) 置于 Resource Manager 模式 (`azure config mode arm`)。

适当地输入你自己的名称和位置，以创建网络安全组：

	azure network nsg create --resource-group TestRG --name TestNSG --location chinanorth

添加规则以允许 HTTP 流量流向 Web 服务器（或者根据自己的情况（例如 SSH 访问或数据库连接）来调整此规则）：

	azure network nsg rule create --protocol tcp --direction inbound --priority 1000 \
	    --destination-port-range 80 --access allow --resource-group TestRG --nsg-name TestNSG --name AllowHTTP

将网络安全组与 VM 网络接口相关联：

	azure network nic set --resource-group TestRG --name TestNIC --network-security-group-name TestNSG

或者，你也可以将网络安全组与虚拟网络的子网相关联，而不是只与单个 VM 上的网络接口相关联：

	azure network vnet subnet set --resource-group TestRG --name TestSubnet --network-security-group-name TestNSG

## <a name="more-information-on-network-security-groups"></a>有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。网络安全组提供许多出色的功能和粒度来控制资源的访问。[在此处详细了解如何创建网络安全组和 ACL 规则](/documentation/articles/virtual-networks-create-nsg-arm-cli/)。

可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。详细了解如何[使用模板创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-template/)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，则需要使用负载平衡器和网络地址转换 (NAT) 规则。例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。了解如何[创建面向 Internet 的负载平衡器](/documentation/articles/load-balancer-get-started-internet-arm-cli/)。

## 后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。你可以从下列文章中，找到有关创建更详细环境的信息：

- [Azure 资源管理器概述](/documentation/articles/resource-group-overview/)
- [什么是网络安全组 (NSG)？](/documentation/articles/virtual-networks-nsg/)
- [Azure Resource Manager 中负载平衡器的概述](/documentation/articles/load-balancer-arm/)

<!---HONumber=Mooncake_0905_2016-->