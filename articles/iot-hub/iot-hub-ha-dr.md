<properties
    pageTitle="Azure IoT 中心 HA 和 DR | Azure"
    description="介绍了Azure 和 IoT 中心功能，这些功能有助于构建带灾难恢复功能的 Azure IoT 高可用性解决方案。"
    services="iot-hub"
    documentationcenter=""
    author="fsautomata"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ae320e58-aa20-45b9-abdc-fa4faae8e6dd"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/16/2016"
    wacn.date="01/13/2017"
    ms.author="elioda" />

# IoT 中心高可用性和灾难恢复

作为一项 Azure 服务，IoT 中心在 Azure 区域级别使用冗余来提供高可用性 \(HA\)，而解决方案不需要执行任何额外的工作。 Azure 平台还包含相关功能，有助于构建带灾难恢复 \(DR\) 功能或跨区域可用性的解决方案。若要向设备或用户提供全局跨区域的高可用性，必须妥善设计并准备好解决方案，以便利用这些 Azure DR 功能。

## Azure IoT 中心 DR
除了区域内部的 HA，IoT 中心还实施了无需用户干预的灾难恢复故障转移机制。IoT 中心 DR 自行启动，其恢复时间目标 (RTO) 为 2 到 26 小时，恢复点目标 (RPO) 如下所示。

| 功能 | RPO |
| --- | --- |
| 注册表和通信操作的服务可用性 |可能丢失 CName |
| 标识注册表中的标识数据 |丢失 0-5 分钟的数据 |
| 设备到云的消息 |所有未读的消息都丢失 |
| 操作监视消息 |所有未读的消息都丢失 |
| 云到设备的消息 |丢失 0-5 分钟的数据 |
| 云到设备的反馈队列 |所有未读的消息都丢失 |

## IoT 中心区域故障转移
本文不讨论 IoT 解决方案中部署拓扑的完整处理方式。本文讨论了用于实现高可用性和灾难恢复的*区域故障转移*部署模型。

在区域故障转移模型中，该解决方案后端主要在一个数据中心位置运行，次要 IoT 中心和后端则部署在另一数据中心位置。应对的情况是主数据中心的 IoT 中心出现服务中断，或者设备到主数据中心之间断开网络连接。每当无法连接主要网关时，设备将使用辅助服务终结点。使用跨区域故障转移功能可改善解决方案的可用性，使其优于单个区域的高可用性。

从较高层面上讲，为了实现 IoT 中心的区域故障转移模型，需要做好以下准备：

* **辅助 IoT 中心和设备路由逻辑**：如果主要区域的服务中断，设备必须开始连接到次要区域。由于大多数服务状态感知的性质，解决方案管理员通常触发区域间的故障转移过程。若要实现新终结点与设备间的通信并掌控此过程，最好让其定期检查服务中是否存在当前活动的终结点。该监控服务可为 Web 应用程序，可使用 DNS 重定向技术进行复制和访问（例如使用 [Azure 流量管理器][Azure Traffic Manager]）。
* **标识注册表复制** - 若要进行使用，次要 IoT 中心必须包含所有可连接到解决方案的设备标识。解决方案应该保留设备标识的异地复制备份，并在切换设备的活动终结点之前将其上传到辅助 IoT 中心。IoT 中心的设备标识导出功能在此情景中很有用。有关详细信息，请参阅 [IoT 中心开发人员指南 - 标识注册表][IoT Hub developer guide - identity registry]。
* **合并逻辑** - 当主要区域再次可供使用时，所有在辅助站点中创建的状态和数据都必须迁移回到主要区域。此状态和数据主要与设备标识和应用程序元数据相关，必须与主要 IoT 中心合并，也可能要与主要区域中的其他应用程序特定存储合并。可使用幂等操作简化此步骤。幂等操作可最大程度降低事件的最终一致分布以及事件的重复项/失序传送所造成的副作用。此外，应用程序逻辑应该设计为能够容许潜在的不一致或“稍微”过期的状态。出现此情况的原因是系统需要额外的时间来根据恢复点目标 \(RPO\) 进行“修复”。

## 后续步骤

若要了解有关 Azure IoT 中心的详细信息，请参阅以下链接：

- [IoT 中心入门（教程）][lnk-get-started]
- [Azure IoT 中心是什么？][]


[防故障：弹性云体系结构指南]: https://msdn.microsoft.com/zh-cn/library/azure/jj853352.aspx
[Azure Traffic Manager]: /documentation/services/traffic-manager/
[IoT Hub Developer Guide - identity registry]: /documentation/articles/iot-hub-devguide-identity-registry/

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT 中心是什么？]: /documentation/articles/iot-hub-what-is-iot-hub/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->