<properties 
   pageTitle="Azure 搜索层（Azure 体系结构模式）" 
   description="Azure 搜素层模式是 Foundation 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="multiple"
   ms.date="03/25/2015"
   wacn.date="12/17/2015"/>

# Azure 搜索层（Azure 体系结构模式）

[云平台集成框架 (CPIF)](/documentation/articles/azure-architectures-cpif-overview/) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。

**Azure 搜索层**模式是 **Foundation** 区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。

##  Azure 搜素层

Azure 搜素层设计模式详细说明了 Azure 功能，以及能跨地理边界提供可预测性能和高可用性搜索服务所需的服务，并提供了体系结构模式，用于使用 Azure 搜索来提供搜索解决方案。Azure 搜索是构建在 Azure 中的“将搜索作为服务”，使开发人员将搜索功能合入应用程序，而无需部署、管理或维护基础结构服务便可为应用程序提供此功能。此模式的目的是为不同情况和设计提供一个可重复使用的解决方案。

## 体系结构模式概述 

本文档描述了使用带有两种核心变体的 Azure 搜素的核心模式，以演示该服务的体系结构范围。核心模式包含了 Azure 搜索和相关 Azure 服务，旨在为创建端到端的设计提供指导。模式的变体，尤其对于共享服务和并发模式，也包括在本部分中，以根据不同的要求、服务等级协议 (SLA) 和其他特定的条件提供指导。

##  其他资源
[Azure 搜素层 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

## 另请参阅
[CPIF 体系结构](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

[全局负载平衡的 Web 层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

[负载平衡的数据层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

[混合网络](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

[批处理层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

<!---HONumber=Mooncake_1207_2015-->