<properties
    pageTitle="什么是流量管理器 | Azure"
    description="本文将有助于你了解什么是流量管理器，以及流量管理器是否是适合你应用程序的流量路由选择"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date="11/07/2016"
    ms.author="sewhee"
/>  


# 流量管理器概述

使用 Azure 流量管理器可以控制用户流量在不同数据中心内的服务终结点上的分布。流量管理器支持的服务终结点包括 Azure VM、Web 应用和云服务。也可将流量管理器用于外部的非 Azure 终结点。

流量管理器根据[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)和终结点的运行状况，使用域名系统 (DNS) 将客户端请求定向到最合适的终结点。流量管理器提供多种流量路由方法来满足不同的应用程序需求、终结点运行状况[监视](/documentation/articles/traffic-manager-monitoring/)和自动故障转移。流量管理器能够灵活应对故障，包括整个 Azure 区域的故障。

## 流量管理器优点

流量管理器可帮助你：

- **提高关键应用程序的可用性**

    流量管理器可以监视终结点，在终结点发生故障时提供自动故障转移，实现应用程序的高可用性。

- **改进高性能应用程序的响应能力**

    在 Azure 中，可以运行位于世界各地的数据中心内的云服务或网站。流量管理器通过将流量定向到客户端网络延迟最低的终结点，改进应用程序的响应能力。

- **在不停机的情况下执行服务维护**

    无需停机即可在应用程序上执行计划内的维护操作。在维护过程中，流量管理器会将流量定向到备用的终结点。

- **合并本地应用程序和基于云的应用程序**

    流量管理器支持外部非 Azure 终结点，因此可以用于混合云部署和本地部署，包括“云爆发”、“云迁移”和“云故障转移”方案。

- **分发大型复杂部署的流量**

    使用[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)可以合并流量路由方法，创建复杂、灵活的规则来满足更大、更复杂部署的需求。

[AZURE.INCLUDE [load-balancer-compare-tm-ag-lb-include.md](../../includes/load-balancer-compare-tm-ag-lb-include.md)]

## 后续步骤

- 详细了解[流量管理器工作原理](/documentation/articles/traffic-manager-how-traffic-manager-works/)。

- 了解如何使用[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/)开发高可用性应用程序。

- 详细了解流量管理器支持的[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

- [创建流量管理器配置文件](/documentation/articles/traffic-manager-manage-profiles/)。

<!---HONumber=Mooncake_1031_2016-->