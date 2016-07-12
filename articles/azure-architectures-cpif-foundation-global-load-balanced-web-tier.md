<properties 
   pageTitle="全局负载平衡的 Web 层（Azure 体系结构模式）" 
   description="全局负载平衡的 Web 层模式是 Foundation 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="03/25/2015"
   wacn.date="01/21/2016"/>

# 全局负载平衡的 Web 层（Azure 体系结构模式）

[云平台集成框架 (CPIF)](/documentation/articles/azure-architectures-cpif-overview/) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。

**全局负载平衡的 Web 层**模式是 **Foundation** 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛地介绍。

##  全局负载平衡的 Web 层

全局负载平衡的 Web 层设计模式详细说明了 Azure 功能，以及能跨地理边界提供可预测性能和高可用性的 Web 层服务所需的服务。

出于这种设计模式，Web 层定义为以隔离方式或作为多层 Web 应用的一部分，提供传统 HTTP/HTTPS 内容或应用程序服务的一个服务层。在此模式下，提供了本地区域内和跨区域的负载平衡的 Web 层。从计算的角度来看，可以通过 Azure 虚拟机、 Web 应用或这两者的组合提供这些服务。提供 Web 服务的虚拟机可使用任何受支持的 Microsoft Windows 或 Azure 中 Linux 分布式来宾操作系统来承载内容。


## 体系结构模式概述 

本文档介绍了出于可用性和冗余性的目的，在多个地理区域对 Web 服务或 Web 服务器内容提供访问的模式。关键的服务如下图所示，不考虑 Web 平台限制或 Web 服务本身内的开发方法。此模式有两种变体 – 其中一个承载虚拟机（使用 Azure 支持的操作系统和系列）上的 Web 内容或服务，另一个使用 Azure Web 应用。下图是相关服务的简单说明，并通过虚拟机示例展示了这些服务是如何作为本模式的一部分来使用的。

##  其他资源
[全局负载平衡的 Web 层 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

## 另请参阅
[CPIF 体系结构](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

[负载平衡的数据层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

[混合网络](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

[Azure 搜索层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

[批处理层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

<!---HONumber=71-->