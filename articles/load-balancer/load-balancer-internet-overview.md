
<properties 
   pageTitle="面向 Internet 的负载平衡器概述 | Azure "
   description="面向 Internet 的负载平衡器及其功能的概述。使用虚拟机和云服务的 Azure 的负载平衡器的工作原理。"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags 
   ms.service="load-balancer"
   ms.date="04/05/2016"
   wacn.date="08/29/2016" />


# 多个虚拟机或服务之间的面向 Internet 的负载平衡器

终结点的一种用途是配置 Azure Load Balancer，在多个虚拟机或服务之间分配特定类型的流量。例如，你可将 Web 请求流量负载分配到多个 Web 服务器或 Web 角色。

Azure Load Balancer 将传入流量的公用 IP 地址和端口号映射到虚拟机的专用 IP 地址和端口号，对于来自虚拟机的响应流量，则进行反向的映射。

>[AZURE.NOTE] Azure Load Balancer 将使用默认设置在多个虚拟机实例之间提供哈希分布网络流量（有关哈希分布的详细信息，请参阅[负载平衡器功能](/documentation/articles/load-balancer-overview#load-balancer-features)）。如果你要找的是会话相关性，请查看[负载平衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)。

对于包含 Web 角色或辅助角色实例的云服务，你可以在服务定义 (.csdef) 中定义一个公共终结点。
 
servicedefinition.csdef 文件将包含终结点配置，而当你有多个进行 Web 角色或辅助角色部署的角色实例时，将会针对该部署来设置负载平衡器。若要将实例添加到云部署，可更改服务配置文件 (.csfg) 中的实例计数。

下图显示了公用和专用 TCP 端口 443 的加密 Web 流量的负载平衡终结点，由三个虚拟机共享。这三个虚拟机位于一个负载平衡集中。


![公共负载平衡器示例](./media/load-balancer-internet-overview/IC727496.png))

当 Internet 客户端将网页请求发送到云服务的公用 IP 地址和 TCP 端口 443 时，Azure Load Balancer 会在负载平衡集中的三个虚拟机之间对这些请求进行基于哈希的负载平衡。若要获取有关负载平衡器算法的详细信息，可参阅[负载平衡器概述页](/documentation/articles/load-balancer-overview/#load-balancer-features)。


## 后续步骤

了解面向 Internet 的负载平衡器以后，你还可以阅读[内部负载平衡器](/documentation/articles/load-balancer-internal-overview/)，看哪个负载平衡器更适合你的云部署。

你还可以[开始创建面向 Internet 的负载平衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)，并配置适合特定负载平衡器网络流量行为的[分发模式](/documentation/articles/load-balancer-distribution-mode/)类型。

如果应用程序需要始终保持对负载平衡器后面的服务器的连接，你可以详细了解[负载平衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)。该文章将有助于你了解使用 Azure Load Balancer 时的空闲连接行为。


 

<!---HONumber=Mooncake_0822_2016-->