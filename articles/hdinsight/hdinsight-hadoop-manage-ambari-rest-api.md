<properties
    pageTitle="使用 Ambari REST API 监视和管理 Azure HDInsight |Azure"
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
    ms.date="02/16/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />  


# 使用 Ambari REST API 管理 HDInsight 群集

[AZURE.INCLUDE [ambari-selector](../../includes/hdinsight-ambari-selector.md)]

Apache Ambari 提供简单易用的 Web UI 和 REST API 来简化 Hadoop 群集的管理和监视。Ambari 包含在使用 Linux 操作系统的 HDInsight 群集上，用于监视群集和进行配置更改。通过本文，可了解有关使用 Ambari REST API 的基础知识。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a id="whatis"></a>什么是 Ambari

[Apache Ambari](http://ambari.apache.org) 提供简单易用的 Web UI，可用于预配、管理和监视 Hadoop 群集，以此简化 Hadoop 管理。开发人员可以使用 [Ambari REST API](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md) 在其应用程序中集成这些功能。

基于 Linux 的 HDInsight 群集已按默认提供 Ambari。

## 如何使用 Ambari REST API

> [AZURE.IMPORTANT]
本文档中的信息和示例需要使用 Linux 操作系统的 HDInsight 群集。有关详细信息，请参阅 [HDInsight 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。

本文档中的示例同时适用于 Bourne shell (bash) 和 PowerShell。Bash 示例使用 GNU bash 4.3.11 进行了测试，但也应适用于其他 Unix shell。PowerShell 示例使用 PowerShell 5.0 进行了测试，但也应适用于 PowerShell 3.0 或更高版本。

如果使用 __Bourne shell__ (Bash)，则必须安装以下工具：

* [cURL](http://curl.haxx.se/)：cURL 是一个可用于从命令行使用 REST API 的实用工具。在本文档中，将使用它来与 Ambari REST API 通信。

不论使用 Bash 还是 PowerShell，还必须安装 [jq](https://stedolan.github.io/jq/)。Jq 是用于处理 JSON 文档的实用工具。**所有** Bash 示例以及其中**一个** PowerShell 示例中都使用到此工具。

### 用于 Ambari Rest API 的基 URI

HDInsight 上的 Ambari REST API 的基本 URI 为 https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME，其中 **CLUSTERNAME** 是群集的名称。

> [AZURE.IMPORTANT]
URI 的完全限定域名 (FQDN) 部分 (CLUSTERNAME.azurehdinsight.cn) 中的群集名称不区分大小写，但 URI 中的其他部分则区分大小写。例如，如果群集命名为 `MyCluster`，则有效的 URI 如下所示：
> <p> 
> `https://mycluster.azurehdinsight.cn/api/v1/clusters/MyCluster`  
><p>
> `https://MyCluster.azurehdinsight.cn/api/v1/clusters/MyCluster`  
> <p> 
> 下面的 URI 返回一个错误，因为第二个出现的名称的大小写不正确。
> <p> 
> `https://mycluster.azurehdinsight.cn/api/v1/clusters/mycluster`  
> <p>
> `https://MyCluster.azurehdinsight.cn/api/v1/clusters/mycluster`  


### 身份验证

连接到 HDInsight 上的 Ambari 需要 HTTPS。对连接进行身份验证时，必须使用创建群集时提供的管理员帐户名（默认为 **admin**）和密码。

## 示例：身份验证和分析 JSON

以下示例演示如何针对基本 Ambari REST API 发出 GET 请求：

    curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME"

> [AZURE.IMPORTANT]
本文档中的 Bash 示例作出以下假设：
><p>
><p> *群集的登录名是 `admin` 的默认值。
><p> * `$PASSWORD` 包含 HDInsight 登录命令的密码。可使用 `PASSWORD='mypassword'` 设置该值。
><p> * `$CLUSTERNAME` 包含群集名称。可使用 `set CLUSTERNAME='clustername'` 设置该值。

    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName" `
        -Credential $creds
    $resp.Content

> [AZURE.IMPORTANT]
本文档中的 PowerShell 示例作出以下假设：
><p>
><p> * `$creds` 是一个凭据对象，包含用于群集的管理员登录名和密码。通过使用 `$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"` 并在系统提示时提供密码，可设置该值。
><p> * `$clusterName` 是一个包含群集名称的字符串。可使用 `$clusterName="clustername"` 设置该值。

两个示例均返回一个 JSON 文档，该文档以类似于如下示例的信息开头：

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
        ...

### 分析 JSON 数据

以下示例使用 `jq` 分析 JSON 响应文档，并仅显示结果中的 `health_report` 信息。

    curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME" \
    | jq '.Clusters.health_report'

PowerShell 3.0 及更高版本提供 `ConvertFrom-Json` cmdlet，它会将 JSON 文档转换成一个更易于从 PowerShell 处理的对象。如下示例使用 `ConvertFrom-Json` 来仅显示结果中的 `health_report` 信息。

    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.Clusters.health_report

> [AZURE.NOTE]
虽然本文档中的大多数示例使用 `ConvertFrom-Json` 显示响应文档中的元素，但是[更新 Ambari 配置](#example-update-ambari-configuration)示例使用 jq。本示例中使用 Jq 从 JSON 响应文档构造一个新模板。

有关 REST API 的完整参考，请参阅 [Ambari API 参考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

## <a name="example-get-the-fqdn-of-cluster-nodes"></a>示例：获取群集节点的 FQDN

使用 HDInsight 时，可能需要知道群集节点的完全限定域名 (FQDN)。可使用以下示例轻松检索群集中各个节点的 FQDN：

* **所有节点**

        curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/hosts" \
        | jq '.items[].Hosts.host_name'

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/hosts" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.items.Hosts.host_name

* **头节点**

        curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/HDFS/components/NAMENODE" \
        | jq '.host_components[].HostRoles.host_name'

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/HDFS/components/NAMENODE" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.host_components.HostRoles.host_name

* **辅助角色节点**

        curl -u admin:PASSWORD -sS -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/HDFS/components/DATANODE" \
        | jq '.host_components[].HostRoles.host_name'

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/HDFS/components/DATANODE" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.host_components.HostRoles.host_name

* **Zookeeper 节点**

        curl -u admin:PASSWORD -sS -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" \
        | jq '.host_components[].HostRoles.host_name'

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.host_components.HostRoles.host_name

## 示例：获取群集节点的内部 IP 地址

> [AZURE.IMPORTANT]
本部分中的示例所返回的 IP 地址不可直接通过 Internet 进行访问。仅可在包含 HDInsight 群集的 Azure 虚拟网络内部对其进行访问。
><p>
> 有关使用 HDInsight 和虚拟网络的详细信息，请参阅[通过使用自定义 Azure 虚拟网络扩展 HDInsight 功能](/documentation/articles/hdinsight-extend-hadoop-virtual-network/)。

必须首先知道主机的 FQDN 才可获取其 IP 地址。拥有 FQDN 后即可获取主机的 IP 地址。下面的示例首先会向 Ambari 查询所有主机节点的 FQDN，然后再向 Ambari 查询每个主机的 IP 地址。

    for HOSTNAME in $(curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/hosts" | jq -r '.items[].Hosts.host_name')
    do
        IP=$(curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/hosts/$HOSTNAME" | jq -r '.Hosts.ip')
      echo "$HOSTNAME <--> $IP"
    done

<br/>  

    $uri = "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/hosts"
    $resp = Invoke-WebRequest -Uri $uri -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    foreach($item in $respObj.items) {
        $hostName = [string]$item.Hosts.host_name
        $hostInfoResp = Invoke-WebRequest -Uri "$uri/$hostName" `
            -Credential $creds
        $hostInfoObj = ConvertFrom-Json $hostInfoResp 
        $hostIp = $hostInfoObj.Hosts.ip
        "$hostName <--> $hostIp"
    }

## 示例：获取默认存储

创建 HDInsight 群集时，必须使用 Azure 存储帐户作为群集的默认存储。创建群集后，可以使用 Ambari 来检索此信息。例如，如果要从 HDInsight 外部的容器读取数据或向其写入数据。

以下示例将从群集检索默认存储配置：

    curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/configurations/service_config_versions?service_name=HDFS&service_config_version=1" \
    | jq '.items[].configurations[].properties["fs.defaultFS"] | select(. != null)'

<br/>  

    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/configurations/service_config_versions?service_name=HDFS&service_config_version=1" `
        -Credential $creds
    $respObj = ConvertFrom-Json $resp.Content
    $respObj.items.configurations.properties.'fs.defaultFS'

> [AZURE.IMPORTANT]
这些示例会返回应用到服务器的第一个配置 (`service_config_version=1`)，其中包含此信息。如果要检索创建群集后修改的值，可能需要列出配置版本并检索最新版本。

返回值类似于以下其中一个示例：

* `wasbs://CONTAINER@ACCOUNTNAME.blob.core.chinacloudapi.cn` - 此值指示群集正在将 Azure 存储帐户用于默认存储。值 `ACCOUNTNAME` 是存储帐户的名称。`CONTAINER` 部分是存储帐户中 Blob 容器的名称。容器是群集的 HDFS 兼容存储的根。

> [AZURE.NOTE]
[Azure PowerShell](https://docs.microsoft.com/powershell/) 提供的 `Get-AzureRmHDInsightCluster` cmdlet 还返回群集的存储信息。

## <a name="example-update-ambari-configuration"></a>示例：更新 Ambari 配置

1. 获取当前配置，即 Ambari 存储为“所需配置”的配置：

        curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME?fields=Clusters/desired_configs"

    <br/>  

        Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName`?fields=Clusters/desired_configs" `
            -Credential $creds

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

        curl -u admin:$PASSWORD -sS -G "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/configurations?type=spark-thrift-sparkconf&tag=INITIAL" \
        | jq --arg newtag $(echo version$(date +%s%N)) '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json

    <br/>  

        $epoch = Get-Date -Year 1970 -Month 1 -Day 1 -Hour 0 -Minute 0 -Second 0
        $now = Get-Date
        $unixTimeStamp = [math]::truncate($now.ToUniversalTime().Subtract($epoch).TotalMilliSeconds)
        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/configurations?type=spark-thrift-sparkconf&tag=INITIAL" `
            -Credential $creds
        $resp.Content | jq --arg newtag "version$unixTimeStamp" '.items[] | del(.href, .version, .Config) | .tag |= $newtag | {"Clusters": {"desired_config": .}}' > newconfig.json

    Jq 用于将从 HDInsight 中检索的数据转换成新的配置模板。具体而言，这些示例会执行以下操作：
   
    * 创建一个包含字符串“version”和日期并存储在 `newtag` 中的唯一值。

    * 为新的所需配置创建根文档。

    * 获取 `.items[]` 数组的内容，并将其添加在 **desired\_config** 元素下。

    * 删除 `href`、`version` 和 `Config` 元素，因为提交新配置时不需要这些元素。

    * 添加值为 `version#################` 的新 `tag` 元素。数字部分基于当前日期。每个配置必须有唯一的标记。
     
    最后，数据将保存到 `newconfig.json` 文档。该文档结构类似于下面的示例：

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

3. 打开 `newconfig.json` 文档并在 `properties` 对象中修改/添加值。以下示例将 `"spark.yarn.am.memory"` 的值从 `"1g"` 更改为 `"3g"`。此外，还会添加一个值为 `"256m"` 的 `"spark.kryoserializer.buffer.max"`。
   
        "spark.yarn.am.memory": "3g",
        "spark.kyroserializer.buffer.max": "256m",
   
    完成修改后，保存该文件。

4. 使用以下命令将更新的配置提交到 Ambari。

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" -X PUT -d @newconfig.json "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME"

    <br/>  

        $newConfig = Get-Content .\newconfig.json
        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName" `
            -Credential $creds `
            -Method PUT `
            -Headers @{"X-Requested-By" = "ambari"} `
            -Body $newConfig
        $resp.Content

    这些命令会将 **newconfig.json** 文件的内容提交到群集作为所需的新配置。该请求会返回一个 JSON 文档。此文档中的 **versionTag** 元素应该与提交的版本相匹配，并且 **configs** 对象包含所请求的配置更改。

### 示例：重新启动服务组件

此时，如果查看 Ambari Web UI，会发现 Spark 服务指出需要将它重新启动才能使新配置生效。使用以下步骤重新启动该服务。

1. 使用以下命令启用 Spark 服务的维护模式。

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        -X PUT -d '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}' \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/SPARK"

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/SPARK" `
            -Credential $creds `
            -Method PUT `
            -Headers @{"X-Requested-By" = "ambari"} `
            -Body '{"RequestInfo": {"context": "turning on maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"ON"}}}'
        $resp.Content

    这些命令将 JSON 文档发送到启用了维护模式的服务器。可以使用以下请求来验证服务当前是否处于维护模式：

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/SPARK" \
        | jq .ServiceInfo.maintenance_state

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/SPARK2" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.ServiceInfo.maintenance_state

    返回值为 `ON`。

2. 接下来，使用以下命令关闭服务：

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}' \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/SPARK"

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/SPARK" `
            -Credential $creds `
            -Method PUT `
            -Headers @{"X-Requested-By" = "ambari"} `
            -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"INSTALLED"}}}'
        $resp.Content

    其响应类似于如下示例：

        {
            "href" : "http://10.0.0.18:8080/api/v1/clusters/CLUSTERNAME/requests/29",
            "Requests" : {
                "id" : 29,
                "status" : "Accepted"
            }
        }

    > [AZURE.IMPORTANT]
    此 URI 返回的 `href` 值正在使用群集节点的内部 IP 地址。若要从群集外部使用该地址，请将“10.0.0.18:8080”部分替换为群集的 FQDN。
    
    以下命令检索请求的状态：

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/requests/29" \
        | jq .Requests.request_status

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/requests/29" `
            -Credential $creds
        $respObj = ConvertFrom-Json $resp.Content
        $respObj.Requests.request_status

    响应 `COMPLETED` 指示请求已完成。

3. 完成前一个请求后，请使用以下命令来启动服务。

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        -X PUT -d '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}' \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/SPARK"

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/SPARK" `
            -Credential $creds `
            -Method PUT `
            -Headers @{"X-Requested-By" = "ambari"} `
            -Body '{"RequestInfo":{"context":"_PARSE_.STOP.SPARK","operation_level":{"level":"SERVICE","cluster_name":"CLUSTERNAME","service_name":"SPARK"}},"Body":{"ServiceInfo":{"state":"STARTED"}}}'

    服务现在正在使用新配置。

4. 最后，使用以下命令关闭维护模式。

        curl -u admin:$PASSWORD -sS -H "X-Requested-By: ambari" \
        -X PUT -d '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}' \
        "https://$CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/$CLUSTERNAME/services/SPARK"

    <br/>  

        $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/SPARK" `
            -Credential $creds `
            -Method PUT `
            -Headers @{"X-Requested-By" = "ambari"} `
            -Body '{"RequestInfo": {"context": "turning off maintenance mode for SPARK"},"Body": {"ServiceInfo": {"maintenance_state":"OFF"}}}'

## 后续步骤

有关 REST API 的完整参考，请参阅 [Ambari API 参考 V1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)。

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: adding more details and adding powershell rest solution-->