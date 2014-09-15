<properties linkid="manage-services-hdinsight-use-Ambari" urlDisplayName="Monitor HDInsight clusters using the Ambari API" pageTitle="Monitor HDInsight clusters using the Ambari API | Azure" metaKeywords="" description="Use the Apache Ambari APIs for provisioning, managing, and monitoring Hadoop clusters. Ambari’s intuitive operator tools and APIs hide the complexity of Hadoop." services="hdinsight" documentationCenter="" title="Monitor HDInsight clusters using the Ambari API" umbracoNaviHide="0" disqusComments="1" authors="jgao" editor="cgronlun" manager="paulettm" />

# 使用 Ambari API 监视 HDInsight 群集

学习如何使用 Ambari API 监视 HDInsight 群集版本 2.1。

估计完成时间：15 分钟

## 本文内容

-   [什么是 Ambari？][]
-   [必备条件][]
-   [启动][]
-   [Ambari 监视 API][]
-   [后续步骤][]

## 什么是 Ambari？

[Apache Ambari][] 用于设置、管理和监视 Apache Hadoop 群集。它包括一系列直观的操作员工具和一组隐藏 Hadoop 复杂性的可靠 API，可简化群集操作。有关这些 API 的详细信息，请参阅 [Ambari API 参考][]。

HDInsight 目前只支持 Ambari 监视功能。HDInsight 群集版本 2.1 和 3.0 支持 Ambari API 1.0 版。本文只涉及在 HDInsight 群集版本 2.1 上运行 Ambari API。

## 必备条件

在开始阅读本教程前，你必须具有：

-   已安装并已配置 Azure PowerShell 的**工作站**。有关说明，请参阅[安装和配置 Azure PowerShell][]。若要执行 PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为“RemoteSigned”**。请参阅[运行 Windows PowerShell 脚本][]。

    [Curl][] 是可选的。可从[此处][]进行安装。

    > [WACOM.NOTE] 在 Windows 上使用 curl 命令时，请使用双引号（而不是单引号）括起选项值。

-   **一个 Azure HDInsight 群集**。有关群集设置的说明，请参阅[开始使用 HDInsight][] 或[设置 HDInsight 群集][]。你将需要以下数据才能完成本教程：

	<table border="1">
	<tr><th>群集属性</th><th>PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>你的 HDInsight 群集的名称。</td></tr>
	<tr><td>群集用户名</td><td>$clusterUsername</td><td></td><td>设置时指定的群集用户名。</td></tr>
	<tr><td>群集密码</td><td>$clusterPassword</td><td></td><td>群集用户密码。</td></tr>
	</table>

    > [WACOM.NOTE] 将值填入表。这将有助于学习本教程。

## 启动

使用 Ambari 来监视 HDInsight 群集有几种方法。

**使用 Azure PowerShell**

下面是用于获取 MapReduce 作业跟踪器信息的 PowerShell 脚本：

    $clusterName = "<HDInsightClusterName>"
    $clusterUsername = "<HDInsightClusterUsername>"
    $clusterPassword = "<HDInsightClusterPassword>"

    $ambariUri = "https://$clusterName.hdinsightservice.cn:443/ambari"
    $uriJobTracker = "$ambariUri/api/v1/clusters/$clusterName.hdinsightservice.cn/services/mapreduce/components/jobtracker"

    $passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
    $creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)

    $response = Invoke-RestMethod -Method Get -Uri $uriJobTracker -Credential $creds -OutVariable $OozieServerStatus 

    $response.metrics.'mapred.JobTracker'

输出为：

![作业跟踪器输出][]

**使用 curl**

下面是使用 Curl 获取群集信息的示例：

    curl -u <username>:<password> -k https://<ClusterName>.hdinsightservice.cn:443/ambari/api/v1/clusters/<ClusterName>.hdinsightservice.cn

输出为：

    {"href":"https://hdi0211v2.hdinsightservice.cn/ambari/api/v1/clusters/hdi0211v2.hdinsightservice.cn/",
    "Clusters":{"cluster_name":"hdi0211v2.hdinsightservice.cn","version":"2.1.3.0.432823"},
    "services"[
    {"href":"https://hdi0211v2.hdinsightservice.cn/ambari/api/v1/clusters/hdi0211v2.hdinsightservice.cn/services/hdfs",
    "ServiceInfo":{"cluster_name":"hdi0211v2.hdinsightservice.cn","service_name":"hdfs"}},
    {"href":"https://hdi0211v2.hdinsightservice.cn/ambari/api/v1/clusters/hdi0211v2.hdinsightservice.cn/services/mapreduce",
    "ServiceInfo":{"cluster_name":"hdi0211v2.hdinsightservice.cn","service_name":"mapreduce"}}],
    "hosts":[
    {"href":"https://hdi0211v2.hdinsightservice.cn/ambari/api/v1/clusters/hdi0211v2.hdinsightservice.cn/hosts/headnode0",
    "Hosts":{"cluster_name":"hdi0211v2.hdinsightservice.cn",
    "host_name":"headnode0"}},
    {"href":"https://hdi0211v2.hdinsightservice.cn/ambari/api/v1/clusters/hdi0211v2.hdinsightservice.cn/hosts/workernode0",
    "Hosts":{"cluster_name":"hdi0211v2.hdinsightservice.cn",
    "host_name":"workernode0"}}]}

## Ambari 监视 API

下表列出了一些最常用的 Ambari 监视 API 调用。有关该 API 的详细信息，请参阅 [Ambari API 参考][]。

<table border="1">
<tr><th>监视 API 调用</th><th>URI</th><th>说明</th></tr>
<tr><td>获取群集</td><td>/api/v1/clusters</td><td></td></tr>
<tr><td>获取群集信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn</td><td>群集、服务、主机</td></tr>
<tr><td>获取服务</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/services</td><td>服务包括：hdfs、mapreduce</td></tr>
<tr><td>获取服务信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/services/&lt;ServiceName&gt;</td><td></td></tr>
<tr><td>获取服务组件</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/services/&lt;ServiceName&gt;/components</td><td>HDFS：namenode, datanode<br/>MapReduce:jobtracker; tasktracker</td></tr>
<tr><td>获取组件信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/services/&lt;ServiceName&gt;/components/&lt;ComponentName&gt;</td><td>ServiceComponentInfo、主机组件、指标</td></tr>
<tr><td>获取主机</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/hosts</td><td>headnode0, workernode0</td></tr>
<tr><td>获取主机信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/hosts/&lt;HostName&gt; 
</td><td></td></tr>
<tr><td>获取主机组件</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/hosts/&lt;HostName&gt;/host_components
</td><td>namenode, resourcemanager</td></tr>
<tr><td>获取主机组件信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/hosts/&lt;HostName&gt;/host_components/&lt;ComponentName&gt;
</td><td>HostRoles、组件、主机、指标</td></tr>
<tr><td>获取配置</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/configurations 
</td><td>配置类型：core-site, hdfs-site, mapred-site, hive-site</td></tr>
<tr><td>获取配置信息。</td><td>/api/v1/clusters/&lt;ClusterName&gt;.hdinsightservice.cn/configurations?type=&lt;ConfigType&gt;&amp;tag=&lt;VersionName&gt; 
</td><td>配置类型：core-site, hdfs-site, mapred-site, hive-site</td></tr>
</table>

## 后续步骤

现在你已经学习了如何使用 Ambari 监视 API 调用。若要了解更多信息，请参阅以下文章：

-   [使用管理门户管理 HDInsight 群集][]
-   [使用 Azure PowerShell 管理 HDInsight 群集][]
-   [使用命令行接口管理 HDInsight 群集][]
-   [HDInsight 文档][]
-   [HDInsight 入门][开始使用 HDInsight]

  [什么是 Ambari？]: #whatisambari
  [必备条件]: #prerequisites
  [启动]: #jumpstart
  [Ambari 监视 API]: #monitor
  [后续步骤]: #nextsteps
  [Apache Ambari]: http://ambari.apache.org/
  [Ambari API 参考]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [运行 Windows PowerShell 脚本]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx
  [Curl]: http://curl.haxx.se
  [此处]: http://curl.haxx.se/download.html
  [开始使用 HDInsight]: ../hdinsight-get-started/
  [设置 HDInsight 群集]: ../hdinsight-provision-clusters/
  [作业跟踪器输出]: ./media/hdinsight-monitor-use-ambari-api/hdi.ambari.monitor.jobtracker.output.png
  [使用管理门户管理 HDInsight 群集]: ../hdinsight-administer-use-management-portal/
  [使用 Azure PowerShell 管理 HDInsight 群集]: ../hdinsight-administer-use-powershell/
  [使用命令行接口管理 HDInsight 群集]: ../hdinsight-administer-use-command-line/
  [HDInsight 文档]: /en-us/documentation/services/hdinsight/
