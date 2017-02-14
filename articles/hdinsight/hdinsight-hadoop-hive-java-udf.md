<properties
    pageTitle="将 Java 用户定义函数 (UDF) 与 HDInsight 中的 Hive 配合使用 |Azure"
    description="了解如何在 HDInsight 的 Hive 中创建并使用 Java 用户定义函数 (UDF)。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="8d4f8efe-2f01-4a61-8619-651e873c7982"
    ms.service="hdinsight"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 将 Java UDF 与 HDInsight 中的 Hive 配合使用
Hive 非常适用于在 HDInsight 中处理数据，但有时你需要一种更通用的语言。Hive 可让你使用各种编程语言创建用户定义的功能 (UDF)。在本文档中，你将了解如何在 Hive 中使用 Java UDF。

## 要求

* HDInsight 群集（基于 Windows 或 Linux）

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。
  
    本文档中的大多数步骤将适用于这两种群集类型；但是，用于将已编译的 UDF 上载到群集并运行它的步骤特定于基于 Linux 的群集。对于可用于基于 Windows 的群集的信息，提供了链接。

* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 7 或更高版本（或类似版本，如 OpenJDK）
* [Apache Maven](http://maven.apache.org/)
* 文本编辑器或 Java IDE
  
    > [AZURE.IMPORTANT]
    如果使用的是基于 Linux 的 HDInsight 服务器，但创建 Windows 客户端上的 Python 文件，则必须采用使用 LF 作为行结束的编辑器。

## 创建示例 UDF
1. 从命令行中，使用以下命令新建 Maven 项目：
   
        mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=ExampleUDF -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   
    > [AZURE.NOTE]
    如果使用 PowerShell，必须将参数用引号引起来。例如，`mvn archetype:generate "-DgroupId=com.microsoft.examples" "-DartifactId=ExampleUDF" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"`。
    > 
    > 
   
    这将新建一个名为 **exampleudf** 的目录，其中包含 Maven 项目。
2. 创建该项目后，删除作为项目的一部分创建的 **exampleudf/src/test** 目录；该示例中不使用此目录。
3. 打开 **exampleudf/pom.xml**，将现有的 `<dependencies>` 条目替换为以下内容：
   
        <dependencies>
            <dependency>
                <groupId>org.apache.hadoop</groupId>
                <artifactId>hadoop-client</artifactId>
                <version>2.7.1</version>
                <scope>provided</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.hive</groupId>
                <artifactId>hive-exec</artifactId>
                <version>1.2.1</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
   
    这些条目指定了 HDInsight 3.3 和 3.4 群集中包含的 Hadoop 和 Hive 的版本。你可以在 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/)文档中找到 HDInsight 提供的 Hadoop 和 Hive 的版本信息。
   
    在文件末尾的 `</project>` 行前面添加 `<build>` 部分。该部分应包含以下内容：
   
        <build>
            <plugins>
                <!-- build for Java 1.7, even if you're on a later version -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.3</version>
                    <configuration>
                        <source>1.7</source>
                        <target>1.7</target>
                    </configuration>
                </plugin>
                <!-- build an uber jar -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-shade-plugin</artifactId>
                    <version>2.3</version>
                    <configuration>
                        <!-- Keep us from getting a can't overwrite file error -->
                        <transformers>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
                            </transformer>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer">
                            </transformer>
                        </transformers>
                        <!-- Keep us from getting a bad signature error -->
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
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
   
    这些条目用于定义如何生成项目。具体而言，项目使用的 Java 版本以及如何生成部署到群集的 uberjar。
   
    保存发生了更改的文件。
4. 将 **exampleudf/src/main/java/com/microsoft/examples/App.java** 重命名为 **ExampleUDF.java**，然后在编辑器中打开该文件。
5. 将 **ExampleUDF.java** 文件的内容替换为以下内容，然后保存该文件。
   
        package com.microsoft.examples;
   
        import org.apache.hadoop.hive.ql.exec.Description;
        import org.apache.hadoop.hive.ql.exec.UDF;
        import org.apache.hadoop.io.*;
   
        // Description of the UDF
        @Description(
            name="ExampleUDF",
            value="returns a lower case version of the input string.",
            extended="select ExampleUDF(deviceplatform) from hivesampletable limit 10;"
        )
        public class ExampleUDF extends UDF {
            // Accept a string input
            public String evaluate(String input) {
                // If the value is null, return a null
                if(input == null)
                    return null;
                // Lowercase the input string and return it
                return input.toLowerCase();
            }
        }
   
    该代码将实现接受一个字符串值，并返回该字符串的小写形式的 UDF。

## 生成并安装 UDF
1. 使用以下命令编译并打包 UDF：
   
        mvn compile package
   
    这将生成 UDF，然后将 UDF 打包到 **exampleudf/target/ExampleUDF-1.0-SNAPSHOT.jar**。
2. 使用 `scp` 命令将文件复制到 HDInsight 群集。
   
        scp ./target/ExampleUDF-1.0-SNAPSHOT.jar myuser@mycluster-ssh.azurehdinsight
   
    将 **myuser** 替换为群集的 SSH 用户帐户。将 **mycluster** 替换为群集名称。如果使用密码保护 SSH 帐户，系统将提示输入该密码。如果使用了证书，则可能需要使用 `-i` 参数指定私钥文件。
3. 使用 SSH 连接到群集。
   
        ssh myuser@mycluster-ssh.azurehdinsight.cn
   
    有关如何将 SSH 与 HDInsight 配合使用的详细信息，请参阅以下文档。
   
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
4. 从 SSH 会话将 jar 文件复制到 HDInsight 存储。
   
        hdfs dfs -put ExampleUDF-1.0-SNAPSHOT.jar /example/jars

## 在 Hive 中使用 UDF
1. 使用以下命令在 SSH 会话中启动 Beeline 客户端。
   
        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin
   
    该命令假定你使用默认的群集登录帐户 **admin**。
2. 显示 `jdbc:hive2://localhost:10001/>` 提示符后，输入以下代码将 UDF 添加到 Hive，并将其作为函数公开。
   
        ADD JAR wasbs:///example/jars/ExampleUDF-1.0-SNAPSHOT.jar;
        CREATE TEMPORARY FUNCTION tolower as 'com.microsoft.examples.ExampleUDF';
3. 使用该 UDF 将从表中检索的值转换为小写字符串。
   
        SELECT tolower(deviceplatform) FROM hivesampletable LIMIT 10;
   
    这将从表中选择设备平台（Android、Windows、iOS 等）、将字符串转换为小写字符串，然后进行显示。输出结果如下所示。
   
        +----------+--+
        |   _c0    |
        +----------+--+
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        | android  |
        +----------+--+

## 后续步骤
有关使用 Hive 的其他方式，请参阅[将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)。

有关 Hive 用户定义函数的详细信息，请参阅 apache.org 网站上的 Hive wiki 的 [Hive 运算符和用户定义函数](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)部分。

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->