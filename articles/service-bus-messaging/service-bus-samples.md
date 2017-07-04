<properties 
   pageTitle="服务总线示例概述 | Azure"
    description="分类并介绍相互链接的服务总线消息传送示例。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
    editor="" />
<tags 
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/07/2016"
    ms.author="sethm"  
    wacn.date="01/04/2017" />

# 服务总线消息传送示例

服务总线消息传送示例演示了[服务总线](/home/features/messaging/)（云服务）和 [Windows Server 服务总线](https://msdn.microsoft.com/zh-cn/library/dn282144.aspx)中的主要功能。本文分类并介绍了可用的示例，每个示例均具有链接。

>[AZURE.NOTE] 服务总线示例未安装 SDK。若要获取这些示例，请访问 [Azure SDK 示例页](https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=5)。
>
>另外，[此处](https://github.com/Azure-Samples/azure-servicebus-messaging-samples)提供一组已更新的服务总线消息传送示例（撰写本文时未介绍这些示例）。[此处](https://github.com/Azure-Samples/azure-servicebus-relay-samples)是中继示例。


## 服务总线消息传送

下面的示例说明了如何编写使用服务总线消息传送的应用程序。

请注意，消息传送示例需要一个连接字符串以访问服务总线命名空间。

### 获取 Azure 服务总线的连接字符串

1. 登录到 [Azure 经典管理门户](http://manage.windowsazure.cn)。

2. 在左侧列中，单击“服务总线”。

1. 在列表中，单击命名空间的名称。

3. 在命名空间边栏选项卡中，单击“共享访问策略”。

4. 在“共享访问策略”边栏选项卡中，单击“RootManageSharedAccessKey”。

6. 将连接字符串复制到剪贴板。

### 获取 Windows Server 服务总线的连接字符串

1. 运行以下 Azure Powershell cmdlet：

    	get-sbClientConfiguration

2. 将连接字符串粘贴到示例的 App.config 文件中。

### 入门示例

这些示例说明了基本的消息传送功能。

|示例名称|说明|最低 SDK 版本|可用性|
|---|---|---|---|
|[入门：使用队列传送消息](http://code.msdn.microsoft.com/Getting-Started-Brokered-aa7a0ac3)|演示如何使用 Azure 服务总线发送和接收来自队列的消息。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[入门：使用主题传送消息](http://code.msdn.microsoft.com/Getting-Started-Brokered-614d42e5)|演示如何使用 Azure 服务总线发送和接收来自具有多个订阅的主题的消息。|1\.8|Azure 服务总线；Windows Server 服务总线|

### 探索功能

下面的示例演示了服务总线的各种功能。

|示例名称|说明|最低 SDK 版本|可用性|
|---|---|---|---|
|[HTTP 令牌提供程序](http://code.msdn.microsoft.com/windowsazure/Service-Bus-HTTP-Token-38f2cfc5)|演示对服务总线的 HTTP/REST 客户端进行身份验证的不同方法。|2\.1|Azure 服务总线；Windows Server 服务总线|
|[服务总线 HTTP 客户端](http://code.msdn.microsoft.com/windowsazure/Service-Bus-HTTP-client-fe7da74a)|演示如何通过 HTTP/REST 将消息发送到服务总线和接收来自服务总线的消息。|2\.3|Azure 服务总线；Windows Server 服务总线|
|[服务总线自动转发](http://code.msdn.microsoft.com/windowsazure/Service-Bus-Autoforwarding-b9df470b)|演示如何将从队列、订阅或死信队列中的消息自动转发到另一个队列或主题。还演示如何通过传输队列将消息发送到队列或主题。|2\.3|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：WCF 通道会话示例](http://code.msdn.microsoft.com/Brokered-Messaging-WCF-0a526451)|演示如何通过 Windows Communication Foundation (WCF) 通道使用 Azure 服务总线。此示例演示了如何使用 WCF 通道来通过服务总线队列发送和接收消息。此示例演示了通过服务总线进行的会话和非会话通信。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：事务](http://code.msdn.microsoft.com/Brokered-Messaging-8cd41d1e)|演示如何使用事务范围内的 Azure 服务总线消息传送功能来确保消息传送操作的批处理是以原子方式提交的。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：使用 REST 管理操作](http://code.msdn.microsoft.com/Brokered-Messaging-569cff88)|演示如何使用 REST 在服务总线上执行管理操作。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[资源提供程序 REST API](http://code.msdn.microsoft.com/Service-Bus-Resource-5d887203)|演示如何使用新的服务总线 RDFE REST API 来管理命名空间和消息传送实体。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：WCF 服务会话示例](http://code.msdn.microsoft.com/Brokered-Messaging-WCF-db4262c2)|演示如何通过 WCF 服务模型来使用 Azure 服务总线。此示例演示了通过服务总线队列使用 WCF 服务模型完成基于会话的通信。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：请求响应](http://code.msdn.microsoft.com/Brokered-Messaging-Request-2b4ff5d8)|演示如何使用 Azure 服务总线和请求/响应功能。此示例演示了通过服务总线队列通信的简单客户端和服务器。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：死信队列](http://code.msdn.microsoft.com/Brokered-Messaging-Dead-22536dd8)|演示如何使用 Azure 服务总线和消息传送“死信队列”功能。此示例演示了通过服务总线队列通信的发送方和接收方。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：延迟消息](http://code.msdn.microsoft.com/Brokered-Messaging-ccc4f879)|演示如何使用 Azure 服务总线的消息延迟功能。此示例演示了通过服务总线队列通信的发送方和接收方。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：会话消息](http://code.msdn.microsoft.com/Brokered-Messaging-Session-41c43fb4)|演示如何使用 Azure 服务总线和消息传送会话功能。此示例演示了通过服务总线队列通信的发送方和接收方。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：请求响应主题](http://code.msdn.microsoft.com/Brokered-Messaging-Request-6759a36e)|演示如何使用 Azure 服务总线主题和订阅实现请求/响应模式。此示例演示了通过服务总线主题通信的简单客户端和服务器。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：请求响应队列](http://code.msdn.microsoft.com/Brokered-Messaging-Request-0ce8fcaf)|演示如何使用 Azure 服务总线和请求/响应功能。此示例演示了通过两个服务总线队列进行通信的简单客户端和服务器。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：重复检测](http://code.msdn.microsoft.com/Brokered-Messaging-c0acea25)|演示如何通过队列使用 Azure 服务总线重复消息检测。它创建两个队列，一个启用了重复检测，另一个未启用重复检测。|1\.8|Microsoft Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：异步消息传送](http://code.msdn.microsoft.com/Brokered-Messaging-Async-211c1e74)|演示如何使用 Azure 服务总线异步发送和接收来自队列的消息。队列提供了发送方和任意数量的接收方（此处为一个接收方）之间的分离式异步通信。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：高级筛选器](http://code.msdn.microsoft.com/Brokered-Messaging-6b0d2749)|演示如何使用 Azure 服务总线发布/订阅高级筛选器。创建了的一个主题和具有不同筛选器定义的三个订阅，用于将消息发送到主题，以及接受来自订阅的所有消息。|1\.8|Azure 服务总线；Windows Server 服务总线|
|[中转消息传送：消息预提取](http://code.msdn.microsoft.com/Brokered-Messaging-be2dac1d)|演示如何使用 Azure 服务总线预提取功能。演示了如何在接收时使用消息预提取功能。|1\.8|Azure 服务总线；Windows Server 服务总线|

## 服务总线参考工具

下面的示例演示了服务的各种功能。

|示例名称|说明|最低 SDK 版本|可用性|
|---|---|---|---|
|[服务总线资源管理器](http://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)|服务总线资源管理器允许用户连接到服务总线服务命名空间并以一种简单的方式管理消息传送实体。该工具提供了各种高级功能，例如导入/导出功能以及测试消息实体和中继服务的功能。|1\.8| Azure 服务总线；Windows Server 服务总线|
|[授权：SBAzTool](http://code.msdn.microsoft.com/Authorization-SBAzTool-6fd76d93)|此示例演示了如何在 Azure Active Directory（也称为访问控制服务或 ACS）中创建和管理服务标识，以便使用服务总线。|不适用|Azure Service Bus|

## 后续步骤

有关服务总线的概念性概述，请参阅以下主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线体系结构](/documentation/articles/service-bus-architecture/)
- [服务总线基础知识](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)

<!---HONumber=Mooncake_Quality_Review_1230_2016-->