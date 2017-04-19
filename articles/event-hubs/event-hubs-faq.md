<properties
    pageTitle="Azure 事件中心常见问题解答 | Azure"
    description="Azure 事件中心常见问题 (FAQ)"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="bfa10984-eb22-4671-861a-f377a90d9372"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/13/2017"
    wacn.date="04/17/2017"
    ms.author="sethm;jotaub;shvija"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="027069e7bee618db5418c1fc4331dc9221a6eb0c"
    ms.lasthandoff="04/07/2017" />

# <a name="event-hubs-frequently-asked-questions"></a>事件中心常见问题

## <a name="general"></a>常规

### <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>事件中心基本和标准这两种服务层有什么不同？
Azure 事件中心标准层提供的功能超出了基本层中提供的功能。 以下功能是标准层附带的：

* 更长的事件保留期
* 其他中转连接，对于超出包含的数量的部分收取超额费用
* 多于单个使用者组
* [存档](https://docs.microsoft.com/azure/event-hubs/event-hubs-archive-overview)

有关定价的更多详细信息（包括专用事件中心），请参阅[事件中心定价详细信息](/pricing/details/event-hubs/)。

### <a name="what-are-event-hubs-throughput-units"></a>什么是事件中心吞吐量单元？
通过 Azure 门户预览版或事件中心 Resource Manager 模板明确选择事件中心吞吐量单位。 吞吐量单位适用于事件中心命名空间中的所有事件中心，每个吞吐量单位为命名空间赋予以下功能：

* 高达每秒 1 MB 的入口事件（发送到事件中心的事件），但每秒不超过 1000 个入口事件、管理操作或控制 API 调用。
* 高达每秒 2 MB 的出口事件（从事件中心耗用的事件）。
* 高达 84 GB 的事件存储空间（对于默认为 24 小时的保留期来说是很充足的）。

事件中心吞吐量单元根据在指定的某个小时内选择的最大单元数量按小时计费。

### <a name="how-are-event-hubs-throughput-unit-limits-enforced"></a>怎样强制实施事件中心吞吐量单元？

如果命名空间中所有事件中心间的总出口吞吐量或总出口事件率超过了聚合吞吐量单位限额，接收者会受到限制，并会收到指明已超出出口配额的错误信息。 入口和出口配额是分开强制实施的，这样，发送者无法使事件耗用速度减慢，接收者也无法阻止事件发送到事件中心。

### <a name="is-there-a-limit-on-the-number-of-throughput-units-that-can-be-selected"></a>可选择的吞吐量单元的数量有限制吗？
默认配额为每个命名空间 20 个吞吐量单元。 可以通过填写支持票证来要求更大的吞吐量单元配额。 超出 20 个吞吐量单元这一限制时，将以 20 个和 100 个吞吐量单元的形式提供捆绑包。 请注意，使用 20 个以上的吞吐量单元会失去在不填写支持票证的情况下更改吞吐量单元数的能力。

### <a name="can-i-use-a-single-amqp-connection-to-send-and-receive-from-multiple-event-hubs"></a>是否可以使用单个 AMQP 连接来与多个事件中心相互收发数据？
可以，但前提是所有事件中心都在同一个命名空间中。

### <a name="what-is-the-maximum-retention-period-for-events"></a>事件的最长保留期有多久？
事件中心标准目前支持的最长保留期为 7 天。 请注意，事件中心并不是永久性的数据存储区。 大于 24 小时的保留期适用于将事件流重播到相同系统中的情形；例如，为了基于现有数据来培训或验证新计算机学习模型。 如果需要将消息保留 7 天以上，那么启用事件中心的[存档](https://docs.microsoft.com/azure/event-hubs/event-hubs-archive-overview)功能可将数据从事件中心拉取到你选择的存储。 启用存档功能会基于你购买的吞吐量单位进行计价。

### <a name="where-is-azure-event-hubs-available"></a>Azure 事件中心在哪些区域可用？
在所有支持的 Azure 区域中都可使用 Azure 事件中心。

## <a name="best-practices"></a>最佳实践

### <a name="how-many-partitions-do-i-need"></a>需要多少分区？
请记住，事件中心的分区数在设置后无法修改。 鉴于这一点，请务必在开始之前考虑需要多少分区。 

事件中心设计用于允许每个用户组使用单个分区读取器。 在大多数用例中，四个分区的默认设置就足够了。 如果你希望扩展事件处理，则可以考虑添加其他分区。 对分区没有特定的吞吐量限制，但是命名空间中的聚合吞吐量受吞吐量单位数限制。 增加命名空间中吞吐量单位的数量时，可能需要添加额外分区来允许并发读取器实现其自身的最大吞吐量。

但是，如果你有一个模型，其中应用程序具有到特定分区的关联性，则增加分区数可能对你没有任何益处。 有关详细信息，请参阅[可用性和一致性](/documentation/articles/event-hubs-availability-and-consistency/)。
## <a name="pricing"></a>定价

### <a name="where-can-i-find-more-pricing-information"></a>我可以在哪里找到更多定价信息？
有关事件中心定价的完整信息，请参阅[事件中心定价详细信息](/pricing/details/event-hubs/)。

### <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>保留事件中心事件超过 24 小时的话，要收取费用吗？
事件中心标准层允许消息保留期长于 24 小时，最长以 7 天为限。 如果所存储事件总量的大小超过所选吞吐量单位数量的存储限制（每个吞吐量单位 84 GB），超出限制的部分会按公布的 Azure Blob 存储区费率收费。 每个吞吐量单元的存储限制包括 24 小时（默认值）的保留期期间的所有存储成本，即使吞吐量单元已经用到了最大入口限制。

### <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>怎样对事件中心存储区大小进行计算和收费？
所有存储事件的总大小，包括所有事件中心中用于事件标头或磁盘存储结构上的任何内部开销，在一整天当中都会被测量。 在一天结束时，计算存储区大小峰值。 每天的存储限制根据在当天所选择的吞吐量单元的最小数量（每个吞吐量单元提供 84 GB 的限制）来计算。 如果总大小超过计算出的每日存储限额，超出的存储量会采用 Azure Blob 存储费率（按**本地冗余存储**费率）来计费。

### <a name="how-are-event-hubs-ingress-events-calculated"></a>事件中心入口事件是怎样计算的？
发送到事件中心的每个事件都计为一条可计费消息。 *入口事件* 定义为小于等于 64 KB 的数据单位。 任何小于等于 64 KB 的事件均被视为一个计费事件。 如果该事件大于 64 KB，则根据事件大小按 64 KB 的倍数来计算计费事件的数量。 例如，发送到事件中心的 8 KB 事件按一个事件计费，而发件送到事中心的 96 KB 的消息则按两个事件计费。

从事件中心耗用的事件，以及管理操作和控制调用（例如检查点），不统计为计费入口事件，但会累计，上限为吞吐量单元限额。

### <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>中转连接费用是否适用于事件中心？
连接费用只在使用 AMQP 协议时适用。 使用 HTTP 发送事件没有连接费用，无论发送系统或设备的数量是多少。 如果计划使用 AMQP（例如，为了实现更高效的事件流式传输，或者为了对 IoT 命令和控制方案启用双向通信），请参阅[事件中心定价信息](/pricing/details/event-hubs/)页，了解有关每个服务层中包括多少连接的详细信息。

### <a name="how-is-event-hubs-archive-billed"></a>事件中心存档如何计费？
如果命名空间中的任何事件中心启用存档功能，即可启用存档功能。 存档按所购买的吞吐量单位进行小时计费。 吞吐量单位计数增加或减少时，事件中心存档计费将在整个的小时增量中反映这些改变。
有关事件中心存档计费的详细信息，请参阅[事件中心定价信息](/pricing/details/event-hubs/)页。

### <a name="will-i-be-billed-for-the-storage-account-i-select-for-event-hubs-archive"></a>是否要为事件中心存档所选择的存储帐户付费？
存档使用在事件中心启用存档时提供的存储帐户。 因为这是你的存储帐户，任何针对此存储帐户的费用更改都将计入你的 Azure 订阅。

## <a name="quotas"></a>配额

### <a name="are-there-any-quotas-associated-with-event-hubs"></a>是否有与事件中心关联的配额？
如需所有事件中心配额的列表，请参阅[配额](/documentation/articles/event-hubs-quotas/)。

## <a name="troubleshooting"></a>故障排除

### <a name="what-are-some-of-the-exceptions-generated-by-event-hubs-and-their-suggested-actions"></a>事件中心生成的异常有哪些，建议采取什么操作？
有关可能的事件中心异常的列表，请参阅[异常概述](/documentation/articles/event-hubs-messaging-exceptions/)。

<!--
### <a name="diagnostic-logs"></a>诊断日志
事件中心支持两种类型的[诊断日志](/documentation/articles/event-hubs-diagnostic-logs/)：存档错误日志和操作日志；这两种日志都用 json 表示并可以通过 Azure 门户预览版启用。
-->
<!--event-hubs-diagnostic-logs is not supported on ACN -->

### <a name="support-and-sla"></a>支持和 SLA
事件中心的技术支持可通过 [社区论坛](https://social.msdn.microsoft.com/forums/azure/home)获得。 计费和订阅管理支持免费提供。

若要详细了解我们的 SLA，请参阅[服务级别协议](/support/legal/sla/)页面。

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)



<!--Update_Description:update meta properties;update wording;update link reference-->