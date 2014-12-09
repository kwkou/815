<properties linkid="manage-services-hdinsight-hbase-provision-on-vnet" urlDisplayName="Provision HBase clusters on Azure Virtual Network" pageTitle="在 Azure 虚拟网络上设置 HBase 群集 | Azure" metaKeywords="" description="Learn how to create HDInsight clusters on Azure Virtual Network." metaCanonical="" services="hdinsight" documentationCenter="" title="Provision HBase clusters on Azure Virtual Network" authors="jgao" solutions="big-data" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="08/21/2014" ms.author="jgao"></tags>

# 在 Azure 虚拟网络上设置 HBase 群集

了解如何在 Azure 虚拟网络上创建 HDInsight 群集。

通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。优点包括：

-   Web 应用程序直接连接到使用 HBase Java RPC API 实现通信的 HBase 群集的节点。
-   流量不必通过多个网关和负载平衡器，性能得到了提高。
-   以更安全的方式处理敏感信息，而不会公开公共终结点。

## 本文内容

-   [先决条件][]
-   [在虚拟网络中设置 HBase 群集][]
-   [使用 HBase Java RPC API 连接到虚拟网络中设置的 HBase 群集][]
-   [使用 PowerShell 设置 HBase 群集][]
-   [后续步骤][]

## <a id="prerequisites"></a> 先决条件

在开始阅读本教程前，你必须具有：

-   **Azure 订阅**。Azure 是基于订阅的平台。有关获取订阅的详细信息，请参阅[购买选项][]或[免费试用][]。

-   **已安装并已配置 Azure PowerShell 的工作站**。有关说明，请参阅[安装和配置 Azure PowerShell][]。若要执行 PowerShell 脚本，必须以管理员身份运行 Azure PowerShell 并将执行策略设为“RemoteSigned”。请参阅[运行 Windows PowerShell 脚本][]。

    在运行 PowerShell 脚本之前，请确保你已使用以下 cmdlet 连接到 Azure 订阅：

        Add-AzureAccount

    如果你有多个 Azure 订阅，请使用以下 cmdlet 来设置当前订阅：

        Select-AzureSubscription <AzureSubscriptionName>

## <a id="hbaseprovision"></a>在虚拟网络中设置 HBase 群集。

**要使用管理门户创建虚拟网络：**

1.  登录到 [Azure 管理门户][]。
2.  单击页面底部的“新建”，然后依次单击“网络服务”、“虚拟网络”和“快速创建”。
3.  键入或选择以下值：

    -   "名称"：虚拟网络的名称。
    -   "地址空间"：为虚拟网络选择一个地址空间，该空间必须足够大以便为群集中的所有节点提供地址。否则，设置将失败。
    -   "最大虚拟机数"：选择以下最大虚拟机数。
    -   "位置"：该位置必须与要创建的 HBase 群集相同。
    -   "DNS 服务器"：本文使用 Azure 提供的内部 DNS 服务器，因此，您可以选择"无"。此外，也支持使用自定义 DNS 服务器的高级网络配置。有关详细指南，请参见 [http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx](http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx)。

4.  单击“创建虚拟网络”。新虚拟网络名称将显示在列表中。等到“状态”列显示“已创建”。
5.  在主窗格中，单击刚创建的虚拟网络。
6.  在页面顶部，单击“仪表板”。
7.  在“速览”下，记住“虚拟网络 ID”。在设置 HBase 群集时将要用到它。
8.  在页面顶部，单击“配置”。
9.  在页面底部，默认子网名称为 **Subnet-1**。你可以选择重命名子网或者向 HBase 群集添加一个新子网。记下子网名称，设置群集时将会用到它。
10. 验证将用于该群集的子网的“CIDR（地址数）”。地址数必须大于工作节点数加上七（网关：2，头节点：2，ZooKeeper：3). 例如，如果需要一个 10 节点 HBase 群集，子网的地址数必须大于 17 (10+7)。否则，部署将失败。

    > [WACOM.NOTE] 强烈建议为一个群集指定一个子网。

11. 如果更新了子网值，请单击页面底部的“保存”。

> [WACOM.NOTE] HDInsight 群集使用 Azure Blob 存储来存储数据。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 中的 Hadoop 配合使用][]。你需要存储帐户和 Blob 存储容器。存储帐户位置必须与虚拟网络位置和群集位置匹配。

**要创建 Azure 存储帐户和 Blob 存储容器：**

1.  登录到 [Azure 管理门户][]。
2.  单击左下角的“新建”，指向“数据服务”，指向“存储”，然后单击“快速创建”。

3.  键入或选择以下值：

    -   "URL"：存储帐户的名称
    -   "位置"：存储帐户的位置。确保其与虚拟网络位置匹配。不支持地缘组。
    -   "复制"：出于测试目的，可使用“本地冗余”降低成本。

4.  单击**“创建存储帐户”**。你将在存储列表中看到新的存储帐户。
5.  等到新存储帐户的“状态”更改为“联机”。
6.  从列表中单击该新存储帐户以便选择它。
7.  单击页底部的“管理访问密钥”。
8.  记下**存储帐户名称**和**主访问密钥**（或**辅助访问密钥**，这两个密钥中的任一个均有效）。本教程后面的步骤中将会用到它们。
9.  在页面顶部，单击“容器”。
10. 在页面底部，单击“添加”。
11. 输入容器名称。该容器将用作 HBase 群集的默认容器。默认情况下，默认容器名称与群集名称匹配。保留“访问”字段为“私有”。
12. 单击复选标记以创建容器。

**要使用 Azure 门户设置 HBase 群集：**

> [WACOM.NOTE] 有关使用 PowerShell 设置新 HBase 群集的信息，请参阅[使用 PowerShell 设置 HBase 群集][]。

1.  登录到 [Azure 管理门户][]。
2.  单击左下角的“新建”，指向“数据服务”，指向“HDINSIGHT”，然后单击“自定义创建”。
3.  输入“群集名称”，并选择用于该群集的“HDINSIGHT 版本”。

    ![群集名称和版本字段][]

4.  选择要为群集创建的“数据节点”数，以及用于该群集的区域或 Azure 虚拟网络（“区域/虚拟网络”）。

    ![节点数和区域字段][]

5.  输入用于该群集的管理员“用户名”和“密码”。

    ![管理员名称和密码字段][]

6.  选择是使用新存储帐户还是现有存储帐户。如果使用新帐户，输入要创建的“帐户名称”和“默认容器”。最后，选中复选标记以创建群集。

    ![存储帐户选择][]

要开始处理新 HBase 群集，可以按照[开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][]中的步骤操作。

## <a id="connect"></a>使用 HBase Java RPC API 连接到虚拟网络中设置的 HBase 群集

1.  在相同的 Azure 虚拟网络和子网中设置 IaaS 虚拟机。因此，虚拟机和 HBase 群集使用相同的内部 DNS 服务器来解析主机名。为此，你必须选择“从库中”选项，然后选择虚拟网络而不是数据中心。有关说明，请参阅[创建运行 Windows Server 的虚拟机][]。具有小型虚拟机的标准 Windows Server 2012 映像已足够。

2.  获取 HBase 群集的连接特定 DNS 后缀。若要执行此操作，请将 RDP 连接到 HBase 群集（将连接到头节点），然后使用命令提示符运行 **ipconfig**。有关启用 RDP 并使用 RDP 连接到群集的说明，请参阅[使用 Azure 管理门户管理 HDInsight 中的 Hadoop 群集][]。

    ![hdinsight.hbase.dns.surffix][]

3.  更改虚拟机的主要 DNS 后缀配置。这样，虚拟机便可自动解析 HBase 群集的主机名，而不必显式指定后缀。例如，*workernode0* 主机名将被正确解析为 HBase 群集的工作节点 0。
    要进行配置更改：

    1.  RDP 连接到虚拟机。
    2.  打开“本地组策略编辑器”。可执行文件为 gpedit.msc。
    3.  依次展开“计算机配置”、“管理模板”和“网络”，然后单击“DNS 客户端”。。
    -   将“主 DNS 后缀”设置为在步骤 2 中获取的值：

        ![hdinsight.hbase.primary.dns.suffix][]
    1.  单击“确定”。
    2.  重新启动虚拟机。

    现在，虚拟机可以随时与 HBase 群集进行通信。要测试连接，请在虚拟机上运行 “ping headnode0”。

## <a id="powershell"></a>使用 Azure PowerShell 设置 HBase 群集：

1.  打开 PowerShell ISE。
2.  将以下命令复制并粘贴到脚本窗格。

        $hbaseClusterName = "<HBaseClusterName>"
        $hadoopUserName = "<HBaseClusterUsername>"
        $hadoopUserPassword = "<HBaseClusterUserPassword>"
        $location = "<HBaseClusterLocation>"  #i.e. "China East"
        $clusterSize = <HBaseClusterSize>  
        $vnetID = "<AzureVirtualNetworkID>"
        $subNetName = "<AzureVirtualNetworkSubNetName>"
        $storageAccountName = "<AzureStorageAccountName>" # Do not use the full name here
        $storageAccountKey = "<AzureStorageAccountKey>"
        $storageContainerName = "<AzureBlobStorageContainer>"

        $password = ConvertTo-SecureString $hadoopUserPassword -AsPlainText -Force
        $creds = New-Object System.Management.Automation.PSCredential ($hadoopUserName, $password) 

        New-AzureHDInsightCluster -Name $hbaseClusterName `
                                  -ClusterType HBase `
                                  -Version 3.1 `
                                  -Location $location `
                                  -ClusterSizeInNodes $clusterSize `
                                  -Credential $creds `
                                  -VirtualNetworkId $vnetID `
                                  -SubnetName $subNetName `
                                  -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" `
                                  -DefaultStorageAccountKey $storageAccountKey `
                                  -DefaultStorageContainerName $storageContainerName

3.  单击“运行脚本”或按 **F5**。
4.  要验证该群集，可以在管理门户中检查该群集，也可以在底部窗格中运行以下 PowerShell cmdlet：

    Get-AzureHDInsightCluster

## <a id="nextsteps"></a> 后续步骤

在本教程中，我们已经学习了如何设置 HBase 群集。若要了解更多信息，请参阅以下文章：

-   [HDInsight 入门][]
-   [在 HDInsight 中设置 Hadoop 群集][]
-   [开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][]
-   [虚拟网络概述][]。

  [先决条件]: #prerequisites
  [在虚拟网络中设置 HBase 群集]: #hbaseprovision
  [使用 HBase Java RPC API 连接到虚拟网络中设置的 HBase 群集]: #connect
  [使用 PowerShell 设置 HBase 群集]: #powershell
  [后续步骤]: #nextsteps
<<<<<<< HEAD
  [购买选项]: http://www.windowsazure.cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/pricing/1rmb-trial/
  [安装和配置 Azure PowerShell]: /zh-cn/documentation/articles/install-configure-powershell
  [运行 Windows PowerShell 脚本]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn
=======
  [购买选项]: /pricing/overview/
  [免费试用]: /pricing/1rmb-trial/
  [安装和配置 Azure PowerShell]: /zh-cn/documentation/articles/install-configure-powershell
  [运行 Windows PowerShell 脚本]: http://technet.microsoft.com/zh-cn/library/ee176949.aspx
  [Azure 管理门户]: https://manage.windowsazure.cn
  []: http://msdn.microsoft.com/zh-cn/library/azure/jj156088.aspx
>>>>>>> stage
  [将 Azure Blob 存储与 HDInsight 中的 Hadoop 配合使用]: /zh-cn/documentation/articles/hdinsight-use-blob-storage/
  [群集名称和版本字段]: ./media/hdinsight-hbase-provision-vnet/hbasewizard1.png
  [节点数和区域字段]: ./media/hdinsight-hbase-provision-vnet/hbasewizard2.png
  [管理员名称和密码字段]: ./media/hdinsight-hbase-provision-vnet/hbasewizard3.png
  [存储帐户选择]: ./media/hdinsight-hbase-provision-vnet/hbasewizard4.png
  [开始在 HDInsight 中将 HBase 与 Hadoop 配合使用]: /zh-cn/documentation/articles/hdinsight-hbase-get-started/
  [创建运行 Windows Server 的虚拟机]: /zh-cn/documentation/articles/virtual-machines-windows-tutorial/
  [使用 Azure 管理门户管理 HDInsight 中的 Hadoop 群集]: /zh-cn/documentation/articles/hdinsight-administer-use-management-portal/#rdp
  [hdinsight.hbase.dns.surffix]: ./media/hdinsight-hbase-provision-vnet/DNSSuffix.png
  [hdinsight.hbase.primary.dns.suffix]: ./media/hdinsight-hbase-provision-vnet/PrimaryDNSSuffix.png
  [HDInsight 入门]: /zh-cn/documentation/articles/hdinsight-get-started/
  [在 HDInsight 中设置 Hadoop 群集]: /zh-cn/documentation/articles/hdinsight-provision-clusters/
  [在 HDInsight 中使用 HBase 分析 Twitter 传感器数据]: /zh-cn/documentation/articles/hdinsight-hbase-twitter-sentiment/
  [虚拟网络概述]: http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx
