<properties
    pageTitle="面向 Internet 的负载均衡器概述 | Azure"
    description="面向 Internet 的负载均衡器及其功能的概述。 使用虚拟机和云服务的 Azure 的负载均衡器的工作原理。"
    services="load-balancer"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="529b37aa-a45c-41d1-8877-fee8cc1fa375"
    ms.service="load-balancer"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/24/2016"
    wacn.date="05/08/2017"
    ms.author="kumud"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="ee3fc27f9a48c9312c6c3e495d0188da7aafd53f"
    ms.lasthandoff="04/28/2017" />

# <a name="internet-facing-load-balancer-overview"></a>面向 Internet 的负载均衡器概述

Azure 负载均衡器将传入流量的公用 IP 地址和端口号映射到虚拟机的专用 IP 地址和端口号，对于来自虚拟机的响应流量，则进行反向的映射。 借助负载均衡规则，可在多个虚拟机或服务之间分配特定类型的流量。 例如，可将 Web 请求流量负载分配到多个 Web 服务器或 Web 角色。

对于包含 Web 角色或辅助角色实例的云服务，可在服务定义文件 (.csdef) 中定义一个公共终结点。

*servicedefinition.csdef* 文件包含终结点配置，当有多个用于 Web 角色或辅助角色部署的角色实例时，将针对该部署设置负载均衡器。 若要将实例添加到云部署，可更改服务配置文件 (.csfg) 中的实例计数。

下图显示了公用和专用 TCP 端口 80 的加密 Web 流量的负载均衡终结点，由三个虚拟机共享。 三台虚拟机位于负载均衡集中。

![公共负载均衡器示例](./media/load-balancer-internet-overview/IC727496.png))

图 1 - Web 流量的负载均衡终结点

当 Internet 客户端将网页请求发送到 TCP 端口 80 上的云服务的公共 IP 地址时，Azure 负载均衡器会在负载均衡集中的三个虚拟机之间分配请求。 有关负载均衡器算法的详细信息，请参阅[负载均衡器概述页](/documentation/articles/load-balancer-overview/#load-balancer-features)。

默认情况下，Azure 负载均衡器在多个虚拟机实例之间平均分发网络流量。 还可以配置会话关联，有关详细信息，请参阅[负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)。

## <a name="next-steps"></a>后续步骤

了解[内部负载均衡器](/documentation/articles/load-balancer-internal-overview/)，以便更好地了解哪个负载均衡器更适合相关云部署。

还可以[开始创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)，并配置适合特定负载均衡器网络流量行为的[分发模式](/documentation/articles/load-balancer-distribution-mode/)类型。

如果应用程序需要始终保持对负载均衡器后面的服务器的连接，可详细了解[负载均衡器的空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)。 该文章将有助于你了解使用 Azure 负载均衡器时的空闲连接行为。

<!--Update_Description:update meta properties;wording update-->
