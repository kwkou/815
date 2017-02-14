<!-- not suitable for Mooncake -->

<properties
    pageTitle="使用 Apache Ambari REST API 监视和管理 HDInsight 群集 | Azure"
    description="了解如何使用 Ambari 监视和管理基于 Linux 的 HDInsight 群集。在本文档中，你将学习如何使用 HDInsight 群集随附的 Ambari REST API。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="2400530f-92b3-47b7-aa48-875f028765ff"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="12/02/2016"
    wacn.date="02/06/2017"
    ms.author="larryfr" />

# 使用 Ambari REST API 管理 HDInsight 群集

[AZURE.INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

Apache Ambari 提供简单易用的 Web UI 和 REST API 来简化 Hadoop 群集的管理和监视。基于 Linux 的 HDInsight 群集包含 Ambari，用于监视群集和进行配置更改。在本文中，可以了解通过使用 cURL 执行常见任务来使用 Ambari REST API 的基础知识。

> [AZURE.IMPORTANT]
已使用 Windows 10 上的 PowerShell 和 Bash 对本文档中的示例进行了测试。在许多情况下，同一命令适用于这两者。在有差异的情况下，同时提供了 PowerShell 和 Bash 示例。

## 先决条件

* [cURL](http://curl.haxx.se/)：cURL 是一个跨平台实用工具，可用于从命令行使用 REST API。在本文档中，将使用它来与 Ambari REST API 通信。

    > [AZURE.WARNING]
    如果使用 PowerShell，则必须使用 `remove-item alias:curl` 命令删除 `curl` 命令的默认别名。如果不删除别名，则将收到错误，例如“由于参数名称 'u' 不明确，无法处理参数”。

* [jq](https://stedolan.github.io/jq/)：jq 是用于处理 JSON 文档的跨平台命令行实用工具。在本文档中，将使用它来分析从 Ambari REST API 返回的 JSON 文档。

* [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2)（预览版）：用于使用 Azure 服务的跨平台命令行实用工具。

    [AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a id="whatis"></a>什么是 Ambari？

[Apache Ambari](http://ambari.apache.org) 提供简单易用的 Web UI，可用于预配、管理和监视 Hadoop 群集，以此简化 Hadoop 管理。开发人员可以使用 [Ambari REST API](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md) 在其应用程序中集成这些功能。

基于 Linux 的 HDInsight 群集已按默认提供 Ambari。

## REST API

HDInsight 上的 Ambari REST API 的基本 URI 为 https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME，其中 **CLUSTERNAME** 是群集的名称。

> [AZURE.IMPORTANT]
URI 的完全限定域名 (FQDN) 部分 (CLUSTERNAME.azurehdinsight.cn) 中的群集名称不区分大小写，但 URI 中的其他部分则区分大小写。例如，如果群集名称为 MyCluster，则有效的 URI 如下：
> <p>
> `https://mycluster.azurehdinsight.cn/api/v1/clusters/MyCluster` 
`https://MyCluster.azurehdinsight.cn/api/v1/clusters/MyCluster`
> <p>
> 下面的 URI 返回一个错误，因为第二个出现的名称的大小写不正确。
> <p>
> `https://mycluster.azurehdinsight.cn/api/v1/clusters/mycluster` 
`https://MyCluster.azurehdinsight.cn/api/v1/clusters/mycluster`

连接到 HDInsight 上的 Ambari 需要 HTTPS。对连接进行身份验证时，必须使用创建群集时提供的管理员帐户名（默认为 **admin**）和密码。

下面是使用 cURL 对 REST API 发出 GET 请求的示例：将 **PASSWORD** 替换为群集的管理员密码。将 **CLUSTERNAME** 替换为群集的名称：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME"

响应是 JSON 文档，以类似于下面的内容的信息开头：

    {
    "href" : "http://10.0.0.10:8080/api/v1/clusters/CLUSTERNAME",
    "Clusters" : {
        "cluster_id" : 2,
        "cluster_name" : "CLUSTERNAME",
        "health_report" : {
        "Host/stale_config" : 0,
        "Host/maintenance_state" : 0,
        "Host/host_state/HEALTHY" : 7,
        "Host/host_state/UNHEALTHY" : 0,
        "Host/host_state/HEARTBEAT_LOST" : 0,
        "Host/host_state/INIT" : 0,
        "Host/host_status/HEALTHY" : 7,
        "Host/host_status/UNHEALTHY" : 0,
        "Host/host_status/UNKNOWN" : 0,
        "Host/host_status/ALERT" : 0

由于这是一个 JSON 文档，因此使用 JSON 解析器来处理数据通常会更简单。例如，下面的示例使用 jq 仅显示 `health_report` 元素。

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME" | jq '.Clusters.health_report'

## <a name="example-get-the-fqdn-of-cluster-nodes"></a> 示例：获取群集节点的 FQDN

使用 HDInsight 时，可能需要知道群集节点的完全限定域名 (FQDN)。可以使用以下命令轻松检索群集中各个节点的 FQDN：

* **头节点**：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/HDFS/components/NAMENODE" | jq '.host_components[].HostRoles.host_name'`

* **工作节点**：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/HDFS/components/DATANODE" | jq '.host_components[].HostRoles.host_name'`

* **Zookeeper 节点**：`curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq '.host_components[].HostRoles.host_name'`

请注意上述每个示例遵循相同的模式：

1. 查询在这些节点上运行的已知组件。

2. 检索 `host_name` 元素，该元素包含这些节点的 FQDN。

返回文档的 `host_components` 元素包含多个项。使用 `.host_components[]`，然后在该元素内指定一个路径，这样可以循环访问每个项，并从特定的路径提取值。如果你只想指定一个值（例如第一个 FQDN 条目），可以集合形式返回项，然后选择特定的条目：

    jq '[.host_components[].HostRoles.host_name][0]'

该代码返回集合中的第一个 FQDN。

## 示例：获取默认存储帐户和容器

创建 HDInsight 群集时，必须使用 Azure 存储帐户和 Blob 容器作为群集的默认存储。创建群集后，可以使用 Ambari 来检索此信息。例如，如果你要以编程方式将数据直接写入容器。

以下命令将检索群集默认存储的 WASB URI：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'

PowerShell 对于使用单引号和双引号具有略有不同的规则。使用 PowerShell 中的以下命令：

    curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" | jq '.items[].configurations[].properties["""fs.defaultFS"""] | select(. != null)'

> [AZURE.NOTE]
以上代码返回应用到服务器的第一个配置 (`service_config_version=1`)，其中包含此信息。如果要检索创建群集后修改的值，可能需要列出配置版本并检索最新版本。

上述代码返回的值类似于下面示例，其中，**CONTAINER** 是默认容器，**ACCOUNTNAME** 是 Azure 存储帐户名：

    wasbs://CONTAINER@ACCOUNTNAME.blob.core.chinacloudapi.cn

然后，可以在 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2) 中使用此信息上载数据到容器或者从容器下载数据。

1. 使用以下命令获取存储帐户的唯一 ID。在以下命令中，将 **ACCOUNTNAME** 替换为从 Ambari 检索的存储帐户名：

        az storage account list --query "[?name=='ACCOUNTNAME'].id" --out list

2. 使用以下命令获取存储帐户的密钥。将 **STORAGEID** 替换为存储帐户 ID：

        az storage account keys list --ids STORAGEID --query "keys[?keyName=='key1'].value" --out list

    这将返回帐户的主密钥。

3. 使用 upload 命令将文件存储在容器中：
   
        az storage blob upload --account-name ACOUNTNAME --account-key ACCOUNTKEY -f FILEPATH -c CONTAINER -n BLOBPATH
   
    将 **ACCOUNTNAME** 替换为存储帐户名。将 **ACCOUNTKEY** 替换为前面检索到的密钥。将 __CONTAINER__ 替换为容器名称。**FILEPATH** 是要上载的文件的路径，**BLOBPATH** 是容器中的路径。
   
    例如，如果希望文件出现在 HDInsight 上的 wasbs://example/data/filename.txt 中，则 **BLOBPATH** 为 `example/data/filename.txt`。

## 示例：更新 Ambari 配置

1. 获取当前配置，即 Ambari 存储为“所需配置”的配置：
   
        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME?fields=Clusters/desired_configs"
   
    该示例返回一个 JSON 文档，其中包含群集上安装的组件的当前配置（由 *tag* 值标识）。下面的示例是从 Spark 群集类型返回的数据摘录。
   
        "spark-metrics-properties" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        },
        "spark-thrift-fairscheduler" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        },
        "spark-thrift-sparkconf" : {
            "tag" : "INITIAL",
            "user" : "admin",
            "version" : 1
        }
   
    需要从此列表中复制组件的名称（例如，**spark\_thrift\_sparkconf** 和 **tag** 值）。

2. 使用以下命令检索组件和标记的配置。将 **spark-thrift-sparkconf** 和 **INITIAL** 替换为要检索其配置的组件和标记。

        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations?type=spark-thrift-sparkconf&tag=INITIAL" | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json

<br/>  


        $epoch = Get-Date -Year 1970 -Month 1 -Day 1 -Hour 0 -Minute 0 -Second 0
        $now = Get-Date
        $unixTimeStamp = [math]::truncate($now.ToUniversalTime().Subtract($epoch).TotalMilliSeconds)

        curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/configurations?type=spark-thrift-sparkconf&tag=INITIAL" | jq --arg newtag "version$unixTimeStamp" '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json

Curl 检索 JSON 文档，然后使用 jq 修改数据以创建模板。然后使用模板添加/修改配置值。具体操作如下：

* 创造一个值，这个值包含“版本”字符串和日期，然后保存在“newtag”。

* 为新的所需配置创建根文档。

* 得到 `.items[]` 数组的内容，然后加到“desired_config”元素下。

* 删除“href”、“version”和“Config”元素，因为新建配置是不需要这些元素的。

* 添加一个新的“tag”元素，然后把值设置为“version#################”。数值部分根据当钱的日期来设置。每一个配置都得有一个独特的标签。

    最后，数据保存到“newconfig.json”文档。这个文档的结构应该跟以下示例类似：

        {
            "Clusters": {
                "desired_config": {
                "tag": "version1459260185774265400",
                "type": "spark-thrift-sparkconf",
                "properties": {
                    ....
                },
                "properties_attributes": {
                    ....
                }
            }
        }

3. 打开 **newconfig.json** 文档并在 **properties** 对象中修改/添加值。例如，下面的示例将“spark.yarn.am.memory”的值从“1g”更改为“3g”，并针对值为“256m”的“spark.kryoserializer.buffer.max”添加新元素。
   
        "spark.yarn.am.memory": "3g",
        "spark.kyroserializer.buffer.max": "256m",
   
    完成修改后，保存该文件。
4. 使用以下命令将更新的配置提交到 Ambari。
   
        cat newconfig.json | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME"
   
    此命令将 **newconfig.json** 文件的内容输送到 curl 请求，后者会将它作为新的所需配置提交到群集。cURL 请求返回 JSON 文档。此文档中的 **versionTag** 元素应该与提交的版本相匹配，并且 **configs** 对象包含你请求的配置更改。

### 示例：重新启动服务组件

此时，如果你查看 Ambari Web UI，会发现 Spark 服务指出需要将它重新启动才能使新配置生效。使用以下步骤重新启动该服务。

1. 使用以下命令启用 Spark 服务的维护模式。
   
        echo '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/SPARK"
   
    该命令将 JSON 文档发送到启用维护模式的服务器（包含在 `echo` 语句中）。可以使用以下请求来验证服务当前是否处于维护模式：
   
        curl -u admin:PASSWORD -H "X-Requested-By: ambari" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/SPARK" | jq .ServiceInfo.maintenance_state
   
    此时返回 `"ON"` 值。

2. 接下来，使用以下命令关闭服务：

        echo '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/SPARK"

此命令返回如下响应。

        {
            "href" : "http://10.0.0.18:8080/api/v1/clusters/CLUSTERNAME/requests/29",
            "Requests" : {
                "id" : 29,
                "status" : "Accepted"
            }
        }

此 URI 返回的 `href` 值正在使用群集节点的内部 IP 地址。若要从群集外部使用该地址，请将“10.0.0.18:8080”部分替换为群集的 FQDN。例如，以下命令检索请求的状态。

        curl -u admin:PASSWORD -H "X-Requested-By: ambari" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/requests/29" | jq .Requests.request_status

如果此命令返回值 `"COMPLETED"`，则表示请求已完成。

3. 完成前一个请求后，请使用以下命令来启动服务。
   
        echo '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/SPARK"
   
    服务重启后，即为新的配置设置。

4. 最后，使用以下命令关闭维护模式。
   
        echo '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' | curl -u admin:PASSWORD -H "X-Requested-By: ambari" -X PUT -d "@-" "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/SPARK"

## 后续步骤

有关 REST API 的完整参考，请参阅 [Ambari API 参考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

<!---HONumber=Mooncake_0103_2017-->