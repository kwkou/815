<properties
    pageTitle="在虚拟网络中创建 HBase 群集 | Azure"
    description="开始在 Azure HDInsight 中使用 HBase。了解如何在 Azure 虚拟网络上创建 HDInsight HBase 群集。"
    keywords=""
    services="hdinsight,virtual-network"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="8de8e446-f818-4e61-8fad-e9d38421e80d"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/22/2017"
    wacn.date="04/27/2017"
    ms.author="jgao" />

# 在 Azure 虚拟网络中创建 HBase 群集
了解如何在 [Azure 虚拟网络][1]中创建 Azure HDInsight HBase 群集。

通过虚拟网络集成，可以将 HBase 群集部署到应用程序所在的虚拟网络，以便应用程序直接与 HBase 进行通信。优点包括：

* 将 Web 应用程序直接连接到 HBase 群集节点，以通过 HBase Java 远程过程调用 (RPC) API 实现通信。
* 提高性能，因为流量不必通过多个网关和负载均衡器。
* 能够以更安全的方式处理敏感信息，而无需公开公共终结点。

### 先决条件
要阅读本教程，必须具备：

* **Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 在虚拟网络上创建 HBase 群集
在本部分中，通过 [Azure Resource Manager 模板](/documentation/articles/resource-group-template-deploy/)在 Azure 虚拟网络中使用从属 Azure 存储帐户创建基于 Linux 的 HBase 群集。对于其他群集创建方法以及了解设置，请参阅 [Create HDInsight clusters](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)（创建 HDInsight 群集）。有关使用模板在 HDInsight 中创建 Hadoop 群集的详细信息，请参阅[使用 Azure Resource Manager 模板在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)

> [AZURE.NOTE]
某些属性已在模板中硬编码。例如：
><p>
><p> * **位置**：中国东部
><p> * **群集版本**：3.5
><p> * **群集辅助角色节点计数**：4
><p> * **默认存储帐户**：唯一字符串
><p> * **虚拟网络名称**：<群集名称>-vnet
><p> * **虚拟网络地址空间**：10.0.0.0/16
><p> * **子网名称**：subnet1
><p> * **子网地址范围**：10.0.0.0/24
><p>
> \<群集名称\> 会替换为使用模板时提供的群集名称。
>
>

1. 单击下面的图像可在 Azure 门户预览中打开模板。模板位于 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-hbase-linux-vnet/)中。

    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux-vnet%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-hbase-provision-vnet/deploy-to-azure.png" alt="Deploy to Azure"></a>

    >[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；把允许的地域改成“China North”和“China East”；把 HDInsight Linux 版本改为 Azure 中国所支持的 3.5。

2. 在“自定义部署”边栏选项卡中输入以下项：

    * **订阅**：选择用于创建 HDInsight 群集、从属存储帐户和 Azure 虚拟网络的 Azure 订阅。
    * **资源组**：选择“新建”，并指定新资源组名称。
    * **位置**：选择资源组的位置。
    * **ClusterName**：为要创建的 Hadoop 群集输入名称。
    * **群集登录名和密码**：默认登录名是 **admin**。
    * **SSH 用户名和密码**：默认用户名是 **sshuser**。可以将它重命名。
    * **我同意以述条款和条件**：（选择）
3. 单击“购买”。创建群集大约需要 20 分钟时间。创建群集之后，便可以在门户中单击群集边栏选项卡以打开它。

完成教程之后，可能要删除群集。有了 HDInsight，可将数据存储在 Azure 存储空间，以便在不使用群集时可将其安全删除。此外，还需要支付 HDInsight 群集费用，即使未使用。由于群集费用高于存储空间费用数倍，因此在不使用群集时将其删除可以节省费用。有关删除群集的说明，请参阅[使用 Azure 门户预览在 HDInsight 中管理 Hadoop 群集](/documentation/articles/hdinsight-administer-use-management-portal/#delete-clusters)。

要开始处理新 HBase 群集，可以按照[开始在 HDInsight 中将 HBase 与 Hadoop 配合使用](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)中的步骤操作。

## 使用 HBase Java RPC API 连接到 HBase 群集。
1. 将基础结构即服务 (IaaS) 虚拟机创建到相同的 Azure 虚拟网络和子网中。有关创建新 IaaS 虚拟机的说明，请参阅[创建运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)。按照本文档中的步骤操作时，必须使用以下内容进行网络配置：

    * **虚拟网络**：<群集名称>-vnet
    * **子网**：subnet1

    > [AZURE.IMPORTANT]
    使用在之前步骤中创建 HDInsight 群集时使用的名称替换<群集名称>。
    >
    >

    使用这些值可将虚拟机放置在与 HDInsight 群集相同的虚拟网络和子网中。此配置让它们能够直接相互通信。有一种方法可使用空的边缘节点创建 HDInsight 群集。该边缘节点可用于管理群集。有关详细信息，请参阅 [Use empty edge nodes in HDInsight](/documentation/articles/hdinsight-apps-use-edge-node/)（在 HDInsight 中使用空边缘节点）。

2. 使用 Java 应用程序远程连接到 HBase 时，必须使用完全限定域名 (FQDN)。要确定这一点，你必须获取 HBase 群集的连接特定的 DNS 后缀。为此，可以使用以下方法之一：

    * 使用 Web 浏览器发出 Ambari 调用：

        浏览到 https://&lt;ClusterName>.azurehdinsight.cn/api/v1/clusters/<ClusterName>/hosts?minimal\_response=true。随后将返回带有 DNS 后缀的 JSON 文件。
    * 使用 Ambari 网站：

        1. 浏览到 https://&lt;ClusterName>.azurehdinsight.cn。
        2. 在顶部菜单中单击**主机**。
    * 使用 Curl 发出 REST 调用：

            curl -u <username>:<password> -k https://<clustername>.azurehdinsight.cn/ambari/api/v1/clusters/<clustername>.azurehdinsight.cn/services/hbase/components/hbrest

        在返回的 JavaScript 对象表示法 (JSON) 数据中，找到“host\_name”条目。此条目包含群集中的节点的 FQDN。例如：

            ...
            "host_name": "wordkernode0.<clustername>.b1.chinacloudapp.cn
            ...

        以群集名称开头的域名的部分是 DNS 后缀。例如，mycluster.b1.chinacloudapp.cn。
    * 使用 Azure PowerShell

        使用以下 Azure PowerShell 脚本注册 **Get-ClusterDetail** 函数，该函数可用于返回 DNS 后缀：

            function Get-ClusterDetail(
                [String]
                [Parameter( Position=0, Mandatory=$true )]
                $ClusterDnsName,
                [String]
                [Parameter( Position=1, Mandatory=$true )]
                $Username,
                [String]
                [Parameter( Position=2, Mandatory=$true )]
                $Password,
                [String]
                [Parameter( Position=3, Mandatory=$true )]
                $PropertyName
                )
            {
            <#
                .SYNOPSIS
                 Displays information to facilitate an HDInsight cluster-to-cluster scenario within the same virtual network.
                .Description
                 This command shows the following 4 properties of an HDInsight cluster:
                 1. ZookeeperQuorum (supports only HBase type cluster)
                    Shows the value of HBase property "hbase.zookeeper.quorum".
                 2. ZookeeperClientPort (supports only HBase type cluster)
                    Shows the value of HBase property "hbase.zookeeper.property.clientPort".
                 3. HBaseRestServers (supports only HBase type cluster)
                    Shows a list of host FQDNs that run the HBase REST server.
                 4. FQDNSuffix (supports all cluster types)
                    Shows the FQDN suffix of hosts in the cluster.
                .EXAMPLE
                 Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName ZookeeperQuorum
                 This command shows the value of HBase property "hbase.zookeeper.quorum".
                .EXAMPLE
                 Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName ZookeeperClientPort
                 This command shows the value of HBase property "hbase.zookeeper.property.clientPort".
                .EXAMPLE
                 Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName HBaseRestServers
                 This command shows a list of host FQDNs that run the HBase REST server.
                .EXAMPLE
                 Get-ClusterDetail -ClusterDnsName {clusterDnsName} -Username {username} -Password {password} -PropertyName FQDNSuffix
                 This command shows the FQDN suffix of hosts in the cluster.
            #>

                $DnsSuffix = ".azurehdinsight.cn"

                $ClusterFQDN = $ClusterDnsName + $DnsSuffix
                $webclient = new-object System.Net.WebClient
                $webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)

                if($PropertyName -eq "ZookeeperQuorum")
                {
                    $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum"
                    $Response = $webclient.DownloadString($Url)
                    $JsonObject = $Response | ConvertFrom-Json
                    Write-host $JsonObject.items[0].properties.'hbase.zookeeper.quorum'
                }
                if($PropertyName -eq "ZookeeperClientPort")
                {
                    $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.property.clientPort"
                    $Response = $webclient.DownloadString($Url)
                    $JsonObject = $Response | ConvertFrom-Json
                    Write-host $JsonObject.items[0].properties.'hbase.zookeeper.property.clientPort'
                }
                if($PropertyName -eq "HBaseRestServers")
                {
                    $Url1 = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.rest.port"
                    $Response1 = $webclient.DownloadString($Url1)
                    $JsonObject1 = $Response1 | ConvertFrom-Json
                    $PortNumber = $JsonObject1.items[0].properties.'hbase.rest.port'

                    $Url2 = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/hbase/components/hbrest"
                    $Response2 = $webclient.DownloadString($Url2)
                    $JsonObject2 = $Response2 | ConvertFrom-Json
                    foreach ($host_component in $JsonObject2.host_components)
                    {
                        $ConnectionString = $host_component.HostRoles.host_name + ":" + $PortNumber
                        Write-host $ConnectionString
                    }
                }
                if($PropertyName -eq "FQDNSuffix")
                {
                    $Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/home/features/YARN/components/RESOURCEMANAGER"
                    $Response = $webclient.DownloadString($Url)
                    $JsonObject = $Response | ConvertFrom-Json
                    $FQDN = $JsonObject.host_components[0].HostRoles.host_name
                    $pos = $FQDN.IndexOf(".")
                    $Suffix = $FQDN.Substring($pos + 1)
                    Write-host $Suffix
                }
            }

        运行 Azure PowerShell 脚本后，使用以下命令通过 **Get-ClusterDetail** 函数来返回 DNS 后缀。使用此命令时，指定你的 HDInsight HBase 群集名称、管理员名称和管理员密码。

            Get-ClusterDetail -ClusterDnsName <yourclustername> -PropertyName FQDNSuffix -Username <clusteradmin> -Password <clusteradminpassword>

        这将返回 DNS 后缀。例如，**yourclustername.b4.internal.chinacloudapp.cn**。
    * 使用 RDP

        还可以使用远程桌面来连接 HBase 群集（将连接到头节点），并从命令提示符运行 **ipconfig** 来获取 DNS 后缀。有关启用远程桌面协议 (RDP) 并使用 RDP 连接到群集的说明，请参阅[使用 Azure 门户预览在 HDInsight 中管理 Hadoop 群集][hdinsight-admin-portal]。

        ![hdinsight.hbase.dns.surffix][img-dns-surffix]

<!--
3.    Change the primary DNS suffix configuration of the virtual machine. This enables the virtual machine to automatically resolve the host name of the HBase cluster without explicit specification of the suffix. For example, the *workernode0* host name will be correctly resolved to workernode0 of the HBase cluster.

    To make the configuration change:

    1. RDP into the virtual machine.
    2. Open **Local Group Policy Editor**. The executable is gpedit.msc.
    3. Expand **Computer Configuration**, expand **Administrative Templates**, expand **Network**, and then click **DNS Client**.
    - Set **Primary DNS Suffix** to the value obtained in step 2:

        ![hdinsight.hbase.primary.dns.suffix][img-primary-dns-suffix]
    4. Click **OK**.
    5. Reboot the virtual machine.
-->

要验证虚拟机是否可与 HBase 群集进行通信，请从虚拟机使用 `ping headnode0.<dns suffix>` 命令。例如，ping headnode0.mycluster.b1.chinacloudapp.cn。

要在 Java 应用程序中使用此信息，可以按照[使用 Maven 构建将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序](/documentation/articles/hdinsight-hbase-build-java-maven/)中的步骤来创建应用程序。要让应用程序连接到远程 HBase 服务器，请修改本示例中的 **hbase-site.xml** 文件，以对 Zookeeper 使用 FQDN。例如：

    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>zookeeper0.<dns suffix>,zookeeper1.<dns suffix>,zookeeper2.<dns suffix></value>
    </property>

> [AZURE.NOTE]
有关 Azure 虚拟网络中的名称解析的详细信息，包括如何使用自己的 DNS 服务器，请参阅[名称解析 (DNS)](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)。
>
>

## 后续步骤
在本教程中，你已学习了如何创建 HBase 群集。要了解更多信息，请参阅以下文章：

* [开始使用 HDInsight](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)
* [在 HDInsight 中使用空边缘节点](/documentation/articles/hdinsight-apps-use-edge-node/)
* [在 HDInsight 中配置 HBase 复制](/documentation/articles/hdinsight-hbase-replication/)
* [在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)
* [开始在 HDInsight 中将 HBase 与 Hadoop 配合使用](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/)
* [虚拟网络概述][vnet-overview]

[1]: /home/features/networking/
[2]: http://technet.microsoft.com/zh-cn/library/ee176961.aspx
[3]: http://technet.microsoft.com/zh-cn/library/hh847889.aspx

[hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-linux/
[vnet-overview]: /documentation/articles/virtual-networks-overview/
[vm-create]: /documentation/articles/virtual-machines-windows-hero-tutorial/

[azure-portal]: https://portal.azure.cn
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal/#connect-to-clusters-using-rdp

[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx


[twitter-streaming-api]: https://dev.twitter.com/docs/streaming-apis
[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter


[powershell-install]: /documentation/articles/powershell-install-configure/


[hdinsight-customize-cluster]: /documentation/articles/hdinsight-hadoop-customize-cluster-v1/
[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-storage-powershell]: /documentation/articles/hdinsight-hadoop-use-blob-storage/#powershell
[hdinsight-analyze-flight-delay-data]: /documentation/articles/hdinsight-analyze-flight-delay-data/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/
[hdinsight-hive-odbc]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/

[img-dns-surffix]: ./media/hdinsight-hbase-provision-vnet-v1/DNSSuffix.png
[img-primary-dns-suffix]: ./media/hdinsight-hbase-provision-vnet-v1/PrimaryDNSSuffix.png
[img-provision-cluster-page1]: ./media/hdinsight-hbase-provision-vnet-v1/hbasewizard1.png "预配新 HBase 群集的详细信息"
[img-provision-cluster-page5]: ./media/hdinsight-hbase-provision-vnet-v1/hbasewizard5.png "使用脚本操作来自定义 HBase 群集"

[azure-preview-portal]: https://portal.azure.cn

<!---HONumber=Mooncake_1205_2016-->