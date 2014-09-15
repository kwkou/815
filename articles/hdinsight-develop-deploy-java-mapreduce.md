<properties linkid="manage-services-hdinsight-develop-Java-MapReduce-programs-for-HDInsight" urlDisplayName="HDInsight Tutorials" pageTitle="Develop Java MapReduce programs for HDInsight | Azure" metaKeywords="hdinsight, hdinsight development, hadoop development, hdinsight deployment, development, deployment, tutorial, MapReduce, Java" description="Learn how to develop Java MapReduce programs on HDInsight emulator, how to deploy them to HDInsight." services="hdinsight" title="Develop Java MapReduce programs for HDInsight" umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" authors="jgao" />

# 为 HDInsight 开发 Java MapReduce 程序

本教程指导你完成端到端方案：从在 HDInsight Emulator 上开发并测试单词计数 MapReduce 作业，到在 Azure HDInsight 上部署并运行该作业。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   安装 Azure HDInsight Emulator。有关说明，请参阅[开始使用 HDInsight Emulator][]。
-   在模拟器计算机上安装 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][]。
-   获取 Azure 订阅。有关说明，请参阅[购买选项][]、[免费试用][]。

## 本文内容

-   [使用 Java 开发单词计数 MapReduce 程序][]
-   [在模拟器中测试该程序][]
-   [将数据文件及应用程序上载到 Azure Blob 存储][]
-   [在 Azure HDInsight 中运行 MapReduce 程序][]
-   [检索 MapReduce 结果][]
-   [后续步骤][]

## 使用 Java 开发单词计数 MapReduce 程序

单词计数是一个简单的应用程序，它计算每个单词在给定输入集中出现的次数。

**使用 Java 编写单词计数 MapReduce 作业**

1.  打开记事本。
2.  将以下程序复制并粘贴到记事本中。

        package org.apache.hadoop.examples;

        import java.io.IOException;
        import java.util.StringTokenizer;
        import org.apache.hadoop.conf.Configuration;
        import org.apache.hadoop.fs.Path;
        import org.apache.hadoop.io.IntWritable;
        import org.apache.hadoop.io.Text;
        import org.apache.hadoop.mapreduce.Job;
        import org.apache.hadoop.mapreduce.Mapper;
        import org.apache.hadoop.mapreduce.Reducer;
        import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
        import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
        import org.apache.hadoop.util.GenericOptionsParser;

        public class WordCount {

        public static class TokenizerMapper 
        extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
        StringTokenizer itr = new StringTokenizer(value.toString());
        while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
              }
            }
          }

        public static class IntSumReducer 
        extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, 
        Context context
        ) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val :values) {
        sum += val.get();
              }
        result.set(sum);
        context.write(key, result);
            }
          }

        public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
        System.err.println("Usage:wordcount <in> <out>");
        System.exit(2);
            }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
          }
        }

    请注意，包名为 **org.apache.hadoop.examples**，类名为 **WordCount**。提交 MapReduce 作业时，将使用这些名称。

3.  将该文件另存为 **c:\\Tutorials\\WordCountJava\\WordCount.java**。创建文件夹结构（如果不存在）。

HDInsight Emulator 附带了 *javac* 编译器。

**编译 MapReduce 程序**

1.  打开命令提示符。
2.  将目录更改为 **c:\\Tutorials\\WordCountJava**。这是单词计数 MapReduce 程序所在的文件夹。
3.  运行以下命令以检查两个 jar 文件是否存在：

        dir %hadoop_home%\hadoop-core-1.1.0-SNAPSHOT.jar
        dir %hadoop_home%\lib\commons-cli-1.2.jar

4.  运行以下命令以编译该程序：

        C:\Hadoop\java\bin\javac -classpath %hadoop_home%\hadoop-core-1.1.0-SNAPSHOT.jar;%hadoop_home%\lib\commons-cli-1.2.jar WordCount.java

    javac 位于 C:\\Hadoop\\java\\bin 文件夹中。最后一个参数是当前文件夹中的 java 程序。编译器将在当前文件夹中创建 3 个类文件。

5.  运行以下命令以创建 jar 文件：

        C:\Hadoop\java\bin\jar -cvf WordCount.jar *.class

    该命令在当前文件夹中创建一个 WordCount.jar 文件。

    ![HDI.EMulator.WordCount.Compile][]

## 在模拟器中测试该程序

在模拟器中测试 MapReduce 作业包括以下过程：

1.  将数据文件上载到模拟器上的 HDFS
2.  提交单词计数 MapReduce 作业
3.  检查作业状态
4.  检索作业结果

默认情况下，HDInsight Emulator 使用 HDFS 作为默认文件系统。（可选）你可以将HDInsight Emulator 配置为使用 Azure Blob 存储。有关详细信息，请参阅 [HDInsight Emulator 入门][]。

在本教程中，你将使用 HDFS 的 *copyFromLocal* 命令将数据文件上载到 HDFS。下一节说明如何使用 Azure PowerShell 将文件上载到 Azure Blob 存储。有关将文件上载到 Azure Blob 存储的其他方法，请参阅[将数据上载到 HDInsight][]。

本教程使用以下 HDFS 文件夹结构：

<table border="1">
<tr><td>文件夹</td><td>说明</td></tr>
<tr><td>/WordCount</td><td>单词计数项目的根文件夹。 </td></tr>
<tr><td>/WordCount/Apps</td><td>映射器和化简器可执行文件所在的文件夹。</td></tr>
<tr><td>/WordCount/Input</td><td>MapReduce 源文件文件夹。</td></tr>
<tr><td>/WordCount/Output</td><td>MapReduce 输出文件文件夹。</td></tr>
<tr><td>/WordCount/MRStatusOutput</td><td>作业输出文件夹。</td></tr>
</table>

本教程使用位于 %hadoop\_home% 目录中的 .txt 文件作为数据文件。

> [WACOM.NOTE] Hadoop HDFS 命令区分大小写。

**将数据文件复制到模拟器 HDFS**

1.  从桌面打开 Hadoop 命令行。Hadoop 命令行是由模拟器安装程序安装的。
2.  在 Hadoop 命令行窗口中，运行以下命令以生成输入文件的目录：

        hadoop fs -mkdir /WordCount/Input

    此处使用的路径是相对路径。它等效于以下命令：

        hadoop fs -mkdir hdfs://localhost:8020/WordCount/Input

3.  运行以下命令，将一些文本文件复制到 HDFS 中的输入文件夹：

        hadoop fs -copyFromLocal %hadoop_home%\*.txt /WordCount/Input

    MapReduce 作业将对这些文件中的单词进行计数。

4.  运行以下命令以列出并验证已上载的文件：

        hadoop fs -ls /WordCount/Input

    你会看到大约八个 .txt 文件。

**使用 Hadoop 命令行运行 MapReduce 作业**

1.  从桌面打开 Hadoop 命令行。
2.  运行以下命令，从 HDFS 中删除 /WordCount/Output 文件夹结构。/WordCount/Output 是单词计数 MapReduce 作业的输出文件夹。如果该文件夹已存在，则 MapReduce 作业将失败。如果这是你第二次运行该作业，则这一步是必需的。

        hadoop fs -rmr /WordCount/Output

3.  运行以下命令：

        hadoop jar c:\Tutorials\WordCountJava\wordcount\target\wordcount-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /WordCount/Input /WordCount/Output

    如果作业成功完成，你会获得类似于以下屏幕截图的输出：

    ![HDI.EMulator.WordCount.Run][]

    从屏幕截图中，你可以看到映射和化简操作已完成 100%。它还列出作业 ID job\_201312092021\_0002。同一报告可通过从桌面打开“Hadoop MapReduce 状态” 快捷方式，然后查找作业 ID 来检索到。

运行 MapReduce 作业的另一个选择是使用 Azure PowerShell。有关说明，请参阅 [HDInsight Emulator 入门][开始使用 HDInsight Emulator]。

**显示 HDFS 的输出**

1.  打开 Hadoop 命令行。
2.  运行以下命令以显示输出：

        hadoop fs -ls /WordCount/Output/
        hadoop fs -cat /WordCount/Output/part-00000

    你可以在命令结尾处追加“|more”以获取页面视图。也可以使用 findstr 命令查找字符串模式：

        hadoop fs -cat /WordCount/Output/part-00000 | findstr "there"

至此，你已开发一个单词计数 MapReduce 作业，并在模拟器上测试成功。下一步是在 Azure HDInsight 上部署并运行该作业。

## 将数据上载到 Azure Blob 存储

Azure HDInsight 将 Azure Blob 存储用于数据存储。设置 HDInsight 群集时，将使用 Azure Blob 存储容器来存储系统文件。可以使用此默认容器或其他容器（可以在同一 Azure 存储帐户上，也可以在群集所在的数据中心内的其他存储帐户上）来存储数据文件。

在本教程中，你将在单独的存储帐户上为数据文件和 MapReduce 应用程序创建容器。数据文件是工作站上 %hadoop\_home% 目录中的文本文件。

**创建 Blob 存储和容器**

1.  打开 Azure PowerShell。
2.  设置变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName_Data = "<AzureStorageAccountName>"  
        $containerName_Data = "<ContainerName>"
        $location = "<MicrosoftDataCenter>"  # 例如，“China East”

    **\$subscripionName** 与你的 Azure 订阅相关联。必须命名 **\$storageAccountName\_Data** 和 **\$containerName\_Data**。有关命名限制，请参阅[命名和引用容器、Blob 和元数据][]。

3.  运行以下命令以创建存储帐户，并在该帐户上创建 Blob 存储容器

        # 选择 Azure 订阅
        Select-AzureSubscription $subscriptionName

        # 创建存储帐户
        New-AzureStorageAccount -StorageAccountName $storageAccountName_Data -location $location

        # 创建 Blob 存储容器
        $storageAccountKey = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
        $destContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Data –StorageAccountKey $storageAccountKey  
        New-AzureStorageContainer -Name $containerName_Data -Context $destContext

4.  运行以下命令以验证存储帐户和容器：

        Get-AzureStorageAccount -StorageAccountName $storageAccountName_Data
        Get-AzureStorageContainer -Context $destContext

**上载数据文件**

1.  打开 Azure PowerShell。
2.  设置前三个变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName_Data = "<AzureStorageAccountName>"  
        $containerName_Data = "<ContainerName>"

        $localFolder = "c:\Hadoop\hadoop-1.1.0-SNAPSHOT"
        $destFolder = "WordCount/Input"

    **\$storageAccountName\_Data** 和 **\$containerName\_Data** 与你在上一个过程中定义的一样。

    请注意，源文件文件夹是 **c:\\Hadoop\\hadoop-1.1.0-SNAPSHOT**，目标文件夹是 **WordCount/Input**。

3.  运行以下命令以获取源文件文件夹中的 txt 文件的列表：

        # 获取 txt 文件的列表
        $filesAll = Get-ChildItem $localFolder
        $filesTxt = $filesAll | where {$_.Extension -eq ".txt"}

4.  运行以下命令以创建存储上下文对象：

        # 创建存储上下文对象
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName_Data | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName_Data -StorageAccountKey $storageaccountkey

5.  运行以下命令以复制文件：

        # 将本地工作站中的文件复制到 Blob 容器        
        foreach ($file in $filesTxt){

        $fileName = "$localFolder\$file"
        $blobName = "$destFolder/$file"

        write-host "Copying $fileName to $blobName"

        Set-AzureStorageBlobContent -File $fileName -Container $containerName_Data -Blob $blobName -Context $destContext
        }

6.  运行以下命令以列出已上载的文件：

        # 列出 Blob 存储容器中已上载的文件
        Write-Host "The Uploaded data files:"-BackgroundColor Green
        Get-AzureStorageBlob -Container $containerName_Data -Context $destContext -Prefix $destFolder

    你会看到大约 8 个文本数据文件。

**上载单词计数应用程序**

1.  打开 Azure PowerShell。
2.  设置前三个变量，然后运行命令：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName_Data = "<AzureStorageAccountName>"  
        $containerName_Data = "<ContainerName>"

        $jarFile = "C:\Tutorials\WordCountJava\WordCount.jar"
        $blobFolder = "WordCount/jars"

    **\$storageAccountName\_Data** 和 **\$containerName\_Data** 与你在上一个过程中定义的一样，这意味着你要将数据文件和应用程序上载到同一存储帐户上的同一容器中。

    请注意，目标文件夹是 **WordCount/jars**。

3.  运行以下命令以创建存储上下文对象：

        # 创建存储上下文对象
        Select-AzureSubscription $subscriptionName
        $storageaccountkey = get-azurestoragekey $storageAccountName_Data | %{$_.Primary}
        $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName_Data -StorageAccountKey $storageaccountkey

4.  运行以下命令以复制应用程序：

        Set-AzureStorageBlobContent -File $jarFile -Container $containerName_Data -Blob "$blobFolder/WordCount.jar" -Context $destContext

5.  运行以下命令以列出已上载的文件：

        # 列出 Blob 存储容器中已上载的文件
        Write-Host "The Uploaded application file:"-BackgroundColor Green
        Get-AzureStorageBlob -Container $containerName_Data -Context $destContext -Prefix $blobFolder

    你会看到列出的 jar 文件。

## 在 Azure HDInsight 中运行 MapReduce 作业

下面的 PowerShell 脚本执行以下任务：

1.  设置 HDInsight 群集

    1.  创建将用作默认 HDInsight 群集文件系统的存储帐户
    2.  创建 Blob 存储容器
    3.  创建 HDInsight 群集

2.  提交 MapReduce 作业

    1.  创建 MapReduce 作业定义
    2.  提交 MapReduce 作业
    3.  等待作业完成
    4.  显示标准错误
    5.  显示标准输出

3.  删除群集

    1.  删除 HDInsight 群集
    2.  删除用作默认 HDInsight 群集文件系统的存储帐户

**运行 PowerShell 脚本**

1.  打开记事本。
2.  复制并粘贴以下代码：

        # 存储帐户和 HDInsight 群集变量
        $subscriptionName = "<AzureSubscriptionName>"
        $serviceNameToken = "<ServiceNameTokenString>"
        $location = "<MicrosoftDataCenter>"     ### 必须与数据存储帐户位置匹配
        $clusterNodes = <NumberOFNodesInTheCluster>

        $storageAccountName_Data = "<TheDataStorageAccountName>"
        $containerName_Data = "<TheDataBlobStorageContainerName>"

        $clusterName = $serviceNameToken + "hdicluster"

        $storageAccountName_Default = $serviceNameToken + "hdistore"
        $containerName_Default =  $serviceNameToken + "hdicluster"

        # MapReduce 作业变量
        $jarFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/jars/WordCount.jar"
        $className = "org.apache.hadoop.examples.WordCount"
        $mrInput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Input/"
        $mrOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Output/"
        $mrStatusOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/MRStatusOutput/"

        # 创建一个 PSCredential 对象。用户名和密码在此处进行硬编码。你可以根据需要更改它们。
        $password = ConvertTo-SecureString "Pass@word1" -AsPlainText -Force
        $creds = New-Object System.Management.Automation.PSCredential ("Admin", $password) 

        Select-AzureSubscription $subscriptionName

        #=============================
        # 创建用作默认文件系统的存储帐户
        Write-Host "Create a storage account" -ForegroundColor Green
        New-AzureStorageAccount -StorageAccountName $storageAccountName_Default -location $location

        #=============================
        # 创建用作默认文件系统的 Blob 存储容器
        Write-Host "Create a Blob storage container" -ForegroundColor Green
        $storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
        $destContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Default –StorageAccountKey $storageAccountKey_Default

        New-AzureStorageContainer -Name $containerName_Default -Context $destContext

        #=============================
        # 创建 HDInsight 群集
        Write-Host "Create an HDInsight cluster" -ForegroundColor Green
        $storageAccountKey_Data = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }

        $config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
        Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
        Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Data.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Data

        New-AzureHDInsightCluster -Name $clusterName -Location $location -Credential $creds -Config $config

        #=============================
        # 创建 MapReduce 作业定义
        Write-Host "Create a MapReduce job definition" -ForegroundColor Green
        $mrJobDef = New-AzureHDInsightMapReduceJobDefinition -JobName mrWordCountJob  -JarFile $jarFile -ClassName $className -Arguments $mrInput, $mrOutput -StatusFolder /WordCountStatus

        #=============================
        # 运行 MapReduce 作业
        Write-Host "Run the MapReduce job" -ForegroundColor Green
        $mrJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $mrJobDef 
        Wait-AzureHDInsightJob -Job $mrJob -WaitTimeoutInSeconds 3600 

        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardError 
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardOutput

        #=============================
        # 删除 HDInsight 群集
        Write-Host "Delete the HDInsight cluster" -ForegroundColor Green
        Remove-AzureHDInsightCluster -Name $clusterName  

        # 删除默认文件系统存储帐户
        Write-Host "Delete the storage account" -ForegroundColor Green
        Remove-AzureStorageAccount -StorageAccountName $storageAccountName_Default

3.  设置此脚本中的前六个变量。**\$serviceNameToken** 将用于 HDInsight 群集名称、默认存储帐户名称和默认 Blob 存储容器名称。由于服务名称必须介于 3 到 24 个字符之间，并且此脚本将包含多达 10 字符的字符串追加到名称，因此你必须将字符串限制为 14 个字符或更少字符。\$serviceNameToken 必须使用小写字母。**\$storageAccountName\_Data** 和 **\$containerName\_Data** 是用于存储数据文件和应用程序的存储帐户和容器。**\$location** 必须与数据存储帐户位置匹配。
4.  复查其余变量。
5.  保存脚本文件。
6.  打开 Azure PowerShell。
7.  运行以下命令，将执行策略设为 RemoteSigned：

        PowerShell -File <FileName> -ExecutionPolicy RemoteSigned

8.  出现提示时，输入 HDInsight 群集的用户名和密码。由于你将在脚本末尾删除群集，并且将不再需要用户名和密码，因此用户名和密码可以是任何字符串。如果你不想让系统提示你输入凭据，请参阅[在 Windows PowerShell 中使用密码、安全字符串和凭据][]

## 检索 MapReduce 作业输出

本节演示如何下载和显示输出。有关在 Excel 中显示结果的信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][] 和[利用 Power Query 将 Excel 连接到 HDInsight][]。

**检索输出**

1.  打开 Azure PowerShell 窗口。
2.  将目录更改为 **C:\\Tutorials\\WordCountJava**。默认 Azure PowerShell 文件夹是 **C:\\Windows\\System32\\WindowsPowerShell\\v1.0**。你将运行的 cmdlet 会将输出文件下载到当前文件夹。你无权将文件下载到系统文件夹。
3.  运行以下命令以设置值：

        $subscriptionName = "<AzureSubscriptionName>"
        $storageAccountName_Data = "<TheDataStorageAccountName>"
        $containerName_Data = "<TheDataBlobStorageContainerName>"
        $blobName = "WordCount/Output/part-r-00000"

4.  运行以下命令以创建 Azure 存储上下文对象：

        Select-AzureSubscription $subscriptionName
        $storageAccountKey = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
        $storageContext = New-AzureStorageContext –StorageAccountName $storageAccountName_Data –StorageAccountKey $storageAccountKey  

5.  运行以下命令以下载并显示输出：

        Get-AzureStorageBlobContent -Container $containerName_Data -Blob $blobName -Context $storageContext -Force
        cat "./$blobName" | findstr "there"

作业完成后，你可以选择使用 [Sqoop][] 将数据导出到 SQL Server 或 Azure SQL Database，或者将数据导出到 Excel。

## 后续步骤

在本教程中，你已学习如何执行以下操作：开发 Java MapReduce 作业、在 HDInsight Emulator 中测试应用程序、编写 PowerShell 脚本以设置 HDInsight 群集以及在群集上运行 MapReduce。若要了解更多信息，请参阅下列文章：

-   [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序][]
-   [Azure HDInsight 入门][]
-   [HDInsight Emulator 入门][开始使用 HDInsight Emulator]
-   [将 Azure Blob 存储与 HDInsight 配合使用][]
-   [使用 PowerShell 管理 HDInsight][]
-   [将数据上载到 HDInsight][]
-   [将 Hive 与 HDInsight 配合使用][]
-   [将 Pig 与 HDInsight 配合使用][]
-   [利用 Power Query 将 Excel 连接到 HDInsight][]
-   [使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight][使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]

  [开始使用 HDInsight Emulator]: ../hdinsight-get-started-emulator/
  [安装和配置 Azure PowerShell]: ../install-configure-powershell/
  [购买选项]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [免费试用]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [使用 Java 开发单词计数 MapReduce 程序]: #develop
  [在模拟器中测试该程序]: #test
  [将数据文件及应用程序上载到 Azure Blob 存储]: #upload
  [在 Azure HDInsight 中运行 MapReduce 程序]: #run
  [检索 MapReduce 结果]: #retrieve
  [后续步骤]: #nextsteps
  [HDI.EMulator.WordCount.Compile]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Compile-Java-MapReduce.png
  [HDInsight Emulator 入门]: ../hdinsight-get-started-emulator/#blobstorage
  [将数据上载到 HDInsight]: ../hdinsight-upload-data/
  [HDI.EMulator.WordCount.Run]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Run-Java-MapReduce.png
  [命名和引用容器、Blob 和元数据]: http://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx
  [在 Windows PowerShell 中使用密码、安全字符串和凭据]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx
  [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]: ../hdinsight-connect-excel-hive-ODBC-driver/
  [利用 Power Query 将 Excel 连接到 HDInsight]: ../hdinsight-connect-excel-power-query/
  [Sqoop]: ../hdinsight-use-sqoop/
  [为 HDInsight 开发 C\# Hadoop 流 MapReduce 程序]: ../hdinsight-hadoop-develop-deploy-streaming-jobs/
  [Azure HDInsight 入门]: ../hdinsight-get-started/
  [将 Azure Blob 存储与 HDInsight 配合使用]: ../hdinsight-use-blob-storage/
  [使用 PowerShell 管理 HDInsight]: ../hdinsight-administer-use-powershell/
  [将 Hive 与 HDInsight 配合使用]: ../hdinsight-use-hive/
  [将 Pig 与 HDInsight 配合使用]: ../hdinsight-use-pig/
