<properties
pageTitle="在 HDInsight 中通过 Hive 使用 Java 用户定义函数 (UDF) | Azure"
description="了解如何在 HDInsight 的 Hive 中创建并使用 Java 用户定义函数 (UDF)。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="07/07/2016"
	wacn.date="08/01/2016"/>

# 在 HDInsight 中通过 Hive 使用 Java UDF

Hive 非常适用于在 HDInsight 中处理数据，但有时你需要一种更通用的语言。Hive 可让你使用各种编程语言创建用户定义的功能 (UDF)。在本文档中，你将学习如何在 Hive 中使用 Java UDF。

## 要求

* Azure 订阅

* HDInsight 群集 (Windows)

* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 7 或更高版本（或类似版本，如 OpenJDK）

* [Apache Maven](http://maven.apache.org/)

* 文本编辑器或 Java IDE

## 创建示例 UDF

1. 从命令行中，使用以下命令创建新 Maven 项目：

        mvn archetype:generate -DgroupId=com.microsoft.examples -DartifactId=ExampleUDF -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    > [AZURE.NOTE] 如果使用 PowerShell，必须将参数用引号引起来。例如，`mvn archetype:generate "-DgroupId=com.microsoft.examples" "-DartifactId=ExampleUDF" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"`。

    这将创建一个名为 __exampleudf__ 的新目录，其中包含 Maven 项目。

2. 创建该项目后，删除作为项目的一部分创建的 __exampleudf/src/test__ 目录；该示例中不使用此目录。

3. 打开 __exampleudf/pom.xml__，将现有的 `<dependencies>` 输入替换为以下内容：

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

    这些输入指定了 HDInsight 3.3 和 3.4 群集中包含的 Hadoop 和 Hive 的版本。你可以在[HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning-v1/)文档中找到 HDInsight 提供的 Hadoop 和 Hive 的版本信息。

    进行此更改之后，保存该文件。

4. 将 __exampleudf/src/main/java/com/microsoft/examples/App.java__ 重命名为 __ExampleUDF.java__，然后在编辑器中打开该文件。

5. 将 __ExampleUDF.java__ 文件的内容替换为以下代码，然后保存该文件。

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

## 在 Hive 中使用 UDF

1. 使用以下命令在 SSH 会话中启动 Beeline 客户端。

        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin

    该命令假定你使用默认的群集登录帐户 __admin__。

2. 当到达 `jdbc:hive2://localhost:10001/>` 提示符时，输入以下代码将 UDF 添加到 Hive，并将其作为函数公开。

        ADD JAR wasbs:///example/jar/ExampleUDF-1.0-SNAPSHOT.jar;
        CREATE TEMPORARY FUNCTION tolower as 'com.microsoft.examples.ExampleUDF';

3. 使用该 UDF 将从表中检索的值转换为小写字符串。

        SELECT tolower(deviceplatform) FROM hivesampletable LIMIT 10;

    上面的代码将从表中选择设备平台（Android、Windows、iOS 等），然后将字符串转换为小写字符串，并显示它们。输出结果如下所示。

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

有关使用 Hive 的其他方式，请参阅[在 HDInsight 中使用 Hive](/documentation/articles/hdinsight-use-hive/)。

有关 Hive 用户定义函数的详细信息，请参阅 apache.org 网站上的 Hive wiki 的 [Hive Operators and User-Defined Functions（Hive 运算符和用户定义函数）](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)部分。

<!---HONumber=Mooncake_0725_2016-->