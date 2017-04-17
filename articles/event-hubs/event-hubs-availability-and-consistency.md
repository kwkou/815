<properties
    pageTitle="Azure 事件中心内的可用性和一致性 | Azure"
    description="如何使用分区为 Azure 事件中心提供最大程度的可用性和一致性。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="8f3637a1-bbd7-481e-be49-b3adf9510ba1"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/21/2017"
    wacn.date="04/17/2017"
    ms.author="sethm;jotaub"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="505aa470845d94b8a54cb41ea77b6548f7eb4477"
    ms.lasthandoff="04/07/2017" />

# <a name="availability-and-consistency-in-event-hubs"></a>事件中心内的可用性和一致性

## <a name="overview"></a>概述
Azure 事件中心使用[分区模型](/documentation/articles/event-hubs-what-is-event-hubs/#partitions)在单个事件中心中提高可用性和并行化。 例如，如果事件中心具有四个分区，并且其中一个分区要在负载均衡操作中从一台服务器移动到另一台服务器，则仍可以对其他三个分区进行发送和接收。 此外，通过更多分区可以让更多并发读取器处理数据，从而提高聚合吞吐量。 了解分布式系统中分区和排序的意义是解决方案设计的重要方面。

为了帮助说明排序与可用性之间的权衡，请参阅 [CAP 定理](https://en.wikipedia.org/wiki/CAP_theorem)（也称为 Brewer 的定理）。 此定理指出必须在一致性、可用性和分区容差之间进行选择。

Brewer 的定理按如下所示定义一致性和可用性：
* 分区容差 - 系统即使在出现分区故障时也能继续处理数据的数据处理能力。
* 可用性：非故障节点在合理时间内返回合理响应（没有错误或超时）。
* 一致性：读取保证针对给定客户端返回最新写入。

## <a name="partition-tolerance"></a>分区容差
事件中心在分区数据模型的基础上构建。 可以在设置过程中配置事件中心中的分区数，但是以后无法更改此值。 由于必须对事件中心使用分区，因此只需在应用程序的可用性和一致性方面进行决策。

## <a name="availability"></a>可用性
开始使用事件中心的最简单方法是使用默认行为。 如果创建新的 `EventHubClient` 对象并使用 `Send` 方法，则事件会自动在事件中心中的各个分区之间分布。 此行为可实现最大运行时间量。

对于需要最大运行时间的用例，此模型是首选模型。

## <a name="consistency"></a>一致性
在某些方案中，事件的排序可能十分重要。 例如，可能希望后端系统在删除命令之前处理更新命令。 在此情况下，可以对事件设置分区键，也可以使用 `PartitionSender` 对象仅将事件发送到特定分区。 这样做可确保从分区读取这些事件时，按顺序读取它们。

使用这种类型的配置，必须记住，如果发送到的特定分区不可用，则会收到错误响应。 作为对比，如果未与单个分区关联，则事件中心服务会将事件发送到下一个可用分区。

确保排序的一个可能解决方法（同时还最大限度地延长运行时间）是将事件作为事件处理应用程序的一部分进行聚合。 实现此目的的最简单方法是使用自定义序号属性标记事件。 下面是一个示例：

    // Get the latest sequence number from your application
    var sequenceNumber = GetNextSequenceNumber();
    // Create a new EventData object by encoding a string as a byte array
    var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));
    // Set a custom sequence number property
    data.Properties.Add("SequenceNumber", sequenceNumber);
    // Send single message async
    await eventHubClient.SendAsync(data);

前面的示例将事件发送到事件中心中的一个可用分区，并从应用程序设置对应序号。 此解决方案要求处理应用程序保持状态，不过会为发送者提供更可能可用的终结点。

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)

<!--Update_Description:update meta properties;wording update -->