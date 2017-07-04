<properties
    pageTitle="HDInsight 上的示例 Apache Storm 拓扑 | Azure"
    description="使用 Apache Storm on HDInsight 创建和测试的示例 Storm 拓扑列表，包括基本 C# 和 Java 拓扑，以及事件中心的用法。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="f9b1bdff-5928-4705-a76d-52fd200917cb"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/03/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="55053de87f4e3d88d8e42a2d8a34a871698e5fbc"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="example-storm-toplogies-and-components-for-apache-storm-on-hdinsight"></a>Apache Storm on HDInsight 的示例 Storm 拓扑和组件

以下是 Microsoft 创建和维护的、可配合 Apache Storm on HDInsight 使用的示例列表。 这些示例涵盖各种主题，从创建基本 C# 和 Java 拓扑，到使用 Azure 服务（例如事件中心、DocumentDb、Power BI、SQL 数据库、HBase on HDInsight 和 Azure 存储）。 一些示例还演示了如何使用非 Azure 或甚至非 Microsoft 的技术，例如 SignalR 和 Socket.IO

| 说明 | 演示 | 语言/框架 |
|:--- |:--- |:--- |
| [事件中心 Spout 和 Bolt 源](https://github.com/apache/storm/tree/master/external/storm-eventhubs) |事件中心 Spout 和 Bolt 的源 |Java |
| [为 Apache Storm on HDInsight 开发基于 Java 的拓扑][5797064f] |Maven |Java |
| [使用 Visual Studio 为 Apache Storm on HDInsight 开发 C# 拓扑][16fce2d1] |用于 Visual Studio 的 HDInsight 工具 |C#、Java |
| [在 C# Storm 拓扑中创建多个数据流][ec5a4064] |多个流 |C# |
| [使用 Storm on HDInsight 处理 Azure 事件中心的事件 (C#)][844d1d81] |事件中心 |C# 和 Java |
| [使用 Storm on HDInsight 从 Azure 事件中心处理事件 (Java)](/documentation/articles/hdinsight-storm-develop-java-event-hub-topology/) |事件中心 |Java |
| [使用 Power Bi 直观显示 Storm 拓扑中的数据][94d15238] |Power BI |C# |
| [使用 HDInsight 中的 Storm 和 HBase 分析传感器数据][ab894747] |事件中心、HBase、Socket.IO、Web 仪表板 |C#、Java、JavaScript、HTML |
| [使用 Storm on HDInsight 处理事件中心的汽车传感器数据][246ee964] |事件中心、DocumentDb、Azure 存储 Blob (WASB) |C#、Java |
| [使用 Storm on HDInsight 从 Azure 事件中心提取、转换及加载 (ETL) 到 HBase][b4b68194] |事件中心、HBase |C# |
| [通过 Storm on HDInsight 使用模板 C# Storm 拓扑项目来处理 Azure 服务][ce0c02a2] |事件中心、DocumentDb、SQL 数据库、HBase、SignalR |C#、Java |
| [使用 Storm on HDInsight 建立从 Azure 事件中心读取数据的可缩放性基准][d6c540e3] |消息吞吐量、事件中心、SQL 数据库 |C#、Java |
| [使用 HDInsight 上的 Storm 和 HBase 对事件进行关联](/documentation/articles/hdinsight-storm-correlation-topology/) |HBase |C# |
| [将 HDInsight 上的 Python 与 Storm 配合使用](/documentation/articles/hdinsight-storm-develop-python-topology/) |具有 Flux 拓扑的 Python 组件 |Python |

## <a name="next-steps"></a>后续步骤

* [Apache Storm on HDInsight 入门][2b8c3488]
* [了解如何使用 Storm on HDInsight 部署和管理 Storm 拓扑][6eb0d3b8]

[2b8c3488]: /documentation/articles/hdinsight-apache-storm-tutorial-get-started-linux/ "了解如何创建 Storm on HDInsight 群集，以及如何使用 Storm 仪表板部署示例拓扑。"
[6eb0d3b8]: /documentation/articles/hdinsight-storm-deploy-monitor-topology/ "了解如何使用基于 Web 的 Storm 仪表板和 Storm UI 或 Visual Studio 的 HDInsight 工具来部署和管理拓扑。"
[16fce2d1]: /documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/ "了解如何使用 Visual Studio 的 HDInsight 工具创建 C# Storm 拓扑。"
[5797064f]: /documentation/articles/hdinsight-storm-develop-java-topology/ "了解如何通过创建一个基本的单词计数拓扑，使用 Maven 以 Java 语言创建 Storm 拓扑。"
[94d15238]: /documentation/articles/hdinsight-storm-power-bi-topology/ "演示如何从 C# 拓扑将数据写入 Power BI，然后基于这些数据创建图表和仪表板。"
[ec5a4064]: https://github.com/Blackmist/csharp-storm-example "演示一个执行单词计数的基本 Storm 拓扑（以 C# 实现）。此外，还演示如何在一个 C# 拓扑中创建多个数据流。"
[844d1d81]: /documentation/articles/hdinsight-storm-develop-csharp-event-hub-topology/ "了解如何使用 Storm on HDInsight 从 Azure 事件中心读取和写入数据。"
[ab894747]: /documentation/articles/hdinsight-storm-sensor-data-analysis/ "了解如何使用 Apache Storm on HDInsight 处理来自 Azure 事件中心的传感器数据，使用 D3.js 可视化这些数据，然后（可选）将数据存储到 HBase。"
[246ee964]: /documentation/articles/hdinsight-storm-iot-eventhub-documentdb/ "了解如何使用 Storm 拓扑从 Azure 事件中心读取消息，从 Azure DocumentDb 读取数据参考文档，并将数据保存到 Azure 存储。"
[d6c540e3]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/EventCountExample "用于演示使用 Apache Storm on HDInsight 从 Azure 事件中心读取数据以及将数据存储到 SQL 数据库时的吞吐量的多个拓扑。"
[b4b68194]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample "了解如何从 Azure 事件中心读取数据，聚合并转换数据，然后将数据存储到 HBase on HDInsight。"
[ce0c02a2]: https://github.com/hdinsight/hdinsight-storm-examples/tree/master/templates/HDInsightStormExamples "此项目包含用来与各种 Azure 服务（例如事件中心、DocumentDb 和 SQL 数据库）进行交互的 Spout、Bolt 和拓扑的模板。"

<!--Update_Description: wording update-->