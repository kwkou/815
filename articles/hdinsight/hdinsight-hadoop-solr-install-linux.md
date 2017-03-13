<properties
    pageTitle="使用脚本操作在基于 Linux 的 HDInsight 上安装 Solr | Azure"
    description="了解如何使用脚本操作在基于 Linux 的 HDInsight Hadoop 群集上安装 Solr。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="cc93ed5c-a358-456a-91a4-f179185c0e98"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/17/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />

# 在 HDInsight Hadoop 群集上安装并使用 Solr
本主题介绍如何使用脚本操作在 Azure HDInsight 上安装 Solr。Solr 是一种功能强大的搜索平台，提供了企业级搜索功能，用于搜索由 Hadoop 管理的数据。在 HDInsight 群集上安装了 Solr 后，你还将学习如何通过使用 Solr 搜索数据。

> [AZURE.IMPORTANT]
本文档中的步骤需要使用 Linux 的 HDInsight 群集。Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

本主题中使用的示例脚本将使用特定配置创建 Solr 群集。如果你要使用不同集合、分片、架构、副本等配置 Solr 群集，则必须相应地修改脚本和 Solr 二进制文件。

## <a name="whatis"></a>什么是 Solr？
[Apache Solr](http://lucene.apache.org/solr/features.html) 是一种企业搜索平台，用于对数据实现功能强大的全文搜索。虽然 Hadoop 可用于存储和管理大量数据，但是，Apache Solr 提供了快速检索数据的搜索功能。本主题提供有关如何自定义 HDInsight 群集以安装 Solr 的说明。

> [AZURE.WARNING]
完全支持通过 HDInsight 群集提供的组件，Azure 支持部门将帮助找出并解决与这些组件相关的问题。
> <p> 
> 自定义组件（如 Solr）可获得合理范围的支持，以帮助你进一步排查问题。这可能会促进解决问题，或要求使用可用的开源技术渠道，在渠道中可找到该技术的深厚的专业知识。例如，有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-cn/home?forum=hdinsight)和 [Azure CSDN](http://azure.csdn.net)。此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)。
> 
> 

## 脚本功能
此脚本对 HDInsight 群集进行以下更改：

* 将 Solr 安装到 `/usr/hdp/current/solr` 中
* 创建新用户 **solrusr** 用于运行 Solr 服务
* 将 **solruser** 设置为 `/usr/hdp/current/solr` 的所有者
* 添加 [Upstart](http://upstart.ubuntu.com/) 配置，用于在重新启动群集节点时启动 Solr。Solr 在安装后也会在群集节点上自动启动

## <a name="install"></a>使用脚本操作安装 Solr
在 HDInsight 群集上安装 Solr 的示例脚本位于以下位置。

    https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh

本部分提供有关如何在通过使用 Azure 门户预览创建新群集时使用示例脚本的说明。

> [AZURE.NOTE]
Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 模板也可用于应用脚本操作。你也可以将脚本操作应用于已在运行的群集。有关详细信息，请参阅 [Customize HDInsight clusters with Script Actions](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)（使用脚本操作自定义 HDInsight 群集）。
> 
> 

1. 使用 [Provision Linux-based HDInsight clusters](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)（预配基于 Linux 的 HDInsight 群集）中的步骤开始预配群集，但不要完成预配。
2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供以下信息：
   
    * **名称**：输入脚本操作的友好名称。
    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh
    * **标头**：选中此选项
    * **辅助角色**：选中此选项
    * **ZOOKEEPER**：选中此选项以在 Zookeeper 节点上安装
    * **参数**：将此字段留空
3. 在“脚本操作”的底部，使用“选择”按钮保存配置。最后，使用“可选配置”边栏选项卡底部的“选择”按钮保存可选配置信息。
4. 根据 [Provision Linux-based HDInsight clusters](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)（预配基于 Linux 的 HDInsight 群集）中所述继续预配群集。

## <a name="usesolr"></a>如何在 HDInsight 中使用 Solr？
### 为数据编制索引
你必须从使用一些数据文件为 Solr 编制索引开始。然后，可以使用 Solr 对索引数据运行搜索查询。使用以下步骤将一些示例数据添加到 Solr，然后查询这些数据：

1. 使用 SSH 连接到 HDInsight 群集：
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn
   
    有关如何在 HDInsight 中使用 SSH 的详细信息，请参阅以下文档：
   
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
     
     > [AZURE.IMPORTANT]
     本文档中的后续步骤将使用 SSL 隧道连接到 Solr Web UI。若要使用这些步骤，必须建立 SSL 隧道，然后将浏览器配置为使用该隧道。
     > <p> 
     > 有关详细信息，请参阅 [Use SSH Tunneling to access Ambari web UI, ResourceManager, JobHistory, NameNode, Oozie, and other web UI's](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)（使用 SSH 隧道访问 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 和其他 Web UI）
     > 
     > 
2. 使用以下命令让 Solr 为示例数据编制索引：
   
        cd /usr/hdp/current/solr/example/exampledocs
        java -jar post.jar solr.xml monitor.xml
   
    你将会在控制台上看到以下输出：
   
        POSTing file solr.xml
        POSTing file monitor.xml
        2 files indexed.
        COMMITting Solr index changes to http://localhost:8983/solr/update..
        Time spent: 0:00:01.624
   
    post.jar 实用程序通过以下两个示例文档为 Solr 编制索引：**solr.xml** 和 **monitor.xml**。这些信息存储在 Solr 中的 **collection1** 内。
3. 使用以下命令来查询 Solr 公开的 REST API：
   
        curl "http://localhost:8983/solr/collection1/select?q=*%3A*&wt=json&indent=true"
   
    这将会针对 **collection1** 中匹配 ***:***（在查询字符串中编码为 *%3A*）的所有文档发出查询，并且响应将以 JSON 形式返回。响应看上去应如下所示：
   
            "response": {
                "numFound": 2,
                "start": 0,
                "maxScore": 1,
                "docs": [
                  {
                    "id": "SOLR1000",
                    "name": "Solr, the Enterprise Search Server",
                    "manu": "Apache Software Foundation",
                    "cat": [
                      "software",
                      "search"
                    ],
                    "features": [
                      "Advanced Full-Text Search Capabilities using Lucene",
                      "Optimized for High Volume Web Traffic",
                      "Standards Based Open Interfaces - XML and HTTP",
                      "Comprehensive HTML Administration Interfaces",
                      "Scalability - Efficient Replication to other Solr Search Servers",
                      "Flexible and Adaptable with XML configuration and Schema",
                      "Good unicode support: héllo (hello with an accent over the e)"
                    ],
                    "price": 0,
                    "price_c": "0,USD",
                    "popularity": 10,
                    "inStock": true,
                    "incubationdate_dt": "2006-01-17T00:00:00Z",
                    "_version_": 1486960636996878300
                  },
                  {
                    "id": "3007WFP",
                    "name": "Dell Widescreen UltraSharp 3007WFP",
                    "manu": "Dell, Inc.",
                    "manu_id_s": "dell",
                    "cat": [
                      "electronics and computer1"
                    ],
                    "features": [
                      "30\" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
                    ],
                    "includes": "USB cable",
                    "weight": 401.6,
                    "price": 2199,
                    "price_c": "2199,USD",
                    "popularity": 6,
                    "inStock": true,
                    "store": "43.17614,-90.57341",
                    "_version_": 1486960637584081000
                  }
                ]
              }

### 使用 Solr 仪表板
Solr 仪表板是一个 Web UI，可让你通过 Web 浏览器使用 Solr。Solr 仪表板不会直接从 HDInsight 群集公开到 Internet，必须使用 SSH 隧道来访问它。有关使用 SSH 隧道的详细信息，请参阅 [Use SSH Tunneling to access Ambari web UI, ResourceManager, JobHistory, NameNode, Oozie, and other web UI's](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)（使用 SSH 隧道访问 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 和其他 Web UI）

创建 SSH 隧道后，请执行以下步骤来使用 Solr 仪表板：

1. 确定主头节点的主机名：
   
    1. 在端口 22 上使用 SSH 连接到群集。例如 `ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn`，其中 **USERNAME** 是 SSH 用户名，**CLUSTERNAME** 是群集的名称。
      
        有关使用 SSH 的详细信息，请参阅以下文档：
      
        * [Use SSH with Linux-based HDInsight from a Linux, Unix, or Mac OS X client（从 Linux、Unix 或 Mac OS X 客户端使用 SSH 连接基于 Linux 的 HDInsight）](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
        * [Use SSH with Linux-based HDInsight from a Windows client（从 Windows 客户端使用 SSH 连接基于 Linux 的 HDInsight）](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
    2. 使用以下命令获取完全限定的主机名：
      
            hostname -f
      
        此命令将返回类似于下面的名称：
      
            hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.chinacloudapp.cn
      
        这是应在以下步骤中使用的主机名。
2. 在浏览器中，连接到 **http://HOSTNAME:8983/solr/#/**，其中 **HOSTNAME** 是在前面步骤中确定的名称。
   
    应该通过 SSH 隧道将请求路由到 HDInsight 群集的头节点。你应看到类似于下面的页面：
   
    ![Solr 仪表板图像](./media/hdinsight-hadoop-solr-install-linux/solrdashboard.png)
3. 在左窗格中，使用“核心选择器”下拉列表选择“collection1”。然后，**collection1** 下面应会出现多个条目。
4. 从 **collection1** 下面的条目中选择“查询”。使用以下值来填充搜索页：
   
    * 在 **q** 文本框中，输入 ***:***。这将返回所有已在 Solr 中编制索引的文档。如果你要在文档中搜索特定字符串，则可以在此处输入该字符串。
    * 在 **wt** 文本框中，选择输出格式。默认值为 **json**。
     
     最后，选择搜索页面底部的“执行查询”按钮。
     
     ![使用脚本操作自定义群集](./media/hdinsight-hadoop-solr-install-linux/hdi-solr-dashboard-query.png)
     
     输出返回两个我们用于为 Solr 编制索引的文档。输出如下所示：
     
           "response": {
               "numFound": 2,
               "start": 0,
               "maxScore": 1,
               "docs": [
                 {
                   "id": "SOLR1000",
                   "name": "Solr, the Enterprise Search Server",
                   "manu": "Apache Software Foundation",
                   "cat": [
                     "software",
                     "search"
                   ],
                   "features": [
                     "Advanced Full-Text Search Capabilities using Lucene",
                     "Optimized for High Volume Web Traffic",
                     "Standards Based Open Interfaces - XML and HTTP",
                     "Comprehensive HTML Administration Interfaces",
                     "Scalability - Efficient Replication to other Solr Search Servers",
                     "Flexible and Adaptable with XML configuration and Schema",
                     "Good unicode support: héllo (hello with an accent over the e)"
                   ],
                   "price": 0,
                   "price_c": "0,USD",
                   "popularity": 10,
                   "inStock": true,
                   "incubationdate_dt": "2006-01-17T00:00:00Z",
                   "_version_": 1486960636996878300
                 },
                 {
                   "id": "3007WFP",
                   "name": "Dell Widescreen UltraSharp 3007WFP",
                   "manu": "Dell, Inc.",
                   "manu_id_s": "dell",
                   "cat": [
                     "electronics and computer1"
                   ],
                   "features": [
                     "30\" TFT active matrix LCD, 2560 x 1600, .25mm dot pitch, 700:1 contrast"
                   ],
                   "includes": "USB cable",
                   "weight": 401.6,
                   "price": 2199,
                   "price_c": "2199,USD",
                   "popularity": 6,
                   "inStock": true,
                   "store": "43.17614,-90.57341",
                   "_version_": 1486960637584081000
                 }
               ]
             }

### 启动和停止 Solr
如果需要手动停止或启动 Solar，请使用以下命令：

    sudo stop solr

    sudo start solr

## 备份已编制索引的数据
作为一种很好的做法，你应该将索引数据从 Solr 群集节点备份到 Azure Blob 存储上。执行以下步骤来完成此操作：

1. 使用 SSH 连接到群集，然后使用以下命令来获取头节点的主机名：
   
        hostname -f
2. 使用以下命令创建已编制索引的数据的快照。将 **HOSTNAME** 替换为上一命令返回的名称：
   
        curl http://HOSTNAME:8983/solr/replication?command=backup
   
    你应该看到如下所示的响应：
   
        <?xml version="1.0" encoding="UTF-8"?>
        <response>
          <lst name="responseHeader">
            <int name="status">0</int>
            <int name="QTime">9</int>
          </lst>
          <str name="status">OK</str>
        </response>
3. 接下来，将目录更改为 **/usr/hdp/current/solr/example/solr**。此处，每个集合都有一个子目录。每个集合目录包含一个 **data** 目录，这是该集合的快照所在的位置。
   
    例如，如果使用前面的步骤来编制示例文档的索引，则 **/usr/hdp/current/solr/example/solr/collection1/data** 目录现在应包含一个名为 **snapshot.###########** 的目录，其中的 # 是快照的日期和时间。
4. 使用类似于下面的命令创建快照文件夹的压缩存档：
   
        tar -zcf snapshot.20150806185338855.tgz snapshot.20150806185338855
   
    这将创建名为 **snapshot.20150806185338855.tgz** 的新存档，其中包含 **snapshot.20150806185338855** 目录的内容。
5. 然后，可以使用以下命令将存档存储到群集的主存储：
   
        hadoop fs -copyFromLocal snapshot.20150806185338855.tgz /example/data
   
    > [AZURE.NOTE]
    你可能想要创建一个专用目录来存储 Solr 快照。例如，`hadoop fs -mkdir /solrbackup`。
    > 
    > 

有关使用 Solr 备份和还原的详细信息，请参阅 [Making and restoring backups of SolrCores](https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups+of+SolrCores)（创建和还原 SolrCores 的备份）。

## 另请参阅
* [Install and use Hue on HDInsight clusters](/documentation/articles/hdinsight-hadoop-hue-linux/)（在 HDInsight 群集上安装并使用 Hue）。Hue 是一个 Web UI，可让你轻松地创建、运行和保存 Pig 与 Hive 作业，以及浏览 HDInsight 群集的默认存储。
* [在 HDInsight 群集上安装 R][hdinsight-install-r]。使用群集自定义在 HDInsight Hadoop 群集上安装 R。R 是一种用于统计计算的开放源代码语言和环境。它提供了数百个内置统计函数及其自己的编程语言，可结合各方面的函数编程和面向对象的编程。它还提供了各种图形功能。
* [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-linux/)。使用群集自定义在 HDInsight Hadoop 群集上安装 Giraph。Giraph 可让你通过使用 Hadoop 执行图形处理，并可以在 Azure HDInsight 上使用。
* [Install Hue on HDInsight clusters](/documentation/articles/hdinsight-hadoop-hue-linux/)（在 HDInsight 群集上安装 Hue）。使用群集自定义在 HDInsight Hadoop 群集上安装 Hue。Hue 是用来与 Hadoop 群集交互的一系列 Web 应用程序。

[hdinsight-install-r]: /documentation/articles/hdinsight-hadoop-r-scripts-linux/
[hdinsight-cluster-customize]: /documentation/articles/hdinsight-hadoop-customize-cluster-linux/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->