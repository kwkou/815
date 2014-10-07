<properties linkid="manage-services-hdinsight-provision-hdinsight-clusters" urlDisplayName="HDInsight Administration" pageTitle="Provision HDInsight clusters | Azure" metaKeywords="hdinsight, hdinsight administration, hdinsight administration azure" description="Learn how to provision clusters for Azure HDInsight using the management portal, PowerShell, or the command line." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="Provision HDInsight clusters" authors="jgao" />

# 配置 HDInsight 群集

在本文中，你将了解设置 HDInsight 群集所用的不同方法。

**必备条件：**

在开始阅读本文前，你必须具有：

-   Azure 订阅。Azure 是基于订阅的平台。HDInsight PowerShell cmdlet 通过订阅执行任务。有关获取订阅的详细信息，请参阅[购买选项][]、[免费试用][]。

## 本文内容

-   [使用 Azure 管理门户][]
-   [使用 Azure PowerShell][]
-   [使用跨平台命令行][]
-   [使用 HDInsight .NET SDK][]
-   [后续步骤][]

## 使用 Azure 管理门户

HDInsight 群集使用 Azure Blob 存储容器作为默认文件系统。创建 HDInsight 群集前，要先具有位于同一数据中心的 Azure 存储帐户。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。有关创建 Azure 存储帐户的详细信息，请参阅[如何创建存储帐户][]。

> [WACOM.NOTE] 目前，只有中国东部和中国北部地区可以承载 HDInsight 群集。

本课介绍使用“自定义创建”选项创建 HDInsight 群集的过程。有关使用“快速创建”选项的信息，请参阅 [Azure HDInsight 入门][]。

**使用“自定义创建”选项创建 HDInsight 群集**

1.  登录到 [Azure 管理门户][]。
2.  单击页面底部的“+ 新建' ，然后依次单击“数据服务” 、“HDINSIGHT” 和“自定义创建” 。
3.  从“群集详细信息”页上，键入或选择以下值：

	<table border='1'>
	<tr><th>属性</th><th>值</th></tr>
	<tr><td>群集名称</td>
	<td><p>命名群集。 </p>
	            <ul>
	<li>DNS 名称必须以字母数字开头和结尾，且可包含短划线。</li>
	<li>字段必须是介于 3 到 63 个字符之间的字符串。</li>
	            </ul></td></tr>
	<tr><td>数据节点</td>
	<td>指定群集中节点的数量。默认值为 4。</td></tr>
	<tr><td>HDInsight 版本</td>
	<td>3.0 版使用 Hadoop 2.2 群集。有关详细信息，请参阅 <a href="http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-component-versioning/">Azure HDInsight 包含哪个版本的 Hadoop？</a>。</td></tr>
	<tr><td>区域</td>
	<td>指定在其中安装群集的数据中心。该位置必须与用作默认文件系统的 Azure Blob 存储所在位置相同。目前你可以选择 *中国北部* 或 *中国东部*。</td>
	    </tr>
	</table>

    ![HDI.CustomProvision.Page1][]

4.  单击页面右下角的向右箭头。
5.  从“配置群集用户”页上，键入或选择以下值：

	<table border='1'>
	<tr><th>属性</th><th>值</th></tr>
	<tr><td>用户名</td>
	<td>指定 HDInsight 群集用户名。</td></tr>
	<tr><td><p>密码</p><p>确认密码</p></td>
	<td>指定 HDInsight 群集用户密码。</td></tr>
	<tr><td>输入配置单元/Oozie 元存储</td>
	<td>在同一数据中心指定要作为配置单元/Oozie 元存储的 SQL 数据库。</td></tr>
	<tr><td>配置单元/Oozie 元存储数据库</td>
	<td>指定将用作 Hive/OOzie 的元存储的 Azure SQL 数据库。此 SQL 数据库必须与 HDInsight 群集位于同一数据中心。该列表框只列出你在&ldquo;群集详细信息&rdquo;页面指定的同一数据中心的 SQL 数据库。</td></tr>
	<tr><td>数据库用户</td>
	<td>指定将用于连接到数据库的 SQL 数据库用户。</td></tr>
	<tr><td>数据库用户密码</td>
	<td>指定 SQL 数据库用户密码。</td></tr>
	</table>

    
6.  单击页面右下角的向右箭头。
7.  从“存储帐户”页上，键入或选择以下值：

	<table border='1'>
	<tr><th>属性</th><th>值</th></tr>
	<tr><td>存储帐户</td>
	<td>为 HDInsight 群集指定将用作默认文件系统的 Azure 存储帐户。可以选择以下三个选项之一：
	        <ul>
	<li>使用现有存储</li>
	<li>创建新存储</li>
	<li>使用其他订阅中的存储</li>
	        </ul>
	        </td></tr>
	<tr><td>帐户名称</td>
	<td>在选中了&ldquo;使用现有存储&rdquo;<b></b>时，该列表框只列出位于同一数据中心的存储帐户。</td></tr>
	<tr><td>帐户密钥</td>
	<td>此字段只在选中了&ldquo;存储帐户&rdquo;字段中的&ldquo;使用其他订阅中的存储&rdquo;<strong></strong>时可用。</td></tr>  
	<tr><td>默认容器</td>
	<td>存储帐户上的默认容器将用作 HDInsight 群集的默认文件系统。在&ldquo;存储帐户&rdquo;字段中选择&ldquo;使用现有存储&rdquo;<strong></strong>且在&ldquo;默认容器&rdquo;字段中选择&ldquo;创建默认容器&rdquo;<strong></strong>时，默认容器名称与群集名称相同。如果具有群集名称的容器已存在，将在容器名称后追加一个序列号。例如，mycontainer1、mycontainer2，等等。</td></tr>
	<tr><td>其他存储帐户</td>
	<td>HDInsight 支持多个存储帐户。一个群集可以使用的其他存储帐户数没有限制。但是，如果你使用管理门户创建群集，由于 UI 限制你将发现这一数目限制为 7。在此字段中指定的每个其他存储帐户将添加一个&ldquo;存储帐户&rdquo;页，以便你在此指定帐户信息。例如，在下面的屏幕快照中，选择了 1 个附加的存储帐户，第 4 页添加到了对话框。</td></tr>      
	</table>

    ![HDI.CustomProvision.Page3][]

8.  单击页面右下角的向右箭头。
9.  在“存储帐户”页上，输入其他存储帐户的帐户信息：

    ![HDI.CustomProvision.Page4][]

10. 单击右下角的“完成”按钮（选中图标）以启动 HDInsight 群集设置过程。

设置一个群集可能需要几分钟时间。设置过程成功完成后，该群集的状态栏将显示“正在运行” 。

> [WACOM.NOTE] 一旦为 HDInsight 群集选择了 Azure 存储帐户，就不能再删除该帐户，也不能将它更改为另一帐户。

## 使用 Azure PowerShell

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。有关配置工作站运行 HDInsight Powershell cmdlet 的信息，请参阅[安装和配置 Azure PowerShell][]。有关将 PowerShell 用于 HDInsight 的更多信息，请参见[使用 PowerShell 管理 HDInsight][]。有关 HDInsight PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][]。

使用 PowerShell 设置 HDInsight 群集的步骤如下：

-   创建 Azure 存储帐户
-   创建 Azure Blob 容器
-   创建 HDInsight 群集

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。存储帐户必须与 HDInsight 群集位于同一数据中心。

**创建 Azure 存储帐户**

-   在 Azure PowerShell 控制台窗口中运行以下命令：

        $storageAccountName = "<StorageAcccountName>"
        $location = "<MicrosoftDataCenter>"     # 例如，"China East"

        # 创建 Azure 存储帐户
        New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location

    如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下 PowerShell 命令来检索该信息：

        # 列出当前订阅的存储帐户
        Get-AzureStorageAccount

        # 列出存储帐户的密钥
        Get-AzureStorageKey "<StorageAccountName>"

**创建 Azure 存储容器**

-   在 Azure PowerShell 窗口中运行以下命令：

        $storageAccountName = "<StorageAccountName>"
        $storageAccountKey = "<StorageAccountKey>"
        $containerName="<ContainerName>"

        # 创建存储上下文对象
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName 
        -StorageAccountKey $storageAccountKey  

        # 创建 Blob 存储容器
        New-AzureStorageContainer -Name $containerName -Context $destContext

准备好存储帐户和 blob 容器后，你就可以创建群集了。

**设置 HDInsight 群集**

-   在 Azure PowerShell 窗口中运行以下命令：

        $subscriptionName = "<SubscriptionName>"        # Azure 订阅的名称。
        $storageAccountName = "<StorageAccountName>"    # 承载该默认容器的 Azure 存储帐户。默认容器将被用作默认文件系统。
        $containerName = "<ContainerName>"              # 将被用作 HDInsight 群集的默认文件系统的 Azure Blob 存储容器。

        $clusterName = "<HDInsightClusterName>"         # HDInsight 群集将要采用的名称。
        $location = "<MicrosoftDataCenter>"             # HDInsight 群集的位置。它必须与存储帐户位于同一数据中心。
        $clusterNodes = <ClusterSizeInNodes>            # HDInsight 群集中的节点数。

        # 基于帐户名称获取存储主键
        Select-AzureSubscription $subscriptionName
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }

        # 新建 HDInsight 群集
        New-AzureHDInsightCluster -Name $clusterName -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes

    设置群集可能需要几分钟时间。

    ![HDI.CLI.Provision][]

你也可以设置群集，并将它配置为连接到多个 Azure Blob 存储或自定义的 Hive 元存储和 Oozie 元存储。利用这项高级功能，你可以将数据和元数据的生存期与群集的生存期分开。

**使用配置设置 HDInsight 群集**

-   从 Windows PowerShell 窗口运行以下命令：

        $subscriptionName = "<SubscriptionName>"
        $clusterName = "<ClusterName>"
        $location = "<MicrosoftDataCenter>"
        $clusterNodes = <ClusterSizeInNodes>

        $storageAccountName_Default = "<DefaultFileSystemStorageAccountName>"
        $containerName_Default = "<DefaultFileSystemContainerName>"

        $storageAccountName_Add1 = "<AdditionalStorageAccountName>"

        $hiveSQLDatabaseServerName = "<SQLDatabaseServerNameForHiveMetastore>"
        $hiveSQLDatabaseName = "<SQLDatabaseDatabaseNameForHiveMetastore>"
        $oozieSQLDatabaseServerName = "<SQLDatabaseServerNameForOozieMetastore>"
        $oozieSQLDatabaseName = "<SQLDatabaseDatabaseNameForOozieMetastore>"

        # 获取存储帐户密钥
        Select-AzureSubscription $subscriptionName
        $storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
        $storageAccountKey_Add1 = Get-AzureStorageKey $storageAccountName_Add1 | %{ $_.Primary }

        $oozieCreds = Get-Credential -Message "Oozie metastore"
        $hiveCreds = Get-Credential -Message "Hive metastore"

        # 创建 Blob 存储容器
        #$dest1Context = New-AzureStorageContext -StorageAccountName $storageAccountName_Default -StorageAccountKey $storageAccountKey_Default  
        #New-AzureStorageContainer -Name $containerName_Default -Context $dest1Context

        # 新建 HDInsight 群集
        $config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
        Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
        Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Add1.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Add1 |
        Add-AzureHDInsightMetastore -SqlAzureServerName "$hiveSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $hiveSQLDatabaseName -Credential $hiveCreds -MetastoreType HiveMetastore |
        Add-AzureHDInsightMetastore -SqlAzureServerName "$oozieSQLDatabaseServerName.database.chinacloudapi.cn" -DatabaseName $oozieSQLDatabaseName -Credential $oozieCreds -MetastoreType OozieMetastore |
        New-AzureHDInsightCluster -Name $clusterName -Location $location

**列出 HDInsight 群集**

-   在 Azure PowerShell 窗口中运行以下命令：

        Get-AzureHDInsightCluster -Name <ClusterName>

## 使用跨平台命令行

设置 HDInsight 群集的另一方法是使用跨平台命令行界面。该命令行工具是在 Node.js 中实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。该命令行工具是开源的。在 GitHub 中管理源代码（网址为 <https://github.com/WindowsAzure/azure-sdk-tools-xplat>）。有关如何使用命令行界面的一般指南，请参阅[如何使用针对 Mac 和 Linux 的 Azure 命令行工具][]。有关完整的参考文档，请参阅[针对 Mac 和 Linux 的 Azure 命令行工具][]。本文只涉及从 Windows 使用命令行界面。

使用跨平台命令行设置 HDInsight 群集的步骤如下：

-   安装跨平台命令行
-   下载并导入 Azure 帐户发布设置
-   创建 Azure 存储帐户
-   设置群集

可以使用 Node.js 包管理器 (NPM)** 或 Windows 安装程序安装该命令行界面。

**使用 NPM 安装该命令行界面**

1.  浏览到“www.nodejs.org” 。
2.  单击“安装” ，然后使用默认设置按说明操作。
3.  从工作站打开“命令提示符” （或“Azure 命令提示符”**，或“VS2012 的开发人员命令提示符”**）。
4.  在命令提示符窗口中运行以下命令。

        npm install -g azure-cli

    > [WACOM.NOTE] 如果收到“未找到 NPM 命令”的错误消息，请验证以下路径是否在 PATH 环境变量中：*C:\\Program Files (x86)\\nodejs;C:\\Users[username]\\AppData\\Roaming\\npm* 或 *C:\\Program Files\\nodejs;C:\\Users[username]\\AppData\\Roaming\\npm*

5.  运行以下命令以验证安装：

        azure hdinsight -h

    可以在不同级别使用 *-h* 开关以显示帮助信息。例如：

        azure -h
        azure hdinsight -h
        azure hdinsight cluster -h
        azure hdinsight cluster create -h

**使用 Windows 安装程序安装该命令行界面**

1.  浏览到 **<http://www.windowsazure.cn/zh-cn/downloads/>**。
2.  向下滚动到“命令行工具” 部分，然后单击“跨平台命令行界面” ，按 Web 平台安装程序向导的要求操作。

在使用命令行界面前，你必须配置工作站和 Azure 之间的连接。命令行界面使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 的发布设置文件中获取此信息。稍后可以导入发布设置文件作为永久性本地配置设置，命令行界面会将此设置用于后续操作。你只需导入你的发布设置一次。

> [WACOM.NOTE] 发布设置文件包含敏感信息。建议你删除该文件或采取其他措施来加密包含该文件的用户文件夹。在 Windows 上，修改文件夹属性或使用 BitLocker。

**下载和导入发布设置**

1.  打开“命令提示符” 。
2.  运行以下命令来下载发布设置文件。

        azure account download

    ![HDI.CLIAccountDownloadImport][]

    该命令显示下载文件的说明，包括 URL。

3.  打开 Internet Explorer  并浏览到命令行提示符窗口中所列的 URL。
4.  单击“保存” 以将文件保存到工作站。
5.  从命令提示符窗口，运行以下命令以导入发布设置文件：

        azure account import <文件>

    在上一屏幕快照中，发布设置文件已保存到工作站上的 C:\\HDInsight 文件夹。

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。存储帐户必须位于同一数据中心。

**创建 Azure 存储帐户**

-   从命令提示符窗口，运行以下命令：

        azure account storage create [options] <StorageAccountName>

有关使用 Azure 管理门户创建 Azure 存储帐户的信息，请参阅[如何创建存储帐户][]。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下命令来检索该信息：

    -- 列出存储帐户
    azure account storage list
    -- 显示存储帐户
    azure account storage show <StorageAccountName>
    -- 列出存储帐户的密钥
    azure account storage keys list <StorageAccountName>

有关使用管理门户获取信息的详细信息，请参阅[如何管理存储帐户][]中的*如何：查看、复制和重新生成存储访问密钥*一节。

如果容器不存在，可使用 *azure hdinsight cluster create* 命令创建它。如果选择预先创建容器，可以使用以下命令：

    azure storage container create --account-name <StorageAccountName> --account-key <StorageAccountKey> [ContainerName]
        

准备好存储帐户和 blob 容器后，你就可以创建群集了。

**创建 HDInsight 群集**

-   从命令提示符窗口，运行以下命令：

        azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName <StorageAccountName> --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

    ![HDI.CLIClusterCreation][]

通常，你设置 HDInsight 群集，运行它们的作业，然后删除该群集以降低成本。在命令行界面上，你可以选择将配置保存到文件，以便在每次设置群集时重用这些配置。

**使用配置文件设置 HDInsight 群集**

-   从命令提示符窗口，运行以下命令：

        azure hdinsight cluster config create <file>

        azure hdinsight cluster config set <file> --clusterName <ClusterName> --nodes <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --username "<Username>" --clusterPassword "<UserPassword>"

        azure hdinsight cluster config storage add <file> --storageAccountName "<StorageAccountName>.blob.core.chinacloudapi.cn"
        --storageAccountKey "<StorageAccountKey>"

        azure hdinsight cluster config metastore set <file> --type "hive" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
        --database "<HiveDatabaseName>" --user "<Username>" --metastorePassword "<UserPassword>"

        azure hdinsight cluster config metastore set <file> --type "oozie" --server "<SQLDatabaseName>.database.chinacloudapi.cn"
        --database "<OozieDatabaseName>" --user "<SQLUsername>" --metastorePassword "<SQLPassword>"

        azure hdinsight cluster create --config <file>

    ![HDI.CLIClusterCreationConfig][]

**列出和显示群集详细信息**

-   使用以下命令来列出和显示群集详细信息：

        azure hdinsight cluster list
        azure hdinsight cluster show <ClusterName>

    ![HDI.CLIListCluster][]

**删除群集**

-   使用以下命令来删除群集：

        azure hdinsight cluster delete <ClusterName>

## 使用 HDInsight .NET SDK

HDInsight .NET SDK 提供了一组 .NET 客户端库，使你能够在 .NET 中更轻松地使用 HDInsight。

使用 SDK 设置 HDInsight 群集的步骤如下：

-   安装 HDInsight .NET SDK
-   创建控制台应用程序
-   运行应用程序

**安装 HDInsight .NET SDK**
 可以从 [NuGet][] 安装 SDK 最新发布的版本。下一过程中将显示说明。

**创建 Visual Studio 控制台应用程序**

1.  打开 Visual Studio 2012。

2.  在“文件”菜单中，单击“新建” ，然后单击“项目” 。

3.  从“新建项目”中，键入或选择以下值：

<table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
<tr>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">属性</th>
<th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">值</th></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">类别</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">模板/Visual C#/Windows</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">模板</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">控制台应用程序</td></tr>
<tr>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">名称</td>
<td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">CreateHDICluster</td></tr>
</table>

4.  单击“确定” 以创建该项目。

5.  从“工具”菜单中 ，单击“库程序包管理器” ，然后单击“程序包管理器控制台” 。

6.  在控制台中运行下列命令以安装程序包。

        Install-Package Microsoft.WindowsAzure.Management.HDInsight

    此命令将 .NET 库以及对这些库的引用添加到当前 Visual Studio 项目中。

7.  从解决方案资源管理器中，双击 **Program.cs** 将其打开。

8.  将下列 using 语句添加到文件顶部：

        using System.Security.Cryptography.X509Certificates;
        using Microsoft.WindowsAzure.Management.HDInsight;
        using Microsoft.WindowsAzure.Management.HDInsight.ClusterProvisioning;

9.  在 Main() 函数中，复制并粘贴以下代码：

        string certfriendlyname = "<CertificateFriendlyName>";
        string subscriptionid = "<AzureSubscriptionID>";
        string clustername = "<HDInsightClusterName>";
        string location = "<MicrosoftDataCenter>";
        string storageaccountname = "<AzureStorageAccountName>";     // 必须使用完整路径
        string storageaccountkey = "<AzureStorageAccountKey>";
        string containername = "<HDInsightDefaultContainerName>";
        string username = "<HDInsightUsername>";
        string password = "<HDInsightUserPassword>";
        int clustersize = <NumberOfNodesInTheCluster>;

        // 通过使用友好名称进行标识从证书存储区获取证书对象
        X509Store store = new X509Store();
        store.Open(OpenFlags.ReadOnly);
        X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certfriendlyname);

        // 创建存储帐户（如果不存在）。

        // 创建容器（如果不存在）。

        // 创建 HDInsightClient 对象
        HDInsightCertificateCredential creds = new HDInsightCertificateCredential(new Guid(subscriptionid), cert);
        var client = HDInsightClient.Connect(creds);

        // 提供群集信息
        ClusterCreateParameters clusterInfo = new ClusterCreateParameters()
        {
        Name = clustername,
        Location = location,
        DefaultStorageAccountName = storageaccountname,
        DefaultStorageAccountKey = storageaccountkey,
        DefaultStorageContainer = containername,
        UserName = username,
        Password = password,
        ClusterSizeInNodes = clustersize
        };

        // 创建群集
        Console.WriteLine("Creating the HDInsight cluster ...");

        ClusterDetails cluster = client.CreateCluster(clusterInfo);

        Console.WriteLine("Created cluster:{0}.", cluster.ConnectionUrl);
        Console.WriteLine("Press ENTER to continue.");
        Console.ReadKey();

10. 替换 main() 函数开头的变量。

**运行应用程序**

该应用程序在 Visual Studio 中打开时，按 **F5** 键以运行该应用程序。控制台窗口应打开并显示应用程序的状态。设置一个 HDInsight 群集可能需要几分钟时间。

## 后续步骤

在本文中，你已经学习了几种设置 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][]
-   [使用 PowerShell 管理 HDInsight][]
-   [以编程方式提交 Hadoop 作业][]
-   [Azure HDInsight SDK 文档][]

  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [使用 Azure 管理门户]: #portal
  [使用 Azure PowerShell]: #powershell
  [使用跨平台命令行]: #cli
  [使用 HDInsight .NET SDK]: #sdk
  [后续步骤]: #nextsteps
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [如何创建存储帐户]: /en-us/manage/services/storage/how-to-create-a-storage-account/
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [Azure HDInsight 包含哪个版本的 Hadoop？]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-component-versioning/
  [HDI.CustomProvision.Page1]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page1.png
  [HDI.CustomProvision.Page2]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page2.png
  [HDI.CustomProvision.Page3]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page3.png
  [HDI.CustomProvision.Page4]: ./media/hdinsight-provision-clusters/HDI.CustomProvision.Page4.png
  [安装和配置 Azure PowerShell]: /zh-cn/documentation/articles/install-configure-powershell/
  [使用 PowerShell 管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [HDInsight cmdlet 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dn479228.aspx
  [HDI.CLI.Provision]: ./media/hdinsight-provision-clusters/HDI.ps.provision.png
  [如何使用针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/develop/nodejs/how-to-guides/command-line-tools/
  [针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/manage/linux/other-resources/command-line-tools/
  [HDI.CLIAccountDownloadImport]: ./media/hdinsight-provision-clusters/HDI.CLIAccountDownloadImport.png
  [如何管理存储帐户]: /en-us/manage/services/storage/how-to-manage-a-storage-account/
  [HDI.CLIClusterCreation]: ./media/hdinsight-provision-clusters/HDI.CLIClusterCreation.png
  [HDI.CLIClusterCreationConfig]: ./media/hdinsight-provision-clusters/HDI.CLIClusterCreationConfig.png
  [HDI.CLIListCluster]: ./media/hdinsight-provision-clusters/HDI.CLIListClusters.png "列出并显示群集"
  [NuGet]: http://nuget.codeplex.com/wikipage?title=Getting%20Started
  [以编程方式提交 Hadoop 作业]: /en-us/manage/services/hdinsight/submit-hadoop-jobs-programmatically/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
