<properties
	pageTitle="HDInsight 中的 Pi estimator Hadoop 示例 | Windows Azure"
	description="了解如何在 HDInsight 上运行 Hadoop MapReduce 示例 Pi estimator。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="10/15/2015"
	wacn.date="11/27/2015"/>

# 在 HDInsight 中的 Hadoop 群集上运行 pi estimator MapReduce 程序

HDInsight 群集附带一个 jar 文件，其中包含几个 MarReduce 示例。本文说明如何使用 Azure PowerShell 运行 pi estimator 示例。pi estimator 将会估算数学常量 pi 的值

该程序使用统计学方法（拟蒙特卡罗法）来估算 pi 值。单位平方形内部随机放置的点也落入该平方形内嵌的圆圈内，其概率等于圆圈面积 pi/4。可以从 4R 的值来估算 pi 的值，其中 R 是落入圆圈内的点数与平方形内总点数的比率。所使用的取样点越多，估算值越准确。

为此示例提供的脚本提交了一个 Hadoop jar 作业，设置为使用特定的值（16 个映射）运行，其中每个映射都必须通过参数值计算 1 千万个取样点。可以更改这些参数值以改进 pi 的估算值。例如，pi 采用前 10 位小数时为 3.1415926535。

有助于你快速了解如何使用 HDInsight 来运行 MapReduce 作业的其他示例列在[运行 HDInsight 示例][hdinsight-samples]页上，其中提供的链接指向有关如何运行这些作业的说明。


**其他相关文章：**

* [Azure HDInsight 入门][hdinsight-get-started]
* [为 HDInsight 中的 Hadoop 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce)
* [在 HDInsight 中提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically)
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [示例：Wordcount][hdinsight-sample-wordcount]
* [示例：C# 流式处理][hdinsight-sample-cs-streaming]







**先决条件**：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
- **一个 HDInsight 群集**。有关可用于创建这类群集的不同方法的说明，请参阅[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters)。
- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell](/documentation/articles/install-configure-powershell)。<a id="run-sample"></a>



## 使用 Azure PowerShell 运行示例

**提交 MapReduce 作业**

1. 打开 Azure PowerShell。有关如何使用 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
2. 在以下命令中设置两个变量，然后运行它们：

		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name

4. 运行以下命令创建 MapReduce 定义：

		$piEstimatorJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "pi" -Arguments "16", "10000000"


	第一个参数指明创建多少个映射（默认值为 16）。第二个参数指示每个映射生成多少个示例（默认为一千万）。因此，此程序使用 16*1 千万 = 1.6 亿个随机点估算 pi。第三个参数指示用于在 HDInsight 3.0 和 3.1 群集上运行示例的 jar 文件的位置和名称。（有关文件的内容，请参阅下文。）

5. 运行以下命令以提交 MapReduce 作业并等待作业完成：

		# Run the pi estimator MapReduce job
		Select-AzureSubscription $subscriptionName
		$piJob = $piEstimatorJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName

		# Wait for the job to finish  
		$piJob | Wait-AzureHDInsightJob -Subscription $subscriptionName -WaitTimeoutInSeconds 3600  

6. 运行以下命令以检索 MapReduce 作业的标准输出：

		# Print output and standard error file of the MapReduce job
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $piJob.JobId -StandardOutput

	作为比较，pi 采用前十位小数时为 3.1415926535。


<a id="java-code"></a>
## MapReduce Java 源代码

在下面可以检查包含映射器函数和化简器函数的 pi estimator Java 代码。映射器程序生成在单位平方形内部随机放置的指定点数，然后计算位于圆圈内部的这些点的数目。化简器程序累计由映射器统计的点数，然后根据公式 4R 估算 pi 的值，其中 R 是圆圈内统计的点数与方形内总点数的比率。


 	/**
 	* Licensed to the Apache Software Foundation (ASF) under one
 	* or more contributor license agreements. See the NOTICE file
 	* distributed with this work for additional information
 	* regarding copyright ownership. The ASF licenses this file
 	* to you under the Apache License, Version 2.0 (the
 	* "License"); you may not use this file except in compliance
 	* with the License. You may obtain a copy of the License at
 	*
	* http://www.apache.org/licenses/LICENSE-2.0
 	*
 	* Unless required by applicable law or agreed to in writing, software
 	* distributed under the License is distributed on an "AS IS" BASIS,
 	* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or 	implied.
 	* See the License for the specific language governing permissions and
 	* limitations under the License.
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


	//A Map-reduce program to estimate the value of Pi
	//using quasi-Monte Carlo method.
	//
	//Mapper:
	//Generate points in a unit square
	//and then count points inside/outside of the inscribed circle of the square.
	//
	//Reducer:
	//Accumulate points inside/outside results from the mappers.
	//Let numTotal = numInside + numOutside.
	//The fraction numInside/numTotal is a rational approximation of
	//the value (Area of the circle)/(Area of the square),
	//where the area of the inscribed circle is Pi/4
	//and the area of unit square is 1.
	//Then, Pi is estimated value to be 4(numInside/numTotal).
	//

 	public class PiEstimator extends Configured implements Tool {
	//tmp directory for input/output
 	static private final Path TMP_DIR = new Path(
 	PiEstimator.class.getSimpleName() + "_TMP_3_141592654");

	//2-dimensional Halton sequence {H(i)},
	//where H(i) is a 2-dimensional point and i >= 1 is the index.
	//Halton sequence is used to generate sample points for Pi estimation.
 	private static class HaltonSequence {
	// Bases
 	static final int[] P = {2, 3};
	//Maximum number of digits allowed
 	static final int[] K = {63, 40};

 	private long index;
 	private double[] x;
 	private double[][] q;
 	private int[][] d;

	//Initialize to H(startindex),
	//so the sequence begins with H(startindex+1).
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
 	q[i][j] = (j == 0? 1.0: q[i][j-1])/P[i];
 	d[i][j] = (int)(k % P[i]);
 	k = (k - d[i][j])/P[i];
 	x[i] += d[i][j] * q[i][j];
 	}
 	}
 	}

	//Compute next point.
	//Assume the current point is H(index).
	//Compute H(index+1).
	//@return a 2-dimensional point with coordinates in [0,1)^2
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
 	x[i] -= (j == 0? 1.0: q[i][j-1]);
 	}
 	}
 	return x;
 	}
 	}

	//Mapper class for Pi estimation.
	//Generate points in a unit square and then
	//count points inside/outside of the inscribed circle of the square.
 	public static class PiMapper extends MapReduceBase
 	implements Mapper<LongWritable, LongWritable, BooleanWritable, LongWritable> {

	//Map method.
	//@param offset samples starting from the (offset+1)th sample.
	//@param size the number of samples for this map
	//@param out output {ture->numInside, false->numOutside}
	//@param reporter
 	public void map(LongWritable offset,
 	LongWritable size,
 	OutputCollector<BooleanWritable, LongWritable> out,
 	Reporter reporter) throws IOException {

 	final HaltonSequence haltonsequence = new HaltonSequence(offset.get());
 	long numInside = 0L;
 	long numOutside = 0L;

 	for(long i = 0; i < size.get(); ) {
 	//generate points in a unit square
 	final double[] point = haltonsequence.nextPoint();

 	//count points inside/outside of the inscribed circle of the square
 	final double x = point[0] - 0.5;
 	final double y = point[1] - 0.5;
 	if (x*x + y*y > 0.25) {
 	numOutside++;
 	} else {
 	numInside++;
 	}

 	//report status
 	i++;
 	if (i % 1000 == 0) {
 	reporter.setStatus("Generated " + i + " samples.");
 	}
 	}

 	//output map results
 	out.collect(new BooleanWritable(true), new LongWritable(numInside));
 	out.collect(new BooleanWritable(false), new LongWritable(numOutside));
 	}
 	}


	//Reducer class for Pi estimation.
	//Accumulate points inside/outside results from the mappers.
 	public static class PiReducer extends MapReduceBase
 	implements Reducer<BooleanWritable, LongWritable, WritableComparable<?>, Writable> {

 	private long numInside = 0;
 	private long numOutside = 0;
 	private JobConf conf; //configuration for accessing the file system

	//Store job configuration.
 	@Override
 	public void configure(JobConf job) {
 	conf = job;
 	}


	// Accumulate number of points inside/outside results from the mappers.
	// @param isInside Is the points inside?
	// @param values An iterator to a list of point counts
	// @param output dummy, not used here.
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

 	//Reduce task done, write output to a file.
 	@Override
 	public void close() throws IOException {
 	//write output to a file
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

	//Run a map/reduce job for estimating Pi.
	//@return the estimated value of Pi.
 	public static BigDecimal estimate(int numMaps, long numPoints, JobConf jobConf
 	)
 	throws IOException {
 	//setup job conf
 	jobConf.setJobName(PiEstimator.class.getSimpleName());

 	jobConf.setInputFormat(SequenceFileInputFormat.class);

 	jobConf.setOutputKeyClass(BooleanWritable.class);
 	jobConf.setOutputValueClass(LongWritable.class);
 	jobConf.setOutputFormat(SequenceFileOutputFormat.class);

 	jobConf.setMapperClass(PiMapper.class);
 	jobConf.setNumMapTasks(numMaps);

 	jobConf.setReducerClass(PiReducer.class);
 	jobConf.setNumReduceTasks(1);

 	// turn off speculative execution, because DFS doesn't handle
 	// multiple writers to the same file.
 	jobConf.setSpeculativeExecution(false);

 	//setup input/output directories
 	final Path inDir = new Path(TMP_DIR, "in");
 	final Path outDir = new Path(TMP_DIR, "out");
 	FileInputFormat.setInputPaths(jobConf, inDir);
 	FileOutputFormat.setOutputPath(jobConf, outDir);

 	final FileSystem fs = FileSystem.get(jobConf);
 	if (fs.exists(TMP_DIR)) {
	 throw new IOException("Tmp directory " + fs.makeQualified(TMP_DIR)
	 + " already exists. Please remove it first.");
	 }
	 if (!fs.mkdirs(inDir)) {
	 throw new IOException("Cannot create input directory " + inDir);
	 }

	 //generate an input file for each map task
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

	 //start a map/reduce job
	 System.out.println("Starting Job");
	 final long startTime = System.currentTimeMillis();
	 JobClient.runJob(jobConf);
	 final double duration = (System.currentTimeMillis() - startTime)/1000.0;
	 System.out.println("Job Finished in " + duration + " seconds");

	 //read outputs
	 Path inFile = new Path(outDir, "reduce-out");
	 LongWritable numInside = new LongWritable();
	 LongWritable numOutside = new LongWritable();
	 SequenceFile.Reader reader = new SequenceFile.Reader(fs, inFile, jobConf);
	 try {
	 reader.next(numInside, numOutside);
	 } finally {
	 reader.close();
	 }

	 //compute estimated value
	 return BigDecimal.valueOf(4).setScale(20)
	 .multiply(BigDecimal.valueOf(numInside.get()))
	 .divide(BigDecimal.valueOf(numMaps))
	 .divide(BigDecimal.valueOf(numPoints));
	 } finally {
	 fs.delete(TMP_DIR, true);
	 }
	 }

	//Parse arguments and then runs a map/reduce job.
	//Print output in standard out.
	//@return a non-zero if there is an error. Otherwise, return 0.
	 public int run(String[] args) throws Exception {
	 if (args.length != 2) {
	 System.err.println("Usage: "+getClass().getName()+" <nMaps> <nSamples>");
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

	 //main method for running it as a stand alone command.
	 public static void main(String[] argv) throws Exception {
	 System.exit(ToolRunner.run(null, new PiEstimator(), argv));
	 }
	 }

## <a id="summary"></a>摘要

在本教程中，你了解了如何在 HDInsight 上运行 MapReduce 作业，以及如何使用蒙特卡罗方法，这些方法需要且会生成可以由此服务管理的大型数据集。

下面是用于在默认 HDInsight 3.1 或 3.0 群集上运行此示例的完整脚本。

	### Provide the Azure subscription name and the HDInsight cluster name
	$subscriptionName = "<SubscriptionName>"
	$clusterName = "<ClusterName>"  

	###Select the Azure subscription to use
	Select-AzureSubscription $subscriptionName

	### Create a MapReduce job definition
	$piEstimatorJobDefinition = New-AzureHDInsightMapReduceJobDefinition -ClassName "pi" –Arguments “32”, “1000000000” -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar"

	### Run the MapReduce job
	$piJob = $piEstimatorJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName

	### Wait for the job to finish  
	$piJob | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600

	### Print the standard error file of the MapReduce job
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $piJob.JobId -StandardOutput



## <a id="next-steps"></a>后续步骤

有关介绍其他示例运行方法的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

* [Azure HDInsight 入门][hdinsight-get-started]
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [示例：Wordcount][hdinsight-sample-wordcount]
* [示例：C# 流式处理][hdinsight-sample-csharp-streaming]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[powershell-install-configure]: /documentation/articles/install-configure-powershell

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started

[hdinsight-samples]: /documentation/articles/hdinsight-run-samples
[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort
[hdinsight-sample-csharp-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator
[hdinsight-sample-wordcount]: /documentation/articles/hdinsight-sample-wordcount

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig
[hdinsight-sample-cs-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming

<!---HONumber=82-->