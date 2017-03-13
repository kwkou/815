<properties
    pageTitle="在 HDInsight 中创建基于 Windows 的 Hadoop 群集 | Azure"
    description="了解如何使用 Azure 门户预览在 HDInsight 中创建基于 Windows 的 Hadoop。"
    services="hdinsight"
    documentationcenter=""
    tags="azure-portal"
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="c8689ef3-f56f-4708-8a3a-cc00abc54e8d"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/18/2017"
    wacn.date="03/10/2017"
    ms.author="jgao" />  


# 使用 Azure 门户预览在 HDInsight 中创建基于 Windows 的 Hadoop 群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-selector-create-clusters.md)]

了解如何使用 Azure 门户预览在 HDInsight 中创建基于 Windows 的 Hadoop 群集。

本文中的信息仅适用于基于 Windows 的 HDInsight 群集。如要了解如何创建基于 Linux 的群集，请参阅[使用 Azure 门户预览在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)。

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## 先决条件：
[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

在开始按照本文中的说明操作之前，你必须具有以下内容：

* Azure 订阅。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

### 访问控制要求
[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## 创建群集
**创建 HDInsight 群集**

1. 登录 [Azure 门户预览](https://portal.azure.cn)。
2. 依次单击“新建”、“智能 + 分析”、“HDInsight”。
3. 键入或选择以下值：
   
    * **群集名称**：给群集命名。
    * **订阅**：选择用于此群集的 Azure 订阅。
    * **群集配置**：

     * **群集类型**：对于本教程，选择“Hadoop”。
     * **操作系统**：对于本教程，选择“Windows”。
     * **版本**：选择 **Hadoop 2.7.0 (HDI 3.3)**。这是基于 Windows 的 Hadoop 群集的最新版本。
     * **群集层**：对于本教程，选择“标准”。

    * **凭据**：

     * **群集登录用户名**：默认名称为 **admin**。
     * **群集登录密码**：输入密码。
     * **启用远程桌面**：本教程中不需要远程桌面。

    * **数据源**：

     * **选择方法**：选择“从所有订阅”。
     * **选择存储帐户**：单击“新建”，然后输入默认存储帐户的名称。
     * **默认容器**：默认情况下，默认容器使用与群集相同的名称。
     * **位置**：选择最近的位置。此位置同时供群集和默认存储帐户使用。
    * **群集大小**：

     * **辅助角色节点数**：本教程仅需 1 个节点。
    * **高级配置**：对于本教程，可以跳过此部分。
    * **资源组**：选择“新建”，然后输入名称。

4. 选择“固定到仪表板”，然后单击“创建”。选择“固定到启动板”会将群集的磁贴添加到门户的启动板。创建群集大概需要 15-20 分钟。使用启动板上的磁贴或页面左侧的“通知”项检查预配进程。
5. 创建完成后，在启动板中单击群集磁贴，以启动群集边栏选项卡。群集边栏选项卡提供有关该群集的基本信息，如名称、其所属的资源组、位置、操作系统、群集仪表板 URL 等。

## 自定义群集
* 请参阅 [Customize HDInsight clusters using Bootstrap](/documentation/articles/hdinsight-hadoop-customize-cluster-bootstrap/)（使用 Bootstrap 自定义 HDInsight 群集）。
* 请参阅 [Customize Windows-based HDInsight clusters using Script Action](/documentation/articles/hdinsight-hadoop-customize-cluster/)（使用脚本操作自定义基于 Windows 的 HDInsight 群集）。

## 后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

* [Azure HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/) - 了解如何开始使用你的 HDInsight 群集
* [以编程方式提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/) - 了解如何以编程方式将作业提交到 HDInsight
* [使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-management-portal/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned and update the new portal's UI-->