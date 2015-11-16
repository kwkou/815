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
   ms.date="07/29/2015"
   wacn.date="10/03/2015"/>

# 将 Apache Phoenix 和 SQuirreL 与 HDinsight 中的 HBase 配合使用  

了解如何在 HDInsight 中使用 [Apache Phoenix](http://phoenix.apache.org/)，以及如何在工作站上安装和配置 SQuirrel 以连接到 HDInsight 中的 HBase 群集。有关 Phoenix 的详细信息，请参阅[在 15 分钟或更短时间内了解 Phoenix](http://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html)。有关 Phoenix 语法，请参阅 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。

>[AZURE.NOTE]有关 HDInsight 中的 Phoenix 版本信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][hdinsight-versions]。

## 使用 SQLLine
[SQLLine](http://sqlline.sourceforge.net/) 是用于执行 SQL 的命令行实用工具。

### 先决条件
在使用 SQLLine 之前，必须先准备好以下各项：

- **HDInsight 中的 HBase 群集**。有关预配 HBase 群集的信息，请参阅 [HDInsight 中的 Apache HBase 入门][hdinsight-hbase-get-started]。
- **通过远程桌面协议连接到 HBase 群集**。有关说明，请参阅[使用 Azure 门户在 HDInsight 中管理 Hadoop 群集][hdinsight-manage-portal]。

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
		
		!tables;
		
		UPSERT INTO Company VALUES(1, 'Microsoft');
		
		SELECT * FROM Company;

有关详细信息，请参阅 [SQLLine 手册](http://sqlline.sourceforge.net/#manual)和 [Phoenix 语法](http://phoenix.apache.org/language/index.html)。


















## 使用 SQuirrel

[SQuirreL SQL 客户端](http://squirrel-sql.sourceforge.net/)是一种图形 Java 程序，可让你查看 JDBC 兼容数据库的结构，浏览表中的数据，发出 SQL 命令，等等。它可用于连接到 HDInsight 上的 Apache Phoenix。

本部分说明了如何在工作站上安装和配置 Squirrel，以通过 VPN 连接到 HDInsight 中的 HBase 群集。

### 先决条件

在执行步骤之前，必须准备好以下各项：

- 已将一个 HBase 群集部署到包含 DNS 虚拟机的 Azure 虚拟网络。有关说明，请参阅[在 Azure 虚拟网络上预配 HBase 群集][hdinsight-hbase-provision-vnet]。 

	>[AZURE.IMPORTANT]必须在虚拟网络中安装一个 DNS 服务器。有关说明，请参阅[在两个 Azure 虚拟网络之间配置 DNS](/documentation/articles/hdinsight-hbase-geo-replication-configure-DNS)

- 获取 HBase 群集的特定于连接的 DNS 后缀。若要获取该后缀，请与群集建立连接桌面连接 (RDP)，然后运行 IPConfig。DNS 后缀类似于：

		myhbase.b7.internal.chinacloudapp.cn
- 在工作站中下载并安装 [Microsoft Visual Studio Express 2013 for Windows Desktop](https://www.visualstudio.com/products/visual-studio-express-vs.aspx)。你将需要使用该程序包的 makecert 来创建证书。  
- 在工作站中下载并安装 [Java 运行时环境](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)。SQuirrel SQL 客户端 3.0 和更高版本需要 JRE 1.6 或更高版本。  


### 配置与 Azure 虚拟网络的点到站点 VPN 连接

配置点到站点 VPN 连接包括 3 个步骤：

1. [配置虚拟网络和动态路由网关](#Configure-a-virtual-network-and-a-dynamic-routing-gateway)
2. [创建证书](#Create-your-certificates)
3. [配置 VPN 客户端](#Configure-your-VPN-client)

有关详细信息，请参阅[配置与 Azure 虚拟网络的点到站点 VPN 连接](/documentation/articles/vpn-gateway-point-to-site-create)。

#### 配置虚拟网络和动态路由网关

确保你已在 Azure 虚拟网络中设置 HBase 群集（请参阅本部分的先决条件）。下一步是配置点到站点连接。

**配置点到站点连接**

1. 登录到 [Azure 门户][azure-portal]。
2. 在左侧单击“网络”。
3. 单击你创建的虚拟网络（请参阅[在 Azure 虚拟网络上预配 HBase 群集][hdinsight-hbase-provision-vnet]）。
4. 在顶部单击“配置”。
5. 在“点到站点连接”部分中，选择“配置点到站点连接”。 
6. 配置“起始 IP”和“CIDR”，以指定你的 VPN 客户端在连接后接收 IP 地址时的 IP 地址范围。范围不能与本地网络和要连接到的 Azure 虚拟网络上的任何范围重叠。例如，如果你为虚拟网络选择了 10.0.0.0/20，则可为客户端地址空间选择 10.1.0.0/24。页。
7. 在虚拟网络地址空间部分中，单击“添加网关子网”。
7. 单击页面底部的“保存”。
8. 单击“是”确认更改。等到系统完成更改，然后转到下一过程。


**创建动态路由网关**

1. 在 Azure 门户中，单击页面顶部的“仪表板”。
2. 单击页面底部的“创建网关”。
3. 单击“是”确认。等到网关创建完成。
4. 在顶部单击“仪表板”。你将看到虚拟网络的可视示意图：

	![Azure 虚拟网络点到站点虚拟图][img-vnet-diagram]

	该图显示了 0 个客户端连接。在与虚拟网络建立连接后，数字将更新为 1。

#### 创建证书

创建 X.509 证书的方法之一是使用 [Microsoft Visual Studio Express 2013 for Windows Desktop](https://www.visualstudio.com/products/visual-studio-express-vs.aspx) 随附的证书创建工具 (makecert.exe)。


**创建自签名根证书**

1. 在工作站上打开命令提示窗口。
2. 导航到 Visual Studio 工具文件夹。 
3. 示例中的以下命令将在工作站上的“个人”证书存储区中创建和安装根证书，并创建你随后将要上载到 Azure 门户的相应 .cer 文件。 

		makecert -sky exchange -r -n "CN=HBaseVnetVPNRootCertificate" -pe -a sha1 -len 2048 -ss My "C:\Users\JohnDole\Desktop\HBaseVNetVPNRootCertificate.cer"

	切换到要用于放置该 .cer 文件的目录，其中，HBaseVnetVPNRootCertificate 是你希望用于证书的名称。

	请不要关闭命令提示符。下一个过程将要用到它。

	>[AZURE.NOTE]因为你创建了将从其生成客户端证书的根证书，可能需要导出此根证书以及私钥，并将它保存到一个可以恢复的安全位置。

**创建客户端证书**

- 从同一个命令提示符（必须在创建根证书的同一计算机上。客户端证书必须从根证书生成），运行以下命令：

  		makecert.exe -n "CN=HBaseVnetVPNClientCertificate" -pe -sky exchange -m 96 -ss My -in "HBaseVnetVPNRootCertificate" -is my -a sha1

	HBaseVnetVPNRootCertificate 是根证书名称。它必须与根证书名称匹配。

	根证书和客户端证书都存储在计算机上的“个人”证书存储中。使用 certmgr.msc 进行验证。

	![Azure 虚拟网络点到站点 VPN 证书][img-certificate]

	必须在要连接到虚拟网络的每台计算机上都安装客户端证书。建议为要连接到虚拟网络的每台计算机都创建唯一的客户端证书。若要导出客户端证书，请使用 certmgr.msc。

**将根证书上载到 Azure 门户**

1. 在 Azure 门户中，单击左侧的“网络”。
2. 单击 HBase 群集部署到的虚拟网络。
3. 在顶部单击“证书”。
4. 在底部单击“上载”，然后指定最后一个过程前面的过程中创建的根证书文件。等到证书导入完成。
5. 在顶部单击“仪表板”。虚拟图将显示状态。


#### 配置 VPN 客户端



**下载并安装客户端 VPN 程序包**

1. 在虚拟网络“仪表板”页上的“速览”部分中，根据工作站 OS 版本，单击“下载 64 位客户端 VPN 程序包”或“下载 32 位客户端 VPN 程序包”。
2. 单击“运行”安装该程序包。
3. 在安全提示符下，单击“更多信息”，然后单击“仍然运行”。
4. 单击“是”两次。

**连接到 VPN**

1. 在工作站的桌面上，单击任务栏上的“网络”图标。你应会看到包含虚拟网络名称的 VPN 连接。
2. 单击 VPN 连接名称。
3. 单击“连接”。

**测试 VPN 连接和域名解析**

- 在工作站上打开命令提示符并 ping 以下名称之一（如果 HBase 群集的 DNS 后缀是 myhbase.b7.internal.chinacloudapp.cn）：

		zookeeper0.myhbase.b7.internal.chinacloudapp.cn
		zookeeper0.myhbase.b7.internal.chinacloudapp.cn
		zookeeper0.myhbase.b7.internal.chinacloudapp.cn
		headnode0.myhbase.b7.internal.chinacloudapp.cn
		headnode1.myhbase.b7.internal.chinacloudapp.cn
		workernode0.myhbase.b7.internal.chinacloudapp.cn

### 在工作站上安装并配置 SQuirrel

**安装 SQuirrel**

1. 从 [http://squirrel-sql.sourceforge.net/#installation](http://squirrel-sql.sourceforge.net/#installation) 下载 SQuirrel SQL 客户端 jar 文件。
2. 打开/运行该 jar 文件。它需要 [Java 运行时环境](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)。
3. 单击“下一步”两次。
4. 指定你具有写入权限的路径，然后单击“下一步”。
	>[AZURE.NOTE]默认的安装文件夹为 C:\\Program Files\\squirrel-sql-3.6 文件夹。若要写入此路径，必须为安装程序授予管理员权限。你可以管理员身份打开命令提示符，导航到 Java 的 bin 文件夹，然后运行
	>
	>     java.exe -jar [the path of the SQuirrel jar file] 
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
	- **用户名**：可以是任何文本。由于你在此处使用了 VPN 连接，因此根本不需要用户名。
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
- [在 Azure 虚拟网络上设置 HBase 群集][hdinsight-hbase-provision-vnet]：通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。
- [在 HDInsight 中配置 HBase 地域复制](/documentation/articles/hdinsight-hbase-geo-replication)：了解如何跨两个 Azure 数据中心配置 HBase 复制。 

[azure-portal]: https://manage.windowsazure.cn

[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning
[hdinsight-hbase-get-started]: /documentation/articles/hdinsight-hbase-get-started
[hdinsight-manage-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1#connect-to-hdinsight-clusters-by-using-rdp
[hdinsight-hbase-provision-vnet]: /documentation/articles/hdinsight-hbase-provision-vnet
[hdinsight-hbase-overview]: /documentation/articles/hdinsight-hbase-overview
[hdinsight-hbase-phoenix-sqlline]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-phoenix-sqlline.png
[img-certificate]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vpn-certificate.png
[img-vnet-diagram]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-vnet-point-to-site.png
[img-squirrel-driver]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-driver.png
[img-squirrel-alias]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-alias.png
[img-squirrel]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel.png
[img-squirrel-sql]: ./media/hdinsight-hbase-phoenix-squirrel/hdinsight-hbase-squirrel-sql.png

<!---HONumber=71-->