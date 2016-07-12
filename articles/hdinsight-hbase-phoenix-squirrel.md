<properties 
   pageTitle="在 HDInsight 中使用 Apache Phoenix 和 Squirrel | Azure" 
   description="了解如何在 HDInsight 中使用 Apache Phoenix，以及如何在工作站上安装和配置 SQuirrel 以连接到 HDInsight 中的 HBase 群集。" 
   services="hdinsight" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/04/2016"
	wacn.date="04/26/2016"/>

# 将 Apache Phoenix 和 SQuirreL 与 HDinsight 中的 HBase 配合使用  

了解如何在 HDInsight 中使用 [Apache Phoenix](http://phoenix.apache.org/)，以及如何在工作站上安装和配置 SQuirrel 以连接到 HDInsight 中的 HBase 群集。有关 Phoenix 的详细信息，请参阅[在 15 分钟或更短时间内了解 Phoenix](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html)。有关 Phoenix 语法，请参阅 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。

>[AZURE.NOTE]有关 HDInsight 中的 Phoenix 版本信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][hdinsight-versions]。

## 使用 SQLLine
[SQLLine](http://sqlline.sourceforge.net/) 是用于执行 SQL 的命令行实用工具。

### 先决条件
在使用 SQLLine 之前，必须先准备好以下各项：

- **HDInsight 中的 HBase 群集**。有关预配 HBase 群集的信息，请参阅 [HDInsight 中的 Apache HBase 入门][hdinsight-hbase-get-started]。
- **通过远程桌面协议连接到 HBase 群集**。有关说明，请参阅[使用 Azure 经典管理门户在 HDInsight 中管理 Hadoop 群集][hdinsight-manage-portal]。

**找出主机名**

1. 从桌面打开 **Hadoop 命令行**。
2. 运行以下命令以获取 DNS 后缀:

		ipconfig

	记下**特定于连接的 DNS 后缀**。例如 *myhbasecluster.f5.internal.chinacloudapp.cn*。在连接到 HBase 群集时，你需要使用 FQDN 连接到 Zookeeper 之一。每个 HDInsight 群集有 3 个 Zookeeper。它们分别是 *zookeeper0*、*zookeeper1* 和 *zookeeper2*。FQDN 类似于 *zookeeper2.myhbasecluster.f5.internal.chinacloudapp.cn*。

**使用 SQLLine**

1. 从桌面打开 **Hadoop 命令行**。
2. 运行以下命令以打开 SQLLine：

		cd %phoenix_home%\bin
		sqlline.py [The FQDN of one of the Zookeepers]

	![hdinsight hbase phoenix sqlline][hdinsight-hbase-phoenix-sqlline]

	示例中使用的命令：

		CREATE TABLE Company (COMPANY_ID INTEGER PRIMARY KEY, NAME VARCHAR(225));
		
		!tables
		
		UPSERT INTO Company VALUES(1, 'Microsoft');
		
		SELECT * FROM Company;

有关详细信息，请参阅 [SQLLine 手册](http://sqlline.sourceforge.net/#manual)和 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。


















## 使用 SQuirrel

[SQuirreL SQL 客户端](http://squirrel-sql.sourceforge.net/)是一种图形 Java 程序，可让你查看 JDBC 兼容数据库的结构，浏览表中的数据，发出 SQL 命令，等等。它可用于连接到 HDInsight 上的 Apache Phoenix。

### 先决条件

在执行步骤之前，必须准备好以下各项：

- 已将一个 HBase 群集部署到包含 DNS 虚拟机的 Azure 虚拟网络。有关说明，请参阅[在 Azure 虚拟网络上预配 HBase 群集][hdinsight-hbase-provision-vnet-v1]。 

	>[AZURE.IMPORTANT]必须在虚拟网络中安装一个 DNS 服务器。有关说明，请参阅[在两个 Azure 虚拟网络之间配置 DNS](/documentation/articles/hdinsight-hbase-geo-replication-configure-DNS/)

- 获取 HBase 群集的特定于连接的 DNS 后缀。若要获取该后缀，请与群集建立连接桌面连接 (RDP)，然后运行 IPConfig。DNS 后缀类似于：

		myhbase.b7.internal.chinacloudapp.cn
- 在工作站中下载并安装 [Microsoft Visual Studio Express 2013 for Windows Desktop](https://www.visualstudio.com/products/visual-studio-express-vs.aspx)。你将需要使用该程序包的 makecert 来创建证书。  
- 在工作站中下载并安装 [Java 运行时环境](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)。SQuirrel SQL 客户端 3.0 和更高版本需要 JRE 1.6 或更高版本。  


### 在工作站上安装并配置 SQuirrel

**安装 SQuirrel**

1. 从 [http://squirrel-sql.sourceforge.net/#installation](http://squirrel-sql.sourceforge.net/#installation) 下载 SQuirrel SQL 客户端 jar 文件。
2. 打开/运行该 jar 文件。它需要 [Java 运行时环境](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)。
3. 单击“下一步”两次。
4. 指定你具有写入权限的路径，然后单击“下一步”。
	>[AZURE.NOTE]默认的安装文件夹为 C:\\Program Files\\squirrel-sql-3.6 文件夹。若要写入此路径，必须为安装程序授予管理员权限。你可以管理员身份打开命令提示符，导航到 Java 的 bin 文件夹，然后运行
	><p>`java.ejava.exe -jar [the path of the SQuirrel jar file]`
5. 单击“确定”确认创建目标目录。
6. 默认设置是安装基本和标准程序包。单击**“下一步”**。
7. 单击“下一步”两次，然后单击“完成”。


**安装 Phoenix 驱动程序**

Phoenix 驱动程序 jar 文件位于 HBase 群集上。根据具体的版本，其路径类似于：

	C:\apps\dist\phoenix-4.0.0.2.1.11.0-2316\phoenix-4.0.0.2.1.11.0-2316-client.jar
你需要将它复制到工作站的 [SQuirrel 安装文件夹]/lib 路径下。最简单的方法是与群集建立 RDP，然后使用文件复制/粘贴功能（CTRL+C 和 CTRL+V）将其复制到工作站。

**将 Phoenix 驱动程序添加到 SQuirrel**

1. 从工作站打开 SQuirrel SQL 客户端。
2. 在左侧单击“驱动程序”选项卡。
2. 在“驱动程序”菜单中，单击“新建驱动程序”。
3. 输入以下信息：

	- **名称**：Phoenix
	- **示例 URL**：jdbc:phoenix:zookeeper2.contoso-hbase-eu.f5.internal.chinacloudapp.cn
	- **类名**：org.apache.phoenix.jdbc.PhoenixDriver

	>[AZURE.WARNING]在“示例 URL”中使用全小写。当其中一个主机关闭时，你可以使用整个 zookeeper 仲裁。主机名为 zookeeper0、zookeeper1 和 zookeeper2。

	![HDInsight HBase Phoenix SQuirrel 驱动程序][img-squirrel-driver]
4. 单击**“确定”**。

**创建 HBase 群集的别名**

1. 在 SQuirrel 中，单击左侧的“别名”选项卡。
2. 在“别名”菜单中，单击“新建别名”。
3. 输入以下信息：

	- **名称**：HBase 群集的名称，或者所需的任何名称。
	- **驱动程序**：Phoenix。它必须与你在上一过程中创建的驱动程序名称匹配。
	- **URL**：从驱动程序配置中复制的 URL。确保使用全小写。
	- **用户名**：可以是任何文本。
	- **密码**：可以是任何文本。

	![HDInsight HBase Phoenix SQuirrel 驱动程序][img-squirrel-alias]
4. 单击“测试”。 
5. 单击“连接”。在建立连接时，SQuirrel 类似于：

	![HBase Phoenix SQuirrel][img-squirrel]

**运行测试**

1. 单击“对象”选项卡旁边的“SQL”选项卡。
2. 复制并粘贴以下代码：

		CREATE TABLE IF NOT EXISTS us_population (state CHAR(2) NOT NULL, city VARCHAR NOT NULL, population BIGINT  CONSTRAINT my_pk PRIMARY KEY (state, city))
3. 单击运行按钮。

	![HBase Phoenix SQuirrel][img-squirrel-sql]
4. 切换回到“对象”选项卡。
5. 展开别名，然后展开“表”。你应会看到下面列出新表。
 
## 后续步骤
在本文中，你已了解如何在 HDInsight 中使用 Apache Phoenix。若要了解详细信息，请参阅

- [HDInsight HBase 概述][hdinsight-hbase-overview]：HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。
- [在 Azure 虚拟网络上设置 HBase 群集][hdinsight-hbase-provision-vnet-v1]：通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。
- [在 HDInsight 中配置 HBase 地域复制](/documentation/articles/hdinsight-hbase-geo-replication/)：了解如何跨两个 Azure 数据中心配置 HBase 复制。 

[azure-portal]: https://manage.windowsazure.cn

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning-v1/
[hdinsight-hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-v1/
[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/#connect-to-hdinsight-clusters-by-using-rdp
[hdinsight-hbase-provision-vnet-v1]: /documentation/articles/hdinsight-hbase-provision-vnet-v1/
[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview/
[hdinsight-hbase-phoenix-sqlline]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-phoenix-sqlline.png
[img-certificate]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vpn-certificate.png
[img-vnet-diagram]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vnet-point-to-site.png
[img-squirrel-driver]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-driver.png
[img-squirrel-alias]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-alias.png
[img-squirrel]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel.png
[img-squirrel-sql]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-sql.png

<!---HONumber=71-->