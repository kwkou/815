<properties linkid="service-bus-monitor-messaging-entitites" urlDisplayName="Traffic Manager" pageTitle="Monitor Service Bus Messaging Entities - Azure" metaKeywords="" description="Learn how to monitor your Service Bus entities using the Azure Management Portal." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="service-bus" documentationCenter="" title="How to Monitor Service Bus Messaging Entities" authors="sethm" solutions="" />

# 如何监视 Service Bus 消息传送实体

本主题介绍如何使用 [Azure 管理门户][Azure 管理门户]管理和监视 Service Bus 实体。利用此门户，你可以全面了解队列和主题的状态。也可以监视其使用情况。

## 如何监视有关 Service Bus 队列的活动

若要监视 Service Bus 队列，请执行以下操作：

1.  登录到 [Azure（预览版）管理门户][Azure 管理门户]
2.  在左侧导航栏上单击 **Service Bus** 图标可获取服务命名空间的列表。
3.  单击包含要监视的队列的命名空间。
4.  在页面顶部的透视栏中，单击“队列”。
5.  单击要监视的队列的名称。队列仪表板将出现。
6.  你可以查看有关已创建的队列的活动。可以在多个时间范围内查看此信息。默认值为 1 小时，但你可以单击时间旁边的下拉列表来选择其他时间范围：过去 24 小时，过去 7 天或过去 30 天。对于 1 小时时间窗口，你可以查看精度低至以 5 分钟为度量单位的数据；对于 24 小时时间窗口，则可以查看精度为 1 小时的数据；对于 7 天和 30 天的窗口，可以查看精度为 1 天的数据。

对于任何队列，你可查看以下图表：

-   **传入消息**：在此时间间隔内已排队的消息数。
-   **传出消息**：在此时间间隔内已取消排队的消息数。
-   **长度**：此时间间隔结束时实体中的消息数。
-   **大小**：此时间间隔结束时该实体所使用的存储空间（以 MB 为单位）。

### 速览

仪表板上的“速览”将当前队列大小反映为“队列长度”。它还将显示队列或主题的其他属性。此信息每 10 秒刷新一次。

![][0]

## 如何监视有关 Service Bus 主题的活动

若要监视 Service Bus 主题，请执行以下操作：

1.  登录到 [Azure（预览版）管理门户][Azure 管理门户]
2.  在左侧导航栏上单击 **Service Bus** 图标可获取服务命名空间的列表。
3.  单击包含要监视的主题的命名空间。
4.  在页面顶部的透视栏中，单击“主题”。
5.  单击要监视的主题的名称。主题仪表板将出现。

主题仪表板与队列仪表板类似，只是使用指标不同。传出消息和长度将不会显示在主题仪表板中，因为对于某个主题的每个订阅而言，该信息都是不同的。利用“监视”选项卡，你可以为每个主题订阅添加使用指标（传出消息的数目和长度）。若要添加这些指标，请单击“监视”选项卡。然后，在页面底部单击“添加指标”，再从主题下方的订阅中进行选择。

![][1]

  [Azure 管理门户]: http://manage.windowsazure.cn
  [0]: ./media/service-bus-monitor-message-entities/QueueDashboard.png
  [1]: ./media/service-bus-monitor-message-entities/AddMetrics.png
