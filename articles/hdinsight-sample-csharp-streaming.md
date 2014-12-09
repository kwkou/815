<properties linkid="manage-services-hdinsight-sample-csharp-streaming" urlDisplayName="HDInsight Samples" pageTitle="The HDInsight C# streaming wordcount sample | Azure" metaKeywords="hadoop, hdinsight, hdinsight administration, hdinsight administration azure" description="Learn how to run a sample TBD." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="The HDInsight C# streaming wordcount sample" authors="bradsev" />

# HDInsight C\# 流式处理字数统计示例

Hadoop 向 MapReduce 提供了一个流式 API，利用它，你可以采用 Java 之外的其他语言来编写映射函数和化简函数。本教程介绍了如何用 C\# 编写使用 Hadoop 流式传输接口的 MapReduce 程序，以及如何使用 Azure PowerShell cmdlet 在 Azure HDInsight 上运行程序。

在示例中，映射器和化简器都是可执行的，它们从 [stdin][] 读取输入（逐行）并将输出结果发送到 [stdout][stdin]。程序计算文本中所有单词的数量。

如果为“映射器” 指定可执行文件，则当初始化映射器时，每个映射器任务都将启动此可执行文件作为一个单独的进程。当映射器任务运行时，它将其输入转换为行，并将这些行馈送到进程的 [stdin][]。同时，映射器从进程的 stdout 中收集面向行的输出，然后将每行转换为一个键/值对（作为映射器的输出收集）。默认情况下，一行的前缀直至第一个制表符是键，而该行的剩余部分（不包括制表符）是值。如果行中没有制表符，则整行被视为键，而值为 Null。

如果为“化简器” 指定可执行文件，则当初始化化简器时，每个化简器任务都将启动此可执行文件作为一个单独的进程。当化简器任务运行时，它将其输入键/值对转换为行，并将这些行馈送到进程的 [stdin][]。同时，化简器从进程的 [stdout][stdin] 中收集面向行的输出，然后将每行转换为一个键/值对（作为化简器的输出收集）。默认情况下，一行的前缀直至第一个制表符是键，而该行的剩余部分（不包括制表符）是值。

有关 Hadoop 流接口的详细信息，请参阅 [Hadoop 流][]。

**你将了解到以下内容：**

-   如何使用 Azure PowerShell 在 HDInsight 上运行 C\# 流式传输程序以分析文件中包含的数据。
-   如何编写使用 Hadoop 流接口的 C\# 代码。

**先决条件**：

-   你必须具有 Azure 帐户。有关注册帐户的选项，请参阅[免费试用 Azure][] 页。

-   你必须已经设置了 HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅[设置 HDInsight 群集][]。

-   你必须已经安装了 Azure PowerShell，并且已将其配置为可用于你的帐户。有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][]。

## 本文内容

本主题说明了如何运行该示例，展示了 MapReduce 程序的 Java 代码，总结了你已经学习到的内容，并概括了一些后续步骤。本文包括以下各节。

1.  [使用 Azure PowerShell 运行示例][]
2.  [Hadoop 流式传输的 C\# 代码][]
3.  [摘要][]
4.  [后续步骤][]

## 使用 Azure PowerShell 运行示例

**运行 MapReduce 作业**

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  在以下命令中设置两个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称
        $clusterName = "<ClusterName>"             # HDInsight 群集名称

3.  运行以下命令来定义 MapReduce 作业。

        # 为流式传输作业创建 MapReduce 作业定义。
        $streamingWC = New-AzureHDInsightStreamingMapReduceJobDefinition -Files "/example/apps/wc.exe", "/example/apps/cat.exe" -InputPath "/example/data/gutenberg/davinci.txt" -OutputPath "/example/data/StreamingOutput/wc.txt" -Mapper "cat.exe" -Reducer "wc.exe" 

    这些参数指定映射器和化简器函数以及输入文件和输出文件。

4.  运行以下命令以运行 MapReduce 作业，等待作业完成，然后打印标准错误：

        # 运行 C# 流式传输 MapReduce 作业。
        # 等待作业完成。
        # 打印 MapReduce 作业的输出结果和标准错误文件
        Select-AzureSubscription $subscriptionName
        $streamingWC | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 

5.  运行下列命令以显示字数统计的结果。

        $subscriptionName = "<SubscriptionName>"   
        $storageAccountName = "<StorageAccountName>" 
        $containerName = "<ContainerName>

    "

        # 选择当前订阅
        Select-AzureSubscription $subscriptionName

        # Blob 存储容器和帐户名称

    \$storageAccountKey = Get-AzureStorageKey -StorageAccountName \$storageAccountName | %{ \$\_.Primary }
     \$storageContext = New-AzureStorageContext -StorageAccountName \$storageAccountName -StorageAccountKey \$storageAccountKey

        # 检索输出结果
        Get-AzureStorageBlobContent -Container $containerName -Blob "example/data/StreamingOutput/wc.txt/part-00000" -Context $storageContext -Force 

        # 文本中单词的数量为：
        cat ./example/data/StreamingOutput/wc.txt/part-00000

    请注意，MapReduce 作业的输出文件是不可变的。所以，如果你重新运行此示例，将需要更改输出文件的名称。

## Hadoop 流式传输的 C\# 代码

MapReduce 程序使用 cat.exe 应用程序作为映射接口将文本流式传送到控制台，并使用 wc.exe 应用程序作为化简接口来统计从文档中流式传送的字数。映射器和化简器都从标准输入流 (stdin) 逐行读取字符，并写入到标准输出流 (stdout)。

    // cat.exe（映射器）的源代码。 
     
    using System;
    using System.IO;

    namespace cat
    {
    class cat
        {
    static void Main(string[] args)
            {
    if (args.Length > 0)
                {
    Console.SetIn(new StreamReader(args[0])); 
                }

    string line;
    while ((line = Console.ReadLine()) != null) 
                {
    Console.WriteLine(line);
                }
            }
        }
    }

cat.cs 文件中的映射器代码使用 StreamReader 对象将传入流的字符读入到控制台，然后控制台使用静态 Console.Writeline 方法将流写入标准输出流。

    // wc.exe（化简器）的源代码为：

    using System;
    using System.IO;
    using System.Linq;

    namespace wc
    {
    class wc
        {
    static void Main(string[] args)
            {
    string line;
    var count = 0;

    if (args.Length > 0){
    Console.SetIn(new StreamReader(args[0]));
                }

    while ((line = Console.ReadLine()) != null) {
    count += line.Count(cr => (cr == ' ' || cr == '\n'));
                }
    Console.WriteLine(count);
            }
        }
    }

wc.cs 文件中的化简器代码使用 [StreamReader][] 对象从 cat.exe 映射器输出的标准输入流读取字符。当它使用 [Console.Writeline][] 方法读取字符时，它将通过统计位于每个单词末尾的空格和行结束字符的数目来计算单词数量，然后使用 [Console.Writeline][] 方法将总数写入标准输出流中。

## 总结

在本教程中，你了解了如何使用 Hadoop 流在 HDInsight 上部署 MapReduce 作业。

## 后续步骤

有关运行其他示例的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

-   [Azure HDInsight 入门][]
-   [示例：Pi Estimator][]
-   [示例：Wordcount][]
-   [示例：10GB GraySort][]
-   [Pig 与 HDInsight 配合使用][]
-   [Hive 与 HDInsight 配合使用][]
-   [Azure HDInsight SDK 文档][]

  [stdin]: http://msdn.microsoft.com/zh-cn/library/3x292kth(v=vs.110).aspx
  [Hadoop 流]: http://wiki.apache.org/hadoop/HadoopStreaming
  [免费试用 Azure]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [使用 Azure PowerShell 运行示例]: #run-sample
  [Hadoop 流式传输的 C\# 代码]: #java-code
  [摘要]: #summary
  [后续步骤]: #next-steps
  [StreamReader]: http://msdn.microsoft.com/zh-cn/library/system.io.streamreader.aspx
  [Console.Writeline]: http://msdn.microsoft.com/zh-cn/library/system.console.writeline
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [示例：Pi Estimator]: /en-us/manage/services/hdinsight/howto-run-samples/sample-pi-estimator/
  [示例：Wordcount]: /en-us/manage/services/hdinsight/howto-run-samples/sample-wordcount/
  [示例：10GB GraySort]: /en-us/manage/services/hdinsight/howto-run-samples/sample-10gb-graysort/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
