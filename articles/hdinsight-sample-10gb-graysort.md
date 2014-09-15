<properties linkid="manage-services-hdinsight-sample-10gb-graysort" urlDisplayName="HDInsight Samples" pageTitle="The 10GB GraySort sample | Azure" metaKeywords="hdinsight, hadoop, hdinsight administration, hdinsight administration azure" description="Learn how to run a general purpose GraySort with HDInsight using Azure PowerShell." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="The 10GB GraySort sample" authors="bradsev" />

# 10GB GraySort 示例

此示例主题介绍如何使用 Azure PowerShell 在 Azure HDInsight 上运行常规用途的 GraySort Hadoop MapReduce 程序。GraySort 是一个基准排序，其指标为在给非常大量的数据（通常至少 100 TB）排序时达到的排序速率（TB/分钟）。

此示例使用适中的 10 GB 数据，这样它运行时能相对快一点。它使用由 Owen O'Malley 和 Arun Murthy 开发的 MapReduce 应用程序，此应用程序以 0.578 TB/分钟（100 TB 用时 173 分钟）的速率赢得了 2009 年年度常用（“daytona”）TB 级排序基准。有关这一排序基准和其他排序基准的详细信息，请参阅 [Sortbenchmark][] 网站。

此示例使用三组 MapReduce 程序：

1.  **TeraGen** 是一个 MapReduce 程序，你可用来生成要排序的数据行。
2.  **TeraSort** 以输入数据为例，使用 MapReduce 将数据排序到总数订单中。TeraSort 是 MapReduce 函数的一种标准排序，但自定义的分区程序除外，此分区程序使用 N-1 个抽样键（用于定义每次简化的键范围）的已排序列表。尤其是，所有 sample[i-1] \<= key \< sample[i] 这样的键都会发送到化简 i。这样就保证了化简 i 都小于化简 i+1 的输出结果。
3.  **TeraValidate** 是一个 MapReduce 程序，用于验证数据已全局排序。它在输出目录中对于每个文件创建一个映射，每个映射都确保每个键均小于或等于前一个键。映射函数也会生成每个文件的第一个和最后一个键的记录，而化简函数会确保文件 i 的第一个键大于文件 i-1 的最后一个键。任何问题都会报告为包含故障键的化简的输出结果。

所有三个应用程序所使用的输入和输出格式都以正确格式读写文本文件。化简的输出结果的复制设置为 1，而不是默认值 3，因为基准比赛不要求输出结果数据复制到多个节点上。

**你将了解到以下内容：**

-   如何使用 Azure PowerShell 在 Azure HDInsight 上运行一系列 MapReduce 程序。
-   用 Java 编写的 MapReduce 程序是怎样的。

**先决条件**：

-   你必须具有 Azure 帐户。有关注册帐户的选项，请参阅[免费试用 Azure][] 页。

-   你必须已经设置了 HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅[设置 HDInsight 群集][]。

-   你必须已经安装了 Azure PowerShell，并且已将其配置为可用于你的帐户。有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][]。

## 本文内容

本主题说明了如何运行组成该示例的 MapReduce 系列程序，展示了 MapReduce 程序的 Java 代码，总结了你已经学习到的内容，并概况了一些后续步骤。本文包括以下各节。

1.  [使用 Azure PowerShell 运行示例][]
2.  [TeraSort MapReduce 程序的 Java 代码][]
3.  [摘要][]
4.  [后续步骤][]

## 使用 Azure PowerShell 运行示例

此示例要求三个任务，每个任务对应于简介部分介绍的一个 MapReduce 程序：

1.  通过运行 **TeraGen** MapReduce 作业生成用于排序的数据。
2.  通过运行 **TeraSort** MapReduce 作业对数据排序。
3.  通过运行 **TeraValidate** MapReduce 作业确认数据已正确排序。

**运行 TeraGen 程序**

1.  打开 Azure PowerShell。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。
2.  在以下命令中设置两个变量，然后运行它们：

        # 提供 Azure 订阅名称和 HDInsight 群集名称。
        $subscriptionName = "myAzureSubscriptionName"   
        $clusterName = "myClusterName"

3.  运行以下命令创建 MapReduce 作业定义：

        # 为 TeraGen MapReduce 程序创建 MapReduce 作业定义
        $teragen = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-examples.jar" -ClassName "teragen" -Arguments "-Dmapred.map.tasks=50", "100000000", "/example/data/10GB-sort-input" 

    > [WACOM.NOTE] *hadoop-examples.jar* 随版本 2.1 HDInsight 群集提供。该文件在版本 3.0 HDInsight 群集上已被重命名为 *hadoop-mapreduce.jar*。

    *"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射来执行该作业。参数 100000000** 指定要生成的数据量。最后一个参数 */example/data/10GB-sort-input* 指定要将结果保存到其中的输出目录（包含用于后续排序阶段的输入）。

4.  运行以下命令以提交作业，等待作业完成，然后打印标准错误：

        # 运行 TeraGen MapReduce 作业。
        # 等待作业完成。
        # 打印 MapReduce 作业的输出结果和标准错误文件
        Select-AzureSubscription $subscriptionName         
        $teragen | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 

**运行 TeraSort 程序**

1.  打开 Azure PowerShell。
2.  在以下命令中设置两个变量，然后运行它们：

        # 提供 Azure 订阅名称和 HDInsight 群集名称。
        $subscriptionName = "myAzureSubscriptionName"   
        $clusterName = "myClusterName"

3.  运行以下命令以定义 MapReduce 作业：

        # 为 TeraSort MapReduce 程序创建 MapReduce 作业定义
        $terasort = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-examples.jar" -ClassName "terasort" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-input", "/example/data/10GB-sort-output" 

    *"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射来执行该作业。参数 100000000** 指定要生成的数据量。最后两个参数指定输入目录和输出目录。

4.  运行以下命令以提交作业，等待作业完成，然后打印标准错误：

        # 运行 TeraSort MapReduce 作业。
        # 等待作业完成。
        # 打印 MapReduce 作业的输出结果和标准错误文件 
        Select-AzureSubscription $subscriptionName        
        $terasort | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 

**运行 TeraValidate 程序**

1.  打开 Azure PowerShell。
2.  在以下命令中设置两个变量，然后运行它们：

        # 提供 Azure 订阅名称和 HDInsight 群集名称。
        $subscriptionName = "myAzureSubscriptionName"   
        $clusterName = "myClusterName"

3.  运行以下命令以定义 MapReduce 作业：

        #   为 TeraValidate MapReduce 程序创建 MapReduce 作业定义
        $teravalidate = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-examples.jar" -ClassName "teravalidate" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-output", "/example/data/10GB-sort-validate" 

    *"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射来执行该作业。*"-Dmapred.reduce.tasks=25"* 参数指定将创建 25 个化简任务来执行该作业。最后两个参数指定输入目录和输出目录。

4.  运行以下命令以提交 MapReduce 作业，等待作业完成，然后打印标准错误：

        # 运行 TeraSort MapReduce 作业。
        # 等待作业完成。
        # 打印 MapReduce 作业的输出结果和标准错误文件 
        Select-AzureSubscription $subscriptionName 
        $teravalidate | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 

## TerraSort MapReduce 程序的 Java 代码

这一部分提供了 TerraSort MapReduce 程序的代码以供检查。

    /**
    * 许可授予 Apache Software Foundation (ASF)（根据一个   
    * 或多个贡献者许可协议）。请参阅注意事项文件 
    * （随本作品分发），用于了解关于版权归属的    
    * 其他信息。ASF 根据 Apache 许可证   
    * 2.0 版（下称”许可证“）许可你使用此文件；    
    * 不得在不符合许可证要求的情况下   
    * 使用此文件。你可以获得一份许可证副本，网址是：   
     *  
    *     http://www.apache.org/licenses/LICENSE-2.0   
     *  
    * 除非适用法律或书面协议要求，该许可证涉及分发的软件  
    * 基于“按原样”方式分发，    
    * 无任何明示或默示的担保或条件。 
    * 请参阅该许可证以了解该许可证下的特定语言管辖权限和  
    * 限制。   
     */ 

    package org.apache.hadoop.examples.terasort;

    import java.io.IOException;
    import java.io.PrintStream;
    import java.net.URI;
    import java.util.ArrayList;
    import java.util.List;

    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.apache.hadoop.conf.Configured;
    import org.apache.hadoop.filecache.DistributedCache;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.NullWritable;
    import org.apache.hadoop.io.SequenceFile;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapred.FileOutputFormat;
    import org.apache.hadoop.mapred.JobClient;
    import org.apache.hadoop.mapred.JobConf;
    import org.apache.hadoop.mapred.Partitioner;
    import org.apache.hadoop.util.Tool;
    import org.apache.hadoop.util.ToolRunner;

    /** 
    * 生成取样拆分点，启动该作业，    
    * 并等待它完成。  
    * <p>
    * 运行该程序：  
    * <b>bin/hadoop jar hadoop-examples-*.jar terasort in-dir out-dir</b>  
     */ 

    public class TeraSort extends Configured implements Tool {
    private static final Log LOG = LogFactory.getLog(TeraSort.class);

      /**   
    * 分区程序，将文本键拆分为大致相等的     
    * 分区（顺序按全局性排序排列）。   
       */   

    static class TotalOrderPartitioner implements Partitioner<Text,Text>{
    private TrieNode trie;
    private Text[] splitPoints;

        /**
    * 一般 trie 节点
         */
    static abstract class TrieNode {
    private int level;
    TrieNode(int level) {
    this.level = level;
          }
    abstract int findPartition(Text key);
    abstract void print(PrintStream strm) throws IOException;
    int getLevel() {
    return level;
          }
        }

        /**
    * 内部 trie 节点，包含基于下一个字符的 256 个
    * 子节点。
         */
    static class InnerTrieNode extends TrieNode {
    private TrieNode[] child = new TrieNode[256];
          
    InnerTrieNode(int level) {
    super(level);
          }
    int findPartition(Text key) {
    int level = getLevel();
    if (key.getLength() <= level) {
    return child[0].findPartition(key);
            }
    return child[key.getBytes()[level]].findPartition(key);
          }
    void setChild(int idx, TrieNode child) {
    this.child[idx] = child;
          }
    void print(PrintStream strm) throws IOException {
    for(int ch=0; ch < 255; ++ch) {
    for(int i = 0; i < 2*getLevel(); ++i) {
    strm.print(' ');
              }
    strm.print(ch);
    strm.println(" ->");
    if (child[ch] != null) {
    child[ch].print(strm);
              }
            }
          }
        }

        /**
    * 叶 trie 节点，通过字符串比较来明确指定的
    * 键在 lower..upper 之间所属的位置。
         */
    static class LeafTrieNode extends TrieNode {
    int lower;
    int upper;
    Text[] splitPoints;
    LeafTrieNode(int level, Text[] splitPoints, int lower, int upper) {
    super(level);
    this.splitPoints = splitPoints;
    this.lower = lower;
    this.upper = upper;
          }
    int findPartition(Text key) {
    for(int i=lower; i<upper; ++i) {
    if (splitPoints[i].compareTo(key) >= 0) {
    return i;
              }
            }
    return upper;
          }
    void print(PrintStream strm) throws IOException {
    for(int i = 0; i < 2*getLevel(); ++i) {
    strm.print(' ');
            }
    strm.print(lower);
    strm.print(", ");
    strm.println(upper);
          }
        }


        /**
    * 从指定的序列文件中读取剪切点。
    * @param fs 文件系统
    * @param p 要读取的路径
    * @param job 作业配置
    * @返回字符串以拆分分区
    * @引发 IOException
         */
    private static Text[] readPartitions(FileSystem fs, Path p, 
    JobConf job) throws IOException {
    SequenceFile.Reader reader = new SequenceFile.Reader(fs, p, job);
    List<Text> parts = new ArrayList<Text>();
    Text key = new Text();
    NullWritable value = NullWritable.get();
    while (reader.next(key, value)) {
    parts.add(key);
    key = new Text();
          }
    reader.close();
    return parts.toArray(new Text[parts.size()]);  
        }

        /**
    * 考虑到一组排序的剪切点，生成将迅速找到
    * 正确分区的 trie。
    * @param 拆分剪切点的列表
    * @param lower 分区 0..numPartitions-1 的下限
    * @param upper 分区 0..numPartitions-1 的上限
    * @param prefix 我们已经对其进行过检查的前缀
    * @param maxDepth 我们将为其生成 trie 的最大深度
    * @返回将正确划分拆分的 trie 节点
         */
    private static TrieNode buildTrie(Text[] splits, int lower, int upper, 
    Text prefix, int maxDepth) {
    int depth = prefix.getLength();
    if (depth >= maxDepth || lower == upper) {
    return new LeafTrieNode(depth, splits, lower, upper);
          }
    InnerTrieNode result = new InnerTrieNode(depth);
    Text trial = new Text(prefix);
    // 在前缀上多追加一个字节
    trial.append(new byte[1], 0, 1);
    int currentBound = lower;
    for(int ch = 0; ch < 255; ++ch) {
    trial.getBytes()[depth] = (byte) (ch + 1);
    lower = currentBound;
    while (currentBound < upper) {
    if (splits[currentBound].compareTo(trial) >= 0) {
    break;
              }
    currentBound += 1;
            }
    trial.getBytes()[depth] = (byte) ch;
    result.child[ch] = buildTrie(splits, lower, currentBound, trial, 
    maxDepth);
          }
    // 选取其余
    trial.getBytes()[depth] = 127;
    result.child[255] = buildTrie(splits, currentBound, upper, trial,
    maxDepth);
    return result;
        }

    public void configure(JobConf job) {
    try {
    FileSystem fs = FileSystem.getLocal(job);
    Path partFile = new Path(TeraInputFormat.PARTITION_FILENAME);
    splitPoints = readPartitions(fs, partFile, job);
    trie = buildTrie(splitPoints, 0, splitPoints.length, new Text(), 2);
    } catch (IOException ie) {
    throw new IllegalArgumentException("can't read paritions file", ie);
          }
        }

    public TotalOrderPartitioner() {
        }

    public int getPartition(Text key, Text value, int numPartitions) {
    return trie.findPartition(key);
        }
        
      }
      
    public int run(String[] args) throws Exception {
    LOG.info("starting");
    JobConf job = (JobConf) getConf();
    Path inputDir = new Path(args[0]);
    inputDir = inputDir.makeQualified(inputDir.getFileSystem(job));
    Path partitionFile = new Path(inputDir, TeraInputFormat.PARTITION_FILENAME);
    URI partitionUri = new URI(partitionFile.toString() +
    "#" + TeraInputFormat.PARTITION_FILENAME);
    TeraInputFormat.setInputPaths(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    job.setJobName("TeraSort");
    job.setJarByClass(TeraSort.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(Text.class);
    job.setInputFormat(TeraInputFormat.class);
    job.setOutputFormat(TeraOutputFormat.class);
    job.setPartitionerClass(TotalOrderPartitioner.class);
    TeraInputFormat.writePartitionFile(job, partitionFile);
    DistributedCache.addCacheFile(partitionUri, job);
    DistributedCache.createSymlink(job);
    job.setInt("dfs.replication", 1);
    TeraOutputFormat.setFinalSync(job, true);
    JobClient.runJob(job);
    LOG.info("done");
    return 0;
      }

      /**
    * @param 参数
       */

    public static void main(String[] args) throws Exception {
    int res = ToolRunner.run(new JobConf(), new TeraSort(), args);
    System.exit(res);
      }

    }

## 总结

本示例已经演示了如何使用 Azure HDInsight 运行一系列 MapReduce 作业，其中一个作业的数据输出结果成为这一系列中下一作业的输入。

## 后续步骤

有关运行其他示例的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

-   [Azure HDInsight 入门][]
-   [示例：Pi estimator][]
-   [示例：Wordcount][]
-   [示例：C\# Steaming][]
-   [Pig 与 HDInsight 配合使用][]
-   [Hive 与 HDInsight 配合使用][]
-   [Azure HDInsight SDK 文档][]

  [Sortbenchmark]: http://sortbenchmark.org/
  [免费试用 Azure]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [使用 Azure PowerShell 运行示例]: #run-sample
  [TeraSort MapReduce 程序的 Java 代码]: #java-code
  [摘要]: #summary
  [后续步骤]: #next-steps
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [示例：Pi estimator]: /en-us/manage/services/hdinsight/howto-run-samples/sample-pi-estimator/
  [示例：Wordcount]: /en-us/manage/services/hdinsight/howto-run-samples/sample-wordcount/
  [示例：C\# Steaming]: /en-us/manage/services/hdinsight/howto-run-samples/sample-csharp-streaming/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
