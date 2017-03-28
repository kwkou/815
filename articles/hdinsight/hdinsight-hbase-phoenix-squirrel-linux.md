<properties 
   pageTitle="在 Azure HDInsight (HBase) 中使用 Apache Phoenix 和 SQuirreL | Azure" 
   description="了解如何在 HDInsight 中使用 Apache Phoenix，以及如何在工作站上安装和配置 SQuirreL 以连接到 HDInsight 中的 HBase 群集。" 
   services="hdinsight" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="09/02/2016"
	wacn.date="03/28/2017"/>

# 将 Apache Phoenix 与 HDinsight 中基于 Linux 的 HBase 群集配合使用  

了解如何在 HDInsight 中使用 [Apache Phoenix](http://phoenix.apache.org/)，以及如何使用 SQLLine。有关 Phoenix 的详细信息，请参阅[在 15 分钟或更短时间内了解 Phoenix](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html)。有关 Phoenix 语法，请参阅 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。

>[AZURE.NOTE] 有关 HDInsight 中的 Phoenix 版本信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][hdinsight-versions]。

##使用 SQLLine
[SQLLine](http://sqlline.sourceforge.net/) 是用于执行 SQL 的命令行实用工具。

###先决条件
在使用 SQLLine 之前，必须先准备好以下各项：

- **HDInsight 中的 HBase 群集**。有关预配 HBase 群集的信息，请参阅 [HDInsight 中的 Apache HBase 入门][hdinsight-hbase-get-started]。
- **通过远程桌面协议连接到 HBase 群集**。有关说明，请参阅[使用 Azure 门户预览管理 HDInsight 中的 Hadoop 群集][hdinsight-manage-portal]。


在连接到 HBase 群集时，你需要连接到 Zookeeper 之一。每个 HDInsight 群集有 3 个 Zookeeper。

**了解 Zookeeper 主机名**

1. 通过浏览到 **https://\<ClusterName\>.azurehdinsight.cn** 打开 Ambari。
2. 输入 HTTP（群集）用户名和密码以登录。
3. 单击左侧菜单中的 **ZooKeeper**。你应看到列出 3 个 **ZooKeeper 服务器**。
4. 单击列出的其中一个 **ZooKeeper 服务器**。在“摘要”窗格中，找到**主机名**。它类似于 *zk1-jdolehb.3lnng4rcvp5uzokyktxs4a5dhd.bx.internal.chinacloudapp.cn*。

**使用 SQLLine**

1. 使用 SSH 连接到群集。有关说明，请参阅[在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)或 [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)，具体取决于你的客户端计算机 OS。

2. 从 SSH 运行以下命令以运行 SQLLine：

        cd /usr/hdp/2.2.9.1-7/phoenix/bin
        ./sqlline.py <ClusterName>:2181:/hbase-unsecure

2. 运行以下命令以创建 HBase 表，并插入一些数据：

		CREATE TABLE Company (COMPANY_ID INTEGER PRIMARY KEY, NAME VARCHAR(225));
	
		!tables
		
		UPSERT INTO Company VALUES(1, 'Microsoft');
		
		SELECT * FROM Company;
        
        !quit

有关详细信息，请参阅 [SQLLine 手册](http://sqlline.sourceforge.net/#manual)和 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。


 
##后续步骤
在本文中，你已了解如何在 HDInsight 中使用 Apache Phoenix。若要了解详细信息，请参阅

- [HDInsight HBase 概述][hdinsight-hbase-overview]：
HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。
- [在 Azure 虚拟网络上设置 HBase 群集][hdinsight-hbase-provision-vnet]：
通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。
- [在 HDInsight 中配置 HBase 地域复制](/documentation/articles/hdinsight-hbase-geo-replication/)： 了解如何跨两个 Azure 数据中心配置 HBase 复制。 

[azure-portal]: https://manage.windowsazure.cn
[vnet-point-to-site-connectivity]: https://msdn.microsoft.com/zh-cn/library/azure/09926218-92ab-4f43-aa99-83ab4d355555#BKMK_VNETPT

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning/
[hdinsight-hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-linux/
[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal/#connect-to-hdinsight-clusters-by-using-rdp
[hdinsight-hbase-provision-vnet]: /documentation/articles/hdinsight-hbase-provision-vnet/
[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview/

[hdinsight-hbase-phoenix-sqlline]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-phoenix-sqlline.png
[img-certificate]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vpn-certificate.png
[img-vnet-diagram]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vnet-point-to-site.png
[img-squirrel-driver]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-driver.png
[img-squirrel-alias]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-alias.png
[img-squirrel]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel.png
[img-squirrel-sql]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-sql.png


 

<!---HONumber=Mooncake_0405_2016-->