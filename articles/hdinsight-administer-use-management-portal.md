<properties linkid="manage-services-hdinsight-howto-administer-hdinsight" urlDisplayName="Administration" pageTitle="Administer HDInsight clusters with Management Portal | Azure" metaKeywords="" description="Learn how to administer HDInsight Service. Create an HDInsight cluster, open the interactive JavaScript console, and open the Hadoop command console." metaCanonical="" services="hdinsight" documentationCenter="" title="Administer HDInsight clusters using Management Portal" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# 使用管理门户管理 HDInsight 群集

使用 Azure 管理门户，可以设置 HDInsight 集群、更改 Hadoop 用户密码，以及启用 RDP 以访问群集上的 Hadoop 命令控制台。除了管理门户外，还有其他可用于管理 HDInsight 的工具。

-   有关使用 Azure PowerShell 管理 HDInsight 的详细信息，请参阅[使用 PowerShell 管理 HDInsight][]。

-   有关使用跨平台命令行工具管理 HDInsight 的详细信息，请参阅[使用跨平台命令行界面管理 HDInsight][]。

**先决条件：**

在开始阅读本文前，你必须具有：

-   **Azure 订阅**。Azure 是基于订阅的平台。有关获取订阅的信息，请参阅[购买选项][]、[成员优惠][购买选项]或[免费试用][]。

## 本文内容

-   [配置 HDInsight 群集][]
-   [自定义 HDInsight 群集][]
-   [更改 HDInsight 群集用户名和密码][]
-   [使用 RDP 连接到 HDInsight 群集][]
-   [授予/撤消 HTTP 服务访问权限][]
-   [打开 Hadoop 命令控制台][]
-   [后续步骤][]

## 设置 HDInsight 群集

有几种方法可创建 HDInsight 群集，本文只介绍如何使用 Azure 管理门户上的“快速创建”选项。有关其他选项，请参阅[设置 HDInsight 群集][]。

HDInsight 群集使用 Azure Blob 存储容器作为默认文件系统。有关 Azure Blob 存储如何提供与 HDInsight 群集的无缝体验的详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

必须在设置 HDInsight 群集的同一数据中心内创建 Azure 存储帐户。当前可以在五个数据中心内设置 HDInsight 群集：

-   亚洲东南部
-   欧洲北部
-   欧洲西部
-   美国东部
-   美国西部

有关创建 Azure 存储帐户的详细信息，请参阅[如何创建存储帐户][]。

**设置 HDInsight 群集**

1.  登录到 [Azure 管理门户][]。
2.  单击页面底部的“新建” ，然后依次单击“数据服务”、 “HDINSIGHT” 和“快速创建” 。

3.  提供“群集名称” 、“群集大小” 、“群集管理员密码” 和 Azure“存储帐户” ，然后单击“创建 HDInsight 群集” 。创建并运行群集后，状态将显示“正在运行”**。

    ![HDI.QuickCreate][]

    当使用“快速创建”选项创建群集时，管理员帐户的默认用户名为“admin”**。若要为该帐户提供其他用户名，可以使用“自定义创建”选项，而不是“快速创建”选项。也可以在设置之后更改帐户名称。

    使用“快速创建”选项创建群集时，将在指定的存储帐户中自动创建一个带 HDInsight 群集名称的新容器。如果要自定义由群集默认使用的容器的名称，则必须使用“自定义创建”选项。

    > [WACOM.NOTE] 为 HDInsight 群集选择 Azure 存储帐户后，更改存储帐户的唯一方法是删除群集并使用所需的存储帐户创建新群集。

4.  单击新创建的群集。这将显示登录页：

    ![HDI.ClusterLanding][]

## 自定义 HDInsight 群集

HDInsight 使用各种 Hadoop 组件。有关已获得验证和支持的组件的列表，请参阅 [Azure HDInsight 包含哪个版本的 Hadoop？][]。可使用以下选项之一进行 HDInsight 自定义：

-   在群集设置期间使用 HDInsight .NET SDK 或 Azure PowerShell 中的群集自定义参数。通过这样做，这些配置更改将在群集的整个生存期内保留，并且不受 Azure 平台在维护时定期执行的群集节点重置映像影响。有关使用群集自定义参数的详细信息，请参阅[设置 HDInsight 群集][]。
-   一些本机 Java 组件（如 Mahout、Cascading）可以在群集上作为 JAR 文件运行。可以使用 Hadoop 作业提交机制将这些 JAR 文件分发到 Azure Blob 存储 (WASB)，并提交到 HDInsight 群集。有关详细信息，请参阅[以编程方式提交 Hadoop 作业][]。

    > [WACOM.NOTE] 如果你在将 jar 文件部署到 HDInsight 群集或调用 HDInsight 群集上的 jar 文件时遇到问题，请联系 [Microsoft 技术支持][]。

    > Mahout 和 Cascading 不受 HDInsight 支持，因此不符合 Microsoft 技术支持的条件。有关支持的组件的列表，请参阅 [HDInsight 提供的群集版本有哪些新功能？][Azure HDInsight 包含哪个版本的 Hadoop？]。

不支持使用远程桌面连接在群集上安装自定义软件。你应该避免在头节点的驱动器上存储任何文件，因为如果你需要重新创建群集，这些文件会丢失。建议你在 Azure Blob 存储中存储文件。Blob 存储是持久性的。

## 更改 HDInsight 群集用户名和密码

HDInsight 群集可以有两个用户帐户。HDInsight 群集用户帐户是在设置过程中创建的。你还可以创建通过 RDP 访问群集的 RDP 用户帐户。请参阅[启用远程桌面][]。

**更改 HDInsight 群集用户名和密码**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT” 。这将显示已部署的 HDInsight 群集的列表。
3.  单击要重设用户名和密码的 HDInsight 群集。
4.  在页面顶部，单击“配置” 。
5.  单击“HADOOP 服务” 旁边的“关闭” 。
6.  单击页面底部的“保存” ，然后等待禁用操作完成。
7.  在该服务已禁用后，单击“HADOOP 服务” 旁边的“打开” 。
8.  输入**用户名**和**新密码**。这些将是群集的新用户名和密码。
9.  单击“保存” 。

## 使用 RDP 连接到 HDInsight 群集

通过你在创建群集时提供的凭据，可以访问群集上的服务，但无法通过远程桌面访问群集本身。远程桌面访问默认情况下处于关闭状态，因此，使用它来直接访问群集时，要求一些其他的创建后配置。

**启用远程桌面**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT” 。这将显示已部署的 HDInsight 群集的列表。
3.  单击要连接到的 HDInsight 群集。
4.  在页面顶部，单击“配置” 。
5.  从页面底部，单击“启用远程” 。
6.  在“配置远程桌面” 向导中，输入远程桌面的用户名和密码。注意，此用户名必须不同于用来创建群集的用户名（默认情况下，指使用“快速创建”选项时的“admin”**）。在“截止日期” 框中输入到期日期。注意，到期日期必须为将来日期，但不超过从现在开始的一周。此日期的到期时间默认为指定日期的午夜。然后，单击复选图标。

    ![HDI.CreateRDPUser][]

    到期日期必须是将来的日期，且必须与现在最多相隔七天。时间为所选日期的午夜。

> [WACOM.NOTE] 为群集启用 RDP 后，必须刷新页面，然后才能连接到群集。

**使用 RDP 连接到群集**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT” 。这将显示已部署的 HDInsight 群集的列表。
3.  单击要连接到的 HDInsight 群集。
4.  在页面顶部，单击“配置” 。
5.  单击“连接” ，然后按照说明进行操作。

## 授予/撤消 HTTP 服务访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

-   ODBC
-   JDBC
-   Ambari
-   Oozie
-   Templeton

默认情况下，将授权这些服务进行访问。你可以从管理门户撤消/授予访问权限。

> [WACOM.NOTE] 授予/撤消访问权限时，你将重设群集用户的用户名和密码。

**授予/撤消 HTTP Web 服务访问权限**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT” 。这将显示已部署的 HDInsight 群集的列表。
3.  单击要配置的 HDInsight 群集。
4.  在页面顶部，单击“配置” 。
5.  单击“HADOOP 服务” 旁边的“打开” 或“关闭” 。
6.  输入**用户名**和**新密码**。这些将是群集的新用户名和密码。
7.  单击“保存” 。

也可以使用 Azure PowerShell cmdlet 来完成此操作：

-   Grant-AzureHDInsightHttpServicesAccess
-   Revoke-AzureHDInsightHttpServicesAccess

请参阅[使用 PowerShell 管理 HDInsight][]。

## 打开 Hadoop 命令行

若要使用远程桌面连接到群集并使用 Hadoop 命令行，首先必须对群集启用远程桌面访问，如上一部分中所述。

**打开 Hadoop 命令行**

1.  登录到 [Azure 管理门户][]。
2.  单击左窗格中的“HDINSIGHT” 。这将显示已部署的 Hadoop 群集的列表。
3.  单击要连接到的 HDInsight 群集。
4.  单击页面顶部的“配置” 。
5.  单击页面底部的“连接” 。
6.  单击“打开” 。
7.  输入你的凭据，然后单击“确定” 。使用在创建群集时配置的用户名和密码。
8.  单击“是” 。
9.  从桌面上双击“Hadoop 命令行” 。

    ![HDI.HadoopCommandLine][]

    有关 Hadoop 命令的详细信息，请参阅 [Hadoop 命令参考（可能为英文页面）][]。

在上面的屏幕快照中，文件夹名称嵌入了 Hadoop 版本号。版本号可以根据群集上安装的 Hadoop 组件的版本而更改。可以使用 Hadoop 环境变量来引用这些文件夹。例如：

    cd %hadoop_home%
    cd %hive_home%
    cd %pig_home%
    cd %sqoop_home%   
    cd %hcatalog_home%

## 后续步骤

在本文中，你学习了如何使用 Azure 管理门户创建 HDInsight 群集以及如何打开 Hadoop 命令行工具。若要了解更多信息，请参阅下列文章：

-   [使用 PowerShell 管理 HDInsight][]
-   [使用跨平台命令行界面管理 HDInsight][]
-   [配置 HDInsight 群集][设置 HDInsight 群集]
-   [以编程方式提交 Hadoop 作业][]
-   [Azure HDInsight 入门][]
-   [Azure HDInsight 包含哪个版本的 Hadoop？][]

  [使用 PowerShell 管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [使用跨平台命令行界面管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-command-line-interface/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: https://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [配置 HDInsight 群集]: #create
  [自定义 HDInsight 群集]: #customize
  [更改 HDInsight 群集用户名和密码]: #password
  [使用 RDP 连接到 HDInsight 群集]: #rdp
  [授予/撤消 HTTP 服务访问权限]: #httpservice
  [打开 Hadoop 命令控制台]: #hadoopcmd
  [后续步骤]: #nextsteps
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [如何创建存储帐户]: /en-us/manage/services/storage/how-to-create-a-storage-account/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [HDI.QuickCreate]: ./media/hdinsight-administer-use-management-portal/HDI.QuickCreateCluster.png
  [HDI.ClusterLanding]: ./media/hdinsight-administer-use-management-portal/HDI.ClusterLanding.PNG "群集登录页"
  [Azure HDInsight 包含哪个版本的 Hadoop？]: /en-us/manage/services/hdinsight/versioning-in-hdinsight/
  [以编程方式提交 Hadoop 作业]: /en-us/manage/services/hdinsight/submit-hadoop-jobs-programmatically/
  [Microsoft 技术支持]: http://www.windowsazure.cn/zh-cn/support/contact/
  [启用远程桌面]: #enablerdp
  [HDI.CreateRDPUser]: ./media/hdinsight-administer-use-management-portal/HDI.CreateRDPUser.png
  [HDI.HadoopCommandLine]: ./media/hdinsight-administer-use-management-portal/HDI.HadoopCommandLine.PNG "Hadoop 命令行"
  [Hadoop 命令参考（可能为英文页面）]: http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/CommandsManual.html
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
