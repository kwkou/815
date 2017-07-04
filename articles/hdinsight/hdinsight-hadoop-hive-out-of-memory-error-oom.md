<properties
    pageTitle="解决 Auzre HDInsight 中的 Hive 内存不足错误 | Azure"
    description="解决 HDInsight 中的 Hive 内存不足错误。 客户方案为跨多个大型表运行查询。"
    keywords="内存不足错误, OOM, Hive 设置"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="7bce3dff-9825-4fa0-a568-c52a9f7d1dad"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/25/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="1cf4862bb5c418e1ee6fa422cfab7712931effa9"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="fix-a-hive-out-of-memory-error-in-azure-hdinsight"></a>解决 Azure HDInsight 中的 Hive 内存不足错误

了解处理大型表时如何通过配置 Hive 内存设置解决 Hive 内存不足错误。

## <a name="scenario-run-a-hive-query-against-large-tables"></a>方案：对大型表运行 Hive 查询

客户运行了 Hive 查询：

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
* 其他表没有那么大，但包含许多列。
* 所有表都彼此联接，在某些情况下，TABLE1 和其他表中的多个列也相互联接。

Hive 查询在 24 节点 A3 HDInsight 群集上用了 26 分钟才完成。 客户注意到以下警告消息：

    Warning: Map Join MAPJOIN[428][bigTable=?] in task 'Stage-21:MAPRED' is a cross product
    Warning: Shuffle Join JOIN[8][tables = [t1933775, t1932766]] in Stage 'Stage-4:MAPRED' is a cross product

通过使用 Tez 执行引擎， 相同的查询运行了 15 分钟，然后引发以下错误：

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

使用更大的虚拟机（例如，D12）时，也出现了该错误。

## <a name="debug-the-out-of-memory-error"></a>调试内存不足错误

我们的支持团队和工程团队合作发现了造成内存不足错误的原因之一是 [Apache JIRA 中所述的已知问题](https://issues.apache.org/jira/browse/HIVE-8306)：

    When hive.auto.convert.join.noconditionaltask = true we check noconditionaltask.size and if the sum  of tables sizes in the map join is less than noconditionaltask.size the plan would generate a Map join, the issue with this is that the calculation doesnt take into account the overhead introduced by different HashTable implementation as results if the sum of input sizes is smaller than the noconditionaltask size by a small margin queries will hit OOM.

hive-site.xml 文件中的 **hive.auto.convert.join.noconditionaltask** 已设置为 **true**：

    <property>
        <name>hive.auto.convert.join.noconditionaltask</name>
        <value>true</value>
        <description>
              Whether Hive enables the optimization about converting common join into mapjoin based on the input file size.
              If this parameter is on, and the sum of size for n-1 of the tables/partitions for a n-way join is smaller than the
              specified size, the join is directly converted to a mapjoin (there is no conditional task).
        </description>
      </property>

映射联接很可能是 Java 堆空间内存不足错误的原因。 如博客文章 [HDInsight 中的 Hadoop Yarn 内存设置](http://blogs.msdn.com/b/shanyu/archive/2014/07/31/hadoop-yarn-memory-settings-in-hdinsigh.aspx)所述，使用 Tez 执行引擎时，所用的堆空间事实上属于 Tez 容器。 请参阅下图，其中描述了 Tez 容器内存。

![Tez 容器内存示意图：Hive 内存不足错误](./media/hdinsight-hadoop-hive-out-of-memory-error-oom/hive-out-of-memory-error-oom-tez-container-memory.png)

如该博客文章中所述，以下两项内存设置定义了堆的容器内存：**hive.tez.container.size** 和 **hive.tez.java.opts**。 从我们的经验来看，内存不足异常并不意味着容器太小， 而是表示 Java 堆大小 (hive.tez.java.opts) 太小。 因此，每当看到内存不足时，可尝试增大 **hive.tez.java.opts**。 必要时，可能需要增大 **hive.tez.container.size**。 **java.opts** 设置应该大约为 **container.size** 的 80%。

> [AZURE.NOTE]
> **hive.tez.java.opts** 设置必须始终小于 **hive.tez.container.size**。
> 
> 

由于 D12 计算机具有 28GB 内存，因此我们决定使用 10GB (10240MB) 的容器大小并将 80% 分配给 java.opts：

    SET hive.tez.container.size=10240
    SET hive.tez.java.opts=-Xmx8192m

使用新设置，查询可在 10 分钟内成功运行。

## <a name="conclusion-oom-errors-and-container-size"></a>结论：OOM 错误和容器大小

遇到 OOM 错误不一定表示容器太小。 相反地，应该配置内存设置，以便将堆大小增加为至少是容器内存大小的 80%。

## <a name="next-steps"></a>后续步骤

- 有关优化 Hive 查询，请参阅[在 HDInsight 中优化 Hadoop 的 Hive 查询](/documentation/articles/hdinsight-hadoop-optimize-hive-query/)。

<!--Update_Description: wording update-->