<properties
	pageTitle="内存不足 (OOM) 错误 - Hive 设置 | Azure"
	description="在 HDInsight 的 Hadoop 中解决运行 Hive 查询时内存不足 (OOM) 错误。客户方案为跨多个大型表运行查询。"
	keywords="内存不足错误, OOM, Hive 设置"
	services="hdinsight"
	documentationCenter=""
	authors="rashimg"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="12/10/2015"
	wacn.date="01/14/2016"/>

# 在 Azure HDInsight 的 Hadoop 中使用 Hive 内存设置解决内存不足 (OOM) 错误

客户面临的一个常见问题是在使用 Hive 时遇到内存不足 (OOM) 错误。本文将会讲解一种客户方案，并建议相应的 Hive 设置来解决该问题。

## 方案：跨大型表运行 Hive 查询

某个客户使用 Hive 运行了以下查询。

	SELECT
		COUNT (T1.COLUMN1) as DisplayColumn1,
		…
		…
		….
	FROM
		TABLE1 T1,
		TABLE2 T2,
		TABLE3 T3,
		TABLE5 T4,
		TABLE6 T5,
		TABLE7 T6
	where (T1.KEY1 = T2.KEY1….
		…
		…

此查询有一些繁琐之处：

* T1 是大型表 TABLE1 的别名，其中包含多个 STRING 列类型。
* 其他表的规模都比较小，但也包含许多列。
* 所有表都彼此联接，在某些情况下，TABLE1 和其他表中的多个列也相互联接。

当客户在 24 节点 A3 群集上的 MapReduce 上使用 Hive 运行查询时，该查询的运行时间大约是 26 分钟。客户在 MapReduce 上使用 Hive 运行查询时注意到以下警告消息：

	Warning: Map Join MAPJOIN[428][bigTable=?] in task 'Stage-21:MAPRED' is a cross product
	Warning: Shuffle Join JOIN[8][tables = [t1933775, t1932766]] in Stage 'Stage-4:MAPRED' is a cross product

由于查询大约在 26 分钟内完成运行，因此客户忽略了这些警告，反而开始将重点放在如何进一步改善此查询的性能上。

客户参考了[在 HDInsight 中优化 Hadoop 的 Hive 查询](/documentation/articles/hdinsight-hadoop-optimize-hive-query-v1/)，并决定使用 Tez 执行引擎。启用 Tez 设置运行同一个查询之后，该查询运行了 15 分钟，然后引发以下错误：

	Status: Failed
	Vertex failed, vertexName=Map 5, vertexId=vertex_1443634917922_0008_1_05, diagnostics=[Task failed, taskId=task_1443634917922_0008_1_05_000006, diagnostics=[TaskAttempt 0 failed, info=[Error: Failure while running task:java.lang.RuntimeException: java.lang.OutOfMemoryError: Java heap space
        at
	org.apache.hadoop.hive.ql.exec.tez.TezProcessor.initializeAndRunProcessor(TezProcessor.java:172)
        at org.apache.hadoop.hive.ql.exec.tez.TezProcessor.run(TezProcessor.java:138)
        at
	org.apache.tez.runtime.LogicalIOProcessorRuntimeTask.run(LogicalIOProcessorRuntimeTask.java:324)
        at
	org.apache.tez.runtime.task.TezTaskRunner$TaskRunnerCallable$1.run(TezTaskRunner.java:176)
        at
	org.apache.tez.runtime.task.TezTaskRunner$TaskRunnerCallable$1.run(TezTaskRunner.java:168)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:415)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1628)
        at
	org.apache.tez.runtime.task.TezTaskRunner$TaskRunnerCallable.call(TezTaskRunner.java:168)
        at
	org.apache.tez.runtime.task.TezTaskRunner$TaskRunnerCallable.call(TezTaskRunner.java:163)
        at java.util.concurrent.FutureTask.run(FutureTask.java:262)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)
	Caused by: java.lang.OutOfMemoryError: Java heap space

于是，客户决定使用更大的 VM（例如 D12），因为觉得更大的 VM 具有更多的堆空间。尽管如此，客户仍然看到了此错误。客户与 HDInsight 团队联系，以求帮助调试此问题。

## 调试内存不足 (OOM) 错误

我们的支持团队和工程团队合作发现了造成内存不足 (OOM) 错误的原因之一是 [Apache JIRA 中所述的已知问题](https://issues.apache.org/jira/browse/HIVE-8306)。JIRA 中的说明：

	When hive.auto.convert.join.noconditionaltask = true we check noconditionaltask.size and if the sum  of tables sizes in the map join is less than noconditionaltask.size the plan would generate a Map join, the issue with this is that the calculation doesnt take into account the overhead introduced by different HashTable implementation as results if the sum of input sizes is smaller than the noconditionaltask size by a small margin queries will hit OOM.

我们通过调查 hive-site.xml 文件，确认 **hive.auto.convert.join.noconditionaltask** 确实已设置为 **true**：

	<property>
    	<name>hive.auto.convert.join.noconditionaltask</name>
    	<value>true</value>
    	<description>
      		Whether Hive enables the optimization about converting common join into mapjoin based on the input file size.
      		If this parameter is on, and the sum of size for n-1 of the tables/partitions for a n-way join is smaller than the
      		specified size, the join is directly converted to a mapjoin (there is no conditional task).
    	</description>
  	</property>

根据警告和 JIRA，我们假设映射联接是引发 Java 堆空间 OOM 错误的原因。因此我们更深入分析了此问题。

如博客文章 [HDInsight 中的 Hadoop Yarn 内存设置](http://blogs.msdn.com/b/shanyu/archive/2014/07/31/hadoop-yarn-memory-settings-in-hdinsigh.aspx)所述，使用 Tez 执行引擎时，所用的堆空间事实上属于 Tez 容器。请参阅下图，其中描述了 Tez 容器内存。

![Tez 容器内存示意图：Hive 内存不足 (OOM) 错误](./media/hdinsight-hadoop-hive-out-of-memory-error-oom/hive-out-of-memory-error-oom-tez-container-memory.png)


如该博客文章中所述，以下两项内存设置定义了堆的容器内存：**hive.tez.container.size** 和 **hive.tez.java.opts**。从我们的经验来看，OOM 异常不表示容器太小，而是表示 Java 堆大小 (hive.tez.java.opts) 太小。因此，每当你看到 OOM 时，可尝试增大 **hive.tez.java.opts**。必要时，你可能需要增大 **hive.tez.container.size**。**java.opts** 设置应该大约为 **container.size** 的 80%。

> [AZURE.NOTE]**hive.tez.java.opts** 设置始终必须小于 **hive.tez.container.size**。

由于 D12 计算机具有 28GB 内存，因此我们决定使用 10GB (10240MB) 的容器大小并将 80% 分配给 java.opts。可以在 Hive 控制台上使用以下设置完成此操作：

	SET hive.tez.container.size=10240
	SET hive.tez.java.opts=-Xmx8192m

使用这些设置后，查询可在 10 分钟内成功运行。

## 结论：OOM 错误和容器大小

遇到 OOM 错误不一定表示容器太小。相反地，应该配置内存设置，以便将堆大小增加为至少是容器内存大小的 80%。

<!---HONumber=Mooncake_0104_2016-->