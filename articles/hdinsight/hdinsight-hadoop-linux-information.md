<properties
    pageTitle="有关在基于 Linux 的 HDInsight 上使用 Hadoop 的提示 - Azure | Azure"
    description="获取有关在 Azure 云中运行的你所熟悉的 Linux 环境中使用基于 Linux 的 HDInsight (Hadoop) 群集的实施提示。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="c41c611c-5798-4c14-81cc-bed1e26b5609"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/02/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />  


# 有关在 Linux 上使用 HDInsight 的信息

Azure HDInsight 群集在熟悉的 Linux 环境中提供可在 Azure 云中运行的 Hadoop。在大多数情况下，它的工作方式应该与其他任何 Hadoop-on-Linux 安装完全相同。本文档指出了你应该注意的具体差异。

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## 先决条件

本文档中的许多步骤使用以下实用程序，这些程序可能需要在系统上安装。

* [cURL](https://curl.haxx.se/) - 用于与基于 Web 的服务通信
* [jq](https://stedolan.github.io/jq/) - 用于分析 JSON 文档
* [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)（预览版）- 用于远程管理 Azure 服务

    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## 用户

应将 HDInsight 视为**单用户**系统。使用群集时，将创建单个具有管理员级别权限的 SSH 用户帐户。可以创建其他 SSH 帐户，但这些帐户也具有对群集的管理员访问权限。

## 域名

从 Internet 连接到群集时要使用的完全限定域名 (FQDN) 是 **&lt;clustername>.azurehdinsight.cn** 或（仅限 SSH）**&lt;clustername-ssh>.azurehdinsight.cn**。

就内部来说，群集中的每个节点都有一个在群集配置期间分配的名称。若要查找群集名称，请参阅 Ambari Web UI 上的**主机**页。也可使用以下方法从 Ambari REST API 返回主机列表：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/hosts" | jq '.items[].Hosts.host_name'

将 **PASSWORD** 替换为管理员帐户的密码，将 **CLUSTERNAME** 替换为群集的名称。这会返回一个 JSON 文档，其中包含群集中主机的列表，然后 jq 会拉取群集中每个主机的 `host_name` 元素值。

若需查找特定服务的节点的名称，可查询 Ambari 以获取该组件。例如，若需查找 HDFS 名称节点的主机，请使用以下命令。

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'

此请求会返回一个描述该服务的 JSON 文档，然后 jq 就会只拉取主机的 `host_name` 值。

## 对服务的远程访问

* **Ambari (web)** - https://&lt;clustername>.azurehdinsight.cn

    使用群集管理员用户和密码进行身份验证，然后登录到 Ambari。必须使用群集管理员用户和密码进行身份验证。

    身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

    > [AZURE.IMPORTANT]
    虽然可以直接通过 Internet 访问群集的 Ambari，但若要使用某些功能，则需要根据访问群集所用的内部域名的节点来达到目的。由于这是内部域名且未公开，因此，在尝试通过 Internet 访问某些功能时，可能会出现“找不到服务器”的错误。
    ><p>
    > 若要使用 Ambari web UI 的全部功能，请使用 SSH 隧道通过代理将 Web 流量传送到群集头节点。请参阅[使用 SSH 隧道访问 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 和其他 Web UI](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)

* **Ambari (REST)** - https://&lt;clustername>.azurehdinsight.cn/ambari

    > [AZURE.NOTE]
    通过使用群集管理员用户和密码进行身份验证。
    ><p>
    > 身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

* **WebHCat (Templeton)** - https://&lt;clustername>.azurehdinsight.cn/templeton

    > [AZURE.NOTE]
    通过使用群集管理员用户和密码进行身份验证。
    ><p>
    > 身份验证是纯文本身份验证 - 始终使用 HTTPS 来帮助确保连接是安全的。

* **SSH** - &lt;clustername>-ssh.azurehdinsight.cn，使用端口 22 或 23。端口 22 用于连接主要头节点，而端口 23 用于连接辅助头节点。有关头节点的详细信息，请参阅 [HDInsight 中的 Hadoop 群集的可用性和可靠性](/documentation/articles/hdinsight-high-availability-linux/)。

    > [AZURE.NOTE]
    只能通过 SSH 从客户端计算机访问群集头节点。在连接后，可以通过使用 SSH 从头节点访问从节点。

## 文件位置

Hadoop 相关文件可在群集节点上的 `/usr/hdp` 中找到。此目录包含以下子目录：

* **2.2.4.9-1**：此目录是根据 HDInsight 使用的 Hortonworks 数据平台版本命名的，因此你的群集上的编号可能不同于此处列出的编号。
* **current**：此目录包含 **2.2.4.9-1** 目录下的目录的链接，有了此目录，你每次访问某个文件时，便不需要键入版本号（可能会变化）。

示例数据和 JAR 文件可以在 Hadoop 分布式文件系统 (HDFS) 或 Azure Blob 存储上的 `/example` 和 `/HdiSamples` 处找到。

## HDFS 和 Blob 存储

在大部分的 Hadoop 分发中，HDFS 受群集中计算机上的本地存储的支持。尽管这种方式很有效率，但用于基于云的解决方案时可能费用高昂，因为计算资源以小时或分钟为单位计费。

HDInsight 使用 Azure Blob 存储作为默认存储。这可以提供以下优点：

* 成本低廉的长期存储
* 可从外部服务访问，例如网站、文件上载/下载实用程序、各种语言 SDK 和 Web 浏览器

> [AZURE.IMPORTANT]
Blob 存储容量最多为 4.75 TB，而单个 Blob（从 HDInsight 角度来说是文件）大小最多为 195 GB。

使用 Azure 存储时，通常不需要从 HDInsight 执行任何特殊操作即可访问数据。例如，以下命令会列出 `/example/data` 文件夹中的文件：

    hdfs dfs -ls /example/data

### URI 和方案

在访问文件时，一些命令可能需要用户将方案指定为 URI 的一部分。例如，Storm-HDFS 组件需要用户指定方案。使用非默认存储（作为“额外”存储添加到群集的存储）时，必须始终将方案用作 URI 的一部分。

使用 __Blob 存储__时，方案可为以下其中一种：

* `wasb:///`：使用未经加密的通信访问默认存储。

* `wasbs:///`：使用加密通信访问默认存储。

* `wasbs://<container-name>@<account-name>.blob.core.chinacloudapi.cn/`：与非默认存储帐户通信时使用。例如，具有其他存储帐户或访问可公开访问的存储帐户中存储的数据时。

### 群集使用的是哪种存储

可以使用 Ambari 来检索群集的默认存储配置。可以使用以下命令通过 curl 检索 HDFS 配置信息，然后使用 [jq](https://stedolan.github.io/jq/) 对其进行筛选：

```curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'```  

> [AZURE.NOTE]
以上代码返回应用到服务器的第一个配置 (`service_config_version=1`)，其中包含此信息。如果要检索创建群集后修改的值，可能需要列出配置版本并检索最新版本。

此命令将返回类似于以下的值：

* 如果使用的是 Azure 存储帐户，则为 `wasbs://<container-name>@<account-name>.blob.core.chinacloudapi.cn`。

    帐户名称是 Azure 存储帐户的名称，容器名称是作为群集存储的根的 Blob 容器。

也可使用 Azure 门户预览按照以下步骤查找存储信息：

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，选择 HDInsight 群集。

2. 从“属性”部分，选择“存储帐户”。这会显示群集的存储信息。

### 如何从 HDInsight 外部访问文件

从 HDInsight 群集外部访问数据的方法有多种。以下是一些可用于处理数据的实用工具和 SDK 的链接：

如果使用的是 __Azure 存储__，请参阅以下链接了解可用于访问数据的方式：

* [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)：适用于 Azure 的命令行接口命令。在安装后，使用 `az storage` 命令获取有关使用存储的帮助，或者使用 `az storage blob` 获取特定于 Blob 的命令。
* [blobxfer.py](https://github.com/Azure/azure-batch-samples/tree/master/Python/Storage)：用于 Azure 存储中的 Blob 的 python 脚本。
* 多种 SDK：

    * [Java](https://github.com/Azure/azure-sdk-for-java)
    * [Node.js](https://github.com/Azure/azure-sdk-for-node)
    * [PHP](https://github.com/Azure/azure-sdk-for-php)
    * [Python](https://github.com/Azure/azure-sdk-for-python)
    * [Ruby](https://github.com/Azure/azure-sdk-for-ruby)
    * [.NET](https://github.com/Azure/azure-sdk-for-net)
    * [存储 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd135733.aspx)

## <a name="scaling"></a>缩放你的群集

使用群集缩放功能，可更改群集使用的数据节点数，而无需删除然后再重新创建群集。你可以在其他作业或进程正在群集上运行时执行缩放操作。

不同的群集类型会受缩放操作影响，如下所示：

* **Hadoop**：减少群集中的节点数时，群集中的某些服务将重新启动。这会导致正在运行或挂起的作业在缩放操作完成时失败。你可以在操作完成后重新提交这些作业。
* **HBase**：在完成缩放操作后的几分钟内，区域服务器会自动进行平衡。若要手动平衡区域服务器，请使用以下步骤：

    1. 使用 SSH 连接到 HDInsight 群集。有关如何将 SSH 与 HDInsight 配合使用的详细信息，请参阅以下文档之一：

        * [在 Linux、Unix 和 Mac OS X 上将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
        * [在 Windows 上将 SSH (PuTTY) 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)

    2. 使用以下命令来启动 HBase shell：

            hbase shell
    
    3. 加载 HBase shell 后，使用以下方法来手动平衡区域服务器：

            balancer

* **Storm**：你应在执行缩放操作后重新平衡任何正在运行的 Storm 拓扑。这允许拓扑根据群集中的新节点数重新调整并行度设置。若要重新平衡正在运行的拓扑，请使用下列选项之一：

    * **SSH**：连接到服务器并使用以下命令来重新平衡拓扑：

            storm rebalance TOPOLOGYNAME

        你还可以指定参数来替代拓扑原来提供的并行度提示。例如，`storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10` 会将拓扑重新配置为 5 个辅助角色进程，蓝色的 BlueSpout 组件有 3 个 executor，黄色 YellowBolt 组件有 10 个 executor。

    * **Storm UI**：使用以下步骤来重新平衡使用 Storm UI 的拓扑。

        1. 在 Web 浏览器中打开 **https://CLUSTERNAME.azurehdinsight.cn/stormui**，其中 CLUSTERNAME 是 Storm 群集的名称。如果出现提示，请输入在创建 HDInsight 群集时指定的群集管理员用户名和密码。
        2. 选择要重新平衡的拓扑，然后选择“重新平衡”按钮。输入执行重新平衡操作前的延迟。

有关缩放 HDInsight 群集的特定信息，请参阅：

* [使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-portal-linux/#scale-clusters)
* [使用 Azure PowerShell 管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-command-line/#scale-clusters)

## 如何安装 Hue（或其他 Hadoop 组件）？

HDInsight 是一项托管服务，这意味着如果检测到问题，Azure 可能会自动破坏并重新预配群集中的节点。因此，不建议直接在群集节点上手动安装内容。请在需要安装以下内容时改用 [HDInsight 脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster/)：

* 服务或网站，例如 Spark 或 Hue。
* 需要在群集的多个节点上进行配置更改的组件。例如，必需的环境变量、创建日志记录目录，或者创建配置文件。

脚本操作是在群集预配期间运行的 Bash 脚本，可用于在群集上安装和配置其他组件。提供了用于安装以下组件的示例脚本：

* [Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)
* [Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-linux/)
* [Solr](/documentation/articles/hdinsight-hadoop-solr-install-linux/)

有关开发你自己的脚本操作的信息，请参阅[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions-linux/)。

### Jar 文件

某些 Hadoop 技术以自包含 jar 文件形式提供，这些文件包含某些函数，这些函数用作 MapReduce 作业的一部分，或来自 Pig 或 Hive 内部。这些技术可以使用脚本操作安装，但通常不需任何设置，在预配后上载到群集即可直接使用。如需确保该组件在重置群集映像后仍然存在，可将 jar 文件存储在群集 (WASB or ADL) 的默认存储中。

例如，如需使用最新版本的 [DataFu](http://datafu.incubator.apache.org/)，可下载包含该项目的 jar，然后将其上载到 HDInsight 群集。然后按照 DataFu 文档的说明通过 Pig 或 Hive 使用它。

> [AZURE.IMPORTANT]
某些属于独立 jar 文件的组件通过 HDInsight 提供，但不在路径中。若要查找特定组件，可使用以下命令在群集上搜索：
><p>
> ```find / -name *componentname*.jar 2>/dev/null```  
><p>
> 这会返回任何匹配的 jar 文件的路径。

如果群集已经以独立 jar 文件的形式提供了某个版本的组件，但用户需要使用其他版本，则可将新版组件上载到群集，然后尝试在作业中使用该组件。

> [AZURE.WARNING]
完全支持通过 HDInsight 群集提供的组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。
><p>
> 自定义组件可获得合理范围的支持，有助于进一步解决问题。这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=hdinsight)和 [Azure CSDN](http://azure.csdn.net)。此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)、[Spark](http://spark.apache.org/)。

## 后续步骤

* [从基于 Windows 的 HDInsight 迁移到基于 Linux 的 HDInsight](/documentation/articles/hdinsight-migrate-from-windows-to-linux/)
* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned and update the details of "HDFS and Storage"-->