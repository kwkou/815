<properties
   pageTitle="使用 Storm on HDInsight 从事件中心处理事件 | Azure"
   description="了解如何使用在 Visual Studio 中通过 HDInsight Tools for Visual Studio 创建的 C# Storm 拓扑处理事件中心数据。"
   services="hdinsight,notification hubs"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="05/10/2016"
	wacn.date="06/06/2016"/>

# 使用 Storm on HDInsight 从 Azure 事件中心处理事件 (C#)

Azure 事件中心可让你处理网站、应用程序和设备中的大量数据。借助事件中心 Spout，你可以轻松使用 Apache Storm on HDInsight 实时分析这些数据。你还可以使用事件中心 Bolt 从 Storm 向事件中心写入数据。

在本教程中，你将学习如何使用连同 HDInsight Tools for Visual Studio 一起安装的 Visual Studio 模板，来创建两个可以配合 Azure 事件中心运行的拓扑。

* **EventHubWriter**：随机生成数据，并将其写入事件中心

* **EventHubReader**：从事件中心读取数据，并将其存储在 Azure 表存储中

## 先决条件

* 一个 [Apache Storm on HDInsight 群集](/documentation/articles/hdinsight-apache-storm-tutorial-get-started/)

* 一个 [Azure 事件中心](/documentation/articles/event-hubs-csharp-ephcs-getstarted/)

* [Azure .NET SDK](/downloads/)

* [HDInsight Tools for Visual Studio](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)

## 已完成的项目

你可以从 GitHub 下载本教程中所创建的项目的完整版本：[eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub)。不过，你仍然必须根据本教程中的步骤提供配置设置。

> [AZURE.NOTE] 在使用已完成的项目时，你必须使用 **NuGet 包管理器**来还原此解决方案所需的程序包。

## 事件中心 Spout 和 Bolt

事件中心 Spout 和 Bolt 是可让你轻松从 Apache Storm 使用事件中心的 Java 组件。虽然这些组件是用 Java 语言编写的，但 HDInsight Tools for Visual Studio 允许你创建混用 C# 和 Java 组件的混合拓扑。

Spout 和 Bolt 以名为 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 的单个 Java 存档 (.jar) 文件的形式分发。

### 下载 .jar 文件

[HDInsight Storm 示例](https://github.com/hdinsight/hdinsight-storm-examples)项目中 **lib** 文件夹下包含了最新版本的 **eventhubs-storm-Spout-0.9-jar-with-dependencies.jar** 文件。若要下载该文件，请使用以下方法之一。

> [AZURE.NOTE] 已提交 Spout 和 Bolt 以包含在 Apache Storm 项目中。有关详细信息，请参阅 GitHub 中的 [STORM-583: Initial check-in for storm-event hubs（STORM-583：Storm 事件中心的初始签入）](https://github.com/apache/storm/pull/336/files)。

* **下载 ZIP 文件**：在 [HDInsight Storm 示例](https://github.com/hdinsight/hdinsight-storm-examples)站点中，选择右窗格中的“下载 ZIP”来下载包含项目的 .zip 文件。

	![下载 zip 按钮](./media/hdinsight-storm-develop-csharp-event-hub-topology/download.png)

	下载文件后，你可以解压缩存档，该文件位于 **lib** 目录中。

* **克隆项目**：如果你已安装 [Git](http://git-scm.com/)，请使用以下命令在本地克隆存储库，然后查找 **lib** 目录中的文件。

		git clone https://github.com/hdinsight/hdinsight-storm-examples

## 配置事件中心

事件中心是此示例的数据源。按照下列步骤创建一个新的事件中心。

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，选择“新建”>“应用程序服务”>“服务总线”>“事件中心”>“自定义创建”。

2. 在“添加新事件中心”屏幕中，输入“事件中心名称”，选择要在其中创建中心的“区域”，然后创建新的命名空间或选择现有的命名空间。单击**箭头**继续。

	![向导页 1](./media/hdinsight-storm-develop-csharp-event-hub-topology/wiz1.png)

	> [AZURE.NOTE] 应该选择与 Storm on HDInsight 服务器相同的**位置**，以降低延迟和成本。

2. 在“配置事件中心”屏幕中，输入“分区计数”和“消息保留期”值。对于本示例，请使用分区计数 8，消息保留期 1。记下分区计数，因为稍后需要用到。

3. 创建事件中心之后，请选择命名空间，选择“事件中心”，然后选择你前面创建的事件中心。

4. 选择“配置”，然后使用以下信息创建两个新的访问策略。

	| 名称 | 权限 |
    | ----- | ----- |
	| 写入器 | 发送 |
	| 读取器 | 侦听 |

	创建权限后，在页面底部选择“保存”图标。这将会创建共享访问策略，用于对此事件中心进行发送 (writer) 和侦听 (reader)。

	![策略](./media/hdinsight-storm-develop-csharp-event-hub-topology/policy.png)

5. 保存策略后，使用页面底部的“共享访问密钥生成器”检索 **writer** 和 **reader** 策略的密钥。保存这些密钥，因为稍后将要用到。

## 配置表存储

表存储用于保存从事件中心读取的值，你可以轻松地在 Visual Studio 中通过“服务器资源管理器”查看表存储。使用以下步骤创建新的表存储：

1. 在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，选择“新建”>“数据服务”>“存储”>“快速创建”。

	![快速创建存储](./media/hdinsight-storm-develop-csharp-event-hub-topology/storagecreate.png)

2. 输入存储帐户的**名称**，选择一个**位置**，然后单击**复选标记**以创建存储帐户。

	> [AZURE.NOTE] 应该选择与事件中心和 Storm on HDInsight 服务器相同的**位置**，以降低延迟和成本。

3. 创建新的存储帐户后，请选择该帐户，然后使用页面底部的“管理访问密钥”链接检索“存储帐户名称”和“主访问密钥”。保存此信息，因为稍后将要用到。

	![访问密钥](./media/hdinsight-storm-develop-csharp-event-hub-topology/managekeys.png)

4. 打开 Visual Studio。在“视图”菜单中，选择“云资源管理器”。在“云资源管理器”中，展开“存储帐户”，然后选择之前创建的存储帐户。

    ![云资源管理器](./media/hdinsight-storm-develop-csharp-event-hub-topology/createtablestorage.png)

5. 右键单击存储帐户对应的“表”，然后选择“创建表”。出现提示时，请输入 **events** 作为表的名称。保存该名称，因为需要在后续步骤中用到。

## 创建 EventHubWriter

在本部分中，你将要使用事件中心 Bolt 创建向事件中心写入数据的拓扑。

1. 如果你尚未安装最新版本的 HDInsight Tools for Visual Studio，请参阅 [Get started using HDInsight Tools for Visual Studio（开始使用 HDInsight Tools for Visual Studio）](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

2. 打开 Visual Studio，选择“文件”>“新建”>“项目”。

3. 在“新建项目”屏幕中，展开“已安装”>“模板”，然后选择“HDInsight”。从模板列表中，选择“Storm 应用程序”。在屏幕底部，输入 **EventHubWriter** 作为应用程序名称。

	![图像](./media/hdinsight-storm-develop-csharp-event-hub-topology/newproject.png)

4. 创建项目后，你应该会获得以下文件：

	* **Program.cs**：定义项目的拓扑。请注意，默认情况下会创建包含一个 Spout 和一个 Bolt 的默认拓扑。

	* **Spout.cs**：示例 Spout。

	* **Bolt.cs**：示例 Bolt。稍后需要删除此文件，因为你要使用事件中心 Bolt 向事件中心写入数据

### 配置

1. 在“解决方案资源管理器”中，右键单击“EventHubWriter”，然后选择“属性”。

2. 在项目属性中，选择“设置”，然后选择“此项目不包含默认的设置文件。单击此处可创建一个”。

3. 输入以下设置。在“值”列中使用前面创建的事件中心的信息。

	| Name | 类型 | 范围 |
    | ----- | ----- | ----- |
	| EventHubPolicyName | 字符串 | 应用程序 |
	| EventHubPolicyKey | 字符串 | 应用程序 |
	| EventHubNamespace | 字符串 | 应用程序 |
	| EventHubName | 字符串 | 应用程序 |
	| EventHubPartitionCount | int | 应用程序 |

4. 保存并关闭“属性”页。

### 定义拓扑

1. 在“解决方案资源管理器”中，右键单击“Bolt.cs”并选择“删除”。由于你使用的是 Java 事件中心 Bolt，因此不需要此文件。

2. 打开 **Program.cs** 文件，并紧接在 `TopologyBuilder topologyBuilder = new TopologyBuilder("EventHubWriter" + DateTime.Now.ToString("yyyyMMddHHmmss"));` 行后添加以下内容。

		int partitionCount = Properties.Settings.Default.EventHubPartitionCount;
		List<string> javaDeserializerInfo =
            new List<string>() { "microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer", "java.lang.String" };

	第一行从前面定义的属性中读取分区计数。第二行定义一个反序列化程序，用于将 Spout 生成的 JSON 数据反序列化为 `java.lang.String`，使 Java 组件可以使用数据。

4. 找到以下代码：

		topologyBuilder.SetSpout(
            "Spout",
            Spout.Get,
            new Dictionary<string, List<string>>()
            {
                {Constants.DEFAULT_STREAM_ID, new List<string>(){"count"}}
            },
            1);

	将它替换为以下代码：

		topologyBuilder.SetSpout(
            "Spout",
            Spout.Get,
            new Dictionary<string, List<string>>()
            {
                {Constants.DEFAULT_STREAM_ID, new List<string>(){"Event"}}
            },
            partitionCount).
            DeclareCustomizedJavaDeserializer(javaDeserializerInfo);

	这将会创建一个 Spout，并使用事件中心分区计数作为此组件的并行度提示。这还应该为每个分区创建 Spout 的实例。

	这还会将前面创建的反序列化程序与此组件的输出流相关联。这样，下游 EventHubSpout 组件便可以使用 C# Spout 生成的数据。

5. 紧接在上述代码的后面添加以下代码：

		JavaComponentConstructor constructor =
            JavaComponentConstructor.CreateFromClojureExpr(
            String.Format(@"(com.microsoft.eventhubs.bolt.EventHubBolt. (com.microsoft.eventhubs.bolt.EventHubBoltConfig. " +
            @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
            Properties.Settings.Default.EventHubPolicyName,
            Properties.Settings.Default.EventHubPolicyKey,
            Properties.Settings.Default.EventHubNamespace,
            "servicebus.chinacloudapi.cn", //suffix for servicebus fqdn
            Properties.Settings.Default.EventHubName,
			"true"));

	这将为 Java Bolt 创建一个新的构造函数，在运行时，将使用此构造函数配置 Bolt 的新实例。在这种情况下，你将要通过 <a href="http://storm.apache.org/documentation/Clojure-DSL.html" target="_blank">Apache Storm Clojure DSL</a> 使用前面添加的事件中心配置信息来配置 Spout。更具体地说，HDInsight 在运行时将使用此代码执行以下操作：

	* 使用你提供的事件中心信息创建 **com.microsoft.eventhubs.bolt.EventHubBoltConfig** 的新实例。
	* 创建 **com.microsoft.eventhubs.bolt.EventHubBolt** 的新实例并传入 **EventHubBoltConfig** 实例。

6. 找到以下代码：

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            1).shuffleGrouping("Spout");

	将它替换为以下代码：

		topologyBuilder.SetJavaBolt(
            "EventHubBolt",
            constructor,
            partitionCount).
            shuffleGrouping("Spout");

	这会指示拓扑使用上述步骤中的 **JavaComponentConstructor** 作为 Bolt。可以在此拓扑中使用友好名称“EventHubBolt”引用该组件。 并行度提示设置为事件集线器的分区数，它订阅 Spout（“Spout”）生成的数据。

此时，你已完成了 **Program.cs**。已经定义了拓扑，但现在，你必须修改 **Spout.cs**，使它能够以事件中心 Bolt 可以使用的格式生成数据。

> [AZURE.NOTE] 此拓扑将默认为创建一个工作进程，这足以满足示例目的。如果要针对生产群集改写此拓拟，应添加以下代码以更改工作进程数目：

    StormConfig config = new StormConfig();
    config.setNumWorkers(1);
    topologyBuilder.SetTopologyConfig(config);


### 修改 Spout

事件中心 Bolt 需要单个字符串值，该值将路由到事件中心。在以下示例中，你将要修改默认的 **Spout.cs** 文件以生成 JSON 字符串。

1. 在“解决方案资源管理器”中，打开“Spout.cs”，在该文件的顶部添加以下内容：

		using Newtonsoft.Json;
		using Newtonsoft.Json.Linq;

	这样，我们便可以更轻松地使用 JSON 数据。
    
    > [AZURE.NOTE] JSON.NET 包应已安装，因为它是用于 C# Storm 拓扑的 SCP.NET 框架所必需的。

3. 找到以下代码：

		Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        outputSchema.Add("default", new List<Type>() { typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));

	将它替换为以下代码：

		Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));
        this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer());

	这会更改 Spout 创建的数据定义，以使用**字符串**数据以及前面在拓扑中声明的 **CustomizedInteropJSONSerializer**（在 program.cs 中）。

2. 将 **NextTuple** 方法替换为以下内容：

		public void NextTuple(Dictionary<string, Object> parms)
        {
            JObject eventData = new JObject();
            eventData.Add("deviceId", r.Next(10));
            eventData.Add("deviceValue", r.Next());
            ctx.Emit(new Values(eventData.ToString(Formatting.None)));
        }

	这将随机生成一个设备 ID 和一个值，并使用 Json.NET 发出使用这些值的 JSON 对象。

3. 保存 **Spout.cs** 文件。

此时，你已创建了一个基本拓扑，该拓扑将生成随机数据，并使用事件中心 Bolt 将其存储在事件中心。接下来，你要创建读取器。

## 创建 EventHubReader

在本部分中，你将要使用事件中心 Spout 创建从事件中心读取数据的拓扑。

2. 打开 Visual Studio，选择“文件”>“新建”>“项目”。

3. 在“新建项目”屏幕中，展开“已安装”>“模板”，然后选择“HDInsight”。从模板列表中，选择“Storm 应用程序”。在屏幕底部，输入 **EventHubReader** 作为应用程序名称。

### 配置

1. 在“解决方案资源管理器”中，右键单击“EventHubReader”，然后选择“属性”。

2. 依次选择“工具”、“NuGet 包管理器”、“包管理器控制台”。当控制台出现时，请使用以下命令来安装 Azure 存储空间包。

        NuGet install WindowsAzure.Storage

2. 在项目属性中，选择“设置”，然后选择“此项目不包含默认的设置文件。单击此处可创建一个”。

3. 输入以下设置。在“值”列中使用前面创建的事件中心和存储帐户的信息。

	| Name | 类型 | 范围 |
    | ----- | ----- | ----- |
	| EventHubPolicyName | 字符串 | 应用程序 |
	| EventHubPolicyKey | 字符串 | 应用程序 |
	| EventHubNamespace | 字符串 | 应用程序 |
	| EventHubName | 字符串 | 应用程序 |
	| EventHubPartitionCount | int | 应用程序 |
	| StorageConnection | （连接字符串） | 应用程序 |
	| TableName | 字符串 | 应用程序 |

	对于 **TableName**，请输入要在其中存储事件的表的名称。

    对于 **StorageConnection**，请输入值 `DefaultEndpointsProtocol=https;AccountName=myAccount;AccountKey=myKey;`。将 **myAccount** 和 **myKey** 分别替换为前面获取的存储帐户名和密钥。

	拓扑将使用这些值来与事件中心和表存储通信。

4. 保存并关闭“属性”页。

### 定义拓扑

1. 在“解决方案资源管理器”中，右键单击“Spout.cs”并选择“删除”。由于你使用的是 Java 事件中心 Spout，因此不需要此文件。

2. 打开 **Program.cs** 文件，并紧接在 `TopologyBuilder topologyBuilder = new TopologyBuilder("EventHubReader" + DateTime.Now.ToString("yyyyMMddHHmmss"));` 行后添加以下代码：

		int partitionCount = Properties.Settings.Default.EventHubPartitionCount;
		EventHubSpoutConfig ehConfig = new EventHubSpoutConfig(
                Properties.Settings.Default.EventHubPolicyName,
                Properties.Settings.Default.EventHubPolicyKey,
                Properties.Settings.Default.EventHubNamespace,
                Properties.Settings.Default.EventHubName,
                partitionCount);

	将读取分区计数并将其分配到本地变量。该计数将多次使用。

	`EventHubSpoutConfig` 定义事件中心 Spout 的配置。在本例中，此为你在前面添加的事件中心配置信息。此代码在幕后使用 Java 事件中心 Spout，并使用事件中心信息创建 **com.microsoft.eventhubs.spout.EventHubSpoutConfig** 的新实例。

5. 找到以下代码：

		topologyBuilder.SetSpout(
            "Spout",
            Spout.Get,
            new Dictionary<string, List<string>>()
            {
                {Constants.DEFAULT_STREAM_ID, new List<string>(){"count"}}
            },
            1);

	将它替换为以下代码：

        topologyBuilder.SetEventHubSpout(
            "EventHubSpout", 
            ehConfig, 
            partitionCount); 

	这会指示拓扑创建新的事件中心 Spout 并使用前一个步骤的 `EventHubSpoutConfig` 作为配置。“EventHubSpout”设置 Spout 的友好名称，`partitionCount` 用于设置并行度提示。此代码在幕后使用提供的配置信息来创建 **com.microsoft.eventhubs.Spout.EventHubSpout** Java 组件的新实例。

2. 紧接在上述代码的后面添加以下内容：

         List<string> javaSerializerInfo = new List<string>() { "microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer" };

	这创建一个自定义的序列化程序，可用来将 Java 组件（例如 ）生成的信息序列化为下游 C# 组件可使用的 JSON 格式。

3. 找到以下代码：

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            1).shuffleGrouping("Spout");

	将它替换为以下代码：

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            partitionCount,
            true).
            DeclareCustomizedJavaSerializer(javaSerializerInfo).
            shuffleGrouping("EventHubSpout");

	此代码指示拓扑使用某个 Bolt（在 Bolt.cs 中定义）。此处使用前面定义的自定义序列化程序，以便此 Bolt 可使用上游 Java 组件生成的数据。在此情况下为 EventHubSpout。

    > [AZURE.IMPORTANT] SetBolt 的最后一个参数（值为 `true`）启用此 Bolt 的 ACK 功能。这是必需的 EventHubSpout 组件需要它会发出的数据的 ACK。如果下游组件不会返回确认，Spout 将停止处理大约 1000 个消息后接收。

此时，你已完成了 **Program.cs**。已经定义了拓扑，但现在，你必须创建一个帮助器类，以将数据写入表存储，然后，必须修改 **Bolt.cs**，以便它可以理解 Spout 生成的数据。

> [AZURE.NOTE] 此拓扑将默认为创建一个工作进程，这足以满足示例目的。如果要针对生产群集改写此拓拟，应添加以下代码以更改工作线程数目：

    StormConfig config = new StormConfig();
    config.setNumWorkers(1);
    topologyBuilder.SetTopologyConfig(config);


### 创建帮助器类

将数据写入表存储时，你必须创建一个类来描述要写入的数据。

1. 在“解决方案资源管理器”中，右键单击“EventHubReader”项目，然后依次选择“添加”和“类”。将新类命名为 **Device.cs**。

2. 打开 **Device.cs**，将默认代码替换为以下代码：

		using System;
		using System.Collections.Generic;
		using System.Linq;
		using System.Text;
		using System.Threading.Tasks;
		using Microsoft.WindowsAzure.Storage.Table;

		namespace EventHubReader
		{
		    class Device : TableEntity
		    {
		        public int value { get; set; }

		        public Device() { }
		        public Device(int id)
		        {
		            this.PartitionKey = id.ToString();
		            this.RowKey = System.Guid.NewGuid().ToString();
		        }
		    }
		}

	这将在表存储中创建由分区键（设置为从事件中心读取的设备 ID）、唯一行键和从事件中心读取的值构成的实体。每个实体还有一个时间戳，在表中插入实体时将自动创建该时间戳。

### 修改 Bolt

1. 在“解决方案资源管理器”中，展开 **EventHubReader** 项目，然后打开 **Bolt.cs** 文件。在该文件的顶部，添加以下内容：

		using Newtonsoft.Json.Linq;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Table;

	这样，我们便可以更轻松地处理来自 Bolt 的 JSON 数据并将数据写入表存储。

2. 找到 `private int count;` 语句，并将它替换为以下内容：

        private CloudTable table;

	连接到表时将使用这些代码。

4. 找到以下代码：

		Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        inputSchema.Add("default", new List<Type>() { typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));

	将它替换为以下代码：

		Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        inputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
        this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());

	这会指示 Bolt 接收**字符串**值而不是 **int** 值，并且应该使用前面在拓扑中声明的 **CustomizedInteropJSONDeserialzer**（在 program.cs 文件中）反序列化数据。

3. 紧接在上述代码的后面添加以下代码：

		CloudStorageAccount storageAccount = CloudStorageAccount.Parse(Properties.Settings.Default.StorageConnection);
        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
        table = tableClient.GetTableReference(Properties.Settings.Default.TableName);
        table.CreateIfNotExists();

	随后将连接到你前面使用 `TableName` 中存储的连接字符串所创建的 Azure 存储表。

2. 找到 **Execute** 方法，并将它替换为以下内容：

		public void Execute(SCPTuple tuple)
        {
            Context.Logger.Info("Processing events");
            string eventValue = (string)tuple.GetValue(0);
            if (eventValue != null)
            {
                JObject eventData = JObject.Parse(eventValue);

                Device device = new Device((int)eventData["deviceId"]);
                device.value = (int)eventData["deviceValue"];

                TableOperation insertOperation = TableOperation.Insert(device);

                table.Execute(insertOperation);
                this.ctx.Ack(tuple);
            }
        }

	这会使用 Json.NET 分析来自 Spout 的 JSON 数据，然后找出 **deviceId** 和 **deviceValue** 字段。然后，在初始化期间，将使用 **deviceId** 创建新的 **Device** 对象，以设置表的分区键。然后，将值设置为 **deviceValue**，最后，将实体插入表中。

    该实体插入到表中后，将为元组调用 `Ack()`，以通知 Spout 我们已成功处理了数据。

    > [AZURE.IMPORTANT] EventHubSpout 组件需要下游组件 Bolt 每个元组的 ACK。如果未收到 ACK，EventHubSpout 将假定元组处理失败。

此时，你已完成创建了一个拓扑，该拓扑可从事件中心读取数据，并将数据存储在表存储中的、你前面所创建的表内。

## 部署拓扑

1. 在“解决方案资源管理器”中，右键单击“EventHubReader”项目，然后选择“提交到 Storm on HDInsight”。

	![提交到 Storm](./media/hdinsight-storm-develop-csharp-event-hub-topology/submittostorm.png)

2. 在“提交拓扑”屏幕中，选择你的“Storm 群集”。展开“其他配置”，选择“Java 文件路径”，选择“...”，然后选择前面下载的 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 文件所在的目录。最后，单击“提交”。

	![提交对话框的图像](./media/hdinsight-storm-develop-csharp-event-hub-topology/submit.png)

3. 提交拓扑之后，将会出现“Storm 拓扑查看器”。在左窗格中选择 **EventHubReader** 拓扑，以查看该拓扑的统计信息。当前应该不会发生任何情况，因为尚未将任何事件写入事件中心。

	![示例存储视图](./media/hdinsight-storm-develop-csharp-event-hub-topology/topologyviewer.png)

4. 在“解决方案资源管理器”中，右键单击“EventHubReader”项目，然后选择“提交到 Storm on HDInsight”。

2. 在“提交拓扑”屏幕中，选择你的“Storm 群集”。展开“其他配置”，选择“Java 文件路径”，选择“...”，然后选择前面下载的 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 文件所在的目录。最后，单击“提交”。

5. 提交拓扑后，在“Storm 拓扑查看器”中刷新拓扑列表，以检查这两个拓扑是否在群集上运行。

6. 如果两个拓扑都在运行，请选择“服务器资源管理器”，展开“Azure”>“存储”，然后选择前面创建的存储帐户。在存储帐户下，展开“表”。最后，双击 **events** 表以打开该表。你应会看到，来自 **EventHubReader** 拓扑的数据已存储在该表中。

	* **EventHubWriter** 拓扑正在生成事件，并将这些事件写入事件中心。

	* 然后，**EventHubReader** 将从事件中心读取事件，并将其存储在表存储中的 **events** 表内。

## 停止拓扑

若要停止拓扑，请在“Storm 拓扑查看器”中选择每个拓扑，然后单击“终止”。

![终止拓扑的图像](./media/hdinsight-storm-develop-csharp-event-hub-topology/killtopology.png)

##删除群集

[AZURE.INCLUDE [delete-cluster-warning](../includes/hdinsight-delete-cluster-warning.md)]

## 说明

### 检查点

EventHubSpout 定期检查点其状态为 Zookeeper 节点，将保存当前的偏移量的消息从队列中读取。这样，要开始在以下情况下接收已保存的偏移量处的消息的组件：

* 组件实例失败，并已重新启动。

* 通过添加或删除节点扩大或收缩群集。

* 拓扑已终止并已**使用相同的名称**重新启动。

你还可以将持久性检查点导入和导出到 WASB（HDInsight 群集使用的 Azure 存储。） 用于执行此操作的脚本位于 Storm on HDInsight 上的 **c:\\apps\\dist\\storm-0.9.3.2.2.1.0-2340\\zkdatatool-1.0\\bin** 中。

>[AZURE.NOTE] 路径中的版本号可能不同，因为群集上安装的 Storm 版本将来可能会更改。

此目录中的脚本是：

* **stormmeta\_import.cmd**：将所有 Storm 元数据从群集默认存储容器导入 Zookeeper。

* **stormmeta\_export.cmd**：将所有 Storm 元数据从 Zookeeper 导出到群集默认存储容器。

* **stormmeta\_delete.cmd**：从 Zookeeper 中删除所有 Storm 元数据。

当你需要删除群集，但在将新群集重新联机的情况下想要从中心的当前偏移量恢复处理时，可以使用导出和导入来保存检查点数据。

> [AZURE.NOTE] 由于数据将保存到默认的存储容器，新群集**必须**使用前一群集所用的同一个存储帐户和容器。

## 后续步骤

在本文档中，你已学习如何使用 C# 拓扑中的 Java 事件中心 Spout 和 Bolt 处理 Azure 事件中心内的数据。若要了解有关创建 C# 拓扑的详细信息，请参阅以下主题。

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)

* [Storm on HDInsight 的示例拓扑](/documentation/articles/hdinsight-storm-example-topology/)
 

<!---HONumber=Mooncake_0530_2016-->