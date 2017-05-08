<properties
    pageTitle="在 HDInsight 中将 Hadoop Hive 与 PowerShell 配合使用 | Azure"
    description="使用 PowerShell 在 HDInsight 上的 Hadoop 中运行 Hive 查询。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="cb795b7c-bcd0-497a-a7f0-8ed18ef49195"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="03/21/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="d3ea7c7326124efaea63bd745037da185996ad07"
    ms.lasthandoff="04/28/2017" />

# <a name="run-hive-queries-using-powershell"></a>使用 PowerShell 运行 Hive 查询
[AZURE.INCLUDE [hive-selector](../../includes/hdinsight-selector-use-hive.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文档提供使用 Azure 资源组模式中的 Azure PowerShell 在 HDInsight 群集上的 Hadoop 中运行 Hive 查询的示例。

> [AZURE.NOTE]
> 本文档未详细描述示例中使用的 HiveQL 语句的作用。 有关此示例中使用的 HiveQL 的信息，请参阅[将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)。

**先决条件**

* **Azure HDInsight 群集**：无论该群集是基于 Windows 还是基于 Linux 都行。

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.IMPORTANT]
    > Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* **配备 Azure PowerShell 的工作站**。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

## <a id="powershell"></a> 使用 Azure PowerShell 运行 Hive 查询

Azure PowerShell 提供 *cmdlet* ，可让你在 HDInsight 上远程运行 Hive 查询。 cmdlet 在内部对 HDInsight 群集上的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) 进行 REST 调用。

在远程 HDInsight 群集上运行 Hive 查询时，将使用以下 Cmdlet：

* **Add-AzureRmAccount**：在 Azure 订阅中进行 Azure PowerShell 身份验证
* **New-AzureRmHDInsightHiveJobDefinition**：使用指定的 HiveQL 语句创建作业定义
* **Start-AzureRmHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业*对象
* **Wait-AzureRmHDInsightJob**：使用作业对象来检查作业的状态。 它等到作业完成或超出等待时间。
* **Get-AzureRmHDInsightJobOutput**：用于检索作业的输出
* **Invoke-AzureRmHDInsightHiveJob**：用于运行 HiveQL 语句。 此 cmdlet 将阻止查询完成，然后返回结果
* **Use-AzureRmHDInsightCluster**：设置要用于 **Invoke-AzureRmHDInsightHiveJob** 命令的当前群集

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业：

1. 使用编辑器将以下代码保存为 **hivejob.ps1**。

        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        #Get cluster info
        $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
        $creds=Get-Credential -Message "Enter the login for the cluster"

        #HiveQL
        #Note: set hive.execution.engine=tez; is not required for
        #      Linux-based HDInsight
        $queryString = "set hive.execution.engine=tez;" +
                    "DROP TABLE log4jLogs;" +
                    "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasbs:///example/data/';" +
                    "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"

        #Create an HDInsight Hive job definition
        $hiveJobDefinition = New-AzureRmHDInsightHiveJobDefinition -Query $queryString 

        #Submit the job to the cluster
        Write-Host "Start the Hive job..." -ForegroundColor Green

        $hiveJob = Start-AzureRmHDInsightJob -ClusterName $clusterName -JobDefinition $hiveJobDefinition -ClusterCredential $creds

        #Wait for the Hive job to complete
        Write-Host "Wait for the job to complete..." -ForegroundColor Green
        Wait-AzureRmHDInsightJob -ClusterName $clusterName -JobId $hiveJob.JobId -ClusterCredential $creds

        # Print the output
        Write-Host "Display the standard output..." -ForegroundColor Green
        Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $hiveJob.JobId `
            -HttpCredential $creds

2. 打开一个新的 **Azure PowerShell** 命令提示符。 将目录更改为 **hivejob.ps1** 文件的所在位置，然后使用以下命令来运行脚本：

        .\hivejob.ps1

    脚本运行时，系统将提示输入群集名称和该群集的 HTTPS/管理员帐户凭据。 可能还会提示登录到 Azure 订阅。

3. 作业完成时，它会返回类似以下文本的信息：

        Display the standard output...
        2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
        2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
        2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id

4. 如前所述，**Invoke-Hive** 可以用来运行查询，并等待响应。 使用以下脚本查看 Invoke-Hive 的工作原理：

        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        #Get cluster info
        $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
        $creds=Get-Credential -Message "Enter the login for the cluster"

        # Set the cluster to use
        Use-AzureRmHDInsightCluster -ClusterName $clusterName -HttpCredential $creds

        $queryString = "set hive.execution.engine=tez;" +
                    "DROP TABLE log4jLogs;" +
                    "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/';" +
                    "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"
        Invoke-AzureRmHDInsightHiveJob `
            -StatusFolder "statusout" `
            -Query $queryString

    输出类似于以下文本：

        2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
        2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
        2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id

    > [AZURE.NOTE]
    > 对于较长的 HiveQL 查询，可以使用 Azure PowerShell **Here-Strings** cmdlet 或 HiveQL 脚本文件。 以下代码段显示了如何使用 **Invoke-Hive** cmdlet 来运行 HiveQL 脚本文件。 必须将 HiveQL 脚本文件上载到 wasb://。
    > <p>
    > `Invoke-AzureRmHDInsightHiveJob -File "wasbs://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
    > <p>
    > 有关 **Here-Strings** 的详细信息，请参阅<a href="http://technet.microsoft.com/zh-cn/library/ee692792.aspx" target="_blank">使用 Windows PowerShell Here-Strings</a>。

## <a name="troubleshooting"></a>故障排除

如果在作业完成时未返回任何信息，则可能表示处理期间发生错误。 若要查看此作业的错误信息，请将以下内容添加到 **hivejob.ps1** 文件的末尾，保存，然后重新运行该文件。

    # Print the output of the Hive job.
    Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

运行作业时，此 cmdlet 返回写入到服务器上的 STDERR 中的信息。

## <a name="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询，监视作业状态，以及检索输出。

## <a name="next-steps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Pig 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)