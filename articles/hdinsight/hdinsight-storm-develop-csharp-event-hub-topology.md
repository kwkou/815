<properties
    pageTitle="使用 Storm on HDInsight 处理事件中心的事件 | Azure"
    description="了解如何使用在 Visual Studio 中通过 HDInsight Tools for Visual Studio 创建的 C# Storm 拓扑处理事件中心数据。"
    services="hdinsight,notification hubs"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="67f9d08c-eea0-401b-952b-db765655dad0"
    ms.service="hdinsight"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 使用 Storm on HDInsight 从 Azure 事件中心处理事件 (C#)

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

Azure 事件中心可处理网站、应用和设备的大量数据。借助事件中心 Spout，可轻松使用 Apache Storm on HDInsight 实时分析这些数据。还可以使用事件中心 Bolt 从 Storm 向事件中心写入数据。

在本教程中，将学习如何使用随 HDInsight Tools for Visual Studio 一起安装的 Visual Studio 模板，创建两个可以配合 Azure 事件中心运行的拓扑。

* **EventHubWriter**：随机生成数据，并将其写入事件中心
* **EventHubReader**：从事件中心读取数据并将数据记录到 Storm 日志中

> [AZURE.NOTE] 
如需此项目的 Java 版，请参阅[使用 Storm on HDInsight 从 Azure 事件中心处理事件 (Java)](/documentation/articles/hdinsight-storm-develop-java-event-hub-topology/)。

## SCP.NET

这些项目使用 SCP.NET，后者是一个 NuGet 包，方便用户创建适用于 Storm on HDInsight 的 C# 拓扑和组件。

> [AZURE.IMPORTANT]
虽然本文档中的步骤依赖于带 Visual Studio 的 Windows 开发环境，但是也可将编译的项目提交到使用 Linux 的 Storm on HDInsight 群集。__仅在 2016 年 10 月 28 日以后创建的基于 Linux 的群集支持 SCP.NET 拓扑。__

### 群集版本控制

项目所使用的 Microsoft.SCP.Net.SDK NuGet 包必须与安装在 HDInsight 上的 Storm 的主要版本匹配。Storm on HDInsight 版本 3.3 和 3.4 使用 Storm 版本 0.10.x，因此必须对这些群集使用 SCP.NET 版本 0.10.x.x。HDInsight 3.5 使用 Storm 1.0.x.，因此必须对此群集版本使用 SCP.NET 版本 1.0.x.x。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

HDInsight 3.4 及更高版本使用 Mono 运行 C# 拓扑。大多数功能会兼容 Mono，但应查看 [Mono 兼容性](http://www.mono-project.com/docs/about-mono/compatibility/)文档，了解可能的不兼容性。

C# 拓扑还必须针对 .NET 4.5 运行。

## 如何使用事件中心

Microsoft 提供一组 Java 组件，适用于与 Storm 拓扑中的 Azure 事件中心通信。如需包含这些组件的最新版本的 jar 文件，可访问 [https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/)。

> [AZURE.IMPORTANT]
虽然组件是以 Java 编写的，但可通过 C# 拓扑轻松使用它们。

将要使用的组件主要有：

* __EventHubSpout__：从事件中心读取数据。
* __EventHubBolt__：将数据写入事件中心。
* __EventHubSpoutConfig__：用于配置 EventHubSpout。
* __EventHubBoltConfig__：用于配置 EventHubBolt。
* __UnicodeEventDataScheme__：用于配置 Spout，以便从事件中心进行读取时使用 UTF-8 编码。如果不使用此组件，Spout 将默认设置为 String 编码。

### Spout 用法示例

SCP.NET 提供将 EventHubSpout 添加到拓扑的专用方法。与使用泛型方法添加 Java 组件相比，这些方法可以更轻松地添加 Spout。以下示例演示了如何使用 SCP.NET 所提供的 __SetEventHubSpout__ 和 EventHubSpoutConfig 方法创建新的 Spout：

    topologyBuilder.SetEventHubSpout(
        "EventHubSpout",
        new EventHubSpoutConfig(
            // the shared access signature name and key used to read the data
            ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"],
            ConfigurationManager.AppSettings["EventHubSharedAccessKey"],
            // The namespace that contains the Event Hub to read from
            ConfigurationManager.AppSettings["EventHubNamespace"],
            // The Event Hub name to read from
            ConfigurationManager.AppSettings["EventHubEntityPath"],
            // The number of partitions in the Event Hub
            eventHubPartitions),
        // Parallelism hint for this component. Should be set to the partition count.
        eventHubPartitions);

上面的示例创建了名为 __EventHubSpout__ 的全新 Spout 组件，并将其配置为与事件中心通信。请注意，组件的并行度提示设置为事件中心的分区数。因此，Storm 可以为每个分区创建一个组件实例。

> [AZURE.WARNING]
从 2017 年 1 月 1 日开始，使用 SetEventHubSpout 和 EventHubSpoutConfig 方法创建的 Spout 可以在从事件中心读取数据时使用 String 编码。若需使用 UTF-8 编码，请参阅以下示例。

也可在创建 Spout 时使用泛型 JavaCompoentConstructor 方法。以下示例演示如何使用 JavaComponentConstructor 方法创建新的 Spout。它还演示了如何将 Spout 配置为使用 UTF-8 编码而非 String 编码来读取数据：

    // Create an instance of UnicodeEventDataScheme
    var schemeConstructor = new JavaComponentConstructor("com.microsoft.eventhubs.spout.UnicodeEventDataScheme");
    // Create an instance of EventHubSpoutConfig
    var eventHubSpoutConfig = new JavaComponentConstructor(
        "com.microsoft.eventhubs.spout.EventHubSpoutConfig",
        new List<Tuple<string, object>>()
        {
            // the shared access signature name and key used to read the data
            Tuple.Create<string, object>(JavaComponentConstructor.JAVA_LANG_STRING, ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"]),
            Tuple.Create<string, object>(JavaComponentConstructor.JAVA_LANG_STRING, ConfigurationManager.AppSettings["EventHubSharedAccessKey"]),
            // The namespace that contains the Event Hub to read from
            Tuple.Create<string, object>(JavaComponentConstructor.JAVA_LANG_STRING, ConfigurationManager.AppSettings["EventHubNamespace"]),
            // The Event Hub name to read from
            Tuple.Create<string, object>(JavaComponentConstructor.JAVA_LANG_STRING, ConfigurationManager.AppSettings["EventHubEntityPath"]),
            // The number of partitions in the Event Hub
            Tuple.Create<string, object>("int", eventHubPartitions),
            // The encoding scheme to use when reading
            Tuple.Create<string, object>("com.microsoft.eventhubs.spout.IEventDataScheme", schemeConstructor)
        }
        );
    // Create an instance of the spout
    var eventHubSpout = new JavaComponentConstructor(
        "com.microsoft.eventhubs.spout.EventHubSpout",
        new List<Tuple<string, object>>()
        {
            Tuple.Create<string, object>("com.microsoft.eventhubs.spout.EventHubSpoutConfig", eventHubSpoutConfig)
        }
        );
    // Set the spout in the topology
    topologyBuilder.SetJavaSpout("EventHubSpout", eventHubSpout, eventHubPartitions);

> [AZURE.IMPORTANT]
UnicodeEventDataScheme 仅在 9.5 版事件中心组件中提供，该版本可从 [https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/) 获取。

### Bolt 用法示例

必须使用 JavaComponmentConstructor 方法创建 Bolt 的实例。以下示例演示如何创建和配置 EventHubBolt 的新实例：

    //Create constructor for the Java bolt
    JavaComponentConstructor constructor =
        // Use a Clojure expression to create the EventHubBoltCOnfig
        JavaComponentConstructor.CreateFromClojureExpr(
            String.Format(@"(org.apache.storm.eventhubs.bolt.EventHubBolt. (org.apache.storm.eventhubs.bolt.EventHubBoltConfig. " +
            @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
        // The policy name and key used to read from Event Hubs
        ConfigurationManager.AppSettings["EventHubPolicyName"],
        ConfigurationManager.AppSettings["EventHubPolicyKey"],
        // The namespace that contains the Event Hub
        ConfigurationManager.AppSettings["EventHubNamespace"],
        "servicebus.chinacloudapi.cn", //suffix for the namespace fqdn
        // The Evetn Hub Name)
        ConfigurationManager.AppSettings["EventHubName"],
        "true"));

    //Set the bolt
    topologyBuilder.SetJavaBolt(
            "EventHubBolt",
            constructor,
            partitionCount). //Parallelism hint uses partition count
        shuffleGrouping("Spout"); //Consume data from spout

> [AZURE.NOTE]
该示例使用作为字符串传递的 Clojure 表达式，而不是像 Spout 示例那样使用 JavaComponentConstructor 创建单独的 EventHubBoltConfig。任一方法均可使用；请使用自己感觉最合适的方法。

## 下载已完成的项目

可以从 GitHub 下载本教程中创建的项目的完整版本：[eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub)。但是，仍然需要按照本教程中的步骤提供配置设置。

### 先决条件

* 一个 [3\.5 版 Apache Storm on HDInsight 群集](/documentation/articles/hdinsight-apache-storm-tutorial-get-started/)

    > [AZURE.WARNING]
    本文档中使用的示例需要 3.5 版 Storm on HDInsight。在旧式群集上的 Storm 版本与 HDInsight 3.5 随附的 Storm 版本之间，用于核心 Storm 组件的类名存在变化。如需此示例的版本（兼容旧式群集），请参阅 [https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub/releases](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub/releases)。

* [Azure 事件中心](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)

* [Azure .NET SDK](/downloads/)

* [HDInsight Tools for Visual Studio](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)

* Java JDK 1.7 或更高版本，适用于开发环境。[http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 上提供 JDK 下载内容。

    * **JAVA\_HOME** 环境变量必须指向包含 Java 的目录。
    * **%JAVA\_HOME%/bin** 目录必须位于路径中

## 下载事件中心组件

Spout 和 Bolt 以名为 **eventhubs-storm-spout-#.#-jar-with-dependencies.jar** 的单个 Java 存档 (.jar) 文件的形式分发，其中 #.# 是文件的版本。

兼容 3.5 版 Storm on HDInsight 的 jar 文件版本可从 [https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/lib/eventhubs/) 下载。

创建名为 `eventhubspout` 的新目录，然后将文件保存到该目录中。

## 配置事件中心

事件中心是此示例的数据源。使用[事件中心入门](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)文档的“创建事件中心” 部分中的信息。

1. 创建事件中心后，在 Azure 门户预览中查看“EventHub”边栏选项卡，然后选择“共享访问策略”。使用“+ 添加”条目添加以下策略：
   
    | 名称 | 权限 |
    | --- | --- |
    | writer |发送 |
    | reader |侦听 |
   
    ![策略](./media/hdinsight-storm-develop-csharp-event-hub-topology/sas.png)  


2. 选择“读取器”和“编写器”策略。复制并保存这两个策略的 **PRIMARY KEY** 值，因为会在后面用到这些值。

## 配置 EventHubWriter

1. 如果尚未安装最新版本的 HDInsight Tools for Visual Studio，请参阅 [Get started using HDInsight Tools for Visual Studio](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)（开始使用 HDInsight Tools for Visual Studio）。

2. 从 [eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub) 下载解决方案。打开解决方案，花一些时间查看 **EventHubWriter** 项目的代码。

3. 在 **EventHubWriter** 项目中，打开 **App.config** 文件。使用此前配置的事件中心提供的信息，填充以下项的值：
   
    | 键 | 值 |
    | --- | --- |
    | EventHubPolicyName |编写器（如果曾通过*发送* 权限对策略使用了其他名称，请改用该名称。） |
    | EventHubPolicyKey |编写器策略的项 |
    | EventHubNamespace |包含事件中心的命名空间 |
    | EventHubName |事件中心名称 |
    | EventHubPartitionCount |事件中心的分区数 |

4. 保存并关闭 **App.config** 文件。

## 配置 EventHubReader

1. 打开 **EventHubReader** 项目，花点时间查看代码。

2. 打开 **EventHubWriter** 的 **App.config**。使用此前配置的事件中心提供的信息，填充以下项的值：
   
    | 键 | 值 |
    | --- | --- |
    | EventHubPolicyName |读取器（如果曾通过*侦听* 权限对策略使用了其他名称，请改用该名称。） |
    | EventHubPolicyKey |读取器策略的项 |
    | EventHubNamespace |包含事件中心的命名空间 |
    | EventHubName |事件中心名称 |
    | EventHubPartitionCount |事件中心的分区数 |

3. 保存并关闭 **App.config** 文件。

## 部署拓扑

1. 在“解决方案资源管理器”中，右键单击“EventHubReader”项目，然后选择“提交到 Storm on HDInsight”。
   
    ![提交到 Storm](./media/hdinsight-storm-develop-csharp-event-hub-topology/submittostorm.png)

2. 在“提交拓扑”屏幕上，选择“Storm 群集”。展开“其他配置”，选择“Java 文件路径”，选择“...”，然后选择前面下载的 jar 文件所在的目录。最后，单击“提交”。
   
    ![提交对话框的图像](./media/hdinsight-storm-develop-csharp-event-hub-topology/submit.png)  


3. 提交拓扑后，将会出现“Storm 拓扑查看器”。选择左窗格中的 **EventHubReader** 拓扑，以查看该拓扑的统计信息。目前，不会发生任何情况，因为尚未将任何事件写入事件中心。
   
    ![示例存储视图](./media/hdinsight-storm-develop-csharp-event-hub-topology/topologyviewer.png)

4. 在“解决方案资源管理器”中，右键单击“EventHubReader”项目，然后选择“提交到 Storm on HDInsight”。

5. 在“提交拓扑”屏幕上，选择“Storm 群集”。展开“其他配置”，选择“Java 文件路径”，选择“...”，然后选择前面下载的 jar 文件所在的目录。最后，单击“提交”。

6. 提交拓扑后，在“Storm 拓扑查看器”中刷新拓扑列表，以检查这两个拓扑是否在群集上运行。

7. 在“Storm 拓扑查看器”中，选择“EventHubReader”拓扑。

8. 在图形视图中，双击“LogBolt”组件。此时会打开 Bolt 的“组件摘要”页。

9. 在“执行器”部分，选择“端口”列中的一个链接。此时会显示由组件记录的信息。记录的信息类似于以下内容：
   
        2016-10-20 13:26:44.186 m.s.s.b.ScpNetBolt [INFO] Processing tuple: source: com.microsoft.eventhubs.spout.EventHubSpout:7, stream: default, id: {5769732396213255808=520853934697489134}, [{"deviceId":3,"deviceValue":1379915540}]
        2016-10-20 13:26:44.234 m.s.s.b.ScpNetBolt [INFO] Processing tuple: source: com.microsoft.eventhubs.spout.EventHubSpout:7, stream: default, id: {7154038361491319965=4543766486572976404}, [{"deviceId":3,"deviceValue":459399321}]
        2016-10-20 13:26:44.335 m.s.s.b.ScpNetBolt [INFO] Processing tuple: source: com.microsoft.eventhubs.spout.EventHubSpout:6, stream: default, id: {513308780877039680=-7571211415704099042}, [{"deviceId":5,"deviceValue":845561159}]
        2016-10-20 13:26:44.445 m.s.s.b.ScpNetBolt [INFO] Processing tuple: source: com.microsoft.eventhubs.spout.EventHubSpout:7, stream: default, id: {-2409895457033895206=5479027861202203517}, [{"deviceId":8,"deviceValue":2105860655}]

## 停止拓扑

要停止拓扑，选择“Storm 拓扑查看器”中的每个拓扑，然后单击“终止”。

![终止拓扑的图像](./media/hdinsight-storm-develop-csharp-event-hub-topology/killtopology.png)

## 删除群集

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## 后续步骤

在本文档中，已学习如何使用 C# 拓扑中的 Java 事件中心 Spout 和 Bolt 处理 Azure 事件中心中的数据。要了解有关创建 C# 拓扑的详细信息，请参阅以下主题。

* [使用 Visual Studio 开发适用于 Apache Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)
* [SCP 编程指南](/documentation/articles/hdinsight-storm-scp-programming-guide/)
* [Storm on HDInsight 的示例拓扑](/documentation/articles/hdinsight-storm-example-topology/)

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->