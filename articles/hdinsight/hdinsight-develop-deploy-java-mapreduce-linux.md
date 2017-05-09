<properties
    pageTitle="为基于 Linux 的 HDInsight 开发 Java MapReduce 程序 | Azure"
    description="了解如何开发 Java MapReduce 程序并将其部署到基于 Linux 的 HDInsight。"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    author="Blackmist"
    documentationcenter=""
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="9ee6384c-cb61-4087-8273-fb53fa27c1c3"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="Java"
    ms.topic="article"
    ms.date="02/17/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="e2c609da661e30367605b552dea77d46da465fac"
    ms.lasthandoff="04/28/2017" />

# <a name="develop-java-mapreduce-programs-for-hadoop-on-hdinsight-linux"></a>为 HDInsight Linux 上的 Hadoop 开发 Java MapReduce 程序

了解如何使用 Apache Maven 创建基于 Java 的 MapReduce 应用程序，然后在 HDInsight 群集中基于 Linux 的 Hadoop 上部署和运行它。

## <a name="prerequisites"></a>先决条件

* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) 8 或更高版本（或类似程序，如 OpenJDK）...

    > [AZURE.NOTE]
    > HDInsight 版本 3.4 及更早版本使用 Java 7。 HDInsight 3.5 使用 Java 8。

* [Apache Maven](http://maven.apache.org/)

* **Azure 订阅**

* **Azure CLI**

[AZURE.INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-cli.md)]

## <a name="configure-environment-variables"></a>配置环境变量
可以在安装 Java 和 JDK 时设置以下环境变量。 但应检查其是否存在并且包含相关系统的适当值。

* `JAVA_HOME` - 应该指向已安装 Java 运行时环境 (JRE) 的目录。 例如，在 OS X、Unix 或 Linux 系统上，它的值应该类似于 `/usr/lib/jvm/java-7-oracle`。 在 Windows 中，它的值类似于 `c:\Program Files (x86)\Java\jre1.7`

* `PATH` - 应该包含以下路径：

    * `JAVA_HOME`（或等效路径）

    * `JAVA_HOME\bin`（或等效路径）

    * 安装 Maven 的目录

## <a name="create-a-maven-project"></a>创建 Maven 项目

1. 在开发环境中，通过中断会话或命令行将目录更改为要存储此项目的位置。

2. 使用随同 Maven 一起安装的 `mvn` 命令，为项目生成基架。

        mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

    此命令将使用 **artifactID** 参数指定的名称（此示例中为 **wordcountjava**）创建目录。此目录包含以下项：

    * `pom.xml` - [项目对象模型 (POM)](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html)，其中包含用于生成项目的信息和配置详细信息。

    * `src` - 包含应用程序的目录。

3. 删除 `src/test/java/org/apache/hadoop/examples/apptest.java` 文件，因为本示例不使用该文件。

## <a name="add-dependencies"></a>添加依赖项

1. 编辑 `pom.xml` 文件，并在 `<dependencies>` 部分中添加以下文本：

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

    这会定义具有特定版本（在 &lt;version\> 中列出）的库（在 &lt;artifactId\> 中列出）。 在编译时，会从默认 Maven 存储库下载这些依赖项。 你可以使用 [Maven 存储库搜索](http://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar) 来查看详细信息。

    `<scope>provided</scope>` 会告知 Maven 这些依赖项不应与此应用程序一起打包，因为它们在运行时由 HDInsight 群集提供。

2. 将以下内容添加到 `pom.xml` 文件中。 此文本必须位于文件中的 `<project>...</project>` 标记内；例如 `</dependencies>` 和 `</project>` 之间。

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

    第一个插件配置 [Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/)，用于生成 uberjar（有时称为 fatjar），其中包含应用程序所需的依赖项。 它还可以防止在 jar 包中复制许可证，复制许可证在某些系统中可能会导致问题。

    第二个插件配置目标 Java 版本。

    > [AZURE.NOTE]
    > HDInsight 3.4 及更早版本使用 Java 7。 HDInsight 3.5 使用 Java 8。

3. 保存 `pom.xml` 文件。

## <a name="create-the-mapreduce-application"></a>创建 MapReduce 应用程序

1. 转到 `wordcountjava/src/main/java/org/apache/hadoop/examples` 目录，并将 `App.java` 文件重命名为 `WordCount.java`。

2. 在文本编辑器中打开 `WordCount.java` 文件，然后将其内容替换为以下文本：

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

    请注意，包名称为 `org.apache.hadoop.examples`，类名称为 `WordCount`。 提交 MapReduce 作业时，使用这些名称。

3. 保存文件。

## <a name="build-the-application"></a>构建应用程序

1. 如果尚未到达此目录，请更改为 `wordcountjava` 目录。

2. 使用以下命令生成包含该应用程序的 JAR 文件：

        mvn clean package

    此命令将清除任何以前构建的项目，下载任何尚未安装的依赖项，然后生成并打包应用程序。

3. 命令完成后，`wordcountjava/target` 目录将包含一个名为 `wordcountjava-1.0-SNAPSHOT.jar` 的文件。

    > [AZURE.NOTE]
    > `wordcountjava-1.0-SNAPSHOT.jar` 文件是一种 uberjar，其中不仅包含 WordCount 作业，还包含作业在运行时需要的依赖项。

## <a id="upload"></a>上载该 jar

使用以下命令将该 jar 文件上载到 HDInsight 头节点：

    scp wordcountjava-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:

将 __USERNAME__ 替换为群集的 SSH 用户名。 将 __CLUSTERNAME__ 替换为 HDInsight 群集名称。

此命令会将文件从本地系统复制到头节点。

> [AZURE.NOTE]
> 如果使用了密码来保护 SSH 帐户，系统会提示输入密码。 如果你使用了 SSH 密钥，你可能必须使用 `-i` 参数和私钥的路径。 例如， `scp -i /path/to/private/key wordcountjava-1.0-SNAPSHOT.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.cn:`。

## <a name="run"></a>运行 MapReduce 作业

1. 使用 SSH 连接到 HDInsight。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 在 SSH 会话中，使用以下命令运行 MapReduce 应用程序：

        yarn jar wordcountjava-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /example/data/gutenberg/davinci.txt /example/data/wordcountout

    此命令启动 WordCount MapReduce 应用程序。 输入文件是 **/example/data/gutenberg/davinci.txt**，输出存储在 **/example/data/wordcountout** 中。 输入文件和输出均存储到群集的默认存储中。

3. 作业完成后，请使用以下命令查看结果：

        hdfs dfs -cat /example/data/wordcountout/*

    用户会收到单词和计数列表，其包含的值类似于以下文本：

        zeal    1
        zelus   1
        zenith  2

## <a id="nextsteps"></a>后续步骤

在本文档中，你学习了如何开发 Java MapReduce 作业。 请参阅以下文档，了解使用 HDInsight 的其他方式。

* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

有关详细信息，另请参阅 [Java 开发人员中心](/develop/java/)。

[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-use-sqoop]: /documentation/articles/hdinsight-use-sqoop/
[hdinsight-ODBC]: /documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/

[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-admin-powershell]: /documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-power-query]: /documentation/articles/hdinsight-connect-excel-power-query/

[powershell-PSCredential]: http://social.technet.microsoft.com/wiki/contents/articles/4546.working-with-passwords-secure-strings-and-credentials-in-windows-powershell.aspx

<!--Update_Description: wording update-->