<properties
 pageTitle="HDInsight 上的示例 Apache Storm 拓扑 | Microsoft Azure"
 description="使用 Apache Storm on HDInsight 创建和测试的示例 Storm 拓扑列表，包括基本 C# 和 Java 拓扑，以及事件中心的用法。"
 services="hdinsight"
 documentationCenter=""
 authors="Blackmist"
 manager="paulettm"
 editor="cgronlun"/>

<tags
 ms.service="hdinsight"
 ms.date="04/28/2015"
 wacn.date="06/26/2015"/>

# Apache Storm on HDInsight 的示例 Storm 拓扑

以下是 Microsoft 创建和维护的、可配合 Apache Storm on HDInsight 使用的示例列表。这些示例涵盖各种主题，从创建基本 C# 和 Java 拓扑，到使用 Azure 服务（例如事件中心、、Power BI、SQL Database、HBase on HDInsight 和 Azure 存储空间）。一些示例还演示了如何使用非 Azure 或甚至非 Microsoft 的技术，例如 SignalR 和 Socket.IO

| 说明 | 演示 | 语言/框架 |
|:--------------------------------------------------------------------------------------------------------|:-----------------------------------------------------|:---------------------------|
| [为 Apache Storm on HDInsight 开发基于 Java 的拓扑][5797064f] | Maven | Java |
| [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑][16fce2d1] | HDInsight Tools for Visual Studio | C#、Java |
| [在 C# Storm 拓朴中创建多个数据流][ec5a4064] | 多个流 | C#
| [使用 Storm on HDInsight 从 Azure 事件中心处理事件][844d1d81] | 事件中心 | C# 和 Java |
| [使用 Power BI（预览版）从 Storm 拓扑可视化数据][94d15238] | Power BI | C# |
| [使用 HDInsight 中的 Storm 和 HBase 分析传感器数据][ab894747] | 事件中心、HBase、Socket.IO、Web 仪表板 | C#、Java、JavaScript、HTML |
| [使用 Storm on HDInsight 处理事件中心的汽车传感器数据][246ee964] | 事件中心、、Azure 存储 Blob (WASB) | C#、Java |
| [使用 Storm on HDInsight 在 HBase 中从 Azure 事件中心提取、转换及加载 (ETL) 数据][b4b68194] | 事件中心、HBase | C# |
| [通过 Storm on HDInsight 使用模板 C# Storm 拓朴项目来处理 Azure 服务][ce0c02a2] | 事件中心、、SQL Database、HBase、SignalR | C#、Java |
| [使用 Storm on HDInsight 建立从 Azure 事件中心读取数据的可缩放性基准][d6c540e3] | 消息吞吐量、事件中心、SQL Database | C#、Java |

## 后续步骤

* [Apache Storm on HDInsight 入门][2b8c3488]

* [了解如何使用 Storm on HDInsight 部署和管理 Storm 拓扑][6eb0d3b8]

  [2b8c3488]: hdinsight-storm-getting-started "了解如何创建 Storm on HDInsight 群集，以及如何使用 Storm 仪表板来部署示例拓扑。"
  [6eb0d3b8]: hdinsight-storm-deploy-monitor-topology "了解如何使用基于 Web 的 Storm 仪表板和 Storm UI 或 HDInsight Tools for Visual Studio 来部署和管理拓扑。"
  [16fce2d1]: hdinsight-storm-develop-csharp-visual-studio-topology "了解如何使用 HDInsight Tools for Visual Studio 创建 C# Storm 拓扑。"
  [5797064f]: hdinsight-storm-develop-java-topology "了解如何通过创建一个基本的单词计数拓扑，使用 Maven 以 Java 语言创建 Storm 拓扑。"
  [94d15238]: hdinsight-storm-power-bi-topology "演示如何从 C# 拓扑将数据写入 Power BI，然后基于这些数据创建图表和仪表板。"
  [ec5a4064]: https://github.com/Blackmist/csharp-storm-example "演示一个执行单词计数的基本 Storm 拓扑（以 C# 实现）。此外，还演示如何在一个 C# 拓扑中创建多个数据流。"
  [844d1d81]: hdinsight-storm-develop-csharp-event-hub-topology "了解如何使用 Storm on HDInsight 从 Azure 事件中心读取和写入数据。"
  [ab894747]: hdinsight-storm-sensor-data-analysis "了解如何使用 Apache Storm on HDInsight 处理来自 Azure 事件中心的传感器数据，使用 D3.js 可视化这些数据，然后（可选）将数据存储到 HBase。"
  [3c86c7c8]: hdinsight-storm-twitter-trending "了解如何使用 Trident 创建一个 Storm 拓扑，用于确定 Twitter 上的流行主题（基于哈希标记）。"
  [246ee964]: hdinsight-storm-iot-eventhub- "了解如何使用 Storm 拓扑从 Azure 事件中心读取消息，从  读取数据参考文档，并将数据保存到 Azure 存储空间。"
  [d6c540e3]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/EventCountExample "用于演示使用 Apache Storm on HDInsight 从 Azure 事件中心读取数据以及将数据存储到 SQL Database 时的吞吐量的多个拓扑。"
  [b4b68194]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample "了解如何从 Azure 事件中心读取数据，聚合并转换数据，然后将数据存储到 HBase on HDInsight。"
  [ce0c02a2]: https://github.com/hdinsight/hdinsight-storm-examples/tree/master/templates/HDInsightStormExamples "此项目包含用来与各种 Azure 服务（例如事件中心、 和 SQL Database）进行交互的 spout、bolt 和拓扑的模板。"

<!---HONumber=61-->