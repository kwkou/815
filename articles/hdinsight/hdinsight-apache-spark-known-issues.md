<properties
    pageTitle="Azure HDInsight 中的 Apache Spark 群集的已知问题 | Azure"
    description="Azure HDInsight 中的 Apache Spark 群集的已知问题。"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="610c4103-ffc8-4ec0-ad06-fdaf3c4d7c10"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/24/2017"
    wacn.date="05/08/2017"
    ms.author="nitinme"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="e4e83522c055ff07aed34f001e45e3416689f482"
    ms.lasthandoff="04/28/2017" />

# <a name="known-issues-for-apache-spark-cluster-on-hdinsight"></a>HDInsight 上的 Apache Spark 群集的已知问题

本文档记述了 HDInsight Spark 公共预览版的所有已知问题。  

## <a name="livy-leaks-interactive-session"></a>Livy 泄漏交互式会话
在某个交互式会话仍保持活动状态的情况下，Livy 重新启动（从 Ambari 重新启动，或者由于头节点 0 虚拟机重新启动），同时交互式作业会话泄漏。 因此，新作业可能陷于“已接受”状态中，且无法启动。

**缓解：**

使用以下过程解决该问题：

1. 通过 SSH 连接到头节点。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 运行以下命令，以查找通过 Livy 启动的交互式作业的应用程序 ID。 

        yarn application -list

    如果作业是通过未指定明确名称的 Livy 交互式会话启动的，则默认的作业名称将是 Livy；对于 Jupyter 笔记本启动的 Livy 会话，作业名称以 remotesparkmagics_* 开头。 
3. 运行以下命令以终止这些作业。 

        yarn application -kill <Application ID>

新作业将开始运行。 

## <a name="spark-history-server-not-started"></a>Spark History Server未启动
创建群集后，Spark History Server 不自动启动。  

**缓解：** 

从 Ambari 手动启动 History Server。

## <a name="permission-issue-in-spark-log-directory"></a>Spark 日志目录中的权限问题
当 hdiuser 通过 spark-submit 提交作业时发生错误 java.io.FileNotFoundException：/var/log/spark/sparkdriver_hdiuser.log（权限被拒绝），且不会写入驱动程序日志。 

**缓解：**

1. 将 hdiuser 添加到 Hadoop 组。 
2. 创建群集后，提供对 /var/log/spark 的 777 权限。 
3. 使用 Ambari 将 Spark 日志位置更新为具有 777 权限的目录。  
4. 以 sudo 身份运行 spark-submit。  

## <a name="spark-phoenix-connector-is-not-supported"></a>不支持 Spark-Phoenix 连接器

目前，HDInsight Spark 群集不支持 Spark-Phoenix 连接器。

**缓解：**

必须改用 Spark-HBase 连接器。 有关说明，请参阅[如何使用 Spark-HBase 连接器](https://blogs.msdn.microsoft.com/azuredatalake/2016/07/25/hdinsight-how-to-use-spark-hbase-connector/)。

## <a name="issues-related-to-jupyter-notebooks"></a>Jupyter 笔记本的相关问题
下面是与 Jupyter 笔记本相关的一些已知问题。

### <a name="notebooks-with-non-ascii-characters-in-filenames"></a>笔记本的文件名中包含非 ASCII 字符
可在 Spark HDInsight 群集中使用的 Jupyter 笔记本的文件名不应包含非 ASCII 字符。 如果你尝试通过 Jupyter UI 上载文件名中包含非 ASCII 字符的文件，操作将会失败且不发出提示（即，Jupyter 不允许你上载该文件，但同时也不引发可视的错误）。 

### <a name="error-while-loading-notebooks-of-larger-sizes"></a>加载大型笔记本时发生错误
加载大型笔记本时，可能会看到错误“ **`Error loading notebook`** 。  

**缓解：**

收到此错误并不表示数据已损坏或丢失。  笔记本仍在磁盘上的 `/var/lib/jupyter` 中，可以通过 SSH 连接到群集来访问它。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

一旦使用 SSH 连接到群集后，即可将笔记本从群集复制到本地计算机（使用 SCP 或 WinSCP）作为备份，以免丢失笔记本中的重要数据。 然后，可以使用端口 8001 通过 SSH 隧道（不通过网关）连接到头节点来访问 Jupyter。  你可以从该处清除笔记本的输出，并将其重新保存，以尽量缩小笔记本的大小。

若要防止今后发生此错误，必须遵循一些最佳实践：

* 必须保持较小的笔记本大小。 发回到 Jupyter 的所有 Spark 作业输出都将保存在笔记本中。  一般而言，Jupyter 的最佳实践是避免对大型 RDD 或数据帧运行 `.collect()`；如果想要查看 RDD 的内容，请考虑运行 `.take()` 或 `.sample()`，这样，输出便不会太大。
* 此外，在保存笔记本时，请清除所有输出单元以减小大小。

### <a name="notebook-initial-startup-takes-longer-than-expected"></a>笔记本初次启动花费的时间比预期要长
在使用 Spark magic 的 Jupyter 笔记本中，第一个代码语句可能需要花费一分钟以上。  

**解释：**

这发生在运行第一个代码单元时。 它在后台启动设置会话配置，以及设置 SQL、Spark 和 Hive 上下文。 设置这些上下文后，第一个语句才运行，因此让人觉得完成语句需要花费很长时间。

### <a name="jupyter-notebook-timeout-in-creating-the-session"></a>创建会话时 Jupyter 笔记本超时
如果 Spark 群集的资源不足，Jupyter 笔记本中的 Spark 和 Pyspark 内核在尝试创建会话时会超时。 

**缓解措施：** 

1. 通过以下方式释放 Spark 群集中的一些资源：

    * 转到“关闭并停止”菜单或单击笔记本资源管理器中的“关闭”，以停止其他 Spark 笔记本。
    * 通过 YARN 停止其他 Spark 应用程序。
2. 重新启动先前尝试启动的笔记本。 现在应有足够的资源用于创建会话。

## <a name="see-also"></a>另请参阅
* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview/)

### <a name="scenarios"></a>方案
* [Spark 和 BI：使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 对使用 HVAC 数据生成温度进行分析](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 预测食品检查结果](/documentation/articles/hdinsight-apache-spark-machine-learning-mllib-ipython/)
* [Spark 流式处理：使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)
* [使用 HDInsight 中的 Spark 分析网站日志](/documentation/articles/hdinsight-apache-spark-custom-library-website-log-analysis/)

### <a name="create-and-run-applications"></a>创建和运行应用程序
* [使用 Scala 创建独立的应用程序](/documentation/articles/hdinsight-apache-spark-create-standalone-application/)
* [使用 Livy 在 Spark 群集中远程运行作业](/documentation/articles/hdinsight-apache-spark-livy-rest-interface/)

### <a name="tools-and-extensions"></a>工具和扩展
* [在 HDInsight 上的 Spark 群集中使用 Zeppelin 笔记本](/documentation/articles/hdinsight-apache-spark-use-zeppelin-notebook/)
* [在 HDInsight 的 Spark 群集中可用于 Jupyter 笔记本的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)
* [将外部包与 Jupyter 笔记本配合使用](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-use-external-packages/)
* [在计算机上安装 Jupyter 并连接到 HDInsight Spark 群集](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-install-locally/)

### <a name="manage-resources"></a>管理资源
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager/)
* [跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业](/documentation/articles/hdinsight-apache-spark-job-debugging/)

<!--Update_Description: add issue "spark phoenix connector is not supported"-->