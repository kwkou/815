<properties
   pageTitle="Service Fabric 服务的可伸缩性 | Microsoft Azure"
   description="介绍如何缩放 Service Fabric 服务"
   services="service-fabric"
   documentationCenter=".net"
   authors="appi101"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="01/20/2016"
   wacn.date="07/04/2016"/>

# 缩放 Service Fabric 应用程序
Azure Service Fabric 通过负载平衡服务、分区以及群集中所有节点上的副本，让可伸缩应用程序的生成更简单。这实现了资源利用率的最大化。

可以通过以下两种方式实现 Service Fabric 应用程序的高缩放性：

1. 在分区级别进行缩放

2. 在服务名称级别进行缩放

## 在分区级别进行缩放
Service Fabric 支持将单个服务分区为多个较小的分区。[分区概述](/documentation/articles/service-fabric-concepts-partitioning/)提供了有关受支持分区方案类型的信息。每个分区的副本分布在群集中的各个节点上。请考虑使用具有低键 0、高键 99 和 4 个分区的范围分区方案的服务。在包含三个节点的群集中，该服务可能如此处所示，按四个副本共享每个节点上的资源的方式进行布局：

![三节点式分区布局](./media/service-fabric-concepts-scalability/layout-three-nodes.png)

增加节点数目会允许 Service Fabric 通过移动一些副本到空节点来利用新节点上的资源。通过将节点数目增加到四个，现在服务在（不同分区的）每个节点上运行三个副本，从而获得更高的资源利用率和性能。

![四节点式分区布局](./media/service-fabric-concepts-scalability/layout-four-nodes.png)

## 在服务名称级别进行缩放
服务实例是应用程序名称和服务类型名称的一个特定实例（请参阅 [Service Fabric 应用程序生命周期](/documentation/articles/service-fabric-application-lifecycle/)）。在创建服务的过程中，指定要使用的分区方案（请参阅 [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)）。

第一个缩放级别是按服务名称进行。随着旧服务实例变得繁忙，你可以创建新的服务实例，并可赋予不同的分区级别。这允许新服务使用者使用较不繁忙的服务实例而非较忙碌的实例。

增加容量以及增大或减小分区计数的一种方法是使用新的分区方案创建新的服务实例。不过这会增加复杂性，因为任何使用客户端需要知道何时以及如何使用命名不同的服务。

### 示例方案：嵌入日期
一种可行的方案是将日期信息用作服务名称的一部分。例如，可以使用如下服务实例：2013 年加入的所有客户具有一个特定名称，2014 年加入的客户具有另一个名称。此命名方案允许根据日期以编程方式增加名称（随着 2014 年的临近，可按需创建 2014 年的服务实例）。

但是，此方法基于使用超出 Service Fabric 知识范围外的应用程序特定命名信息的客户端。

- *使用命名约定*：2013 年你的应用程序上线时，你创建了一个名为 fabric:/app/service2013 的服务。临近 2013 年第二季度时，你创建了另一个名为 fabric:/app/service2014 的服务。这两个服务同为一种服务类型。在这种方法中，你的客户端需要使用基于年度构建适当服务名称的逻辑。

- *使用查找服务*：另一种模式是提供辅助的“查找服务”，此服务可为所需密钥提供服务的名称。然后可以通过查找服务来创建新的服务实例。查找服务本身不会保留任何应用程序数据，它仅保留有关自己创建的服务名称的数据。因此，对于以上基于年度的示例，客户端将首先联系查找服务以找到处理给定年度数据的服务的名称，然后使用该服务名称来执行实际操作。首次查找的结果可进行缓存。

## 后续步骤

有关 Service Fabric 概念的详细信息，请参阅以下内容：

- [Service Fabric 服务的可用性](/documentation/articles/service-fabric-availability-services/)

- [Service Fabric 服务分区](/documentation/articles/service-fabric-concepts-partitioning/)

- [定义和管理状态](/documentation/articles/service-fabric-concepts-state/)
 

<!---HONumber=Mooncake_0307_2016-->