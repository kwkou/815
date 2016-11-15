
<properties
   pageTitle="面向 Internet 的负载均衡器概述 | Azure "
   description="面向 Internet 的负载均衡器及其功能的概述。使用虚拟机和云服务的 Azure 的负载均衡器的工作原理。"
   services="load-balancer"
   documentationCenter="na"
   authors="sdwheeler"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/25/2016"
   wacn.date="10/10/2016" />


# 面向 Internet 的负载均衡器概述

Azure Load Balancer 将传入流量的公用 IP 地址和端口号映射到虚拟机的专用 IP 地址和端口号，对于来自虚拟机的响应流量，则进行反向的映射。借助负载均衡规则，可在多个虚拟机或服务之间分配特定类型的流量。例如，你可将 Web 请求流量负载分配到多个 Web 服务器或 Web 角色。


>[AZURE.NOTE] Azure Load Balancer 将使用默认设置在多个虚拟机实例之间提供哈希分布网络流量（有关哈希分布的详细信息，请参阅[负载均衡器功能](/documentation/articles/load-balancer-overview)）。如果你要找的是会话相关性，请查看[负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)。

对于包含 Web 角色或辅助角色实例的云服务，可在服务定义文件 (.csdef) 中定义一个公共终结点。

servicedefinition.csdef 文件将包含终结点配置，而当你有多个进行 Web 角色或辅助角色部署的角色实例时，将会针对该部署来设置负载均衡器。若要将实例添加到云部署，可更改服务配置文件 (.csfg) 中的实例计数。

下图显示了公用和专用 TCP 端口 443 的加密 Web 流量的负载均衡终结点，由三个虚拟机共享。这三个虚拟机位于一个负载均衡集中。

![公共负载均衡器示例](./media/load-balancer-internet-overview/IC727496.png))

当 Internet 客户端将网页请求发送到 TCP 端口 443 上的云服务的公共 IP 地址时，Azure Load Balancer 会在负载均衡集中的三个虚拟机之间分发请求。若要获取有关负载均衡器算法的详细信息，可参阅[负载均衡器概述页](/documentation/articles/load-balancer-overview/)。

## 后续步骤

了解面向 Internet 的负载均衡器以后，你还可以阅读[内部负载均衡器](/documentation/articles/load-balancer-internal-overview/)，更好地了解哪个负载均衡器更适合相关云部署。

还可以[开始创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)，并配置适合特定负载均衡器网络流量行为的[分发模式](/documentation/articles/load-balancer-distribution-mode/)类型。

如果应用程序需要始终保持对负载均衡器后面的服务器的连接，你可以详细了解[负载均衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)。该文章将有助于你了解使用 Azure Load Balancer 时的空闲连接行为。

<!---HONumber=Mooncake_0926_2016-->