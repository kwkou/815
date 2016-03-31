<properties 
   pageTitle="云平台集成框架 | Azure" 
   description="云平台集成框架提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中，其包含了用于 Azure 的体系结构模式" 
   services="" 
   documentationCenter="" 
   authors="arynes" 
   manager="fredhar" 
   editor=""/>

<tags
   ms.service="cloud-services"
   ms.date="03/25/2015"
   wacn.date="10/03/2015"/>


# 云平台集成框架（Azure 体系结构模式）

云平台集成框架 (CPIF) 提供工作负荷整合指南，将应用程序载入到 Microsoft 云解决方案中。

CPIF 介绍了组织、客户和合作伙伴应该如何设计和部署使用混合云平台及 Azure、System Center 和 Windows Server 管理功能的面向云的工作负荷。CPIF 域已被分解为以下功能：

![资源和资源组边栏选项卡上的“标记”部件](./media/azure-architecture-cpif-overview/overview.png)

##  CPIF 体系结构

公有和私有云环境提供常见的元素，以支持复杂工作负荷的运行。尽管这些体系结构在传统的本地物理和虚拟化环境中相对熟悉，在 Azure 中找到构造需要额外的规划，以合理化在公有云环境中发现的基础结构和平台功能。为了支持开发 Azure 中托管的应用程序或服务，需要一系列模式简要说明组成给定的工作负荷解决方案所需的各种组件。

这些体系结构模式属于以下类别：

- **基础结构**– Azure 是一种基础结构- 且平台作为服务解决方案由多个基础服务和功能组成。这些服务很大程度上可以分解为计算、存储和网络服务，但也有几个功能可能会超出这些定义。基础结构模式详细介绍了 Azure 的一个给定功能区域，需要为一个给定 Azure 订阅内托管的一个或多个解决方案提供给定的服务。 
- **Foundation** – 当在 Azure 中编写多层应用程序或服务时，几个组件必须结合使用来提供合适的托管环境。Foundation 模式从 Azure 编写一个或多个服务，以支持一个应用程序中给定的功能层。这可能需要使用上面概括的几个基础结构模式中描述的一个或多个组件。例如，多层应用程序的表示层需要 Azure 中计算、网络和存储功能才能发挥作用。Foundation 模式与其他模式组合作为给定的解决方案的一部分。
- **解决方案** – 解决方案模式由基础结构和/或 foundation 模式组成，表示正在开发的终端应用程序或服务。据设想，复杂的解决方案不会独立于其他模式而开发。相反，它们应使用上面概述的每个模式分类中定义的组件和接口。    

## Azure 体系结构模式概念

为了支持 Azure 中的解决方案体系结构开发，我们为常用场景提供了一系列模式指南。虽然这些方案指南将随着时间的推移而构造，设想的模式包括：

- 全局负载平衡的 Web 层 
- 负载平衡的数据层
- 批处理层
- 混合网络
- Azure 搜索 

##  其他资源
[CPIF 体系结构 (pdf)](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-bd1e434a)

## 另请参阅
[全局负载平衡的 Web 层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-2c3c663a)

[负载平衡的数据层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-dfb09e41)

[批处理层](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-0bc3f8b1)

[Azure 网络](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-5e401f38)

[Azure 搜索](https://gallery.technet.microsoft.com/Cloud-Platform-Integration-e581d65d)

<!---HONumber=71-->