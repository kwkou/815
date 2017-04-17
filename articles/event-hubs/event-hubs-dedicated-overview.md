<properties
    pageTitle="Azure 专用事件中心容量概述 | Azure"
    description="Azure 专用事件中心容量的概述。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/21/2017"
    wacn.date="04/17/2017"
    ms.author="sethm;babanisa"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="3a3db384a725bcb8565ac86b3b292782e8e9fbc7"
    ms.lasthandoff="04/07/2017" />

# <a name="overview-of-event-hubs-dedicated"></a>专用事件中心概述

*专用事件中心* 容量提供单租户部署。 完整规模的 Azure 事件中心可每秒传入超过两百万个事件或每秒高达 2 GB 的遥测，并具有完全持久的存储和次秒级的延迟。 通过在同一系统上实时进行批处理，它还可以实现集成的解决方案。 借助包含在产品中的事件中心存档功能，单个流可以同时支持实时管道和基于批处理的管道，从而降低解决方案的复杂性。

下表比较了事件中心的各个可用服务层。 与标准和基本事件中心内大部分功能的使用定价相比，专用事件中心产品每月价格是固定的。 专用层提供标准计划的功能，但具有企业规模容量，适合工作负荷需求较高的客户。 

| 功能 | 基本 | 标准 | 专用 |
| --- |:---:|:---:|:---:|
| 入口事件 | 按每百万个事件支付 | 按每百万个事件支付 | 已含 |
| 吞吐量单位（传入为 1 MB/秒，传出为 2 MB/秒） | 按每小时支付 | 按每小时支付 | 已含 |
| 消息大小 | 256 KB | 256 KB | 1 MB |
| 发布者策略 | 不适用 | 是 | 是 |
| 使用者组 | 1 — 默认值 | 20 | 20 |
| 消息重播 | 是 | 是 | 是 |
| 最大吞吐量单位 | 20 | 20（可灵活调整至 100） | 1 CU≈200 |
| 中转连接 | 包括 100 个 | 包括 1,000 个 | 包括 100,000 个 |
| 其他中转连接 | 不适用 | 是 | 是 |
| 消息保留期 | 包括 1 天 | 包括 1 天 | 包括最长 7 天 |
| 存档（预览版） | 不适用 | 按每小时支付 | 已含 |

## <a name="benefits-of-event-hubs-dedicated-capacity"></a>专用事件中心容量的优点

使用专用事件中心具有以下优点：

* 单个租户托管，免除来自其他租户的干扰。
* 与标准和基本事件中心的 256 KB 相比，消息大小增至 1 MB。
* 每次可重复性能。
* 有保障的容量，满足迸发需求。
* 容量单位 (CU) 可在 1 至 8 之间缩放 — 提供每秒高达两百万个入口事件。
  * CU 管理专用事件中心的规模，其中，每个 CU 可提供约等于 200 个吞吐量单位 (TU) 的容量。
* 零维护：由我们负责管理负载均衡、OS 更新、安全修补及分区。
* 固定的月度定价。

专用事件中心还去除了标准产品的部分吞吐量限制。 基本层和标准层的吞吐量单位允许每 TU 1000 个事件/秒或 1 MBps 的入口量以及两倍的出口量。 专用规模产品对入口和出口事件计数不设限制。 这些限制仅由购买的事件中心处理容量管理。

该服务面向最大的遥测用户，也可提供给具有企业协议的客户。

## <a name="how-to-onboard"></a>如何加入

专用事件中心平台通过企业协议提供给公众，它具有不同大小的 CU。 每个 CU 提供约等于 200 吞吐量计价单位。 通过添加或删除 CU，可以在一个月内随时扩展或缩小容量，满足自身需求。 专用计划独一无二，用户可从事件中心产品团队处获得适合自己的灵活部署，提供一种亲身实践操作体验。 

## <a name="next-steps"></a>后续步骤
请与 Microsoft 销售代表或 Microsoft 支持部门联系，获取有关专用事件中心容量的其他详细信息。 还可访问以下链接，了解有关事件中心的详细信息：

有关定价的详细信息，请访问以下链接：

- [专用事件中心定价](/pricing/details/event-hubs/)。 还可以联系 Microsoft 销售代表或 Microsoft 支持部门，获取有关专用事件中心容量的其他详细信息。
- [事件中心常见问题解答](/documentation/articles/event-hubs-faq/)中包含了定价信息并解答了一些有关事件中心的常见问题。


<!--Update_Description:update meta properties;wording update-->