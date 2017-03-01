<properties
   pageTitle="使用 HDInsight 开发 Python MapReduce 作业 | Azure"
   description="了解如何在基于 Linux 的 HDInsight 群集上创建和运行 Python MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="10/11/2016"
	wacn.date="02/14/2017"/>

#开发适用于 HDInsight 的 Python 流式处理程序

Hadoop 为 MapReduce 提供了一个流式处理 API，这样，除 Java 外，你还能使用其他语言编写映射和化简函数。本文介绍如何使用 Python 执行 MapReduce 操作。

> [AZURE.NOTE] 对于基于 Windows 的 HDInsight 群集，尽管也可以使用本文档中的 Python 代码，但本文所述步骤是专门针对基于 Linux 的群集编写的。

本文的内容基于 Michael Noll 在 [Writing an Hadoop MapReduce Program in Python](http://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/)（以 Python 编写 Hadoop MapReduce 程序）中发布的信息和示例。

##先决条件

要完成本文中的步骤，需要：

* 基于 Linux 的 HDInsight 上的 Hadoop 群集

* 文本编辑器

    > [AZURE.IMPORTANT] 文本编辑器必须使用 LF 作为行尾。如果使用 CRLF，则在基于 Linux 的 HDInsight 群集上运行 MapReduce 作业时会出错。如果不确定它使用哪种行尾，请使用[运行 MapReduce](#run-mapreduce) 部分中的可选步骤，将所有 CRLF 转换为 LF。

* 对于 Windows 客户端，需要 PuTTY 和 PSCP。这些实用程序可通过 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">PuTTY 下载页</a>获得。

##字数统计

在本示例中，你将通过使用映射器和化简器实现基本字数统计。映射器会将句子分解成不同的单词，而化简器会汇总单词并统计字数以生成输出。

下图演示了在映射和化简阶段发生的情况。

![映射化简示意图](./media/hdinsight-hadoop-streaming-python/HDI.WordCountDiagram.png)

##为何要使用 Python？

Python 是一种通用高级编程语言，可让你以少于其他许多语言的代码行快速表达概念。数据科学家最近很喜欢将它用作原型设计语言，因为它的自释性、动态类型化和简洁语法非常适合用于快速开发应用程序。

Python 已安装在所有 HDInsight 群集上。

##流式处理 MapReduce

Hadoop 允许你指定包含作业所用映射和化简逻辑的文件。映射和化简逻辑的具体要求如下：

* **输入**：映射和化简组件必须从 STDIN 读取输入数据。

* **输出**：映射和化简组件必须将输出数据写入到 STDOUT。

* **数据格式**：使用和生成的数据必须是键/值对，并以制表符分隔。

Python 可以使用 **sys** 模块从 STDIN 读取数据，并使用 **print** 输出到 STDOUT，从而可以轻松应对这些要求。余下的任务就是在键和值之间以制表符 (`\t`) 设置数据的格式。

##创建映射器和化简器

映射器和化简器是文本文件，在这种情况下是 **mapper.py** 和 **reducer.py**（以使它清楚该做什么）。你可以通过使用自选的编辑器创建这些文件。

###Mapper.py

创建名为 **mapper.py** 的新文件并使用以下代码作为内容：

    #!/usr/bin/env python

    # Use the sys module
    import sys

    # 'file' in this case is STDIN
    def read_input(file):
        # Split each line into words
        for line in file:
            yield line.split()

    def main(separator='\t'):
        # Read the data using read_input
        data = read_input(sys.stdin)
        # Process each words returned from read_input
        for words in data:
            # Process each word
            for word in words:
                # Write to STDOUT
                print '%s%s%d' % (word, separator, 1)

    if __name__ == "__main__":
        main()

花些时间通读代码，以便你可以了解它的功能。

###Reducer.py

创建名为 **reducer.py** 的新文件并使用以下代码作为内容：

    #!/usr/bin/env python

    # import modules
    from itertools import groupby
    from operator import itemgetter
    import sys

    # 'file' in this case is STDIN
    def read_mapper_output(file, separator='\t'):
        # Go through each line
        for line in file:
            # Strip out the separator character
            yield line.rstrip().split(separator, 1)

    def main(separator='\t'):
        # Read the data using read_mapper_output
        data = read_mapper_output(sys.stdin, separator=separator)
        # Group words and counts into 'group'
        #   Since MapReduce is a distributed process, each word
        #   may have multiple counts. 'group' will have all counts
        #   which can be retrieved using the word as the key.
        for current_word, group in groupby(data, itemgetter(0)):
            try:
                # For each word, pull the count(s) for the word
                #   from 'group' and create a total count
                total_count = sum(int(count) for current_word, count in group)
                # Write to stdout
                print "%s%s%d" % (current_word, separator, total_count)
            except ValueError:
                # Count was not a number, so do nothing
                pass

    if __name__ == "__main__":
        main()

##上载文件

**mapper.py** 和 **reducer.py** 都必须位于群集的头节点上才能运行。要上载这些文件，最简单的方法是使用 **scp**（如果使用的是 Windows 客户端，则是 **pscp**）。

从客户端上与 **mapper.py** 和 **reducer.py** 相同的目录中，使用以下命令。将 **username** 替换为 SSH 用户，将 **clustername** 替换为群集名称。

	scp mapper.py reducer.py username@clustername-ssh.azurehdinsight.cn:

这样就会将两个文件从本地系统复制到头节点。

> [AZURE.NOTE] 如果使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果使用了 SSH 密钥，则可能需要使用 `-i` 参数和私钥路径，例如 `scp -i /path/to/private/key mapper.py reducer.py username@clustername-ssh.azurehdinsight.cn:`。

## <a name="run-mapreduce"></a>运行 MapReduce

1. 通过使用 SSH 连接到群集：

		ssh username@clustername-ssh.azurehdinsight.cn

	> [AZURE.NOTE] 如果使用了密码来保护 SSH 帐户，系统会提示你输入密码。如果使用了 SSH 密钥，则可能需要使用 `-i` 参数和私钥路径，例如 `ssh -i /path/to/private/key username@clustername-ssh.azurehdinsight.cn`。

2. （可选）在创建 mapper.py 和 reducer.py 文件时，如果使用的文本编辑器使用 CRLF 作为行尾，或者不知道该编辑器使用哪种行尾，请使用以下命令将 mapper.py 和 reducer.py 中出现的 CRLF 转换为 LF。

        perl -pi -e 's/\r\n/\n/g' mappery.py
        perl -pi -e 's/\r\n/\n/g' reducer.py

2. 使用以下命令启动 MapReduce 作业。

		yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -files mapper.py,reducer.py -mapper mapper.py -reducer reducer.py -input wasbs:///example/data/gutenberg/davinci.txt -output wasbs:///example/wordcountout

	此命令包括以下几个部分：

	* **hadoop-streaming.jar**：运行流式处理 MapReduce 操作时使用。它可以将 Hadoop 和你提供的外部 MapReduce 代码连接起来。

	* **-files**：告知 Hadoop 此 MapReduce 作业需要指定的文件，而且应将这些文件复制到所有辅助节点。

	* **-mapper**：告诉 Hadoop 要用作映射器的文件。

	* **-reducer**：告诉 Hadoop 要用作化简器的文件。

	* **-input**：要从中统计字数的输入文件。

	* **-output**：要将输出写入到的目录。

		> [AZURE.NOTE] 该作业会创建此目录。

作业启动后，你会看到一堆 **INFO** 语句，最后会看到以百分比显示的**映射**和**化简**操作。

	15/02/05 19:01:04 INFO mapreduce.Job:  map 0% reduce 0%
	15/02/05 19:01:16 INFO mapreduce.Job:  map 100% reduce 0%
	15/02/05 19:01:27 INFO mapreduce.Job:  map 100% reduce 100%

在作业完成时，你将收到有关作业的状态信息。

##查看输出

在作业完成后，使用以下命令查看输出：

	hdfs dfs -text /example/wordcountout/part-00000

这应会显示文本及文本出现次数的列表。下面是输出数据的示例：

	wrenching       1
	wretched        6
	wriggling       1
	wrinkled,       1
	wrinkles        2
	wrinkling       2

##后续步骤

了解如何将流式处理 MapRedcue 作业用于 HDInsight 后，就可以使用以下链接来学习 Azure HDInsight 的其他用法。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->