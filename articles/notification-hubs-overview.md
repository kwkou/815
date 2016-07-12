<properties
	pageTitle="Azure 通知中心"
	description="了解如何在 Azure 中使用通知中心。代码示例是使用 .NET API 通过 C# 编写的。"
	authors="wesmc7777"
	manager="dwrede"
	editor=""
	services="notification-hubs"
	documentationCenter=""/>
<tags 
	ms.service="notification-hubs" 
	ms.date="02/11/2016"
	wacn.date="05/18/2016"/>


#Azure 通知中心

##概述

Azure 通知中心提供易用的基础结构，使你能够从任何后端（云中或本地）向任何移动平台发送移动推送通知。

借助通知中心，你可以轻松发送跨平台的个性化推送通知，提取不同平台通知系统 (PNS) 的详细信息。只需 API 调用一次，就可以将通知发送到单个用户或包含数百万用户的整个受众群的所有设备上。

你可以将通知中心用于企业和使用者方案。例如：

- 向数百万用户发送突发新闻通知且只会出现短暂的延迟（通知中心为所有 Windows 和 Windows Phone 设备上预安装的“必应”应用程序提供支持）。

- 向用户群发送基于位置的优惠券。

- 向体育/财经/游戏应用程序的用户或组发送事件通知。

- 向用户通知企业事件，如新消息/电子邮件和销售商机。
- 发送 Multi-Factor Authentication 所需的一次性密码。



##什么是推送通知？

智能手机和平板电脑能够在事件发生时“通知”用户。这些通知有多种多样的形式。

在 Windows 应用商店和 Windows Phone 应用程序中，通知可以采用 _toast_ 形式：显示一个无模式窗口，并伴有声音，以提醒有新的通知。支持的其他通知类型包括“磁贴”、“raw”和“提醒”通知。有关 Windows 设备支持的通知类型的详细信息，请参阅[磁贴、提醒和通知](http://msdn.microsoft.com/library/windows/apps/hh779725.aspx)。

在 Apple iOS 设备上，推送时将以类似的方式提醒用户：出现一个请求用户查看或关闭通知的对话框。单击“查看”会打开收到消息的应用程序。有关 iOS 通知的详细信息，请参阅 [iOS 通知](http://go.microsoft.com/fwlink/?LinkId=615245)。

推送通知可帮助移动设备显示最新信息，同时保持高能效。后端系统可以将通知发送给移动设备，即使设备上的相关应用处于非活动状态。推送通知是使用者应用程序的重要组件，它们可用于增加应用程序的关注度和使用率。通知对企业也很有用，最新信息会增加员工对业务活动的响应能力。

下面是移动应用场景的一些具体示例：

1.  使用当前财务信息更新 Windows 8 或 Windows Phone 上的磁贴。
2.  在基于工作流的企业应用程序中，使用 toast 通知用户已向其分配某个工作项。
3.  在 CRM 应用程序（例如 Microsoft Dynamics CRM）中，显示指明当前潜在销售顾客数的提醒。

##推送通知的工作原理

推送通知通过称为_平台通知系统_ (PNS) 的特定于平台的基础结构进行传送。PNS 提供的功能有限（即，不支持广播和个性化设置），并且没有常见界面。例如，若要将通知发送给 Windows 应用商店应用程序，开发人员必须与 WNS（Windows 通知服务）联系，若要将通知发送给 iOS 设备，该开发人员必须与 APNS（Apple 推送通知服务）联系并第二次发送该消息。Azure 通知中心提供通用接口以及其他功能，以帮助支持跨平台推送通知。

但概括地说，所有平台通知系统都遵循相同的模式：

1.  客户端应用程序与 PNS 联系以检索其_句柄_。句柄类型取决于系统。对于 WNS，它是 URI 或“通知通道”。 对于 APNS，它是令牌。
2.  客户端应用将此句柄存储在应用_后端_中以供日后使用。对于 WNS，后端通常是云服务。对于 Apple，该系统称为_提供程序_。
3.  为发送推送通知，应用程序后端使用句柄与 PNS 联系以定位到特定客户端应用程序实例。
4.  PNS 将通知转发到句柄所指定的设备。
![][0]

##推送通知的难点

虽然这些系统非常强大，但它们仍为应用程序开发人员留下很多实现常见推送通知方案的工作，例如将推送通知广播或发送给用户。

推送通知是云服务中最受期待的移动应用程序功能之一。原因是使它们运行所需的基础结构相当复杂，并且该基础结构通常与应用程序的主业务逻辑无关。生成按需推送基础结构面临以下一些问题：

- **平台依赖性。** 若要将通知发送到位于不同平台的设备，必须在后端中为多个界面编码。不但具体细节不同，而且通知的呈现方式（磁贴、toast 或提醒）也与平台相关。这些差异可能会迫使开发人员编写复杂且难以维护的后端代码。

- **扩展。** 扩展此基础结构涉及两方面：
	+ 根据 PNS 规定，每次启动应用程序时都必须刷新设备令牌。这会产生大量流量（和随之发生的数据库访问）以保持设备令牌最新。当设备数量增长（可能达到数百万台）时，创建并维护此基础结构的成本不可忽视。

	+ 大多数 PNS 不支持广播到多台设备。因此，广播到数百万台设备将调用 PNS 数百万次。能够扩展这些请求很重要，因为应用程序开发人员通常要缩短总延迟（例如，接收消息的最后一台设备不应在发送通知 30 分钟后才收到通知，因为在许多情况下，这将违背使用推送通知的目的）。
- **路由。** PNS 提供了一种将消息发送到设备的方法。但是，在大多数应用程序中，通知将定向到用户和/或兴趣组（例如，分配到某个客户帐户的所有员工）。因此，为了将通知路由到正确的设备，应用程序后端必须维护将兴趣组与设备令牌相关联的注册表。此开销增加了应用程序的上市总时间和维护成本。

##为何使用通知中心？

通知中心消除了复杂性：你不必应对推送通知的难题，只需使用通知中心即可。通知中心使用完整的多平台、向外扩展的推送通知基础结构，并显著减少了在应用程序后端运行的特定于推送的代码。通知中心可实现推送基础结构的所有功能。设备只负责注册 PNS 句柄，而后端负责将独立于平台的消息发送给用户或兴趣组，如下图中所示：

![][1]


通知中心提供了随时可用的推送通知基础结构，该基础结构具有以下优势：

- **多个平台。**
	+  支持所有主要移动平台。通知中心可将推送通知发送到 Windows 应用商店、iOS、Android 和 Windows Phone 应用程序。

	+  通知中心提供了用于将通知发送到所有受支持平台的常见界面。不需要特定于平台的协议。应用程序后端可以采用特定于平台的格式或独立于平台的格式发送通知。应用程序只与通知中心通信。

	+  设备句柄管理。通知中心维护 PNS 的句柄注册表和反馈。

- **适用于任何后端**：云或本地后端、.NET、PHP、Java、Node 等。

- **扩展。** 通知中心可扩展到数百万台设备，并且无需重新架构或分片。


- **丰富的传送模式集**：

	- *广播*：只需单次 API 调用，即可向数百万台设备几乎同时地进行广播。

	- *单播/多播*：推送到表示单个用户的标记，包括其所有设备；或更广泛的组；例如，单独的窗体因素（平板电脑与手机）。

	- *分段*：推送到由标记表达式定义的复杂段（例如，纽约市属于美国北部人群的设备）。

	每台设备在将其句柄发送到通知中心时，都可以指定一个或多个_标记_。有关[标记](http://msdn.microsoft.com/library/azure/dn530749.aspx)的详细信息。无需预设置或处理标记。标记提供了一种用于将通知发送给用户或兴趣组的简单方法。由于标记可以包含任何特定于应用程序的标识符（例如用户 ID 或组 ID），因此，使用它们将使应用程序后端不必再存储和管理设备句柄。

- **个性化**：每个设备都可以有一个或多个模板，以实现按设备本地化和个性化设置，而不会影响后端代码。

- **安全性**：共享访问机密 (SAS) 或联合身份验证。

- **丰富的遥测功能**：可以在门户中或以编程方式使用。


##与 App Service Mobile Apps 集成

为了在 Azure 服务之间促成完美且统一的体验， App Service Mobile Apps 原生支持使用通知中心来推送通知。App Service Mobile Apps 提供面向企业开发人员和系统集成商的高度可缩放、全局可用的移动应用程序平台，该平台向移动开发人员提供一组丰富的功能。

Mobile Apps 开发人员可以借助以下工作流来利用通知中心：

1. 检索设备 PNS 句柄
2. 通过便利的 Mobile Apps Client SDK 注册 API，使用通知中心注册设备和[模板]
    + 请注意，出于安全方面的考虑，Mobile Apps 将在注册中去除所有标记。直接从后端使用通知中心将标记与设备相关联。
3. 从应用后端使用通知中心发送通知

以下是这种集成为开发人员带来的便利：
- **Mobile Apps 客户端 SDK。** 这些多平台 SDK 提供简单的 API 用于注册，然后自动与链接到移动应用的通知中心联系。开发人员不需要深入了解通知中心凭据和使用其他服务。
    + SDK 将使用 Mobile Apps 的已经过身份验证的用户 ID 来自动标记给定设备，以实现推送到用户的方案。
    + SDK 自动使用 Mobile Apps 安装 ID 作为 GUID 来注册到通知中心，省去了开发人员维护多个服务 GUID 的麻烦。
- **安装模型。** Mobile Apps 使用通知中心的最新推送模型来呈现 JSON 安装中所有与设备关联的推送属性，该模型与推送通知密切合作且易于使用。
- **灵活性。** 即使是就地集成的，开发人员也始终可以选择直接使用通知中心。
- **[Azure 门户] 中的集成体验。** Mobile Apps 以可视化方式呈现推送功能，开发人员可以通过 Mobile Apps 轻松使用关联的通知中心。



## 后续步骤

可以通过以下主题了解有关通知中心的更多信息：

+ **[客户如何使用通知中心]**

+ **[通知中心教程和指南]**

+ **通知中心入门教程**（[iOS]、[Windows Universal]、[Windows Phone]、[Kindle]、[Xamarin.iOS]）

通知中心的相关 .NET 托管 API 参考位于此处：

+ [Microsoft.WindowsAzure.Messaging.NotificationHub]
+ [Microsoft.ServiceBus.Notifications] 

  [Azure 门户]: https://manage.windowsazure.cn/
  [0]: ./media/notification-hubs-overview/registration-diagram.png
  [1]: ./media/notification-hubs-overview/notification-hub-diagram.png
  [客户如何使用通知中心]: /zh-cn/services/notification-hubs
  [通知中心教程和指南]: /documentation/services/notification-hubs
  [iOS]: /documentation/articles/notification-hubs-ios-get-started/
  [Windows Universal]: /documentation/articles/notification-hubs-windows-store-dotnet-get-started/
  [Windows Phone]: /documentation/articles/notification-hubs-windows-phone-get-started/
  [Kindle]: /documentation/articles/notification-hubs-kindle-get-started/
  [Xamarin.iOS]: /documentation/articles/partner-xamarin-notification-hubs-ios-get-started/
  [Xamarin.Android]: /documentation/articles/partner-xamarin-notification-hubs-android-get-started/
  [Microsoft.WindowsAzure.Messaging.NotificationHub]: http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.messaging.notificationhub.aspx
  [Microsoft.ServiceBus.Notifications]: http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.notifications.aspx
  [App Service Mobile Apps]: https://azure.microsoft.com/zh-cn/documentation/articles/app-service-mobile-value-prop/
  [模板]: /documentation/articles/notification-hubs-templates/

  [标记]: (http://msdn.microsoft.com/library/azure/dn530749.aspx)

<!---HONumber=Mooncake_0503_2016-->