<properties
 pageTitle="使用 Apache Storm on HDInsight 处理车辆传感器数据 | Azure"
 description="了解如何使用 Apache Storm on HDInsight 处理事件中心的车辆传感器数据。添加 DocumentDB 提供的车型数据，然后将输出存储到存储空间。"
 services="hdinsight,documentdb,notification-hubs"
 documentationCenter=""
 authors="Blackmist"
 manager="paulettm"
 editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/18/2016"
	wacn.date="04/26/2016"/>

#使用 Apache Storm on HDInsight 处理 Azure 事件中心的车辆传感器数据

了解如何使用 Apache Storm on HDInsight 处理 Azure 事件中心的车辆传感器数据。此示例读取 Azure 事件中心的传感器数据，通过引用存储在 Azure DocumentDB 中的数据来丰富数据，最后使用 Hadoop 文件系统 (HDFS) 将数据存储在 Azure 存储空间。

![HDInsight 和物联网 (IoT) 体系结构关系图](./media/hdinsight-storm-iot-eventhub-documentdb/iot.png)

##概述

向车辆添加传感器可以根据历史数据趋势预测设备问题，并可根据使用模式分析改进未来版本。虽然传统的 MapReduce 批处理可以用于这种分析，但你必须能够快速有效地将所有车辆的数据加载到 Hadoop 中，然后才能进行 MapReduce 处理。此外，你可能还希望能够实时地针对关键故障路径（引擎温度、刹车等）进行分析。

Azure 事件中心可用于处理传感器生成的大量数据，而 Apache Storm on HDInsight 则可用于加载和处理这些数据，然后将这些数据存储到 HDFS（由 Azure 存储空间提供支持）中，以便进行额外的 MapReduce 处理。

##解决方案

关于引擎温度、环境温度和车速的遥测数据将由传感器记录，然后与车辆的车辆识别号 (VIN) 及时间戳一起发送到事件中心。在事件中心，运行在 Apache Storm on HDInsight 群集上的 Storm 拓扑将读取并处理数据，然后将其存储到 HDFS 中。

在处理期间，VIN 用于从 Azure DocumentDB 检索车型信息。该信息在存储之前会添加到数据流中。

在 Storm 拓扑中使用的组件包括：

* **EventHubSpout** - 从 Azure 事件中心读取数据

* **TypeConversionBolt** - 将事件中心提供的 JSON 字符串转换为包含引擎温度、环境温度、车速、VIN 和时间戳等各项数据值的元组

* **DataReferencBolt** - 使用 VIN 从 DocumentDB 查找车型

* **WasbStoreBolt** - 将数据存储到 HDFS（Azure 存储空间）

下面是此解决方案的图示：

![storm 拓扑](./media/hdinsight-storm-iot-eventhub-documentdb/iottopology.png)

> [AZURE.NOTE]这是一个简化的关系图，解决方案中的每个组件可能有多个实例。例如，拓扑中每个组件的多个实例分布在 Storm on HDInsight 群集的多个节点中。

##实现

GitHub 上的 [HDInsight-Storm-Examples](https://github.com/hdinsight/hdinsight-storm-examples) 存储库中提供了这种情况的完整自动化解决方案。若要使用此示例，请遵循 [IoTExample README.MD](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/IotExample/README.md) 中的步骤。

## 后续步骤

如需更多 Storm 拓扑示例，请参阅 [Storm on HDInsight 拓扑示例](/documentation/articles/hdinsight-storm-example-topology/)。

<!---HONumber=Mooncake_1207_2015-->