<properties
    pageTitle="使用 Web 浏览器创建 Azure HDInsight (Hadoop) | Azure"
    description="了解如何使用 Web 浏览器和 Azure 门户预览在 Linux 上创建适用于 HDInsight 的 Hadoop、HBase、Storm 或 Spark 群集。"
    services="hdinsight"
    documentationcenter=""
    author="nitinme"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="697278cf-0032-4f7c-b9b2-a84c4347659e"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/16/2017"
    wacn.date="05/08/2017"
    ms.author="nitinme"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="827c788dfb8b604334aabf6c327b7d98a1136af0"
    ms.lasthandoff="04/28/2017" />

# <a name="create-linux-based-clusters-in-hdinsight-using-the-azure-portal-preview"></a>使用 Azure 门户预览在 HDInsight 中创建基于 Linux 的群集
[AZURE.INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Azure 门户预览是一种基于 Web 的管理工具，用于管理 Azure 云中托管的服务和资源。 在本文中，你将了解如何使用门户创建基于 Linux 的 HDInsight 群集。

## <a name="prerequisites"></a>先决条件
[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **一个 Azure 订阅**。 请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **一个现代 Web 浏览器**。 Azure 门户预览使用 HTML5 和 Javascript，可能无法在旧版 Web 浏览器中正确运行。

### <a name="access-control-requirements"></a>访问控制要求
[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="create-clusters"></a>创建群集
Azure 门户预览会公开大部分的群集属性。 使用 Azure Resource Manager 模板可以隐藏许多详细信息。 有关详细信息，请参阅[在 HDInsight 中使用 Azure Resource Manager 模板创建基于 Linux 的 Hadoop 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)。

1. 登录 [Azure 门户预览](https://portal.azure.cn)。
2. 依次单击“+”、“智能 + 分析”、“HDInsight”。

    ![在 Azure 门户预览中创建新群集](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.1.png "在 Azure 门户预览中创建新群集")

3. 在“HDInsight”边栏选项卡中，单击“自定义(大小、设置、应用)”，单击“基本信息”，然后输入以下信息。

    ![在 Azure 门户预览中创建新群集](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.basics.png "在 Azure 门户预览中创建新群集")

    * 输入 **群集名称**：此名称必须全局唯一。

    * 从“订阅”下拉列表中选择将用于此群集的 Azure 订阅  。

    * 单击“群集类型”，然后选择：

        * **群集类型**：如果不知道要选择哪种群集，请选择“Hadoop”。 它是最受欢迎的群集类型。

             > [AZURE.IMPORTANT]
             > HDInsight 群集有各种类型，分别与针对其优化群集的工作负荷或技术相对应。 不支持在一个群集上创建合并了多个类型（如 Storm 和 HBase）的群集。 
             > 
             > 

        * **操作系统**：选择“Linux”。

        * **版本**：如果不知道要选择哪个版本，请使用默认版本。 有关详细信息，请参阅 [HDInsight 群集版本](/documentation/articles/hdinsight-component-versioning/)。

    * 对于“群集登录用户名”和“群集登录密码”，请分别为管理员用户提供用户名和密码。

    * 输入“SSH 用户名”，如果要让 SSH 密码与在前面指定的管理员密码相同，则选中“使用与群集登录相同的密码”复选框。 如果不是，则提供“密码”或“公钥”，这会用于对 SSH 用户验证身份。 建议使用公钥。 单击底部的“选择”  ，保存凭据配置。

        有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

    * 对于“资源组”，指定是要创建新的资源组还是使用现有资源组。

    * 指定要在其中创建群集的数据中心 **位置** 。

    * 单击“下一步”。

4. 在“存储”边栏选项卡上，指定是否要将 Azure 存储 (WASB) 作为默认存储。 有关详细信息，请参阅下面的表格。

    ![在 Azure 门户预览中创建新群集](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.storage.png "在 Azure 门户预览中创建新群集")

    | 存储                                      | 说明 |
    |----------------------------------------------|-------------|
    | **将 Azure 存储 Blob 作为默认存储** | <ul><li>对于“主存储类型”，选择“Azure 存储”。 在此之后，如果要指定属于你的 Azure 订阅的存储帐户，则对于“选择方法”，可以选择“我的订阅”，然后选择存储帐户。 否则，请单击“访问密钥”，并提供想要从你的 Azure 订阅外部选择的存储帐户的信息。</li><li>对于“默认容器”，可以选择使用门户建议的默认容器名称或自己指定。</li><li>如果使用 WASB 作为默认存储，则可以（可选）单击“其他存储帐户”以指定要与群集关联的其他存储帐户。 在“Azure 存储密钥”边栏选项卡中，单击“添加存储密钥”，然后可以从 Azure 订阅或其他订阅提供存储帐户（通过提供存储帐户访问密钥）。</li></ul> |
    | **外部元存储**                      | （可选）可以指定 SQL 数据库用于保存与群集关联的 Hive 和 Oozie 元数据。 对于“为 Hive 选择 SQL 数据库”，选择 SQL 数据库，然后提供该数据库的用户名/密码。 为 Oozie 元数据重复以上这些步骤。<br><br>将 Azure SQL 数据库用于远存储时的一些注意事项。 <ul><li>用于元存储的 Azure SQL 数据库必须允许连接到其他 Azure 服务，包括 Azure HDInsight。 在 Azure SQL 数据库仪表板的右侧单击服务器名称。 这是运行 SQL 数据库实例的服务器。 进入服务器视图后，请单击“配置”，针对“Azure 服务”单击“是”，然后单击“保存”。</li><li>创建元存储时，请勿使用包含短划线或连字符的数据库名称，因为这可能会导致群集创建过程失败。</li></ul>                                                                                                                                                                       |

    单击“下一步”。 

    > [AZURE.WARNING]
    > 不支持在 HDInsight 群集之外的其他位置使用别的存储帐户。

5. （可选）单击“应用程序”以安装适用于 HDInsight 群集的应用程序。 这些应用程序可能是 Microsoft、独立软件供应商 (ISV) 或你自己开发的。 有关详细信息，请参阅[安装 HDInsight 应用程序](/documentation/articles/hdinsight-apps-install-applications/#install-applications-during-cluster-creation)。

6. 单击“群集大小”以显示针对此群集创建的节点的相关信息。 设置群集所需的工作节点数。 该群集的预估成本将显示在边栏选项卡内。

    ![节点定价层边栏选项卡](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.nodes.png "指定群集节点数")

    > [AZURE.IMPORTANT]
    > 如果你计划使用 32 个以上的工作节点（在创建群集时或是在创建之后通过扩展群集进行），则必须选择至少具有 8 个核心和 14GB ram 的头节点大小。
    > <p> 有关节点大小和相关费用的详细信息，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。

    单击“下一步”以保存节点定价配置。

7. 单击“高级设置”以配置其他可选设置，如使用“脚本操作”自定义群集以安装自定义组件或加入“虚拟网络”。 有关详细信息，请查看下表。

    ![节点定价层边栏选项卡](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.advanced.png "指定群集节点数")

    | 选项 | 说明 |
    |--------|-------------|
    | **脚本操作** | 如果想要在创建群集时使用自定义脚本自定义群集，请选择此选项。 有关脚本操作的详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。 |
    | **虚拟网络** | 如果想要将群集放入虚拟网络，请选择 Azure 虚拟网络和子网。 有关将 HDInsight 与虚拟网络配合使用的信息（包括虚拟网络的特定配置要求），请参阅[使用 Azure 虚拟网络扩展 HDInsight 功能](/documentation/articles/hdinsight-extend-hadoop-virtual-network/)。 |

    单击“下一步”。

8. 在“摘要”边栏选项卡上，验证之前输入的信息，然后单击“创建”。

    ![节点定价层边栏选项卡](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.summary.png "指定群集节点数")

    > [AZURE.NOTE]
    > 创建群集需要一些时间，通常约 15 分钟左右。 使用启动板上的磁贴或页面左侧的“通知”条目检查预配进程。
    > 
    > 
12. 创建过程完成后，在启动板中单击群集磁贴，启动群集边栏选项卡。 群集边栏选项卡提供以下信息。

    ![群集边栏选项卡](./media/hdinsight-hadoop-create-linux-cluster-portal/HDI.CreateCluster.completed.png "群集属性")

    参考以下内容了解此边栏选项卡顶部的图标。

    *  “概览”边栏选项卡提供有关该群集的基本信息，如名称、所属的资源组、位置、操作系统、群集仪表板 URL 等。
    * **仪表板** 可将你定向到与群集关联的 Ambari 门户。
    * **安全外壳**：使用 SSH 访问群集时所需的信息。
    * **缩放群集** 可增加与群集关联的辅助角色节点数。
    * **删除**：删除 HDInsight 群集。

## <a name="customize-clusters"></a>自定义群集
* 请参阅[使用 Bootstrap 自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-bootstrap/)。
* 请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

## <a name="delete-the-cluster"></a>删除群集
[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>后续步骤
成功创建 HDInsight 群集后，请参考以下主题来了解如何使用群集：

### <a name="hadoop-clusters"></a>Hadoop 群集
* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

### <a name="hbase-clusters"></a>HBase 群集
* [HBase on HDInsight 入门](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)
* [为 HBase on HDInsight 开发 Java 应用程序](/documentation/articles/hdinsight-hbase-build-java-maven-linux/)

### <a name="storm-clusters"></a>Storm 群集
* [为 Storm on HDInsight 开发 Java 拓扑](/documentation/articles/hdinsight-storm-develop-java-topology/)
* [在 Storm on HDInsight 中使用 Python 组件](/documentation/articles/hdinsight-storm-develop-python-topology/)
* [使用 Storm on HDInsight 部署和监视拓扑](/documentation/articles/hdinsight-storm-deploy-monitor-topology-linux/)

### <a name="spark-clusters"></a>Spark 群集
* [使用 Scala 创建独立的应用程序](/documentation/articles/hdinsight-apache-spark-create-standalone-application/)
* [使用 Livy 在 Spark 群集中远程运行作业](/documentation/articles/hdinsight-apache-spark-livy-rest-interface/)
* [Spark 和 BI：使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools/)
* [Spark 和机器学习：使用 HDInsight 中的 Spark 预测食品检查结果](/documentation/articles/hdinsight-apache-spark-machine-learning-mllib-ipython/)
* [Spark 流式处理：使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)