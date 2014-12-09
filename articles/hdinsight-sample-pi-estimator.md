<properties linkid="manage-services-hdinsight-sample-pi-estimator" urlDisplayName="HDInsight Samples" pageTitle="The HDInsight Pi estimator sample | Azure" metaKeywords="hdinsight, hdinsight sample,  hadoop, mapreduce" description="Learn how to run an Hadoop MapReduce sample on HDInsight." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="The HDInsight Pi estimator sample" authors="bradsev" />

# HDInsight Pi estimator 示例

本主题介绍如何使用 Azure PowerShell 在 HDInsight 中运行一个估算数学常量 Pi 的值的 Hadoop MapReduce 程序。另外还提供了在 MapReduce 程序中使用的用于估算 Pi 值的 Java 代码。

该程序使用统计学方法（拟蒙特卡罗法）来估算 Pi 值。单位平方形内部随机放置的点也落入该平方形内嵌的圆圈内，其概率等于圆圈面积 Pi/4。可以从 4R 的值来估算 Pi 的值，其中 R 是落入圆圈内的点数与平方形内总点数的比率。所使用的取样点越多，估算值越准确。

在下面可以检查包含映射器函数和化简器函数的 PiEstimator Java 代码。映射器程序生成在单位平方形内部随机放置的指定点数，然后计算位于圆圈内部的这些点的数目。化简器程序累计由映射器统计的点数，然后根据公式 4R 估算 Pi 的值，其中 R 是圆圈内统计的点数与方形内总点数的比率。

为此示例提供的脚本提交了一个 Hadoop JAR 作业，设置为使用特定的值（16 个映射）运行，其中每个映射都必须通过参数值计算 1 千万个取样点。可以更改这些参数值以改进 Pi 的估算值。例如，Pi 采用前 10 位小数时为 3.1415926535。

包含 Hadoop 在 Azure 上部署该应用程序所需文件的 .jar 文件是一个可以下载的 .zip 文件。你可以使用各种压缩实用程序来对其解压缩，然后随时浏览这些文件。

有助于你快速了解如何使用 HDInsight 来运行 MapReduce 作业的其他示例列在[运行 HDInsight 示例][]页上，其中提供的链接指向有关如何运行这些作业的说明。

**你将了解到以下内容：**

-   如何使用 Azure PowerShell 在 Azure HDInsight 上运行 Pi Estimator MapReduce 程序。
-   用 Java 编写的 MapReduce 程序是怎样的。

**先决条件**：

-   你必须具有 Azure 帐户。有关注册帐户的选项，请参阅[免费试用 Azure][] 页。

-   你必须已经设置了 HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅[设置 HDInsight 群集][]。

-   你必须已经安装了 Azure PowerShell，并且已将其配置为可用于你的帐户。有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][]。

## 本文内容

本主题说明了如何运行该示例，展示了 Pi Estimator MapReduce 程序的 Java 代码，总结了你已经学习到的内容，并概况了一些后续步骤。本文包括以下各节。

1.  [使用 Azure PowerShell 运行示例][]
2.  [Pi estimator MapReduce 程序的 Java 代码][]
3.  [摘要][]
4.  [后续步骤][]

## 使用 Azure PowerShell 运行示例

**提交 MapReduce 作业**

1.  打开 Azure PowerShell。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。
2.  在以下命令中设置两个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称
        $clusterName = "<ClusterName>"             # HDInsight 群集名称

3.  运行以下命令创建 MapReduce 定义：

        $piEstimatorJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "pi" -Arguments "16", "10000000" 

    > [WACOM.NOTE] *hadoop-examples.jar* 随版本 2.1 HDInsight 群集提供。该文件在版本 3.0 HDInsight 群集上已被重命名为 *hadoop-mapreduce.jar*。

    第一个参数指明创建多少个映射（默认值为 16）。第二个参数指示每个映射生成多少个示例（默认为一千万）。因此，此程序使用 16\*1 千万 = 1.6 亿个随机点估算 Pi。

4.  运行以下命令以提交 MapReduce 作业并等待作业完成：

        # 运行 Pi Estimator MapReduce 作业。
        Select-AzureSubscription $subscriptionName
        $piJob = $piEstimatorJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName

        # 等待作业完成。  
        $piJob | Wait-AzureHDInsightJob -Subscription $subscriptionName -WaitTimeoutInSeconds 3600  

5.  运行以下命令来检索 MapReduce 作业标准输出：

        # 打印 MapReduce 作业的输出结果和标准错误文件
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $piJob.JobId -StandardOutput

    作为比较，Pi 采用前十位小数时为 3.1415926535

## Pi Estimator MapReduce 程序的 Java 代码

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

    package org.apache.hadoop.examples;

    import java.io.IOException;
    import java.math.BigDecimal;
    import java.util.Iterator;

    import org.apache.hadoop.conf.Configured;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.BooleanWritable;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.SequenceFile;
    import org.apache.hadoop.io.Writable;
    import org.apache.hadoop.io.WritableComparable;
    import org.apache.hadoop.io.SequenceFile.CompressionType;
    import org.apache.hadoop.mapred.FileInputFormat;
    import org.apache.hadoop.mapred.FileOutputFormat;
    import org.apache.hadoop.mapred.JobClient;
    import org.apache.hadoop.mapred.JobConf;
    import org.apache.hadoop.mapred.MapReduceBase;
    import org.apache.hadoop.mapred.Mapper;
    import org.apache.hadoop.mapred.OutputCollector;
    import org.apache.hadoop.mapred.Reducer;
    import org.apache.hadoop.mapred.Reporter;
    import org.apache.hadoop.mapred.SequenceFileInputFormat;
    import org.apache.hadoop.mapred.SequenceFileOutputFormat;
    import org.apache.hadoop.util.Tool;
    import org.apache.hadoop.util.ToolRunner;


    //用于估算 Pi 值的 Map-reduce 程序  
    //使用拟蒙特卡罗方法。   
    //  
    //映射器：   
    //在单元方形内生成点，  
    //然后统计该方形的内接圆内部/外部的点数。   
    //  
    //化简器：  
    //累计映射器结果内部/外部的点数。    
    //让 numTotal = numInside + numOutside。    
    //分数 numInside/numTotal 是    
    //（圆形面积）/（方形面积）值的有理数近似值，  
    //其中内接圆的面积是 Pi/4，    
    //单元方形的面积是 1。 
    //然后，估算 Pi 值为 4(numInside/numTotal)。  
    //  

    public class PiEstimator extends Configured implements Tool {   
    //输入/输出结果的临时目录    
    static private final Path TMP_DIR = new Path(   
    PiEstimator.class.getSimpleName() + "_TMP_3_141592654");    

    //2 维 Halton 数列 {H(i)}，     
    //其中 H(i) 是 2 维点，i >= 1 是指数。      
    //Halton 数列可生成用于估算 Pi 值的取样点。   
    private static class HaltonSequence {   
    // 基数    
    static final int[] P = {2, 3};  
    //允许的最大位数  
    static final int[] K = {63, 40};    

    private long index; 
    private double[] x; 
    private double[][] q;   
    private int[][] d;  

    //初始化为 H(startindex)，  
    //因此，该数列以 H(startindex+1) 开头。  
    HaltonSequence(long startindex) {   
    index = startindex; 
    x = new double[K.length];   
    q = new double[K.length][]; 
    d = new int[K.length][];    
    for(int i = 0; i < K.length; i++) { 
    q[i] = new double[K[i]];    
    d[i] = new int[K[i]];   
    }

    for(int i = 0; i < K.length; i++) {
    long k = index;
    x[i] = 0;

    for(int j = 0; j < K[i]; j++) { 
    q[i][j] = (j == 0?1.0: q[i][j-1])/P[i];    
    d[i][j] = (int)(k % P[i]);  
    k = (k - d[i][j])/P[i]; 
    x[i] += d[i][j] * q[i][j];  
    }   
    }
    }

    //计算下一个点。   
    //假定当前点为 H(index)。 
    //计算 H(index+1)。   
    //@返回一个坐标在 [0,1)^2 中的 2 维点     
    double[] nextPoint() {      
    index++;    
    for(int i = 0; i < K.length; i++) { 
    for(int j = 0; j < K[i]; j++) { 
    d[i][j]++;  
    x[i] += q[i][j];    
    if (d[i][j] < P[i]) {   
    break;      
    }   
    d[i][j] = 0;    
    x[i] -= (j == 0?1.0: q[i][j-1]);   
    }   
    }   
    return x;   
    }   
    }   

    //用于估算 Pi 值的映射器类。   
    //在单元方形内生成点，然后     
    //统计该方形的内接圆内部/外部的点数。    
    public static class PiMapper extends MapReduceBase
    implements Mapper<LongWritable, LongWritable, BooleanWritable, LongWritable> {

    //映射方法。   
    //@param offset 从第 (offset+1) 个示例开始的示例。  
    //@param size 此映射的示例数    
    //@param out 输出 {true->numInside, false->numOutside}    
    // @param reporter   
    public void map(LongWritable offset,
    LongWritable size,
    OutputCollector<BooleanWritable, LongWritable> out,
    Reporter reporter) throws IOException {

    final HaltonSequence haltonsequence = new HaltonSequence(offset.get());     
    long numInside = 0L;    
    long numOutside = 0L;   

    for(long i = 0; i < size.get(); ) {
    //在单元方形内生成点
    final double[] point = haltonsequence.nextPoint();

    //统计该方形的内接圆内部/外部的点数
    final double x = point[0] - 0.5;
    final double y = point[1] - 0.5;
    if (x*x + y*y > 0.25) {
    numOutside++;
    } else {
    numInside++;
    }

    //报告状态
    i++;
    if (i % 1000 == 0) {
    reporter.setStatus("Generated " + i + " samples.");
    }
    }

    //输出映射结果
    out.collect(new BooleanWritable(true), new LongWritable(numInside));
    out.collect(new BooleanWritable(false), new LongWritable(numOutside));
    }
    }


    //用于估算 Pi 值的化简器类。      
    //累计映射器结果内部/外部的点数。        
    public static class PiReducer extends MapReduceBase
    implements Reducer<BooleanWritable, LongWritable, WritableComparable<?>, Writable> {

    private long numInside = 0;
    private long numOutside = 0;
    private JobConf conf; //configuration for accessing the file system

    //存储作业配置。  
    @Override   
    public void configure(JobConf job) {
    conf = job;
    }


    // 累计映射器结果内部/外部的点数。     
    // @param isInside 是否为内部的点？    
    // @param values，点计数列表的迭代器  
    // @param output dummy，不在此使用。  
    // @param reporter  

    public void reduce(BooleanWritable isInside,
    Iterator<LongWritable> values,
    OutputCollector<WritableComparable<?>, Writable> output,
    Reporter reporter) throws IOException {
    if (isInside.get()) {
    for(; values.hasNext(); numInside += values.next().get());
    } else {
    for(; values.hasNext(); numOutside += values.next().get());
    }
    }

    //化简任务完成，将输出写入某个文件。 
    @Override   
    public void close() throws IOException {
    //将输出写入某个文件
    Path outDir = new Path(TMP_DIR, "out");
    Path outFile = new Path(outDir, "reduce-out");
    FileSystem fileSys = FileSystem.get(conf);
    SequenceFile.Writer writer = SequenceFile.createWriter(fileSys, conf,
    outFile, LongWritable.class, LongWritable.class, 
    CompressionType.NONE);
    writer.append(new LongWritable(numInside), new LongWritable(numOutside));
    writer.close();
    }
    }

    //运行用于估算 Pi 值的映射/化简作业。   
    //@返回估算的 Pi 值。        
    public static BigDecimal estimate(int numMaps, long numPoints, JobConf jobConf
    )   
    throws IOException {
    //设置作业配置
    jobConf.setJobName(PiEstimator.class.getSimpleName());

    jobConf.setInputFormat(SequenceFileInputFormat.class);

    jobConf.setOutputKeyClass(BooleanWritable.class);
    jobConf.setOutputValueClass(LongWritable.class);
    jobConf.setOutputFormat(SequenceFileOutputFormat.class);

    jobConf.setMapperClass(PiMapper.class);
    jobConf.setNumMapTasks(numMaps);

    jobConf.setReducerClass(PiReducer.class);
    jobConf.setNumReduceTasks(1);

    // 关闭推测执行，因为 DFS 不将       
    // 多个编写器处理到同一文件。       
    jobConf.setSpeculativeExecution(false);

    //设置输入/输出目录    
    final Path inDir = new Path(TMP_DIR, "in"); 
    final Path outDir = new Path(TMP_DIR, "out");   
    FileInputFormat.setInputPaths(jobConf, inDir);  
    FileOutputFormat.setOutputPath(jobConf, outDir);    

    final FileSystem fs = FileSystem.get(jobConf);      
    if (fs.exists(TMP_DIR)) {   
    throw new IOException("Tmp directory " + fs.makeQualified(TMP_DIR)
    + " already exists.Please remove it first."); 
     }  
    if (!fs.mkdirs(inDir)) {   
    throw new IOException("Cannot create input directory " + inDir);   
     }  

    //为每个映射任务生成输入文件     
    try {
    for(int i=0; i < numMaps; ++i) {
    final Path file = new Path(inDir, "part"+i);   
    final LongWritable offset = new LongWritable(i * numPoints);   
    final LongWritable size = new LongWritable(numPoints); 
    final SequenceFile.Writer writer = SequenceFile.createWriter(
    fs, jobConf, file,
    LongWritable.class, LongWritable.class, CompressionType.NONE);
    try {
    writer.append(offset, size);
    } finally {
    writer.close();    
     }
    System.out.println("Wrote input for Map #"+i);
     }

    //启动映射/化简作业       
    System.out.println("Starting Job");
    final long startTime = System.currentTimeMillis();
    JobClient.runJob(jobConf);
    final double duration = (System.currentTimeMillis() - startTime)/1000.0;
    System.out.println("Job Finished in " + duration + " seconds");

    //读取输出结果     
    Path inFile = new Path(outDir, "reduce-out");
    LongWritable numInside = new LongWritable();
    LongWritable numOutside = new LongWritable();
    SequenceFile.Reader reader = new SequenceFile.Reader(fs, inFile, jobConf);
    try {
    reader.next(numInside, numOutside);
    } finally {
    reader.close();
     }

    //计算预估的值
    return BigDecimal.valueOf(4).setScale(20)
    .multiply(BigDecimal.valueOf(numInside.get()))
    .divide(BigDecimal.valueOf(numMaps))
    .divide(BigDecimal.valueOf(numPoints));
    } finally {
    fs.delete(TMP_DIR, true);
     }
     }

    //解析参数，然后运行映射/化简作业。   
    //以标准输出格式打印输出结果。     
    //@如果有错误，则返回非零。否则，返回 0。     
    public int run(String[] args) throws Exception {
    if (args.length != 2) {
    System.err.println("Usage:"+getClass().getName()+" <nMaps> <nSamples>");
    ToolRunner.printGenericCommandUsage(System.err);
    return -1;
     }

    final int nMaps = Integer.parseInt(args[0]);
    final long nSamples = Long.parseLong(args[1]);

    System.out.println("Number of Maps = " + nMaps);
    System.out.println("Samples per Map = " + nSamples);

    final JobConf jobConf = new JobConf(getConf(), getClass());
    System.out.println("Estimated value of Pi is "
    + estimate(nMaps, nSamples, jobConf));
    return 0;
     }

    //作为独立命令运行的 main 方法。         
    public static void main(String[] argv) throws Exception {
    System.exit(ToolRunner.run(null, new PiEstimator(), argv));
     }
     }

## 总结

在本教程中，你了解了如何在 HDInsight 上运行 MapReduce 作业，以及如何使用蒙特卡罗方法，这些方法需要且会生成可以由此服务管理的大型数据集。

下面是用于在默认 HDInsight 2.1 群集上运行此示例的完整脚本。（在 HDInsight 3.0 群集上运行该示例只需更改 .jar 文件的名称：hadoop-examples.jar becomes hadoop-mapreduce-examples.jar。）

    ### 提供 Windows Azure 订阅名称和 HDInsight 群集名称。 
    $subscriptionName = "<SubscriptionName>" 
    $clusterName = "<ClusterName>"  

    ###选择要使用的 Azure 订阅。
    Select-AzureSubscription $subscriptionName 

    ### 创建 MapReduce 作业定义。 
    $piEstimatorJobDefinition = New-AzureHDInsightMapReduceJobDefinition -ClassName "pi" –Arguments “32”, “1000000000” -JarFile "wasb:///example/jars/hadoop-examples.jar"

    ### 运行 MapReduce 作业。 
    $piJob = $piEstimatorJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName

    ### 等待作业完成。  
    $piJob | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600

    ### 打印 MapReduce 作业的标准错误文件。
    Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $piJob.JobId -StandardOutput

## 后续步骤

有关运行其他示例的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

-   [Azure HDInsight 入门][]
-   [示例：10GB GraySort][]
-   [示例：Wordcount][]
-   [示例：C\# Steaming][]
-   [Pig 与 HDInsight 配合使用][]
-   [Hive 与 HDInsight 配合使用][]
-   [Azure HDInsight SDK 文档][]

  [运行 HDInsight 示例]: /en-us/manage/services/hdinsight/howto-run-samples
  [免费试用 Azure]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [使用 Azure PowerShell 运行示例]: #run-sample
  [Pi estimator MapReduce 程序的 Java 代码]: #java-code
  [摘要]: #summary
  [后续步骤]: #next-steps
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [示例：10GB GraySort]: /en-us/manage/services/hdinsight/howto-run-samples/sample-10gb-graysort/
  [示例：Wordcount]: /en-us/manage/services/hdinsight/howto-run-samples/sample-wordcount/
  [示例：C\# Steaming]: /en-us/manage/services/hdinsight/howto-run-samples/sample-csharp-streaming/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
