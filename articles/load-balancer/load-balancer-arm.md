<properties
   pageTitle="Azure Resource Manager 对负载平衡器预览版的支持 | Azure "
   description="对包含 Azure Resource Manager (ARM) 预览版的负载平衡器使用 PowerShell。对负载平衡器使用模板"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="load-balancer"
   ms.date="03/17/2016"
   wacn.date="08/29/2016" />


# Azure Resource Manager 对负载平衡器的支持 

Azure 资源管理器 (ARM) 是针对 Azure 中的服务的新管理框架。现在，你可以使用基于 Azure Resource Manager 的 API 和工具来管理 Azure Load Balancer。

## 概念

使用 ARM 时，Azure Load Balancer 包含以下子资源：

- 前端 IP 配置 – 一个负载平衡器可以包含一个或多个前端 IP 地址，也称为虚拟 IP (VIP)。这些 IP 地址充当流量的入口。

- 后端地址池 – 这是与负载要分配到的、与虚拟机网络接口卡 (NIC) 关联的 IP 地址。

- 负载平衡规则 – 规则属性将给定的前端 IP 和端口组合映射到一组后端 IP 地址和端口组合。只需定义一个负载平衡器资源，就能定义多个负载平衡规则，每个规则反映与 VM 关联的前端 IP 与端口以及后端 IP 与端口的组合。

- 探测 – 使用探测可以跟踪 VM 实例的运行状况。如果运行状况探测失败，VM 实例将自动从轮转列表中删除。

- 入站 NAT 规则 – NAT 规则定义流过前端 IP 并分配到后端 IP 的入站流量。


![](./media/load-balancer-arm/load-balancer-arm.png)



## 快速入门模板
Azure Resource Manager 可让你使用声明性模板预配应用程序。在单个模板中，可以部署多个服务及其依赖项。可在应用程序生命周期的每个阶段中使用相同模板重复部署应用程序

模板包含虚拟机、虚拟网络、可用性集、网络接口 (NIC)、存储帐户、负载平衡器、网络安全组和公开 IP。有了模板，可以使用一个可以签入和协作处理的简单文件来创建复杂应用程序所需的所有项目。

[详细了解模板](http://go.microsoft.com/fwlink/?LinkId=544798)

[详细了解网络资源](/documentation/articles/resource-groups-networking/)

可以在 [GitHub 存储库](https://github.com/Azure/azure-quickstart-templates)（托管社区生成的一组模板）中找到使用 Azure Load Balancer 的模板

模板示例：

- [负载平衡器中的 2 个 VM 和负载平衡规则](http://go.microsoft.com/fwlink/?LinkId=544799)

- [VNET 中包含负载平衡器和负载平衡规则的 2 个 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-internal-load-balancer)

- [负载平衡器中的 2 个 VM，在 LB 上配置 NAT 规则](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-natrules)


## 使用 PowerShell 或 CLI 设置 Azure Load Balancer

[Azure 网络 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt163510.aspx) 可用于创建负载平衡器。开始使用 ARM cmdlet 和 REST API

- [How to create a load balancer using Azure Resource Manager](/documentation/articles/load-balancer-get-started-internet-arm-ps/)（如何使用 Azure Resource Manager 创建负载平衡器）

- [Using the Azure CLI with Azure Resource Management](/documentation/articles/xplat-cli-azure-resource-manager/)（将 Azure CLI 与 Azure 资源管理配合使用）

- [Load Balancer REST APIs](https://msdn.microsoft.com/zh-cn/library/azure/mt163651.aspx)（负载平衡器 REST API）


## 后续步骤

你还可以[开始创建面向 Internet 的负载平衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)，并配置适合特定负载平衡器网络流量行为的[分发模式](/documentation/articles/load-balancer-distribution-mode/)类型。

如果应用程序需要始终保持对负载平衡器后面的服务器的连接，你可以详细了解[负载平衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)。该文章将有助于你了解使用 Azure Load Balancer 时的空闲连接行为。

<!---HONumber=Mooncake_0822_2016-->