<properties
    pageTitle="在本地安装 Jupyter notebook 并连接到 Azure HDInsight Spark 群集 | Azure"
    description="了解如何在计算机上本地安装 Jupyter 笔记本并将其连接到 Azure HDInsight 上的 Apache Spark 群集。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="48593bdf-4122-4f2e-a8ec-fdc009e47c16"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/10/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="cfff4aa2ade4f082628f5ac5b13f1240518697a6"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="install-jupyter-notebook-on-your-computer-and-connect-to-apache-spark-cluster-on-hdinsight"></a>在计算机上安装 Jupyter 笔记本并连接到 HDInsight 上的 Apache Spark 群集

本文介绍如何安装具有自定义 PySpark（适用于 Python）以及具有 Spark（适用于 Scala）内核和 Spark magic 的 Jupyter notebook，然后将 notebook 连接到 HDInsight 群集。 在本地计算机上安装 Jupyter 的原因多种多样，同时这种安装也面临着多种难题。 有关此方面的详细信息，请参阅本文末尾的[为何要在计算机上安装 Jupyter](#why-should-i-install-jupyter-on-my-computer)。

在计算机上安装 Jupyter 和 Spark magic 包括三个重要步骤。

* 安装 Jupyter 笔记本
* 安装 PySpark 和具有 Spark magic 的 Spark 内核
* 配置 Spark magic 以访问 HDInsight 上的 Spark 群集

有关适用于装有 HDInsight 群集的 Jupyter notebook 的自定义内核和 Spark magic 的详细信息，请参阅[适用于装有 HDInsight 上的 Apache Spark Linux 群集的 Jupyter notebook 的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)。

## <a name="prerequisites"></a>先决条件
此处所列的先决条件不适用于安装 Jupyter。 这些先决条件适用于安装笔记本之后将 Jupyter 笔记本连接到 HDInsight 群集。

* Azure 订阅。 请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* HDInsight 上的 Apache Spark 群集。 有关说明，请参阅[在 Azure HDInsight 中创建 Apache Spark 群集](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)。

## <a name="install-jupyter-notebook-on-your-computer"></a>在计算机上安装 Jupyter 笔记本

必须先安装 Python，然后才能安装 Jupyter notebook。 Python 和 Jupyter 都是作为 [Ananconda分发版](https://www.continuum.io/downloads)的一部分提供的。 安装 Anaconda 时，安装的是某个 Python 发行版。 安装 Anaconda 之后，可通过运行相应命令来添加 Jupyter 安装。

1. 下载适用于你的平台的 [Anaconda 安装程序](https://www.continuum.io/downloads) ，然后运行安装。 运行安装向导时，请确保选择将 Anaconda 添加到 PATH 变量的选项。
2. 运行以下命令来安装 Jupyter。

        conda install jupyter

    有关安装 Jupyter 的详细信息，请参阅 [Installing Jupyter using Anaconda](http://jupyter.readthedocs.io/en/latest/install.html)（使用 Anaconda 安装 Jupyter）。

## <a name="install-the-kernels-and-spark-magic"></a>安装内核和 Spark magic

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

有关如何安装 Spark magic、PySpark 和 Spark 内核的说明，请参阅 GitHub 上的 [sparkmagic 文档](https://github.com/jupyter-incubator/sparkmagic#installation)中的安装说明。 Spark magic 文档中的第一步会要求安装 Spark magic。 使用以下命令替换该链接中的第一步，具体取决于要连接到的 HDInsight 群集的版本。 之后，按照 Spark magic 文档中的剩余步骤进行操作。 如果要安装不同的内核，则必须执行 Spark magic 安装说明部分中的步骤 3。

* 对于群集 v3.4，请通过执行 `pip install sparkmagic==0.2.3` 安装 sparkmagic 0.2.3

* 对于群集 v3.5 和 v3.6，请通过执行 `pip install sparkmagic==0.11.2` 安装 sparkmagic 0.11.2

## <a name="configure-spark-magic-to-access-the-hdinsight-spark-cluster"></a>配置 Spark magic 以访问 HDInsight Spark 群集

在本部分中，你将配置前面安装的 Spark magic，以连接到 Apache Spark 群集（必须事先在 Azure HDInsight 中创建）。

1. Jupyter 配置信息通常存储在用户主目录中。 若要在任何 OS 平台上找到你的主目录，请键入以下命令。

    启动 Python shell。 在命令窗口中键入以下命令：

        python

    在 Python shell 中，输入以下命令以找到主目录。

        import os
        print(os.path.expanduser('~'))

2. 导航到主目录，然后创建一个名为 **.sparkmagic** 的文件夹（如果尚不存在）。
3. 在该文件夹中，创建一个名为 **config.json** 的文件，然后在该文件中添加以下 JSON 代码片段。

        {
          "kernel_python_credentials" : {
            "username": "{USERNAME}",
            "base64_password": "{BASE64ENCODEDPASSWORD}",
            "url": "https://{CLUSTERDNSNAME}.azurehdinsight.cn/livy"
          },
          "kernel_scala_credentials" : {
            "username": "{USERNAME}",
            "base64_password": "{BASE64ENCODEDPASSWORD}",
            "url": "https://{CLUSTERDNSNAME}.azurehdinsight.cn/livy"
          }
        }

4. 将 **{USERNAME}**、**{CLUSTERDNSNAME}**  和 **{BASE64ENCODEDPASSWORD}** 替换为适当的值。 可以使用许多以常用编程语言编写的实用工具或联机实用工具，来生成 base64 编码的密码作为实际密码。

5. 在 `config.json`中配置相应的检测信号设置。 应在与之前添加的 `kernel_python_credentials` 和 `kernel_scala_credentials` 代码片段相同的级别添加这些设置。 有关如何以及在何处添加检测信号设置的示例，请参阅 [示例 config.json](https://github.com/jupyter-incubator/sparkmagic/blob/master/sparkmagic/example_config.json)。

    * 对于 `sparkmagic 0.2.3`（群集 v3.4），包括：

            "should_heartbeat": true,
            "heartbeat_refresh_seconds": 5,
            "heartbeat_retry_seconds": 1

    * 对于 `sparkmagic 0.11.2`（群集 v3.5 和 v3.6），包括：

            "heartbeat_refresh_seconds": 5,
            "livy_server_heartbeat_timeout_seconds": 60,
            "heartbeat_retry_seconds": 1

    >[AZURE.TIP]
    >将发送检测信号，以确保会话不会泄漏。 请注意，当计算机转到睡眠或关闭状态时，将不会发送检测信号，从而导致会话被清除。 对于群集 v3.4，如果要禁用此行为，可以从 Ambari UI 将 Livy 配置 `livy.server.interactive.heartbeat.timeout` 设置为 `0`。 对于群集 v3.5，如果未设置上述 3.5 配置，会话将不会删除。

6. 启动 Jupyter。 从命令提示符使用以下命令。

        jupyter notebook

7. 验证是否可以使用 Jupyter 笔记本连接到群集，以及是否可以使用内核随附的 Spark magic。 执行以下步骤。

    a.将新的虚拟硬盘附加到 VM。 创建新的笔记本。 在右下角单击“新建”。 用户应该会看到默认内核 **Python2** 和安装的两个新内核：**PySpark** 和 **Spark**。 单击“PySpark”。

    ![创建新的 Jupyter 笔记本](./media/hdinsight-apache-spark-jupyter-notebook-install-locally/jupyter-kernels.png "创建新的 Jupyter 笔记本")

    b.保留“数据库类型”设置，即设置为“共享”。 运行以下代码片段。

        %%sql
        SELECT * FROM hivesampletable LIMIT 5

    如果可以成功检索输出，则表示与 HDInsight 群集的连接已经过测试。

    >[AZURE.TIP]
    >如果你想要更新笔记本配置以连接到不同的群集，请使用一组新值更新 config.json，如上述步骤 3 中所示。

## <a name="why-should-i-install-jupyter-on-my-computer"></a>为何要在计算机上安装 Jupyter？
你可能会出于多种原因而要在计算机上安装 Jupyter，然后将其连接到 HDInsight 上的 Spark 群集。

* 尽管 Azure HDInsight 中的 Spark 群集上已提供 Jupyter 笔记本，但在计算机上安装 Jupyter 可以选择在本地创建笔记本，根据正在运行的群集测试你的应用程序，然后将笔记本上载到群集。 若要将笔记本上载到群集，可以使用群集上运行的 Jupyter 笔记本来上载，或者将它们保存到与群集关联的存储帐户中的 /HdiNotebooks 文件夹。 有关如何在群集上存储 notebook 的详细信息，请参阅 [Jupyter notebook 存储在何处？](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/#where-are-the-notebooks-stored)
* 使用本地提供的笔记本可以根据应用程序要求连接到不同的 Spark 群集。
* 你可以使用 GitHub 实施源代码管理系统，并对笔记本进行版本控制。 此外，还可以建立一个协作环境，其中的多个用户可以使用同一个笔记本。
* 你甚至不需要启动群集就能在本地使用笔记本。 只需创建一个群集以根据它来测试你的笔记本，而不需要手动管理笔记本或开发环境。
* 配置自己的本地开发环境比在群集上配置 Jupyter 安装更容易。  你可以利用本地安装的所有软件，而不需要配置一个或多个远程群集。

> [AZURE.WARNING]
> 在本地计算机上安装 Jupyter 后，多个用户可以同时在同一个 Spark 群集上运行同一个笔记本。 在这种情况下，将会创建多个 Livy 会话。 如果你遇到问题并想要调试，则跟踪哪个 Livy 会话属于哪个用户将是一项复杂的任务。
>
>

## <a name="seealso"></a>另请参阅
* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview/)

### <a name="scenarios"></a>方案
* [Spark 和 BI：使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 对使用 HVAC 数据生成温度进行分析](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 预测食品检查结果](/documentation/articles/hdinsight-apache-spark-machine-learning-mllib-ipython/)
* [Spark 流式处理：使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)

### <a name="create-and-run-applications"></a>创建和运行应用程序
* [使用 Scala 创建独立的应用程序](/documentation/articles/hdinsight-apache-spark-create-standalone-application/)
* [使用 Livy 在 Spark 群集中远程运行作业](/documentation/articles/hdinsight-apache-spark-livy-rest-interface/)

### <a name="tools-and-extensions"></a>工具和扩展
* [在 HDInsight 上的 Spark 群集中使用 Zeppelin 笔记本](/documentation/articles/hdinsight-apache-spark-use-zeppelin-notebook/)
* [在 HDInsight 的 Spark 群集中可用于 Jupyter 笔记本的内核](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-kernels/)
* [Use external packages with Jupyter notebooks（将外部包与 Jupyter 笔记本配合使用）](/documentation/articles/hdinsight-apache-spark-jupyter-notebook-use-external-packages/)

### <a name="manage-resources"></a>管理资源
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager/)
* [跟踪和调试 HDInsight 中的 Apache Spark 群集上运行的作业](/documentation/articles/hdinsight-apache-spark-job-debugging/)

<!--Update_Description: wording update-->