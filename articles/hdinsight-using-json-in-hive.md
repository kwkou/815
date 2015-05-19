<properties
   pageTitle="使用 HDInsight 中的 Hive 分析和处理 JSON 文档 | Microsoft Azure"
   description="了解如何使用 JSON 文档，以及如何使用 HDInsight 中的 Hive 来分析这些文档"
   services="hdinsight"
   documentationCenter=""
   authors="rashimg"
   manager="mwinkle"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="04/14/2015"
   wacn.date="05/15/2015"
   ms.author="rashimg"/>


# 了解如何在 Hive 中处理和分析 JSON 文档

JSON 是 Web 上最常用的格式之一。本教程将帮助你了解用户在 Hive 遇到的常见问题之一 - 如何在 Hive 中使用 JSON 文档，然后对其运行分析。 

## JSON 示例

让我们举个例子。假设我们有一个如下所示的 JSON 文档。我们的目标是要分析此 JSON 文档，然后对此文档运行查询，例如，查找某个键的值、聚合，等等。

	  {
        "StudentId": "trgfg-5454-fdfdg-4346",
	    "Grade": 7,
        "StudentDetails": [
          {
            "FirstName": "Peggy",
            "LastName": "Williams",
            "YearJoined": 2012
          }
        ],
        "StudentClassCollection": [
          {
            "ClassId": "89084343",
            "ClassParticipation": "Satisfied",
            "ClassParticipationRank": "High",
            "Score": 93,
            "PerformedActivity": false
          },
          {
            "ClassId": "78547522",
            "ClassParticipation": "NotSatisfied",
            "ClassParticipationRank": "None",
            "Score": 74,
            "PerformedActivity": false
          },
          {
            "ClassId": "78675563",
            "ClassParticipation": "Satisfied",
            "ClassParticipationRank": "Low",
            "Score": 83,
            "PerformedActivity": true
          }
        ] 
      }

## 平展 JSON 文档（仅当你的 JSON 很完善时才需要执行的步骤）

在我们使用任何 Hive 运算符执行分析之前，必须预处理 JSON 文档，使它随时可供 Hive 使用。 

下一部分中列出的所有选项都假设 JSON 文档在单个行中。换而言之，我们必须将 JSON 文档平展成一个字符串，使它可供 Hive 运算符使用。 

**如果 JSON 文档已平展，并且整个文档在一行中，则你可以跳过此步骤，直接转到有关分析 JSON 数据的下一部分。**

让我们假设跨多行的未平展 JSON 文档出现在 Azure Blob 存储中默认容器下的 */json/input/* 文件夹下。以下代码将创建表 jsonexample，该表指向原始的未平展 JSON 文档。

    drop table jsonexample;
    CREATE EXTERNAL TABLE jsonexample (textcol string) stored as textfile location '/json/input/';

接下来，我们将创建名为 one_line_json 的新表，该表将指向平展的 JSON 文档，并将存储在 Azure Blob 存储的默认容器下的 */json/temp/* 文件夹下。 

    drop table if exists one_line_json;
    create external table one_line_json
    (
      json_body string
    )
    stored as textfile location '/json/temp';

最后，我们将在此表中填充平展 JSON 的数据。

    insert overwrite table one_line_json
    select concat_ws(' ',collect_list(textcol)) as singlelineJSON 
      from (select INPUT__FILE__NAME,BLOCK__OFFSET__INSIDE__FILE,textcol from jsonexample distribute by INPUT__FILE__NAME sort by BLOCK__OFFSET__INSIDE__FILE ) 
      x group by INPUT__FILE__NAME;

此时，如果你查看新建的表 one_line_json，你应会发现它在单个行中包含了该 JSON 文档

    select * from one_line_json;

下面是此查询的输出：

![Flattening of the JSON document.][image-hdi-hivejson-flatten]

## 用于在 Hive 中分析 JSON 文档的选项

现在，包含单个列的表中有了 JSON 文档，我们可以使用 Hive 对此数据运行查询。Hive 提供了三种不同的机制用于对 JSON 文档运行查询：

1.	使用用户定义的函数 (UDF)：get_json_object
2.	使用 UDF：json_tuple
3.	使用自定义 SerDe

让我们详细讨论其中的每种机制。

## get_json_object UDF
Hive 提供了名为 [get_json_object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object) 的内置 UDF，它可以在运行时执行 JSON 查询。此方法采用两个参数 - 表名称和方法名称，具有平展的 JSON 文档和需要进行分析的 JSON 字段。让我们来探讨一个示例，以了解此 UDF 的工作原理。

获取每个学生的名字和姓氏

    select get_json_object(one_line_json.json_body, '$.StudentDetails.FirstName'), get_json_object(one_line_json.json_body, '$.StudentDetails.LastName') from one_line_json;

这是在控制台窗口中运行此查询时的输出。

![get_json_object UDF][image-hdi-hivejson-getjsonobject]

此 UDF 有一些限制。 


- 主要限制之一在于，此 UDF 的性能不高，因为查询中的每个字段都需要重新分析查询，因而会影响性能。
- 其次，get_json_object() 返回数组的字符串表示形式。因此，若要将其转换为 Hive 数组，你必须使用正则表达式来替换方括号"["和"]"，然后调用拆分来获取数组。


正因如此，Hive wiki 建议使用我们接下来要讨论的 json_tuple。  


## json_tuple UDF

Hive 提供的另一个 UDF 称为 [json_tuple](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-json_tuple)，它的性能高于 [get_json_object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object)。此方法采用一组键和一个 JSON 字符串，并使用一个函数返回值的元组。让我们来探讨一个示例，以了解此 UDF 的工作原理。 

让我们从 JSON 文档获取学生 id 和年级

    select q1.StudentId, q1.Grade 
      from one_line_json jt
      LATERAL VIEW json_tuple(jt.json_body, 'StudentId', 'Grade') q1
        as StudentId, Grade;

我们在 Hive 中使用了[横向视图](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView)语法，使 json_tuple 能够通过将 UDT 函数应用于原始表的每一行来创建虚拟表。 

让我们在 Hive 控制台中看一下此脚本的输出：

![json_tuple UDF][image-hdi-hivejson-jsontuple]

此 UDF 的主要缺点之一是，由于重复使用横向视图，复杂的 JSON 会变得过于庞大。事实上，此 UDF 无法处理嵌套的 JSON。

## 使用自定义 SerDe

要分析嵌套的 JSON 文档，SerDe 是**最好的选择**，因为你可以定义 JSON 架构，使其很容易运行查询。 

我们将讨论如何使用 [rcongiu](https://github.com/rcongiu) 开发的较热门 SerDe 之一。

步骤 1：确保已安装 [Java SE 开发工具包 7u55 JDK 1.7.0_55](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u55-oth-JPR)（注意：如果安装的是 JDK 1.8，该版本不能配合 SerDe）。 


- 如果你打算使用 HDInsight 的 Windows 部署，请选择 x64 Windows 版本的 JDK。
- 完成安装后，请转到"控制面板"->"添加环境变量"并添加一个指向 C:\Program Files\Java\jdk1.7.0_55 或任意 JDK 安装位置的新 JAVA_HOME 环境变量。以下屏幕截图显示了如何设置该环境变量。

![Setting up correct config values for JDK][image-hdi-hivejson-jdk]

步骤 2：转到[此](http://mirror.olnevhost.net/pub/apache/maven/maven-3/3.3.1/binaries/apache-maven-3.3.1-bin.zip)链接，然后下载 Maven 3.3.1。将存档解包到你想要存储二进制文件的位置。在本例中，我要将其解压缩到 C:\Program Files\Maven\。转到"控件面板"->"编辑系统变量"（对应于你帐户的 nvironment 变量），将 bin 文件夹添加到你的路径。以下屏幕截图显示了如何执行此操作。 

![Setting up Maven][image-hdi-hivejson-maven]

步骤 3：从 [Hive-JSON-SerDe](https://github.com/sheetaldolas/Hive-JSON-Serde/tree/master) github 站点克隆项目。可以通过按以下屏幕截图中所示单击"下载 Zip"按钮来实现此目的。

![Cloning the project][image-hdi-hivejson-serde]

步骤 4：转到此包下载到的文件夹，然后键入"mvn package"。这应会创建必要的 jar 文件，然后你可以将其复制到群集。 

步骤 5：接下来，转到根文件夹下存放所下载包的目标文件夹。将 json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar 文件上载到群集的头节点。我通常会将它放在 hive bin 文件夹下：C:\apps\dist\hive-0.13.0.2.1.11.0-2316\bin 或类似的文件夹。
 
步骤 6：在 hive 提示符下，键入"add jar /path/to/json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar"。由于在我的示例中，jar 在 C:\apps\dist\hive-0.13.x\bin 文件夹中，因此我可以直接添加名称如下的 jar：

    add jar json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar;

![Adding JAR to your project][image-hdi-hivejson-addjar]

现在，我们可以使用 SerDe 对 JSON 文档运行查询。

让我们先创建具有 JSON 文档架构的表。请注意，我们可以处理丰富得多的类型，包括映射、数组和结构。在 Hive 控制台中键入以下代码，使 Hive 创建名为 json_table 的表，该表可以基于下面的架构读取 JSON 文档。 

    drop table json_table;
    create external table json_table (
      StudentId string, 
      Grade int,
      StudentDetails array<struct<
          FirstName:string,
          LastName:string,
          YearJoined:int
          >
      >,
      StudentClassCollection array<struct<
          ClassId:string,
          ClassParticipation:string,
          ClassParticipationRank:string,
          Score:int,
          PerformedActivity:boolean
          >
      >
    ) row format serde 'org.openx.data.jsonserde.JsonSerDe'
    location '/json/temp';

现在，我们定义了架构，让我们运行一些示例，以了解如何使用 Hive 来查询 JSON 文档：

a)	列出学生的名字和姓氏

    select StudentDetails.FirstName, StudentDetails.LastName from json_table;

下面是 Hive 控制台输出的结果。

![SerDe Query 1][image-hdi-hivejson-serde_query1]

b)	此查询将计算 JSON 文档的总分

    select sum(scores)
    from json_table jt
      lateral view explode(jt.StudentClassCollection.Score) collection as scores;
       
以上查询使用 [lateral view explode](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView) UDF 展开分数数组，以便可以求和。 

下面是 Hive 控制台的输出。

![SerDe Query 2][image-hdi-hivejson-serde_query2]

c)	查找给定的学生在哪些科目取得了 80 以上的分数
    select  
      jt.StudentClassCollection.ClassId 
    from json_table jt
      lateral view explode(jt.StudentClassCollection.Score) collection as score  where score > 80;
      
上面的查询将返回一个 Hive 数组，这与 get_json_object 不同，后者返回一个字符串。 

![SerDe Query 3][image-hdi-hivejson-serde_query3]

如果你想要跳过格式不正确的 JSON，可以根据此 SerDe 的 [wiki 页面](https://github.com/sheetaldolas/Hive-JSON-Serde/tree/master)中所述，通过键入以下代码实现此目的：  

    ALTER TABLE json_table SET SERDEPROPERTIES ( "ignore.malformed.json" = "true");


## 其他选项
除了下面列出的选项以外，你也可以使用 Python 或其他特定于方案的语言编写自己的自定义代码。准备好 python 脚本后，请参阅有关使用 Hive 运行自己的 Python 代码的[这篇文章][hdinsight-python]。 

## 摘要
总之，在 Hive 中选择的 JSON 运算符类型取决于你的方案。如果你有一个简单的 JSON 文档，并只有一个要查找的字段，则你可以选择使用 Hive UDF get_json_object。如果你有多个键用于查找，则可以使用 json_tuple。如果你有嵌套的文档，则应使用 JSON SerDe。

HDInsight 正在致力于让你更轻松地在 Hive 中使用更多的本机格式，且不会给你带来大量的工作。一旦我们有分享的内容，就会在本教程中更新更多的详情。  

[hdinsight-python]: /documentation/articles/hdinsight-python

[image-hdi-hivejson-flatten]: ./media/hdinsight-using-json-in-hive/flatten.png
[image-hdi-hivejson-getjsonobject]: ./media/hdinsight-using-json-in-hive/getjsonobject.png
[image-hdi-hivejson-jsontuple]: ./media/hdinsight-using-json-in-hive/jsontuple.png
[image-hdi-hivejson-jdk]: ./media/hdinsight-using-json-in-hive/jdk.png
[image-hdi-hivejson-maven]: ./media/hdinsight-using-json-in-hive/maven.png
[image-hdi-hivejson-serde]: ./media/hdinsight-using-json-in-hive/serde.png
[image-hdi-hivejson-addjar]: ./media/hdinsight-using-json-in-hive/addjar.png
[image-hdi-hivejson-serde_query1]: ./media/hdinsight-using-json-in-hive/serde_query1.png
[image-hdi-hivejson-serde_query2]: ./media/hdinsight-using-json-in-hive/serde_query2.png
[image-hdi-hivejson-serde_query3]: ./media/hdinsight-using-json-in-hive/serde_query3.png
[image-hdi-hivejson-serde_result]: ./media/hdinsight-using-json-in-hive/serde_result.png

<!--HONumber=53-->