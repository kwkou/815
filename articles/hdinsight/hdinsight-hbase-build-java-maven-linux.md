<properties
    pageTitle="为 Azure HDInsight 构建 Java HBase 应用程序 | Azure"
    description="了解如何使用 Apache Maven 构建基于 Java 的 Apache HBase 应用程序，然后将其部署到 Azure 云中基于 Linux 的 HDInsight。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="1d1ed180-e0f4-4d1c-b5ea-72e0eda643bc"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/17/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="4b4b5d4e6a4c5dc40d6a9ab14b26ec8faf84f59b"
    ms.lasthandoff="04/28/2017" />

# <a name="use-maven-to-build-java-applications-that-use-hbase-with-linux-based-hdinsight-hadoop"></a>借助 Maven 构建可将 HBase 与基于 Linux 的 HDInsight (Hadoop) 配合使用的 Java 应用程序
了解如何通过使用 Apache Maven 在 Java 中创建和构建 [Apache HBase](http://hbase.apache.org/) 应用程序。 然后，在基于 Linux 的 HDInsight 群集上使用该应用程序。

[Maven](http://maven.apache.org/) 是一种软件项目管理和综合工具，可用于为 Java 项目构建软件、文档和报告。 在本文中，你将要了解如何使用 Maven 创建一个基本的 Java 应用程序，该应用程序可在基于 Linux 的 HDInsight 群集上创建、查询和删除 HBase 表。

> [AZURE.IMPORTANT]
> 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="requirements"></a>要求

* [Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 8 或更高版本。

    > [AZURE.NOTE]
    > HDInsight 3.5 需要 Java 8。 早期的 HDInsight 版本需要 Java 7。

* [Maven](http://maven.apache.org/)

* [装有 HBase 的基于 Linux 的 Azure HDInsight 群集](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/#create-hbase-cluster)

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.NOTE]
    > 本文档中的步骤已在 HDInsight 群集版本 3.2、3.3、3.4 和 3.5 中测试。 示例中提供的默认值适用于 HDInsight 3.5 群集。

* **熟悉 SSH 和 SCP** 或 **Azure PowerShell**。 本文档提供有关运行此示例时使用 SSH/SCP 和 Azure PowerShell 的步骤。

    有关安装 Azure PowerShell 的信息，请参阅 [Azure PowerShell 入门](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

    有关详细信息，请参阅[对 HDInsight 使用 SSH](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a name="create-the-project"></a>创建项目

1. 在开发环境中，通过命令行将目录更改为要创建项目的位置，例如 `cd code/hdinsight`。

2. 使用随同 Maven 一起安装的 **mvn** 命令，为项目生成基架。

        mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    此命令使用与 **artifactID** 参数相同的名称（此示例中为 **hbaseapp**）创建目录。此目录包含以下项：

    * **pom.xml**：项目对象模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html))，其中包含用于生成项目的信息和配置详细信息。
    * **src**：包含 **main/java/com/microsoft/examples** 目录的目录，用户将在其中创作应用程序。

3. 删除 **src/test/java/com/microsoft/examples/apptest.java** 文件，因为本示例中不使用该文件。

## <a name="update-the-project-object-model"></a>更新项目对象模型

1. 编辑 **pom.xml** 文件，并将以下代码添加到 `<dependencies>` 部分：

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.1.2</version>
        </dependency>

    此部分指示项目需要 **hbase-client** 版本**1.1.2**。 在编译时，将从默认的 Maven 存储库下载该依赖项。 你可以使用 [Maven 中央存储库](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar) 搜索来了解有关此依赖性的详细信息。

    > [AZURE.IMPORTANT]
    > 版本号必须与 HDInsight 群集随附的 HBase 版本匹配。 可以使用下表来查找正确的版本号。

    | HDInsight 群集版本 | 要使用的 HBase 版本 |
    | --- | --- |
    | 3.2 |0.98.4-hadoop2 |
    | 3.3、3.4 和 3.5 |1.1.2 |

    有关 HDInsight 版本和组件的详细信息，请参阅 [HDInsight 提供哪些不同的 Hadoop 组件](/documentation/articles/hdinsight-component-versioning/)。

2. 如果要使用 HDInsight 3.3、3.4 或 3.5 群集，还必须将以下代码添加到 `<dependencies>` 节：

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>4.4.0-HBase-1.1</version>
        </dependency>

    此节会加载需要在 Hbase 版本 1.1.x 中使用的 phoenix-core 组件。

3. 将以下代码添加到 **pom.xml** 文件。 此文本必须位于文件中的 `<project>...</project>` 标记内，例如 `</dependencies>` 和 `</project>` 之间。

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
                    <source>1.8</source>
                    <target>1.8</target>
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

    此部分将配置包含与 HBase 有关的配置信息的资源 (**conf/hbase-site.xml**)。

    > [AZURE.NOTE]
    > 你也可以通过代码设置配置值。 请参阅 **CreateTable** 示例中的注释。

    此部分还将配置 [Maven 编译器插件](http://maven.apache.org/plugins/maven-compiler-plugin/)和 [Maven 阴影插件](http://maven.apache.org/plugins/maven-shade-plugin/)。 该编译器插件用于编译拓扑。 该阴影插件用于防止在由 Maven 构建的 JAR 程序包中复制许可证。 此插件用于防止在 HDInsight 群集上运行时出现“重复的许可证文件”错误。 将 maven-shade-plugin 用于 `ApacheLicenseResourceTransformer` 实现可防止发生此错误。

    maven-shade-plugin 还会生成 uber jar，其中包含应用程序所需的所有依赖项。

4. 保存 **pom.xml** 文件。

5. 在 **hbaseapp** 目录中创建名为 **conf** 的目录。 此目录用于保存连接到 HBase 所需的配置信息。

6. 使用以下命令将 HBase 配置从 HDInsight 服务器复制到 **conf** 目录。 将 **USERNAME** 替换为 SSH 登录名。 将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

        scp USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml

    > [AZURE.NOTE]
    > 如果使用了 SSH 帐户的密码，则系统将提示你输入该密码。 如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。 以下示例从 `~/.ssh/id_rsa`加载私钥：
    > <p>
    > `scp -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml`

## <a name="create-the-application"></a>创建应用程序

1. 转到 **hbaseapp/src/main/java/com/microsoft/examples** 目录，然后将 app.java 文件重命名为 **CreateTable.java**。

2. 打开 **CreateTable.java** 文件，并将现有内容替换为以下文本：

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
            // in code instead of an hbase-site.xml file
            //
            // config.set("hbase.zookeeper.quorum",
            //            "zookeepernode0,zookeepernode1,zookeepernode2");
            //config.set("hbase.zookeeper.property.clientPort", "2181");
            //config.set("hbase.cluster.distributed", "true");
            //
            //NOTE: Actual zookeeper host names can be found using Ambari:
            //curl -u admin:PASSWORD -G "https://CLUSTERNAME.azurehdinsight.cn/api/v1/clusters/CLUSTERNAME/hosts"

            //Linux-based HDInsight clusters use /hbase-unsecure as the znode parent
            config.set("zookeeper.znode.parent","/hbase-unsecure");

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

    此代码是 **CreateTable** 类，该类会创建名为 **people** 的表，并使用一些预定义的用户填充它。

3. 保存 **CreateTable.java** 文件。

4. 在 **hbaseapp/src/main/java/com/microsoft/examples** 目录中，创建名为 **SearchByEmail.java** 的文件。 将以下文本用作此文件的内容：

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

            // Create a regex filter
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

    **SearchByEmail** 类可用于按电子邮件地址查询行。 由于它使用正则表达式筛选器，因此，你可以在使用类时提供字符串或正则表达式。

5. 保存 **SearchByEmail.java** 文件。

6. 在 **hbaseapp/src/main/hava/com/microsoft/examples** 目录中，创建名为 **DeleteTable.java** 的文件。 将以下文本用作此文件的内容：

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

    此类用于清除本示例中创建的 HBase 表，方法是禁用并删除由 **CreateTable** 类创建的表。

7. 保存 **DeleteTable.java** 文件。

## <a name="build-and-package-the-application"></a>生成并打包应用程序

1. 在 **hbaseapp** 目录中，使用以下命令构建包含应用程序的 JAR 文件：

        mvn clean package

    该指令会清除任何以前构建的项目，下载任何尚未安装的依赖项，然后构建和打包应用程序。

2. 完成该命令后，**hbaseapp/target** 目录将包含名为 **hbaseapp-1.0-SNAPSHOT.jar** 的文件。

    > [AZURE.NOTE]
    > **hbaseapp-1.0-SNAPSHOT.jar** 文件是 uber jar。 它包含运行应用程序所需的所有依赖项。

## <a name="upload-the-jar-and-run-jobs-ssh"></a>上载 JAR 并运行作业 (SSH)

以下步骤使用 `scp` 将 JAR 复制到 HDInsight 群集的主要头节点。 然后，使用 `ssh` 命令连接到群集并直接在头节点上运行此示例。

1. 使用以下命令将该 jar 文件上载到 HDInsight 群集。 将 **USERNAME** 替换为 SSH 登录名。 将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

        scp ./target/hbaseapp-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:hbaseapp-1.0-SNAPSHOT.jar

    此命令会将文件上载到 SSH 用户帐户的主目录。

    > [AZURE.NOTE]
    > 如果使用了 SSH 帐户的密码，则系统将提示你输入该密码。 如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。 以下示例从 `~/.ssh/id_rsa`加载私钥：
    > <p>
    > `scp -i ~/.ssh/id_rsa ./target/hbaseapp-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:hbaseapp-1.0-SNAPSHOT.jar`

2. 使用 SSH 连接到 HDInsight 群集。 将 **USERNAME** 替换为 SSH 登录名。 将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn

    > [AZURE.NOTE]
    > 如果使用了 SSH 帐户的密码，则系统将提示你输入该密码。 如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。 以下示例从 `~/.ssh/id_rsa`加载私钥：
    > <p>
    > `ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn`

3. 连接后，使用以下命令在 Java 应用程序中创建 HBase 表：

        hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable

    此命令将创建名为 **people**的新 HBase 表，并在其中填充数据。

4. 接下来，使用以下命令搜索存储在表中的电子邮件地址：

        hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com

    你应该会收到以下结果：

        Franklin Holtz - ID: 2
        Franklin Holtz - franklin@contoso.com - ID: 2
        Rae Schroeder - ID: 4
        Rae Schroeder - rae@contoso.com - ID: 4
        Gabriela Ingram - ID: 6
        Gabriela Ingram - gabriela@contoso.com - ID: 6

## <a name="upload-the-jar-and-run-jobs-powershell"></a>上载 JAR 并运行作业 (PowerShell)

以下步骤使用 Azure PowerShell 将 JAR 上载到 HDInsight 群集的默认存储。 然后使用 HDInsight cmdlet 远程运行这些示例。

1. 安装并配置 Azure PowerShell 后，请创建名为 **hbase-runner.psm1**的文件。 使用以下文本作为此文件的内容：

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
        $creds=Get-Credential -Message "Enter the login for the cluster" -UserName "admin"

        # The JAR
        $jarFile = "wasbs:///example/jars/hbaseapp-1.0-SNAPSHOT.jar"

        # The job definition
        $jobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
            -JarFile $jarFile `
            -ClassName $className `
            -Arguments $emailRegex

        # Get the job output
        $job = Start-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobDefinition $jobDefinition `
            -HttpCredential $creds
        Write-Host "Wait for the job to complete ..." -ForegroundColor Green
        Wait-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobId $job.JobId `
            -HttpCredential $creds
        if($showErr)
        {
        Write-Host "STDERR"
        Get-AzureRmHDInsightJobOutput `
                    -Clustername $clusterName `
                    -JobId $job.JobId `
                    -HttpCredential $creds `
                    -DisplayOutputType StandardError
        }
        Write-Host "Display the standard output ..." -ForegroundColor Green
        Get-AzureRmHDInsightJobOutput `
                    -Clustername $clusterName `
                    -JobId $job.JobId `
                    -HttpCredential $creds
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
            $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
            if(-not($sub))
            {
                throw "No active Azure subscription found! If you have a subscription, use the Login-AzureRmAccount -EnvironmentName AzureChinaCloud cmdlet to login to your subscription."
            }
        }

        function GetStorage {
            param(
                [Parameter(Mandatory = $true)]
                [String]$clusterName
            )
            $hdi = Get-AzureRmHDInsightCluster -ClusterName $clusterName
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
            $storageAccountKey=(Get-AzureRmStorageAccountKey `
                -Name $storageAccountName `
            -ResourceGroupName $resourceGroup)[0].Value
            # Get the resource group, in case we need that
            $return.resourceGroup = $resourceGroup
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

    * **Add-HDInsightFile** - 用于将文件上载到 HDInsight
    * **Start-HBaseExample** - 用于运行以前创建的类

2. 保存 **hbase-runner.psm1** 文件。

3. 打开新的 Azure PowerShell 窗口，将目录切换到 **hbaseapp** 目录，然后运行以下命令：

        PS C:\ Import-Module c:\path\to\hbase-runner.psm1

    将路径切换到前面创建的 **hbase-runner.psm1** 文件所在的位置。 此命令使用 Azure PowerShell 注册模块。

4. 使用以下命令将 **hbaseapp-1.0-SNAPSHOT.jar** 上载到你的 HDInsight 群集。

        Add-HDInsightFile -localPath target\hbaseapp-1.0-SNAPSHOT.jar -destinationPath example/jars/hbaseapp-1.0-SNAPSHOT.jar -clusterName hdinsightclustername

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。 该命令将 **hbaseapp-1.0-SNAPSHOT.jar** 上传到 HDInsight 群集的主存储中的 **example/jars** 位置。

5. 上传这些文件后，使用以下代码通过 **hbaseapp** 创建表：

        Start-HBaseExample -className com.microsoft.examples.CreateTable -clusterName hdinsightclustername

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。

    此命令将在 HDInsight 群集中创建名为 **people** 的新表。 此命令在控制台窗口中不显示任何输出。

6. 若要在表中搜索条目，请使用以下命令：

        Start-HBaseExample -className com.microsoft.examples.SearchByEmail -clusterName hdinsightclustername -emailRegex contoso.com

    将 **hdinsightclustername** 替换为 HDInsight 群集的名称。

    此命令使用 **SearchByEmail** 类搜索任何 **contactinformation** 列系列和 **email** 列包含字符串 **contoso.com** 的行。 你应该会收到以下结果：

          Franklin Holtz - ID: 2
          Franklin Holtz - franklin@contoso.com - ID: 2
          Rae Schroeder - ID: 4
          Rae Schroeder - rae@contoso.com - ID: 4
          Gabriela Ingram - ID: 6
          Gabriela Ingram - gabriela@contoso.com - ID: 6

    将 **fabrikam.com** 用于 `-emailRegex` 值会返回电子邮件字段中包含 **fabrikam.com** 的用户。 还可以使用正则表达式作为搜索词。 例如， **^r** 将返回以字母“r”开头的电子邮件地址。

### <a name="no-results-or-unexpected-results-when-using-start-hbaseexample"></a>使用 Start-HBaseExample 时无结果或意外结果

使用 `-showErr` 参数可查看运行作业时生成的标准错误 (STDERR)。

## <a name="delete-the-table"></a>删除表

在完成该示例后，使用以下命令删除本示例中使用的 **people** 表：

__从 `ssh` 会话__：

`hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable`

__从 Azure PowerShell__：

`Start-HBaseExample -className com.microsoft.examples.DeleteTable -clusterName hdinsightclustername`