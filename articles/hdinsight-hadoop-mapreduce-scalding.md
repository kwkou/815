<properties
 pageTitle="使用 Maven 开发 Scalding MapReduce 作业 | Azure"
 description="了解如何使用 Maven 创建 Scalding MapReduce 作业，然后在 Hadoop on HDInsight 群集上部署并运行该作业。"
 services="hdinsight"
 documentationCenter=""
 authors="Blackmist"
 manager="paulettm"
 editor="cgronlun"
	tags="azure-portal"/>
<tags
	ms.service="hdinsight"
	ms.date="04/26/2016"
	wacn.date="06/29/2016"/>

# 使用 Apache Hadoop on HDInsight 开发 Scalding MapReduce 作业

Scalding 是一种 Scala 库，它可以让你轻松地创建 Hadoop MapReduce 作业。它提供简明的语法以及与 Scala 的紧密集成。

在本文档中，你将学习如何使用 Maven 来创建以 Scalding 编写的单词计数 MapReduce 作业。然后，你将学习如何在 HDInsight 群集上部署并运行此作业。

## 先决条件

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。
* **HDInsight 群集上基于 Windows 的 Hadoop**。有关详细信息，请参阅[在 HDInsight 上预配基于 Windows 的 Hadoop](/documentation/articles/hdinsight-provision-clusters-v1/)。

* **[Maven](http://maven.apache.org/)**

* **[Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更高版本**

## 创建和生成项目

1. 输入以下命令来创建新的 Maven 项目：

        mvn archetype:generate -DgroupId=com.microsoft.example -DartifactId=scaldingwordcount -DarchetypeGroupId=org.scala-tools.archetypes -DarchetypeArtifactId=scala-archetype-simple -DinteractiveMode=false

    此命令将创建一个名为 **scaldingwordcount** 的新目录，并创建 Scala 应用程序的基架。

2. 在 **scaldingwordcount** 目录中，打开 **pom.xml** 文件并将其内容替换为以下内容：

        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <groupId>com.microsoft.example</groupId>
          <artifactId>scaldingwordcount</artifactId>
          <version>1.0-SNAPSHOT</version>
          <name>${project.artifactId}</name>
          <properties>
            <maven.compiler.source>1.6</maven.compiler.source>
            <maven.compiler.target>1.6</maven.compiler.target>
            <encoding>UTF-8</encoding>
          </properties>
          <repositories>
            <repository>
              <id>conjars</id>
              <url>http://conjars.org/repo</url>
            </repository>
            <repository>
              <id>maven-central</id>
              <url>http://repo1.maven.org/maven2</url>
            </repository>
          </repositories>
          <dependencies>
            <dependency>
              <groupId>com.twitter</groupId>
              <artifactId>scalding-core_2.11</artifactId>
              <version>0.13.1</version>
            </dependency>
            <dependency>
              <groupId>org.apache.hadoop</groupId>
              <artifactId>hadoop-core</artifactId>
              <version>1.2.1</version>
              <scope>provided</scope>
            </dependency>
          </dependencies>
          <build>
            <sourceDirectory>src/main/scala</sourceDirectory>
            <plugins>
              <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                  <execution>
                    <id>scala-compile-first</id>
                    <phase>process-resources</phase>
                    <goals>
                      <goal>add-source</goal>
                      <goal>compile</goal>
                    </goals>
                  </execution>
                </executions>
              </plugin>
              <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                  <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                    </transformer>
                  </transformers>
                  <filters>
                    <filter>
                      <artifact>*:*</artifact>
                      <excludes>
                        <exclude>META-INF/*.SF</exclude>
                        <exclude>META-INF/*.DSA</exclude>
                        <exclude>META-INF/*.RSA</exclude>
                      </excludes>
                    </filter>
                  </filters>
                </configuration>
                <executions>
                  <execution>
                    <phase>package</phase>
                    <goals>
                      <goal>shade</goal>
                    </goals>
                    <configuration>
                      <transformers>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                          <mainClass>com.twitter.scalding.Tool</mainClass>
                        </transformer>
                      </transformers>
                    </configuration>
                  </execution>
                </executions>
              </plugin>
            </plugins>
          </build>
        </project>

    此文件描述了项目、依赖关系和插件。以下是重要条目：

    * **maven.compiler.source** 和 **maven.compiler.target**：设置此项目的 Java 版本

    * **repositories**：包含此项目使用的依赖文件的存储库

    * **scalding-core\_2.11** 和 **hadoop-core**：此项目依赖于 Scalding 和 Hadoop 核心程序包

    * **maven-scala-plugin**：用于编译 scala 应用程序的插件

    * **maven-shade-plugin**：用于创建阴影 (fat) jar 的插件。此插件将应用筛选器和转换，具体包括：

        * **筛选器**：应用的筛选器将修改 jar 文件中包含的元信息。为了防止运行时发生签名异常，它会排除依赖项中可能包含的各种签名文件。

        * **执行**：程序包阶段执行配置将 **com.twitter.scalding.Tool** 类指定为程序包的主类。如果没有此项配置，则你在使用 hadoop 命令运行作业时，需要指定 com.twitter.scalding.Tool 以及包含应用程序逻辑的类。

3. 删除 **src/test** 目录，因为你不需要使用此示例创建测试。

4. 打开 **src/main/scala/com/microsoft/example/app.scala** 文件，并将其内容替换为以下内容：

        package com.microsoft.example

        import com.twitter.scalding._

        class WordCount(args : Args) extends Job(args) {
          // 1. Read lines from the specified input location
          // 2. Extract individual words from each line
          // 3. Group words and count them
          // 4. Write output to the specified output location
          TextLine(args("input"))
            .flatMap('line -> 'word) { line : String => tokenize(line) }
            .groupBy('word) { _.size }
            .write(Tsv(args("output")))

          //Tokenizer to split sentance into words
          def tokenize(text : String) : Array[String] = {
            text.toLowerCase.replaceAll("[^a-zA-Z0-9\\s]", "").split("\\s+")
          }
        }

    这将会实现基本的单词计数作业。

5. 保存并关闭文件。

6. 使用以下命令从 **scaldingwordcount** 目录生成并打包应用程序：

        mvn package

    完成此作业后，可以在 **target/scaldingwordcount-1.0-SNAPSHOT.jar** 中找到包含 WordCount 应用程序的程序包。

## 在基于 Windows 的群集上运行作业

> [AZURE.NOTE]以下步骤使用 Windows PowerShell。有关运行 MapReduce 作业的其他方法，请参阅[在 HDInsight 上的 Hadoop 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

1. [安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

2. 下载 [hdinsight-tools.psm1](https://github.com/Blackmist/hdinsight-tools/blob/master/hdinsight-tools.psm1) 并将其保存到名为 **hdinsight-tools.psm1** 的文件。

3. 打开一个新的 **Azure PoweShell** 会话并输入以下命令。如果 hdinsight-tools.psm1 不在当前目录中，请提供该文件的路径：

        import-module hdinsight-tools.psm1

    这将会导入多个函数用于处理 HDInsight 中的文件。

4. 使用以下命令上载包含 WordCount 作业的 jar 文件。将 `CLUSTERNAME` 替换为 HDInsight 群集的名称。

        $clusterName="CLUSTERNAME"
        Add-HDInsightFile -clusterName $clusterName -localPath \path\to\scaldingwordcount-1.0-SNAPSHOT.jar -destinationPath example/jars/scaldingwordcount-1.0-SNAPSHOT.jar

5. 完成上载后，使用以下命令运行作业：

        $jobDef=New-AzureHDInsightMapReduceJobDefinition -JobName ScaldingWordCount -JarFile wasb:///example/jars/scaldingwordcount-1.0-SNAPSHOT.jar -ClassName com.microsoft.example.WordCount -arguments "--hdfs", "--input", "wasb:///example/data/gutenberg/davinci.txt", "--output", "wasb:///example/wordcountout"
        $job = start-azurehdinsightjob -cluster $clusterName -jobdefinition $jobDef
        wait-azurehdinsightjob -Job $job -waittimeoutinseconds 3600

6. 完成作业后，使用以下命令下载作业输出：

        Get-HDInsightFile -clusterName $clusterName -remotePath example/wordcountout/part-00000 -localPath output.txt

7. 输出中包括制表符分隔的单词和计数值。使用以下命令以显示结果。

        cat output.txt

    该文件应包含如下所示的值：

        writers 9
        writes  18
        writhed 1
        writing 51
        writings        24
        written 208
        writtenthese    1
        wrong   11
        wrongly 2
        wrongplace      1
        wrote   34
        wrotefootnote   1
        wrought 7

## 后续步骤

现在，你已学习如何使用 Scalding 来创建适用于 HDInsight 的 MapRedcue 作业，接下来请使用以下链接学习 Azure HDInsight 的其他用法。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0104_2016-->