<properties
    pageTitle="为 Azure HDInsight 中的 Apache Spark 群集分配资源 | Azure"
    description="了解如何对 Azure HDInsight 上的 Apache Spark 群集使用 Resource Manager，以提高性能。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="9da7d4e3-458e-4296-a628-77b14643f7e4"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/03/2017"
    wacn.date="05/31/2017"
    ms.author="nitinme" />

# 管理 Azure HDInsight 上 Apache Spark 群集的资源
在本文中，你将了解如何访问与 Spark 群集关联的界面，例如 Ambari UI、YARN UI 和 Spark History Server。你还将了解如何优化群集配置以获得最佳性能。

**先决条件：**

你必须具有以下各项：

* Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* HDInsight 上的 Apache Spark 群集。有关说明，请参阅[在 Azure HDInsight 中创建 Apache Spark 群集](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)。

## 如何启动 Ambari Web UI？
1. 在 [Azure 门户](https://portal.azure.cn/)上的启动板中，单击 Spark 群集的磁贴（如果已将它固定到启动板）。也可以在“全部浏览”>“HDInsight 群集”下导航到你的群集。
2. 在 Spark 群集边栏选项卡中，单击“仪表板”。出现提示时，输入 Spark 群集的管理员凭据。

    ![启动 Ambari](./media/hdinsight-apache-spark-resource-manager/hdispark.cluster.launch.dashboard.png "启动 Resource Manager")
3. 这应启动 Ambari Web UI，如下所示。

    ![Ambari Web UI](./media/hdinsight-apache-spark-resource-manager/ambari-web-ui.png "Ambari Web UI")

## 如何启动 Spark History Server？
1. 在 [Azure 门户](https://portal.azure.cn/)上的启动板中，单击 Spark 群集的磁贴（如果已将它固定到启动板）。
2. 在群集边栏选项卡中的“快速链接”下，单击“群集仪表板”。在“群集仪表板”边栏选项卡中，单击“Spark History Server”。

    ![Spark History Server](./media/hdinsight-apache-spark-resource-manager/launch-history-server.png "Spark History Server")

    出现提示时，输入 Spark 群集的管理员凭据。

## 如何启动 Yarn UI？
可以使用 YARN UI 监视当前正在 Spark 群集上运行的应用程序。

1. 在群集边栏选项卡中，单击“群集仪表板”，然后单击“YARN”。

    ![启动 YARN UI](./media/hdinsight-apache-spark-resource-manager/launch-yarn-ui.png)

    > [AZURE.TIP]
    或者，也可以从 Ambari UI 启动 YARN UI。若要启动 Ambari UI，请在群集边栏选项卡中单击“群集仪表板”，然后单击“HDInsight 群集仪表板”。在 Ambari UI 中，依次单击“YARN”、“快速链接”、活动的 Resource Manager 和“ResourceManager UI”。
    >
    >

## 用于运行 Spark 应用程序的最佳群集配置是什么？
根据应用程序的要求，可用于 Spark 配置的三个关键参数为 `spark.executor.instances`、`spark.executor.cores` 和 `spark.executor.memory`。执行器是针对 Spark 应用程序启动的进程。它在工作节点上运行，负责执行应用程序的任务。执行器的默认数目和每个群集的执行器大小均根据工作节点数目和工作节点大小计算。这些值存储在群集头节点上的 `spark-defaults.conf` 中。

这三个配置参数可在群集级别配置（适用于群集上运行的所有应用程序），也可以针对每个单独的应用程序指定。

### 使用 Ambari UI 更改参数
1. 在 Ambari UI 中，依次单击“Spark”、“配置”，然后展开“自定义 spark-defaults”。

    ![使用 Ambari 设置参数](./media/hdinsight-apache-spark-resource-manager/set-parameters-using-ambari.png)
2. 默认值支持在群集上并发运行 4 个 Spark 应用程序你可以从用户界面更改这些值，如下所示。

    ![使用 Ambari 设置参数](./media/hdinsight-apache-spark-resource-manager/set-executor-parameters.png)
3. 单击“保存”保存配置更改。在页面顶部，系统将提示你重新启动所有受影响的服务。单击“重新启动”。

    ![重新启动服务](./media/hdinsight-apache-spark-resource-manager/restart-services.png)

### 更改 Jupyter 笔记本中运行的应用程序的参数
对于在 Jupyter 笔记本中运行的应用程序，可以使用 `%%configure` magic 进行配置更改。理想情况下，必须先在应用程序开头进行此类更改，然后再运行第一个代码单元。这可以确保在创建 Livy 会话时将配置应用到该会话。如果你想要更改处于应用程序中后面某个阶段的配置，必须使用 `-f` 参数。但是，这样做会使应用程序中的所有进度丢失。

以下片段演示如何更改在 Jupyter 中运行的应用程序的配置。

    %%configure
    {"executorMemory": "3072M", "executorCores": 4, "numExecutors":10}

配置参数必须以 JSON 字符串形式传入，并且必须位于 magic 后面的下一行，如示例列中所示。

### 更改使用 spark-submit 提交的应用程序的参数
以下命令示范了如何更改使用 `spark-submit` 提交的批处理应用程序的配置参数。

    spark-submit --class <the application class to execute> --executor-memory 3072M --executor-cores 4 --num-executors 10 <location of application jar file> <application parameters>

### 更改使用 cURL 提交的应用程序的参数
以下命令示范了如何更改使用 cURL 提交的批处理应用程序的配置参数。

    curl -k -v -H 'Content-Type: application/json' -X POST -d '{"file":"<location of application jar file>", "className":"<the application class to execute>", "args":[<application parameters>], "numExecutors":10, "executorMemory":"2G", "executorCores":5' localhost:8998/batches

### 如何更改 Spark Thrift 服务器上的这些参数？
Spark Thrift 服务器提供对 Spark 群集的 JDBC/ODBC 访问，并用于为 Spark SQL 查询提供服务。Power BI、Tableau 等工具使用 ODBC 协议与 Spark Thrift 服务器通信，以便将 Spark SQL 查询作为 Spark 应用程序执行。创建 Spark 群集时，将启动 Spark Thrift 服务器的两个实例（每个头节点上各有一个实例）。在 YARN UI 中，每个 Spark Thrift 服务器显示为一个 Spark 应用程序。

Spark Thrift 服务器使用 Spark 动态执行器分配，因此未使用 `spark.executor.instances`。Spark Thrift 服务器改用 `spark.dynamicAllocation.minExecutors` 和 `spark.dynamicAllocation.maxExecutors` 指定执行器计数。使用配置参数 `spark.executor.cores` 和 `spark.executor.memory` 可以修改执行器大小。可按如下所示更改这些参数。

* 展开“高级 spark-thrift-sparkconf”类别可更新参数 `spark.dynamicAllocation.minExecutors`、`spark.dynamicAllocation.maxExecutors` 和 `spark.executor.memory`。

    ![配置 Spark Thrift 服务器](./media/hdinsight-apache-spark-resource-manager/spark-thrift-server-1.png)
* 展开“自定义 spark-thrift-sparkconf”类别可更新参数 `spark.executor.cores`。

    ![配置 Spark Thrift 服务器](./media/hdinsight-apache-spark-resource-manager/spark-thrift-server-2.png)

### 如何更改 Spark Thrift 服务器的驱动程序内存？
Spark Thrift 服务器驱动程序内存配置为头节点 RAM 大小的 25%，前提是头节点的 RAM 总大小大于 14GB。可按如下所示使用 Ambari UI 更改驱动程序内存配置。

* 在 Ambari UI 中，单击“Spark”、单击“配置”、展开“高级 spark-env”，然后提供“spark\_thrift\_cmd\_opts”的值。

    ![配置 Spark Thrift 服务器 RAM](./media/hdinsight-apache-spark-resource-manager/spark-thrift-server-ram.png)

## 我没有将 BI 与 Spark 群集配合使用。该如何回收资源？
因为我们使用 Spark 动态分配，Thrift 服务器使用的唯一资源是两个应用程序主机的资源。若要回收这些资源，必须停止群集上运行的 Thrift 服务器服务。

1. 在 Ambari UI 的左侧窗格中，单击“Spark”。
2. 在下一页中，单击“Spark Thrift 服务器”。

    ![重新启动 Thrift 服务器](./media/hdinsight-apache-spark-resource-manager/restart-thrift-server-1.png)
3. 你应看到正在运行 Spark Thrift 服务器的两个头节点。单击其中一个头节点。

    ![重新启动 Thrift 服务器](./media/hdinsight-apache-spark-resource-manager/restart-thrift-server-2.png)
4. 下一页将列出该头节点上运行的所有服务。在该列表中，单击 Spark Thrift 服务器旁边的下拉按钮，然后单击“停止”。

    ![重新启动 Thrift 服务器](./media/hdinsight-apache-spark-resource-manager/restart-thrift-server-3.png)
5. 同时对其他头节点重复上述步骤。

## 我的 Jupyter 笔记本未按预期运行。如何重新启动该服务？
1. 如上所示启动 Ambari Web UI。在左侧导航窗格中，依次单击“Jupyter”、“服务操作”和“全部重新启动”。随后将在所有头节点上启动 Jupyter 服务。

    ![重新启动 Jupyter](./media/hdinsight-apache-spark-resource-manager/restart-jupyter.png "重新启动 Jupyter")  


## 如何知道我的资源是否用完了？
1. 如上所示启动 Yarn UI。在屏幕顶部的“群集度量值”表中，选中“已用内存”和“内存总计”列的值。如果这 2 个值非常接近，则不可能资源不足，无法启动下一个应用程序。这同样适用于“已用 VCore”和“VCore 总计”列。此外，在主视图中，如果有应用程序保持“已接受”状态，而不转换为“正在运行”或“失败”状态，这也可能指示该应用程序未获得足够的资源来启动。

    ![资源限制](./media/hdinsight-apache-spark-resource-manager/resource-limit.png "资源限制")  


## 如何终止正在运行的应用程序以释放资源？
1. 在 Yarn UI 中，从左侧面板中，单击“正在运行”。在正在运行的应用程序的列表中，确定要终止的应用程序，并单击 **ID**。

    ![终止 App1](./media/hdinsight-apache-spark-resource-manager/kill-app1.png "终止 App1")  


2. 单击右上角的“终止应用程序”，然后单击“确定”。

    ![终止 App2](./media/hdinsight-apache-spark-resource-manager/kill-app2.png "终止 App2")  


## <a name="seealso"></a>另请参阅
* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview/)

### 方案
* [Spark 和 BI：使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 对使用 HVAC 数据生成温度进行分析](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 预测食品检查结果](/documentation/articles/hdinsight-apache-spark-machine-learning-mllib-ipython/)
* [Spark 流式处理：使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)
* [使用 HDInsight 中的 Spark 分析网站日志](/documentation/articles/hdinsight-apache-spark-custom-library-website-log-analysis/)

### 创建和运行应用程序
* [使用 Scala 创建独立的应用程序](/documentation/articles/hdinsight-apache-spark-create-standalone-application/)
* [使用 Livy 在 Spark 群集中远程运行作业](/documentation/articles/hdinsight-apache-spark-livy-rest-interface/)

### 工具和扩展
* [在 HDInsight 上的 Spark 群集中使用 Zeppelin 笔记本](/documentation/articles/hdinsight-apache-spark-use-zeppelin-notebook/)
* [在 HDInsight 的 Spark 群集中可用于 Jupyter 笔记本的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)
* [Use external packages with Jupyter notebooks（将外部包与 Jupyter 笔记本配合使用）](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-use-external-packages/)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster（在计算机上安装 Jupyter 并连接到 HDInsight Spark 群集）](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-install-locally/)

### 管理资源
* [跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业](/documentation/articles/hdinsight-apache-spark-job-debugging/)

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

<!---HONumber=Mooncake_0120_2017-->