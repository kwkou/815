<properties
	pageTitle="Azure 事件中心 Java 编程的那些事"
	description="Azure 事件中心 Java 编程的那些事"
	service=""
	resource="eventhub"
	authors=""
	displayOrder=""
	selfHelpType=""
	supportTopicIds=""
	productPesIds=""
	resourceTags="Event Hub, Java, REST API, SDK"
	cloudEnvironments="MoonCake" />
<tags 
	ms.service="event-hub-aog"
	ms.date=""
	wacn.date="02/07/2017" />
# Azure 事件中心 Java 编程的那些事

Azure 事件中心是支持多种语言访问的，这个多语言支持除了因为提供了 REST API 之外，还因为 AMQP 协议的支持，更直观的是对应语言 SDK 的提供。基于 Java 在国内的广泛使用，这片文章将重点总结下 Azure 事件中心 Java 编程的那些事。

总的来说，Azure 事件中心 Java 编程可使用的 SDK 分为以下三种：

1.	原生 SDK
2.	整合型 SDK
3.	第三方 SDK（主要是基于 AMQP 协议）

下面就分类具体讲述下。

## 原生SDK

目前官方的是在命名空间 `com.microsoft.azure` 下的 azure-eventhubs（以下简称**新 SDK**）。它是基于 QPID.Proton 实现的，这样就绕过了 JMS，更直接。

-	源代码：[https://github.com/Azure/azure-event-hubs-java/tree/master/azure-eventhubs ](https://github.com/Azure/azure-event-hubs-java/tree/master/azure-eventhubs )
-	jar包：[https://mvnrepository.com/artifact/com.microsoft.azure/azure-eventhubs](https://mvnrepository.com/artifact/com.microsoft.azure/azure-eventhubs) 

在这个 SDK 推出之前有一个老的 SDK 是在命名空间 `com.microsoft.eventhubs.client` 下的 eventhubs-client（以下简称**老 SDK**），这个是基于 QPID.JMS 的老版本（qpid-amqp-1-0-client-jms）实现的。

-	源代码：[https://github.com/hdinsight/eventhubs-client](https://github.com/hdinsight/eventhubs-client)
-	Jar包：[https://mvnrepository.com/artifact/com.microsoft.eventhubs.client/eventhubs-client](https://mvnrepository.com/artifact/com.microsoft.eventhubs.client/eventhubs-client)

另外微软官方还提供了 EventProcessorHost 的 Java 版本，跟**新 SDK **在同一个命名空间下，叫 azure-eventhubs-eph。（以下简称**eph-SDK**）

-	源代码：[https://github.com/Azure/azure-event-hubs-java/tree/master/azure-eventhubs-eph](https://github.com/Azure/azure-event-hubs-java/tree/master/azure-eventhubs-eph)
-	Jar包：[https://mvnrepository.com/artifact/com.microsoft.azure/azure-eventhubs-eph](https://mvnrepository.com/artifact/com.microsoft.azure/azure-eventhubs-eph) 

## 整合型 SDK

事件中心也支持与社区流行的数据处理的框架相整合，因此也提供了相应的整合型 SDK，主要有两个，Apache Storm 和 Spark Streaming。

Apache Storm 整合 SDK（以下简称 **Storm 整合 SDK**）

-	源代码：[https://github.com/apache/storm/tree/master/external/storm-eventhubs](https://github.com/apache/storm/tree/master/external/storm-eventhubs)
-	Jar包：[https://mvnrepository.com/artifact/org.apache.storm/storm-eventhubs](https://mvnrepository.com/artifact/org.apache.storm/storm-eventhubs) 

Spark Streaming 整合 SDK（以下简称 **Spark 整合 SDK**）

-	源代码：[https://github.com/hdinsight/spark-eventhubs](https://github.com/hdinsight/spark-eventhubs)
-	Jar包：[https://mvnrepository.com/artifact/com.microsoft.azure/spark-streaming-eventhubs_2.11](https://mvnrepository.com/artifact/com.microsoft.azure/spark-streaming-eventhubs_2.11)

**Storm 整合 SDK** 是基于**老 SDK** 实现的，而 **Spark 整合 SDK** 是基于**新 SDK** 实现的。这种差别主要是因为 SDK 实现的时间先后造成的。

## 第三方 SDK

事件中心支持 AMQP 协议，因此也就相应支持第三方的 AMQP 协议兼容的 SDK。目前事件中心 Java 编程里主要在用的有两个，也有新老之分。

**老 JMS SDK**（qpid-amqp-1-0-client-jms）

根据该 SDK 的作者[所说](http://qpid.2158936.n2.nabble.com/What-Qpid-AMQP-1-0-client-to-use-td7635443.html)，他已经不再维护更新这个 SDK，而是主推新的 JMS SDK。所以即使当前这个 SDK 还是有着广泛用途的，但推荐使用新的 JMS SDK。

-	Jar 包（需要以下四个包结合使用）：

	[https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-client](https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-client)<br>
	[https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-client-jms](https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-client-jms)<br>
	[https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-common](https://mvnrepository.com/artifact/org.apache.qpid/qpid-amqp-1-0-common)<br>
	[https://mvnrepository.com/artifact/org.apache.geronimo.specs/geronimo-jms_1.1_spec](https://mvnrepository.com/artifact/org.apache.geronimo.specs/geronimo-jms_1.1_spec)<br>

**新 JMS SDK**（qpid-jms-client）

-	源代码：[https://github.com/apache/qpid-jms](https://github.com/apache/qpid-jms)
-	Jar包：[https://mvnrepository.com/artifact/org.apache.qpid/qpid-jms-client](https://mvnrepository.com/artifact/org.apache.qpid/qpid-jms-client)

## 总结

| 类型		| 名称						| 说明										|
|-------	|-----------------------	|---------------------------------------	|
| 原生SDK	| azure-eventhubs			| 官方最新SDK，推荐使用						|
| 原生SDK	| eventhubs-client			| 老SDK，缺乏维护，不推荐使用					|
| 原生SDK	| azure-eventhubs-eph		| 官方EventProcessorHost的Java实现，推荐使用	|
| 整合型SDK	| storm-eventhubs			| 适用于与Apache Storm整合使用				|
| 整合型SDK	| spark-streaming-eventhubs	| 适用于与Spark Streaming整合使用				|
| 第三方SDK	| qpid-amqp-1-0-client-jms	| 老JMS SDK，不再维护，不推荐使用				|
| 第三方SDK	| qpid-jms-client			| QPID最新JMS SDK，推荐使用					|

## 示例

所有 SDK 使用的示例可参照：[https://github.com/allenhula/azure-china-get-started/tree/master/EventHub/Java](https://github.com/allenhula/azure-china-get-started/tree/master/EventHub/Java) 
