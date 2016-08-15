<properties
pageTitle="使用共享访问签名限制对数据进行 HDInsight 访问"
description="了解如何使用共享访问签名限制对 Azure 存储 Blob 中存储的数据进行 HDInsight 访问。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="04/19/2016"
	wacn.date="06/29/2016"/>

#使用 Azure 存储空间共享访问签名来限制使用 HDInsight 访问数据

HDInsight 使用 Azure 存储空间 Blob 来存储数据。HDInsight 必须对用作群集默认存储的 Blob 拥有完全访问权限，但你可以限制其他群集所用 Blob 中存储的数据的访问权限。例如，你可能想要将一些数据设为只读。为此，可以使用共享访问签名。

共享访问签名 (SAS) 是可用于限制数据访问权限的一项 Azure 存储帐户功能。例如，它可以提供对数据的只读访问。在本文档中，你将了解如何使用 SAS 启用对 HDInsight 中 Blob 容器的读取和仅限列出访问权限。

##要求

* Azure 订阅

* C# 或 Python。已提供 C# 示例代码作为 Visual Studio 解决方案。

    * Visual Studio 的版本必须是 2013 或 2015。
    
    * Python 的版本必须是 2.7 或更高

* [Azure PowerShell][powershell] - 可以使用 Azure PowerShell 创建新群集，并在创建群集期间添加共享访问签名。

* [https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature) 中的示例文件。此存储库包含以下项目：

    * Visual Studio 项目，可以创建存储容器、存储策略，以及配合 HDInsight 使用的 SAS
    
    * Python 脚本，可以创建存储容器、存储策略，以及配合 HDInsight 使用的 SAS
    
    * PowerShell 脚本，可以创建新的 HDInsight 群集并将其配置为使用 SAS。

##共享访问签名

共享访问签名有两种形式：

* 即席：在你创建一个临时 SAS 时，针对该 SAS 的开始时间、到期时间和权限全都在 SAS URI 上指定（在省略开始时间的情况下，也可以是隐式的）。

* 存储访问策略：存储访问策略是对资源容器（Blob 容器、表、队列或文件共享）定义的，可用于管理针对一个或多个共享访问签名的约束。在你将某一 SAS 与一个存储访问策略相关联时，该 SAS 将继承对该存储访问策略定义的约束：开始时间、到期时间和权限。

这两种形式之间的差异对于一个关键情形而言十分重要：吊销。一个 SAS 就是一个 URL，因此，获取该 SAS 的任何人都可以使用它，而与是谁请求它以便开始的无关。如果某一 SAS 是公开发布的，则世界上的任何人都可以使用它。在发生以下四种情况之一前分发的 SAS 是有效的：

1. 达到了对该 SAS 指定的到期时间。

2. 达到了对该 SAS 引用的存储访问策略指定的到期时间（如果引用某一存储访问策略并且该存储访问策略指定一个到期时间）。这可能是因为经过了该间隔而发生，或者是因为你修改了该存储访问策略以使到期时间已经是过去时间而发生（这是用于吊销该 SAS 的一种方法）。

3. 删除了该 SAS 引用的存储访问策略，这是用于吊销 SAS 的另一种方法。请注意，如果你使用完全相同的名称重新创建该存储访问策略，则根据与该存储访问策略相关联的权限，所有现有 SAS 标记都将再次有效（假定尚未经过该 SAS 的到期时间）。如果你想要吊销该 SAS，请确保使用不同时间（如果你使用将来的到期时间重新创建该访问策略）。

4. 将重新生成用于创建 SAS 的帐户密钥。请注意，这样做将导致使用该帐户密钥的所有应用程序组件身份验证失败，直到这些组件更新为使用其他有效帐户密钥或者重新生成的新帐户密钥。

> [AZURE.IMPORTANT] 共享访问签名 URI 与用于创建签名的帐户密钥和关联的存储访问策略（如果有）相关联。如果未指定存储访问策略，则吊销共享访问签名的唯一方法是更改帐户密钥。

建议始终使用存储访问策略，以便可以根据需要吊销签名或延长过期日期。本文档中的步骤使用存储访问策略生成 SAS。

有关共享访问签名的详细信息，请参阅[了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。

##创建存储策略并生成 SAS

目前必须以编程方式创建存储策略。可在以下位置找到创建存储策略和 SAS 的 C# 与 Python 示例：[https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature)。

###使用 C# 创建存储策略和 SAS

1. 在 Visual Studio 中打开解决方案。

2. 在解决方案资源管理器中，右键单击“SASToken”项目并选择“属性”。

3. 选择“设置”，并添加以下条目的值：

    * StorageConnectionString：想要为其创建存储策略和 SAS 的存储帐户的连接字符串。其格式应为 `DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey`，其中 `myaccount` 是存储帐户名称，`mykey` 是存储帐户密钥。
    
    * ContainerName：想要限制访问的存储帐户中的容器。
    
    * SASPolicyName：要创建的存储策略所用的名称。
    
    * FileToUpload：将上载到容器的文件的路径。
    
4. 运行该项目。将显示控制台窗口。生成 SAS 之后，将显示如下所示的信息：

        Container SAS token using stored access policy: sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14
        
    保存 SAS 策略令牌，因为在将存储帐户关联到 HDInsight 群集时需要用到此信息。你还需要使用存储帐户名称和容器名称。
    
###使用 Python 创建存储策略和 SAS

1. 打开 SASToken.py 文件并更改以下值：

    * policy\_name：要创建的存储策略所用的名称。
    
    * storage\_account\_name：存储帐户的名称。
    
    * storage\_account\_key：存储帐户的密钥。
    
    * storage\_container\_name：想要限制访问的存储帐户中的容器。
    
    * example\_file\_path：将上载到容器的文件的路径。

2. 运行该脚本。脚本完成后，将显示如下所示的 SAS 令牌：

        sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14
    
    保存 SAS 策略令牌，因为在将存储帐户关联到 HDInsight 群集时需要用到此信息。你还需要使用存储帐户名称和容器名称。
    
##配合 HDInsight 使用 SAS

创建 HDInsight 群集时，必须指定主存储帐户，可以选择性地指定其他存储帐户。这两种添加存储的方法都需要对所用存储帐户和容器拥有完全访问权限。

若要使用共享访问签名来限制对容器的访问，必须将一个自定义条目添加到群集的 __core-site__ 配置中。

* 对于__基于 Windows__ 的 HDInsight 群集，可以使用 PowerShell 在创建群集期间执行此操作。

###创建使用 SAS 的新群集

存储库的 `CreateCluster` 目录中包含创建使用 SAS 的 HDInsight 群集的示例。若要使用该示例，请执行以下步骤：

1. 在文本编辑器中打开 `CreateCluster\HDInsightSAS.ps1` 文件，然后修改位于文档开头的以下值。

        # Replace 'mycluster' with the name of the cluster to be created
        $clusterName = 'mycluster'
        # Valid value is 'Windows'
        $osType = 'Windows'
        # Replace with the Azure data center you want to the cluster to live in
        $location = 'China North'
        # Replace with the name of the default storage account to be created
        $defaultStorageAccountName = 'mystorageaccount'
        # Replace with the name of the SAS container created earlier
        $SASContainerName = 'sascontainer'
        # Replace with the name of the SAS storage account created earlier
        $SASStorageAccountName = 'sasaccount'
        # Replace with the SAS token generated earlier
        $SASToken = 'sastoken'
        # Set the number of worker nodes in the cluster
        $clusterSizeInNodes = 2
        
    例如，将 `'mycluster'` 更改为要创建的群集的名称。创建存储帐户和 SAS 令牌时，SAS 值应该与先前步骤中的值匹配。
    
    更改值之后，请保存该文件。

1. 打开新的 Azure PowerShell 提示符。如果你不熟悉或尚未安装 Azure PowerShell，请参阅[安装和配置 Azure PowerShell][powershell]。

2. 在提示符下使用以下命令向 Azure 订阅进行身份验证：

        Add-AzureAccount -Environment AzureChinaCloud
    
    出现提示时，请用 Azure 订阅帐户登录。
    
    如果你的登录名与多个 Azure 订阅关联，你可能需要使用 `Select-AzureSubscription` 来选择想要使用的订阅。

2. 在提示符下，将目录切换到包含 HDInsightSAS.ps1 文件的 `CreateCluster` 目录。然后使用以下命令以运行该脚本
        
        .\HDInsightSAS.ps1
    
    脚本运行后，会在创建资源组和存储帐户时将输出记录到 PowerShell 命令提示符下。然后它会提示你输入 HDInsight 群集的 HTTP 用户。这是用于保护群集的 HTTP/s 访问的用户帐户。

    > [AZURE.IMPORTANT] 出现输入 HTTP/s 用户名和密码的提示时，必须提供符合以下条件的密码：
    >
    > - 长度必须至少为 10 个字符
    > - 必须至少包含一个数字
    > - 必须至少包含一个非字母数字字符
    > - 必须至少包含一个大写或小写字母


需要等待一段时间让此脚本完成，通常大约是 15 分钟。如果脚本完成且没有发生任何错误，则会创建群集。

##测试限制的访问

若要验证已限制的访问，请使用以下方法：

* 对于__基于 Windows__ 的 HDInsight 群集，请使用远程桌面连接到群集。有关详细信息，请参阅[使用 RDP 连接到 HDInsight](/documentation/articles/hdinsight-administer-use-management-portal-v1/#connect-to-clusters-using-rdp)。

    连接之后，请使用桌面上的“Hadoop 命令行”图标打开命令提示符。

连接到群集后，使用以下步骤验证是否只能读取和列出 SAS 存储帐户中的项：

1. 在提示符下键入以下命令。将 __SASCONTAINER__ 替换为针对 SAS 存储帐户创建的容器名称。将 __SASACCOUNTNAME__ 替换为用于 SAS 的存储帐户名称：

        hdfs dfs -ls wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.chinacloudapi.cn/
    
    这会列出容器的内容，其中应包含创建容器和 SAS 时上载的文件。
    
2. 使用以下命令验证是否可以读取该文件的内容。如同上一步骤替换 __SASCONTAINER__ 和 __SASACCOUNTNAME__。将 __FILENAME__ 替换为前一个命令中显示的名称：

        hdfs dfs -text wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.chinacloudapi.cn/FILENAME
        
    这会列出文件的内容。
    
3. 使用以下命令将文件下载到本地文件系统：

        hdfs dfs -get wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.chinacloudapi.cn/FILENAME testfile.txt
    
    这会将该文件下载到名为 __testfile.txt__ 的本地文件中。

4. 使用以下命令将本地文件上载到 SAS 存储上名为 __testupload.txt__ 的新文件中：

        hdfs dfs -put testfile.txt wasb://SASCONTAINER@SASACCOUNTNAME.blob.core.chinacloudapi.cn/testupload.txt
    
    你将收到类似于下面的消息：
    
        put: java.io.IOException
        
    发生此错误的原因是存储位置是只读+仅限列出的。使用以下命令将数据放在群集的可写默认存储中：
    
        hdfs dfs -put testfile.txt wasb:///testupload.txt
        
    这一次操作应会成功完成。
    
##故障排除

###任务已取消

__症状__：使用 PowerShell 脚本创建群集时，你可能会收到以下错误消息：

    New-AzureHDInsightCluster : A task was canceled.
    At C:\Users\larryfr\Documents\GitHub\hdinsight-azure-storage-sas\CreateCluster\HDInsightSAS.ps1:62 char:5
    +     New-AzureHDInsightCluster `
    +     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : NotSpecified: (:) [New-AzureHDInsightCluster], CloudException
        + FullyQualifiedErrorId : Hyak.Common.CloudException,Microsoft.Azure.Commands.HDInsight.NewAzureHDInsightClusterCommand

__原因__：如果你使用群集管理员/HTTP 用户的密码，则可能发生此错误。

__解决方法__：使用符合以下条件的密码：

- 长度必须至少为 10 个字符
- 必须至少包含一个数字
- 必须至少包含一个非字母数字字符
- 必须至少包含一个大写或小写字母

##后续步骤

现在你已了解如何将访问受限的存储添加到 HDInsight 群集，接下来请了解在群集上处理数据的其他方法：

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[powershell]: /documentation/articles/powershell-install-configure/

<!---HONumber=Mooncake_0215_2016-->