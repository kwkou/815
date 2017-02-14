<properties
    pageTitle="为 Apache Storm 开发基于 Java 的拓扑 | Azure"
    description="了解如何通过创建一个简单的单词计数拓扑，来以 Java 语言创建一个 Storm 拓扑。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />  

<tags
    ms.assetid="a8838f29-9c08-4fd9-99ef-26655d1bf6d7"
    ms.service="hdinsight"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="11/18/2016"
    wacn.date="12/12/2016"
    ms.author="larryfr" />

# 使用 Apache Storm 和 HDInsight 上的 Maven 为基本的单词计数应用程序开发基于 Java 的拓扑

了解如何使用 Maven 为 HDInsight 上的 Apache Storm 创建基于 Java 的拓扑。本文将会演练使用 Maven 和 Java（如果拓扑是在 Java 中定义的）创建基本单词计数应用程序的过程。然后，介绍如何使用 Flux 框架定义拓扑。

> [AZURE.NOTE]
Storm 0.10.0 或更高版本中提供了 Flux 框架。HDInsight 3.3 随附了 Storm 0.10.0。


完成本文档中的步骤之后，将获得用于部署到 Apache Storm on HDInsight 的基本拓扑。

> [AZURE.NOTE]
[https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) 上提供了本文档中所创建的拓扑的完成版本。


## 先决条件

* [Java 开发人员工具包 (JDK) 版本 7](https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

* [Maven (https://maven.apache.org/download.cgi)](https://maven.apache.org/download.cgi)：Maven 是 Java 项目的项目生成系统。

* 文本编辑器或 IDE。

## 配置环境变量

可以在安装 Java 和 JDK 时设置以下环境变量。但应检查其是否存在并且包含相关系统的适当值。

* **JAVA\_HOME** - 应指向已安装 Java 运行时环境 (JRE) 的目录。例如，在 Windows 中，其值类似于 `c:\Program Files (x86)\Java\jre1.7`

* **PATH** - 应该包含以下路径：
  
    * **JAVA\_HOME**（或等效路径）

    * **JAVA\_HOME\\bin**（或等效路径）

    * 安装 Maven 的目录

## 创建新的 Maven 项目

在命令行中，使用以下代码创建名为 **WordCount** 的新 Maven 项目：

    mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

这会在当前位置创建名为 **WordCount** 的新目录，其中包含基本 Maven 项目。

**WordCount** 目录将包含以下项：

* **pom.xml**：包含 Maven 项目的设置。
* **src\\main\\java\\com\\microsoft\\example**：包含应用程序代码。
* **src\\test\\java\\com\\microsoft\\example**：包含应用程序的测试。对于本示例，我们将不创建测试。

### 删除示例代码

由于我们要创建应用程序，因此请删除生成的测试和应用程序文件：

* **src\\test\\java\\com\\microsoft\\example\\AppTest.java**
* **src\\main\\java\\com\\microsoft\\example\\App.java**

## 添加属性

Maven 允许定义项目级的值，称为属性。在 `<url>http://maven.apache.org</url>` 行的后面添加以下内容：

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!--
        Storm 0.10.0 is for HDInsight 3.3 and 3.4.
        To find the version information for earlier HDInsight cluster
        versions, see /documentation/articles/hdinsight-component-versioning-v1/
        -->
        <storm.version>0.10.0</storm.version>
    </properties>

现在，可以在其他部分中使用这些值。例如，在指定 Storm 组件的版本时，可以使用 `${storm.version}` 而无需将值硬编码。

## 添加依赖项

由于这是 Storm 拓扑，因此必须添加 Storm 组件的依赖项。打开 **pom.xml**，并在 **&lt;dependencies>** 节中添加以下代码：

    <dependency>
      <groupId>org.apache.storm</groupId>
      <artifactId>storm-core</artifactId>
      <version>${storm.version}</version>
      <!-- keep storm out of the jar-with-dependencies -->
      <scope>provided</scope>
    </dependency>

在编译时，Maven 会使用此信息来查找 Maven 存储库中的 **storm-core**。它会先查找本地计算机上的存储库。如果文件不存在，它会从公共 Maven 存储库下载这些文件，并将其存储在本地存储库中。

> [AZURE.NOTE]
请注意我们在该节中添加的 `<scope>provided</scope>` 行。这会告诉 Maven 从我们创建的任何 JAR 文件中排除 **storm-core**，因为系统将会予以提供。这样，便可以稍微减小所创建的包，并确保它们使用 Storm on HDInsight 群集中包含的 **storm-core** 位。

## 生成配置

Maven 插件可让你自定义项目的生成阶段，例如，如何编译项目，或者如何将它打包成 JAR 文件。打开 **pom.xml**，并紧靠在 `</project>` 行的上方添加以下代码。

    <build>
      <plugins>
      </plugins>
      <resources>
      </resources>
    </build>

此节用于添加插件、资源和其他生成配置选项。有关 **pom.xml** 文件的完整参考信息，请参阅 [http://maven.apache.org/pom.html](http://maven.apache.org/pom.html)。

### 添加插件

针对 Storm 拓扑，[Exec Maven 插件](http://www.mojohaus.org/exec-maven-plugin/)十分有用，因为它可让用户轻松地在开发环境本地运行拓扑。将以下内容添加至 **pom.xml** 文件的 `<plugins>` 节，以包括 Exec Maven 插件：

    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>1.4.0</version>
      <executions>
        <execution>
        <goals>
          <goal>exec</goal>
        </goals>
        </execution>
      </executions>
      <configuration>
        <executable>java</executable>
        <includeProjectDependencies>true</includeProjectDependencies>
        <includePluginDependencies>false</includePluginDependencies>
        <classpathScope>compile</classpathScope>
        <mainClass>${storm.topology}</mainClass>
      </configuration>
    </plugin>

> [AZURE.NOTE]
请注意，`<mainClass>` 项使用 `${storm.topology}`。我们并未（但本应当）事先在 properties 节中定义此值。 不过，在稍后的步骤中，将在开发环境中运行拓扑时从命令行设置此值。

另一个有用的插件是用于更改编译选项的 [Apache Maven Compiler 插件](http://maven.apache.org/plugins/maven-compiler-plugin/)。我们需要此插件的主要原因是要更改 Maven 用作应用程序源和目标的 Java 版本。

在 **pom.xml** 的 `<plugins>` 节中添加以下内容，以包括 Apache Maven Compiler 插件并将源和目标版本设置为 1.7。

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.3</version>
      <configuration>
        <source>1.7</source>
        <target>1.7</target>
      </configuration>
    </plugin>

### 配置资源

使用 resources 节可以包含非代码资源，例如拓扑中组件所需的配置文件。本示例将在 **pom.xml** 文件的 `<resources>` 节中添加以下内容。

    <resource>
        <directory>${basedir}/resources</directory>
        <filtering>false</filtering>
        <includes>
          <include>log4j2.xml</include>
        </includes>
    </resource>

这会将项目根目录 (`${basedir}`) 中的 resources 目录添加为包含资源的位置，并包含名为 **log4j2.xml** 的文件。此文件用于配置拓扑所要记录的信息。

## 创建拓扑

基于 Java 的 Storm 拓扑包含必须编写（或引用）为依赖项的三个组件。

* **Spout**：读取外部源中的数据，并发出进入拓扑的数据流。

* **Bolt**：对 Spout 或其他 Bolt 所发出的数据流执行处理，并发出一个或多个数据流。

* **拓扑**：定义如何排列 Spout 和 Bolt，并提供拓扑的入口点。

### 创建 Spout

为了降低设置外部数据源的要求，以下 Spout 只会发出随机句子。它是 [Storm-Starter 示例](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter)随附的 Spout 的修改版本。

> [AZURE.NOTE]
有关从外部数据源读取的 Spout 的示例，请参阅以下示例之一：
><p> 
><p> * [TwitterSampleSPout](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter/spout/TwitterSampleSpout.java)：从 Twitter 读取数据的示例 spout

对于 Spout，在 **src\\main\\java\\com\\microsoft\\example** 目录中创建名为 **RandomSentenceSpout.java** 的新文件，并使用以下内容做为内容：

    package com.microsoft.example;

    import org.apache.storm.spout.SpoutOutputCollector;
    import org.apache.storm.task.TopologyContext;
    import org.apache.storm.topology.OutputFieldsDeclarer;
    import org.apache.storm.topology.base.BaseRichSpout;
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Values;
    import org.apache.storm.utils.Utils;

    import java.util.Map;
    import java.util.Random;

    //This spout randomly emits sentences
    public class RandomSentenceSpout extends BaseRichSpout {
      //Collector used to emit output
      SpoutOutputCollector _collector;
      //Used to generate a random number
      Random _rand;

      //Open is called when an instance of the class is created
      @Override
      public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
      //Set the instance collector to the one passed in
        _collector = collector;
        //For randomness
        _rand = new Random();
      }

      //Emit data to the stream
      @Override
      public void nextTuple() {
      //Sleep for a bit
        Utils.sleep(100);
        //The sentences that will be randomly emitted
        String[] sentences = new String[]{ "the cow jumped over the moon", "an apple a day keeps the doctor away",
            "four score and seven years ago", "snow white and the seven dwarfs", "i am at two with nature" };
        //Randomly pick a sentence
        String sentence = sentences[_rand.nextInt(sentences.length)];
        //Emit the sentence
        _collector.emit(new Values(sentence));
      }

      //Ack is not implemented since this is a basic example
      @Override
      public void ack(Object id) {
      }

      //Fail is not implemented since this is a basic example
      @Override
      public void fail(Object id) {
      }

      //Declare the output fields. In this case, an sentence
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
      }
    }

请花费片刻时间通读代码注释，以了解此 Spout 的工作原理。

> [AZURE.NOTE]
虽然此拓扑只使用一个 Spout，但其他拓扑可能存在将数据从不同源送入拓扑的多个 Spout。

### 创建 Bolt

Bolt 用于处理数据。此拓扑有两个 Bolt：

* **SplitSentence**：将 **RandomSentenceSpout** 发出的句子分割成不同的单词。

* **WordCount**：统计每个单词的出现次数。

> [AZURE.NOTE]
Bolt 几乎可以执行任何操作，例如，计算、保存，或者与外部组件通信。

在 **src\\main\\java\\com\\microsoft\\example** 目录中创建两个新文件：**SplitSentence.java** 和 **WordCount.Java**。将以下内容用作这些文件的内容：

**SplitSentence**

    package com.microsoft.example;

    import java.text.BreakIterator;

    import org.apache.storm.topology.BasicOutputCollector;
    import org.apache.storm.topology.OutputFieldsDeclarer;
    import org.apache.storm.topology.base.BaseBasicBolt;
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Tuple;
    import org.apache.storm.tuple.Values;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class SplitSentence extends BaseBasicBolt {

      //Execute is called to process tuples
      @Override
      public void execute(Tuple tuple, BasicOutputCollector collector) {
        //Get the sentence content from the tuple
        String sentence = tuple.getString(0);
        //An iterator to get each word
        BreakIterator boundary=BreakIterator.getWordInstance();
        //Give the iterator the sentence
        boundary.setText(sentence);
        //Find the beginning first word
        int start=boundary.first();
        //Iterate over each word and emit it to the output stream
        for (int end=boundary.next(); end != BreakIterator.DONE; start=end, end=boundary.next()) {
          //get the word
          String word=sentence.substring(start,end);
          //If a word is whitespace characters, replace it with empty
          word=word.replaceAll("\\s+","");
          //if it's an actual word, emit it
          if (!word.equals("")) {
            collector.emit(new Values(word));
          }
        }
      }

      //Declare that emitted tuples will contain a word field
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
      }
    }

**WordCount**

    package com.microsoft.example;

    import java.util.HashMap;
    import java.util.Map;
    import java.util.Iterator;

    import org.apache.storm.Constants;
    import org.apache.storm.topology.BasicOutputCollector;
    import org.apache.storm.topology.OutputFieldsDeclarer;
    import org.apache.storm.topology.base.BaseBasicBolt;
    import org.apache.storm.tuple.Fields;
    import org.apache.storm.tuple.Tuple;
    import org.apache.storm.tuple.Values;
    import org.apache.storm.Config;

    // For logging
    import org.apache.logging.log4j.Logger;
    import org.apache.logging.log4j.LogManager;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class WordCount extends BaseBasicBolt {
      //Create logger for this class
      private static final Logger logger = LogManager.getLogger(WordCount.class);
      //For holding words and counts
      Map<String, Integer> counts = new HashMap<String, Integer>();
      //How often we emit a count of words
      private Integer emitFrequency;

      // Default constructor
      public WordCount() {
          emitFrequency=5; // Default to 60 seconds
      }

      // Constructor that sets emit frequency
      public WordCount(Integer frequency) {
          emitFrequency=frequency;
      }

      //Configure frequency of tick tuples for this bolt
      //This delivers a 'tick' tuple on a specific interval,
      //which is used to trigger certain actions
      @Override
      public Map<String, Object> getComponentConfiguration() {
          Config conf = new Config();
          conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, emitFrequency);
          return conf;
      }

      //execute is called to process tuples
      @Override
      public void execute(Tuple tuple, BasicOutputCollector collector) {
        //If it's a tick tuple, emit all words and counts
        if(tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
                && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)) {
          for(String word : counts.keySet()) {
            Integer count = counts.get(word);
            collector.emit(new Values(word, count));
            logger.info("Emitting a count of " + count + " for word " + word);
          }
        } else {
          //Get the word contents from the tuple
          String word = tuple.getString(0);
          //Have we counted any already?
          Integer count = counts.get(word);
          if (count == null)
            count = 0;
          //Increment the count and store it
          count++;
          counts.put(word, count);
        }
      }

      //Declare that we will emit a tuple containing two fields; word and count
      @Override
      public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word", "count"));
      }
    }

请花费片刻时间通读代码注释，以了解每个 Bolt 的工作原理。

### 定义拓扑

拓扑将 Spout 和 Bolt 一起绑定到图形，该图形定义了组件之间的数据流动方式。它还提供 Storm 在群集内创建组件的实例时使用的并行度提示。

以下是此拓扑的组件的基本原理图。

![显示 Spout 和 Bolt 排列方式的示意图](./media/hdinsight-storm-develop-java-topology/wordcount-topology.png)

若要实现该拓扑，请在 **src\\main\\java\\com\\microsoft\\example** 目录中创建名为 **WordCountTopology.java** 的新文件。将以下内容用作该文件的内容：

    package com.microsoft.example;

    import org.apache.storm.Config;
    import org.apache.storm.LocalCluster;
    import org.apache.storm.StormSubmitter;
    import org.apache.storm.topology.TopologyBuilder;
    import org.apache.storm.tuple.Fields;

    import com.microsoft.example.RandomSentenceSpout;

    public class WordCountTopology {

      //Entry point for the topology
      public static void main(String[] args) throws Exception {
      //Used to build the topology
        TopologyBuilder builder = new TopologyBuilder();
        //Add the spout, with a name of 'spout'
        //and parallelism hint of 5 executors
        builder.setSpout("spout", new RandomSentenceSpout(), 5);
        //Add the SplitSentence bolt, with a name of 'split'
        //and parallelism hint of 8 executors
        //shufflegrouping subscribes to the spout, and equally distributes
        //tuples (sentences) across instances of the SplitSentence bolt
        builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");
        //Add the counter, with a name of 'count'
        //and parallelism hint of 12 executors
        //fieldsgrouping subscribes to the split bolt, and
        //ensures that the same word is sent to the same instance (group by field 'word')
        builder.setBolt("count", new WordCount(), 12).fieldsGrouping("split", new Fields("word"));

        //new configuration
        Config conf = new Config();
        //Set to false to disable debug information when
        // running in production on a cluster
        conf.setDebug(false);

        //If there are arguments, we are running on a cluster
        if (args != null && args.length > 0) {
          //parallelism hint to set the number of workers
          conf.setNumWorkers(3);
          //submit the topology
          StormSubmitter.submitTopology(args[0], conf, builder.createTopology());
        }
        //Otherwise, we are running locally
        else {
          //Cap the maximum number of executors that can be spawned
          //for a component to 3
          conf.setMaxTaskParallelism(3);
          //LocalCluster is used to run locally
          LocalCluster cluster = new LocalCluster();
          //submit the topology
          cluster.submitTopology("word-count", conf, builder.createTopology());
          //sleep
          Thread.sleep(10000);
          //shut down the cluster
          cluster.shutdown();
        }
      }
    }

请花片刻时间通读代码注释以了解拓扑的定义方式，然后将拓扑提交到群集。

### 配置日志记录

Storm 使用 Apache Log4j 来记录信息。如果未配置日志记录，拓扑将发出许多难以阅读的诊断信息。若要控制所记录的信息，请在 __resources__ 目录中创建名为 __log4j2.xml__ 的文件。将以下内容用作该文件的内容。

    <?xml version="1.0" encoding="UTF-8"?>
    <Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.microsoft.example" level="trace" additivity="false">
            <AppenderRef ref="STDOUT"/>
        </Logger>
        <Root level="error">
            <Appender-Ref ref="STDOUT"/>
        </Root>
    </Loggers>
    </Configuration>

这将为 **com.microsoft.example** 类（包含本示例拓扑的组件）配置一个新记录器。此记录器的级别设置为“跟踪”，可以捕获此拓扑中的组件发出的任何日志记录信息。回顾此项目的代码，会发现只有 WordCount.java 文件实施日志记录 - 它会记录每个单词的计数。

`<Root level="error">` 节将日志记录的根级别（不在 **com.microsoft.example** 中的所有内容）配置为只记录错误信息。

> [AZURE.IMPORTANT]
尽管这可以大幅减少在开发环境中测试拓扑时所记录的信息，但不会删除在生产群集上运行时生成的所有调试信息。若要减少此类信息，还必须在提交到群集的配置中将调试设置为 false。有关示例，请参阅本文档中的 WordCountTopology.java 代码。

有关为 Log4j 配置日志记录的详细信息，请参阅 [http://logging.apache.org/log4j/2.x/manual/configuration.html](http://logging.apache.org/log4j/2.x/manual/configuration.html)。

> [AZURE.NOTE]
Storm 0.10.0 版及更高版本使用 Log4j 2.x。早期版本的 Storm 使用 Log4j 1.x（为日志配置使用的格式不同）。有关旧配置的信息，请参阅 [http://wiki.apache.org/logging-log4j/Log4jXmlFormat](http://wiki.apache.org/logging-log4j/Log4jXmlFormat)。

## 在本地测试拓扑

保存文件之后，请使用以下命令在本地测试拓扑。

    mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology

运行该命令时，拓扑会显示启动信息。然后开始显示与下面类似的行，因为句子是从 Spout 发出，然后由 Bolt 处理的。

    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
    17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word snow

从 WordCount Bolt 发出的日志中可以看到，“and”已发出 113 次。只要拓扑运行，计数就会持续增加，因为 Spout 会连续发出相同的句子。

此外，每两次发出单词和句子的间隔为 5 秒。之所以发生这种情况，是因为 **WordCount** 组件配置为仅当计时周期元组到达时才发出信息，并且要求只能每隔默认 5 秒传送此类元组一次。

## 将拓扑转换为 Flux

Flux 是 Storm 0.10.0 及更高版本随附的一个新框架，可以将配置和实现分离开来。组件（Bolt 和 Spout）仍在 Java 中定义，但拓扑是使用 YAML 文件定义的。

YAML 文件定义要用于拓扑的组件、如何在组件之间流送数据，以及在初始化组件时要使用哪些值。可以在部署项目时将一个 YAML 文件添加为包含项目的 jar 文件的一部分，或者在启动拓扑时使用外部 YAML 文件。

1. 将 **WordCountTopology.java** 文件移出项目。以前，此文件定义拓扑，但现在我们不会将它用于 Flux。

2. 在 **resources** 目录中，创建名为 **topology.yaml** 的新文件。在此文件中使用以下内容。
    
        # topology definition

        # name to be used when submitting. This is what shows up...
        # in the Storm UI/storm command-line tool as the topology name
        # when submitted to Storm
        name: "wordcount"

        # Topology configuration
        config:
        # Hint for the number of workers to create
        topology.workers: 1

        # Spout definitions
        spouts:
        - id: "sentence-spout"
            className: "com.microsoft.example.RandomSentenceSpout"
            # parallelism hint
            parallelism: 1

        # Bolt definitions
        bolts:
        - id: "splitter-bolt"
            className: "com.microsoft.example.SplitSentence"
            parallelism: 1

        - id: "counter-bolt"
            className: "com.microsoft.example.WordCount"
            constructorArgs:
            - 10
            parallelism: 1

        # Stream definitions
        streams:
        - name: "Spout --> Splitter" # name isn't used (placeholder for logging, UI, etc.)
            # The stream emitter
            from: "sentence-spout"
            # The stream consumer
            to: "splitter-bolt"
            # Grouping type
            grouping:
            type: SHUFFLE

        - name: "Splitter -> Counter"
            from: "splitter-bolt"
            to: "counter-bolt"
            grouping:
            type: FIELDS
            # field(s) to group on
            args: ["word"]

    请花费片刻时间通读并了解每个节的作用，以及它与 **WordCountTopology.java** 文件中基于 Java 的定义存在怎样的关联。

3. 对 **pom.xml** 文件进行以下更改。
   
    * 在 `<dependencies>` 节中添加以下新依赖关系：
     
            <!-- Add a dependency on the Flux framework -->
            <dependency>
                <groupId>org.apache.storm</groupId>
                <artifactId>flux-core</artifactId>
                <version>${storm.version}</version>
            </dependency>

    * 将以下插件添加到 `<plugins>` 节。此插件处理项目包（jar 文件）的创建，并在创建包时应用一些特定于 Flux 的转换。
     
            <!-- build an uber jar -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <transformers>
                        <!-- Keep us from getting a "can't overwrite file error" -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                        <!-- We're using Flux, so refer to it as main -->
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>org.apache.storm.flux.Flux</mainClass>
                        </transformer>
                    </transformers>
                    <!-- Keep us from getting a bad signature error -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

    * 在 **exec-maven-plugin** `<configuration>` 节中，将 `<mainClass>` 的值更改为 `org.apache.storm.flux.Flux`。这样，在开发环境本地运行拓扑时，Flux 便可以处理这种运行。

    * 将以下内容添加到 `<resources>` 节中的 `<includes>`。这样就加入了用于将拓扑定义为项目一部分的 YAML 文件。
     
            <include>topology.yaml</include>

## 在本地测试 Flux 拓扑

1. 使用以下命令通过 Maven 编译并执行 Flux 拓扑。
   
        mvn compile exec:java -Dexec.args="--local -R /topology.yaml"
   
    如果使用的是 PowerShell，请使用以下命令：
   
        mvn compile exec:java "-Dexec.args=--local -R /topology.yaml"
   
    运行该命令时，拓扑会显示启动信息。然后开始显示与下面类似的行，因为句子是从 Spout 发出，然后由 Bolt 处理的。
   
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
        17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
        17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
        17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
   
    记录的信息批之间存在 10 秒延迟，因为在创建 WordCount 组件时，`topology.yaml` 文件会传递 `10` 值。这会将计时周期元组的延迟间隔设置为 10 秒。

2. 从项目创建 `topology.yaml` 文件的副本。为其指定类似于 `newtopology.yaml` 的名称。在该文件中，找到以下节，将 `10` 的值更改为 `5`。这会将发出单词计数批的间隔时间从 10 秒更改为 5 秒。
   
         - id: "counter-bolt"
           className: "com.microsoft.example.WordCount"
           constructorArgs:
           - 5
           parallelism: 1

3. 若要运行拓扑，请使用以下命令：
   
        mvn exec:java -Dexec.args="--local /path/to/newtopology.yaml"
   
    将 `/path/to/newtopology.yaml` 更改为前一步骤中创建的 newtopology.yaml 文件的路径。此命令将使用 newtopology.yaml 作为拓扑定义。由于没有包含 `compile` 参数，Maven 将重复使用前面步骤中生成的项目的版本。
   
    启动拓扑后，应会发现发出批的间隔时间已发生更改，反映 newtopology.yaml 中的值。因此可以看到，无需重新编译拓扑即可通过 YAML 文件更改配置。

Flux 还提供其他一些功能（不过本文没有介绍），例如，根据运行时传递的参数替换 YAML 文件中的变量，或者通过环境变量替换变量。有关 Flux 框架的上述功能和其他功能的详细信息，请参阅 [Flux (https://storm.apache.org/releases/0.10.0/flux.html)](https://storm.apache.org/releases/0.10.0/flux.html)。

## Trident

Trident 是 Storm 提供的高级抽象。它支持有状态处理。Trident 的主要优点在于，它可以保证进入拓扑的每个消息只会处理一次。这在保证消息至少处理一次的原始 Java 拓扑中很难实现。两者还有其他方面的差异，例如，可以使用内置组件，而无需创建 Bolt。事实上，可以使用低泛型组件（例如筛选、投影和函数）来完全取代 Bolt。

你可以使用 Maven 项目来创建 Trident 应用程序。使用本文前面所述的相同基本步骤 - 只有代码不同。Trident（目前）还不能与 Flux 框架配合使用。

有关 Trident 的详细信息，请参阅 [Trident API 概述](http://storm.apache.org/documentation/Trident-API-Overview.html)。

## 后续步骤

你已学习如何使用 Java 创建 Storm 拓扑。接下来，请学习如何：

* [在 HDInsight 上部署和管理 Apache Storm 拓扑](/documentation/articles/hdinsight-storm-deploy-monitor-topology/)

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)

如需更多 Storm 拓扑示例，请访问 [Storm on HDInsight 拓扑示例](/documentation/articles/hdinsight-storm-example-topology/)。

<!---HONumber=Mooncake_1205_2016-->