<properties
    pageTitle="Azure 服务总线基础概述 | Azure"
    description="介绍如何使用服务总线将 Azure 应用程序连接到其他软件。"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="service-bus"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/08/2017"
    ms.author="sethm"
    wacn.date="04/17/2017"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="c268fc4ed2feb10af4444709366e41e439f954f5"
    ms.lasthandoff="04/07/2017" />

# <a name="azure-service-bus"></a>Azure 服务总线

无论是在云中还是在本地运行，应用程序或服务通常都需要与其他应用程序或服务交互。 借助 Azure 服务总线，你可以通过更多实用的方法进行交互。 本文将探讨这项技术，并介绍其概念以及为何要使用它。

## <a name="service-bus-fundamentals"></a>服务总线基础知识
不同的情况要求不同的通信方式。 有时，让应用程序通过简单的队列发送和接收消息是最佳解决方案。 在其他情况下，普通队列还不够；具有发布和订阅机制的队列会更好。 在某些情况下，实际需要的只是在应用程序之间建立连接，无需使用队列。 服务总线提供以上所有三个选项，使你的应用程序能够通过多种不同方式进行交互。

服务总线是一种多租户云服务，这意味着该服务可由多个用户共享。 每个用户（如应用程序开发人员）创建一个 *命名空间*，然后定义他（她）在该命名空间内需要的通信机制。 图 1 解释这种情况。

![][1]

**图 1：服务总线为通过云连接应用程序提供多租户服务。**

在命名空间内，您可以使用三种不同通信机制的一个或多个实例，每种机制使用不同方式连接应用程序。 选项有：

- *队列*，允许单向通信。 每个队列均充当一个中介（有时称为 *代理*），可存储发送的消息，直到它们被接收为止。 每条消息由单个接收方接收。
- 主题，使用订阅提供单向通信 — 单个主题可有多个订阅。 主题与队列一样可充当中转站，但每个订阅可以选择使用筛选器接收仅符合特定条件的消息。
- *中继*，提供双向通信。 与队列和主题不同，中继不存储传送中的消息，它不是中转站。 相反，中继只是将消息传递到目标应用程序。

当您创建队列、主题或中继时，请对其进行命名。 结合您对命名空间的任何命名，此名称可创建对象的唯一标识符。 应用程序可将此名称提供给服务总线，然后使用队列、主题或中继相互通信。 

若要在中继场景中使用任意这些对象，Windows 应用程序可使用 Windows Communication Foundation (WCF)。 对于队列和主题，Windows 应用程序还可使用服务总线定义的消息传送 API。 为了更轻松地通过非 Windows 应用程序使用这些对象，Microsoft 提供了 Java、Node.js 和其他语言的 SDK。 此外，也可以使用 [REST API](https://docs.microsoft.com/zh-cn/rest/api/servicebus/) 通过 HTTP 访问队列和主题。 

即使服务总线本身在云（ Azure 数据中心）中运行，使用它的应用程序也能随处运行，了解这一点很重要。您可以使用服务总线连接在 Azure 上运行的应用程序或在您自己的数据中心内运行的应用程序。您也可以使用服务总线通过本地应用程序或通过平板电脑和手机来连接在 Azure 或其他云平台上运行的应用程序。甚至可以将家用电器、传感器和其他设备连接到中央应用程序或其他应用程序。服务总线是云中的通信机制，几乎可从任何位置对其进行访问。使用服务总线的方式取决于应用程序需要执行的操作。

## <a name="queues"></a>队列
假设你决定使用服务总线队列连接两个应用程序。图 2 说明了此情况。

![][2]  


**图 2：服务总线队列提供单向异步排队方法。**

过程很简单：发送方将消息发送至服务总线队列，接收方在随后的某个时间内接收该消息。一个队列只能有一个接收方，如图 2 所示。或者，多个应用程序可从同一个队列读取。在后一种情况下，每条消息仅由一个接收方读取。对于多播服务，应改用主题。

每条消息均由两部分组成：一组属性（每一组都是键/值对）和消息有效负载。 有效负载可以是二进制文件、文本甚或 XML。 使用的方式取决于应用程序尝试执行的操作。 例如，发送近期销售消息的应用程序可能包含属性 Seller="Ava" 和 Amount=10000。 消息正文可能包含已签署的销售合同的扫描图像，如果不包含该合同，只需留空。

接收方可采用两种不同方式从服务总线队列中读取消息。 第一种方式称作 *[ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode)*，即，从队列中移除消息并立即将其删除。 此操作很简单，但如果接收方在完成处理消息之前崩溃，则该消息将丢失。 因为消息已从队列中移除，所以接收方无法访问该消息。 

第二种方式 *[PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode)* 旨在帮助解决这个问题。 与 **ReceiveAndDelete** 类似，**PeekLock** 可从队列中移除消息。 但是，它不会删除该消息。 相反，它会锁定该消息，使其对其他接收方不可见，然后等待以下三个事件之一：

* 如果接收方成功处理了该消息，将调用 **[Complete()](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete)**，并且队列将删除该消息。 
* 如果接收方判定它无法成功处理该消息，将调用 **[Abandon()](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Abandon)**。 队列即会解除对该消息的锁定，使其可供其他接收方使用。
* 如果接收方在可配置时间段（默认为 60 秒）内没有调用这两个命令，队列将假定接收方失败。 在这种情况下，队列的行为就像接收方已调用 **Abandon**一样，即，使消息可供其他接收方使用。

请注意可能发生的情况：同一条消息可能被发送两次，可能将其发送给两个不同的接收方。 使用服务总线队列的应用程序必须为这种情况做好准备。 为了更轻松地进行重复检测，每条消息都具有一个唯一的 [MessageID](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId) 属性，无论从队列中读取消息多少次，该属性在默认情况下始终保持不变。 

队列在很多情况下都非常有用。 即使两个应用程序没有同时运行，队列也可使这两个应用程序之间相互通信，这对于批处理和移动应用程序尤为方便。 当所发送的消息传播给多个接收方时，具有这些接收方的队列还提供自动负载均衡。

## <a name="topics"></a>主题
队列虽然在一些情况下有用，但并非总是正确的解决方案。 有时，服务总线主题更好。 图 3 说明了这一点。

![][3]

**图 3：根据订阅应用程序指定的筛选器，它可接收发送至服务总线主题的部分或全部消息。**

*主题* 在很多方面与队列类似。 发送方将消息提交至主题的方式与将消息提交至队列的方式相同，这些消息与使用队列的消息看起来一样。 最大的区别是主题允许每个接收应用程序通过定义筛选器创建其自己的订阅。 然后，订户将只能看到与该筛选器匹配的消息。 例如，图 3 显示了一个发送方以及一个具有三个订户的主题，每个订户都拥有自己的筛选器：

- 订户 1 仅接收包含 *Seller="Ava"*属性的消息。
- 订户 2 接收包含属性 Seller="Ruby" 和/或包含的 Amount 属性值大于 100,000 的消息。 Ruby 可能是销售经理，因此她希望查看她自己的销售和其他人所做的所有大单销售。
- 订户 3 将其筛选器设置为 *True*，这意味着它将接收所有消息。 例如，此应用程序可能负责维护审核跟踪，因此它需要查看所有消息。

与队列一样，某主题的订户可使用 [**ReceiveAndDelete** 或 **PeekLock**](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 读取消息。 不过与队列不同的是，发送至主题的单个消息可由多个订阅接收。 此方法通常称作发布和订阅（或 pub/sub），在当多个应用程序对相同消息感兴趣时非常有用。 通过定义适当的筛选器，每位订户可以只访问需要查看的消息流部分。

## <a name="relays"></a>中继
队列和主题均通过中转站提供单向异步通信。 流量只按一个方向流动，发送方和接收方之间没有直接连接。 但是，如果您不希望这样怎么办？ 假设你的应用程序需要同时发送和接收消息，或者可能你希望应用程序之间进行直接链接，而不需要使用代理存储消息。 为解决此类情况，服务总线提供了 *中继*，如图 4 所示。

![][4]

**图 4：服务总线中继提供应用程序之间的同步双向通信。**

关于中继的一个很明显的问题是：为何我要使用中继？ 即使我不需要队列，为什么仍然要通过云服务进行应用程序通信，而非直接交互？ 答案是直接对话比您想象的更困难。

假设您希望连接两个本地应用程序，这两个应用程序均在企业数据中心运行。 每个应用程序都位于防火墙之后，并且每个数据中心很可能使用网络地址转换 (NAT)。 防火墙阻止所有端口（少数端口除外）上的传入数据，而 NAT 意味着运行每个应用程序的计算机没有固定的 IP 地址可供从数据中心外部直接连接。 如果不借助一些额外的帮助，那么通过公共 Internet 连接这些应用程序会产生问题。

服务总线中继可以提供帮助。 若要通过中继进行双向通信，每个应用程序需使用服务总线建立出站 TCP 连接，然后一直保持打开状态。 将通过这些连接传输两个应用程序之间的全部通信。 由于每个连接均从数据中心内部建立，因此，防火墙将允许到每个应用程序的传入流量，而无需打开新端口。 此方法还能解决 NAT 问题，因为每个应用程序在整个通信中的云终结点是一致的。 通过中继交换数据，应用程序可以避免导致通信困难的问题。 

为使用服务总线中继，应用程序依赖 Windows Communication Foundation (WCF)。 服务总线提供 WCF 绑定，可使 Windows 应用程序通过中继进行交互变得更加简单。 已使用 WCF 的应用程序通常只需指定其中一个绑定，即可通过中继进行交互。 不过与队列和主题不同，从非 Windows 应用程序使用中继（如有可能）需要一些编程工作；不提供标准库。

与队列和主题不同，应用程序不会显式创建中继。 相反，当希望接收消息的应用程序与服务总线建立 TCP 连接时，会自动创建中继。 当该连接中断时，将删除该中继。 为了让应用程序能够找到由特定侦听程序创建的中继，服务总线提供了可让应用程序按名称查找特定中继的注册表。

当你需要在应用程序之间直接通信时，中继是正确的解决方案。 例如，在本地数据中心运行的航空订票系统，可从值机柜台、移动设备和其他计算机上访问该系统。 在所有这些系统上运行的应用程序都可能依赖云中的服务总线中继进行通信，无论它们在哪里运行。

## <a name="summary"></a>摘要

连接应用程序始终属于构建完整解决方案的一部分，需要应用程序和服务彼此通信的解决方案范围随着连到 Internet 的应用程序和设备的日益增加而增加。 通过提供可实现此目的的基于云的技术（即队列、主题和中继），服务总线旨在让此基本功能更易于实施且使用范围更加广泛。

## <a name="next-steps"></a>后续步骤

现在，你已了解有关 Azure 服务总线的基础知识，请单击下面的链接了解更多信息。

- 如何使用 [Service Bus 队列](/documentation/articles/service-bus-dotnet-get-started-with-queues/)。
- 如何使用[服务总线主题](/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/)。
- 如何使用[服务总线中继](/documentation/articles/service-bus-dotnet-how-to-use-relay/)。
- [服务总线示例](/documentation/articles/service-bus-samples/)。

[1]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_01_architecture.png
[2]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_02_queues.png
[3]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_03_topicsandsubscriptions.png
[4]: ./media/service-bus-fundamentals-hybrid-solutions/SvcBus_04_relay.png


<!--Update_Description:update wording and link references-->