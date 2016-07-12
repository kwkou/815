<properties
	pageTitle="使用 Maven 构建 HBase 应用程序并将其部署到基于 Windows 的 HDInsight | Azure"
	description="了解如何使用 Apache Maven 构建基于 Java 的 Apache HBase 应用程序，然后将其部署到基于 Windows 的 Azure HDInsight 群集。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/19/2016"
	wacn.date="06/29/2016"/>

#借助 Maven 生成可将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序

了解如何通过使用 Apache Maven 在 Java 中创建和构建 [Apache HBase](http://hbase.apache.org/) 应用程序。然后，将该应用程序用于 Azure HDInsight (Hadoop)。

[Maven](http://maven.apache.org/) 是一种软件项目管理和综合工具，可用于为 Java 项目构建软件、文档和报告。在本文中，你将要了解如何使用 Maven 创建一个基本的 Java 应用程序，该应用程序可在 Azure HDInsight 群集中创建、查询和删除 HBase 表。

##要求

* [Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更高版本

* [Maven](http://maven.apache.org/)

* [安装有 HBase 的 Azure HDInsight 群集](/documentation/articles/hdinsight-hbase-tutorial-get-started-v1/#create-hbase-cluster)

##创建项目

1. 在开发环境中，通过命令行将目录更改为要创建项目的位置，例如 `cd code\hdinsight`。

2. 使用随同 Maven 一起安装的 __mvn__ 命令，为项目生成基架。

		mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

	此时，将使用 __artifactID__ 参数（本示例中的 **hbaseapp**）指定的名称在当前目录中创建新目录。 此目录将包含以下项：

	* __pom.xml__：项目对象模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html))，其中包含用于生成项目的信息和配置详细信息。

	* __src__：包含 __main\\java\\com\\microsoft\\examples__ 目录的目录，你将在其中创作应用程序。

3. 删除 __src\\test\\java\\com\\microsoft\\examples\\apptest.java__ 文件，因为本示例用不到该文件。

##更新项目对象模型

1. 编辑 __pom.xml__ 文件，并将以下代码添加到 `<dependencies>` 部分：

		<dependency>
      	  <groupId>org.apache.hbase</groupId>
          <artifactId>hbase-client</artifactId>
          <version>0.98.4-hadoop2</version>
        </dependency>

	这将会告知 Maven，项目需要 __hbase-client__ 版本 __0.98.4-hadoop2__。在编译时，将从默认的 Maven 存储库下载该版本。你可以使用 [Maven 中央存储库](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar)搜索来了解有关此依赖性的详细信息。

2. 将以下代码添加到 __pom.xml__ 文件。它必须位于文件中的 `<project>...</project>` 标记内，例如 `</dependencies>` 和 `</project>` 之间。

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
        	  <artifactId>maven-compiler-plugin</artifactId>
						<version>3.3</version>
        	  <configuration>
          	    <source>1.6</source>
          	    <target>1.6</target>
        	  </configuration>
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

	这将配置包含与 HBase 有关的配置信息的资源 (__conf\\hbase-site.xml__)。

	> [AZURE.NOTE]你也可以通过代码设置配置值。有关如何完成此操作的说明，请参阅所采用的 __CreateTable__ 示例中的注释。

	此外，这还将配置 [Maven 编译器插件](http://maven.apache.org/plugins/maven-compiler-plugin/)和 [Maven 阴影插件](http://maven.apache.org/plugins/maven-shade-plugin/)。该编译器插件用于编译拓扑。该阴影插件用于防止在由 Maven 构建的 JAR 程序包中复制许可证。使用此插件的原因在于，重复的许可证文件会导致 HDInsight 群集在运行时出错。将 maven-shade-plugin 用于 `ApacheLicenseResourceTransformer` 实现可防止发生此错误。

	maven-shade-plugin 还会生成 uber jar（或 fat jar），其中包含应用程序所需的所有依赖项。

3. 保存 __pom.xml__ 文件。

4. 在 __hbaseapp__ 目录中创建名为 __conf__ 的新目录。在 __conf__ 目录中，创建名为 __hbase-site.xml__ 的新文件，然后使用以下项作为内容：

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
		  <property>
		    <name>hbase.cluster.distributed</name>
		    <value>true</value>
		  </property>
		  <property>
		    <name>hbase.zookeeper.quorum</name>
		    <value>zookeeper0,zookeeper1,zookeeper2</value>
		  </property>
		  <property>
		    <name>hbase.zookeeper.property.clientPort</name>
		    <value>2181</value>
		  </property>
		</configuration>

	此文件将用于加载 HDInsight 群集的 HBase 配置。

	> [AZURE.NOTE]这是极小的 hbase-site.xml 文件，其中包含 HDInsight 群集的基本最低设置。有关 HDInsight 使用的 hbase-site.xml 配置文件的完整版本，请参阅[使用 Azure 经典管理门户管理 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp)。hbase-site.xml 文件位于 C:\\apps\\dist\\hbase-&lt;version number>-hadoop2\\conf 目录中。在群集上更新 HBase 后，文件路径的版本号部分将发生变化。

3. 保存 __hbase-site.xml__ 文件。

##创建应用程序

1. 转到 __hbaseapp\\src\\main\\java\\com\\microsoft\\examples__ 目录，然后将 app.java 文件重命名为 __CreateTable.java__。

2. 打开 __CreateTable.java__ 文件，并将现有内容替换为以下内容：

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

	这是 __CreateTable__ 类，该类将创建名为 __people__ 的表，并使用一些预定义的用户填充它。

3. 保存 __CreateTable.java__ 文件。

4. 在 __hbaseapp\\src\\main\\java\\com\\microsoft\\examples__ 目录中，创建名为 __SearchByEmail.java__ 的新文件。使用以下项作为此文件的内容：

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

	__SearchByEmail__ 类可用于按电子邮件地址查询行。由于它使用正则表达式筛选器，因此，你可以在使用类时提供字符串或正则表达式。

5. 保存 __SearchByEmail.java__ 文件。

6. 在 __hbaseapp\\src\\main\\hava\\com\\microsoft\\examples__ 目录中，创建名为 __DeleteTable.java__ 的新文件。使用以下项作为此文件的内容：

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

	此类用于清除本示例，方法是禁用并删除由 __CreateTable__ 类创建的表。

7. 保存 __DeleteTable.java__ 文件。

##生成并打包应用程序

1. 打开命令提示符，然后将目录更改为 __hbaseapp__ 目录。

2. 使用以下命令来构建包含应用程序的 JAR 文件：

		mvn clean package

	这将清除任何以前构建的项目，下载任何尚未安装的依赖项，然后构建和打包应用程序。

3. 完成该命令后，__hbaseapp\\target__ 目录将包含名为 __hbaseapp-1.0-SNAPSHOT.jar__ 的文件。

	> [AZURE.NOTE]__hbaseapp-1.0-SNAPSHOT.jar__ 文件是 uber jar（有时称为 fat jar），其中包含运行应用程序所需的所有依赖项。

##上载 JAR 文件并启动作业

> [AZURE.NOTE]你可以使用多种方法将文件上载到 HDInsight 群集，如[在 HDInsight 中为 Hadoop 作业上载数据](/documentation/articles/hdinsight-upload-data/)中所述。以下步骤使用 [Azure PowerShell](/documentation/articles/powershell-install-configure/)。

1. 安装并配置 Azure PowerShell 后，请创建名为 __hbase-runner.psm1__ 的新文件。使用以下项作为此文件的内容：

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
        
        # Get the login for the HDInsight cluster
        $creds = Get-Credential
        
        # Get storage information
        $storage = GetStorage -clusterName $clusterName
        
        # The JAR
        $jarFile = "wasb:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"
        
        # The job definition
        $jobDefinition = New-AzureHDInsightMapReduceJobDefinition `
            -JarFile $jarFile `
            -ClassName $className `
            -Arguments $emailRegex
        
        # Get the job output
        $job = Start-AzureHDInsightJob `
            -Cluster $clusterName `
            -JobDefinition $jobDefinition `
            -Credential $creds
        Write-Host "Wait for the job to complete ..." -ForegroundColor Green
        Wait-AzureHDInsightJob `
            -Cluster $clusterName `
            -JobId $job.JobId `
            -Credential $creds
        if($showErr)
        {
        Write-Host "STDERR"
        Get-AzureHDInsightJobOutput `
                    -Cluster $clusterName `
                    -JobId $job.JobId `
                    -Credential $creds
                    -StandardError
        }
        Write-Host "Display the standard output ..." -ForegroundColor Green
        Get-AzureHDInsightJobOutput `
                    -Cluster $clusterName `
                    -JobId $job.JobId `
                    -Credential $creds
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
            
            # Get authentication for the cluster
            $creds=Get-Credential
            
            # Does the local path exist?
            if (-not (Test-Path $localPath))
            {
                throw "Source path '$localPath' does not exist."
            }
            
            # Get the primary storage container
            $storage = GetStorage -clusterName $clusterName
            
            # Upload file to storage, overwriting existing files if -force was used.
            Set-AzureStorageBlobContent -File $localPath `
                -Blob $destinationPath `
                -force:$force `
                -Container $storage.container `
                -Context $storage.context
        }
        
        function FindAzure {
            # Is there an active Azure subscription?
            $sub = Get-AzureSubscription -ErrorAction SilentlyContinue
            if(-not($sub))
            {
                throw "No active Azure subscription found! If you have a subscription, use the Add-AzureAccount cmdlet to login to your subscription."
            }
        }
        
        function GetStorage {
            param(
                [Parameter(Mandatory = $true)]
                [String]$clusterName
            )
            $hdi = Get-AzureHDInsightCluster -Name $clusterName
            # Does the cluster exist?
            if (!$hdi)
            {
                throw "HDInsight cluster '$clusterName' does not exist."
            }
            # Create a return object for context & container
            $return = @{}
            $storageAccounts = @{}
            
            # Get storage information
            $resourceGroup = $hdi.ResourceGroup
            $storageAccountName=$hdi.DefaultStorageAccount.split('.')[0]
            $container=$hdi.DefaultStorageContainer
            $storageAccountKey=Get-AzureStorageKey `
                -StorageAccountName $storageAccountName `
                | %{ $_.Primary }
            # Get the storage context, as we can't depend
            # on using the default storage context
            $return.context = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
            # Get the container, so we know where to
            # find/store blobs
            $return.container = $container
            # Return storage accounts to support finding all accounts for
            # a cluster
            $return.storageAccount = $storageAccountName
            $return.storageAccountKey = $storageAccountKey
            
            return $return
        }
        # Only export the verb-phrase things
        export-modulemember *-*

	此文件包含两个模块：

	* __Add-HDInsightFile__ - 用于将文件上载到 HDInsight

	* __Start-HBaseExample__ - 用于运行以前创建的类

2. 保存 __hbase-runner.psm1__ 文件。

3. 打开新的 Azure PowerShell 窗口，将目录切换到 __hbaseapp__ 目录，然后运行以下命令。

		PS C:\ Import-Module c:\path\to\hbase-runner.psm1

	将路径切换到前面创建的 __hbase-runner.psm1__ 文件所在的位置。这将为此 Azure PowerShell 会话注册模块。

2. 使用以下命令将 __hbaseapp-1.0-SNAPSHOT.jar__ 上载到你的 HDInsight 群集。

		Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName hdinsightclustername

	将 __hdinsightclustername__ 替换为 HDInsight 群集的名称。此后，该命令将 __hbaseapp-1.0-SNAPSHOT.jar__ 上载到 HDInsight 群集的主存储中的 __example/jars__ 位置。

3. 在上载这些文件后，使用以下代码来通过 __hbaseapp__ 创建新表：

		Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName hdinsightclustername

	将 __hdinsightclustername__ 替换为 HDInsight 群集的名称。

	此命令将在 HDInsight 群集中创建名为 __people__ 的新表。此命令在控制台窗口中不显示任何输出。

2. 若要在表中搜索条目，请使用以下命令：

		Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName hdinsightclustername -emailRegex contoso.com

	将 __hdinsightclustername__ 替换为 HDInsight 群集的名称。

	此命令使用 **SearchByEmail** 类搜索任何 __contactinformation__ 列系列和 __email__ 列包含字符串 __contoso.com__ 的行。你应该会收到以下结果：

		Franklin Holtz - ID: 2
		Franklin Holtz - franklin@contoso.com - ID: 2
		Rae Schroeder - ID: 4
		Rae Schroeder - rae@contoso.com - ID: 4
		Gabriela Ingram - ID: 6
		Gabriela Ingram - gabriela@contoso.com - ID: 6

	将 __fabrikam.com__ 用于 `-emailRegex` 值会返回电子邮件字段中包含 __fabrikam.com__ 的用户。由于此搜索是使用基于正则表达式的筛选器执行的，因此你也可以输入正则表达式，例如 __^r__，这样就会返回电子邮件以字母“r”开头的条目。

##删除表

在完成该示例后，请在 Azure PowerShell 会话中使用以下命令，以删除本示例中使用的 __people__ 表：

	Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName hdinsightclustername

将 __hdinsightclustername__ 替换为 HDInsight 群集的名称。

##故障排除

###使用 Start-HBaseExample 时无结果或意外结果

使用 `-showErr` 参数可查看运行作业时生成的标准错误 (STDERR)。

<!---HONumber=Mooncake_1207_2015-->