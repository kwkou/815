<properties
    pageTitle="在 Azure HDInsight 上的 Apache Spark 群集中将外部 python 包与 Jupyter 笔记本配合使用 | Azure"
    description="逐步说明如何配置可在 HDInsight Spark 群集中使用的 Jupyter 笔记本，以使用外部 python 包。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="21978b71-eb53-480b-a3d1-c5d428a7eb5b"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/28/2016"
    wacn.date="02/06/2017"
    ms.author="nitinme" />

# 在 HDInsight Linux 上的 Apache Spark 群集中将外部 python 包与 Jupyter 笔记本配合使用

了解如何使用脚本操作在 HDInsight (Linux) 上配置 Apache Spark 群集，以使用未现成包含在群集中的、由社区贡献的 **python** 外部包。

可以在[包索引](https://pypi.python.org/pypi)中搜索可用包的完整列表。也可以从其他源获取可用包的列表。例如，可以安装通过 [Anaconda](https://docs.continuum.io/anaconda/pkg-docs) 或 [conda-forge](https://conda-forge.github.io/feedstocks.html) 提供的包。

本文将介绍如何使用脚本操作在群集上安装 [TensorFlow](https://www.tensorflow.org/) 包并通过 Jupyter 笔记本使用它。

## 先决条件
必须满足以下条件：

* Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* HDInsight Linux 上的 Apache Spark 群集。有关说明，请参阅[在 Azure HDInsight 中创建 Apache Spark 群集](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)。

    > [AZURE.NOTE]
    如果 HDInsight Linux 上还没有 Spark 群集，则可以在群集创建过程中运行脚本操作。访问有关[如何使用自定义脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster/)的文档。
    > 
    > 

## 将外部包与 Jupyter 笔记本配合使用

1. 在 [Azure 门户预览](https://portal.azure.cn/)上的启动板中，单击 Spark 群集的磁贴（如果已将它固定到启动板）。也可以在“全部浏览”>“HDInsight 群集”下导航到你的群集。

2. 在 Spark 群集边栏选项卡中，单击“使用情况”下的“脚本操作”。运行用于在头节点和工作节点中安装 TensorFlow 的自定义操作。可以参考 https://hdiconfigactions.blob.core.windows.net/linuxtensorflow/tensorflowinstall.sh 中的 bash 脚本；访问有关[如何使用自定义脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster/)的文档。

    > [AZURE.NOTE]
    群集中有两个 python 安装。Spark 将使用位于 `/usr/bin/anaconda/bin` 中的 Anaconda python 安装。通过 `/usr/bin/anaconda/bin/pip` 和 `/usr/bin/anaconda/bin/conda` 在自定义操作中引用该安装。
    > 
    > 

3. 打开 PySpark Jupyter 笔记本

    ![创建新的 Jupyter 笔记本](./media/hdinsight-apache-spark-python-package-installation/hdispark.note.jupyter.createpysparknotebook.png "创建新的 Jupyter 笔记本")  


4. 新笔记本随即已创建，并以 Untitled.pynb 名称打开。在顶部单击笔记本名称，然后输入一个友好名称。

    ![提供笔记本的名称](./media/hdinsight-apache-spark-jupyter-notebook-use-external-packages/hdispark.note.jupyter.notebook.name.png "提供笔记本的名称")  


5. 现在将`import tensorflow` 并运行 hello world 示例。

    要复制的代码：

	    import tensorflow as tf
	    hello = tf.constant('Hello, TensorFlow!')
	    sess = tf.Session()
	    print(sess.run(hello))

	结果将如下所示：
	
	![TensorFlow 代码执行](./media/hdinsight-apache-spark-python-package-installation/execution.png "执行 TensorFlow 代码")  




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
* [在 HDInsight 上的 Apache Spark 群集中将外部包与 Jupyter 笔记本配合使用](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-use-external-packages/)
* [在 HDInsight 上的 Spark 群集中使用 Zeppelin 笔记本](/documentation/articles/hdinsight-apache-spark-use-zeppelin-notebook/)
* [在 HDInsight 的 Spark 群集中可用于 Jupyter 笔记本的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)
* [Install Jupyter on your computer and connect to an HDInsight Spark cluster（在计算机上安装 Jupyter 并连接到 HDInsight Spark 群集）](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-install-locally/)

### 管理资源
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager/)
* [Track and debug jobs running on an Apache Spark cluster in HDInsight（跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业）](/documentation/articles/hdinsight-apache-spark-job-debugging/)

<!---HONumber=Mooncake_0103_2017-->