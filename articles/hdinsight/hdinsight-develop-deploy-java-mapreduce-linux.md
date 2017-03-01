<properties
	pageTitle="为 Hadoop 开发 Java MapReduce 程序 | Microsoft Azure"
	description="了解如何在 HDInsight Emulator 上开发 Java MapReduce 程序，以及如何将这些程序部署到 HDInsight。"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	authors="Blackmist"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="10/11/2016"
	wacn.date="02/14/2017"/>

# 为 HDInsight 中的 Hadoop 开发 Java MapReduce 程序

本文档介绍如何使用 Apache Maven 来创建 MapReduce 应用程序，然后在 HDInsight 群集中基于 Linux 的 Hadoop 上部署和运行它。

##<a name="prerequisites"></a>先决条件

在开始阅读本教程前，你必须具有：

- [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 7 或更高版本（或类似版本，如 OpenJDK）

- [Apache Maven](http://maven.apache.org/)

- **Azure 订阅**

- **Azure CLI**

##配置环境变量

可以在安装 Java 和 JDK 时设置以下环境变量。不过，你应该检查它们是否存在并且包含系统的正确值。

* **JAVA\_HOME** - 应该指向已安装 Java 运行时环境 (JRE) 的目录。例如，在 OS X、Unix 或 Linux 系统中，它的值应该类似于 `/usr/lib/jvm/java-7-oracle`。在 Windows 中，它的值类似于 `c:\Program Files (x86)\Java\jre1.7`

* **PATH** - 应该包含以下路径：

	* **JAVA\_HOME**（或等效的路径）

	* **JAVA\_HOME\\bin**（或等效的路径）

	* 安装 Maven 的目录

##创建新的 Maven 项目

1. 在开发环境中，通过中断会话或命令行将目录更改为要存储此项目的位置。

3. 使用随同 Maven 一起安装的 __mvn__ 命令，为项目生成基架。

		mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

	这将在当前目录中创建一个新目录，该目录使用 __artifactID__ 参数指定的名称（在本示例中为 **wordcountjava**）。 此目录将包含以下项：

	* __pom.xml__ - [项目对象模型 (POM)](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)，其中包含用于生成项目的信息和配置详细信息。

	* __src__ - 包含 __main/java/org/apache/hadoop/examples__ 目录的目录，你将在其中创作应用程序。

3. 删除 __src/test/java/org/apache/hadoop/examples/apptest.java__ 文件，因为本示例用不到该文件。

##添加依赖项

1. 编辑__pom.xml__ 文件，并在 `<dependencies>` 节中添加以下代码。

		<dependency>
		  <groupId>org.apache.hadoop</groupId>
	      <artifactId>hadoop-mapreduce-examples</artifactId>
	      <version>2.5.1</version>
		  <scope>provided</scope>
	    </dependency>
		<dependency>
	  	  <groupId>org.apache.hadoop</groupId>
	  	  <artifactId>hadoop-mapreduce-client-common</artifactId>
	  	  <version>2.5.1</version>
		  <scope>provided</scope>
		</dependency>
		<dependency>
	  	  <groupId>org.apache.hadoop</groupId>
	  	  <artifactId>hadoop-common</artifactId>
	  	  <version>2.5.1</version>
		  <scope>provided</scope>
		</dependency>

	这将告知 Maven，项目需要具有特定版本（在 <version> 中列出）的库（在 <artifactId> 中列出）。在编译时，将从默认的 Maven 存储库下载该版本。你可以使用 [Maven 存储库搜索](http://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar)来查看详细信息。

	`<scope>provided</scope>` 告诉 Maven 这些依赖关系不应与此应用程序一起打包，因为它们将在运行时由 HDInsight 群集提供。

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
			<plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
               <source>1.7</source>
               <target>1.7</target>
              </configuration>
            </plugin>
  		  </plugins>
	    </build>

	第一个插件配置 [Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)，用于生成 uberjar（有时称为 fatjar），其中包含应用程序所需的依赖项。它还可以防止在 jar 包中复制许可证，复制许可证在某些系统中可能会导致问题。

	第二个插件配置 Maven 编译器，用于将此应用程序所需的 Java 版本设置为在 HDInsight 群集上使用的版本。

3. 保存 __pom.xml__ 文件。

##创建 MapReduce 应用程序

1. 转到 __wordcountjava/src/main/java/org/apache/hadoop/examples__ 目录，并将 __App.java__ 文件重命名为 __WordCount.java__。

2. 在文本编辑器中打开 __WordCount.java__ 文件，然后将其内容替换为以下内容：

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

##构建应用程序

1. 更改为 __wordcountjava__ 目录（如果你尚未到达此目录）。

2. 使用以下命令生成包含该应用程序的 JAR 文件：

		mvn clean package

	这将会清除以前的所有生成项目，下载尚未安装的所有依赖项，然后生成并打包应用程序。

3. 完成该命令后，__wordcountjava/target__ 目录将包含名为__wordcountjava-1.0-SNAPSHOT.jar__ 的文件。

	> [AZURE.NOTE]__wordcountjava-1.0-SNAPSHOT.jar__ 文件是一种 uberjar，其中不仅包含 WordCount 作业，还包含该作业在运行时需要的依赖关系。


##<a id="upload"></a>上载该 jar

使用以下命令将该 jar 文件上载到 HDInsight 头节点：

	scp wordcountjava-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:

	Replace __USERNAME__ with your SSH user name for the cluster. Replace __CLUSTERNAME__ with the HDInsight cluster name.

这样就会将两个文件从本地系统复制到头节点。

> [AZURE.NOTE]如果你使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果你使用了 SSH 密钥，你可能必须使用 `-i` 参数和私钥的路径。例如，`scp -i /path/to/private/key wordcountjava-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:`。

##<a name="run"></a>运行 MapReduce 作业

1. 使用 SSH 连接到 HDInsight，如以下文章所述：

    - [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)

    - [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)

2. 在 SSH 会话中，使用以下命令运行 MapReduce 应用程序：

		hadoop jar wordcountjava.jar org.apache.hadoop.examples.WordCount wasbs:///example/data/gutenberg/davinci.txt wasbs:///example/data/wordcountout

	此时会使用 WordCount MapReduce 应用程序来计算 davinci.txt 文件中的字数，并将结果保存到 __wasbs:///example/data/wordcountout__。输入文件和输出均存储到群集的默认存储中。

3. 完成作业后，使用以下命令查看结果：

		hadoop fs -cat wasbs:///example/data/wordcountout/*

	你会收到包含单词和计数的列表，其值类似于以下值：

		zeal    1
		zelus   1
		zenith  2

##<a id="nextsteps"></a>后续步骤

在本文档中，你学习了如何开发 Java MapReduce 作业。请参阅以下文档，了解使用 HDInsight 的其他方式。

- [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
- [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
- [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/


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
[powershell-install-configure]: /documentation/articles/install-configure-powershell/



[image-emulator-wordcount-compile]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Compile-Java-MapReduce.png
[image-emulator-wordcount-run]: ./media/hdinsight-develop-deploy-java-mapreduce/HDI-Emulator-Run-Java-MapReduce.png

<!---HONumber=74-->