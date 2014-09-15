<properties title="Build an HBase application using Maven" pageTitle="Build an HBase application using Maven" description="Learn how to use Apache Maven to build a Java-based Apache HBase application, then deploy it to Azure HDInsight" metaKeywords="Maven hbase hadoop, hbase hadoop, maven java hbase, maven java hbase hadoop, maven java hadoop, hbase hdinsight, hbase java hdinsight, maven hdinsight, maven java hdinsight, hadoop database, hdinsight database" services="hdinsight" solutions="big-data" documentationCenter="" authors="larryfr" videoId="" scriptId="" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="08/21/2014" ms.author="larryfr"></tags>

## 借助 Maven 生成可将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序

了解如何使用 Apache Maven 创建并生成一个 [Apache HBase](http://hbase.apache.org/) Java 应用程序。然后，在 Azure HDInsight (Hadoop) 上使用该应用程序。

[Maven][] 是一个软件项目管理和分析工具，可让你生成 Java 项目的软件、文档和报告。在本文中，你将要了解如何使用 Maven 创建一个基本的 Java 应用程序，该应用程序可在 Azure HDInsight 群集中创建、查询和删除 HBase 表。

## 要求

-   [Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更高版本

-   [Maven](http://maven.apache.org/)

-   [一个包含 HBase 的 Azure HDInsight 群集](/en-us/documentation/articles/hdinsight-hbase-get-started/#create-hbase-cluster)

## 创建项目

1.  在开发环境中的命令行下，将目录切换到要在其中创建项目的位置。例如，`cd code\hdinsight`

2.  使用随 Maven 一起安装的 **mvn** 命令来生成项目的基架。

        mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    这将在当前目录中创建一个新目录，该目录使用 **artifactID** 参数指定的名称（在本示例中为 **hbaseapp**）。此目录将包含以下项。

    -   **pom.xml** - 项目对象模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)) 包含用于生成项目的信息和配置详细信息

    -   **src** - 包含 **main\\java\\com\\microsoft\\examples** 目录的目录，你将在其中创作应用程序。

3.  删除 **src\\test\\java\\com\\microsoft\\examples\\apptest.java** 文件，因为本示例用不到该文件。

## 更新项目对象模型

1.  编辑 **pom.xml** 文件，并在`<dependencies>` 节中添加以下代码。

        <dependency>
          <groupId>org.apache.hbase</groupId>
          <artifactId>hbase-client</artifactId>
          <version>0.98.4-hadoop2</version>
        </dependency>

    这将会告知 Maven，项目需要 **hbase-client** 版本 **0.98.4-hadoop2**。在编译时，将从默认的 Maven 存储库下载该版本。你可以使用 [Maven 存储库搜索](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar)来查看有关此依赖项的详细信息。

2.  将以下代码添加到 **pom.xml** 文件。必须将这些内容放置在文件中的`<project>...</project>` 标记内部；例如，介于 `</dependencies>` 与 `</project>`.

        <build>
          <sourceDirectory>src</sourceDirectory>
          <resources>
            <resource>
              <directory>${basedir}/conf</directory>
              <filtering>false</filtering>
              <includes>
                <include>hbase-site.xml</include>
              </includes>
            </resource>
          </resources>
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

    这将会配置一个包含 HBase 配置信息的资源 (**conf\\hbase-site.xml**)。

    > [WACOM.NOTE] 你也可以通过代码设置配置值。请参阅以下 **CreateTable** 示例中的注释，以了解如何操作。

    这还会配置 [maven-shade-plugin](http://maven.apache.org/plugins/maven-shade-plugin/)，用于防止 Maven 生成的 JAR 中发生许可证重复。使用此插件的原因在于，重复的许可证文件会导致 HDInsight 群集在运行时出错。将 maven-shade-plugin 与 `ApacheLicenseResourceTransformer` 实现一起使用可防止此错误。

    maven-shade-plugin 还将生成 uberjar（或 fatjar），其中包含应用程序所需的所有依赖项。

3.  保存 **pom.xml** 文件。

4.  在 **hbaseapp** 目录中创建名为 **conf** 的新目录。在 **conf** 目录中，创建名为 **hbase-site.xml** 的新文件，并在其中使用以下内容。

        <?xml version="1.0"?>
        <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
        <!--
        /**
         * Copyright 2010 The Apache Software Foundation
         *
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
        -->
        <configuration>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
          </property>
          <property>
            <name>hbase.zookeeper.quorum</name>
            <value>zookeepernode0:2181 zookeepernode1:2181 zookeepernode2:2181</value>
          </property>

        </configuration>

    此文件将用于加载 HDInsight 群集的 HBase 配置。

    > [WACOM.NOTE] 这是一个非常精简的 hbase-site.xml 文件，它只包含 HDInsight 群集的最少量设置。有关 HDInsight 使用的 hbase-site.xml 配置文件的完整版本，请参阅[使用远程桌面连接到 HDInsight 群集](http://azure.microsoft.com/en-us/documentation/articles/hdinsight-administer-use-management-portal/#rdp)。hbase-site.xml 文件位于 C:\\apps\\dist\\hbase-\<版本号\>-hadoop2\\conf 目录中。在群集上更新 HBase 后，文件路径的版本号部分将发生变化。

5.  保存 **hbase-site.xml** 文件。

## 创建应用程序

1.  转到 **hbaseapp\\src\\com\\microsoft\\examples** 目录，然后将 app.java\_\_ 文件重命名为 **CreateTable.java**。

2.  打开 **CreateTable.java** 文件，并将现有内容替换为以下内容。

        package com.microsoft.examples;
        import java.io.IOException;

        import org.apache.hadoop.conf.Configuration;
        import org.apache.hadoop.hbase.HBaseConfiguration;
        import org.apache.hadoop.hbase.client.HBaseAdmin;
        import org.apache.hadoop.hbase.HTableDescriptor;
        import org.apache.hadoop.hbase.TableName;
        import org.apache.hadoop.hbase.HColumnDescriptor;
        import org.apache.hadoop.hbase.client.HTable;
        import org.apache.hadoop.hbase.client.Put;
        import org.apache.hadoop.hbase.util.Bytes;

        public class CreateTable {
          public static void main(String[] args) throws IOException {
            Configuration config = HBaseConfiguration.create();

            // Example of setting zookeeper values for HDInsight
            //   in code instead of an hbase-site.xml file
            //
            // config.set("hbase.zookeeper.quorum",
            //            "zookeepernode0,zookeepernode1,zookeepernode2");
            //config.set("hbase.zookeeper.property.clientPort", "2181");
            //config.set("hbase.cluster.distributed", "true");

            // create an admin object using the config
            HBaseAdmin admin = new HBaseAdmin(config);

            // create the table...
            HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf("people"));
            // ... with two column families
            tableDescriptor.addFamily(new HColumnDescriptor("name"));
            tableDescriptor.addFamily(new HColumnDescriptor("contactinfo"));
            admin.createTable(tableDescriptor);

            // define some people
            String[][] people = {
                { "1", "Marcel", "Haddad", "marcel@fabrikam.com"},
                { "2", "Franklin", "Holtz", "franklin@contoso.com" },
                { "3", "Dwayne", "McKee", "dwayne@fabrikam.com" },
                { "4", "Rae", "Schroeder", "rae@contoso.com" },
                { "5", "Rosalie", "burton", "rosalie@fabrikam.com"},
                { "6", "Gabriela", "Ingram", "gabriela@contoso.com"} };

            HTable table = new HTable(config, "people");

            // Add each person to the table
            //   Use the `name` column family for the name
            //   Use the `contactinfo` column family for the email
            for (int i = 0; i< people.length; i++) {
              Put person = new Put(Bytes.toBytes(people[i][0]));
              person.add(Bytes.toBytes("name"), Bytes.toBytes("first"), Bytes.toBytes(people[i][1]));
              person.add(Bytes.toBytes("name"), Bytes.toBytes("last"), Bytes.toBytes(people[i][2]));
              person.add(Bytes.toBytes("contactinfo"), Bytes.toBytes("email"), Bytes.toBytes(people[i][3]));
              table.put(person);
            }
            // flush commits and close the table
            table.flushCommits();
            table.close();
          }
        }

    这是 **CreateTable** 类，它将创建名为 **people** 的表，并在该表中填充一些预定义的用户。

3.  保存 **CreateTable.java** 文件。

4.  在 **hbaseapp\\com\\microsoft\\examples** 目录中，创建名为 **SearchByEmail.java** 的新文件。在此文件中使用以下内容。

        package com.microsoft.examples;
        import java.io.IOException;

        import org.apache.hadoop.conf.Configuration;
        import org.apache.hadoop.hbase.HBaseConfiguration;
        import org.apache.hadoop.hbase.client.HTable;
        import org.apache.hadoop.hbase.client.Scan;
        import org.apache.hadoop.hbase.client.ResultScanner;
        import org.apache.hadoop.hbase.client.Result;
        import org.apache.hadoop.hbase.filter.RegexStringComparator;
        import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
        import org.apache.hadoop.hbase.filter.CompareFilter.CompareOp;
        import org.apache.hadoop.hbase.util.Bytes;
        import org.apache.hadoop.util.GenericOptionsParser;

        public class SearchByEmail {
          public static void main(String[] args) throws IOException {
            Configuration config = HBaseConfiguration.create();

            // Use GenericOptionsParser to get only the parameters to the class
            // and not all the parameters passed (when using WebHCat for example)
            String[] otherArgs = new GenericOptionsParser(config, args).getRemainingArgs();
            if (otherArgs.length != 1) {
              System.out.println("usage: [regular expression]");
              System.exit(-1);
            }

            // Open the table
            HTable table = new HTable(config, "people");

            // Define the family and qualifiers to be used
            byte[] contactFamily = Bytes.toBytes("contactinfo");
            byte[] emailQualifier = Bytes.toBytes("email");
            byte[] nameFamily = Bytes.toBytes("name");
            byte[] firstNameQualifier = Bytes.toBytes("first");
            byte[] lastNameQualifier = Bytes.toBytes("last");

            // Create a new regex filter
            RegexStringComparator emailFilter = new RegexStringComparator(otherArgs[0]);
            // Attach the regex filter to a filter
            //   for the email column
            SingleColumnValueFilter filter = new SingleColumnValueFilter(
              contactFamily,
              emailQualifier,
              CompareOp.EQUAL,
              emailFilter
            );

            // Create a scan and set the filter
            Scan scan = new Scan();
            scan.setFilter(filter);

            // Get the results
            ResultScanner results = table.getScanner(scan);
            // Iterate over results and print  values
            for (Result result : results ) {
              String id = new String(result.getRow());
              byte[] firstNameObj = result.getValue(nameFamily, firstNameQualifier);
              String firstName = new String(firstNameObj);
              byte[] lastNameObj = result.getValue(nameFamily, lastNameQualifier);
              String lastName = new String(lastNameObj);
              System.out.println(firstName + " " + lastName + " - ID: " + id);
              byte[] emailObj = result.getValue(contactFamily, emailQualifier);
              String email = new String(emailObj);
              System.out.println(firstName + " " + lastName + " - " + email + " - ID: " + id);
            }
            results.close();
            table.close();
          }
        }

    **SearchByEmail** 类可用于按电子邮件地址查询行。由于它使用正则表达式筛选器，因此，在使用该类时，你可以提供字符串或正则表达式。

5.  保存 **SearchByEmail.java** 文件。

6.  在 **hbaseapp\\com\\microsoft\\examples** 目录中，创建名为 **DeleteTable.java** 的新文件。在此文件中使用以下内容。

        package com.microsoft.examples;
        import java.io.IOException;

        import org.apache.hadoop.conf.Configuration;
        import org.apache.hadoop.hbase.HBaseConfiguration;
        import org.apache.hadoop.hbase.client.HBaseAdmin;

        public class DeleteTable {
          public static void main(String[] args) throws IOException {
            Configuration config = HBaseConfiguration.create();

            // Create an admin object using the config
            HBaseAdmin admin = new HBaseAdmin(config);

            // Disable, and then delete the table
            admin.disableTable("people");
            admin.deleteTable("people");
          }
        }

    此类只是用于清理本示例，它的工作方式是先禁用 **CreateTable** 类创建的表，然后再将其删除。

7.  保存 **CreateTable.java** 文件。

## 生成并打包应用程序

1.  打开命令提示符，并将目录切换到 **hbaseapp** 目录。

2.  使用以下命令生成包含该应用程序的 JAR。

        mvn clean package

    这将会清除以前的所有生成项目，下载尚未安装的所有依赖项，然后生成并打包应用程序。

3.  完成该命令后，**hbaseapp\\target** 目录将包含名为 **hbaseapp-1.0-SNAPSHOT.jar** 的文件。

    > [WACOM.NOTE] **hbaseapp-1.0-SNAPSHOT.jar** 文件是一个 uberjar（有时称为 fatjar），其中包含运行应用程序所需的所有依赖项。

## 上载 JAR 并启动作业

> [WACOM.NOTE] 可以根据[在 HDInsight 中上载 Hadoop 作业的数据](/en-us/documentation/articles/hdinsight-upload-data/)中所述，通过许多方式将文件上载到 HDInsight 群集。下面的步骤使用了 [Azure PowerShell](/en-us/documentation/articles/install-configure-powershell/)。

1.  安装并配置 [Azure PowerShell](/en-us/documentation/articles/install-configure-powershell/) 后，请创建名为 **hbase-runner.psm1** 的新文件。在此文件中使用以下内容。

        <#
        .SYNOPSIS
            Copies a file to the primary storage of an HDInsight cluster.
        .DESCRIPTION
            Copies a file from a local directory to the blob container for
            the HDInsight cluster.
        .EXAMPLE
            Start-HBaseExample -className "com.microsoft.examples.CreateTable"
                -clusterName "MyHDInsightCluster"

        .EXAMPLE
            Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
                -clusterName "MyHDInsightCluster"
                -emailRegex "contoso.com"

        .EXAMPLE
            Start-HBaseExample -className "com.microsoft.examples.SearchByEmail"
                -clusterName "MyHDInsightCluster"
                -emailRegex "^r" -showErr
        #>

        function Start-HBaseExample {
            [CmdletBinding(SupportsShouldProcess = $true)]
            param(
                #The class to run
                [Parameter(Mandatory = $true)]
                [String]$className,

                #The name of the HDInsight cluster
                [Parameter(Mandatory = $true)]
                [String]$clusterName,

                #Only used when using SearchByEmail
                [Parameter(Mandatory = $false)]
                [String]$emailRegex,

                #Use if you want to see stderr output
                [Parameter(Mandatory = $false)]
                [Switch]$showErr
            )

            Set-StrictMode -Version 3

            # Is the Azure module installed?
            FindAzure

            # The JAR
            $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

            # The job definition
            $jobDefinition = New-AzureHDInsightMapReduceJobDefinition `
              -JarFile $jarFile `
              -ClassName $className `
              -Arguments $emailRegex

            # Get the job output
            $job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
            Write-Host "Wait for the job to complete ..." -ForegroundColor Green
            Wait-AzureHDInsightJob -Job $job
            if($showErr)
            {
                Write-Host "STDERR"
                Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardError
            }
            Write-Host "Display the standard output ..." -ForegroundColor Green
            Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput
        }

        <#
        .SYNOPSIS
            Copies a file to the primary storage of an HDInsight cluster.
        .DESCRIPTION
            Copies a file from a local directory to the blob container for
            the HDInsight cluster.
        .EXAMPLE
            Add-HDInsightFile -localPath "C:\temp\data.txt"
                -destinationPath "example/data/data.txt"
                -ClusterName "MyHDInsightCluster"
        .EXAMPLE
            Add-HDInsightFile -localPath "C:\temp\data.txt"
                -destinationPath "example/data/data.txt"
                -ClusterName "MyHDInsightCluster"
                -Container "MyContainer"
        #>

        function Add-HDInsightFile {
            [CmdletBinding(SupportsShouldProcess = $true)]
            param(
                #The path to the local file.
                [Parameter(Mandatory = $true)]
                [String]$localPath,

                #The destination path and file name, relative to the root of the container.
                [Parameter(Mandatory = $true)]
                [String]$destinationPath,

                #The name of the HDInsight cluster
                [Parameter(Mandatory = $true)]
                [String]$clusterName,

                #If specified, overwrites existing files without prompting
                [Parameter(Mandatory = $false)]
                [Switch]$force
            )

            Set-StrictMode -Version 3

            # Is the Azure module installed?
            FindAzure

            # Does the local path exist?
            if (-not (Test-Path $localPath))
            {
                throw "Source path '$localPath' does not exist."
            }

            # Get the primary storage container
            $storage = GetStorage -clusterName $clusterName

            # Upload file to storage, overwriting existing files if -force was used.
            Set-AzureStorageBlobContent -File $localPath -Blob $destinationPath -force:$force `
                                        -Container $storage.container `
                                        -Context $storage.context
        }

        function FindAzure {
            # Is the Azure module installed?
            if (-not(Get-Module -ListAvailable Azure))
            {
                throw "Windows Azure PowerShell not found! For help, see http://www.windowsazure.com/zh-cn/documentation/articles/install-configure-powershell/"
            }

            # Is there an active Azure subscription?
            $sub = Get-AzureSubscription -ErrorAction SilentlyContinue
            if(-not($sub))
            {
                throw "No active Azure subscription found! If you have a subscription, use the Add-AzureAccount or Import-PublishSettingsFile cmdlets to make the Azure account available to Windows PowerShell."
            }
        }

        function GetStorage {
            param(
                [Parameter(Mandatory = $true)]
                [String]$clusterName
            )
            $hdi = Get-AzureHDInsightCluster -name $clusterName
            # Does the cluster exist?
            if (!$hdi)
            {
                throw "HDInsight cluster '$clusterName' does not exist."
            }
            # Create a return object for context & container
            $return = @{}
            $storageAccounts = @{}
            # Get the primary storage account information
            $storageAccountName = $hdi.DefaultStorageAccount.StorageAccountName.Split(".",2)[0]
            $storageAccountKey = $hdi.DefaultStorageAccount.StorageAccountKey
            # Build the hash of storage account name/keys
            $storageAccounts.Add($hdi.DefaultStorageAccount.StorageAccountName, $storageAccountKey)
            foreach($account in $hdi.StorageAccounts)
            {
                $storageAccounts.Add($account.StorageAccountName, $account.StorageAccountKey)
            }
            # Get the storage context, as we can't depend
            # on using the default storage context
            $return.context = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
            # Get the container, so we know where to
            # find/store blobs
            $return.container = $hdi.DefaultStorageAccount.StorageContainerName
            # Return storage accounts to support finding all accounts for
            # a cluster
            $return.storageAccounts = $storageAccounts

            return $return
        }
        # Only export the verb-phrase things
        export-modulemember *-*

    此文件包含两个模块：

    -   **Add-HDInsightFile** - 用于将文件上载到 HDInsight

    -   **Start-HBaseExample** - 用于运行前面创建的类

2.  保存 **hbase-runner.psm1** 文件。

3.  打开新的 Azure PowerShell 窗口，将目录切换到 **hbaseapp** 目录，然后运行以下命令。

        PS C:\ Import-Module c:\path\to\hbase-runner.psm1

    将路径切换到前面创建的 **hbase-runner.psm1** 文件所在的位置。这将会注册此 PowerShell 会话的模块。

4.  使用以下命令将 **hbaseapp-1.0-SNAPSHOT.jar** 上载到你的 HDInsight 群集。

        Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName hdinsightclustername

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。然后，该命令会将 **hbaseapp-1.0-SNAPSHOT.jar** 上载到 HDInsight 群集主存储中的 **example/jars** 位置。

5.  上载文件后，请使用以下命令来创建使用 **hbaseapp** 的新表

        Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName hdinsightclustername

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。

    此命令将在 HDInsight 群集上创建名为 **people** 的新表。

6.  若要搜索该表中的条目，请使用以下命令。

        Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName hdinsightclustername -emailRegex contoso.com

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。

    这将会使用 SearchByEmail 类来搜索其中的列 **email**（属于列系列 **contactinformation**）包含字符串 **contoso.com** 的所有行。你应该会收到以下结果：

        Rae Schroeder - rae@contoso.com - ID: 032439814
        Franklin Holtz - franklin@contoso.com - ID: 229772110
        Gabriela Ingram - gabriela@contoso.com - ID: 655336201

    将 **fabrikam.com** 用于`-emailRegex` 值会返回电子邮件字段中包含 **fabrikam.com** 的用户。由于此搜索是使用基于正则表达式的筛选器执行的，因此你也可以输入正则表达式，例如 **^r**，这样就会返回电子邮件以字母“r”开头的条目。

## 删除表

完成本示例后，请在 PowerShell 会话中使用以下命令删除本示例使用的 **people** 表。

    Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName hdinsightclustername

将 **hdinsightclustername** 替换为 HDInsight 群集的名称。

## 故障排除

### 使用 **Start-HBaseExample** 时未返回结果或者返回了意外的结果

使用 `-showErr` 参数查看运行作业时生成的 STDERR。

  [Apache HBase]: http://hbase.apache.org/
  [Maven]: http://maven.apache.org/
  [Java 平台 JDK]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
  [一个包含 HBase 的 Azure HDInsight 群集]: /en-us/documentation/articles/hdinsight-hbase-get-started/#create-hbase-cluster
  [POM]: http://maven.apache.org/guides/introduction/introduction-to-the-pom.html
  [Maven 存储库搜索]: http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar
  [maven-shade-plugin]: http://maven.apache.org/plugins/maven-shade-plugin/
  [使用远程桌面连接到 HDInsight 群集]: http://azure.microsoft.com/en-us/documentation/articles/hdinsight-administer-use-management-portal/#rdp
  [在 HDInsight 中上载 Hadoop 作业的数据]: /en-us/documentation/articles/hdinsight-upload-data/
  [Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
