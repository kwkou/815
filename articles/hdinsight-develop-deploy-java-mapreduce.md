<properties 
	pageTitle="为 Hadoop 开发 Java MapReduce 程序 | Azure" 
	description="了解如何在 HDInsight Emulator 上开发 Java MapReduce 程序，以及如何将这些程序部署到 HDInsight。" 
	services="hdinsight" 
	editor="cgronlun" 
	manager="paulettm" 
	authors="nitinme" 
	documentationCenter=""/>

<tags 
   ms.service="hdinsight"
   ms.date="07/11/2015"
   wacn.date="08/14/2015"/>

# 为 HDInsight 中的 Hadoop 开发 Java MapReduce 程序

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-maven-mapreduce-selector.md)]

本教程将引导完成一项端对端方案，在 Apache Maven 中使用 Java 来开发单词计数 Hadoop MapReduce 作业。本教程还将说明如何在 HDInsight Emulator for Azure 上测试该应用程序，然后在基于 Windows 的 HDInsight 群集上部署并运行它。

##<a name="prerequisites"></a>先决条件

在开始本教程之前，必须先完成以下操作：

- 安装 HDInsight Emulator。有关说明，请参阅[开始使用 HDInsight Emulator][hdinsight-emulator]。确保所有必需的服务都在运行。在安装 HDInsight Emulator 的计算机上，从桌面快捷方式启动 Hadoop 命令行，导航到 **C:\\hdp**，然后运行命令 **start_local_hdp_services.cmd**。	
- 在模拟器计算机上安装 Azure PowerShell。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
- 在模拟器计算机上安装 Java 平台 JDK 7 或更高版本。模拟器计算机上已有此版本。
- 安装并配置 [Apache Maven](http://maven.apache.org/)。
- 获取 Azure 订阅。有关说明，请参阅[购买选项][azure-purchase-options]、[试用][azure-trial]。


##<a name="develop"></a>使用 Apache Maven 以 Java 创建 MapReduce 程序

创建单词计数 MapReduce 应用程序。它是一个简单的应用程序，它计算每个单词在给定输入集中出现的次数。在本部分中，我们将执行以下任务：

1. 使用 Apache Maven 创建项目
2. 更新项目对象模型 (POM)
3. 创建单词计数 MapReduce 应用程序
4. 生成并打包应用程序

**使用 Apache Maven 创建项目**

1. 将目录切换到 **C:\\Tutorials\\WordCountJava**。
2. 从开发环境的命令行中，将目录切换到你创建的位置。
3. 使用随同 Maven 一起安装的 __mvn__ 命令，以为项目生成基架。

		mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

	这将在当前目录中创建一个新目录，该目录使用 __artifactID__ 参数指定的名称（在本示例中为 **wordcountjava**）。 此目录将包含以下项：

	* __pom.xml__ - [项目对象模型 (POM)](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)，其中包含用于生成项目的信息和配置详细信息。

	* __src__ - 包含 __main\\java\\org\\apache\\hadoop\\examples__ 目录的目录，你将在其中创作应用程序。
3. 删除 __src\\test\\java\\org\\apache\\hadoop\\examples\\apptest.java__ 文件，因为本示例用不到该文件。

**更新 POM**

1. 编辑 __pom.xml__ 文件，并在 `<dependencies>` 节中添加以下代码。

		<dependency>
		  <groupId>org.apache.hadoop</groupId>
          <artifactId>hadoop-mapreduce-examples</artifactId>
          <version>2.5.1</version>
        </dependency>
    	<dependency>
      	  <groupId>org.apache.hadoop</groupId>
      	  <artifactId>hadoop-mapreduce-client-common</artifactId>
      	  <version>2.5.1</version>
    	</dependency>
    	<dependency>                                                                                     
      	  <groupId>org.apache.hadoop</groupId>                                                                                                       
      	  <artifactId>hadoop-common</artifactId>                                                                                                         
      	  <version>2.5.1</version>                                                                                            
    	</dependency>

	这将告知 Maven，项目需要具有特定版本（在 <version> 中列出）的库（在 <artifactId> 中列出）。在编译时，将从默认的 Maven 存储库下载该版本。你可以使用 [Maven 存储库搜索](http://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar)来查看详细信息。

2. 将以下代码添加到 __pom.xml__ 文件。它必须位于文件中的 `<project>...</project>` 标记内，例如 `</dependencies>` 和 `</project>` 之间。

		<build>
  		  <plugins>
    		<plugin>
      		  <groupId>org.apache.maven.plugins</groupId>
      		  <artifactId>maven-shade-plugin</artifactId>
      		  <version>2.3</version>
      		  <configuration>
        		<transformers>
          		  <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
		          </transformer>
        		</transformers>
      		  </configuration>
      		  <executions>
        		<execution>
          		  <phase>package</phase>
          			<goals>
            		  <goal>shade</goal>
          			</goals>
        	    </execution>
      		  </executions>
      	    </plugin>       
  		  </plugins>
	    </build>

	这会配置 [Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)，用于防止 Maven 所生成的 JAR 中发生许可证重复。使用此插件的原因在于，重复的许可证文件会导致 HDInsight 群集在运行时出错。将 Maven Shade Plugin 与 `ApacheLicenseResourceTransformer` 实现一起使用可防止此错误。

	Maven Shade Plugin 还将生成 uberjar（有时称为 fatjar），其中包含应用程序所需的所有依赖项。

3. 保存 __pom.xml__ 文件。

**创建单词计数应用程序**

1. 转到 __wordcountjava\\src\\main\\java\\org\\apache\\hadoop\\examples__ 目录，并 __app.java__ 文件重命名为 __WordCount.java__。
2. 打开记事本。
2. 将以下程序复制并粘贴到记事本中。

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
		      for (IntWritable val : values) {
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
		      System.err.println("Usage: wordcount <in> <out>");
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
	
3. 保存文件。

**生成并打包应用程序**

1. 打开命令提示符，并将目录切换到 __wordcountjava__ 目录。

2. 使用以下命令生成包含该应用程序的 JAR 文件：

		mvn clean package

	这将会清除以前的所有生成项目，下载尚未安装的所有依赖项，然后生成并打包应用程序。

3. 完成该命令后，__wordcountjava\\target__ 目录将包含名为 __wordcountjava-1.0-SNAPSHOT.jar__ 的文件。

	> [AZURE.NOTE]__wordcountjava-1.0-SNAPSHOT.jar__ 文件是一个 uberjar。


##<a name="test"></a>在模拟器中测试该程序

在 HDInsight Emulator 中测试 MapReduce 作业包括以下过程：

1. 将数据文件上载到模拟器上的 Hadoop 分布式文件系统 (HDFS)
2. 创建本地用户组
3. 运行单词计数 MapReduce 作业
4. 检索作业结果

默认情况下，HDInsight Emulator 使用 HDFS 作为文件系统。（可选）你可以将 HDInsight Emulator 配置为使用 Azure Blob 存储。有关详细信息，请参阅 [HDInsight Emulator 入门][hdinsight-emulator-wasb]。

在本教程中，你将使用 HDFS 的 **copyFromLocal** 命令将数据文件上载到 HDFS。下一部分说明如何使用 Azure PowerShell 将文件上载到 Azure Blob 存储。有关将文件上载到 Azure Blob 存储的其他方法，请参阅[将数据上载到 HDInsight][hdinsight-upload-data]。

本教程使用以下 HDFS 文件夹结构：

<table border="1">
<tr><td>文件夹</td><td>注意</td></tr>
<tr><td>/WordCount</td><td>单词计数项目的根文件夹。</td></tr>
<tr><td>/WordCount/Apps</td><td>映射器和化简器可执行文件所在的文件夹。</td></tr>
<tr><td>/WordCount/Input</td><td>MapReduce 源文件文件夹。</td></tr>
<tr><td>/WordCount/Output</td><td>MapReduce 输出文件文件夹。</td></tr>
<tr><td>/WordCount/MRStatusOutput</td><td>作业输出文件夹。</td></tr>
</table>

本教程使用位于 %hadoop_home% 目录中的 .txt 文件作为数据文件。

> [AZURE.NOTE]Hadoop HDFS 命令区分大小写。

**将数据文件复制到模拟器 HDFS**

1. 从桌面打开 Hadoop 命令行。Hadoop 命令行是由模拟器安装程序安装的。
2. 在 Hadoop 命令行窗口中，运行以下命令以生成输入文件的目录：

		hadoop fs -mkdir /WordCount
		hadoop fs -mkdir /WordCount/Input

	此处使用的路径是相对路径。它等效于以下命令：

		hadoop fs -mkdir hdfs://localhost:8020/WordCount/Input

3. 运行以下命令，将一些文本文件复制到 HDFS 中的输入文件夹：

		hadoop fs -copyFromLocal C:\hdp\hadoop-2.4.0.2.1.3.0-1981\share\doc\hadoop\common*.txt /WordCount/Input

	MapReduce 作业将对这些文件中的单词进行计数。

4. 运行以下命令以列出并验证已上载的文件：

		hadoop fs -ls /WordCount/Input

**创建本地用户组**

为了在群集上成功运行 MapReduce 作业，你必须创建名为 hdfs 的用户组。在此组中，还必须添加用户 hadoop 和用来登入模拟器的本地用户。在提升权限的命令提示符下使用以下命令：

		# Add a user group called hdfs		
		net localgroup hdfs /add

		# Adds a user called hadoop to the group
		net localgroup hdfs hadoop /add

		# Adds the local user to the group
		net localgroup hdfs <username> /add

**使用 Hadoop 命令行运行 MapReduce 作业**

1. 从桌面打开 Hadoop 命令行。
2. 运行以下命令，从 HDFS 中删除 /WordCount/Output 文件夹结构。/Wordcount/output 是单词计数 MapReduce 作业的输出文件夹。如果该文件夹已存在，则 MapReduce 作业将失败。如果这是你第二次运行该作业，则这一步是必需的。

		hadoop fs -rm - r /WordCount/Output

2. 运行以下命令：

		hadoop jar C:\Tutorials\WordCountJava\wordcountjava\target\wordcountjava-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /WordCount/Input /WordCount/Output

	如果作业成功完成，你会获得类似于以下屏幕截图的输出：

	![HDI.EMulator.WordCount.Run][image-emulator-wordcount-run]

	从屏幕截图中，你可以看到映射和化简操作已完成 100%。它还列出作业 ID。同一报告可通过从桌面打开“Hadoop MapReduce 状态”快捷方式，然后查找同一作业 ID 来检索到。

运行 MapReduce 作业的另一个选择是使用 Azure PowerShell。有关说明，请参阅 [HDInsight Emulator 入门][hdinsight-emulator]。

**显示 HDFS 的输出**

1. 打开 Hadoop 命令行。
2. 运行以下命令以显示输出：

		hadoop fs -ls /WordCount/Output/
		hadoop fs -cat /WordCount/Output/part-r-00000

	你可以在命令结尾处追加“|more”以获取页面视图。也可以使用 **findstr** 命令查找字符串模式：

		hadoop fs -cat /WordCount/Output/part-r-00000 | findstr "there"

至此，你已开发一个单词计数 MapReduce 作业，并在模拟器上成功测试。下一步是在 Azure HDInsight 上部署并运行该作业。



##<a id="upload"></a>将数据和应用程序上载到 Azure Blob 存储
Azure HDInsight 将 Azure Blob 存储用于数据存储。设置 HDInsight 群集时，将使用 Azure Blob 存储容器来存储系统文件。可以使用此默认容器或其他容器（可以在同一 Azure 存储帐户上，也可以在群集所在的数据中心内的其他存储帐户上）来存储数据文件。

在本教程中，你将在单独的存储帐户上为数据文件和 MapReduce 应用程序创建容器。数据文件是模拟器工作站上 **C:\\hdp\\hadoop-2.4.0.2.1.3.0-1981\\share\\doc\\hadoop\\common** 目录中的文本文件。

**创建 Blob 存储帐户和容器**

1. 打开 Azure PowerShell。
2. 设置变量，然后运行命令：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName_Data = "<AzureStorageAccountName>"  
		$containerName_Data = "<ContainerName>"
		$location = "<MicrosoftDataCenter>"  # For example, "China East"

	**$subscripionName** 变量与你的 Azure 订阅相关联。必须命名 **$storageAccountName\_Data** 和 **$containerName\_Data**。有关命名限制，请参阅[命名和引用容器、Blob 与元数据](https://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx)。

3. 运行以下命令以创建存储帐户，并在该帐户上创建 Blob 存储容器

		# Select an Azure subscription
		Select-AzureSubscription $subscriptionName
		
		# Create a Storage account
		New-AzureStorageAccount -StorageAccountName $storageAccountName_Data -location $location
				
		# Create a Blob storage container
		$storageAccountKey = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $storageAccountName_Data –StorageAccountKey $storageAccountKey  
		New-AzureStorageContainer -Name $containerName_Data -Context $destContext

4. 运行以下命令以验证存储帐户和容器：

		Get-AzureStorageAccount -StorageAccountName $storageAccountName_Data
		Get-AzureStorageContainer -Context $destContext

**上载数据文件**

1. 打开 Azure PowerShell。
2. 设置前三个变量，然后运行命令：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName_Data = "<AzureStorageAccountName>"  
		$containerName_Data = "<ContainerName>"

		$localFolder = "C:\hdp\hadoop-2.4.0.2.1.3.0-1981\share\doc\hadoop\common\"
		$destFolder = "WordCount/Input"

	**$storageAccountName_Data** 和 **$containerName_Data** 变量与你在上一个过程中定义的一样。

	请注意，源文件文件夹是 **c:\\Hadoop\\hadoop-1.1.0-SNAPSHOT**，目标文件夹是 **WordCount/Input**。

3. 运行以下命令以获取源文件文件夹中的 .txt 文件的列表：

		# Get a list of the .txt files
		$filesAll = Get-ChildItem $localFolder
		$filesTxt = $filesAll | where {$_.Extension -eq ".txt"}
		
4. 运行以下命令以创建存储上下文对象：

		# Create a storage context object
		Select-AzureSubscription $subscriptionName
		$storageaccountkey = get-azurestoragekey $storageAccountName_Data | %{$_.Primary}
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName_Data -StorageAccountKey $storageaccountkey

5. 运行以下命令以复制文件：

		# Copy the files from the local workstation to the Blob container        
		foreach ($file in $filesTxt){
		 
		    $fileName = "$localFolder\$file"
		    $blobName = "$destFolder/$file"
		
		    write-host "Copying $fileName to $blobName"
		
		    Set-AzureStorageBlobContent -File $fileName -Container $containerName_Data -Blob $blobName -Context $destContext
		}

6. 运行以下命令以列出已上载的文件：

		# List the uploaded files in the Blob storage container
        Write-Host "The Uploaded data files:" -BackgroundColor Green
		Get-AzureStorageBlob -Container $containerName_Data -Context $destContext -Prefix $destFolder

	你会看到有关已上载的文本数据文件的信息。

**上载单词计数应用程序**

1. 打开 Azure PowerShell。
2. 设置前三个变量，然后运行命令：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName_Data = "<AzureStorageAccountName>"  
		$containerName_Data = "<ContainerName>"

		$jarFile = "C:\Tutorials\WordCountJava\wordcountjava\target\wordcountjava-1.0-SNAPSHOT.jar"
		$blobFolder = "WordCount/jars"

	**$storageAccountName_Data** 和 **$containerName_Data** 变量与你在上一个过程中定义的一样，这意味着你要将数据文件和应用程序上载到同一存储帐户上的同一容器中。

	请注意，目标文件夹是 **WordCount/jars**。

4. 运行以下命令以创建存储上下文对象：

		# Create a storage context object
		Select-AzureSubscription $subscriptionName
		$storageaccountkey = get-azurestoragekey $storageAccountName_Data | %{$_.Primary}
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName_Data -StorageAccountKey $storageaccountkey

5. 运行以下命令以复制应用程序：

		Set-AzureStorageBlobContent -File $jarFile -Container $containerName_Data -Blob "$blobFolder/WordCount.jar" -Context $destContext

6. 运行以下命令以列出已上载的文件：

		# List the uploaded files in the Blob storage container
        Write-Host "The Uploaded application file:" -BackgroundColor Green
		Get-AzureStorageBlob -Container $containerName_Data -Context $destContext -Prefix $blobFolder

	你会看到列出的 JAR 文件。

##<a name="run"></a>在 Azure HDInsight 中运行 MapReduce 作业

在此部分中，你将创建一个用于执行以下任务的 Azure PowerShell 脚本：

1. 设置 HDInsight 群集
	
	1. 创建将用作默认 HDInsight 群集文件系统的存储帐户
	2. 创建 Blob 存储容器 
	3. 创建 HDInsight 群集

2. 提交 MapReduce 作业

	1. 创建 MapReduce 作业定义
	2. 提交 MapReduce 作业
	3. 等待作业完成
	4. 显示标准错误
	5. 显示标准输出

3. 删除群集

	1. 删除 HDInsight 群集
	2. 删除用作默认 HDInsight 群集文件系统的存储帐户


**运行 Azure PowerShell 脚本**

1. 打开记事本。
2. 复制并粘贴以下代码：
		
		# The Storage account and the HDInsight cluster variables
		$subscriptionName = "<AzureSubscriptionName>"
		$stringPrefix = "<StringForPrefix>"
		$location = "<MicrosoftDataCenter>"     ### Must match the data Storage account location
		$clusterNodes = <NumberOFNodesInTheCluster>

		$storageAccountName_Data = "<TheDataStorageAccountName>"
		$containerName_Data = "<TheDataBlobStorageContainerName>"
		
		$clusterName = $stringPrefix + "hdicluster"
		
		$storageAccountName_Default = $stringPrefix + "hdistore"
		$containerName_Default =  $stringPrefix + "hdicluster"

		# The MapReduce job variables
		$jarFile = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/jars/WordCount.jar"
		$className = "org.apache.hadoop.examples.WordCount"
		$mrInput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Input/"
		$mrOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/Output/"
		$mrStatusOutput = "wasb://$containerName_Data@$storageAccountName_Data.blob.core.chinacloudapi.cn/WordCount/MRStatusOutput/"
		
		# Create a PSCredential object. The username and password are hardcoded here.  You can change them if you want.
		$password = ConvertTo-SecureString "Pass@word1" -AsPlainText -Force
		$creds = New-Object System.Management.Automation.PSCredential ("Admin", $password) 
		
		Select-AzureSubscription $subscriptionName
		
		#=============================
		# Create a Storage account used as the default file system
		Write-Host "Create a storage account" -ForegroundColor Green
		New-AzureStorageAccount -StorageAccountName $storageAccountName_Default -location $location
		
		#=============================
		# Create a Blob storage container used as the default file system
		Write-Host "Create a Blob storage container" -ForegroundColor Green
		$storageAccountKey_Default = Get-AzureStorageKey $storageAccountName_Default | %{ $_.Primary }
		$destContext = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $storageAccountName_Default –StorageAccountKey $storageAccountKey_Default
		
		New-AzureStorageContainer -Name $containerName_Default -Context $destContext
		
		#=============================
		# Create an HDInsight cluster
		Write-Host "Create an HDInsight cluster" -ForegroundColor Green
		$storageAccountKey_Data = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
		
		$config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes |
		    Set-AzureHDInsightDefaultStorage -StorageAccountName "$storageAccountName_Default.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Default -StorageContainerName $containerName_Default |
		    Add-AzureHDInsightStorage -StorageAccountName "$storageAccountName_Data.blob.core.chinacloudapi.cn" -StorageAccountKey $storageAccountKey_Data
		
		New-AzureHDInsightCluster -Name $clusterName -Location $location -Credential $creds -Config $config
		
		#=============================
		# Create a MapReduce job definition
		Write-Host "Create a MapReduce job definition" -ForegroundColor Green
		$mrJobDef = New-AzureHDInsightMapReduceJobDefinition -JobName mrWordCountJob  -JarFile $jarFile -ClassName $className -Arguments $mrInput, $mrOutput -StatusFolder /WordCountStatus
		
		#=============================
		# Run the MapReduce job
		Write-Host "Run the MapReduce job" -ForegroundColor Green
		$mrJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $mrJobDef 
		Wait-AzureHDInsightJob -Job $mrJob -WaitTimeoutInSeconds 3600 
		
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardError 
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $mrJob.JobId -StandardOutput
				
		#=============================
		# Delete the HDInsight cluster
		Write-Host "Delete the HDInsight cluster" -ForegroundColor Green
		Remove-AzureHDInsightCluster -Name $clusterName  
		
		# Delete the default file system Storage account
		Write-Host "Delete the storage account" -ForegroundColor Green
		Remove-AzureStorageAccount -StorageAccountName $storageAccountName_Default

3. 设置此脚本中的前六个变量。**$stringPrefix** 变量用于对 HDInsight 群集名称、存储帐户名称和 Blob 存储容器名称的指定字符串加上前缀。因为这些项目的名称必须为 3 到 24 个字符，请确保你指定的字符串和此脚本使用的名称，合计不超过名称的字符数限制。**$stringPrefix** 必须全部为小写。
 
	**$storageAccountName_Data** 和 **$containerName_Data** 变量是用于存储数据文件和应用程序的存储帐户和容器。**$location** 变量必须与数据存储帐户位置匹配。

4. 复查其余变量。
5. 保存脚本文件。
6. 打开 Azure PowerShell。
7. 运行以下命令，将执行策略设为 RemoteSigned：

		PowerShell -File <FileName> -ExecutionPolicy RemoteSigned

8. 出现提示时，输入 HDInsight 群集的用户名和密码。由于你将在脚本末尾删除群集，并且将不再需要用户名和密码，因此用户名和密码可以是任何字符串。如果你不想让系统提示你输入凭据，请参阅[在 Windows PowerShell 中使用密码、安全字符串和凭据][powershell-PSCredential]。


##<a name="retrieve"></a>检索 MapReduce 作业输出
本节演示如何下载和显示输出。有关在 Excel 中显示结果的信息，请参阅[使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][hdinsight-ODBC] 和[利用 Power Query 将 Excel 连接到 HDInsight][hdinsight-power-query]。


**检索输出**

1. 打开 Azure PowerShell 窗口。
2. 将目录切换到 **C:\\Tutorials\\WordCountJava**。默认 Azure PowerShell 文件夹是 **C:\\Windows\\System32\\WindowsPowerShell\\v1.0**。你将运行的 cmdlet 会将输出文件下载到当前文件夹。你无权将文件下载到系统文件夹。
2. 运行以下命令以设置值：

		$subscriptionName = "<AzureSubscriptionName>"
		$storageAccountName_Data = "<TheDataStorageAccountName>"
		$containerName_Data = "<TheDataBlobStorageContainerName>"
		$blobName = "WordCount/Output/part-r-00000"
	
3. 运行以下命令以创建 Azure 存储上下文对象：
		
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName_Data | %{ $_.Primary }
		$storageContext = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $storageAccountName_Data –StorageAccountKey $storageAccountKey  

4. 运行以下命令以下载并显示输出：

		Get-AzureStorageBlobContent -Container $containerName_Data -Blob $blobName -Context $storageContext -Force
		cat "./$blobName" | findstr "there"

作业完成后，你可以选择使用 [Sqoop][hdinsight-use-sqoop] 将数据导出到 SQL Server 或 Azure SQL Database，或者将数据导出到 Excel。

##<a id="nextsteps"></a>后续步骤
在本教程中，你已学习如何执行以下操作：开发 Java MapReduce 作业、在 HDInsight Emulator 中测试应用程序、编写 Azure PowerShell 脚本以设置 HDInsight 群集以及在群集上运行 MapReduce 作业。若要了解更多信息，请参阅下列文章：

- [为 HDInsight 开发 C# Hadoop 流式处理 MapReduce 程序][hdinsight-develop-streaming]
- [Azure HDInsight 入门][hdinsight-get-started]
- [HDInsight Emulator 入门][hdinsight-emulator]
- [将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]
- [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
- [将数据上载到 HDInsight][hdinsight-upload-data]
- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [利用 Power Query 将 Excel 连接到 HDInsight][hdinsight-power-query]
 <!--[使用 Microsoft Hive ODBC Driver 将 Excel 连接到 HDInsight][hdinsight-ODBC]-->

[azure-purchase-options]: /pricing/overview/
[azure-trial]: /pricing/1rmb-trial/


[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/

[hdinsight-develop-streaming]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/
[hdinsight-emulator]: /documentation/articles/hdinsight-get-started-emulator/
[hdinsight-emulator-wasb]: /documentation/articles/hdinsight-get-started-emulator/#blobstorage
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/

[powershell-PSCredential]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx
[Powershell-install-configure]: /documentation/articles/powershell-install-configure/



[image-emulator-wordcount-compile]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Compile-Java-MapReduce.png
[image-emulator-wordcount-run]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Run-Java-MapReduce.png

<!---HONumber=66-->