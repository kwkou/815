<properties
   pageTitle="使用 HDInsight 中的 Hive 分析和处理 JSON 文档 | Azure"
   description="了解如何使用 JSON 文档，以及如何使用 HDInsight 中的 Hive 来分析这些文档。"
   services="hdinsight"
   documentationCenter=""
   authors="rashimg"
   manager="mwinkle"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.date="06/22/2015"
   wacn.date="12/15/2015"/>


# 使用 HDInsight 中的 Hive 处理和分析 JSON 文档

了解如何使用 HDInsight 中的 Hive 处理和分析 JSON 文件。本教程将使用以下 JSON 文档

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

可以在 wasb://processjson@hditutorialdata.blob.core.windows.net/ 上找到该文件。有关将 Azure Blob 存储与 HDInsight 配合使用的详细信息，请参阅[将 HDFS 兼容的 Azure Blob 存储与 HDInsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。如果需要，你可以将该文件复制到群集的默认容器。

在本教程中，你将使用 Hive 控制台。有关打开 Hive 控制台的说明，请参阅[通过远程桌面将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-use-hive-remote-desktop/)。

##平展 JSON 文档

下一部分中所列的方法需要 JSON 文档在单一行中。因此，你必须将 JSON 文档平展成字符串。如果 JSON 文档已平展，则你可以跳过此步骤，直接转到有关分析 JSON 数据的下一部分。

	DROP TABLE IF EXISTS StudentsRaw;
	CREATE EXTERNAL TABLE StudentsRaw (textcol string) STORED AS TEXTFILE LOCATION "wasb://processjson@hditutorialdata.blob.core.windows.net/";
	
	DROP TABLE IF EXISTS StudentsOneLine;
	CREATE EXTERNAL TABLE StudentsOneLine
	(
	  json_body string
	)
	STORED AS TEXTFILE LOCATION '/json/students';
	
	INSERT OVERWRITE TABLE StudentsOneLine
	SELECT CONCAT_WS(' ',COLLECT_LIST(textcol)) AS singlelineJSON 
	      FROM (SELECT INPUT__FILE__NAME,BLOCK__OFFSET__INSIDE__FILE, textcol FROM StudentsRaw DISTRIBUTE BY INPUT__FILE__NAME SORT BY BLOCK__OFFSET__INSIDE__FILE) x
	      GROUP BY INPUT__FILE__NAME;
	
	SELECT * FROM StudentsOneLine

原始 JSON 文件位于 **wasb://processjson@hditutorialdata.blob.core.windows.net/**。 *StudentsRaw* Hive 表指向原始的未平展 JSON 文档。

*StudentsOneLine* Hive 表将数据存储在 HDInsight 默认文件系统中的 */json/students/* 路径下。

INSERT 语句在 StudentOneLine 表中填充平展的 JSON 数据。

SELECT 语句应只返回 1 行。

下面是 SELECT 语句的输出：

![平展 JSON 文档。][image-hdi-hivejson-flatten]

##在 Hive 中分析 JSON 文档

Hive 提供了三种不同的机制用于对 JSON 文档运行查询：

- 使用 GET\_JSON\_OBJECT UDF（用户定义的函数）
- 使用 JSON\_TUPLE UDF
- 使用自定义 SerDe
- 使用 Python 或其他语言编写你自己的 UDF。请参阅[此文][hdinsight-python]，了解如何使用 Hive 运行自己的 Python 代码。 

### 使用 GET\_JSON\_OBJECT UDF
Hive 提供了名为 [get\_json\_object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object) 的内置 UDF，它可以在运行时执行 JSON 查询。此方法采用两个参数 – 表名称和方法名称，具有平展的 JSON 文档和需要进行分析的 JSON 字段。让我们来探讨一个示例，以了解此 UDF 的工作原理。

获取每个学生的名字和姓氏

	SELECT 
	  GET_JSON_OBJECT(StudentsOneLine.json_body,'$.StudentDetails.FirstName'), 
	  GET_JSON_OBJECT(StudentsOneLine.json_body,'$.StudentDetails.LastName') 
	FROM StudentsOneLine;

这是在控制台窗口中运行此查询时的输出。

![get\_json\_object UDF][image-hdi-hivejson-getjsonobject]

get-json\_object UDF 有一些限制。

- 由于查询中的每个字段都需要重新分析查询，因此会影响性能。
- GET\_JSON\_OBJECT() 返回数组的字符串表示形式。若要将其转换为 Hive 数组，你必须使用正则表达式来替换方括号“[”和“]”，然后调用拆分来获取数组。


正因如此，Hive wiki 建议使用 json\_tuple。

### 使用 JSON\_TUPLE UDF

Hive 提供的另一个 UDF 称为 [json\_tuple](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-json_tuple)，其性能比 [get\_ json \_object](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object) 要高。此方法采用一组键和一个 JSON 字符串，并使用一个函数返回值的元组。以下查询将从 JSON 文档返回学生 ID 和年级：

    SELECT q1.StudentId, q1.Grade 
      FROM StudentsOneLine jt
      LATERAL VIEW JSON_TUPLE(jt.json_body, 'StudentId', 'Grade') q1
        AS StudentId, Grade;

此脚本在 Hive 控制台中的输出：

![json\_tuple UDF][image-hdi-hivejson-jsontuple]

JSON\_TUPLE 在 Hive 中使用了[横向视图](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView)语法，使 json\_tuple 能够通过将 UDT 函数应用于原始表的每一行来创建虚拟表。由于重复使用横向视图，复杂的 JSON 会变得过于庞大。此外，JSON_TUPLE 无法处理嵌套的 JSONs。


###使用自定义 SerDe

SerDe 是用于分析嵌套 JSON 文档的最佳选择，它可让你定义 JSON 架构，并使用该架构来分析文档。在本教程中，你将使用其中 [rcongiu](https://github.com/rcongiu) 开发的热门 SerDe 之一。

**使用自定义 SerDe：**

1. 安装 [Java SE 开发工具包 7u55 JDK 1.7.0_55](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u55-oth-JPR)。如果你要使用 HDInsight 的 Windows 部署，请选择 Windows X64 版本的 JDK。

	>[AZURE.WARNING]JDK 1.8 不适用于此 SerDe。

	安装完成之后，请添加新的用户环境变量：

	1. 从 Windows 屏幕打开“查看高级系统设置”。
	2. 单击“环境变量”。  
	3. 添加指向 **C:\\Program Files\\Java\\jdk1.7.0\_55** 或任何 JDK 安装位置的新 **JAVA_HOME** 环境变量。

	![为 JDK 设置正确的配置值][image-hdi-hivejson-jdk]

2. 安装 [Maven 3.3.1](http://mirror.olnevhost.net/pub/apache/maven/maven-3/3.3.1/binaries/apache-maven-3.3.1-bin.zip)

	转到“控件面板”->“编辑系统变量”（对应于你帐户的环境变量），将 bin 文件夹添加到你的路径。以下屏幕截图显示了如何执行此操作。

	![设置 Maven][image-hdi-hivejson-maven]

3. 从 [Hive-JSON-SerDe](https://github.com/sheetaldolas/Hive-JSON-Serde/tree/master) github 站点克隆项目。可以通过按以下屏幕截图中所示单击“下载 Zip”按钮来实现此目的。

	![克隆项目][image-hdi-hivejson-serde]

4：转到此包下载到的文件夹，然后键入“mvn package”。这应会创建必要的 jar 文件，然后你可以将其复制到群集。

5：转到根文件夹下存放所下载包的目标文件夹。将 json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar 文件上载到群集的头节点。我通常会将它放在 hive bin 文件夹下：C:\\apps\\dist\\hive-0.13.0.2.1.11.0-2316\\bin 或类似的文件夹。
 
6：在 hive 提示符下，键入“add jar /path/to/json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar”。由于在我的示例中，jar 在 C:\\apps\\dist\\hive-0.13.x\\bin 文件夹中，因此我可以直接添加名称如下的 jar：

    add jar json-serde-1.1.9.9-Hive13-jar-with-dependencies.jar;

	![Adding JAR to your project][image-hdi-hivejson-addjar]

现在，你可以使用 SerDe 对 JSON 文档运行查询。

以下语句将创建使用所定义架构的表

    DROP TABLE json_table;
    CREATE EXTERNAL TABLE json_table (
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
    ) ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
    LOCATION '/json/students';

列出学生的名字和姓氏

    SELECT StudentDetails.FirstName, StudentDetails.LastName FROM json_table;

下面是 Hive 控制台输出的结果。

![SerDe 查询 1][image-hdi-hivejson-serde_query1]

计算 JSON 文档的总分

    SELECT SUM(scores)
    FROM json_table jt
      lateral view explode(jt.StudentClassCollection.Score) collection as scores;
       
以上查询使用 [lateral view explode](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView) UDF 展开分数数组，以便可以求和。

下面是 Hive 控制台的输出。

![SerDe 查询 2][image-hdi-hivejson-serde_query2]

查找给定的学生在哪些科目取得了 80 以上的分数 SELECT jt.StudentClassCollection.ClassId FROM json\_table jt lateral view explode(jt.StudentClassCollection.Score) collection as score where score > 80;
      
上面的查询将返回一个 Hive 数组，这与 get\_json\_object 不同，后者返回一个字符串。

![SerDe 查询 3][image-hdi-hivejson-serde_query3]

如果你想要跳过格式不正确的 JSON，可以根据此 SerDe 的 [wiki 页面](https://github.com/sheetaldolas/Hive-JSON-Serde/tree/master)中所述，通过键入以下代码实现此目的：

    ALTER TABLE json_table SET SERDEPROPERTIES ( "ignore.malformed.json" = "true");




##摘要
总之，在 Hive 中选择的 JSON 运算符类型取决于你的方案。如果你有一个简单的 JSON 文档，并只有一个要查找的字段，则你可以选择使用 Hive UDF get\_json\_object。如果你有多个键用于查找，则可以使用 json\_tuple。如果你有嵌套的文档，则应使用 JSON SerDe。

如需其他相关文章，请参阅

- [将 Hive 和 HiveQL 与 HDInsight 中的 Hadoop 配合使用以分析示例 Apache log4j 文件](/documentation/articles/hdinsight-use-hive/)
- [使用 HDInsight 中的 Hive 分析航班延误数据](/documentation/articles/hdinsight-analyze-flight-delay-data/)


[hdinsight-python]: /documentation/articles/hdinsight-python/

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

<!---HONumber=71-->