<properties
	pageTitle="10GB GraySort Hadoop MapReduce 示例 | Azure"
	description="了解如何使用 Azure PowerShell 对 HDInsight Hadoop 中的巨量数据（通常至少为 100 TB）运行常规用途的 GraySort。"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	services="hdinsight"
	documentationCenter=""
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="07/09/2015"
	wacn.date="10/03/2015"/>

# HDInsight 中的 10GB GraySort Hadoop MapReduce 示例

本示例主题介绍如何使用 Azure PowerShell 在 Azure HDInsight 上运行常规用途的 GraySort Hadoop MapReduce 程序。GraySort 是一个基准排序，其指标为在给巨量数据（通常至少 100TB）排序时达到的排序速率（TB/分钟）。

此示例使用适中的 10GB 数据，这样它运行时能相对快一点。它使用由 Owen O'Malley 和 Arun Murthy 开发的 MapReduce 应用程序，此应用程序以 0.578TB/分钟（100TB 用时 173 分钟）的速率赢得了 2009 年年度常用（“daytona”）TB 级排序基准。有关这一排序基准和其他排序基准的详细信息，请参阅 [Sortbenchmark](http://sortbenchmark.org/) 网站。

本示例使用三组 MapReduce 程序：
 
1. **TeraGen** 是一个 MapReduce 程序，你可用来生成要排序的数据行。
2. **TeraSort** 以输入数据为例，使用 MapReduce 将数据排序到总数订单中。TeraSort 是 MapReduce 函数的一种标准排序，但自定义的分区程序除外，此分区程序使用 N-1 个抽样键（用于定义每次简化的键范围）的已排序列表。具体说来，sample[i-1] <= key < sample[i] 的所有键都将会发送到化简变量 i。这样可确保化简变量 i 的输出全都小于化简变量 i+1 的输出。
3. **TeraValidate** 是一个 MapReduce 程序，用于验证数据已全局排序。它在输出目录中对于每个文件创建一个映射，每个映射都确保每个键均小于或等于前一个键。映射函数也会生成每个文件的第一个和最后一个键的记录，而化简函数会确保文件 i 的第一个键大于文件 i-1 的最后一个键。任何问题都会报告为包含故障键的化简的输出结果。

所有三个应用程序所使用的输入和输出格式都以正确格式读写文本文件。化简的输出结果的复制设置为 1，而不是默认值 3，因为基准比赛不要求输出结果数据复制到多个节点上。

 
**学习内容：*** 如何使用 Azure PowerShell 在 Azure HDInsight 上运行一系列 MapReduce 程序。* 用 Java 编写的 MapReduce 程序是怎样的。


**先决条件：**

- **一个 Azure 订阅**。请参阅 [azure-trial](/pricing/1rmb-trial/) 页。

- **一个 HDInsight 群集**。有关可用于创建这类群集的不同方法的说明，请参阅[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters/)。

- **配备 Azure PowerShell 的工作站**。请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

	
##使用 Azure PowerShell 运行示例

此示例要求三个任务，每个任务对应于简介部分介绍的一个 MapReduce 程序：

1. 通过运行 **TeraGen** MapReduce 作业生成用于排序的数据。	
2. 通过运行 **TeraSort** MapReduce 作业对数据排序。		
3. 通过运行 **TeraValidate** MapReduce 作业确认数据已正确排序。	


**运行 TeraGen 程序**

1. 打开 Azure PowerShell。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
2. 在以下命令中设置两个变量，然后运行它们：
	
		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"   
		$clusterName = "myClusterName"
                 
4. 运行以下命令创建 MapReduce 作业定义：

		# Create a MapReduce job definition for the TeraGen MapReduce program
		$teragen = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "teragen" -Arguments "-Dmapred.map.tasks=50", "100000000", "/example/data/10GB-sort-input"

	*"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射以执行此作业。参数 *100000000* 指定要生成的数据量。最后一个参数 */example/data/10GB-sort-input* 指定要将结果保存到其中的输出目录（包含用于后续排序阶段的输入）。
	
5. 运行以下命令以提交作业，等待作业完成，然后打印标准错误：

		# Run the TeraGen MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job
		Select-AzureSubscription $subscriptionName         
		$teragen | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 


**运行 TeraSort 程序**

1. 打开 Azure PowerShell。
2. 在以下命令中设置两个变量，然后运行它们：
	
		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"   
		$clusterName = "myClusterName"

3. 运行以下命令来定义 MapReduce 作业：

		# Create a MapReduce job definition for the TeraSort MapReduce program
		$terasort = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "terasort" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-input", "/example/data/10GB-sort-output"

	*"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射以执行此作业。参数 *100000000* 指定要生成的数据量。最后两个参数指定输入目录和输出目录。

4. 运行以下命令以提交作业，等待作业完成，然后打印标准错误：

		# Run the TeraSort MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job 
		Select-AzureSubscription $subscriptionName        
		$terasort | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 


**运行 TeraValidate 程序**
		 		
1. 打开 Azure PowerShell。
2. 在以下命令中设置两个变量，然后运行它们：
	
		# Provide the Azure subscription name and the HDInsight cluster name
		$subscriptionName = "myAzureSubscriptionName"   
		$clusterName = "myClusterName"
                 
3. 运行以下命令来定义 MapReduce 作业：

		#	Create a MapReduce job definition for the TeraValidate MapReduce program
		$teravalidate = New-AzureHDInsightMapReduceJobDefinition -JarFile "/example/jars/hadoop-mapreduce-examples.jar" -ClassName "teravalidate" -Arguments "-Dmapred.map.tasks=50", "-Dmapred.reduce.tasks=25", "/example/data/10GB-sort-output", "/example/data/10GB-sort-validate"

	*"-Dmapred.map.tasks=50"* 参数指定将创建 50 个映射以执行此作业。*"-Dmapred.reduce.tasks=25"* 参数指定将创建 25 个化简任务来执行该作业。最后两个参数指定输入目录和输出目录。
 

4. 运行以下命令以提交 MapReduce 作业，等待作业完成，然后打印标准错误：

		# Run the TeraSort MapReduce job
		# Wait for the job to finish
		# Print output and standard error file of the MapReduce job 
		Select-AzureSubscription $subscriptionName 
		$teravalidate | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 


##TeraSort MapReduce 程序的 Java 代码

这一部分提供了 TeraSort MapReduce 程序的代码以供检查。


	/**
	 * Licensed to the Apache Software Foundation (ASF) under one	
	 * or more contributor license agreements.  See the NOTICE file	
	 * distributed with this work for additional information	
	 * regarding copyright ownership.  The ASF licenses this file	
	 * to you under the Apache License, Version 2.0 (the	
	 * "License"); you may not use this file except in compliance	
	 * with the License.  You may obtain a copy of the License at	
	 *	
	 *     http://www.apache.org/licenses/LICENSE-2.0	
	 *	
	 * Unless required by applicable law or agreed to in writing, software	
	 * distributed under the License is distributed on an "AS IS" BASIS,	
	 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.	
	 * See the License for the specific language governing permissions and	
	 * limitations under the License.	
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
	 * Generates the sampled split points, launches the job, 	
	 * and waits for it to finish. 	
	 * <p>
	 * To run the program: 	
	 * <b>bin/hadoop jar hadoop-examples-*.jar terasort in-dir out-dir</b>	
	 */	
	
	public class TeraSort extends Configured implements Tool {
	  private static final Log LOG = LogFactory.getLog(TeraSort.class);
	
	  /**	
	   * A partitioner that splits text keys into roughly equal 	
	   * partitions in a global sorted order.	
	   */	
	
	  static class TotalOrderPartitioner implements Partitioner<Text,Text>{
	    private TrieNode trie;
	    private Text[] splitPoints;
	
	    /**
	     * A generic trie node
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
	     * An inner trie node that contains 256 children based on the next
	     * character.
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
	     * A leaf trie node that does string compares to figure out where the given
	     * key belongs between lower..upper.
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
	     * Read the cut points from the given sequence file.
	     * @param fs the file system
	     * @param p the path to read
	     * @param job the job config
	     * @return the strings to split the partitions on
	     * @throws IOException
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
	     * Given a sorted set of cut points, build a trie that will find the correct
	     * partition quickly.
	     * @param splits the list of cut points
	     * @param lower the lower bound of partitions 0..numPartitions-1
	     * @param upper the upper bound of partitions 0..numPartitions-1
	     * @param prefix the prefix that we have already checked against
	     * @param maxDepth the maximum depth we will build a trie for
	     * @return the trie node that will divide the splits correctly
	     */
	    private static TrieNode buildTrie(Text[] splits, int lower, int upper, 
	                                      Text prefix, int maxDepth) {
	      int depth = prefix.getLength();
	      if (depth >= maxDepth || lower == upper) {
	        return new LeafTrieNode(depth, splits, lower, upper);
	      }
	      InnerTrieNode result = new InnerTrieNode(depth);
	      Text trial = new Text(prefix);
	      // append an extra byte on to the prefix
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
	      // pick up the rest
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
	   * @param args
	   */
	
	  public static void main(String[] args) throws Exception {
	    int res = ToolRunner.run(new JobConf(), new TeraSort(), args);
	    System.exit(res);
	  }
	
	}
  

##摘要

本示例已经演示了如何使用 Azure HDInsight 运行一系列 MapReduce 作业，其中一个作业的数据输出结果成为这一系列中下一作业的输入。

##后续步骤

有关引导你运行其他示例的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

* [Azure HDInsight 入门][hdinsight-get-started]
* [示例：Pi Estimator][hdinsight-sample-pi-estimator]
* [示例：Wordcount][hdinsight-sample-wordcount]
* [示例：C# 流式处理][hdinsight-sample-csharp-streaming]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx


[Powershell-install-configure]: /documentation/articles/install-configure-powershell/

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[hdinsight-samples]: /documentation/articles/hdinsight-run-samples/
[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort/
[hdinsight-sample-csharp-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming/
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator/
[hdinsight-sample-wordcount]: /documentation/articles/hdinsight-sample-wordcount/

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

<!---HONumber=71-->