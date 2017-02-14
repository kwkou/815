<!-- not suitable for Mooncake -->

<properties
	pageTitle="使用 Maven 和 Java 构建 HBase 应用程序并将其部署到基于 Linux 的 HDInsight | Azure"
	description="了解如何使用 Apache Maven 构建基于 Java 的 Apache HBase 应用程序，然后将其部署到 Azure 云中基于 Linux 的 HDInsight。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor=""/>

<tags
	ms.service="hdinsight"
	ms.date="10/03/2016"
	wacn.date="02/14/2017"/>

#借助 Maven 构建可将 HBase 与基于 Linux 的 HDInsight (Hadoop) 配合使用的 Java 应用程序

了解如何通过使用 Apache Maven 在 Java 中创建和构建 [Apache HBase](http://hbase.apache.org/) 应用程序。然后，在基于 Linux 的 HDInsight 群集上使用该应用程序。

[Maven](http://maven.apache.org/) 是一种软件项目管理和综合工具，可用于为 Java 项目构建软件、文档和报告。在本文中，你将要了解如何使用 Maven 创建一个基本的 Java 应用程序，该应用程序可在基于 Linux 的 HDInsight 群集上创建、查询和删除 HBase 表。

> [AZURE.NOTE] 本文档中的步骤假设使用基于 Linux 的 HDInsight 群集。有关使用基于 Windows 的 HDInsight 群集的信息，请参阅 [Use Maven to build Java applications that use HBase with Windows-based HDInsight（借助 Maven 构建可将 HBase 与基于 Windows 的 HDInsight 配合使用的 Java 应用程序）](/documentation/articles/hdinsight-hbase-build-java-maven/)

##要求

* [Java 平台 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 7 或更高版本

* [Maven](http://maven.apache.org/)

* [装有 HBase 的基于 Linux 的 Azure HDInsight 群集](/documentation/articles/hdinsight-hbase-tutorial-get-started-linux/#create-hbase-cluster)

    > [AZURE.NOTE] 本文档中的步骤已在 HDInsight 群集版本 3.2、3.3 和 3.4 中测试。示例中提供的默认值适用于 HDInsight 3.4 群集。

* **熟悉 SSH 和 SCP**。有关如何在 HDInsight 中使用 SSH 和 SCP 的详细信息，请参阅以下文档：

    * **Linux、Unix 或 OS X 客户端**：请参阅 [Use SSH with Linux-based Hadoop on HDInsight from Linux, OS X or Unix（在 Linux、OS X 或 Unix 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用）](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)

    * **Windows 客户端**：请参阅 [Use SSH with Linux-based Hadoop on HDInsight from Windows（在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用）](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)

##创建项目

1. 在开发环境中，通过命令行将目录更改为要创建项目的位置，例如 `cd code/hdinsight`。

2. 使用随同 Maven 一起安装的 __mvn__ 命令，为项目生成基架。

		mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=hbaseapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

	此时，将使用 __artifactID__ 参数（本示例中的 **hbaseapp**）指定的名称在当前目录中创建新目录。 此目录将包含以下项：

	* __pom.xml__：项目对象模型 ([POM](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html))，其中包含用于生成项目的信息和配置详细信息。

	* __src__：包含 __main/java/com/microsoft/examples__ 目录的目录，你将在其中创作应用程序。

3. 删除 __src/test/java/com/microsoft/examples/apptest.java__ 文件，因为本示例用不到该文件。

##更新项目对象模型

1. 编辑 __pom.xml__ 文件，并将以下代码添加到 `<dependencies>` 部分：

		<dependency>
      	  <groupId>org.apache.hbase</groupId>
          <artifactId>hbase-client</artifactId>
          <version>1.1.2</version>
        </dependency>

	这将会告知 Maven，项目需要 __hbase-client__ 版本 __1.1.2__。在编译时，将从默认的 Maven 存储库下载该版本。你可以使用 [Maven 中央存储库](http://search.maven.org/#artifactdetails%7Corg.apache.hbase%7Chbase-client%7C0.98.4-hadoop2%7Cjar)搜索来了解有关此依赖性的详细信息。

    > [AZURE.IMPORTANT] 版本号必须与 HDInsight 群集随附的 HBase 版本匹配。可以使用下表来查找正确的版本号。

    | HDInsight 群集版本 | 要使用的 HBase 版本 |
    | ----- | ----- |
    | 3\.2 | 0\.98.4-hadoop2 |
    | 3\.3 和 3.4 | 1\.1.2 |

    有关 HDInsight 版本和组件的详细信息，请参阅 [What are the different Hadoop components available with HDInsight（HDInsight 提供哪些不同的 Hadoop 组件）](/documentation/articles/hdinsight-component-versioning/)。

2. 如果使用 HDInsight 3.3 或 3.4 群集，则还必须将以下代码添加到 `<dependencies>` 节：

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>4.4.0-HBase-1.1</version>
        </dependency>
    
    此代码将会加载需要在 Hbase 版本 1.1.x 中使用的 phoenix-core 组件。

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
          	    <source>1.7</source>
          	    <target>1.7</target>
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

	这将配置包含与 HBase 有关的配置信息的资源 (__conf/hbase-site.xml__)。

	> [AZURE.NOTE] 你也可以通过代码设置配置值。有关如何完成此操作的说明，请参阅所采用的 __CreateTable__ 示例中的注释。

	此外，这还将配置 [Maven 编译器插件](http://maven.apache.org/plugins/maven-compiler-plugin/)和 [Maven 阴影插件](http://maven.apache.org/plugins/maven-shade-plugin/)。该编译器插件用于编译拓扑。该阴影插件用于防止在由 Maven 构建的 JAR 程序包中复制许可证。使用此插件的原因在于，重复的许可证文件会导致 HDInsight 群集在运行时出错。将 maven-shade-plugin 用于 `ApacheLicenseResourceTransformer` 实现可防止发生此错误。

	maven-shade-plugin 还会生成 uber jar（或 fat jar），其中包含应用程序所需的所有依赖项。

3. 保存 __pom.xml__ 文件。

4. 在 __hbaseapp__ 目录中创建名为 __conf__ 的新目录。此目录用于保存连接到 HBase 所需的配置信息。

5. 使用以下命令将 HBase 配置从 HDInsight 服务器复制到 __conf__ 目录。将 **USERNAME** 替换为你的 SSH 登录名。将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

		scp USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml

	> [AZURE.NOTE] 如果你使用了 SSH 帐户的密码，则系统将提示你输入该密码。如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。以下示例将从 `~/.ssh/id_rsa` 加载私钥：
	><p>
	> `scp -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:/etc/hbase/conf/hbase-site.xml ./conf/hbase-site.xml`

##创建应用程序

1. 转到 __hbaseapp/src/main/java/com/microsoft/examples__ 目录，然后将 app.java 文件重命名为 __CreateTable.java__。

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

	这是 __CreateTable__ 类，该类将创建名为 __people__ 的表，并使用一些预定义的用户填充它。

3. 保存 __CreateTable.java__ 文件。

4. 在 __hbaseapp/src/main/java/com/microsoft/examples__ 目录中，创建名为 __SearchByEmail.java__ 的新文件。使用以下项作为此文件的内容：

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

6. 在 __hbaseapp/src/main/hava/com/microsoft/examples__ 目录中，创建名为 __DeleteTable.java__ 的新文件。使用以下项作为此文件的内容：

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

2. 在 __hbaseapp__ 目录中，使用以下命令来构建包含应用程序的 JAR 文件：

		mvn clean package

	这将清除任何以前构建的项目，下载任何尚未安装的依赖项，然后构建和打包应用程序。

3. 完成该命令后，__hbaseapp/target__ 目录将包含名为 __hbaseapp-1.0-SNAPSHOT.jar__ 的文件。

	> [AZURE.NOTE] __hbaseapp-1.0-SNAPSHOT.jar__ 文件是 uber jar（有时称为 fat jar），其中包含运行应用程序所需的所有依赖项。

##上载 JAR 文件并运行作业

1. 使用以下命令将该 jar 文件上载到 HDInsight 群集。将 **USERNAME** 替换为你的 SSH 登录名。将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

		scp ./target/hbaseapp-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:.

	这会将文件上载到 SSH 用户帐户的主目录。

	> [AZURE.NOTE] 如果你使用了 SSH 帐户的密码，则系统将提示你输入该密码。如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。以下示例将从 `~/.ssh/id_rsa` 加载私钥：
	><p>
	> `scp -i ~/.ssh/id_rsa ./target/hbaseapp-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:.`

2. 使用 SSH 连接到 HDInsight 群集。将 **USERNAME** 替换为你的 SSH 登录名。将 **CLUSTERNAME** 替换为 HDInsight 群集名称：

		ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn

	> [AZURE.NOTE] 如果你使用了 SSH 帐户的密码，则系统将提示你输入该密码。如果将 SSH 密钥与帐户配合使用，则可能需要使用 `-i` 参数来指定密钥文件的路径。以下示例将从 `~/.ssh/id_rsa` 加载私钥：
	><p>
	> `ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn`

3. 连接后，使用以下命令在 Java 应用程序中创建新的 HBase 表：

		hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.CreateTable

	这将创建名为 __people__ 的新 HBase 表，并在其中填充数据。

4. 接下来，使用以下命令搜索存储在表中的电子邮件地址：

		hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.SearchByEmail contoso.com

	你应该会收到以下结果：

		Franklin Holtz - ID: 2
		Franklin Holtz - franklin@contoso.com - ID: 2
		Rae Schroeder - ID: 4
		Rae Schroeder - rae@contoso.com - ID: 4
		Gabriela Ingram - ID: 6
		Gabriela Ingram - gabriela@contoso.com - ID: 6

##删除表

在完成该示例后，请在 Azure PowerShell 会话中使用以下命令，以删除本示例中使用的 __people__ 表：

	hadoop jar hbaseapp-1.0-SNAPSHOT.jar com.microsoft.examples.DeleteTable

<!---HONumber=Mooncake_0725_2016-->