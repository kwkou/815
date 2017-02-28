<properties
   pageTitle="使用 Apache Storm on HDInsight 确定 Twitter 趋势主题 | Microsoft Azure"
   description="了解如何使用 Trident 创建一个 Apache Storm 拓扑，根据哈希标记确定 Twitter 上的流行主题。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
   tags="azure-portal"/>

<tags
   ms.service="hdinsight"
   ms.date="09/27/2016"
   wacn.date="02/14/2017"/>

# 使用 Apache Storm on HDInsight 确定 Twitter 趋势主题

了解如何使用 Trident 创建 Storm 拓扑，以此判断 Twitter 上的趋势主题（哈希标记）。

Trident 是一种高级抽象，可提供联接、聚合、分组、函数和筛选器等工具。此外，Trident 还添加了基元，用于执行有状态增量处理。本示例演示如何使用自定义 Spout、函数和 Trident 提供的多个内置函数来生成拓扑。

> [AZURE.NOTE]本示例在很大程度上基于 Juan Alonso 制作的 [Trident Storm](https://github.com/jalonsoramos/trident-storm) 示例。

## 要求

* <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" target="_blank">Java 和 JDK 1.7</a>

* <a href="http://maven.apache.org/what-is-maven.html" target="_blank">Maven</a>

* <a href="http://git-scm.com/" target="_blank">Git</a>

* Twitter 开发人员帐户

## 下载项目

使用以下代码在本地复制项目。

	git clone https://github.com/Blackmist/TwitterTrending

## 拓扑

本示例的拓扑如下：

![拓扑](./media/hdinsight-storm-twitter-trending/trident.png)

> [AZURE.NOTE]这是一个简化的拓扑视图。组件的多个实例分散在群集中的节点上。

实现拓扑的 Trident 代码如下：

	topology.newStream("spout", spout)
	    .each(new Fields("tweet"), new HashtagExtractor(), new Fields("hashtag"))
	    .groupBy(new Fields("hashtag"))
	    .persistentAggregate(new MemoryMapState.Factory(), new Count(), new Fields("count"))
	    .newValuesStream()
	    .applyAssembly(new FirstN(10, "count"))
		.each(new Fields("hashtag", "count"), new Debug());

此代码将执行以下操作：

1. 从 Spout 创建新流。Spout 检索 Twitter 中的推文，然后筛选特定关键字（在本示例中，为 love、music 和 coffee）。

2. 使用自定义函数 HashtagExtractor 提取每则推文中的哈希标记。这些标记将发送到流。

3. 按哈希标记将流分组，并将其传递给聚合器。此聚合器计算每个哈希标记出现的次数。此数据保存在内存中。最后，发出包含哈希标记和计数的新流。

4. 由于我们只对指定的这批推文中最受欢迎的哈希标记感兴趣，因此应用 **FirstN** 程序集，并仅根据计数字段返回前 10 个值。

> [AZURE.NOTE]我们使用的是内置 Trident 功能，而不是 Spout 和 HashtagExtractor。
><p>
> 有关非 MemoryMapState 的 Trident-state 实现，请参阅：
><p>
><p> * <a href="https://github.com/fhussonnois/storm-trident-elasticsearch" target="_blank">Storm Trident 弹性搜索</a>
><p>
><p> * <a href="https://github.com/kstyrc/trident-redis" target="_blank">trident-redis</a>

### Spout

Spout **TwitterSpout** 使用 <a href="http://twitter4j.org/en/" target="_blank">Twitter4j</a> 从 Twitter 检索推文。创建筛选器（在本示例中为 love、music 和 coffee），然后将符合筛选器的传入推文 (status) 存储在链接的阻塞队列中。（有关详细信息，请参阅<a href="http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/LinkedBlockingQueue.html" target="_blank">类 LinkedBlockingQueue</a>。） 最后，从队列中提取出项，并将其发送到拓扑。

### HashtagExtractor

若要提取哈希标记，可以使用 <a href="http://twitter4j.org/javadoc/twitter4j/EntitySupport.html#getHashtagEntities--" target="_blank">getHashtagEntities</a> 来检索推文中包含的所有哈希标记。然后，将这些标记发送到流。

## 启用 Twitter

使用以下步骤注册新的 Twitter 应用程序，并获取读取 Twitter 内容所需的使用者和访问令牌信息。

1. 转到 <a href="https://apps.twitter.com" target="_blank">Twitter 应用</a>，然后单击“创建新应用”按钮。填写表单时，请将“回调 URL”字段保留空白。

2. 创建应用后，单击“密钥和访问令牌”选项卡。

3. 复制“使用者密钥”和“使用者密码”信息。

4. 如果没有令牌，请在页底部选择“创建我的访问令牌”。创建令牌后，复制“访问令牌”和“访问令牌密码”信息。

5. 在前面克隆的 **TwitterSpoutTopology** 项目中，打开 **resources/twitter4j.properties** 文件，并添加前面步骤中收集的信息，然后保存文件。

## 生成拓扑

使用以下代码生成项目：

		cd [directoryname]
		mvn compile

## 测试拓扑

使用以下命令在本地测试拓扑：

	mvn compile exec:java -Dstorm.topology=com.microsoft.example.TwitterTrendingTopology

拓扑启动后，你应会看到包含拓扑发出的哈希标记和计数的调试信息。输出应如下所示：

	DEBUG: [Quicktellervalentine, 7]
	DEBUG: [GRAMMYs, 7]
	DEBUG: [AskSam, 7]
	DEBUG: [poppunk, 1]
	DEBUG: [rock, 1]
	DEBUG: [punkrock, 1]
	DEBUG: [band, 1]
	DEBUG: [punk, 1]
	DEBUG: [indonesiapunkrock, 1]

## 后续步骤

在本地测试拓扑后，请探索如何部署拓扑：[在 HDInsight 上部署和管理 Apache Storm 拓扑](/documentation/articles/hdinsight-storm-deploy-monitor-topology/)。

你也可能对以下 Storm 主题感兴趣：

* [使用 Maven 为 Storm on HDInsight 开发 Java 拓扑](/documentation/articles/hdinsight-storm-develop-java-topology/)

* [使用 Visual Studio 开发 Storm on HDInsight 的 C# 拓扑](/documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/)

有关 HDinsight 的 Storm 示例：

* [Storm on HDInsight 的示例拓扑](/documentation/articles/hdinsight-storm-example-topology/)

<!---HONumber=71-->