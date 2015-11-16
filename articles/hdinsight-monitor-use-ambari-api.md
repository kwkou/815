<properties 
	pageTitle="使用 Ambari API 在 HDInsight 中监视 Hadoop 群集 | Azure" 
	description="使用 Apache Ambari API 设置、管理和监视 Hadoop 群集。直观的操作员工具和 API 消除了 Hadoop 的复杂性。"
	services="hdinsight" 
	documentationCenter="" 
	tags="azure-portal"
	authors="mumian" 
	editor="cgronlun" 
	manager="paulettm"/>

<tags 
	ms.service="hdinsight" 
	ms.date="07/28/2015"
	wacn.date="10/03/2015"/>
 
# 使用 Ambari API 在 HDInsight 中监视 Hadoop 群集
 
了解如何通过使用 Ambari API 监视 HDInsight 群集版本 3.1 和 2.1。

## <a id="whatisambari"></a>什么是 Ambari？

[Apache Ambari][ambari-home] 用于预配、管理和监视 Apache Hadoop 群集。它包括一系列直观的操作员工具和一组免除 Hadoop 复杂性的可靠 API，可简化群集操作。有关这些 API 的详细信息，请参阅 [Ambari API 参考][ambari-api-reference]。


HDInsight 目前仅支持 Ambari 监视功能。Ambari API 1.0 受 HDInsight 版本 3.0 和 2.1 群集支持。本文介绍如何访问 HDInsight 版本 3.1 和 2.1 群集上的 Ambari API。两者之间的主要区别在于，随着新功能（例如作业历史记录服务器）的引入，某些组件已发生更改。


##<a id="prerequisites"></a>先决条件

在开始阅读本教程前，你必须具有：

- **配备 Azure PowerShell 的工作站**。请参阅[安装和配置 Azure PowerShell][powershell-install]。若要执行 Azure PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为 *RemoteSigned*。有关详细信息，请参阅[运行 Windows PowerShell 脚本][powershell-script]。

- （可选）[cURL][curl]。若要安装它，请参阅 [cURL 版本和下载][curl-download]。

	>[AZURE.NOTE]在 Windows 中使用 cURL 命令时，对选项值使用双引号而非单引号。

- **一个 Azure HDInsight 群集**。有关群集预配的说明，请参阅[开始使用 HDInsight][hdinsight-get-started] 或[预配 HDInsight 群集][hdinsight-provision]。你将需要以下数据才能完成本教程：

	<table border="1">
<tr><th>群集属性</th><th>Azure PowerShell 变量名</th><th>值</th><th>说明</th></tr>
<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>你的 HDInsight 群集的名称。</td></tr>
<tr><td>群集用户名</td><td>$clusterUsername</td><td></td><td>在设置时指定的群集用户名。</td></tr>
<tr><td>群集密码</td><td>$clusterPassword</td><td></td><td>群集用户密码。</td></tr>
</table>> [AZURE.NOTE]在表中填充值。这将有助于完成本教程。



##<a id="jumpstart"></a>快速启动

使用 Ambari 来监视 HDInsight 群集有几种方法。

**使用 Azure PowerShell**

下面是用于*在 HDInsight 3.1 群集*中获取 MapReduce 作业跟踪器信息的 Azure PowerShell 脚本。 主要区别在于，我们从 YARN 服务（而非 MapReduce）中拉取这些详细信息。

	$clusterName = "<HDInsightClusterName>"
	$clusterUsername = "<HDInsightClusterUsername>"
	$clusterPassword = "<HDInsightClusterPassword>"
	
	$ambariUri = "https://$clusterName.azurehdinsight.cn:443/ambari"
	$uriJobTracker = "$ambariUri/api/v1/clusters/$clusterName.azurehdinsight.cn/services/yarn/components/resourcemanager"
	
	$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
	$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)
	
	$response = Invoke-RestMethod -Method Get -Uri $uriJobTracker -Credential $creds -OutVariable $OozieServerStatus 
	
	$response.metrics.'yarn.queueMetrics'

下面是用于*在 HDInsight 2.1 群集*中获取 MapReduce 作业跟踪器信息的 Azure PowerShell 脚本：

	$clusterName = "<HDInsightClusterName>"
	$clusterUsername = "<HDInsightClusterUsername>"
	$clusterPassword = "<HDInsightClusterPassword>"
	
	$ambariUri = "https://$clusterName.azurehdinsight.cn:443/ambari"
	$uriJobTracker = "$ambariUri/api/v1/clusters/$clusterName.azurehdinsight.cn/services/mapreduce/components/jobtracker"
	
	$passwd = ConvertTo-SecureString $clusterPassword -AsPlainText -Force
	$creds = New-Object System.Management.Automation.PSCredential ($clusterUsername, $passwd)
	
	$response = Invoke-RestMethod -Method Get -Uri $uriJobTracker -Credential $creds -OutVariable $OozieServerStatus 
	
	$response.metrics.'mapred.JobTracker'

输出为：

![作业跟踪器输出][img-jobtracker-output]

**使用 cURL**

下面是通过使用 cURL 获取群集信息的示例：

	curl -u <username>:<password> -k https://<ClusterName>.azurehdinsight.cn:443/ambari/api/v1/clusters/<ClusterName>.azurehdinsight.cn

输出为：
	
	{"href":"https://hdi0211v2.azurehdinsight.cn/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.cn/",
	 "Clusters":{"cluster_name":"hdi0211v2.azurehdinsight.cn","version":"2.1.3.0.432823"},
	 "services"[
	   {"href":"https://hdi0211v2.azurehdinsight.cn/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.cn/services/hdfs",
	    "ServiceInfo":{"cluster_name":"hdi0211v2.azurehdinsight.cn","service_name":"hdfs"}},
	   {"href":"https://hdi0211v2.azurehdinsight.cn/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.cn/services/mapreduce",
	    "ServiceInfo":{"cluster_name":"hdi0211v2.azurehdinsight.cn","service_name":"mapreduce"}}],
	 "hosts":[
	   {"href":"https://hdi0211v2.azurehdinsight.cn/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.cn/hosts/headnode0",
	    "Hosts":{"cluster_name":"hdi0211v2.azurehdinsight.cn",
	             "host_name":"headnode0"}},
	   {"href":"https://hdi0211v2.azurehdinsight.cn/ambari/api/v1/clusters/hdi0211v2.azurehdinsight.cn/hosts/workernode0",
	    "Hosts":{"cluster_name":"hdi0211v2.azurehdinsight.cn",
	             "host_name":"headnode0.{ClusterDNS}.azurehdinsight.cn"}}]}

**对于 10/8/2014 release 版本**：使用 Ambari 终结点“https://{clusterDns}.azurehdinsight.cn/ambari/api/v1/clusters/{clusterDns}.azurehdinsight.cn/services/{servicename}/components/{componentname}”时，*host_name* 字段会返回节点的完全限定域名 (FQDN)，而不是主机名。在 10/8/2014 版本之前，此示例仅返回 **headnode0**"。在 10/8/2014 版本之后，你将获取 FQDN "**headnode0.{ClusterDNS}.azurehdinsight.cn**"，如以上示例所示。需要这种改变促进实现可以在一个虚拟网络 (VNET) 中部署多个群集类型（如 HBase 和 Hadoop）的方案。例如，使用 HBase 作为 Hadoop 的后端平台时，会发生这种情况。

##<a id="monitor"></a>Ambari 监视 API

下表列出了一些最常用的 Ambari 监视 API 调用。有关该 API 的详细信息，请参阅 [Ambari API 参考][ambari-api-reference]。

<table border="1">
<tr><th>监视 API 调用</th><th>URI</th><th>说明</th></tr>
<tr><td>获取群集</td><td><tt>/api/v1/clusters</tt></td><td></td></tr>
<tr><td>获取群集信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn</tt></td><td>群集、服务、主机</td></tr>
<tr><td>获取服务</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/services</tt></td><td>服务包括：hdfs、mapreduce</td></tr>
<tr><td>获取服务信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/services/&lt;ServiceName></tt></td><td></td></tr>
<tr><td>获取服务组件</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/services/&lt;ServiceName>/components</tt></td><td>HDFS：namenode、datanode<br/>MapReduce：jobtracker；tasktracker</td></tr>
<tr><td>获取组件信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/services/&lt;ServiceName>/components/&lt;ComponentName></tt></td><td>ServiceComponentInfo、主机组件、指标</td></tr>
<tr><td>获取主机</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/hosts</tt></td><td>headnode0、workernode0</td></tr>
<tr><td>获取主机信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/hosts/&lt;HostName> 
</td><td></td></tr>
<tr><td>获取主机组件</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/hosts/&lt;HostName>/host_components </tt></td><td>namenode、resourcemanager</td></tr>
<tr><td>获取主机组件信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/hosts/&lt;HostName>/host_components/&lt;ComponentName> </tt></td><td>HostRoles、组件、主机、指标</td></tr>
<tr><td>获取配置</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/configurations </tt></td><td>配置类型：core-site、hdfs-site、mapred-site、hive-site</td></tr>
<tr><td>获取配置信息</td><td><tt>/api/v1/clusters/&lt;ClusterName>.azurehdinsight.cn/configurations?type=&lt;ConfigType>&amp;tag=&lt;VersionName> </tt></td><td>配置类型：core-site、hdfs-site、mapred-site、hive-site</td></tr>
</table>


##<a id="nextsteps"></a>后续步骤 

现在你已经学习了如何使用 Ambari 监视 API 调用。若要了解更多信息，请参阅以下文章：

- [使用管理门户管理 HDInsight 群集][hdinsight-admin-portal]
- [使用 Azure PowerShell 管理 HDInsight 群集][hdinsight-admin-powershell]
- [使用命令行界面管理 HDInsight 群集][hdinsight-admin-cli]
- [HDInsight 文档][hdinsight-documentation]
- [开始使用 HDInsight][hdinsight-get-started]



[ambari-home]: http://ambari.apache.org/
[ambari-api-reference]: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[microsoft-hadoop-SDK]: http://hadoopsdk.codeplex.com/wikipage?title=Ambari%20Monitoring%20Client

[Powershell-install]: /documentation/articles/install-configure-powershell
[Powershell-script]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx

[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1
[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line
[hdinsight-documentation]: /documentation/services/hdinsight/
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters
[img-jobtracker-output]: ./media/hdinsight-monitor-use-ambari-api/hdi.ambari.monitor.jobtracker.output.png

<!---HONumber=71-->