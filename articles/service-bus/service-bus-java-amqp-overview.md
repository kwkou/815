<properties 
	pageTitle="服务总线 AMQP 概述 (Java) | Microsoft Azure" 
	description="了解如何在 Azure 中使用 Java 和高级消息队列协议 (AMQP) 1.0。" 
	services="service-bus" 
	documentationCenter="java" 
	authors="sethmanheim" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.date="03/09/2016" 
	wacn.date="12/17/2015"/>


# 服务总线的 AMQP 1.0 支持

Azure 服务总线云服务和 [Windows Server 服务总线（服务总线 1.1）](https://msdn.microsoft.com/zh-cn/library/dn282144.aspx)均支持高级消息队列协议 (AMQP) 1.0。AMQP 让您能够使用开放标准协议构建跨平台的混合应用程序。你可以借助使用不同语言和框架构建的且运行在不同操作系统上的组件来构建应用程序。所有这些组件均可连接到服务总线，并且能够高效且完全无损地无缝交换结构化业务消息。

## 简介：什么是 AMQP 1.0，为何它很重要？

通常，面向消息的中间件产品始终使用专用协议来支持客户端应用程序和代理之间的通信。这意味着，在您选择特定供应商的消息传递代理后，您必须使用该供应商的库将您的客户端应用程序连接到该代理。这将导致在一定程度上依赖于该供应商，因为将应用程序传送到其他产品需要对所有已连接应用程序进行代码改写。

此外，连接来自不同供应商的消息传递代理比较棘手。这通常需要通过应用程序级桥接来将消息从一个系统移到另一个系统，并在其专用消息格式之间进行转换。这是一个常见的要求；例如，在必须向较旧的独立系统提供新的统一接口时，或者在合并后集成 IT 系统时。

软件产业是一个飞速发展的产业；新编程语言和应用程序框架的引入速度有时会非常惊人。同样，IT 系统的要求随着时间不断变化，并且开发人员希望利用最新的平台功能。但有时候，所选消息传送供应商不支持这些平台。由于使用的是专用消息协议，其他供应商无法为这些新平台提供库。因此，你必须使用构建网关等手段，或者可让你继续使用消息传递产品的网桥。

这些问题推动了高级消息队列协议 (AMQP) 1.0 的开发。这种协议源于 JP Morgan Chase，像多数金融服务公司一样，该公司大量使用面向消息的中间件。目标非常简单：就是创建一个开放标准消息传送协议，从而能够借助使用不同语言、框架和操作系统构建的组件来构建基于消息的应用程序，而所有这些应用程序都使用各个供应商提供的同类最佳组件。

## AMQP 1.0 技术特性

AMQP 1.0 是一个高效、可靠的线级消息传递协议，可用于构建强大、跨平台的消息传递应用程序。协议有一个简单的目标：定义用于在两方之间安全、可靠且高效传输消息的机制。这些消息本身使用可移植数据表示进行编码，这种表示支持不同发送者和接收者完全无损地交换结构化业务消息。下面简要介绍几个最重要的特性：

*    **高效**：AMQP 1.0 是一个面向连接的协议，它将二进制编码用于协议指令以及通过该协议传输的业务消息。它融合了复杂的流控制方案，可最大限度地利用网络和已连接组件。也就是说，该协议旨在实现有效性、灵活性和互操作性之间的平衡。
*    **可靠**：使用 AMQP 1.0 协议交换消息时，你可以获得一系列可靠性保证，如即发即弃 (fire-and-forget) 和可靠的恰一次确认传送 (exactly-once acknowledged delivery)。
*    **灵活**：AMQP 1.0 是一个灵活的协议，可用于支持不同的拓扑。可以将同一协议用于客户端到客户端、客户端到代理以及代理到代理通信。
*    **独立于代理模型**：AMQP 1.0 规范对代理所使用的消息传送模型不作任何要求。这意味着可以向现有消息传送代理中轻松添加 AMQP 1.0 支持。

## AMQP 1.0 是一种标准（带有大写字母“S”）

自 2008 年以来，AMQP 1.0 一直由一个由 20 多家公司（包括技术提供商和最终用户企业）组成的核心小组进行开发。在此期间，用户企业提供其实际业务需求，技术供应商则开发该协议来满足这些需求。在整个过程中，供应商参加了研讨会，并在会上协作验证了其实现之间的互操作性。

2011 年 10 月，开发工作提交给结构化信息标准促进组织（Organization for the Advancement of Structured Information Standards，OASIS）内的技术委员会，随后 OASIS AMQP 1.0 标准于 2012 年 10 月发布。在开发该标准期间，以下公司参与了技术委员会的工作：

*    **技术供应商**：Axway Software、Huawei Technologies、IIT Software、INETCO Systems、Kaazing、Microsoft、Mitre Corporation、Primeton Technologies、Progress Software、Red Hat、SITA、Software AG、Solace Systems、VMware、WSO2、Zenika。
*    **企业用户**：Bank of America、Credit Suisse、Deutsche Boerse、Goldman Sachs、JPMorgan Chase。

开放标准的公认好处包括：

*    供应商锁定的几率较低
*    互操作性
*    有大量库和工具可供使用
*    不会过时
*    随时可找到知识渊博的工作人员
*    风险较低且可控

## AMQP 1.0 和 Service Bus

Azure 服务总线中对 AMQP 1.0 的支持意味着现在可以通过一系列使用有效的二进制协议的平台利用队列和发布/订阅中转消息传送功能。此外，你还可以生成由结合使用多个语言、框架和操作系统构建的组件组成的应用程序。

下图显示了一个部署示例，其中 Java 客户端运行在 Linux 上，并使用标准 Java 消息服务 (JMS) API 写入数据；而 .NET 客户端运行在 Windows 上，并通过服务总线使用 AMQP 1.0 交换消息。

![][0]

**图 1：演示使用服务总线和 AMQP 1.0 进行跨平台消息传送的部署方案示例**

在这种情况下，已知下列客户端库将使用服务总线：

| 语言 | 库 |
|----------|-------------------------------------------------------------------------------|
| Java | Apache Qpid Java 消息服务 (JMS) 客户端<br/>IIT 软件 SwiftMQ Java 客户端 |
| C | Apache Qpid Proton-C |
| PHP | Apache Qpid Proton-PHP |
| Python | Apache Qpid Proton-Python |


**图 2：AMQP 1.0 客户端库表**

有关如何获取和使用这些库以便用于服务总线的详细信息，请参阅 [服务总线 AMQP 开发人员指南][]。有关更多信息的链接，请参阅[后续步骤](/documentation/articles/service-bus-java-amqp-overview/#next-steps)部分。

## 摘要

*    AMQP 1.0 是一个开放、可靠的消息传递协议，可用于构建跨平台的混合应用程序。AMQP 1.0 是一种 OASIS 标准。
*    Azure 服务总线和 Windows Server 服务总线（服务总线 1.1）都支持 AMQP 1.0。定价与现有协议相同。

## 后续步骤

请访问以下链接以了解有关服务总线中的 AMQP 支持的详细信息。

*    [如何将 AMQP 1.0 与服务总线 .NET API 一起使用](/documentation/articles/service-bus-dotnet-advanced-message-queuing/)
*    [如何将 Java 消息服务 (JMS) API 用于服务总线 和 AMQP 1.0](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)
*    [服务总线 AMQP 开发人员指南](/documentation/articles/service-bus-amqp-dotnet/)
*    [OASIS 高级消息队列协议 (AMQP) 1.0 版规范](http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf)

[0]: ./media/service-bus-java-amqp-overview/Example1.png

 

<!---HONumber=Mooncake_1207_2015-->