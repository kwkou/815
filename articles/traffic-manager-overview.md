<properties 
   pageTitle="什么是流量管理器 | Azure"
   description="本文将有助于你了解什么是流量管理器，以及流量管理器是否是适合你应用程序的流量路由选择"
   services="traffic-manager"
   documentationCenter=""
   authors="jtuliani"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
	ms.date="06/09/2016"
	wacn.date="07/04/2016"/>

# 什么是流量管理器？

使用 Azure 流量管理器，你可以控制用户流量的分布，根据需要将用户流量分布到在全球不同数据中心运行的服务终结点。

流量管理器支持的服务终结点包括 Azure VM、Web Apps 和云服务。也可将流量管理器用于外部的非 Azure 终结点。

流量管理器在工作时，会使用域名系统 (DNS) 根据配置的流量路由方法和当前对终结点运行状况的检查结果将最终用户请求定向到最适当的终结点。然后，客户端即可直接连接到适当的服务终结点。

流量管理器支持根据不同的应用程序需求使用[一系列流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。流量管理器提供[终结点运行状况检查和终结点自动故障转移](/documentation/articles/traffic-manager-monitoring/)功能，允许你构建高可用性应用程序来应对各种故障，包括整个 Azure 区域发生的故障。

## 流量管理器优点

流量管理器可帮助你：

- **提高关键应用程序的可用性** - 使用流量管理器，你可以监视 Azure 中的终结点并在某个终结点出现故障时进行自动故障转移，从而确保关键应用程序的高可用性。
- **提高高性能应用程序的响应能力** - Azure 允许你在世界各地的数据中心运行云服务或网站。通过将最终用户定向到客户端的网络延迟最低的终结点，流量管理器可以改进你的应用程序的响应能力。
- **在不停机的情况下执行升级和服务维护** - 你可以在不造成最终用户停机的情况下，无缝地对应用程序执行升级和其他计划内维护操作，只需在进行维护时使用流量管理器将流量定向到替代终结点即可。
- **组合使用本地应用程序和基于云的应用程序** - 流量管理器支持外部非 Azure 终结点，因此可以用于混合云部署和本地部署，包括“云爆发”、“云迁移”和“云故障转移”方案。
- **针对大型复杂部署分配流量** - 可以通过[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)组合使用各种流量路由方法来创建复杂且灵活的流量路由配置，以满足更大型、更复杂部署的需求。 

[AZURE.INCLUDE [load-balancer-compare-tm-ag-lb-include.md](../includes/load-balancer-compare-tm-ag-lb-include.md)]

## 后续步骤

- 详细了解[流量管理器工作原理](/documentation/articles/traffic-manager-how-traffic-manager-works/)。

- 了解如何使用[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/)开发高可用性应用程序。

- 详细了解流量管理器支持的[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

- [创建流量管理器配置文件](/documentation/articles/traffic-manager-manage-profiles/)。
 

<!---HONumber=Mooncake_0627_2016-->