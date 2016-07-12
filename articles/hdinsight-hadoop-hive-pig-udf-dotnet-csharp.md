<properties
	pageTitle="在 HDInsight 中的 Hadoop 上将 C# 与 Hive 和 Pig 配合使用 | Azure"
	description="了解在 Azure HDInsight 中如何将 C# 用户定义的函数 (UDF) 与 Hive 和 Pig 流式处理配合使用。"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/22/2016"
	wacn.date="06/29/2016"/>


#在 HDInsight 中的 Hadoop 上将 C# 用户定义函数与 Hive 和 Pig 流式处理配合使用

Hive 和 Pig 非常适合用于处理 Azure HDInsight 中的数据，但有时你需要一种更通用的语言。Hive 和 Pig 都允许你通过用户定义的函数 (UDF) 或流式处理来调用外部代码。

在本文档中，你将了解如何将 C# 与 Hive 和 Pig 配合使用。

##先决条件

* Windows 7、Windows 8 或 Windows 8.1。

* 以下版本的 Visual Studio：

	* 包含 [Update 4](http://www.microsoft.com/download/details.aspx?id=39305) 的 Visual Studio 2012 Professional/Premium/Ultimate

	* 包含 [Update 4](https://www.microsoft.com/download/details.aspx?id=44921) 的 Visual Studio 2013 Community/Professional/Premium/Ultimate

	* Visual Studio 2015 预览版

* HDInsight 群集上的 Hadoop - 有关创建群集的步骤，请参阅[设置 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters-v1/)

* Hadoop Tools for Visual Studio。有关安装和配置该工具的步骤，请参阅[开始使用 HDInsight Hadoop Tools for Visual Studio](/documentation/articles/hdinsight-hadoop-visual-studio-tools-get-started/)。

##HDInsight 上的 .NET

基于 Windows 的 HDInsight 群集上会默认安装 .NET 通用语言运行时 (CLR) 和框架。这允许你将 C# 应用程序与 Hive 和 Pig 流式处理配合使用（通过 stdout/stdin 在 Hive/Pig 与 C# 应用程序之间传递数据）。


##.NET 和流式处理

流式处理涉及 Hive 和 Pig 将数据通过 stdout 传递到外部应用程序，并通过 stdin 接收结果。对于 C# 应用程序，这可以通过 `Console.ReadLine()` 和 `Console.WriteLine()` 非常轻松地实现。

因为 Hive 和 Pig 在运行时需要调用应用程序，因此，应当为你的 C# 项目使用**控制台应用程序**模板。

##Hive 和 C&#35;

###创建 C# 项目

1. 打开 Visual Studio 并创建一个新的解决方案。对于项目类型，选择“控制台应用程序”，并将新项目命名为“HiveCSharp”。

2. 将 **Program.cs** 的内容替换为以下内容：

		using System;
		using System.Security.Cryptography;
		using System.Text;
		using System.Threading.Tasks;

		namespace HiveCSharp
		{
		    class Program
		    {
		        static void Main(string[] args)
		        {
		            string line;
		            // Read stdin in a loop
		            while ((line = Console.ReadLine()) != null)
		            {
		                // Parse the string, trimming line feeds
		                // and splitting fields at tabs
		                line = line.TrimEnd('\n');
		                string[] field = line.Split('\t');
		                string phoneLabel = field[1] + ' ' + field[2];
		                // Emit new data to stdout, delimited by tabs
		                Console.WriteLine("{0}\t{1}\t{2}", field[0], phoneLabel, GetMD5Hash(phoneLabel));
		            }
		        }
		        /// <summary>
		        /// Returns an MD5 hash for the given string
		        /// </summary>
		        /// <param name="input">string value</param>
		        /// <returns>an MD5 hash</returns>
		        static string GetMD5Hash(string input)
		        {
		            // Step 1, calculate MD5 hash from input
		            MD5 md5 = System.Security.Cryptography.MD5.Create();
		            byte[] inputBytes = System.Text.Encoding.ASCII.GetBytes(input);
		            byte[] hash = md5.ComputeHash(inputBytes);

		            // Step 2, convert byte array to hex string
		            StringBuilder sb = new StringBuilder();
		            for (int i = 0; i < hash.Length; i++)
		            {
		                sb.Append(hash[i].ToString("x2"));
		            }
		            return sb.ToString();
		        }
		    }
		}

3. 生成项目。

###上载到存储

1. 在 Visual Studio 中，打开“服务器资源管理器”。

3. 依次展开“Azure”和“HDInsight”。

4. 如果出现提示，请输入你的 Azure 订阅凭据，然后单击“登录”。

5. 展开你要将此应用程序部署到的 HDInsight 群集，然后展开“默认存储帐户”。

	![显示群集存储帐户的服务器资源管理器](./media/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/storage.png)

6. 双击群集的“默认容器”。这将打开一个新窗口，其中显示了默认容器的内容。

7. 单击上载图标，然后浏览到 **HiveCSharp** 项目的 **bin\\debug** 文件夹。最后，选择 HiveCSharp.exe 文件并单击“确定”。

	![上载图标](./media/hdinsight-hadoop-hive-pig-udf-dotnet-csharp/upload.png)

8. 在上载完成后，你将能够通过 Hive 查询使用应用程序。

###Hive 查询

1. 在 Visual Studio 中，打开“服务器资源管理器”。

2. 依次展开“Azure”和“HDInsight”。

5. 右键单击你已将“HiveCSharp”应用程序部署到的群集，然后选择“编写 Hive 查询”。

6. 请为 Hive 查询使用以下内容：

		add file wasb:///HiveCSharp.exe;

		SELECT TRANSFORM (clientid, devicemake, devicemodel)
		USING 'HiveCSharp.exe' AS
		(clientid string, phoneLabel string, phoneHash string)
		FROM hivesampletable
		ORDER BY clientid LIMIT 50;

    这将从 `hivesampletable` 中选择 `clientid`、`devicemake` 和 `devicemodel` 字段并将这些字段传递到 HiveCSharp.exe 应用程序。该查询预期应用程序返回三个字段，它们将存储为 `clientid`、`phoneLabel` 和 `phoneHash`。该查询还预期在默认存储容器的根目录中找到 HiveCSharp.exe (`add file wasb:///HiveCSharp.exe`)。

5. 单击“提交”将作业提交到 HDInsight 群集。此时将打开“Hive 作业摘要”窗口。

6. 单击“刷新”以刷新摘要，直到“作业状态”变为“已完成”。若要查看作业输出，请单击“作业输出”。

##Pig 和 C&#35;

###创建 C# 项目

1. 打开 Visual Studio 并创建一个新的解决方案。对于项目类型，选择“控制台应用程序”，并将新项目命名为“PigUDF”。

2. 将 **Program.cs** 文件的内容替换为以下内容：

		using System;

		namespace PigUDF
		{
		    class Program
		    {
		        static void Main(string[] args)
		        {
		            string line;
				    // Read stdin in a loop
				    while ((line = Console.ReadLine()) != null)
		            {
		                // Fix formatting on lines that begin with an exception
		                if(line.StartsWith("java.lang.Exception"))
		                {
		                    // Trim the error info off the beginning and add a note to the end of the line
		                    line = line.Remove(0, 21) + " - java.lang.Exception";
		                }
		                // Split the fields apart at tab characters
		                string[] field = line.Split('\t');
		                // Put fields back together for writing
		                Console.WriteLine(String.Join("\t",field));
		            }
		        }
		    }
		}

	此应用程序将分析发送自 Pig 的行，并对以 `java.lang.Exception` 开头的行重新设置格式。

3. 保存 **Program.cs**，然后生成项目。

###上载应用程序

1. Pig 流式处理预期应用程序位于群集文件系统本地。为 HDInsight 群集启用远程桌面，然后根据[使用 RDP 连接到 HDInsight 群集](/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp)中的说明连接到该群集。

2. 在连接后，从你的本地计算机上 PigUDF 项目的 **bin/debug** 目录中复制 **PigUDF.exe**，并将其粘贴到群集上的 **%PIG\_HOME%** 目录中。

###通过 Pig Latin 使用应用程序

1. 从远程桌面会话中，使用桌面上的“Hadoop 命令行”图标启动 Hadoop 命令行。

2. 使用以下命令来启动 Pig 命令行：

		cd %PIG_HOME%
		bin\pig

	系统将为你提供 `grunt>` 提示符。

3. 输入以下命令以使用 .NET Framework 应用程序运行简单的 Pig 作业：

		DEFINE streamer `pigudf.exe` SHIP('pigudf.exe');
		LOGS = LOAD 'wasb:///example/data/sample.log' as (LINE:chararray);
		LOG = FILTER LOGS by LINE is not null;
		DETAILS = STREAM LOG through streamer as (col1, col2, col3, col4, col5);
		DUMP DETAILS;

	`DEFINE` 语句为 pigudf.exe 应用程序创建别名 `streamer`，`SHIP` 将其在群集的节点中进行分发。以后，可以将 `streamer` 与 `STREAM` 运算符配合使用来处理 LOG 中包含的单一行，并将数据返回为一系列的列。

> [AZURE.NOTE]用于流式处理的应用程序名称在使用别名时必须用 `（反引号）字符括起来，当与 `SHIP` 一起使用时必须用 '（单引号）括起来。

3. 在输入最后一行后，该作业应该启动。最终，它将返回类似于以下内容的输出：

		(2012-02-03 20:11:56 SampleClass5 [WARN] problem finding id 1358451042 - java.lang.Exception)
		(2012-02-03 20:11:56 SampleClass5 [DEBUG] detail for id 1976092771)
		(2012-02-03 20:11:56 SampleClass5 [TRACE] verbose detail for id 1317358561)
		(2012-02-03 20:11:56 SampleClass5 [TRACE] verbose detail for id 1737534798)
		(2012-02-03 20:11:56 SampleClass7 [DEBUG] detail for id 1475865947)

##摘要

在本文档中，你已了解了如何在 HDInsight 上通过 Hive 和 Pig 使用 .NET Framework 应用程序。如果希望了解如何将 Python 与 Hive 和 Pig 配合使用，请参阅[在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用](/documentation/articles/hdinsight-python/)。

若要了解使用 Pig、Hive 的其他方式以及如何使用 MapReduce，请参阅以下文章：

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=79-->