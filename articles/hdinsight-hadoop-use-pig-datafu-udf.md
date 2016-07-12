<properties
pageTitle="在 HDInsight 上通过 Pig 使用 DataFu"
description="DataFu 是适用于 Hadoop 的库的集合。了解如何在 HDInsight 群集上通过 Pig 使用 DataFu。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/18/2016"
	wacn.date="04/26/2016"/>

#在 HDInsight 上通过 pig 使用 DataFu

DataFu 是适用于 Hadoop 的开放源代码库的集合。在本文档中，你将学习如何在 HDInsight 群集上使用 DataFu 以及如何通过 Pig 使用 DataFu 用户定义函数 (UDF)。

##先决条件

* Azure 订阅。

* Azure HDInsight 群集（基于 Windows）

* 基本熟悉[在 HDInsight 上使用 Pig](/documentation/articles/hdinsight-use-pig/)

##通过 Pig 使用 DataFu

本部分中的步骤假定你熟悉在 HDInsight 上使用 Pig，并仅向提供 Pig Latin 语句，而不是如何在群集上使用它们的步骤。有关在 HDInsight 上使用 Pig 的详细信息，请参阅[将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)。

通常，你将为 DataFu 函数定义别名。例如：

    DEFINE SHA datafu.pig.hash.SHA();
    
这将为 SHA 哈希函数定义名为 `SHA` 的别名。然后，你可以在 Pig Latin 脚本中使用此别名生成输入数据的哈希值。例如，以下代码将输入数据中的名称替换为哈希值：

    raw = LOAD '/data/raw/' USING PigStorage(',') AS  
        (name:chararray, 
        int1:int, 
        int2:int,
        int3:int); 
    mask = FOREACH raw GENERATE SHA(name), int1, int2, int3; 
    DUMP mask;

如果这用于以下输入数据：

    Lana Zemljaric,5,9,1
    Qiong Zhong,9,3,6
    Sandor Harsanyi,0,7,3
    Roko Petkovic,2,6,2
    Tibor Rozsa,8,0,0
    Lea Hrastovsek,6,3,6
    Regina Toth,2,1,2
    Eva Makay,8,9,2
    Shi Liao,4,6,0
    Tjasa Zemljaric,0,2,5
    
它会生成以下输出：

    (c1a743b0f34d349cfc2ce00ef98369bdc3dba1565fec92b4159a9cd5de186347,5,9,1)
    (713d030d621ab69aa3737c8ea37a2c7c724a01cd0657a370e103d8cdecac6f99,9,3,6)
    (7a5f5abdd281f68168199319d98a1a662535f988d1443b3a3c497010937bac89,0,7,3)
    (a94818e93807e12079c4b35f8f3c8c8ef8e8acd1954e7f0476bc1a3a86fc96a9,2,6,2)
    (894ead4f48af91df7e088241218a23157bede7c52115272e417e95c046d48902,8,0,0)
    (6f99f163af3448fda672087db306f363e27a98a9e49c1f274a0860e303f8aec4,6,3,6)
    (a03de92a28be3c6a984c7a153fa9ed81c0413f76a9401955b5f7e04a5dd0ab9f,2,1,2)
    (6ceab977c8fb48d9ad0dc413e6bc646cabd89f22e7ab97a6b0133f3d225c6013,8,9,2)
    (fa9c436469096ff1bd297e182831f460501b826272ae97e921f5f6e3f54747e8,4,6,0)
    (bc22db7c238b86c37af79a62c78f61a304b35143f6087eb99c34040325865654,0,2,5)

##后续步骤

有关 DataFu 或 Pig 的详细信息，请参阅以下文档：

* [Apache DataFu Pig 指南](http://datafu.incubator.apache.org/docs/datafu/guide.html)。

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

<!---HONumber=Mooncake_1207_2015-->