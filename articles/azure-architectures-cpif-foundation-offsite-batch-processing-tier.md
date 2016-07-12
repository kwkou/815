<properties 
   pageTitle="非现场批处理层（Azure 体系结构模式）" 
   description="非现场批处理层模式是基础结构区域的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="03/25/2015"
   wacn.date="10/03/2015"/>

# 非现场批处理层（Azure 体系结构模式）

[云平台集成框架 (CPIF)](/documentation/articles/azure-architectures-cpif-overview/) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。

**非现场批处理层**模式是**基础结构**的一部分，在 CPIF 体系结构文档中对其进行了广泛的介绍。

##  非现场批处理层

非现场批处理层设计模式详细说明了 Azure 功能，以及提供容错和可扩展的后端数据处理所需的服务。这些服务作为 Azure 的云服务中的辅助角色而实现，当前可以将其部署到任何 Azure 数据中心。

批处理工作负荷是唯一的，因为它们通常提供很少或根本没有用户界面。这种类型的本地工作负荷的示例是在 Windows Server 上运行的 Windows 服务。在考虑云环境中的这种类型的工作负荷时，部署一个完整的服务器来运行工作负荷是一种浪费，真正需要的是计算、存储和网络连接。辅助角色是在 Azure 上执行此操作。

根据定义，在 Azure 中运行的批处理作业是连接到资源的工作负荷，提供了一些业务逻辑（计算），并提供一些输出。输入和输出资源由用户定义，范围包括平面文件、Azure blob 存储区的 blob、NoSQL 数据库或关系数据库。

业务逻辑在 Azure 辅助角色中执行，通常在 .NET 库中定义所需的业务逻辑。虽然部署一个辅助角色到 Azure 的操作简单，部署容错且可扩展的辅助角色需要设计，因为需要将如何在 Azure 中执行和维护服务考虑在内。此模式将详细介绍考虑这些需求的设计，并描述如何执行。

## 体系结构模式概述 

本文档介绍了利用 Azure 中的云服务所包含的辅助角色实例进行非现场批处理的模式。这种设计的关键组件如下所示。此图显示了实现容错所需的最小实例。可以部署其他实例以提高该服务的性能。此外，可启用自动扩展功能，通过时间或其他服务器指标协助扩展实例。

##  其他资源
[批处理层 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

## 另请参阅
[CPIF 体系结构](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

[全局负载平衡的 Web 层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

[负载平衡的数据层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

[混合网络](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

[Azure 搜索层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

<!---HONumber=71-->