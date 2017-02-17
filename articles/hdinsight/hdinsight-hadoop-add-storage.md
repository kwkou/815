<properties
    pageTitle="将其他 Azure 存储帐户添加到 HDInsight | Azure"
    description="了解如何将其他 Azure 存储帐户添加到现有 HDInsight 群集。"
    services="hdinsight"
    documentationCenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.service="hdinsight"
    ms.devlang=""
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="11/28/2016"
    wacn.date="02/06/2017"
    ms.author="larryfr" />

# 将其他 Azure 存储帐户添加到 HDInsight

了解如何使用脚本操作将其他 Azure 存储帐户添加到现有 HDInsight 群集。

> [AZURE.IMPORTANT]
本文档中的信息是关于在创建群集后将其他存储添加到群集。有关在群集创建过程中添加其他存储帐户的信息，请参阅[创建基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/#use-additional-storage)文档的__使用其他存储__部分。

## 工作原理

此脚本采用以下参数：

* __Azure 存储帐户名称__：要添加到 HDInsight 群集的存储帐户的名称。运行脚本后，HDInsight 将能够读取此存储帐户中存储的数据并将数据写入到此存储帐户中。

* __Azure 存储帐户密钥__：用于授予对存储帐户的访问权限的密钥。

* __-p__（可选）：如果指定，密钥将不会加密并以纯文本格式存储在 core-site.xml 文件中。

在处理期间，脚本将执行以下操作：

* 如果群集的 core-site.xml 配置中已存在该存储帐户，脚本将退出，并不执行任何进一步操作。

* 使用密钥验证该存储帐户是否存在并且是否可以访问。

* 使用群集凭据对密钥进行加密。这可以使 HDInsight 用户无法轻松地从 Ambari 提取并使用存储帐户密钥。

* 将存储帐户添加到 core-site.xml 文件中。

* 停止并重新启动 Oozie、YARN、MapReduce2 和 HDFS 服务，使这些服务选取新的存储帐户信息。

> [AZURE.WARNING]
如果存储帐户与 HDInsight 群集不在同一个区域中，你可能会遇到性能不佳的情况。访问不同区域中的数据会在区域 Azure 数据中心外部跨公共 Internet 发送网络流量，从而会导致延迟。此外，将数据发送出区域的数据中心可能成本更高，因为数据离开数据中心时会产生出口费用。

## 脚本

__脚本位置__：[https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh)

__要求__：

* 必须将该脚本应用于__头节点__。

## 使用脚本

有关通过 Azure 门户预览、Azure PowerShell 和 Azure CLI 使用脚本操作的信息，请参阅[使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#apply-a-script-action-to-a-running-cluster)文档的“将脚本操作应用于正在运行的群集”部分。

使用自定义文档中提供的信息时，请将任何示例脚本操作 URI 替换为此脚本的 URI (https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh)。将任何示例参数替换为要添加到群集的存储帐户的 Azure 存储帐户名称和密钥。

> [AZURE.NOTE]
无需将此脚本标记为__持久化__，因为它直接更新群集的 Ambari 配置。

## 已知问题

### Azure 门户预览或工具中未显示存储帐户

在 Azure 门户预览中查看 HDInsight 群集时，选择“属性”下的存储帐户项不会显示通过此脚本操作添加的存储帐户。Azure PowerShell 和 Azure CLI 也不显示其他存储帐户。

这是因为该脚本只修改群集的 core-site.xml 配置。使用 Azure 管理 API 检索群集信息时，当前未使用此信息。

若要使用此脚本查看添加到群集的存储帐户信息，请使用 Ambari REST API。以下命令演示如何使用 [cURL (http://curl.haxx.se/)](http://curl.haxx.se/) 和 [jq (https://stedolan.github.io/jq/)](https://stedolan.github.io/jq/) 通过 Ambari 检索和分析 JSON 数据：

> ```PowerShell
> curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["""fs.azure.account.key.STORAGEACCOUNT.blob.core.chinacloudapi.cn"""] | select(. != null)'
> ```
> ```Bash
> curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.azure.account.key.STORAGEACCOUNT.blob.core.chinacloudapi.cn"] | select(. != null)'
> ```

使用此命令时，将 __CLUSTERNAME__ 替换为 HDInsight 群集的名称。将 __PASSWORD__ 替换为群集的 HTTP 登录密码。将 __STORAGEACCOUNT__ 替换为使用脚本操作添加的存储帐户的名称。此命令返回的信息类似于以下文本：

    "MIIB+gYJKoZIhvcNAQcDoIIB6zCCAecCAQAxggFaMIIBVgIBADA+MCoxKDAmBgNVBAMTH2RiZW5jcnlwdGlvbi5henVyZWhkaW5zaWdodC5uZXQCEA6GDZMW1oiESKFHFOOEgjcwDQYJKoZIhvcNAQEBBQAEggEATIuO8MJ45KEQAYBQld7WaRkJOWqaCLwFub9zNpscrquA2f3o0emy9Vr6vu5cD3GTt7PmaAF0pvssbKVMf/Z8yRpHmeezSco2y7e9Qd7xJKRLYtRHm80fsjiBHSW9CYkQwxHaOqdR7DBhZyhnj+DHhODsIO2FGM8MxWk4fgBRVO6CZ5eTmZ6KVR8wYbFLi8YZXb7GkUEeSn2PsjrKGiQjtpXw1RAyanCagr5vlg8CicZg1HuhCHWf/RYFWM3EBbVz+uFZPR3BqTgbvBhWYXRJaISwssvxotppe0ikevnEgaBYrflB2P+PVrwPTZ7f36HQcn4ifY1WRJQ4qRaUxdYEfzCBgwYJKoZIhvcNAQcBMBQGCCqGSIb3DQMHBAhRdscgRV3wmYBg3j/T1aEnO3wLWCRpgZa16MWqmfQPuansKHjLwbZjTpeirqUAQpZVyXdK/w4gKlK+t1heNsNo1Wwqu+Y47bSAX1k9Ud7+Ed2oETDI7724IJ213YeGxvu4Ngcf2eHW+FRK"

此文本是用于访问存储帐户的加密密钥的示例。

### 更改密钥后，无法访问存储

如果更改了存储帐户的密钥，HDInsight 将不再能够访问存储帐户。

这是因为群集的 core-site.xml 中存储的密钥是旧密钥。

重新运行脚本操作__不__会更新密钥，因为脚本会检查该存储帐户的条目是否已存在。如果存在，它不进行任何更改。

若要解决此问题，必须删除存储帐户的现有条目。可以按照以下步骤来执行此操作：

1. 在 Web 浏览器中，打开 HDInsight 群集的 Ambari Web UI。该 URI 为 https://CLUSTERNAME.azurehdinsight.cn。将 __CLUSTERNAME__ 替换为群集名称。

    出现提示时，输入群集的 HTTP 登录用户和密码。

2. 从页面左侧的服务列表中选择“HDFS”。然后选择页中心的“配置”选项卡。

3. 在“筛选器...”字段中，输入值“fs.azure.account”。这将返回已添加到群集的任何其他存储帐户的条目。有两种类型的条目：__keyprovider__ 和 __key__。这两种条目都将在密钥名称中包含存储帐户的名称。

    以下是名为 __mystorage__ 的存储帐户的示例条目：

        fs.azure.account.keyprovider.mystorage.blob.core.chinacloudapi.cn
        fs.azure.account.key.mystorage.blob.core.chinacloudapi.cn

4. 确定该存储帐户的密钥后，需要使用该条目右侧的红色“-”图标删除该条目。然后使用“保存”按钮保存更改。

5. 保存更改后，使用脚本操作将该存储帐户和新的密钥值添加到群集。

### 性能不佳

如果存储帐户与 HDInsight 群集不在同一个区域中，你可能会遇到性能不佳的情况。访问不同区域中的数据会在区域 Azure 数据中心外部跨公共 Internet 发送网络流量，从而会导致延迟。

### 额外费用

如果存储帐户与 HDInsight 群集不在同一个区域中，你可能会在 Azure 帐单上发现额外出口费用。当数据离开区域数据中心时，会产生出口费用，即使流量的目标为不同区域中的另一个 Azure 数据中心，也是如此。

## 后续步骤

在本文档中，已学习如何将其他存储帐户添加到现有 HDInsight 群集。有关脚本操作的详细信息，请参阅 [Customize Linux-based HDInsight clusters using script action](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)（使用脚本操作自定义基于 Linux 的 HDInsight 群集）

<!---HONumber=Mooncake_0103_2017-->