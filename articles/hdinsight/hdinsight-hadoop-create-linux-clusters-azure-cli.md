<properties
    pageTitle="使用命令行创建 Azure HDInsight (Hadoop) | Azure"
    description="了解如何使用跨平台 Azure CLI 1.0 创建 HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="50b01483-455c-4d87-b754-2229005a8ab9"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/04/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="828c5d36c40ff780cab6902052c1ef0e87fa8f51"
    ms.lasthandoff="04/28/2017" />

# <a name="create-hdinsight-clusters-using-the-azure-cli"></a>使用 Azure CLI 创建 HDInsight 群集

[AZURE.INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

本文介绍了使用 Azure CLI 1.0 创建 HDInsight 3.5 群集的相关步骤。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="prerequisites"></a>先决条件

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **一个 Azure 订阅**。 请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

* **Azure CLI**。 本文档中的步骤最近已使用 Azure CLI 版本 0.10.1 进行测试。

    > [AZURE.IMPORTANT]
    > 本文中的步骤不适用于 Azure CLI 2.0。 Azure CLI 2.0 不支持创建 HDInsight 群集。

### <a name="access-control-requirements"></a>访问控制要求

[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="log-in-to-your-azure-subscription"></a>登录到 Azure 订阅

按照 [从 Azure 命令行接口 (Azure CLI) 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/) 中所述的步骤，使用 **login** 方法连接到订阅。

## <a name="create-a-cluster"></a>创建群集

在安装并配置 Azure CLI 后，应从命令提示符、shell 或终端会话执行以下步骤。

1. 使用以下命令进行 Azure 订阅的身份验证：

        azure login -e AzureChinaCloud

    系统提示你提供用户名与密码。 如果有多个 Azure 订阅，可以使用 `azure account set <subscriptionname>` 来设置 Azure CLI 命令要使用的订阅。

2. 使用以下命令切换到 Azure Resource Manager 模式：

        azure config mode arm

3. 创建资源组。 此资源组包含 HDInsight 群集和关联的存储帐户。

        azure group create groupname location

    * 将“groupname”替换为组的唯一名称。

    * 将 **location** 替换为要在其中创建该组的地理区域。

        有关有效位置的列表，请使用 `azure location list` 命令，然后使用 **Name** 列中的位置之一。

4. 创建存储帐户。 此存储帐户将用作 HDInsight 群集的默认存储。

        azure storage account create -g groupname --sku-name RAGRS -l location --kind Storage storagename

    * 将“groupname”替换为上一步中创建的组的名称。

    * 将 **location** 替换为上一步骤中使用的同一个位置。

    * 将“storagename”替换为存储帐户的唯一名称。

        > [AZURE.NOTE]
        > 有关此命令中使用参数的详细信息，请使用 `azure storage account create -h` 查看此命令的帮助。

5. 检索用于访问存储帐户的密钥。

        azure storage account keys list -g groupname storagename

    * 将 **groupname** 替换为资源组名称。
    * 将“storagename”替换为存储帐户的名称。

    在返回的数据中，保存 **key1** 的 **key** 值。

6. 创建 HDInsight 群集。

        azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.chinacloudapi.cn --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 2 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername

    * 将 **groupname** 替换为资源组名称。

    * 用希望创建的群集类型替换 **Hadoop**。 例如，Hadoop、HBase、Storm 或 Spark。

    > [AZURE.IMPORTANT]
    > HDInsight 群集具有各种不同的类型，与该群集进行优化的工作负荷或技术相对应。 不支持在一个群集上创建合并了多个类型（如 Storm 和 HBase）的群集。

    * 将 **location** 替换为上一步骤中使用的同一位置。

    * 将 **storagename** 替换为存储帐户名称。

    * 将 **storagekey** 替换为上一步骤中获取的密钥。

    * 对于 `--defaultStorageContainer` 参数，请使用你为群集使用的同一个名称。

    * 将“admin”和“httppassword”替换为通过 HTTPS 访问群集时要使用的用户名和密码。

    * 将“sshuser”和“sshuserpassword”替换为通过 SSH 访问群集时要使用的用户名和密码

    > [AZURE.IMPORTANT]
    > 此示例创建一个具有两个辅助节点的群集。 如果计划（在创建或扩展群集时）辅助节点在 32 个以上，则必须选择至少具有 8 个核心和 14GB RAM 的头节点大小。 可以使用 `--headNodeSize` 参数设置头节点大小。
    > <p>
    > 有关节点大小和相关费用的详细信息，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。

    可能需要几分钟时间才能完成群集创建过程。 通常大约为 15 分钟。

## <a name="next-steps"></a>后续步骤

使用 Azure CLI 成功创建 HDInsight 群集后，请参考以下主题来了解如何使用群集：

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

<!--Update_Description: wording update-->