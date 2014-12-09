<properties linkid="service-bus-manage-messaging-entitites" urlDisplayName="Traffic Manager" pageTitle="Manage Service Bus Messaging Entities - Azure" metaKeywords="" description="Learn how to create and manage your Service Bus entities using the Azure Management Portal." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="service-bus" documentationCenter="" title="How to Manage Service Bus Messaging Entities" authors="sethm" solutions="" />

# 如何管理 Service Bus 消息传送实体

本主题介绍如何使用 [Azure 管理门户][Azure 管理门户]创建和管理 Service Bus 实体。你可以使用该门户创建新的服务命名空间或消息传送实体（队列、主题或订阅）。也可删除实体或更改实体的状态。

若要使用此功能和其他新的 Azure 功能，请注册以获取[免费预览版][免费预览版]。

## 目录

-   [如何：创建 Service Bus 实体][如何：创建 Service Bus 实体]
-   [如何：删除 Service Bus 实体][如何：删除 Service Bus 实体]
-   [如何：禁用或启用 Service Bus 实体][如何：禁用或启用 Service Bus 实体]
-   [其他资源][其他资源]

## <span id="create"></span></a>如何：创建 Service Bus 实体

Azure 管理门户支持两种创建 Service Bus 实体的方法：“快速创建”或“自定义创建”。

### 快速创建

利用快速创建，你只需一个简单的步骤即可创建 Service Bus 队列、主题或中继服务命名空间。按照下列步骤来创建 Service Bus 实体。

1.  登录到 [Azure（预览版）管理门户][Azure 管理门户]
2.  单击管理门户左下角的“新建”图标。
3.  单击“应用程序服务”图标，然后单击“Service Bus 队列”（主题或中继）。单击“快速创建”，并输入队列名称、区域和 Azure 订阅 ID。

    a. 如果这是你在选定区域中的第一个命名空间，则此门户会建议一个命名空间队列；[你的实体名称]-ns。你可以更改此值。

    b. 如果你在此区域中已拥有至少一个服务命名空间，则会自动选择一个命名空间。你可以更改该选定的命名空间。

4.  单击“创建新队列”（或主题）旁边的复选标记。
5.  创建队列或主题后，将显示以下消息：“已完成‘[队列名称]’的创建”。

    a. 如果你在此区域或此 Azure 订阅中没有任何命名空间，则会自动为你创建一个新的命名空间。在上述情况下，你将收到两条成功消息：一条针对命名空间的创建，另一条针对实体的创建。

    ![][0]

在左侧导航栏上单击 **Service Bus** 图标可获取命名空间的列表。你将找到刚创建的新命名空间。在列表中单击此命名空间。你将在该命名空间的下方看到刚创建的实体。

**注意** 你无法立即查看列出的命名空间。创建服务命名空间和更新门户界面需要几秒钟的时间。

**注意** 对中继使用“快速创建”不会创建新的中继终结点。而只会创建一个命名空间，可在其下方以编程方式创建中继终结点。有关详细信息，请参阅 [Service Bus 文档][Service Bus 文档]。

### 自定义创建

“自定义创建”是更详细的版本，它为你提供了可用于更改要创建的实体（队列或主题）的属性的默认值的设置。若要使用“自定义创建”创建主题或实体，请执行下列步骤：

1.  登录到 [Azure（预览版）管理门户][Azure 管理门户]
2.  单击管理门户左下角的“新建”。
3.  单击“应用程序服务”图标，然后单击“Service Bus 队列”（主题或中继）。然后单击“自定义创建”。
4.  在第一个对话框屏幕中，输入队列名称、区域和 Azure 订阅 ID。

    a. 如果这是你在选定区域中的第一个服务命名空间，则此门户会建议一个命名空间队列；[你的实体名称]-ns。你可以更改此值。

    b. 如果你在此区域中已拥有至少一个命名空间，则会自动选择一个命名空间。你可以更改该选定的命名空间。

5.  单击“下一步”以插入其余属性。

    ![][1]

6.  单击复选标记以创建队列。

    ![][2]

在左侧导航栏上单击 **Service Bus** 图标可获取服务命名空间的列表。你将找到刚创建的新命名空间。在列表中单击此命名空间。你将在该命名空间的下方看到刚创建的实体。

## <span id="delete"></span></a>如何：删除 Service Bus 实体

通过使用此门户，可以删除 Service Bus 消息传送实体。此操作适用于队列、主题和主题订阅。若要删除队列或主题，请执行以下操作：

1.  导航到服务命名空间列表视图，并单击你在其下方创建实体（队列或主题）的命名空间。
2.  单击页面底部的“删除”图标并确认删除操作。

    ![][3]

**注意** 实体删除是不可恢复的。一旦执行删除，便无法恢复。但是，你可以创建带同一名称的其他实体。

若要删除主题订阅，请执行以下操作：

1.  导航到命名空间列表视图，并单击你在其下方创建主题的命名空间。
2.  单击你在其下方创建订阅的主题。
3.  单击“订阅”选项卡并选择要删除的订阅。
4.  单击页面底部的“删除”图标并确认删除操作。

## <span id="disableenable"></span></a>如何：禁用或启用 Service Bus 实体

可以使用此门户更改 Service Bus 实体的状态。此操作适用于队列和主题。若要禁用或启用队列或主题，请执行以下操作：

1.  导航到服务命名空间列表视图，并单击你在其下方创建实体（队列或主题）的命名空间。
2.  单击页面底部的“禁用”（或“启用”）。

    ![][4]

## <span id="seealso"></span></a>其他资源

[Azure Service Bus][Azure Service Bus]

Azure 网站上的 [.NET 开发人员中心][.NET 开发人员中心]。

[创建使用 Service Bus 主题和订阅的应用程序][创建使用 Service Bus 主题和订阅的应用程序]

[队列、主题和订阅][队列、主题和订阅]

  [Azure 管理门户]: http://manage.windowsazure.cn
  [免费预览版]: https://account.windowsazure.cn/PreviewFeatures
  [如何：创建 Service Bus 实体]: #create
  [如何：删除 Service Bus 实体]: #delete
  [如何：禁用或启用 Service Bus 实体]: #disableenable
  [其他资源]: #seealso
  [0]: ./media/service-bus-manage-message-entities/QueueQuickCreate.png
  [Service Bus 文档]: http://www.windowsazure.cn/zh-cn/develop/net/how-to-guides/service-bus-relay/
  [1]: ./media/service-bus-manage-message-entities/AddQueue1.png
  [2]: ./media/service-bus-manage-message-entities/ConfigureQueue.png
  [3]: ./media/service-bus-manage-message-entities/DeleteEntity.png
  [4]: ./media/service-bus-manage-message-entities/DisableEnable.png
  [Azure Service Bus]: http://go.microsoft.com/fwlink/?LinkId=266834
  [.NET 开发人员中心]: http://go.microsoft.com/fwlink/?LinkID=262187
  [创建使用 Service Bus 主题和订阅的应用程序]: http://go.microsoft.com/fwlink/?LinkId=264293
  [队列、主题和订阅]: http://go.microsoft.com/fwlink/?LinkId=264291
