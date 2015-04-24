<properties
   pageTitle="使用 Storm on HDInsight 从事件中心处理事件 | Azure"
   description="了解如何使用在 Visual Studio 中通过 HDInsight Tools for Visual Studio 创建的 C# Storm 拓扑处理事件中心数据。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="04/06/2015"
    wacn.date="04/15/2015"
    />



# 使用 Storm on HDInsight 从 Azure 事件中心处理事件 (C#)

Azure 事件中心可让你处理网站、应用程序和设备中的大量数据。借助事件中心 Spout，你可以轻松使用 Apache Storm on HDInsight 实时分析这些数据。你还可以使用事件中心 Bolt 从 Storm 向事件中心写入数据。 

在本文档中，你将学习如何使用 HDInsight Tools for Visual Studio 和事件中心 Spout 和 Bolt 创建两个混合 C#/Java 拓扑：

* **EventHubWriter** - 随机生成数据，并将其写入事件中心

* **EventHubReader** - 从事件中心读取数据，并将其存储在 Azure 表存储中

## 先决条件

* 一个 <a href="/documentation/articles/hdinsight-storm-getting-started/" target="_blank">Apache Storm on HDInsight 群集</a>

* 一个 <a href="/documentation/articles/service-bus-event-hubs-csharp-ephcs-getstarted/" target="_blank">Azure 事件中心</a>

* <a href="http://www.windowsazure.cn/downloads" target="_blank">Azure .NET SDK</a>

* <a href="/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/" target="_blank">HDInsight Tools for Visual Studio</a>

## 已完成的项目

你可以从 [https://github.com/Blackmist/hdinsight-storm-examples/tree/csharpeventhub/CSharpEventHub](https://github.com/Blackmist/hdinsight-storm-examples/tree/csharpeventhub/CSharpEventHub) 下载本文中创建的项目的完整版本；但是，你仍然需要按照本文档中的步骤提供配置设置。

## 事件中心 Spout 和 Bolt

事件中心 Spout 和 Bolt 是可让你轻松从 Apache Storm 使用事件中心的 Java 组件。虽然这些组件是用 Java 语言编写的，但 HDInsight Tools for Visual Studio 允许你创建混用 C# 和 Java 组件的混合拓扑。

Spout 和 Bolt 以名为 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 的单个 Java 存档 (.jar) 文件的形式分发。

### 下载 Jar

<a href="https://github.com/hdinsight/hdinsight-storm-examples" target="_blank">https://github.com/hdinsight/hdinsight-storm-examples</a> 项目中 **lib** 文件夹下包含了最新版本的 **eventhubs-storm-Spout-0.9-jar-with-dependencies.jar** 文件。若要下载该文件，请使用以下方法之一。

> [WACOM.NOTE] 已提交 Spout 和 Bolt 以包含在 Apache Storm 项目中。有关详细信息，请参阅<a href="https://github.com/apache/storm/pull/336/files">提取请求</a>。

* **下载 ZIP 文件** - 在 <a href="https://github.com/hdinsight/hdinsight-storm-examples" target="_blank">https://github.com/hdinsight/hdinsight-storm-examples</a> 站点中，选择"下载 ZIP"按钮以下载包含项目的 .zip 文件。

	![download zip button](./media/hdinsight-storm-develop-csharp-event-hub-topology/download.png)

	下载后，你可以解压缩存档，该文件位于 **lib** 目录中。

* **克隆项目** - 如果你已安装 <a href="http://git-scm.com/" target="_blank">Git</a>，请使用以下命令在本地克隆存储库，然后查找 **lib** 目录中的文件。

		git clone https://github.com/hdinsight/hdinsight-storm-examples

## 配置 Event Hub

事件中心是此示例的数据源。按照下列步骤创建一个新的 Event Hub。

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，选择"新建" |"Service Bus" |"事件中心" |"自定义创建"。

2. 在"添加新事件中心"对话框中，输入"事件中心名称"，选择要在其中创建中心的"区域"，然后创建新的命名空间或选择现有的命名空间。最后，单击**箭头**按钮。

	![wizard page 1](./media/hdinsight-storm-develop-csharp-event-hub-topology/wiz1.png)

	> [WACOM.NOTE] 应该选择与 Storm on HDInsight 服务器相同的**位置**，以降低延迟和成本。

2. 在"配置事件中心"对话框中，输入"分区计数"和"消息保留期"值。对于本示例，请使用分区计数 10，消息保留期 1。记下分区计数，因为稍后需要用到。

	![wizard page 2](./media/hdinsight-storm-develop-csharp-event-hub-topology/wiz2.png)

3. 创建事件中心后，选择命名空间，然后选择"事件中心"。最后，选择前面创建的事件中心。

4. 选择"配置"，然后使用以下信息创建两个新的访问策略。

	<table>
	<tr><th>名称</th><th>权限</th></tr>
	<tr><td>writer</td><td>发送</td></tr>
	<tr><td>reader</td><td>侦听</td></tr>
	</table>

	创建权限后，在页面底部选择"保存"图标。这将会创建共享访问策略，用于对此事件中心进行发送 (writer) 和侦听 (reader)。

	![policies](./media/hdinsight-storm-develop-csharp-event-hub-topology/policy.png)

5. 保存策略后，使用页面底部的"共享访问密钥生成器"检索 **writer** 和 **reader** 策略的密钥。保存这些密钥，因为稍后将要用到。

## 配置表存储

表存储用于保存从事件中心读取的值，你可以轻松地在 Visual Studio 中通过"服务器资源管理器"查看表存储。使用以下步骤创建新的表存储。

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，选择"新建" |"数据服务" |"存储" |"快速创建"。

	![quick create storage](./media/hdinsight-storm-develop-csharp-event-hub-topology/storagecreate.png)

2. 输入存储帐户的名称，选择一个位置，然后单击复选标记以创建存储帐户。

	> [WACOM.NOTE] 应该选择与事件中心和 Storm on HDInsight 服务器相同的**位置**，以降低延迟和成本。

3. 设置新的存储帐户后，请选择该帐户，然后使用页面底部的"管理访问密钥"链接检索"存储帐户名称"和"主访问密钥"。保存此信息，因为稍后将要用到。

	![access keys](./media/hdinsight-storm-develop-csharp-event-hub-topology/managekeys.png)

## 创建 EventHubWriter

在本部分中，你将要使用事件中心 Bolt 创建向事件中心写入数据的拓扑。

1. 如果你尚未安装最新版本的 HDInsight Tools for Visual Studio，请参阅<a href="/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/" target="_blank">开始使用 HDInsight Tools for Visual Studio</a>。

2. 打开 Visual Studio，并依次选择"文件"、"新建"和"项目"。

3. 在"新建项目"对话框中，依次展开"已安装"和"模板"，然后选择"HDInsight"。从模板列表中，选择"Storm 应用程序"。在对话框底部，输入 **EventHubWriter** 作为应用程序名称。

	![image](./media/hdinsight-storm-develop-csharp-event-hub-topology/newproject.png)

4. 创建项目后，你应该会获得以下文件：

	* **Program.cs** - 定义项目的拓扑。请注意，默认情况下会创建包含一个 Spout 和一个 Bolt 的默认拓扑

	* **Spout.cs** - 示例 Spout

	* **Bolt.cs** - 示例 Bolt稍后需要删除此文件，因为你要使用事件中心 Bolt 向事件中心写入数据

### 配置

1. 在"解决方案资源管理器"中，右键单击"EventHubWriter"，然后选择"属性"。

2. 在项目属性中，选择"设置"，然后选择"此项目不包含默认的设置文件。单击此处可创建一个"。

3. 输入以下设置。在"值"列中使用前面创建的事件中心的信息。

	<table>
	<tr><th style="text-align:left">名称</th><th style="text-align:left">类型</th><th style="text-align:left">范围</th></tr>
	<tr><td style="text-align:left">EventHubPolicyName</td><td style="text-align:left">字符串</td><td style="text-align:left">应用程序</td></tr>
	<tr><td style="text-align:left">EventHubPolicyKey</td><td style="text-align:left">字符串</td><td style="text-align:left">应用程序</td></tr>
	<tr><td style="text-align:left">EventHubNamespace</td><td style="text-align:left">字符串</td><td style="text-align:left">应用程序</td></tr>
	<tr><td style="text-align:left">EventHubName</td><td style="text-align:left">字符串</td><td style="text-align:left">应用程序</td></tr>
	<tr><td style="text-align:left">EventHubPartitionCount</td><td style="text-align:left">int</td><td style="text-align:left">应用程序</td></tr>
	</table>

4. 保存并关闭属性页。

### 定义拓扑

1. 在"解决方案资源管理器"中，右键单击"Bolt.cs"并选择"删除"。由于你使用的是 Java 事件中心 Bolt，因此不需要此文件。

2. 打开 **Program.cs** 文件，并紧接在  `TopologyBuilder topologyBuilder = new TopologyBuilder("EventHubWriter");` 行后添加以下内容。

		int partitionCount = Properties.Settings.Default.EventHubPartitionCount;
		List<string> javaDeserializerInfo =
            new List<string>() { "microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer", "java.lang.String" };

	第一行从前面定义的属性中读取分区计数。第二行定义一个反序列化程序，用于将 Spout 生成的 JSON 数据反序列化为  `java.lang.String`，使事件中心 Bolt 可以使用数据。

4. 找到以下代码。

		topologyBuilder.SetSpout(
            "Spout",
            Spout.Get,
            new Dictionary<string, List<string>>()
            {
                {Constants.DEFAULT_STREAM_ID, new List<string>(){"count"}}
            },
            1);

	将它替换为以下代码。

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
	
	此外，还会声明输出流应使用前面创建的反序列化程序。

5. 紧接在上述代码的后面添加以下代码。

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

6. 找到以下代码。

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            1).shuffleGrouping("Spout");

	将它替换为以下代码。

		topologyBuilder.SetJavaBolt(
            "EventHubBolt",
            constructor,
            partitionCount).
            shuffleGrouping("Spout");

	这会指示拓扑使用上述步骤中的 **JavaComponentConstructor** 作为 Bolt。可以在此拓扑中使用友好名称"EventHubBolt"引用该组件。并行度提示设置为事件集线器的分区数，它订阅 Spout（"Spout"）生成的数据。

此时，你已完成了 **Program.cs**。已经定义了拓扑，但现在，你必须修改 **Spout.cs**，使它能够以事件中心 Bolt 可以使用的格式生成数据。

### 修改 Spout

事件中心 Bolt 需要单个字符串值，该值将路由到事件中心。在以下示例中，你将要修改默认的 **Spout.cs** 文件以生成 JSON 字符串。

1. 在"解决方案资源管理器"中，右键单击"EventHubWriter"项目，然后单击"管理 NuGet 包"。搜索 **Json.Net** 包，然后将它添加到解决方案。这样，我们便可以轻松创建将要使用 Bolt 发送到事件中心的 JSON 数据。

1. 打开 **Spout.cs**，在文件顶部添加以下内容。

		using Newtonsoft.Json;
		using Newtonsoft.Json.Linq;

	这样，我们便可以更轻松地使用 JSON 数据。

3. 找到以下代码。

		Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        outputSchema.Add("default", new List<Type>() { typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));

	将它替换为以下代码。

		Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));
        this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer());

	这会更改 Spout 创建的数据定义，以使用**字符串**数据和**自定义 JSON 序列化程序**。

2. 将 **NextTuple** 方法替换为以下内容。

		public void NextTuple(Dictionary<string, Object> parms)
        {
            JObject eventData = new JObject();
            eventData.Add("deviceId", r.Next(10));
            eventData.Add("deviceValue", r.Next());
            ctx.Emit(new Values(eventData.ToString(Formatting.None)));
        }

	这将随机生成一个设备 ID 和一个值，并使用 Json.net 发出使用这些值的 JSON 对象。

3. 保存 **Spout.cs** 文件。

此时，你已创建了一个基本拓扑，该拓扑将生成随机数据，并使用事件中心 Bolt 将其存储在事件中心。接下来，你要创建读取器。

## 创建 EventHubReader

在本部分中，你将要使用事件中心 Spout 创建从事件中心读取数据的拓扑。

2. 打开 Visual Studio，并依次选择"文件"、"新建"和"项目"。

3. 在"新建项目"对话框中，依次展开"已安装"和"模板"，然后选择"HDInsight"。从模板列表中，选择"Storm 应用程序"。在对话框底部，输入 **EventHubReader** 作为应用程序名称。

### 配置

1. 在"解决方案资源管理器"中，右键单击"EventHubReader"，然后选择"属性"。

2. 在项目属性中，选择"设置"，然后选择"此项目不包含默认的设置文件。单击此处可创建一个"。

3. 输入以下设置。在"值"列中使用前面创建的事件中心和存储帐户的信息。

	<table>
	<tr><th style="text-align:left">名称</th><th style="text-align:left">类型</th><th style="text-align:left">范围</th></tr>
	<tr><th style="text-align:left">EventHubPolicyName</th><th style="text-align:left">字符串</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">EventHubPolicyKey</th><th style="text-align:left">字符串</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">EventHubNamespace</th><th style="text-align:left">字符串</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">EventHubName</th><th style="text-align:left">字符串</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">EventHubPartitionCount</th><th style="text-align:left">int</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">StorageConnection</th><th style="text-align:left">（连接字符串）</th><th style="text-align:left">应用程序</th></tr>
	<tr><th style="text-align:left">TableName</th><th style="text-align:left">字符串</th><th style="text-align:left">应用程序</th></tr>
	</table>

	对于 **TableName**，请输入要在其中存储事件的表的名称。

	拓扑将使用这些值来与事件中心和表存储通信。

4. 保存并关闭属性页。

### 定义拓扑

1. 在"解决方案资源管理器"中，右键单击"Spout.cs"并选择"删除"。由于你使用的是 Java 事件中心 Spout，因此不需要此文件。

2. 打开 **Program.cs** 文件，并紧接在  `TopologyBuilder topologyBuilder = new TopologyBuilder("EventHubReader");` 行后添加以下内容。

		int partitionCount = Properties.Settings.Default.EventHubPartitionCount;
		JavaComponentConstructor constructor = JavaComponentConstructor.CreateFromClojureExpr(
            String.Format(@"(com.microsoft.eventhubs.spout.EventHubSpout. (com.microsoft.eventhubs.spout.EventHubSpoutConfig. " +
                @"""{0}"" ""{1}"" ""{2}"" ""{3}"" {4} ""{5}""))",
                Properties.Settings.Default.EventHubPolicyName,
                Properties.Settings.Default.EventHubPolicyKey,
                Properties.Settings.Default.EventHubNamespace,
                Properties.Settings.Default.EventHubName,
                partitionCount,
                "")); //Last value is the zookeeper connection string - leave empty

	将读入分区计数并将其分配给本地变量，因为它将被多次使用。

	 `JavaComponentConstructor` 定义在运行时如何构造 Java Spout。在这种情况下，你将要通过 <a href="http://storm.apache.org/documentation/Clojure-DSL.html" target="_blank">Apache Storm Clojure DSL</a> 使用前面添加的事件中心配置信息来配置 Spout。更具体地说，HDInsight 在运行时将使用此代码执行以下操作：

	* 使用你提供的事件中心信息创建 **com.microsoft.eventhubs.spout.EventHubSpoutConfig** 的新实例。
	
	* 创建 **com.microsoft.eventhubs.spout.EventHubSpout** 的新实例并传入 **EventHubSpoutConfig** 实例。

5. 找到以下代码。

		topologyBuilder.SetSpout(
            "Spout",
            Spout.Get,
            new Dictionary<string, List<string>>()
            {
                {Constants.DEFAULT_STREAM_ID, new List<string>(){"count"}}
            },
            1);

	将它替换为以下代码。

        topologyBuilder.SetJavaSpout(
            "EventHubSpout",
            constructor,
            partitionCount);

	这会指示拓扑使用上述步骤中的 **JavaComponentConstructor** 作为 Spout，并将其名称指定为"EventHubSpout"。此外，这还会将此组件的并行度提示设置为事件中心的分区数。 

2. 紧接在上述代码的后面添加以下内容。

         List<string> javaSerializerInfo = new List<string>() { "microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer" };

	这会创建一个自定义序列化程序，用于将 Java Spout 生成的信息序列化成为 JSON 格式。

3. 找到以下代码。

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            1).shuffleGrouping("Spout");

	将它替换为以下代码。

		topologyBuilder.SetBolt(
            "Bolt",
            Bolt.Get,
            new Dictionary<string, List<string>>(),
            partitionCount).
            DeclareCustomizedJavaSerializer(javaSerializerInfo).
            shuffleGrouping("EventHubSpout");

	此代码指示拓扑使用某个 Bolt（在 Bolt.cs 中定义）。新增的代码指示拓扑对此组件使用的数据使用自定义序列化程序，该序列化程序来自 **EventHubSpout** - Java Spout。

此时，你已完成了 **Program.cs**。已经定义了拓扑，但现在，你必须创建一个帮助器类，以将数据写入表存储，然后，必须修改 **Bolt.cs**，以便它可以理解 Spout 生成的数据。

### 创建帮助器类

将数据写入表存储时，你必须创建一个类来描述要写入的数据。

1. 在"解决方案资源管理器"中，右键单击"EventHubReader"项目，然后依次选择"添加"和"新建类"。将新类命名为 **Devices.cs**。

2. 打开 **Devices.cs**，将默认代码替换为以下代码。

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

1. 在"解决方案资源管理器"中，右键单击"EventHubReader"项目，然后单击"管理 NuGet 包"。搜索 **Json.Net** 包，然后将它添加到解决方案。这样，我们便可以轻松处理从 Spout 接收的 JSON 数据。此外，请添加 **Microsoft Azure 存储空间**包，以便能够写入表存储。

1. 打开 **Bolt.cs**，在该文件的顶部添加以下内容。

		using Newtonsoft.Json.Linq;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Table;

	这样，我们便可以更轻松地处理来自 Bolt 的 JSON 数据并将数据写入表存储。

2. 找到 `private int count;` 语句，并将它替换为以下内容。

        private CloudTable table;

	连接到表时将使用这些代码。

4. 找到以下代码。

		Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        inputSchema.Add("default", new List<Type>() { typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));

	将它替换为以下代码。

		Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        inputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
        this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());

	这会指示 Bolt 接收**字符串**值而不是 **int** 值，并且应该使用 **CustomizedInteropJSONDeserialzer** 反序列化数据。

3. 紧接在上述代码的后面添加以下代码。

		CloudStorageAccount storageAccount = CloudStorageAccount.Parse(Properties.Settings.Default.StorageConnection);
        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
        table = tableClient.GetTableReference(Properties.Settings.Default.TableName);
        table.CreateIfNotExists();

	这将使用前面配置的连接字符串连接到 **events** 表。如果该表不存在，将创建该表。

2. 找到 **Execute** 方法，并将它替换为以下内容。

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
            }
        }

	这会使用 Json.net 分析来自 Spout 的 JSON 数据，然后找出 **deviceId** 和 **deviceValue** 字段。然后，在初始化期间，将使用 **deviceId** 创建新的 **Device** 对象，以设置表的分区键。然后，将值设置为 **deviceValue**，最后，将实体插入表中。

此时，你已完成了一个从事件中心读取数据，并将数据存储在表存储中名为 **events** 的表内的拓扑。

## 部署拓扑

1. 打开 **EventHubReader** 解决方案。在"解决方案资源管理器"中，右键单击"EventHubReader"项目，然后选择"提交到 Storm on HDInsight"。

	![submit to storm](./media/hdinsight-storm-develop-csharp-event-hub-topology/submittostorm.png)

2. 在"提交拓扑"对话框中，选择你的"Storm 群集"。展开"其他配置"，选择"Java 文件路径"，选择"..."，然后选择前面下载的 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 所在的目录。最后，选择"提交"。

	![Image of submission dialog](./media/hdinsight-storm-develop-csharp-event-hub-topology/submit.png)

3. 提交拓扑之后，将会出现"Storm 拓扑查看器"。在对话框左侧选择 **EventHubReader** 拓扑，以查看该拓扑的统计信息。当前应该不会发生任何情况，因为尚未将任何事件写入事件中心。

	![example storage view](./media/hdinsight-storm-develop-csharp-event-hub-topology/topologyviewer.png)

4. 打开 **EventHubWriter** 解决方案。在"解决方案资源管理器"中，右键单击"EventHubWriter"项目，然后选择"提交到 Storm on HDInsight"。

2. 在"提交拓扑"对话框中，选择你的"Storm 群集"。展开"其他配置"，选择"Java 文件路径"，选择"..."，然后选择前面下载的 **eventhubs-storm-spout-0.9-jar-with-dependencies.jar** 所在的目录。最后，选择"提交"。

5. 提交拓扑后，在"Storm 拓扑查看器"中刷新拓扑列表，以检查这两个拓扑是否在群集上运行。

6. 如果两者都在运行，请选择"服务器资源管理器"，然后依次展开"Azure"、"存储"和前面创建的存储帐户。在存储帐户下，展开"表"。最后，双击 **events** 表以打开该表。你应会看到，来自 **EventHubReader** 拓扑的数据已存储在该表中。

	* **EventHubWriter** 拓扑正在生成事件，并将这些事件写入事件中心。

	* 然后，**EventHubReader** 将从事件中心读取事件，并将其存储在表存储中的 **events** 表内。

## 停止拓扑

若要停止拓扑，请在"Storm 拓扑查看器"中选择每个拓扑，然后选择"终止"。

![image of killing a topology](./media/hdinsight-storm-develop-csharp-event-hub-topology/killtopology.png)

## 摘要

在本文档中，你已学习如何使用 C# 拓扑中的 Java 事件中心 Spout 和 Bolt 处理 Azure 事件中心内的数据。若要了解有关创建 C# 拓扑的详细信息，请参阅以下主题。

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)

* [HDInsight Storm 示例](https://github.com/hdinsight/hdinsight-storm-examples)








<!--HONumber=50-->