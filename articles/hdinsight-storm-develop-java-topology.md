<properties
   pageTitle="为 Apache Storm 开发基于 Java 的拓扑 | Azure"
   description="了解如何通过创建一个简单的单词计数拓扑，来以 Java 语言创建一个 Storm 拓扑。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/11/2016"
	wacn.date="05/24/2016"/>

#使用 Apache Storm 和 HDInsight 上的 Maven 为基本的单词计数应用程序开发基于 Java 的拓扑

了解使用 Maven 为 Apache Storm on HDInsight 创建基于 Java 的拓扑的基本过程。将会演练使用 Maven 和 Java 创建基本单词计数应用程序的过程。虽然本文提供了有关使用 Eclipse 的说明，但你也可以使用所选的文本编辑器。

完成本文档中的步骤之后，你将会获得一个用于部署到 Apache Storm on HDInsight 的基本拓扑。

> [AZURE.NOTE]：此拓扑的完整版本在 [https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) 中提供。

##先决条件

* <a href="https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html" target="_blank">Java 开发人员工具包 (JDK) 版本 7</a>

* <a href="https://maven.apache.org/download.cgi" target="_blank">Maven</a>：Maven 是 Java 项目的项目生成系统。

* 文本编辑器，例如记事本、<a href="http://www.gnu.org/software/emacs/" target="_blank">Emacs<a>、<a href="http://www.sublimetext.com/" target="_blank">Sublime Text</a>、<a href="https://atom.io/" target="_blank">Atom.io</a>、<a href="http://brackets.io/" target="_blank">Brackets.io</a>。或者使用集成开发环境 (IDE)，例如 <a href="https://eclipse.org/" target="_blank">Eclipse</a>（Luna 或更高版本）。

	> [AZURE.NOTE] 你的编辑器或 IDE 可能具有处理 Maven 的特定功能，但本文档中未提供说明。有关环境编辑功能的详细信息，请参阅所使用产品的文档。

##配置环境变量

可以在安装 Java 和 JDK 时设置以下环境变量。不过，你应该检查它们是否存在并且包含系统的正确值。

* **JAVA\_HOME** - 应该指向已安装 Java 运行时环境 (JRE) 的目录。例如，在 Windows 中，它的值类似于 `c:\Program Files (x86)\Java\jre1.7`

* **PATH** - 应该包含以下路径：

	* **JAVA\_HOME**（或等效的路径）

	* **JAVA\_HOME\\bin**（或等效的路径）

	* 安装 Maven 的目录

##创建新的 Maven 项目

从命令行中，使用以下代码创建名为 **WordCount** 的新 Maven 项目：

	mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

这会在当前位置创建名为 **WordCount** 的新目录，其中包含基本 Maven 项目。

**WordCount** 目录将包含以下项：

* **pom.xml**：包含 Maven 项目的设置。

* **src\\main\\java\\com\\microsoft\\example**：包含应用程序代码。

* **src\\test\\java\\com\\microsoft\\example**：包含应用程序的测试。对于本示例，我们将不创建测试。

###删除示例代码

由于我们要创建应用程序，因此请删除生成的测试和应用程序文件：

*  **src\\test\\java\\com\\microsoft\\example\\AppTest.java**

*  **src\\main\\java\\com\\microsoft\\example\\App.java**

##添加依赖项

由于这是一个 Storm 拓扑，因此你必须添加 Storm 组件的依赖项。打开 **pom.xml**，并在 **&lt;dependencies>** 节中添加以下代码：

	<dependency>
	  <groupId>org.apache.storm</groupId>
	  <artifactId>storm-core</artifactId>
	  <version>0.9.2-incubating</version>
	  <!-- keep storm out of the jar-with-dependencies -->
	  <scope>provided</scope>
	</dependency>

在编译时，Maven 会使用此信息来查找 Maven 存储库中的 **storm-core**。它会先查找本地计算机上的存储库。如果文件不存在，它会从公共 Maven 存储库下载这些文件，并将其存储在本地存储库中。

> [AZURE.NOTE] 请注意我们在该节中添加的 `<scope>provided</scope>` 行。这会告诉 Maven 从我们创建的任何 JAR 文件中排除 **storm-core**，因为系统将会予以提供。这样，便可以稍微减小所创建的包，并确保它们使用 Storm on HDInsight 群集中包含的 **storm-core** 位。

##生成配置

Maven 插件可让你自定义项目的生成阶段，例如，如何编译项目，或者如何将它打包成 JAR 文件。打开 **pom.xml**，并紧靠在 `</project>` 行的上方添加以下代码。

	<build>
	  <plugins>
	  </plugins>
	</build>

此节将用来添加插件和其他生成配置选项。

###添加插件

针对 Storm 拓扑，<a href="http://mojo.codehaus.org/exec-maven-plugin/" target="_blank">Exec Maven 插件</a>十分有用，因为它可让你轻松地在开发环境本地运行拓扑。将以下内容添加至 **pom.xml** 文件的 `<plugins>` 节，以包括 Exec Maven 插件：

	<plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
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

另一个有用的插件是用于更改编译选项的 <a href="http://maven.apache.org/plugins/maven-compiler-plugin/" target="_blank">Apache Maven Compiler 插件</a>。我们需要此插件的主要原因是要更改 Maven 用作应用程序源和目标的 Java 版本。我们需要的是版本 1.7。

在 **pom.xml** 的 `<plugins>` 节中添加以下内容，以包括 Apache Maven Compiler 插件并将源和目标版本设置为 1.7。

	<plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <source>1.7</source>
        <target>1.7</target>
      </configuration>
    </plugin>

##创建拓扑

基于 Java 的 Storm 拓扑包含你必须编写（或引用）为依赖项的三个组件。

* **Spout**：读取外部源中的数据，并发出进入拓扑的数据流。

* **Bolt**：对 Spout 或其他 Bolt 所发出的数据流执行处理，并发出一个或多个数据流。

* **拓扑**：定义如何排列 Spout 和 Bolt，并提供拓扑的入口点。

###创建 Spout

为了降低设置外部数据源的要求，以下 Spout 只会发出随机句子。它是 <a href="https://github.com/apache/storm/blob/master/examples/storm-starter/" target="_blank">Storm-Starter 示例</a>随附的 Spout 的修改版本。

> [AZURE.NOTE] 有关从外部数据源读取的 Spout 的示例，请参阅以下示例：<a href="https://github.com/apache/storm/tree/master/external/storm-kafka" target="_blank">Storm-Kafka</a>：从 Kafka 读取数据的 Spout

对于 Spout，在 **src\\main\\java\\com\\microsoft\\example** 目录中创建名为 **RandomSentenceSpout.java** 的新文件，并使用以下内容做为内容：

    /**
     * Licensed to the Apache Software Foundation (ASF) under one
     * or more contributor license agreements.  See the NOTICE file
     * distributed with this work for additional information
     * regarding copyright ownership.  The ASF licenses this file
     * to you under the Apache License, Version 2.0 (the
     * "License"); you may not use this file except in compliance
     * with the License.  You may obtain a copy of the License at
     *
     * http://www.apache.org/licenses/LICENSE-2.0
     *
     * Unless required by applicable law or agreed to in writing, software
     * distributed under the License is distributed on an "AS IS" BASIS,
     * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     * See the License for the specific language governing permissions and
     * limitations under the License.
     */

     /**
      * Original is available at https://github.com/apache/storm/blob/master/examples/storm-starter/src/jvm/storm/starter/spout/RandomSentenceSpout.java
      */

    package com.microsoft.example;

    import backtype.storm.spout.SpoutOutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichSpout;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Values;
    import backtype.storm.utils.Utils;

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

> [AZURE.NOTE] 虽然此拓扑只使用一个 Spout，但其他拓扑可能存在将数据从不同源送入拓扑的多个 Spout。

###创建 Bolt

Bolt 用于处理数据。此拓扑有两个 Bolt：

* **SplitSentence**：将 **RandomSentenceSpout** 发出的句子分割成不同的单词。

* **WordCount**：统计每个单词的出现次数。

> [AZURE.NOTE] Bolt 几乎可以执行任何操作，例如，计算、保存，或者与外部组件通信。

在 **src\\main\\java\\com\\microsoft\\example** 目录中创建两个新文件：**SplitSentence.java** 和 **WordCount.Java**。将以下内容用作这些文件的内容：

**SplitSentence**

    package com.microsoft.example;

    import java.text.BreakIterator;

    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;

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

    import backtype.storm.topology.BasicOutputCollector;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseBasicBolt;
    import backtype.storm.tuple.Fields;
    import backtype.storm.tuple.Tuple;
    import backtype.storm.tuple.Values;

    //There are a variety of bolt types. In this case, we use BaseBasicBolt
    public class WordCount extends BaseBasicBolt {
      //For holding words and counts
        Map<String, Integer> counts = new HashMap<String, Integer>();

        //execute is called to process tuples
        @Override
        public void execute(Tuple tuple, BasicOutputCollector collector) {
          //Get the word contents from the tuple
          String word = tuple.getString(0);
          //Have we counted any already?
          Integer count = counts.get(word);
          if (count == null)
            count = 0;
          //Increment the count and store it
          count++;
          counts.put(word, count);
          //Emit the word and the current count
          collector.emit(new Values(word, count));
        }

        //Declare that we will emit a tuple containing two fields; word and count
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
          declarer.declare(new Fields("word", "count"));
        }
      }

请花费片刻时间通读代码注释，以了解每个 Bolt 的工作原理。

###创建拓扑

拓扑将 Spout 和 Bolt 一起绑定到图形，该图形定义了组件之间的数据流动方式。它还提供 Storm 在群集内创建组件的实例时使用的并行度提示。

以下是此拓扑的组件的基本原理图。

![显示 Spout 和 Bolt 排列方式的示意图](./media/hdinsight-storm-develop-java-topology/wordcount-topology.png)

若要实现该拓扑，请在 **src\\main\\java\\com\\microsoft\\example** 目录中创建名为 **WordCountTopology.java** 的新文件。将以下内容用作该文件的内容：

	package com.microsoft.example;

    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.topology.TopologyBuilder;
    import backtype.storm.tuple.Fields;

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
        conf.setDebug(true);

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

##在本地测试拓扑

保存文件之后，请使用以下命令在本地测试拓扑。

	mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology

运行该命令时，拓扑会显示启动信息。然后开始显示与下面类似的行，因为句子是从 Spout 发出，然后由 Bolt 处理的。

    15398 [Thread-16-split] INFO  backtype.storm.daemon.executor - Processing received message source: spout:10, stream: default, id: {}, [an apple a day keeps thedoctor away]]
    15398 [Thread-16-split] INFO  backtype.storm.daemon.task - Emitting: split default [an]
    15399 [Thread-10-count] INFO  backtype.storm.daemon.executor - Processing received message source: split:6, stream: default, id: {}, [an]
    15399 [Thread-16-split] INFO  backtype.storm.daemon.task - Emitting: split default [apple]
    15400 [Thread-8-count] INFO  backtype.storm.daemon.executor - Processing received message source: split:6, stream: default, id: {}, [apple]
    15400 [Thread-16-split] INFO  backtype.storm.daemon.task - Emitting: split default [a]
    15399 [Thread-10-count] INFO  backtype.storm.daemon.task - Emitting: count default [an, 53]
    15400 [Thread-12-count] INFO  backtype.storm.daemon.executor - Processing received message source: split:6, stream: default, id: {}, [a]
    15400 [Thread-16-split] INFO  backtype.storm.daemon.task - Emitting: split default [day]
    15400 [Thread-8-count] INFO  backtype.storm.daemon.task - Emitting: count default [apple, 53]
    15401 [Thread-10-count] INFO  backtype.storm.daemon.executor - Processing received message source: split:6, stream: default, id: {}, [day]
    15401 [Thread-16-split] INFO  backtype.storm.daemon.task - Emitting: split default [keeps]
    15401 [Thread-12-count] INFO  backtype.storm.daemon.task - Emitting: count default [a, 53]

从此输出中可以看到发生了以下情况：

1. Spout 发出“an apple a day keeps the doctor away”。

2. Split Bolt 开始发出句子中的各个单词。

3. Count Bolt 开始发出每个单词及其发出次数。

查看 Count Bolt 所发出的数据，“apple”已发出 53 次。只要拓扑运行，计数就会持续增加，因为系统会随机反复发出相同的句子，并且永不会重置计数。

##Trident

Trident 是 Storm 提供的高级抽象。它支持有状态处理。Trident 的主要优点在于，它可以保证进入拓扑的每个消息只会处理一次。这在保证消息至少处理一次的原始 Java 拓扑中很难实现。两者还有其他方面的差异，例如，可以使用内置组件，而无需创建 Bolt。事实上，可以使用低泛型组件（例如筛选、投影和函数）来完全取代 Bolt。

你可以使用 Maven 项目来创建 Trident 应用程序。使用本文前面所述的相同基本步骤 - 只有代码不同。

有关 Trident 的详细信息，请参阅 <a href="http://storm.apache.org/documentation/Trident-API-Overview.html" target="_blank">Trident API 概述</a>。

##后续步骤

你已学习如何使用 Java 创建 Storm 拓扑。接下来，请学习如何：

* [在 HDInsight 上部署和管理 Apache Storm 拓扑](/documentation/articles/hdinsight-storm-deploy-monitor-topology/)

* [使用 Visual Studio 开发 Apache Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)

如需更多 Storm 拓扑示例，请访问 [Storm on HDInsight 拓扑示例](/documentation/articles/hdinsight-storm-example-topology/)。

<!---HONumber=Mooncake_0307_2016-->