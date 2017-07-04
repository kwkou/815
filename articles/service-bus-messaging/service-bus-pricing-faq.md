<properties 
    pageTitle="服务总线定价常见问题解答 | Azure"
    description="回答了一些关于服务总线定价结构的常见问题。"
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags 
   ms.service="service-bus"
   ms.date="03/16/2016"
   wacn.date="01/09/2017" />

# 服务总线定价常见问题解答

本文回答了一些关于服务总线定价结构的常见问题。你还可以访问 [Azure 支持常见问题](http://go.microsoft.com/fwlink/?LinkID=185083)了解一般的 Azure 定价信息。有关服务总线定价的完整信息，请参阅[服务总线定价](/pricing/details/messaging/)。

>[AZURE.NOTE] 有关事件中心的定价结构，请参阅[事件中心的可用性和支持常见问题解答](/documentation/articles/event-hubs-faq/)一文，有关其详细信息，请参阅[事件中心定价](/pricing/details/event-hubs/)一文。

- [服务总线如何收取费用？](#how-do-you-charge-for-service-bus)
- [服务总线的哪些使用情况受数据传输限制？ 哪些不受其限制？](#what-usage-of-service-bus-is-subject-to-data-transfer-what-is-not)
- [服务总线的“中继”具体指什么？](#what-exactly-is-a-service-bus-quotrelayquot)
- [如何计算中继小时数？](#how-is-the-relay-hours-meter-calculated)
- [如果将多个侦听器连接到给定中继，会怎么样？](#what-if-i-have-more-than-one-listener-connected-to-a-given-relay)
- [如何计算中继的消息数？](#how-is-the-messages-meter-calculated-for-relays)
- [服务总线是否对存储收费？](#does-service-bus-charge-for-storage)
- [服务总线是否有任何使用率配额？](#does-service-bus-have-any-usage-quotas)

## <a name="how-do-you-charge-for-service-bus"></a> 服务总线如何收取费用？

有关服务总线定价的完整信息，请参阅[服务总线定价和计费](/documentation/articles/service-bus-pricing-billing/)以及[服务总线定价详细信息](/pricing/details/messaging/)。除标示的价格外，你还需为在其中部署应用程序的数据中心之外的相关数据输出支付费用。有关详细信息，请参阅下面的[服务总线的哪些使用情况受数据传输限制？ 哪些不受其限制？](#what-usage-of-service-bus-is-subject-to-data-transfer-what-is-not)部分。

## <a name="what-usage-of-service-bus-is-subject-to-data-transfer-what-is-not"></a> 服务总线的哪些使用情况受数据传输限制？ 哪些不受其限制？

在给定的 Azure 区域内的任何数据传输均不收费。
## <a name="what-exactly-is-a-service-bus-quotrelayquot"></a> 服务总线的“中继”具体指什么？

中继是指中继客户端和 Web 服务之间的消息的服务总线实体。中继提供持久性访问、可发现的服务总线地址、可靠的连接（具有防火墙/NAT 遍历功能）和其他功能（例如自动负载均衡）。每当启用了中继的 WCF 服务（即“中继侦听器”）第一次连接到给定服务总线地址（命名空间 URL）时，中继将隐式实例化并在该地址中打开。应用程序使用服务总线 .NET API 创建中继侦听器，API 可提供启用了特殊中继的标准 WCF 绑定版本。

## <a name="how-is-the-relay-hours-meter-calculated"></a>如何计算中继小时数？

将按每个服务总线中继在指定计费期间处于“打开”状态的累积时间量对中继小时数收费。当启用了中继的 WCF 服务（即“中继侦听器”）第一次连接到给定服务总线地址（服务命名空间 URL）时，中继将隐式实例化并在该地址中打开。仅当最后一个侦听器从其地址断开连接时，该中继才会关闭。因此，出于计费目的，在第一个中继侦听器连接到中继的服务总线地址到最后一个中继侦听器从该地址断开连接的这段时间内，该中继将被认为处于“打开”状态。换而言之，当有一个或多个中继侦听器连接到其服务总线地址时，中继都将被认为处于打开状态。

## <a name="what-if-i-have-more-than-one-listener-connected-to-a-given-relay"></a> 如果将多个侦听器连接到给定中继，会怎么样？

在某些情况下，服务总线中的单个中继可能会有多个连接的侦听器。这种情况可能会出现在使用 **netTCPRelay** 或 **HttpRelay** WCF 绑定的负载均衡服务中，或使用 **netEventRelay** WCF 绑定的广播事件侦听器中。当至少有一个中继侦听器连接到中继服务总线时，该中继都就将被视为“打开”状态。出于计费目的将其他侦听器添加到打开的中继时，不会更改该中继的状态。连接到中继的中继发送方（调用或将消息发送至中继的客户端）数量也不会对中继小时数的计算产生影响。

## <a name="how-is-the-messages-meter-calculated-for-relays"></a> 如何计算中继的消息数？

一般而言，使用与上述用于描述中转实例（队列、主题和订阅）相同的方法来计算中继的可计费消息数。但是，有几个明显的区别：

1. 将消息发送到服务总线中继是指将消息“全通式”发送至中继侦听器，而不是发送至服务总线中继后再传递到中继侦听器。因此，针对中继侦听器的请求-应答服务调用（最大 64 KB）将生成两条可计费消息：一条用于请求的可计费消息和一条用于应答（假设响应也不超过 64 KB）的可计费消息。这不同于使用队列在客户端和服务之间进行调谐。对于后一种情况，相同的请求-应答模式需要一个发送到队列的请求，接着是从队列取消排队/从队列传递到服务，然后是发送至另一个队列的响应，最后从该队列取消排队/从该队列传递到客户端。因此，使用相同大小 (<= 64 KB) 的假设吞吐量，经过调谐的队列模式将生成四条可计费消息，此数量是使用中继实现相同模式的计费数的两倍。当然，使用队列来实现此模式好处更多，例如持久性和负载分级。这些好处可能产生额外费用。

2. 使用 **netTCPRelay** WCF 绑定打开的中继不是作为单条消息而是作为流经系统的数据流。换而言之，只有发送方和侦听器可以识别使用此绑定发送/接收的单条分帧消息。因此，对于使用 **netTCPRelay** 绑定的中继，所有数据都将被视为用于计算可计费消息的数据流。在此示例中，服务总线将计算每 5 分钟通关单个中继发送或接收的数据总量，然后将该总量除以 64 KB，从而得出在该时间段出现问题的中继的可计费消息数。

## <a name="does-service-bus-charge-for-storage"></a> 服务总线是否对存储收费？

否，服务总线不对存储收费。但是，对每个队列/主题可以保留的数据最大量设有配额限制。请参阅下一个常见问题。

## <a name="does-service-bus-have-any-usage-quotas"></a> 服务总线是否有任何使用率配额？

默认情况下，对于任何云服务，Microsoft 每月都会设置聚合的使用率配额，此配额基于客户的所有订阅进行计算。由于我们清楚你可能需要这些限制之外的更多限制，因此，请随时联系客户服务，以便我们了解你的需求并相应地调整这些限制。对于服务总线，聚合的使用率配额为如下所示：

- 50 亿条消息

- 200 万个中继小时

虽然我们保留禁用在给定月份超过使用率配额的客户帐户的权利，但我们仍然会在采取任何措施前发送电子邮件通知并会多次尝试与客户联系。超过这些配额的客户仍将负责超出配额的费用。

至于 Azure 上的其他服务，服务总线会强制使用一组特定[配额](/documentation/articles/service-bus-quotas/)，以确保资源的公平使用。服务强制执行的使用率配额如下：

- **队列/主题大小** – 在创建队列或主题时指定最大队列或主题大小。此配额的值可以是 1 GB、2 GB、3 GB、4 GB 或 5 GB。如果达到最大大小，则将拒绝其他传入的消息，调用代码将收到一个异常。

- **并发连接数**
	- **队列/主题/订阅** -队列/主题/订阅上的并发 TCP 连接数限制为 100。如果达到此配额，将拒绝后续的其他连接请求，调用代码将收到一个异常。对于每个消息工厂，如果由该消息传送工厂创建的任何客户端具有活动的操作挂起时，或者刚完成操作不超过 60 秒时间，则服务总线都会保持一个 TCP 连接。REST 操作不计入并发 TCP 连接数。


- **中继上的并发侦听器数** - 中继上的并发 **netTcpRelay** 和 **netHttpRelay ** 侦听器数被限制为 25 个（1 表示 **NetOneway** 中继）。

- **每个命名空间的并发中继侦听器数** – 服务总线对每个服务命名空间强制的并发中继侦听器数为 2000 个。如果达到此配额，将拒绝后续的打开其他中继侦听器连接的请求，调用代码将收到一个异常。

- **每个服务命名空间的主题/队列数** – 服务命名空间上的主题/队列（持久存储备份的实体）的最大数被限制为 10,000。如果达到此配额，将拒绝后续的在服务命名空间上创建新主题/队列的请求。在这种情况下，[Azure 经典管理门户][] 将显示一条错误消息或调用客户端代码将收到一个异常，具体取决于创建尝试是通过门户还是通过客户端代码完成。

- **消息大小配额**
	- **队列/主题/订阅**
		- **消息大小** – 每条消息的总大小限制为 256 KB，包括消息标头。
		- **消息标头大小** – 每条消息标头限制为 64 KB。

	- **NetOneway 和 NetEvent 中继** – 每条消息的总大小限制为 64 KB，包括消息标头。
	- **Http 和 NetTcp 中继** – 服务总线不强制设置这些消息的大小上限。

	系统将拒绝超过这些大小配额的消息，调用代码将收到一个异常。

- **每个主题的订阅数** – 每个主题的最大订阅数被限制为 2000。如果达到此配额，则将拒绝创建该主题的附加订阅的请求。在这种情况下，[Azure 经典管理门户][] 将显示一条错误消息或调用客户端代码将收到一个异常，具体取决于创建尝试是通过门户还是通过客户端代码完成。

- **每个主题的 SQL 筛选器数** – 每个主题的最大 SQL 筛选器数被限制为 2,000。如果达到此配额，将拒绝任何后续的创建该主题的其他筛选器的请求，调用代码将收到一个异常。

- **每个主题的相关筛选器数** – 每个主题的最大相关筛选器数被限制为 100,000。如果达到此配额，将拒绝任何后续的创建该主题的其他筛选器的请求，调用代码将收到一个异常。

有关配额的详细信息，请参阅[服务总线配额](/documentation/articles/service-bus-quotas/)。

## 后续步骤

若要了解有关服务总线消息传送的详细信息，请参阅以下主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [Azure 服务总线体系结构概述](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)
- [如何使用 Service Bus 队列](/documentation/articles/service-bus-dotnet-get-started-with-queues/)
[Azure 经典管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_Quality_Review_0104_2017-->