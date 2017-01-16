<properties
    pageTitle="在 Azure 中打开 Linux VM 的端口和终结点 | Azure"
    description="了解如何使用 Azure Resource Manager 部署模型和 Azure CLI 1.0 在 Linux VM 上打开端口/创建终结点"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor="" />
<tags
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure-services"
    ms.date="10/27/2016"
    wacn.date="01/13/2017"
    ms.author="iainfou" />  


# 在 Azure 中打开 Linux VM 的端口和终结点
通过在子网或 VM 网络接口上创建网络筛选器可为 Azure 中的虚拟机 (VM) 打开端口或创建终结点。将这些筛选器（控制入站和出站流量）放在网络安全组中，并附加到将接收流量的资源。让我们在端口 80 上使用 Web 流量的常见示例。本文说明如何使用 Azure CLI 1.0 打开 VM 的端口。


## 用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#quick-commands)：用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0（预览版）](/documentation/articles/virtual-machines-linux-nsg-quickstart/)：用于资源管理部署模型的下一代 CLI


## <a name="quick-commands"></a> 快速命令
若要创建网络安全组和规则，需要安装 [Azure CLI 1.0](/documentation/articles/xplat-cli-install/) 并使用 Resource Manager 模式：

    azure config mode arm

在以下示例中，请将示例参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`myNetworkSecurityGroup` 和 `myVnet`。

正确输入自己的名称和位置来创建网络安全组。以下示例在 `ChinaNorth` 位置创建名为 `myNetworkSecurityGroup` 的网络安全组：

    azure network nsg create --resource-group myResourceGroup --location chinanorth \
        --name myNetworkSecurityGroup

添加规则以允许 HTTP 流量流向 Web 服务器（或者根据自己的情况（例如 SSH 访问或数据库连接）来调整此规则）。以下示例创建名为 `myNetworkSecurityGroupRule` 的规则，以便允许在端口 80 上传输 TCP 流量：

    azure network nsg rule create --resource-group myResourceGroup \
        --nsg-name myNetworkSecurityGroup --name myNetworkSecurityGroupRule \
        --protocol tcp --direction inbound --priority 1000 \
        --destination-port-range 80 --access allow

将网络安全组与 VM 的网络接口 (NIC) 相关联。以下示例将名为 `myNic` 的现有 NIC 与名为 `myNetworkSecurityGroup` 的网络安全组相关联：

    azure network nic set --resource-group myResourceGroup \
        --network-security-group-name myNetworkSecurityGroup --name myNic

或者，也可以将网络安全组与虚拟网络的子网相关联，而不是只与单个 VM 上的网络接口相关联。以下示例将 `myVnet` 虚拟网络中名为 `mySubnet` 的现有子网与名为 `myNetworkSecurityGroup` 的网络安全组相关联：

    azure network vnet subnet set --resource-group myResourceGroup \
        --network-security-group-name myNetworkSecurityGroup \
        --vnet-name myVnet --name mySubnet

## 有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。网络安全组提供许多出色的功能和粒度来控制资源的访问。请参阅[创建网络安全组和 ACL 规则](/documentation/articles/virtual-networks-create-nsg-arm-cli/)了解更多信息。

可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。深入了解[使用模板创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-template/)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，请使用负载均衡器和网络地址转换 (NAT) 规则。例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。了解如何[创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-cli/)。

## 后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。下列文章更介绍了有关创建更详细环境的信息：

* [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)
* [什么是网络安全组 (NSG)？](/documentation/articles/virtual-networks-nsg/)
* [Azure Resource Manager 中虚拟机规模集的概述](/documentation/articles/load-balancer-arm/)

<!---HONumber=Mooncake_0109_2017-->