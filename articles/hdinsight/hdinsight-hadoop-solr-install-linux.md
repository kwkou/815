<properties
    pageTitle="使用脚本操作在基于 Linux 的 HDInsight 上安装 Solr | Azure"
    description="了解如何使用脚本操作在基于 Linux 的 HDInsight Hadoop 群集上安装 Solr。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="cc93ed5c-a358-456a-91a4-f179185c0e98"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/14/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="eb3391c15c12dda10ac53fce01774e6c9580c3c1"
    ms.lasthandoff="04/28/2017" />

# <a name="install-and-use-solr-on-hdinsight-hadoop-clusters"></a>在 HDInsight Hadoop 群集上安装并使用 Solr

了解如何使用脚本操作在 Azure HDInsight 上安装 Solr。 Solr 是一种功能强大的搜索平台，提供了企业级搜索功能，用于搜索由 Hadoop 管理的数据。

> [AZURE.IMPORTANT]
> 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

> [AZURE.IMPORTANT]
> 此文档中使用的示例脚本通过特定配置创建 Solr 群集。 如果要使用不同集合、分片、架构、副本等配置 Solr 群集，必须修改脚本和 Solr 二进制文件。

## <a name="whatis"></a>什么是 Solr

[Apache Solr](http://lucene.apache.org/solr/features.html) 是一种企业搜索平台，用于对数据实现功能强大的全文搜索。 虽然 Hadoop 可用于存储和管理大量数据，但是，Apache Solr 提供了快速检索数据的搜索功能。

> [AZURE.WARNING]
> Microsoft 完全支持 HDInsight 群集提供的组件。
><p>
> 自定义组件（如 Solr）可获得合理范围的支持，以帮助进一步排查问题。 Azure.cn 支持可能无法解决与自定义组件有关的问题。 用户可能需要寻求开源社区的帮助。 有许多可以使用的社区站点，例如：[HDInsight 的 MSDN 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hdinsight)和 [Azure CSDN](http://azure.csdn.net)。 此外，Apache 项目在 [http://apache.org](http://apache.org) 上提供了项目站点，例如 [Hadoop](http://hadoop.apache.org/)。

## <a name="what-the-script-does"></a>脚本功能

此脚本对 HDInsight 群集进行以下更改：

* 将 Solr 安装到 `/usr/hdp/current/solr` 中
* 创建用于运行 Solr 服务的用户 **solrusr**
* 将 **solruser** 设置为 `/usr/hdp/current/solr` 的所有者
* 添加自动启动 Solr 的 [Upstart](http://upstart.ubuntu.com/) 配置。

## <a name="install"></a>使用脚本操作安装 Solr

在 HDInsight 群集上安装 Solr 的示例脚本位于以下位置：

    https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh

若要创建安装了 Solr 的群集，请使用[创建 HDInsight 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)文档中的步骤。 在创建过程中，使用以下步骤安装 Solr：

1. 在“群集摘要”边栏选项卡中，选择“高级设置”，然后选择“脚本操作”。 使用以下信息填充表格：

    * **名称**：输入脚本操作的友好名称。
    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh
    * **标头**：选中此选项
    * **辅助角色**：选中此选项
    * **ZOOKEEPER**：选中此选项以在 Zookeeper 节点上安装
    * **参数**：将此字段留空

2. 在“脚本操作”边栏选项卡底部，使用“选择”按钮保存配置。 最后，使用“下一步”按钮返回到“群集摘要”

3. 在“群集摘要”页上，选择“创建”以创建群集。

## <a name="usesolr"></a>如何在 HDInsight 中使用 Solr

> [AZURE.IMPORTANT]
> 本节中的步骤将演示基本 Solr 功能。 有关使用 Solr 的详细信息，请参阅 [Apache Solr 站点](http://lucene.apache.org/solr/)。

### <a name="index-data"></a>索引数据

使用以下步骤将示例数据添加到 Solr，然后查询该数据：

1. 使用 SSH 连接到 HDInsight 群集：

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn

    有关详细信息，请参阅[对 HDInsight 使用 SSH](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

     > [AZURE.IMPORTANT]
     > 本文档中的后续步骤使用 SSL 隧道连接到 Solr Web UI。 若要使用这些步骤，必须建立 SSL 隧道，然后将浏览器配置为使用该隧道。
     >
     > 有关详细信息，请参阅[将 SSH 隧道与 HDInsight 配合使用](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)文档。

2. 使用以下命令让 Solr 为示例数据编制索引：

        cd /usr/hdp/current/solr/example/exampledocs
        java -jar post.jar solr.xml monitor.xml

    以下输出将返回到控制台：

        POSTing file solr.xml
        POSTing file monitor.xml
        2 files indexed.
        COMMITting Solr index changes to http://localhost:8983/solr/update..
        Time spent: 0:00:01.624

    `post.jar` 实用工具将 **solr.xml** 和 **monitor.xml** 文档添加到索引。

3. 使用以下命令查询 Solr REST API：

        curl "http://localhost:8983/solr/collection1/select?q=*%3A*&wt=json&indent=true"

    此命令为任何匹配 **\*:\*** 的文档搜索 **collection1**（在查询字符串中编码为 \*%3A\*）。 以下 JSON 文档是响应的示例：

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

### <a name="using-the-solr-dashboard"></a>使用 Solr 仪表板

Solr 仪表板是一个 Web UI，可让你通过 Web 浏览器使用 Solr。 Solr 仪表板不会直接从 HDInsight 群集公开到 Internet。 可以使用 SSH 隧道访问该仪表板。 有关使用 SSH 隧道的详细信息，请参阅[将 SSH 隧道与 HDInsight 配合使用](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)文档。

创建 SSH 隧道后，请执行以下步骤来使用 Solr 仪表板：

1. 确定主头节点的主机名：

    1. 使用 SSH 连接到群集头节点。 例如，`ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn`。

        有关使用 SSH 的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

    2. 使用以下命令获取完全限定的主机名：

            hostname -f

        此命令返回类似于以下主机名的值：

            hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.chinacloudapp.cn

        保存返回的值，后面将用到它。

2. 在浏览器中，连接到 **http://HOSTNAME:8983/solr/#/**，其中 **HOSTNAME** 是在前面步骤中确定的名称。

    请求在群集上通过 SSH 隧道路由到 Solr Web UI。 显示的页面类似于下图：

    ![Solr 仪表板图像](./media/hdinsight-hadoop-solr-install-linux/solrdashboard.png)

3. 在左窗格中，使用“核心选择器”下拉列表选择 **collection1**。 然后，**collection1** 下面应会出现多个条目。

4. 从 **collection1** 下面的条目中选择“查询”。 使用以下值来填充搜索页：

    * 在 **q** 文本框中，输入 **\*:**\*。 此查询返回所有已在 Solr 中编制索引的文档。 如果要在文档中搜索特定字符串，则可以在此处输入该字符串。
    * 在 **wt** 文本框中，选择输出格式。 默认值为 **json**。

    最后，选择搜索页面底部的“执行查询”按钮。

    ![使用脚本操作自定义群集](./media/hdinsight-hadoop-solr-install-linux/hdi-solr-dashboard-query.png)

    输出返回之前已添加到索引的两个文档。 输出类似于以下 JSON 文档：

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

### <a name="starting-and-stopping-solr"></a>启动和停止 Solr

使用以下命令手动停止和启动 Solr：

    sudo stop solr
    sudo start solr

## <a name="backup-indexed-data"></a>备份已编制索引的数据

使用以下步骤，将 Solr 数据备份到群集的默认存储：

1. 使用 SSH 连接到群集，然后使用以下命令来获取头节点的主机名：

        hostname -f

2. 使用以下命令创建已编制索引的数据的快照。 将 **HOSTNAME** 替换为上一命令返回的名称：

        curl http://HOSTNAME:8983/solr/replication?command=backup

    响应类似于以下 XML：

        <?xml version="1.0" encoding="UTF-8"?>
        <response>
          <lst name="responseHeader">
            <int name="status">0</int>
            <int name="QTime">9</int>
          </lst>
          <str name="status">OK</str>
        </response>

3. 将目录更改为 `/usr/hdp/current/solr/example/solr`。 每个集合在此处都有子目录。 每个集合的目录都包含 `data` 目录，该目录包含集合的快照。

4. 若要创建快照文件夹的压缩存档，请使用以下命令：

        tar -zcf snapshot.20150806185338855.tgz snapshot.20150806185338855

    将 `snapshot.20150806185338855` 值替换为集合的快照名称。

    此命令创建名为 **snapshot.20150806185338855.tgz** 的存档，而该存档包含 **snapshot.20150806185338855** 目录的内容。

5. 然后，可以使用以下命令将存档存储到群集的主存储：

        hdfs dfs -put snapshot.20150806185338855.tgz /example/data

有关使用 Solr 备份和还原的详细信息，请参阅[创建和还原 SolrCores 的备份](https://cwiki.apache.org/confluence/display/solr/Making+and+Restoring+Backups+of+SolrCores)。

## <a name="next-steps"></a>后续步骤

* [在 HDInsight 群集上安装 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-linux/)。 使用群集自定义在 HDInsight Hadoop 群集上安装 Giraph。 Giraph 可让你通过使用 Hadoop 执行图形处理，并可以在 Azure HDInsight 上使用。

* [在 HDInsight 群集上安装 Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)。 使用群集自定义在 HDInsight Hadoop 群集上安装 Hue。 Hue 是用来与 Hadoop 群集交互的一系列 Web 应用程序。

[hdinsight-cluster-customize]: /documentation/articles/hdinsight-hadoop-customize-cluster-linux/

<!--Update_Description: wording update-->