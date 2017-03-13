<properties
    pageTitle="使用 Maven 开发 Scalding MapReduce 作业 | Azure"
    description="了解如何使用 Maven 创建 Scalding MapReduce 作业，然后在 HDInsight 群集的 Hadoop 上部署并运行该作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="26a4d4e8-2623-4fae-a0ca-17792b7a5713"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/19/2017"
    wacn.date="03/10/2017"
    ms.author="larryfr" />

# 使用 HDInsight 上的 Apache Hadoop 开发 Scalding MapReduce 作业
Scalding 是一种 Scala 库，它可以让你轻松地创建 Hadoop MapReduce 作业。它提供简明的语法以及与 Scala 的紧密集成。

在本文档中，你将了解如何使用 Maven 创建以 Scalding 编写的单词计数 MapReduce 作业。然后，你将了解如何在 HDInsight 群集上部署并运行此作业。

## 先决条件
* **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

* **HDInsight 群集上基于 Windows 或 Linux 的 Hadoop**。有关详细信息，请参阅[在 HDInsight 上预配基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/) 或[在 HDInsight 上预配基于 Windows 的 Hadoop](/documentation/articles/hdinsight-provision-clusters/)。

    > [AZURE.IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

* **[Maven](http://maven.apache.org/)**

* **[Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更高版本**

## 创建和生成项目

1. 使用以下命令新建 Maven 项目：
   
        mvn archetype:generate -DgroupId=com.microsoft.example -DartifactId=scaldingwordcount -DarchetypeGroupId=org.scala-tools.archetypes -DarchetypeArtifactId=scala-archetype-simple -DinteractiveMode=false
   
    此命令将新建名为 **scaldingwordcount** 的目录，并创建 Scala 应用程序的基架。

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
   
    * **maven.compiler.source** 和 **maven.compiler.target**：设置此项目的 Java 版本。

    * **repositories**：包含此项目使用的相关文件的存储库。

    * **scalding-core\_2.11** 和 **hadoop-core**：此项目依赖于 Scalding 和 Hadoop 核心程序包。

    * **maven-scala-plugin**：用于编译 scala 应用程序的插件。

    * **maven-shade-plugin**：用于创建阴影 (fat) jar 的插件。此插件将应用筛选器和转换，具体包括：
     
        * **筛选器**：应用的筛选器将修改 jar 文件中包含的元信息。为了防止运行时发生签名异常，它会排除依赖项中可能包含的各种签名文件。

        * **执行**：程序包阶段执行配置将 **com.twitter.scalding.Tool** 类指定为程序包的主类。如果没有此项配置，则你在使用 hadoop 命令运行作业时，需要指定 com.twitter.scalding.Tool 以及包含应用程序逻辑的类。
    
3. 删除 **src/test** 目录，因为你不需要使用此示例创建测试。

4. 打开 **src/main/scala/com/microsoft/example/App.scala** 文件，并将其内容替换为以下内容：
   
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

## 在基于 Linux 的群集上运行作业

> [AZURE.NOTE]
以下步骤使用 SSH 和 Hadoop 命令。有关运行 MapReduce 作业的其他方法，请参阅[在 Hadoop on HDInsight 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

1. 使用以下命令将该程序包上载到你的 HDInsight 群集：
   
        scp target/scaldingwordcount-1.0-SNAPSHOT.jar username@clustername-ssh.azurehdinsight.cn:
   
    这样就会将两个文件从本地系统复制到头节点。
   
    > [AZURE.NOTE]
    如果使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果你使用了 SSH 密钥，你可能必须使用 `-i` 参数和私钥的路径。例如 `scp -i /path/to/private/key target/scaldingwordcount-1.0-SNAPSHOT.jar username@clustername-ssh.azurehdinsight.cn:.`
    > 
    > 
2. 输入以下命令连接到群集头节点：
   
        ssh username@clustername-ssh.azurehdinsight.cn
   
    > [AZURE.NOTE]
    如果你使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果你使用了 SSH 密钥，你可能必须使用 `-i` 参数和私钥的路径。例如 `ssh -i /path/to/private/key username@clustername-ssh.azurehdinsight.cn`

3. 连接到头节点后，使用以下命令运行该单词计数作业
   
        yarn jar scaldingwordcount-1.0-SNAPSHOT.jar com.microsoft.example.WordCount --hdfs --input wasbs:///example/data/gutenberg/davinci.txt --output wasbs:///example/wordcountout
   
    这将会运行前面实现的 WordCount 类。`--hdfs` 指示作业使用 HDFS。`--input` 指定输入文本文件，而 `--output` 指定输出位置。
4. 完成作业后，使用以下命令查看输出。
   
        hdfs dfs -text wasbs:///example/wordcountout/*
   
    这将会显示如下信息：
   
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

## 在基于 Windows 的群集上运行作业

以下步骤使用 Windows PowerShell。有关运行 MapReduce 作业的其他方法，请参阅[在 HDInsight 上的 Hadoop 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

1. 启动 Azure PowerShell 并登录到你的 Azure 帐户。提供凭据后，该命令将返回有关你的帐户的信息。
   
        Add-AzureRMAccount
   
        Id                             Type       ...
        --                             ----
        someone@example.com            User       ...

2. 如果你有多个订阅，请提供要用于部署的订阅 ID。
   
        Select-AzureRMSubscription -SubscriptionID <YourSubscriptionId>
   
    > [AZURE.NOTE]
    可以使用 `Get-AzureRMSubscription` 获取与你的帐户关联的所有订阅列表，包括每个订阅的订阅 ID。

3. 使用以下脚本上传和运行 WordCount 作业。将 `CLUSTERNAME` 替换为 HDInsight 群集的名称，并确保 `$fileToUpload` 是 **scaldingwordcount-1.0-SNAPSHOT.jar** 文件的正确路径。

        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Add-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        # Get cluster info
        $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
        $creds=Get-Credential -Message "Enter the login for the cluster"
        $fileToUpload = Read-Host -Prompt "Enter the path to the scaldingwordcount-1.0-SNAPSHOT.jar file"
        $blobPath = "example/jars/scaldingwordcount-1.0-SNAPSHOT.jar"

        #Get the cluster info so we can get the resource group, storage, etc.
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
            -ResourceGroupName $resourceGroup)[0].Value

        #Create a storage content and upload the file
        $context = New-AzureStorageContext `
            -StorageAccountName $storageAccountName `
            -StorageAccountKey $storageAccountKey

        Set-AzureStorageBlobContent `
            -File $fileToUpload `
            -Blob $blobPath `
            -Container $container `
            -Context $context

        #Create a job definition and start the job
        $jobDef=New-AzureRmHDInsightMapReduceJobDefinition `
            -JobName ScaldingWordCount `
            -JarFile wasbs:///example/jars/scaldingwordcount-1.0-SNAPSHOT.jar `
            -ClassName com.microsoft.example.WordCount `
            -arguments "--hdfs", `
                        "--input", `
                        "wasbs:///example/data/gutenberg/davinci.txt", `
                        "--output", `
                        "wasbs:///example/wordcountout"
        $job = Start-AzureRmHDInsightJob `
            -clustername $clusterName `
            -jobdefinition $jobDef `
            -HttpCredential $creds
        Write-Output "Job ID is: $job.JobId"
        Wait-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds
        #Download the output of the job
        Get-AzureStorageBlobContent `
            -Blob example/wordcountout/part-00000 `
            -Container $container `
            -Destination output.txt `
            -Context $context
        #Log any errors that occured
        Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

     运行该脚本时，系统将提示你输入 HDInsight 群集的管理员用户名和密码。运行作业时发生的任何错误都将记录到控制台中。

4. 完成作业后，输出将下载到当前目录中的 **output.txt** 文件。使用以下命令显示结果。
   
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
现在，你已了解如何使用 Scalding 创建适用于 HDInsight 的 MapRedcue 作业，请使用以下链接探索 Azure HDInsight 的其他用法。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: add information about HDInsight Windows is going to be abandoned-->