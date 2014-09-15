<properties title="HDInsight Release Notes" pageTitle="HDInsight Release Notes | Azure" description="HDInsight release notes." metaKeywords="hdinsight, hadoop, hdinsight hadoop, hadoop azure, release notes" services="HDInsight" solutions="" documentationCenter="" editor="cgronlun" manager="paulettm"  authors="bradsev" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="bradsev"></tags>

# Microsoft HDInsight 发行说明

## 8/21/2014 版本发行说明

- 我们正在添加以下新的 WebHCat 配置 (HIVE-7155)，该配置可将 Templeton 控制器作业的默认内存限制设置为 1GB（以前的默认值为 512MB）：

	- templeton.mapper.memory.mb (=1024)
	- 这项更改解决了某些 Hive 查询由于内存限制较低而遇到的以下错误：“容器即将超出物理内存限制”。
	- 若要恢复到旧默认值，你可以在创建群集时使用以下命令通过 PowerShell SDK 将此配置值设置为 512：

        Add-AzureHDInsightConfigValues -Core @{"templeton.mapper.memory.mb"="512";}

- zookeeper 角色的主机名已更改为 zookeeper。这会影响群集内部的名称解析，但不会影响外部 REST API。如果你的组件使用了 zookeepernode 主机名，则需更新这些组件，使其改用新名称。三个 zookeeper 节点的新名称为：

	-   zookeeper0
	-   zookeeper1
	-   zookeeper2

- 已更新 HBase 版本支持矩阵。生产 HBase 工作负载仅支持版本 HDInsight 3.1（HBase 版本 0.98）。用于预览的版本 3.0 将不支持升级。在过渡期，客户仍可以创建 3.0 版本的群集。

- 日志目录和文件名仅支持 ASCII 字符

  HBASE、OOZIE 和 Hive 的日志目录和文件名只能支持 ASCII 字符。

	* 只能使用 ASCII 字符配置 HBase 日志目录。

	The following property in hbase-site.xml under C:\apps\dist\hbase-0.96.0.2.0.9.0-1686-hadoop2\conf can only support ASCII characters for < Hbase Log folder >.

		< property >
		
		< name >hbase.log.dir< /name >
		
		< value >C:\apps\dist\hbase-0.96.0.2.0.9.0-1686-hadoop2\<Hbase Log folder>< /value >
		
		< /property > 
		 

     The “set HBASE_LOG_DIR” command in hbase.cmd, which overrides the logs folder, also supports only unicode characters.

	* Oozie log folder and file name can only support ASCII characters

	In the following property, *< Oozie Log folder >* and *< Oozie Log filename >* can only support ASCII characters.

		log4j.appender.oozie.File=${oozie.log.dir}/< Oozie Log folder >/< Oozie Log filenam >


	* Hive's log folder and file name can only support ASCII characters

	In the following conf\hive-log4j.properties, *< Hive Log folder >* and *< Hive Log filename >* can only support ASCII characters.

		log4j.appender.DRFA.File=${hive.log.dir}/*< Hive Log folder >*/*< Hive Log filename >*${hive.log.file}

    * Oozie action names can only support ASCII characters and match the regular expression pattern '([a-zA-Z_]([\-_a-zA-Z0-9])*) {1,39}'

	For instance, in the map reduce workflow – the action name can only support ASCII characters.

		< workflow-app xmlns="uri:oozie:workflow:0.2" name="map-reduce-wf" >

		< start to="< mr-actionname >"/ >

		< action name="< mr-actionname >">

	If a Unicode character is used in action name, you will see the following error

		Error: E0701 : E0701: XML schema error, cvc-pattern-valid: Value ‘< unicode action name >' is not facet-valid with respect to pattern ' ([a-zA-Z_\]([\\-_a-zA-Z0-9])*) {1,39} ' for type 'IDENTIFIER'.'.

- 有关使用 Templeton 的建议

  若要结合 Unicode 字符使用 Templeton，请使用百分比编码方案为 Unicode 字符编码 - <http://tools.ietf.org/html/rfc3986#page-12>。

  在下面这些例外情况下，即使使用了百分比编码方案，Unicode 字符也可能不会起到原有的作用：

	* Unicode character is part of an execute parameter

	Example command:

		curl -i -u \<hdinsightClusteruser>:\<HDInsightClusterPassword>

		-d user.name=\<hdinsightClusteruser>

		-d statusdir="/testfolder/HiveLikeOutput"

		-d execute="SELECT * FROM <table1> WHERE content LIKE '%步行%';"

		-s https://\<HdinsightClusterName>.hdinsightservices.cn/templeton/v1/hive

	After URL encoding: -d execute="SELECT * FROM table1 WHERE content LIKE '%%E6%AD%A5%E8%A1%8C%';" The ‘%’ symbol causes an issue. Templeton throws Error 500 !hex:5c

    **Workaround:**

    Save query in a script file and execute the script file. The workaround command will be:

        curl -i -u <hdinsightClusteruser>:<HDInsightClusterPassword>
        -d user.name=hdinsightuser
        -d statusdir="/testfolder/HiveLikeOutput"
        -d file=/testfolder/hivetest/HiveLike.hql
        -s https://<HdinsightClusterName>.hdinsightservices.cn/templeton/v1/hive

	* Name of mapper and reducer executable contains Unicode characters.

	Example Command:

		curl -i -u \<hdinsightClusteruser>:\<HDInsightClusterPassword>

		-d user.name=\<hdinsightClusteruser>

		-d input=/testfolder/MapReduce/wordcountStr/input

		-d output=/testfolder/MapReduce/wordcountOutputs/UniStrUniDataoutput

		-d mapper=cat%E4%B6%B4%E3%84%A9%E9%BC%BE%E4%B8%84%E7%8B%9C%E3%80%87cat.exe 

		-d reducer=wc%E4%B6%B4%E3%84%A9%E9%BC%BE%E4%B8%84%E7%8B%9C%E3%80%87wc.exe 

		-d files=/testfolder/MapReduce/wordcountStr/cat%E4%B6%B4%E3%84%A9%E9%BC%BE%E4%B8%84%E7%8B%9C%E3%80%87cat.exe,/testfolder/MapReduce/wordcountStr/wc%E4%B6%B4%E3%84%A9%E9%BC%BE%E4%B8%84%E7%8B%9C%E3%80%87wc.exe

		-d statusdir=/testfolder/MapReduce/MRFullUniStrUniDatastatus 

		-s http://headnodehost:30111/templeton/v1/mapreduce/streaming/


	This fails with the following exception:

		Caused by: java.io.IOException: Cannot run program "我的映射任务.exe": CreateProcess error=2, The system cannot find the file specified

	**Workaround:**

	Only use ASCII characters in executable names.

## 7/28/2014 版本发行说明

-   **HDInsight 已在新区域推出**：随着此版本的发行，我们已将 HDInsight 的地理覆盖范围扩大到了三个新区域。现在，HDInsight 客户可以在这些区域创建群集。
	- 亚洲东部
	- 美国中北部
	- 美国中南部
- 随着此版本的发行，HDInsight v1.6（HDP1.1、Hadoop 1.0.3）和 HDInsight v2.1（HDP1.3、Hadoop 1.2）即将从 Azure 管理门户中删除。你可以继续使用 HDInsight PowerShell cmdlet ([New-AzureHDInsightCluster](http://msdn.microsoft.com/en-us/library/dn593744.aspx)) 或 [HDInsight SDK](http://msdn.microsoft.com/en-us/library/azure/dn469975.aspx) 为这些版本创建 Hadoop 群集。有关详细信息，请参阅 [HDInsight 组件版本](http://azure.microsoft.com/en-us/documentation/articles/hdinsight-component-versioning/)页。
-   此版本中发生的 Hortonworks 数据平台 (HDP) 更改：

| HDP             | 更改                                                          |
|-----------------|---------------------------------------------------------------|
| HDP 1.3/HDI 2.1 | 无更改                                                        |
| HDP 2.0/HDI 3.0 | 无更改                                                        |
| HDP 2.1/HDI 3.1 | zookeeper： ['3.4.5.2.1.3.0-1948'] -\> ['3.4.5.2.1.3.2-0002'] |

## 6/24/2014 版本发行说明

此版本为 HDInsight 服务提供了多项新的增强：

-   **HDP 2.1 可用性**：包含 HDP 2.1 的 HDInsight 3.1 现已正式发布，并成为新群集的默认版本。
-   **HBase – Azure 管理门户改进**：我们将在预览版中提供 HBase 群集。现在，你只需单击三下鼠标，就能从门户创建 HBase 群集。

![](http://i.imgur.com/cmOl5fM.png)

借助 HBase，你可以在 HDInsight 上生成各种实时工作负载 - 从用于处理大型数据集的交互式网站，到用于存储来自数百万个终结点的传感器数据与遥测数据的服务。你接下来要做的就是使用 Hadoop 作业分析这些工作负载中的数据，通过 PowerShell 和 Hive 群集仪表板等工具提供的体验，你随时都可以在 HDInsight 中完成这种分析。

### Apache Mahout 现已预装在 HDInsight 3.1 上

[Mahout](http://hortonworks.com/hadoop/mahout/) 已预装在 HDInsight 3.1 Hadoop 群集上。因此，你无需进行其他任何群集配置，就能运行 Mahout 作业。例如，你可以使用远程桌面协议 (RDP) 远程访问 Hadoop 群集，并且无需执行附加的步骤，就能执行 Hello world Mahout 命令：

        mahout org.apache.mahout.classifier.df.tools.Describe -p /user/hdp/glass.data -f /user/hdp/glass.info -d I 9 N L  

        mahout org.apache.mahout.classifier.df.BreimanExample -d /user/hdp/glass.data -ds /user/hdp/glass.info -i 10 -t 100

有关此过程的更完整说明，请参阅 Apache Mahout 网站上的 [Breiman 示例](https://mahout.apache.org/users/classification/breiman-example.html)文档。

### Hive 查询可以在 HDinsight 3.1 中使用 Tez

Hive 0.13 现已在 HDInsight 3.1 中提供，并且能够使用 Tez 运行查询，这带来了极大的性能改善。
默认情况下，没有为 Hive 查询启用 Tez。若要使用 Tez，你必须选择启用它。可以通过运行以下代码段来启用 Tez：

        set hive.execution.engine=tez;
        select sc_status, count(*), histogram_numeric(sc_bytes,5) from website_logs_orc_local group by sc_status;

Hortonworks 发布了使用以标准基准版提供的 Tez 后，Hive 查询性能得到增强的每条明细。有关详细信息，请参阅[适用于 Enterprise Hadoop 的 Apache Hive 13 基准](http://hortonworks.com/blog/benchmarking-apache-hive-13-enterprise-hadoop/)。

有关将 Hive 与 Tez 结合使用的更多详细信息，请参阅[“Tez 上的 Hive”Wiki 页](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez)。

### 全球推出

随着 Azure HDInsight on Hadoop 2.2 的发行，Microsoft 已在所有主要 Azure 地理覆盖区域推出了 HDInsight。具体来说，欧洲西部和东南亚数据中心已联机。这使客户能够在距离近且可能位于具有类似合规要求的区域的数据中心内找到群集。

### 重大变化

**前缀语法**：
HDInsight 3.0 和 3.1 群集仅支持“wasb://”语法。较早的“asv://”语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 或更高版本的群集中不受支持。这意味着提交到 HDInsight 3.0 或 3.1 群集的任何显式使用“asv://”语法的作业都将会失败。应改用 wasb:// 语法。而且，提交到任何 HDInsight 3.0 或 3.1 群集的作业，如果是使用现有元存储创建的，而该元存储包含对使用 asv:// 语法的资源的显式引用，则这些作业也会失败。这些元存储将需要使用 wasb:// 重新创建以确定资源地址。

**端口**：HDInsight 服务使用的端口已更改。以前所用的端口号在 Windows OS 临时端口范围内。端口是从预定义的临时范围自动分配的，该范围适用于基于 Internet 协议的短期通信。新的一组允许的 Hortonworks 数据平台 (HDP) 服务端口号现已在此范围外，目的是避免遇到头节点上运行的服务所使用的端口时出现冲突。新端口号不会导致任何重大更改。现在使用的端口号如下所示：

**HDInsight 1.6 (HDP 1.1)**

<table border="1">

<tr>
<th>
名称

</th>
<th>
值

</th>
</tr>

<tr>
<td>
dfs.http.address

</td>
<td>
namenodehost:30070

</td>
</tr>

<tr>
<td>
dfs.datanode.address

</td>
<td>
0.0.0.0:30010

</td>
</tr>

<tr>
<td>
dfs.datanode.http.address

</td>
<td>
0.0.0.0:30075

</td>
</tr>

<tr>
<td>
dfs.datanode.ipc.address

</td>
<td>
0.0.0.0:30020

</td>
</tr>

<tr>
<td>
dfs.secondary.http.address

</td>
<td>
0.0.0.0:30090

</td>
</tr>

<tr>
<td>
mapred.job.tracker.http.address

</td>
<td>
jobtrackerhost:30030

</td>
</tr>

<tr>
<td>
mapred.task.tracker.http.address

</td>
<td>
0.0.0.0:30060

</td>
</tr>

<tr>
<td>
mapreduce.history.server.http.address

</td>
<td>
0.0.0.0:31111

</td>
</tr>

<tr>
<td>
templeton.port

</td>
<td>
30111

</td>
</tr>

</table>

</p>
**HDInsight 3.0 和 3.1（HDP 2.0 和 2.1）**

<table border="1">

<tr>
<th>
名称

</th>
<th>
值

</th>
</tr>

<tr>
<td>
dfs.namenode.http-address

</td>
<td>
namenodehost:30070

</td>
</tr>

<tr>
<td>
dfs.namenode.https-address

</td>
<td>
headnodehost:30470

</td>
</tr>

<tr>
<td>
dfs.datanode.address

</td>
<td>
0.0.0.0:30010

</td>
</tr>

<tr>
<td>
dfs.datanode.http.address

</td>
<td>
0.0.0.0:30075

</td>
</tr>

<tr>
<td>
dfs.datanode.ipc.address

</td>
<td>
0.0.0.0:30020

</td>
</tr>

<tr>
<td>
dfs.namenode.secondary.http-address

</td>
<td>
0.0.0.0:30090

</td>
</tr>

<tr>
<td>
yarn.nodemanager.webapp.address

</td>
<td>
0.0.0.0:30060

</td>
</tr>

<tr>
<td>
templeton.port

</td>
<td>
30111

</td>
</tr>

</table>

</p>
### 依赖项

在 HDInsight 3.x (HDP2.x) 中添加了以下依赖项：

-   guice-servlet
-   optiq-core
-   javax.inject
-   activation
-   jsr305
-   geronimo-jaspic\_1.0\_spec
-   jul-to-slf4j
-   java-xmlbuilder
-   ant
-   commons-compiler
-   jdo-api
-   commons-math3
-   paranamer
-   jaxb-impl
-   stringtemplate
-   eigenbase-xom
-   jersey-servlet
-   commons-exec
-   jaxb-api
-   jetty-all-server
-   janino
-   xercesImpl
-   optiq-avatica
-   jta
-   eigenbase-properties
-   groovy-all
-   hamcrest-core
-   mail
-   linq4j
-   jpam
-   jersey-client
-   aopalliance
-   geronimo-annotation\_1.0\_spec
-   ant-launcher
-   jersey-guice
-   xml-apis
-   stax-api
-   asm-commons
-   asm-tree
-   wadl
-   geronimo-jta\_1.1\_spec
-   guice
-   leveldbjni-all
-   velocity
-   jettison
-   snappy-java
-   jetty-all
-   commons-dbcp

HDInsight 3.x (HDP2.x) 中不再存在以下依赖项：

-   jdeb
-   kfs
-   sqlline
-   ivy
-   aspectjrt
-   json
-   core
-   jdo2-api
-   avro-mapred
-   datanucleus-enhancer
-   jsp
-   commons-logging-api
-   commons-math
-   JavaEWAH
-   aspectjtools
-   javolution
-   hdfsproxy
-   hbase
-   snappy

### 版本更改

在 HDInsight 2.x (HDP1.x) 与 HDInsight 3.x (HDP2.x) 之间发生了以下版本更改：

-   metrics-core： ['2.1.2'] -\> ['3.0.0']
-   derbynet： ['10.4.2.0'] -\> ['10.10.1.1']
-   datanucleus：['rdbms-3.0.8'] -\> ['rdbms-3.2.9']
-   jasper-compiler： ['5.5.12'] -\> ['5.5.23']
-   log4j： ['1.2.15', '1.2.16'] -\> ['1.2.16', '1.2.17']
-   derbyclient： ['10.4.2.0'] -\> ['10.10.1.1']
-   httpcore： ['4.2.4'] -\> ['4.2.5']
-   hsqldb： ['1.8.0.10'] -\> ['2.0.0']
-   jets3t： ['0.6.1'] -\> ['0.9.0']
-   protobuf-java： ['2.4.1'] -\> ['2.5.0']
-   derby： ['10.4.2.0'] -\> ['10.10.1.1']
-   jasper：['runtime-5.5.12'] -\> ['runtime-5.5.23']
-   commons-daemon： ['1.0.1'] -\> ['1.0.13']
-   datanucleus-core： ['3.0.9'] -\> ['3.2.10']
-   datanucleus-api-jdo： ['3.0.7'] -\> ['3.2.6']
-   zookeeper： ['3.4.5.1.3.9.0-01320'] -\> ['3.4.5.2.1.3.0-1948']
-   bonecp：['0.7.1.RELEASE'] -\> ['0.8.0.RELEASE']

### 驱动程序

SQL Server JDBC 驱动程序由 HDInsight 在内部使用，不用于外部操作。如果你希望使用 ODBC 连接到 HDInsight，请使用 Microsoft Hive ODBC 驱动程序。有关使用 Hive ODBC 的详细信息，请参阅 [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][connect-excel-with-hive-ODBC]。

### Bug 修复

随着此版本的发行，我们已完成了多项 Bug 修复，并刷新了以下 HDInsight（Hortonworks 数据平台 - HDP）版本：

-   HDInsight 2.1 (HDP 1.3)
-   HDInsight 3.0 (HDP 2.0)
-   HDInsight 3.1 (HDP 2.1)

## Hortonworks 发行说明

以下位置提供了 HDInsight 群集版本使用的 HDP 的发行说明。

-   HDInsight 群集版本 3.1 使用基于 [Hortonworks 数据平台 2.1][hdp-2-1-1] 的 Hadoop 分发版。（这是使用 Azure HDInsight 门户时创建的默认 Hadoop 群集。）

-   HDInsight 群集版本 3.0 使用基于 [Hortonworks 数据平台 2.0][hdp-2-0-8] 的 Hadoop 分发。

-   HDInsight 群集版本 2.1 使用基于 [Hortonworks 数据平台 1.3][hdp-1-3-0] 的 Hadoop 分发。

-   HDInsight 群集版本 1.6 使用基于 [Hortonworks 数据平台 1.1][hdp-1-1-0] 的 Hadoop 分发。

[hdp-2-1-1]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.1/bk_releasenotes_hdp_2.1/content/ch_relnotes-hdp-2.1.1.html

[hdp-2-0-8]: http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.8.0/bk_releasenotes_hdp_2.0/content/ch_relnotes-hdp2.0.8.0.html

[hdp-1-3-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-1.3.0/bk_releasenotes_hdp_1.x/content/ch_relnotes-hdp1.3.0_1.html

[hdp-1-1-0]: http://docs.hortonworks.com/HDPDocuments/HDP1/HDP-Win-1.1/bk_releasenotes_HDP-Win/content/ch_relnotes-hdp-win-1.1.0_1.html