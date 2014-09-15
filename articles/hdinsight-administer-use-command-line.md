<properties linkid="manage-services-hdinsight-administer-hdinsight-using-command-line" urlDisplayName="HDInsight Administration" pageTitle="Administer HDInsight using using the Cross-Platform Command-Line Interface | Azure" metaKeywords="hdinsight, hdinsight administration, hdinsight administration azure" description="Learn how to use the Cross-Platform Command-Line Interface to manage HDInsight clusters on any platform that supports Node.js, including Windows, Mac, and Linux." services="hdinsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" title="Administer HDInsight using the Cross-platform Command-line Interface" authors="jgao" />

# 使用跨平台命令行界面管理 HDInsight

在本文中，你将了解如何使用跨平台命令行界面管理 HDInsight 群集。该命令行工具是在 Node.js 中实现的。可以在支持 Node.js 的任意平台（包括 Windows、Mac 和 Linux）上使用它。

该命令行工具是开源的。在 GitHub 中管理源代码（网址为 <https://github.com/WindowsAzure/azure-sdk-tools-xplat>）。

本文只涉及从 Windows 使用命令行界面。有关如何使用命令行界面的一般指南，请参阅[如何使用针对 Mac 和 Linux 的 Azure 命令行工具][]。有关完整的参考文档，请参阅[针对 Mac 和 Linux 的 Azure 命令行工具][]。

**先决条件：**

在开始阅读本文前，你必须具有：

-   **Azure 订阅**。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅[购买选项][]、[成员优惠][购买选项]或[免费试用][]。

## 本文内容

-   [安装][]
-   [下载和导入 Azure 帐户 publishsettings][]
-   [设置群集][]
-   [使用配置文件设置群集][]
-   [列出并显示群集][]
-   [删除群集][]
-   [后续步骤][]

## 安装

可以使用 Node.js 包管理器 (NPM)** 或 Windows 安装程序安装该命令行界面。

**使用 NPM 安装该命令行界面**

1.  浏览到**www.nodejs.org**。
2.  单击**安装**，然后使用默认设置按说明操作。
3.  从工作站打开**命令提示符**（或*Azure 命令提示符*，或*VS2012 的开发人员命令提示符*）。
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

1.  浏览到 **<http://www.windowsazure.cn/zh-cn/downloads/#cmd-line-tools>**。
2.  向下滚动到“命令行工具” 部分，然后单击“跨平台命令行界面” ，按 Web 平台安装程序向导的要求操作。

## 下载和导入 Azure 帐户 publishsettings

在使用命令行界面前，你必须配置工作站和 Azure 之间的连接。命令行界面使用你的 Azure 订阅信息连接到你的帐户。可从 Azure 的 publishsettings 文件中获取此信息。然后，可以导入 publishsettings 文件作为永久性本地配置设置，命令行界面会将此设置用于后续操作。你只需导入你的 publishsettings 一次。

> [WACOM.NOTE] publishsettings 文件包含敏感信息。建议你删除该文件或采取其他措施来加密包含该文件的用户文件夹。在 Windows 上，修改文件夹属性或使用 BitLocker。

**下载和导入 publishsettings**

1.  打开“命令提示符” 。
2.  运行以下命令来下载 publishsettings 文件。

        azure account download

    ![HDI.CLIAccountDownloadImport][]

    该命令显示下载文件的说明，包括 URL。

3.  打开 **Internet Explorer** 并浏览到命令行提示符窗口中所列的 URL。
4.  单击**保存**以将文件保存到工作站。
5.  从命令提示符窗口，运行以下命令以导入 publishsettings 文件：

        azure account import <文件>

    在上一屏幕快照中，publishsettings 文件已保存到工作站上的 C:\\HDInsight 文件夹。

## 设置 HDInsight 群集

HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户，然后才能创建 HDInsight 群集。

在导入了 publishsettings 文件后，你可以使用以下命令创建一个存储帐户：

    azure account storage create [options] <StorageAccountName>

> [WACOM.NOTE] 存储帐户必须共置于同一数据中心。目前，只能在以下数据中心内设置 HDInsight 群集：

> -   亚洲东南部
> -   欧洲北部
> -   欧洲西部
> -   美国东部
> -   美国西部

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
        

准备好存储帐户和 blob 容器后，你就可以创建群集了：

    azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName <StorageAccountName> --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

![HDI.CLIClusterCreation][]

## 使用配置文件设置 HDInsight 群集

通常，你设置一个 HDInsight 群集，对其运行作业，然后删除该群集以降低成本。在命令行界面上，你可以选择将配置保存到文件，以便在每次设置群集时重用这些配置。

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

## 列出并显示群集详细信息

使用以下命令来列出和显示群集详细信息：

    azure hdinsight cluster list
    azure hdinsight cluster show <ClusterName>

![HDI.CLIListCluster][]

## 删除群集

使用以下命令来删除群集：

    azure hdinsight cluster delete <ClusterName>

## 后续步骤

在本文中，你已了解如何执行不同的 HDInsight 群集管理任务。若要了解更多信息，请参阅下列文章：

-   [使用管理门户管理 HDInsight][]
-   [使用 PowerShell 管理 HDInsight][]
-   [Azure HDInsight 入门][]
-   [如何使用针对 Mac 和 Linux 的 Azure 命令行工具][]
-   [针对 Mac 和 Linux 的 Azure 命令行工具][]

  [如何使用针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/develop/nodejs/how-to-guides/command-line-tools/
  [针对 Mac 和 Linux 的 Azure 命令行工具]: /en-us/manage/linux/other-resources/command-line-tools/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [安装]: #installation
  [下载和导入 Azure 帐户 publishsettings]: #importsettings
  [设置群集]: #provision
  [使用配置文件设置群集]: #provisionconfigfile
  [列出并显示群集]: #listshow
  [删除群集]: #delete
  [后续步骤]: #nextsteps
  [HDI.CLIAccountDownloadImport]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
  [如何创建存储帐户]: /en-us/manage/services/storage/how-to-create-a-storage-account/
  [如何管理存储帐户]: /en-us/manage/services/storage/how-to-manage-a-storage-account/
  [HDI.CLIClusterCreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
  [HDI.CLIClusterCreationConfig]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
  [HDI.CLIListCluster]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "列出并显示群集"
  [使用管理门户管理 HDInsight]: /en-us/manage/services/hdinsight/howto-administer-hdinsight/
  [使用 PowerShell 管理 HDInsight]: /en-us/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
