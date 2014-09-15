<properties linkid="manage-services-hdinsight-howto-pig" urlDisplayName="Use Pig with HDInsight" pageTitle="Use Pig with HDInsight | Azure" metaKeywords="" description="Learn how to use Pig with HDInsight. Write Pig Latin statements to analyze an application log file, and run queries on the data to generate output for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Use Pig with HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# Pig 与 HDInsight 配合使用

学习如何在 HDInsight 上运行 [Apache Pig][] 作业以分析 Apache log4j 日志文件。

估计完成时间： 30 分钟

## 本文内容

-   [Pig 用例][]
-   [先决条件][]
-   [了解 Pig Latin][]
-   [使用 PowerShell 提交 Pig 作业][]
-   [使用 HDInsight .NET SDK 提交 Pig 作业][]
-   [后续步骤][]

## Pig 用例

数据库对于小数据集和低延迟查询非常有用。但对于大型数据和大型数据集（以 TB 为单位）而言，传统的 SQL 数据库并不是理想的解决方案。随着数据库负载增加和性能下降，数据库管理员以前往往需要采购更大的硬件。

一般而言，所有应用程序都会在日志文件中保存错误、异常和其他有代码的问题，这样管理员就可以检查问题或根据日志文件数据生成某些指标。这些日志文件通常都很大，包含着必须处理和挖掘的丰富数据。

日志文件就是大数据的典型范例。很难使用关系数据库和统计信息/可视化包来处理大型数据。由于存在大量数据并需要对这些数据进行计算，在几十、几百甚至几千台服务器上运行的并行软件通常需要在合理的时间内计算此数据。Hadoop 提供了一个 MapReduce 框架，用于编写以非常可靠且容错的方式并行处理大型计算机群集间大量结构化和非结构化数据的应用程序。

[Apache *Pig*][Apache Pig] 提供了一种脚本语言来执行 *MapReduce* 作业以替代编写 Java 代码的方法。Pig 的脚本编写语言被称为 *Pig Latin*。Pig Latin 语句遵循这样的一般流程：

-   **加载**：从文件系统中读取要操作的数据
-   **转换**：操作该数据
-   **转储或存储**：将数据输出到屏幕或存储区以便进行处理

使用 Pig 可减少编写映射器和化简器程序所需的时间。这意味着，不需要 Java，也不需要 boilerplate 代码。你还可以灵活地将 Java 代码与 Pig 结合。用不到五行人可以直接读懂的 Pig 代码就可以编写许多复杂算法。

下面两幅图直观地显示了你在本文中要完成的任务。这些图显示了一个具有代表性的数据集示例，旨在阐释当你运行脚本中的所有 Pig 代码行时的数据流和数据转换。第一张图显示的是 log4j 文件示例：

![整个文件示例][]

第二个图显示了数据转换：

![HDI.PIG.Data.Transformation][]

有关 Pig Latin 的详细信息，请参阅 [Pig Latin 参考手册 1][] 和 [Pig Latin 参考手册 2][]。

## 先决条件

在开始阅读本文前，请注意以下要求：

-   一个 Azure HDInsight 群集。有关说明，请参阅 [Azure HDInsight 入门][]或[设置 HDInsight 群集][]。你将需要以下数据才能完成本教程：

	<table border = "1">
	<tr><th>群集属性</th><th>PowerShell 变量名</th><th>值</th><th>说明</th></tr>
	<tr><td>HDInsight 群集名称</td><td>$clusterName</td><td></td><td>要在其中运行本教程的 HDInsight 群集。</td></tr>
	</table>

-   安装和配置 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。

**了解 HDInsight 存储**

HDInsight 将 Azure Blob 存储用于数据存储。它称为 *WASB* 或 *Azure 存储空间 - Blob*。WASB 是 Microsoft 在 Azure Blob 存储上的 HDFS 实现。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

设置 HDInsight 群集时，请将 Azure 存储帐户和该帐户上的特定 Blob 存储容器指定为默认文件系统，就像在 HDFS 中一样。除了此存储帐户外，在设置过程中，你还可以从同一 Azure 订阅或不同 Azure 订阅添加其他存储帐户。有关添加其他存储帐户的说明，请参阅[设置 HDInsight 群集][]。为了简化本教程中使用的 PowerShell 脚本，所有文件都存储在默认文件系统容器（位于 */tutorials/usepig*）中。默认情况下，此容器与 HDInsight 群集同名。
WASB 语法是：

    wasb[s]://<ContainerName>@<StorageAccountName>.blob.core.chinacloudapi.cn/<路径>/<文件名>

> [WACOM.NOTE] HDInsight 群集 3.0 版只支持 *wasb://* 语法。较早的 *asv://* 语法在 HDInsight 2.1 和 1.6 群集中受支持，但在 HDInsight 3.0 群集中不受支持，以后的版本将不会支持该语法。

> [WACOM.NOTE] WASB 路径是虚拟路径。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][]。

存储在默认文件系统容器中的文件可以使用以下任一 URI 从 HDInsight 进行访问（以 sample.log 为例。此文件是本教程中使用的数据文件）：

    wasb://mycontainer@mystorageaccount.blob.core.windows.net/example/data/sample.log
    wasb:///example/data/sample.log
    /example/data/sample.log

如果要从存储帐户直接访问该文件，则请注意，该文件的 Blob 名称是：

    example/data/sample.log

在本文中，你将使用一个 log4j 示例文件，它是 *\\example\\data\\sample.log* 中存储的 HDInsight 群集附带的。有关上载自己的数据文件的信息，请参阅[将数据上载到 HDInsight][]。

## 了解 Pig Latin

在本课中，你将检查一些 Pig Latin 语句以及运行这些语句后的结果。在下一课中，你将运行 PowerShell 来执行 Pig 语句。

1.  从文件系统加载数据，然后显示结果

        LOGS = LOAD 'wasb:///example/data/sample.log';
        DUMP LOGS;

    输出与下面类似：

        (2012-02-05 19:23:50 SampleClass5 [TRACE] verbose detail for id 313393809)
        (2012-02-05 19:23:50 SampleClass6 [DEBUG] detail for id 536603383)
        (2012-02-05 19:23:50 SampleClass9 [TRACE] verbose detail for id 564842645)
        (2012-02-05 19:23:50 SampleClass8 [TRACE] verbose detail for id 1929822199)
        (2012-02-05 19:23:50 SampleClass5 [DEBUG] detail for id 1599724386)
        (2012-02-05 19:23:50 SampleClass0 [INFO] everything normal for id 2047808796)
        (2012-02-05 19:23:50 SampleClass2 [TRACE] verbose detail for id 1774407365)
        (2012-02-05 19:23:50 SampleClass2 [TRACE] verbose detail for id 2099982986)
        (2012-02-05 19:23:50 SampleClass4 [DEBUG] detail for id 180683124)
        (2012-02-05 19:23:50 SampleClass2 [TRACE] verbose detail for id 1072988373)
        (2012-02-05 19:23:50 SampleClass9 [TRACE] verbose detail)
        ...

2.  浏览数据文件中的每一行，并在 6 个日志级别上查找匹配项：

        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

3.  筛选出没有匹配项的行。例如空行。

        FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
        DUMP FILTEREDLEVELS;

    输出与下面类似：

        (DEBUG)
        (TRACE)
        (TRACE)
        (DEBUG)
        (TRACE)
        (TRACE)
        (DEBUG)
        (TRACE)
        (TRACE)
        (DEBUG)
        (TRACE)
        ...

4.  将所有日志级别组合为自己的行：

        GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
        DUMP GROUPEDLEVELS;

    输出与下面类似：

        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),(TRACE),
        (TRACE), ...

5.  对于每个组，统计日志级别的次数。这些是每个日志级别的频率：

        FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
        DUMP FREQUENCIES;

    输出与下面类似：

        (INFO,3355)
        (WARN,361)
        (DEBUG,15608)
        (ERROR,181)
        (FATAL,37)
        (TRACE,29950)

6.  按降序对频率排序：

        RESULT = order FREQUENCIES by COUNT desc;
        DUMP RESULT;   

    输出与下面类似：

        (TRACE,29950)
        (DEBUG,15608)
        (INFO,3355)
        (WARN,361)
        (ERROR,181)
        (FATAL,37)

## 使用 PowerShell 提交 Pig 作业

本节提供有关使用 PowerShell cmdlet 的说明。在学习本节之前，必须先设置本地环境并配置到 Azure 的连接。有关详细信息，请参阅 [Azure HDInsight 入门][]和[使用 PowerShell 管理 HDInsight][]。

**使用 PowerShell 运行 Pig Latin**

1.  打开 Windows PowerShell ISE（在 Windows 8“开始”屏幕上，键入 **PowerShell\_ISE**，然后单击 **Windows PowerShell ISE**。请参阅[在 Windows 8 和 Windows 上启动 Windows PowerShell][]）。
2.  在底部窗格中，运行以下命令以连接到 Azure 订阅：

        Add-AzureAccount

    系统将提示你输入 Azure 帐户凭据。这种添加订阅连接的方法会超时，12 个小时之后，你将需要再次运行该 cmdlet。

    > [WACOM.NOTE] 如果你有多个 Azure 订阅，而默认订阅不是你想使用的，则请使用 **Select-AzureSubscription** cmdlet 来选择正确的订阅。

3.  在脚本窗格中，复制并粘贴以下行：

        $clusterName = "<HDInsightClusterName>" 
        $statusFolder = "/tutorials/usepig/status"

4.  设置变量 \$clusterName。

5.  在脚本窗格中追加以下行。这些行定义 Pig Latin 查询字符串，并创建 Pig 作业定义：

        # 创建 Pig 作业定义
        $0 = '$0';
        $QueryString =  "LOGS = LOAD 'wasb:///example/data/sample.log';" +
        "LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;" +
        "FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;" +
        "GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;" +
        "FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;" +
        "RESULT = order FREQUENCIES by COUNT desc;" +
        "DUMP RESULT;" 

        $pigJobDefinition = New-AzureHDInsightPigJobDefinition -Query $QueryString -StatusFolder $statusFolder 

    你也可以使用 -File 开关来指定 HDFS 上的 Pig 脚本文件。 -StatusFolder 开关将标准错误日志和标准输出文件放入该文件夹。

6.  追加以下用于提交 Pig 作业的行：

        # 提交 Pig 作业
        Write-Host "Submit the Pig job ..."-ForegroundColor Green
        $pigJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $pigJobDefinition  

7.  追加以下用于等待 Pig 作业完成的行：

        # 等待 Pig 作业完成
        Write-Host "Wait for the Pig job to complete ..."-ForegroundColor Green
        Wait-AzureHDInsightJob -Job $pigJob -WaitTimeoutInSeconds 3600

8.  追加以下行以打印 Pig 作业的输出结果：

        # 打印 Pig 作业的标准错误和标准输出结果。
        #Write-Host "Display the standard error log ..."-ForegroundColor Green
        #Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardError

        Write-Host "Display the standard output ..."-ForegroundColor Green
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardOutput

    > [WACOM.NOTE] Get-AzureHDInsightJobOut cmdlet 中有一个为缩短下面屏幕中的输出结果而加了批注。

9.  按 **F5** 键以运行脚本：
    ![HDI.Pig.PowerShell][]

    Pig 作业计算不同日志类型的频率。

## 使用 HDInsight .NET SDK 提交 Pig 作业

下面是使用 HDInsight .NET SDK 提交 Pig 作业的示例。有关创建 C\# 应用程序以提交 Hadoop 作业的说明，请参阅[以编程方式提交 Hadoop 作业][]。

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;

    using System.IO;
    using System.Threading;
    using System.Security.Cryptography.X509Certificates;

    using Microsoft.WindowsAzure.Management.HDInsight;
    using Microsoft.Hadoop.Client;

    namespace SubmitPigJobs
    {
    class Program
        {
    static void Main(string[] args)
            {
    // 设置变量
    string subscriptionID = "<Azure subscription ID>";
    string certFriendlyName = "<certificate friendly name>";
        
    string clusterName = "<HDInsight cluster name>";
    string statusFolderName = @"/tutorials/usepig/status";

    string queryString = "LOGS = LOAD 'wasb:///example/data/sample.log';" +
    "LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;" +
    "FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;" +
    "GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;" +
    "FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;" +
    "RESULT = order FREQUENCIES by COUNT desc;" +
    "DUMP RESULT;";

    // 定义 Pig 作业
    PigJobCreateParameters myJobDefinition = new PigJobCreateParameters()
                {
    Query = queryString,
    StatusFolder = statusFolderName
                };

    // 通过使用友好名称进行标识从证书存储区获取证书对象
    X509Store store = new X509Store();
    store.Open(OpenFlags.ReadOnly);
    X509Certificate2 cert = store.Certificates.Cast<X509Certificate2>().First(item => item.FriendlyName == certFriendlyName);

    JobSubmissionCertificateCredential creds = new JobSubmissionCertificateCredential(new Guid(subscriptionID), cert, clusterName);

    // 创建 hadoop 客户端以连接到 HDInsight
    var jobClient = JobSubmissionClientFactory.Connect(creds);

    // 运行 MapReduce 作业
    Console.WriteLine("----- Submit the Pig job ...");
    JobCreationResults mrJobResults = jobClient.CreatePigJob(myJobDefinition);

    // 等待作业完成
    Console.WriteLine("----- Wait for the Pig job to complete ...");
    WaitForJobCompletion(mrJobResults, jobClient);

    // 显示错误日志
    Console.WriteLine("----- The Pig job error log.");
    using (Stream stream = jobClient.GetJobErrorLogs(mrJobResults.JobId))
                {
    var reader = new StreamReader(stream);
    Console.WriteLine(reader.ReadToEnd());
                }

    // 显示输出日志
    Console.WriteLine("----- The Pig job output log.");
    using (Stream stream = jobClient.GetJobOutput(mrJobResults.JobId))
                {
    var reader = new StreamReader(stream);
    Console.WriteLine(reader.ReadToEnd());
                }

    Console.WriteLine("----- Press ENTER to continue.");
    Console.ReadLine();
            }

    private static void WaitForJobCompletion(JobCreationResults jobResults, IJobSubmissionClient client)
            {
    JobDetails jobInProgress = client.GetJob(jobResults.JobId);
    while (jobInProgress.StatusCode != JobStatusCode.Completed && jobInProgress.StatusCode != JobStatusCode.Failed)
                {
    jobInProgress = client.GetJob(jobInProgress.JobId);
    Thread.Sleep(TimeSpan.FromSeconds(10));
                }
            }
        }
    }

## 后续步骤

虽然利用 Pig 可以执行数据分析，但你也可能会想试试随 HDInsight 提供的其他语言。Hive 提供了一种类似 SQL 的查询语言，你可使用该语言轻松对存储在 HDInsight 中的数据执行查询，而用 Java 编写的 MapReduce 作业允许你执行复杂数据分析。有关详细信息，请参阅以下内容：

-   [Azure HDInsight 入门][]
-   [将数据上传到 HDInsight][将数据上载到 HDInsight]
-   [以编程方式提交 Hadoop 作业][]
-   [Hive 与 HDInsight 配合使用][]

  [Apache Pig]: http://pig.apache.org/
  [Pig 用例]: #usage
  [先决条件]: #prerequisites
  [了解 Pig Latin]: #understand
  [使用 PowerShell 提交 Pig 作业]: #powershell
  [使用 HDInsight .NET SDK 提交 Pig 作业]: #sdk
  [后续步骤]: #nextsteps
  [整个文件示例]: ./media/hdinsight-use-pig/HDI.wholesamplefile.png
  [HDI.PIG.Data.Transformation]: ./media/hdinsight-use-pig/HDI.DataTransformation.gif
  [Pig Latin 参考手册 1]: http://pig.apache.org/docs/r0.7.0/piglatin_ref1.html
  [Pig Latin 参考手册 2]: http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [将 Azure Blob 存储与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/howto-blob-store/
  [将数据上载到 HDInsight]: /en-us/manage/services/hdinsight/howto-upload-data-to-hdinsight/
  [使用 PowerShell 管理 HDInsight]: /en-use/manage/services/hdinsight/administer-hdinsight-using-powershell/
  [在 Windows 8 和 Windows 上启动 Windows PowerShell]: http://technet.microsoft.com/en-us/library/hh847889.aspx
  [HDI.Pig.PowerShell]: ./media/hdinsight-use-pig/hdi.pig.powershell.png
  [以编程方式提交 Hadoop 作业]: ../hdinsight-submit-hadoop-jobs-programmatically/#mapreduce-sdk
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
