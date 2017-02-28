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

* HDInsight 群集（基于 Windows）
* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 7 或更高版本（或类似版本，如 OpenJDK）
* [Apache Maven](http://maven.apache.org/)
* 文本编辑器或 Java IDE

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
   
    这些条目指定了 HDInsight 3.3 群集中包含的 Hadoop 和 Hive 的版本。你可以在 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/)文档中找到 HDInsight 提供的 Hadoop 和 Hive 的版本信息。
   
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

## 后续步骤
有关使用 Hive 的其他方式，请参阅[将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)。

有关 Hive 用户定义函数的详细信息，请参阅 apache.org 网站上的 Hive wiki 的 [Hive 运算符和用户定义函数](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)部分。

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->