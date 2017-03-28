<properties
    pageTitle="使用 CLI 创建 Azure HDInsight (Hadoop) | Azure"
    description="了解如何使用跨平台 Azure CLI、Azure Resource Manager 模板和 Azure REST API 创建 HDInsight 群集。可以指定群集类型（Hadoop、HBase 或 Storm），或使用脚本来安装自定义组件。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="50b01483-455c-4d87-b754-2229005a8ab9"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="03/28/2017"
    ms.author="larryfr" />

# 使用 Azure CLI 创建 HDInsight 群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-create-linux-cluster-selector.md)]

Azure CLI 是一个跨平台命令行实用工具，可用于管理 Azure 服务。将它与 Azure Resource Manager 模板配合使用可以创建 HDInsight 群集，以及关联的存储帐户和其他服务。

Azure Resource Manager 模板是描述**资源组**及其包含的所有资源（例如 HDInsight）的 JSON 文档。 此基于模板的方法可以定义一个模板中 HDInsight 所需的所有资源。该方法还可以通过**部署**将对组的更改作为一个整体进行管理，此部署将更改应用于整个组。

本文档中的步骤将引导你完成使用 Azure CLI 和模板创建新 HDInsight 群集的过程。

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## 先决条件

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **Azure CLI**。本文档中的步骤最近已使用 Azure CLI 版本 0.10.1 进行测试。

### 访问控制要求
[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## 登录到 Azure 订阅

遵循[从 Azure 命令行接口 (Azure CLI) 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)中所述的步骤，并使用 **login** 方法连接到订阅。

## 创建群集

在安装并配置 Azure CLI 后，应从命令提示符、shell 或终端会话执行以下步骤。

1. 使用以下命令进行 Azure 订阅的身份验证：
   
        azure login -e AzureChinaCloud
   
    系统提示你提供用户名与密码。如果有多个 Azure 订阅，可以使用 `azure account set <subscriptionname>` 来设置 Azure CLI 命令要使用的订阅。
2. 使用以下命令切换到 Azure Resource Manager 模式：
   
        azure config mode arm
3. 创建资源组。该资源组包含 HDInsight 群集和关联的存储帐户。
   
        azure group create groupname location
   
    * 将 **groupname** 替换为组的唯一名称。
    * 将 **location** 替换为要在其中创建该组的地理区域。
     
        有关有效位置的列表，请使用 `azure location list` 命令，然后使用 **Name** 列中的位置之一。
4. 创建存储帐户。该存储帐户用作 HDInsight 群集的默认存储。
   
        azure storage account create -g groupname --sku-name RAGRS -l location --kind Storage storagename
   
    * 将 **groupname** 替换为上一步骤中创建的组的名称。
    * 将 **location** 替换为上一步骤中使用的同一个位置。
    * 将 **storagename** 替换为存储帐户的唯一名称。
     
    > [AZURE.NOTE]
    有关此命令中使用的参数的详细信息，请使用 `azure storage account create -h` 查看有关此命令的帮助。
    > 
    > 
5. 检索用于访问存储帐户的密钥。
   
        azure storage account keys list -g groupname storagename
   
    * 将 **groupname** 替换为资源组名称。
    * 将 **storagename** 替换为存储帐户的名称。
     
        在返回的数据中，保存 **key1** 的 **key** 值。
6. 创建 HDInsight 群集。
   
        azure hdinsight cluster create -g groupname -l location -y Linux --clusterType Hadoop --defaultStorageAccountName storagename.blob.core.chinacloudapi.cn --defaultStorageAccountKey storagekey --defaultStorageContainer clustername --workerNodeCount 2 --userName admin --password httppassword --sshUserName sshuser --sshPassword sshuserpassword clustername
   
    * 将 **groupname** 替换为资源组名称。
    * 将 **Hadoop** 替换为需创建的群集类型。例如 `Hadoop`、`HBase`、`Storm` 或 `Spark`。
     
        > [AZURE.IMPORTANT]
        HDInsight 群集有各种类型，分别与针对其优化群集的工作负荷或技术相对应。没有任何方法支持创建组合多种类型的群集，例如一个群集同时具有 Storm 和 HBase 类型。
        > 
        > 
    * 将 **location** 替换为上一步骤中使用的同一位置。
    * 将 **storagename** 替换为存储帐户名称。
    * 将 **storagekey** 替换为上一步骤中获取的密钥。
    * 对于 `--defaultStorageContainer` 参数，请使用为群集使用的同一个名称。
    * 将 **admin** 和 **httppassword** 替换为通过 HTTPS 访问群集时要使用的用户名和密码。
    * 将 **sshuser** 和 **sshuserpassword** 替换为通过 SSH 访问群集时要使用的用户名和密码。
   
    > [AZURE.IMPORTANT]
    上面的示例使用 2 个工作节点创建群集。如果你计划使用 32 个以上的工作节点（在创建或扩展群集时），则必须选择至少具有 8 个核心和 14GB ram 的头节点大小。可以使用 `--headNodeSize` 参数设置头节点大小。
    > <p>
    > 有关节点大小和相关费用的详细信息，请参阅 [HDInsight 定价](/pricing/details/hdinsight/)。
     
    可能需要几分钟时间才能完成群集创建过程。通常大约为 15 分钟。

## 后续步骤
使用 Azure CLI 成功创建 HDInsight 群集后，请参考以下主题来了解如何使用群集：

### Hadoop 群集
* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

### HBase 群集
* [Get started with HBase on HDInsight（HBase on HDInsight 入门）](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)
* [Develop Java applications for HBase on HDInsight（为 HBase on HDInsight 开发 Java 应用程序）](/documentation/articles/hdinsight-hbase-build-java-maven-linux/)

### Storm 群集
* [Develop Java topologies for Storm on HDInsight（为 Storm on HDInsight 开发 Java 拓扑）](/documentation/articles/hdinsight-storm-develop-java-topology/)
* [Use Python components in Storm on HDInsight（在 Storm on HDInsight 中使用 Python 组件）](/documentation/articles/hdinsight-storm-develop-python-topology/)
* [Deploy and monitor topologies with Storm on HDInsight（使用 Storm on HDInsight 部署和监视拓扑）](/documentation/articles/hdinsight-storm-deploy-monitor-topology-linux/)

<!---HONumber=Mooncake_0120_2017-->