<properties title="HDInsight Release Notes" pageTitle="HDInsight 发行说明 | Azure" description="HDInsight release notes." metaKeywords="hdinsight, hadoop, hdinsight hadoop, hadoop azure, release notes" services="HDInsight" solutions="" documentationCenter="" editor="cgronlun" manager="paulettm"  authors="bradsev" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="bradsev" />


# Microsoft HDInsight 发行说明

## 2014/10/7 版本发行说明 ##

* 使用 Ambari 终结点时，"https://{clusterDns}.azurehdinsight.cn/ambari/api/v1/clusters/{clusterDns}.azurehdinsight.cn/services/{servicename}/components/{componentname}"，现在 *host_name* 字段会返回节点的完全限定域名 (FQDN)，而不只是主机名。例如，你会看到 FQDN"**headnode0.{ClusterDNS}.azurehdinsight.cn**"，而不是返回"**headnode0**"。需要这种改变促进实现可以在一个虚拟网络 (VNET) 中部署多个群集类型（如 HBase 和 Hadoop）的方案。例如，使用 HBase 作为 Hadoop 的后端平台时，会发生这种情况。

* 我们已为 HDInsight 群集的默认部署提供新内存设置。以前的默认内存设置没有充分考虑部署 CPU 内核数指南。下表中逐项列出了默认 4 CPU 内核（8 容器）HDInsight 群集使用的新内存设置。（附带还提供了在本次发布之前使用的值）。 
 
<table border="1">
<tr><th>组件</th><th>内存分配</th></tr>
<tr><td> yarn.scheduler.minimum-allocation</td><td>768MB（以前为 512MB）</td></tr>
<tr><td> yarn.scheduler.maximum-allocation</td><td>6144MB（保持不变）</td></tr>
<tr><td>yarn.nodemanager.resource.memory</td><td>6144MB（保持不变）</td></tr>
<tr><td>mapreduce.map.memory</td><td>768MB（以前为 512MB）</td></tr>
<tr><td>mapreduce.map.java.opts</td><td>opts=-Xmx512m（以前为 -Xmx410m）</td></tr>
<tr><td>mapreduce.reduce.memory</td><td>1536MB（以前为 1024MB）</td></tr>
<tr><td>mapreduce.reduce.java.opts</td><td>opts=-Xmx1024m（以前为 -Xmx819m）</td></tr>
<tr><td>yarn.app.mapreduce.am.resource</td><td>768MB（以前为 1024MB）</td></tr>
<tr><td>yarn.app.mapreduce.am.command</td><td>opts=-Xmx512m（以前为 -Xmx819m）</td></tr>
<tr><td>mapreduce.task.io.sort</td><td>256MB（以前为 200MB）</td></tr>
<tr><td>tez.am.resource.memory</td><td>1536MB（保持不变）</td></tr>

</table><br>

有关 HDInsight 的 Hortonworks 数据平台上 YARN 和 MapReduce 使用的内存配置设置的更多信息，请参阅[确定 HDP 内存配置设置](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1-latest/bk_installing_manually_book/content/rpm-chap1-11.html)。Hortonworks 还提供一款工具，用于计算合适的内存设置。


## HDinsight 3.1## 的 2014/9/12 版本发行说明

* 本版本基于 Hortonworks 数据平台 (HDP) 2.1.5。有关本版本中修复的 Bug 列表，请参阅 Hortonworks 网站的[本版本中修复的 Bug](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.5/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.5-fixed.html) 页面。
* 在 Pig 库文件夹中，文件"avro-mapred-1.7.4.jar"已改为 avro-mapred-1.7.4-hadoop2.jar。这些文件的内容包含一个小 Bug 的不间断修复。建议客户不要直接依赖 JAR 文件名称本身，以避免文件重命名时出现中断。


## 2014/8/21 版本发行说明 ##

* 我们正在添加以下新的 WebHCat 配置 (HIVE-7155)，该配置可将 Templeton 控制器作业的默认内存限制设置为 1GB（以前的默认值为 512MB）：
	
	* templeton.mapper.memory.mb (=1024)
	* 这项更改解决了某些 Hive 查询由于内存限制较低而遇到的以下错误："容器即将超出物理内存限制"。
	* 若要恢复到旧默认值，你可以在创建群集时使用以下命令通过 PowerShell SDK 将此配置值设置为 512：
	
		Add-AzureHDInsightConfigValues -Core @{"templeton.mapper.memory.mb"="512";}


* zookeeper 角色的主机名已更改为 zookeeper。这会影响群集内部的名称解析，但不会影响外部 REST API。如果你的组件使用了 zookeepernode 主机名，则需更新这些组件，使其改用新名称。三个 zookeeper 节点的新名称为： 
	* zookeeper0 
	* zookeeper1 
	* zookeeper2 
* 已更新 HBase 版本支持矩阵。生产 HBase 工作负载仅支持版本 HDInsight 3.1（HBase 版本 0.98）。用于预览的版本 3.0 将不支持升级。在过渡期，客户仍可以创建 3.0 版本的群集。 

## 2014/8/15 ## 之前创建的群集的注意事项

由于 SDK/PowerShell 和群集之间的版本不同，可能遇到 HDInsight PowerShell/SDK 错误，附带消息"群集 <clustername> 没有配置 HTTP 服务访问权限"（或者根据操作，遇到其他错误消息，如："无法连接群集"）。8 月 15 日或之后创建的群集支持虚拟网络的新配置功能。旧版本 SDK/PowerShell 无法正确解释此功能，导致提交作业操作失败。如果使用 SDK API 或 PowerShell cmdlet 提交作业（如 Use-AzureHDInsightCluster 或 Invoke-AzureHDInsightHiveJob），则那些操作可能失败，并附加一条上述错误消息。

在最新版 SDK 和 Azure PowerShell 中，这些兼容性问题均已解决。我们建议将 HDInsight SDK 更新至 1.3.1.6 版本或更高版本，将 Azure PowerShell 工具更新至 0.8.8 版本或更高版本。你可以从 [nuget](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.HDInsight/) 访问最新版 HDInsight SDK，还可以使用 [Microsoft Web PI](http://go.microsoft.com/?linkid=9811175&clcid=0x409) 访问 Azure PowerShell 工具。

你可以期待，只要群集版本保持不变，SDK 和 PowerShell 就将可以继续与群集新更新配合使用。例如，群集版本 3.1 将始终与 SDK/PowerShell 当前版本 1.3.1.6 和 0.8.8 兼容。


## 2014/7/28 版本发行说明 ##

* **HDInsight 已在新区域推出**：随着此版本的发行，我们已将 HDInsight 的地理覆盖范围扩大到了三个新区域。现在，HDInsight 客户可以在这些区域创建群集。 
	* 亚洲东部 
	* 美国中北部 
	* 美国中南部。 
* 随着此版本的发行，HDInsight v1.6（HDP1.1、Hadoop 1.0.3）和 HDInsight v2.1（HDP1.3、Hadoop 1.2）即将从 Azure 管理门户中删除。你可以继续使用 HDInsight PowerShell cmdlet ([New-AzureHDInsightCluster](http://msdn.microsoft.com/zh-cn/library/dn593744.aspx)) 或 [HDInsight SDK](http://msdn.microsoft.com/zh-cn/library/azure/dn469975.aspx) 为这些版本创建 Hadoop 群集。有关详细信息，请参阅 [HDInsight 组件版本](http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-component-versioning/)页。
* 此版本中发生的 Hortonworks 数据平台 (HDP) 更改： 

<table border="1">
<tr><th>HDP</th><th>更改</th></tr>
<tr><td>HDP 2.0 / HDI 3.0</td><td>无更改</td></tr>
<tr><td>HDP 2.1 / HDI 3.1</td><td>zookeeper： ['3.4.5.2.1.3.0-1948'] -> ['3.4.5.2.1.3.2-0002']</td></tr>


</table><br>

## 2014/6/24 版本发行说明 ##

此版本为 HDInsight 服务提供了多项新的增强功能： 

* **HDP 2.1 可用性**：包含 HDP 2.1 的 HDInsight 3.1 现已正式发布，并成为新群集的默认版本。
* **HBase - Azure 管理门户改进**：我们将在预览版中提供 HBase 群集。现在，你只需单击三下鼠标，就能从门户创建 HBase 群集。

![](http://i.imgur.com/cmOl5fM.png)

借助 HBase，你可以在 HDInsight 上生成各种实时工作负载 - 从用于处理大型数据集的交互式网站，到用于存储来自数百万个终结点的传感器数据与遥测数据的服务。你接下来要做的就是使用 Hadoop 作业分析这些工作负载中的数据，通过 PowerShell 和 Hive 群集仪表板等工具提供的体验，你随时都可以在 HDInsight 中完成这种分析。

### Apache Mahout 现已预装在 HDInsight 3.1 ### 上

 [Mahout](http://hortonworks.com/hadoop/mahout/) 已预装在 HDInsight 3.1 Hadoop 群集上。因此，你无需进行其他任何群集配置，就能运行 Mahout 作业。例如，你可以使用远程桌面协议 (RDP) 远程访问 Hadoop 群集，并且无需执行附加的步骤，就能执行 Hello world Mahout 命令：

		mahout org.apache.mahout.classifier.df.tools.Describe -p /user/hdp/glass.data -f /user/hdp/glass.info -d I 9 N L  

		mahout org.apache.mahout.classifier.df.BreimanExample -d /user/hdp/glass.data -ds /user/hdp/glass.info -i 10 -t 100

有关此过程的更完整说明，请参阅 Apache Mahout 网站上的 [Breiman 示例](https://mahout.apache.org/users/classification/breiman-example.html)文档。 


### Hive 查询可以在 HDinsight 3.1 ### 中使用 Tez

Hive 0.13 现已在 HDInsight 3.1 中提供，并且能够使用 Tez 运行查询，这带来了极大的性能改善。 
默认情况下，没有为 Hive 查询启用 Tez。若要使用 Tez，你必须选择启用它。可以通过运行以下代码段来启用 Tez：

		set hive.execution.engine=tez;
		select sc_status, count(*), histogram_numeric(sc_bytes,5) from website_logs_orc_local group by sc_status;

Hortonworks 发布了使用以标准基准版提供的 Tez 后，Hive 查询性能得到增强的每条明细。有关详细信息，请参阅[适用于 Enterprise Hadoop 的 Apache Hive 13 基准](http://hortonworks.com/blog/benchmarking-apache-hive-13-enterprise-hadoop/)。 

有关将 Hive 与 Tez 结合使用的更多详细信息，请参阅 ["Tez 上的 Hive"Wiki 页](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)。

### 全球推出
随着 Azure HDInsight on Hadoop 2.2 的发行，Microsoft 已在所有主要 Azure 地理覆盖区域推出了 HDInsight。具体来说，欧洲西部和东南亚数据中心已联机。这使客户能够在距离近且可能位于具有类似合规要求的区域的数据中心内找到群集。 

### 重大变化
**前缀语法**：
HDInsight 3.0 和 3.1 群集仅支持"wasb://"语法。较早的"asv://"语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 或更高版本的群集中不受支持。这意味着提交到 HDInsight 3.0 或 3.1 群集的任何显式使用"asv://"语法的作业都将会失败。应改用 wasb:// 语法。而且，提交到任何 HDInsight 3.0 或 3.1 群集的作业，如果是使用现有元存储创建的，而该元存储包含对使用 asv:// 语法的资源的显式引用，则这些作业也会失败。这些元存储将需要使用 wasb:// 重新创建以确定资源地址。 


**端口**：HDInsight 服务使用的端口已更改。以前所用的端口号在 Windows OS 临时端口范围内。端口是从预定义的临时范围自动分配的，该范围适用于基于 Internet 协议的短期通信。新的一组允许的 Hortonworks 数据平台 (HDP) 服务端口号现已在此范围外，目的是避免遇到头节点上运行的服务所使用的端口时出现冲突。新端口号不会导致任何重大更改。现在使用的端口号如下所示：

<!--
 **HDInsight 1.6 (HDP 1.1)**
<table border="1">
<tr><th>Name</th><th>Value</th></tr>
<tr><td>dfs.http.address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.secondary.http.address</td><td>0.0.0.0:30090</td></tr>
<tr><td>mapred.job.tracker.http.address</td><td>jobtrackerhost:30030</td></tr>
<tr><td>mapred.task.tracker.http.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>mapreduce.history.server.http.address</td><td>0.0.0.0:31111</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table><br>
-->

 **HDInsight 3.0 and 3.1 (HDP 2.0 and 2.1)**
<table border="1">
<tr><th>Name</th><th>Value</th></tr>
<tr><td>dfs.namenode.http-address</td><td>namenodehost:30070</td></tr>
<tr><td>dfs.namenode.https-address</td><td>headnodehost:30470</td></tr>
<tr><td>dfs.datanode.address</td><td>0.0.0.0:30010</td></tr>
<tr><td>dfs.datanode.http.address</td><td>0.0.0.0:30075</td></tr>
<tr><td>dfs.datanode.ipc.address</td><td>0.0.0.0:30020</td></tr>
<tr><td>dfs.namenode.secondary.http-address</td><td>0.0.0.0:30090</td></tr>
<tr><td>yarn.nodemanager.webapp.address</td><td>0.0.0.0:30060</td></tr>
<tr><td>templeton.port</td><td>30111</td></tr>
</table><br>

### Dependencies 

The following dependencies were add in HDInsight 3.x (HDP2.x):

* guice-servlet
* optiq-core
* javax.inject
* activation
* jsr305
* geronimo-jaspic_1.0_spec
* jul-to-slf4j
* java-xmlbuilder
* ant
* commons-compiler
* jdo-api
* commons-math3
* paranamer
* jaxb-impl
* stringtemplate
* eigenbase-xom
* jersey-servlet
* commons-exec
* jaxb-api
* jetty-all-server
* janino
* xercesImpl
* optiq-avatica
* jta
* eigenbase-properties
* groovy-all
* hamcrest-core
* mail
* linq4j
* jpam
* jersey-client
* aopalliance
* geronimo-annotation_1.0_spec
* ant-launcher
* jersey-guice
* xml-apis
* stax-api
* asm-commons
* asm-tree
* wadl
* geronimo-jta_1.1_spec
* guice
* leveldbjni-all
* velocity
* jettison
* snappy-java
* jetty-all
* commons-dbcp

The following dependencies no longer exist in HDInsight 3.x (HDP2.x):

* jdeb
* kfs
* sqlline
* ivy
* aspectjrt
* json
* core
* jdo2-api
* avro-mapred
* datanucleus-enhancer
* jsp
* commons-logging-api
* commons-math
* JavaEWAH
* aspectjtools
* javolution
* hdfsproxy
* hbase
* snappy

### Version changes 

The following version changes were made between HDInsight 2.x (HDP1.x) and HDInsight 3.x (HDP2.x):

* metrics-core: ['2.1.2'] -> ['3.0.0']
* derbynet: ['10.4.2.0'] -> ['10.10.1.1']
* datanucleus: ['rdbms-3.0.8'] -> ['rdbms-3.2.9']
* jasper-compiler: ['5.5.12'] -> ['5.5.23']
* log4j: ['1.2.15', '1.2.16'] -> ['1.2.16', '1.2.17']
* derbyclient: ['10.4.2.0'] -> ['10.10.1.1']
* httpcore: ['4.2.4'] -> ['4.2.5']
* hsqldb: ['1.8.0.10'] -> ['2.0.0']
* jets3t: ['0.6.1'] -> ['0.9.0']
* protobuf-java: ['2.4.1'] -> ['2.5.0']
* derby: ['10.4.2.0'] -> ['10.10.1.1']
* jasper: ['runtime-5.5.12'] -> ['runtime-5.5.23']
* commons-daemon: ['1.0.1'] -> ['1.0.13']
* datanucleus-core: ['3.0.9'] -> ['3.2.10']
* datanucleus-api-jdo: ['3.0.7'] -> ['3.2.6']
* zookeeper: ['3.4.5.1.3.9.0-01320'] -> ['3.4.5.2.1.3.0-1948']
* bonecp: ['0.7.1.RELEASE'] -> ['0.8.0.RELEASE']


### Drivers
The SQL Server JDBC Driver is used internally by HDInsight and is not used for external operations. If you wish to connect to HDInsight using ODBC, please use the Microsoft Hive ODBC driver. For more information on using Hive ODBC, [Connect Excel to HDInsight with the Microsoft Hive ODBC Driver][connect-excel-with-hive-ODBC].


### Bug fixes ###

With this release, we have refreshed the following HDInsight  (Hortonworks Data Platform - HDP) versions with several bug fixes:

* HDInsight 3.0 (HDP 2.0)
* HDInsight 3.1 (HDP 2.1)

## Hortonworks Release Notes ##

Release notes for the HDPs that are used by the versions of HDInsight cluster are available at the following locations.

* HDInsight cluster version 3.1 uses an Hadoop distribution that is based on the [Hortonworks Data Platform 2.1][hdp-2-1-1].(This is the default Hadoop cluster created when using the Azure HDInsight portal.)

* HDInsight cluster version 3.0 uses an Hadoop distribution that is based on the [Hortonworks Data Platform 2.0][hdp-2-0-8].






[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html





