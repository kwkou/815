<properties 
   pageTitle="多站点数据层（Azure 体系结构模式）" 
   description="多站点数据层模式是 Foundation 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="03/25/2015"
   wacn.date="10/03/2015"/>

# 多站点数据层（Azure 体系结构模式）

[云平台集成框架 (CPIF)](/documentation/articles/azure-architectures-cpif-overview/) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。

**多站点数据层**模式是 **Foundation** 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。

## 多站点数据层

多站点数据层设计模式详细说明了 Azure 功能，以及能跨地理边界提供可预测性能和高可用性的数据层服务所需的服务。出于这种设计模式，数据层定义为以隔离方式或作为多层应用程序的一部分，提供传统数据平台服务的一个服务层。在此模式下，提供了本地区域内和跨区域数据层的负载平衡。

与 SQL Server 2012 共同引入的 AlwaysOn 可用性组具有高可用性和灾难恢复功能，在 Azure 基础结构服务上完全受支持。详细信息和 Azure 基础结构服务中对 AlwaysOn 可用性组的官方支持公告可以在《AlwaysOn 可用性组》一文中找到。

本文档系统地概述了在 Azure 中利用 SQL AlwaysOn 可用性组的一个多站点数据层。在其他 Azure 数据中心，用一个可选只读次要节点完成其他功能和灾难恢复。使用 Azure 中的 SQL AlwaysOn，提供可供 Web 或应用程序层使用的高可用性数据层。

虽然本文档重点介绍体系结构模式和实践，但您可以在官方教程中找到完整的部署指南，其简要描述了 Azure 中的 AlwaysOn 可用性组的配置，以及 AlwaysOn 可用性组侦听器的配置。

## 体系结构模式概述 

本文档介绍了出于可用性和冗余性的目的，在多个地理区域对 Microsoft SQL Server 的内容提供访问的模式。关键的服务如下图所示，不考虑将访问数据本身的应用程序或 Web 层。下图是相关服务的简单说明，及其作为此模式的一部分它们将如何使用。

以下关系图更详细地描述了每个主服务区域。
 
![资源和资源组边栏选项卡上的“标记”部件](./media/azure-architectures-cpif-foundation-multi-site-data-tier/overview.png)

##  其他资源
[负载平衡的数据层 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

## 另请参阅
[CPIF 体系结构](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

[全局负载平衡的 Web 层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

[混合网络](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

[Azure 搜索层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

[批处理层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

<!---HONumber=71-->