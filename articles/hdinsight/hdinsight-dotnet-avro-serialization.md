<properties
	pageTitle="使用 Microsoft Avro Library 序列化数据 | Azure"
	description="了解 Azure HDInsight 如何使用 Avro 来序列化大数据。"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian" 
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="05/04/2016"
	wacn.date="06/29/2016"/>


# 使用 Microsoft Avro Library 序列化 Hadoop 中的数据

本主题演示如何使用 <a href="https://hadoopsdk.codeplex.com/wikipage?title=Avro%20Library" target="_blank">Microsoft Avro Library</a> 将对象及其他数据结构序列化为流，以便将它们持久保存到内存、数据库或文件中，同时还演示如何对这些流进行反序列化以恢复原始对象。

[AZURE.INCLUDE [仅适用于 Windows](../includes/hdinsight-windows-only.md)]

##<a name="apacheAvro"></a>Apache Avro
<a href="https://hadoopsdk.codeplex.com/wikipage?title=Avro%20Library" target="_blank">Microsoft Avro Library</a> 针对 Microsoft.NET 环境实现了 Apache Avro 数据序列化系统。Apache Avro 为序列化提供了一种紧凑的二进制数据交换格式。它使用 <a href="http://www.json.org" target="_blank">JSON</a> 定义与语言无关的架构，以支持语言互操作性。以一种语言序列化的数据可以用另一种语言读取。目前支持 C、C++、C#、Java、PHP、Python 和 Ruby。有关格式的详细信息可以在 <a href="http://avro.apache.org/docs/current/spec.html" target="_blank">Apache Avro 规范</a>中找到。请注意，Microsoft Avro Library 的当前版本不支持此规范的远程过程调用 (RPC) 部分。

Avro 系统中的对象的序列化表示形式由两部分组成：架构和实际值。Avro 架构使用 JSON 描述已序列化数据的与语言无关的数据模型。它与数据的二进制表示形式并排显示。将架构与二进制表示形式分离，使写入每个对象时没有针对值的开销，从而实现快速序列化和较小的表示形式。

##<a name="hadoopScenario"></a>Hadoop 应用场景
Apache Avro 序列化格式广泛应用于 Azure HDInsight 及其他 Apache Hadoop 环境中。Avro 提供了简便的方法来表示 Hadoop MapReduce 作业内的复杂数据结构。Avro 文件（Avro 对象容器文件）格式已设计为支持分布式 MapReduce 编程模型。实现分布的关键功能是文件是“可拆分的”，也就是说，用户可以在文件中搜寻任一点，然后即可从某一特定块开始读取。

##<a name="serializationMAL"></a>Microsoft Avro Library 中的序列化
.NET Library for Avro 支持通过两种方式序列化对象：

- **反射** - 自动从要序列化的 .NET 类型的数据协定特性生成这些类型的 JSON 架构。
- **通用记录** - 当没有 .NET 类型可以用来描述要序列化的数据的架构时，系统会在以 [**AvroRecord**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.avrorecord.aspx) 类表示的记录中显式指定 JSON 架构。

当流的写入器和读取器都知道数据架构时，可以发送没有架构的数据。在未使用 Avro 对象容器文件的情况下，架构将存储在文件中。可以指定其他参数，例如用于数据压缩的编解码器。这些情况将在下面的代码示例中进一步详述和说明。


##<a name="prerequisites"></a> 安装 Avro Library

以下是安装此库之前所需具备的先决条件：

- <a href="http://www.microsoft.com/download/details.aspx?id=17851" target="_blank">Microsoft .NET Framework 4</a>
- <a href="http://james.newtonking.com/json" target="_blank">Newtonsoft Json.NET</a>（6.0.4 或更高版本）

请注意，Newtonsoft.Json.dll 依赖项已随着 Microsoft Avro Library 的安装自动下载。下一部分将提供此操作的相关过程。


Microsoft Avro Library 以 NuGet 包发行，你可以使用以下过程在 Visual Studio 中安装 NuGet 程序包：

1. 选择“项目”选项卡->“管理 NuGet 包...”
2. 在“联机搜索”框中，搜索“Microsoft.Hadoop.Avro”。
3. 单击“Azure HDInsight Avro Library”旁边的“安装”按钮。

请注意，Newtonsoft.Json.dll (>= 6.0.4) 依赖项也将随 Microsoft Avro Library 一起自动下载。

你可能需要浏览 <a href="https://hadoopsdk.codeplex.com/wikipage?title=Avro%20Library" target="_blank">Microsoft Avro Library 主页</a>以阅读最新的发行说明。


<a href="https://hadoopsdk.codeplex.com/wikipage?title=Avro%20Library" target="_blank">Microsoft Avro Library 主页</a>中提供了 Microsoft Avro Library 源代码。

##<a name="compiling"></a>使用 Avro Library 编译架构

Microsoft Avro Library 包含代码生成实用工具，可让你自动根据先前定义的 JSON 架构来创建 C# 类型。代码生成实用工具不是以二进制可执行文件的形式分发的，但你可使用以下过程轻松生成：

1. 从 <a href="http://hadoopsdk.codeplex.com/SourceControl/latest" target="_blank">Microsoft .NET SDK For Hadoop</a> 下载包含最新版 HDInsight SDK 源代码的 ZIP 文件。（单击“下载”图标。）

2. 将 HDInsight SDK 解压缩到已安装 .NET Framework 4.0 并连接到 Internet 的计算机上的目录，以下载必要的依赖项 NuGet 包。下面我们假设源代码已解压缩到 C:\\SDK。

3. 转到文件夹 C:\\SDK\\src\\Microsoft.Hadoop.Avro.Tools 并运行 build.bat。（此文件将从 .NET Framework 的 32 位分发版调用 MSBuild。如果你想要使用 64 位版本，请编辑 build.bat 文件注释后的列。） 确保生成成功。（在某些系统上，MSBuild 可能生成警告。只要没有生成错误，这些警告就不影响实用工具。）

4. 编译的实用工具位于 C:\\SDK\\Bin\\Unsigned\\Release\\Microsoft.Hadoop.Avro.Tools 中。


若要熟悉命令行语法，请从代码生成实用工具所在的文件夹运行以下命令：`Microsoft.Hadoop.Avro.Tools help /c:codegen`

若要测试实用工具，你可以从随着源代码提供的示例 JSON 架构文件生成 C# 类。运行以下命令：

	Microsoft.Hadoop.Avro.Tools codegen /i:C:\SDK\src\Microsoft.Hadoop.Avro.Tools\SampleJSON\SampleJSONSchema.avsc /o:

这应该在当前目录中生成两个 C# 文件：SensorData.cs 和 Location.cs。

若要了解代码生成实用工具在转换 JSON 架构为 C# 类型时使用的逻辑，请参阅 C:\\SDK\\src\\Microsoft.Hadoop.Avro.Tools\\Doc 中的 GenerationVerification.feature 文件。

请注意，该命名空间是使用上一个段落中提及的文件中所描述的逻辑，从 JSON 架构中提取。从架构提取的命名空间，将比实用工具命令行中使用 /n 参数提供的设置具有优先权。如果你想要重写架构中包含的命名空间，请确保使用 /nf 参数。例如，若要将所有命名空间从 SampleJSONSchema.avsc 更改为 my.own.nspace，请运行以下命令：

    Microsoft.Hadoop.Avro.Tools codegen /i:C:\SDK\src\Microsoft.Hadoop.Avro.Tools\SampleJSON\SampleJSONSchema.avsc /o:. /nf:my.own.nspace

##<a name="samples"></a> 示例
本主题中提供的六个示例演示了 Microsoft Avro Library 所支持的不同方案。Microsoft Avro Library 设计为可处理任何流。在这些示例中，为保持简单性和一致性，是使用内存流（而不是文件流或数据库）来操作数据的。在生产环境中所采取的方法将取决于实际的方案要求、数据源和卷、性能限制及其他因素。

前两个示例显示如何使用反射和通用记录将数据序列化到内存流缓冲区，以及如何进行反序列化。这两个方案假设在读取器和写入器之间共享架构。

第三和第四个示例说明如何使用 Avro 对象容器文件，将数据序列化与反序列化。当数据存储在 Avro 容器文件中时，其架构始终随之一起存储，因为必须共享架构才能进行反序列化。

包含前四个示例的样例可以从 <a href="https://github.com/Azure-Samples" target="_blank">Azure 代码示例</a>站点下载。

第五个示例演示如何将自定义压缩编解码器用于 Avro 对象容器文件。包含此示例代码的样例可以从 <a href="https://github.com/Azure-Samples" target="_blank">Azure 代码示例</a>站点下载。

第六个示例显示如何使用 Avro 序列化来上载数据到 Azure Blob 存储，然后使用具有 HDInsight (Hadoop) 群集的 Hive 加以分析。可以从 <a href="https://github.com/Azure-Samples" target="_blank">Azure 代码示例</a>站点下载该示例。

以下是本主题所讨论的六个示例的链接：

 * <a href="#Scenario1">**通过反射进行序列化**</a> - 自动从数据协定特性生成要序列化的类型的 JSON 架构。
 * <a href="#Scenario2">**通过通用记录进行序列化**</a> - 当没有可用于反射的 .NET 类型时，在记录中显式指定 JSON 架构。
 * <a href="#Scenario3">**使用对象容器文件与反射进行序列化**</a> - JSON 架构自动生成并使用 Avro 对象容器文件随着序列化的数据共享。
 * <a href="#Scenario4">**使用对象容器文件与通用记录进行序列化**</a> - JSON 架构是在序列化前显式指定的，并使用 Avro 对象容器文件随着序列化的数据共享。
 * <a href="#Scenario5">**使用对象容器文件和自定义压缩编解码器进行序列化**</a> - 该示例演示如何使用 Deflate 数据压缩编解码器的自定义 .NET 实现，来创建 Avro 对象容器文件。
 * <a href="#Scenario6">**使用 Avro 来上载 Azure HDInsight 服务的数据**</a> - 该示例演示 Avro 序列化如何与 HDInsight 服务交互。要运行此示例，你必须具备有效的 Azure 订阅并且可以访问 Azure HDInsight 群集。

###<a name="Scenario1"></a>示例 1：通过反射进行序列化

Microsoft Avro Library 可以使用反射从要序列化的 C# 对象的数据协定特性自动生成类型的 JSON 架构。Microsoft Avro Library 将创建一个 [**IAvroSeralizer<T>**](http://msdn.microsoft.com/zh-cn/library/dn627341.aspx) 以标识要序列化的字段。

在此示例中，将对象（具有成员 **Location** 结构的 **SensorData** 类）序列化到内存流，继而又将此流反序列化。然后，将结果与初始实例进行比较，以确认恢复的 **SensorData** 对象与原始对象相同。

此示例中的架构假定在读取器与写入器之间共享，因此无需采用 Avro 对象容器格式。有关在架构必须与数据一起共享时，如何使用反射和对象容器格式将数据序列化到内存缓冲区，以及如何对内存缓冲区中的数据进行反序列化的示例，请参阅<a href="#Scenario3">使用对象容器文件通过反射进行序列化</a>。

    namespace Microsoft.Hadoop.Avro.Sample
    {
        using System;
        using System.Collections.Generic;
        using System.IO;
        using System.Linq;
        using System.Runtime.Serialization;
        using Microsoft.Hadoop.Avro.Container;
		using Microsoft.Hadoop.Avro;

        //Sample class used in serialization samples
        [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
        internal class SensorData
        {
            [DataMember(Name = "Location")]
            public Location Position { get; set; }

            [DataMember(Name = "Value")]
            public byte[] Value { get; set; }
        }

        //Sample struct used in serialization samples
        [DataContract]
        internal struct Location
        {
            [DataMember]
            public int Floor { get; set; }

            [DataMember]
            public int Room { get; set; }
        }

        //This class contains all methods demonstrating
        //the usage of Microsoft Avro Library
        public class AvroSample
        {

            //Serialize and deserialize sample data set represented as an object using reflection.
            //No explicit schema definition is required - schema of serialized objects is automatically built.
            public void SerializeDeserializeObjectUsingReflection()
            {

                Console.WriteLine("SERIALIZATION USING REFLECTION\n");
                Console.WriteLine("Serializing Sample Data Set...");

                //Create a new AvroSerializer instance and specify a custom serialization strategy AvroDataContractResolver
                //for serializing only properties attributed with DataContract/DateMember
                var avroSerializer = AvroSerializer.Create<SensorData>();

                //Create a memory stream buffer
                using (var buffer = new MemoryStream())
                {
                    //Create a data set by using sample class and struct
                    var expected = new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } };

                    //Serialize the data to the specified stream
                    avroSerializer.Serialize(buffer, expected);


                    Console.WriteLine("Deserializing Sample Data Set...");

                    //Prepare the stream for deserializing the data
                    buffer.Seek(0, SeekOrigin.Begin);

                    //Deserialize data from the stream and cast it to the same type used for serialization
                    var actual = avroSerializer.Deserialize(buffer);

                    Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

                    //Finally, verify that deserialized data matches the original one
                    bool isEqual = this.Equal(expected, actual);

                    Console.WriteLine("Result of Data Set Identity Comparison is {0}", isEqual);

                }
            }

            //
            //Helper methods
            //

            //Comparing two SensorData objects
            private bool Equal(SensorData left, SensorData right)
            {
                return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }



            static void Main()
            {

                string sectionDivider = "---------------------------------------- ";

                //Create an instance of AvroSample Class and invoke methods
                //illustrating different serializing approaches
                AvroSample Sample = new AvroSample();

                //Serialization to memory using reflection
                Sample.SerializeDeserializeObjectUsingReflection();

                Console.WriteLine(sectionDivider);
                Console.WriteLine("Press any key to exit.");
                Console.Read();
            }
        }
    }
    // The example is expected to display the following output:
    // SERIALIZATION USING REFLECTION
    //
    // Serializing Sample Data Set...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // Result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.


###<a name="Scenario2"></a>示例 2：通过通用记录进行序列化

当数据无法使用具有数据协定的 .NET 类表示而导致不能使用反射时，可以在通用记录中显式指定 JSON 架构。此方法通常比使用反射要慢。在这种情况下，数据架构也可能是动态的，因为在编译之前它是未知的。以逗号分隔值 (CSV) 文件表示的数据（在运行时转换为 Avro 格式之前，其架构一直是未知的）是这种动态方案的一个示例。

此示例演示如何创建 [**AvroRecord**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.avrorecord.aspx) 并使用它显式指定 JSON 架构，如何为其填充数据，然后对其进行序列化和反序列化。然后，将结果与初始实例进行比较，以确认恢复的记录与原始记录相同。

此示例中的架构假定在读取器与写入器之间共享，因此无需采用 Avro 对象容器格式。有关在架构必须包含在已序列化的数据中时，如何使用通用记录和对象容器格式将数据序列化到内存缓冲区，以及对内存缓冲区中的数据进行反序列化的示例，请参阅<a href="#Scenario4">使用对象容器文件通过通用记录进行序列化</a>示例。


	namespace Microsoft.Hadoop.Avro.Sample
	{
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Runtime.Serialization;
    using Microsoft.Hadoop.Avro.Container;
    using Microsoft.Hadoop.Avro.Schema;
	using Microsoft.Hadoop.Avro;

    //This class contains all methods demonstrating
    //the usage of Microsoft Avro Library
    public class AvroSample
    {

        //Serialize and deserialize sample data set by using a generic record.
        //A generic record is a special class with the schema explicitly defined in JSON.
        //All serialized data should be mapped to the fields of the generic record,
        //which in turn will be then serialized.
        public void SerializeDeserializeObjectUsingGenericRecords()
        {
            Console.WriteLine("SERIALIZATION USING GENERIC RECORD\n");
            Console.WriteLine("Defining the Schema and creating Sample Data Set...");

            //Define the schema in JSON
            const string Schema = @"{
                                ""type"":""record"",
                                ""name"":""Microsoft.Hadoop.Avro.Specifications.SensorData"",
                                ""fields"":
                                    [
                                        {
                                            ""name"":""Location"",
                                            ""type"":
                                                {
                                                    ""type"":""record"",
                                                    ""name"":""Microsoft.Hadoop.Avro.Specifications.Location"",
                                                    ""fields"":
                                                        [
                                                            { ""name"":""Floor"", ""type"":""int"" },
                                                            { ""name"":""Room"", ""type"":""int"" }
                                                        ]
                                                }
                                        },
                                        { ""name"":""Value"", ""type"":""bytes"" }
                                    ]
                            }";

            //Create a generic serializer based on the schema
            var serializer = AvroSerializer.CreateGeneric(Schema);
            var rootSchema = serializer.WriterSchema as RecordSchema;

            //Create a memory stream buffer
            using (var stream = new MemoryStream())
            {
                //Create a generic record to represent the data
                dynamic location = new AvroRecord(rootSchema.GetField("Location").TypeSchema);
                location.Floor = 1;
                location.Room = 243;

                dynamic expected = new AvroRecord(serializer.WriterSchema);
                expected.Location = location;
                expected.Value = new byte[] { 1, 2, 3, 4, 5 };

                Console.WriteLine("Serializing Sample Data Set...");

                //Serialize the data
                serializer.Serialize(stream, expected);

                stream.Seek(0, SeekOrigin.Begin);

                Console.WriteLine("Deserializing Sample Data Set...");

                //Deserialize the data into a generic record
                dynamic actual = serializer.Deserialize(stream);

                Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

                //Finally, verify the results
                bool isEqual = expected.Location.Floor.Equals(actual.Location.Floor);
                isEqual = isEqual && expected.Location.Room.Equals(actual.Location.Room);
                isEqual = isEqual && ((byte[])expected.Value).SequenceEqual((byte[])actual.Value);
                Console.WriteLine("Result of Data Set Identity Comparison is {0}", isEqual);
            }
        }

        static void Main()
        {

            string sectionDivider = "---------------------------------------- ";

            //Create an instance of AvroSample class and invoke methods
            //illustrating different serializing approaches
            AvroSample Sample = new AvroSample();

            //Serialization to memory using generic record
            Sample.SerializeDeserializeObjectUsingGenericRecords();

            Console.WriteLine(sectionDivider);
            Console.WriteLine("Press any key to exit.");
            Console.Read();
        }
    }
	}
    // The example is expected to display the following output:
    // SERIALIZATION USING GENERIC RECORD
    //
    // Defining the Schema and creating Sample Data Set...
    // Serializing Sample Data Set...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // Result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.


###<a name="Scenario3"></a>示例 3：使用对象容器文件进行序列化与使用反射进行序列化

此示例与<a href="#Scenario1">第一个示例</a>中使用反射隐式指定架构的方案类似。除了本示例假设要将架构反序列化的读取器不知道架构以外。要序列化的 **SensorData** 对象及其隐式指定的架构存储在由 [**AvroContainer**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.container.avrocontainer.aspx) 类表示的 Avro 对象容器文件中。

在此示例中，数据使用 [**SequentialWriter<SensorData>**](http://msdn.microsoft.com/zh-cn/library/dn627340.aspx) 进行序列化，使用 [**SequentialReader<SensorData>**](http://msdn.microsoft.com/zh-cn/library/dn627340.aspx) 进行反序列化。然后，将结果与初始实例比较，以确保相同。

对象容器文件中的数据是通过 .NET Framework 4 中的默认 [**Deflate**][deflate-100] 压缩编解码器压缩的。请参阅本主题中的<a href="#Scenario5">第五个示例</a>，了解如何使用 .NET Framework 4.5 中提供的更新的 [**Deflate**][deflate-110] 压缩编解码器高级版。

    namespace Microsoft.Hadoop.Avro.Sample
    {
        using System;
        using System.Collections.Generic;
        using System.IO;
        using System.Linq;
        using System.Runtime.Serialization;
        using Microsoft.Hadoop.Avro.Container;
		using Microsoft.Hadoop.Avro;

        //Sample class used in serialization samples
        [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
        internal class SensorData
        {
            [DataMember(Name = "Location")]
            public Location Position { get; set; }

            [DataMember(Name = "Value")]
            public byte[] Value { get; set; }
        }

        //Sample struct used in serialization samples
        [DataContract]
        internal struct Location
        {
            [DataMember]
            public int Floor { get; set; }

            [DataMember]
            public int Room { get; set; }
        }

        //This class contains all methods demonstrating
        //the usage of Microsoft Avro Library
        public class AvroSample
        {

            //Serializes and deserializes the sample data set by using reflection and Avro object container files.
            //Serialized data is compressed with the Deflate codec.
            public void SerializeDeserializeUsingObjectContainersReflection()
            {

                Console.WriteLine("SERIALIZATION USING REFLECTION AND AVRO OBJECT CONTAINER FILES\n");

                //Path for Avro object container file
                string path = "AvroSampleReflectionDeflate.avro";

                //Create a data set by using sample class and struct
                var testData = new List<SensorData>
                        {
                            new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } },
                            new SensorData { Value = new byte[] { 6, 7, 8, 9 }, Position = new Location { Room = 244, Floor = 1 } }
                        };

                //Serializing and saving data to file.
                //Creating a memory stream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Serializing Sample Data Set...");

                    //Create a SequentialWriter instance for type SensorData, which can serialize a sequence of SensorData objects to stream.
                    //Data will be compressed using the Deflate codec.
                    using (var w = AvroContainer.CreateWriter<SensorData>(buffer, Codec.Deflate))
                    {
                        using (var writer = new SequentialWriter<SensorData>(w, 24))
                        {
                            // Serialize the data to stream by using the sequential writer
                            testData.ForEach(writer.Write);
                        }
                    }

                    //Save stream to file
                    Console.WriteLine("Saving serialized data to file...");
                    if (!WriteFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }
                }

                //Reading and deserializing data.
                //Creating a memory stream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Reading data from file...");

                    //Reading data from object container file
                    if (!ReadFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }

                    Console.WriteLine("Deserializing Sample Data Set...");

                    //Prepare the stream for deserializing the data
                    buffer.Seek(0, SeekOrigin.Begin);

                    //Create a SequentialReader instance for type SensorData, which will deserialize all serialized objects from the given stream.
                    //It allows iterating over the deserialized objects because it implements the IEnumerable<T> interface.
                    using (var reader = new SequentialReader<SensorData>(
                        AvroContainer.CreateReader<SensorData>(buffer, true)))
                    {
                        var results = reader.Objects;

                        //Finally, verify that deserialized data matches the original one
                        Console.WriteLine("Comparing Initial and Deserialized Data Sets...");
                        int count = 1;
                        var pairs = testData.Zip(results, (serialized, deserialized) => new { expected = serialized, actual = deserialized });
                        foreach (var pair in pairs)
                        {
                            bool isEqual = this.Equal(pair.expected, pair.actual);
                            Console.WriteLine("For Pair {0} result of Data Set Identity Comparison is {1}", count, isEqual);
                            count++;
                        }
                    }
                }

                //Delete the file
                RemoveFile(path);
            }

            //
            //Helper methods
            //

            //Comparing two SensorData objects
            private bool Equal(SensorData left, SensorData right)
            {
                return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }

            //Saving memory stream to a new file with the given path
            private bool WriteFile(MemoryStream InputStream, string path)
            {
                if (!File.Exists(path))
                {
                    try
                    {
                        using (FileStream fs = File.Create(path))
                        {
                            InputStream.Seek(0, SeekOrigin.Begin);
                            InputStream.CopyTo(fs);
                        }
                        return true;
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during creation and writing to the file "{0}"", path);
                        Console.WriteLine(e.Message);
                        return false;
                    }
                }
                else
                {
                    Console.WriteLine("Can not create file "{0}". File already exists", path);
                    return false;

                }
            }

            //Reading a file content by using the given path to a memory stream
            private bool ReadFile(MemoryStream OutputStream, string path)
            {
                try
                {
                    using (FileStream fs = File.Open(path, FileMode.Open))
                    {
                        fs.CopyTo(OutputStream);
                    }
                    return true;
                }
                catch (Exception e)
                {
                    Console.WriteLine("The following exception was thrown during reading from the file "{0}"", path);
                    Console.WriteLine(e.Message);
                    return false;
                }
            }

            //Deleting file by using given path
            private void RemoveFile(string path)
            {
                if (File.Exists(path))
                {
                    try
                    {
                        File.Delete(path);
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during deleting the file "{0}"", path);
                        Console.WriteLine(e.Message);
                    }
                }
                else
                {
                    Console.WriteLine("Can not delete file "{0}". File does not exist", path);
                }
            }

            static void Main()
            {

                string sectionDivider = "---------------------------------------- ";

                //Create an instance of AvroSample class and invoke methods
                //illustrating different serializing approaches
                AvroSample Sample = new AvroSample();

                //Serialization using reflection to Avro object container file
                Sample.SerializeDeserializeUsingObjectContainersReflection();

                Console.WriteLine(sectionDivider);
                Console.WriteLine("Press any key to exit.");
                Console.Read();
            }
        }
    }
    // The example is expected to display the following output:
    // SERIALIZATION USING REFLECTION AND AVRO OBJECT CONTAINER FILES
    //
    // Serializing Sample Data Set...
    // Saving serialized data to file...
    // Reading data from file...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // For Pair 1 result of Data Set Identity Comparison is True
    // For Pair 2 result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.


###<a name="Scenario4"></a>示例 4：使用对象容器文件进行序列化与使用通用记录进行序列化

此示例与<a href="#Scenario2">第二个示例</a>中使用 JSON 显式指定架构的方案类似。除了本示例假设要将架构反序列化的读取器不知道架构以外。

测试数据集将通过显式定义的 JSON 架构收集到 [**AvroRecord**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.avrorecord.aspx) 对象列表中，然后存储在由 [**AvroContainer**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.container.avrocontainer.aspx) 类表示的对象容器文件中。此容器文件将创建一个写入器，该写入器用于将未压缩的数据序列化到内存流，然后将该内存流保存到文件中。指定不要压缩此数据的是创建读取器时所用的 [**Codex.Null**](http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.container.codec.null.aspx) 参数。

然后，从文件中读取数据，并将数据反序列化为对象的集合。将此集合与 Avro 记录的初始列表进行比较，以确认它们相同。


    namespace Microsoft.Hadoop.Avro.Sample
    {
        using System;
        using System.Collections.Generic;
        using System.IO;
        using System.Linq;
        using System.Runtime.Serialization;
        using Microsoft.Hadoop.Avro.Container;
        using Microsoft.Hadoop.Avro.Schema;
		using Microsoft.Hadoop.Avro;

        //This class contains all methods demonstrating
        //the usage of Microsoft Avro Library
        public class AvroSample
        {

            //Serializes and deserializes a sample data set by using a generic record and Avro object container files.
            //Serialized data is not compressed.
            public void SerializeDeserializeUsingObjectContainersGenericRecord()
            {
                Console.WriteLine("SERIALIZATION USING GENERIC RECORD AND AVRO OBJECT CONTAINER FILES\n");

                //Path for Avro object container file
                string path = "AvroSampleGenericRecordNullCodec.avro";

                Console.WriteLine("Defining the Schema and creating Sample Data Set...");

                //Define the schema in JSON
                const string Schema = @"{
                                ""type"":""record"",
                                ""name"":""Microsoft.Hadoop.Avro.Specifications.SensorData"",
                                ""fields"":
                                    [
                                        {
                                            ""name"":""Location"",
                                            ""type"":
                                                {
                                                    ""type"":""record"",
                                                    ""name"":""Microsoft.Hadoop.Avro.Specifications.Location"",
                                                    ""fields"":
                                                        [
                                                            { ""name"":""Floor"", ""type"":""int"" },
                                                            { ""name"":""Room"", ""type"":""int"" }
                                                        ]
                                                }
                                        },
                                        { ""name"":""Value"", ""type"":""bytes"" }
                                    ]
                            }";

                //Create a generic serializer based on the schema
                var serializer = AvroSerializer.CreateGeneric(Schema);
                var rootSchema = serializer.WriterSchema as RecordSchema;

                //Create a generic record to represent the data
                var testData = new List<AvroRecord>();

                dynamic expected1 = new AvroRecord(rootSchema);
                dynamic location1 = new AvroRecord(rootSchema.GetField("Location").TypeSchema);
                location1.Floor = 1;
                location1.Room = 243;
                expected1.Location = location1;
                expected1.Value = new byte[] { 1, 2, 3, 4, 5 };
                testData.Add(expected1);

                dynamic expected2 = new AvroRecord(rootSchema);
                dynamic location2 = new AvroRecord(rootSchema.GetField("Location").TypeSchema);
                location2.Floor = 1;
                location2.Room = 244;
                expected2.Location = location2;
                expected2.Value = new byte[] { 6, 7, 8, 9 };
                testData.Add(expected2);

                //Serializing and saving data to file.
                //Create a MemoryStream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Serializing Sample Data Set...");

                    //Create a SequentialWriter instance for type SensorData, which can serialize a sequence of SensorData objects to stream.
                    //Data will not be compressed (Null compression codec).
                    using (var writer = AvroContainer.CreateGenericWriter(Schema, buffer, Codec.Null))
                    {
                        using (var streamWriter = new SequentialWriter<object>(writer, 24))
                        {
                            // Serialize the data to stream by using the sequential writer
                            testData.ForEach(streamWriter.Write);
                        }
                    }

                    Console.WriteLine("Saving serialized data to file...");

                    //Save stream to file
                    if (!WriteFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }
                }

                //Reading and deserializing the data.
                //Create a memory stream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Reading data from file...");

                    //Reading data from object container file
                    if (!ReadFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }

                    Console.WriteLine("Deserializing Sample Data Set...");

                    //Prepare the stream for deserializing the data
                    buffer.Seek(0, SeekOrigin.Begin);

                    //Create a SequentialReader instance for type SensorData, which will deserialize all serialized objects from the given stream.
                    //It allows iterating over the deserialized objects because it implements the IEnumerable<T> interface.
                    using (var reader = AvroContainer.CreateGenericReader(buffer))
                    {
                        using (var streamReader = new SequentialReader<object>(reader))
                        {
                            var results = streamReader.Objects;

                            Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

                            //Finally, verify the results
                            var pairs = testData.Zip(results, (serialized, deserialized) => new { expected = (dynamic)serialized, actual = (dynamic)deserialized });
                            int count = 1;
                            foreach (var pair in pairs)
                            {
                                bool isEqual = pair.expected.Location.Floor.Equals(pair.actual.Location.Floor);
                                isEqual = isEqual && pair.expected.Location.Room.Equals(pair.actual.Location.Room);
                                isEqual = isEqual && ((byte[])pair.expected.Value).SequenceEqual((byte[])pair.actual.Value);
                                Console.WriteLine("For Pair {0} result of Data Set Identity Comparison is {1}", count, isEqual.ToString());
                                count++;
                            }
                        }
                    }
                }

                //Delete the file
                RemoveFile(path);
            }

            //
            //Helper methods
            //

            //Saving memory stream to a new file with the given path
            private bool WriteFile(MemoryStream InputStream, string path)
            {
                if (!File.Exists(path))
                {
                    try
                    {
                        using (FileStream fs = File.Create(path))
                        {
                            InputStream.Seek(0, SeekOrigin.Begin);
                            InputStream.CopyTo(fs);
                        }
                        return true;
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during creation and writing to the file "{0}"", path);
                        Console.WriteLine(e.Message);
                        return false;
                    }
                }
                else
                {
                    Console.WriteLine("Can not create file "{0}". File already exists", path);
                    return false;

                }
            }

            //Reading a file content by using the given path to a memory stream
            private bool ReadFile(MemoryStream OutputStream, string path)
            {
                try
                {
                    using (FileStream fs = File.Open(path, FileMode.Open))
                    {
                        fs.CopyTo(OutputStream);
                    }
                    return true;
                }
                catch (Exception e)
                {
                    Console.WriteLine("The following exception was thrown during reading from the file "{0}"", path);
                    Console.WriteLine(e.Message);
                    return false;
                }
            }

            //Deleting file by using the given path
            private void RemoveFile(string path)
            {
                if (File.Exists(path))
                {
                    try
                    {
                        File.Delete(path);
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during deleting the file "{0}"", path);
                        Console.WriteLine(e.Message);
                    }
                }
                else
                {
                    Console.WriteLine("Can not delete file "{0}". File does not exist", path);
                }
            }

            static void Main()
            {

                string sectionDivider = "---------------------------------------- ";

                //Create an instance of the AvroSample class and invoke methods
                //illustrating different serializing approaches
                AvroSample Sample = new AvroSample();

                //Serialization using generic record to Avro object container file
                Sample.SerializeDeserializeUsingObjectContainersGenericRecord();

                Console.WriteLine(sectionDivider);
                Console.WriteLine("Press any key to exit.");
                Console.Read();
            }
        }
    }
    // The example is expected to display the following output:
    // SERIALIZATION USING GENERIC RECORD AND AVRO OBJECT CONTAINER FILES
    //
    // Defining the Schema and creating Sample Data Set...
    // Serializing Sample Data Set...
    // Saving serialized data to file...
    // Reading data from file...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // For Pair 1 result of Data Set Identity Comparison is True
    // For Pair 2 result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.




###<a name="Scenario5"></a>示例 5：使用对象容器文件通过自定义压缩编解码器进行序列化

第五个示例演示如何将自定义压缩编解码器用于 Avro 对象容器文件。包含此示例代码的样例可以从 [Azure 代码示例](https://github.com/Azure-Samples)站点下载。

[Avro 规范](http://avro.apache.org/docs/current/spec.html#Required+Codecs)允许使用可选的压缩编解码器（除了 **Null** 和 **Deflate** 默认压缩编解码器外）。此示例未完全实现类似 Snappy（在 [Avro 规范](http://avro.apache.org/docs/current/spec.html#snappy)中作为支持的可选编解码器提及）的新编解码器。它演示如何使用 [**Deflate**][deflate-110] 编解码器的 .NET Framework 4.5 实现，后者基于 [zlib](http://zlib.net/) 压缩库提供比默认的 .NET Framework 4.0 版本更好的压缩算法。


    //
    // This code needs to be compiled with the parameter Target Framework set as ".NET Framework 4.5"
    // to ensure the desired implementation of the Deflate compression algorithm is used.
    // Ensure your C# project is set up accordingly.
    //

    namespace Microsoft.Hadoop.Avro.Sample
    {
        using System;
        using System.Collections.Generic;
        using System.Diagnostics;
        using System.IO;
        using System.IO.Compression;
        using System.Linq;
        using System.Runtime.Serialization;
        using Microsoft.Hadoop.Avro.Container;
		using Microsoft.Hadoop.Avro;

        #region Defining objects for serialization
        //Sample class used in serialization samples
        [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
        internal class SensorData
        {
            [DataMember(Name = "Location")]
            public Location Position { get; set; }

            [DataMember(Name = "Value")]
            public byte[] Value { get; set; }
        }

        //Sample struct used in serialization samples
        [DataContract]
        internal struct Location
        {
            [DataMember]
            public int Floor { get; set; }

            [DataMember]
            public int Room { get; set; }
        }
        #endregion

        #region Defining custom codec based on .NET Framework V.4.5 Deflate
        //Avro.NET codec class contains two methods,
        //GetCompressedStreamOver(Stream uncompressed) and GetDecompressedStreamOver(Stream compressed),
        //which are the key ones for data compression.
        //To enable a custom codec, one needs to implement these methods for the required codec.

        #region Defining Compression and Decompression Streams
        //DeflateStream (class from System.IO.Compression namespace that implements Deflate algorithm)
        //cannot be directly used for Avro because it does not support vital operations like Seek.
        //Thus one needs to implement two classes inherited from stream
        //(one for compressed and one for decompressed stream)
        //that use Deflate compression and implement all required features.
        internal sealed class CompressionStreamDeflate45 : Stream
        {
            private readonly Stream buffer;
            private DeflateStream compressionStream;

            public CompressionStreamDeflate45(Stream buffer)
            {
                Debug.Assert(buffer != null, "Buffer is not allowed to be null.");

                this.compressionStream = new DeflateStream(buffer, CompressionLevel.Fastest, true);
                this.buffer = buffer;
            }

            public override bool CanRead
            {
                get { return this.buffer.CanRead; }
            }

            public override bool CanSeek
            {
                get { return true; }
            }

            public override bool CanWrite
            {
                get { return this.buffer.CanWrite; }
            }

            public override void Flush()
            {
                this.compressionStream.Close();
            }

            public override long Length
            {
                get { return this.buffer.Length; }
            }

            public override long Position
            {
                get
                {
                    return this.buffer.Position;
                }

                set
                {
                    this.buffer.Position = value;
                }
            }

            public override int Read(byte[] buffer, int offset, int count)
            {
                return this.buffer.Read(buffer, offset, count);
            }

            public override long Seek(long offset, SeekOrigin origin)
            {
                return this.buffer.Seek(offset, origin);
            }

            public override void SetLength(long value)
            {
                throw new NotSupportedException();
            }

            public override void Write(byte[] buffer, int offset, int count)
            {
                this.compressionStream.Write(buffer, offset, count);
            }

            protected override void Dispose(bool disposed)
            {
                base.Dispose(disposed);

                if (disposed)
                {
                    this.compressionStream.Dispose();
                    this.compressionStream = null;
                }
            }
        }

        internal sealed class DecompressionStreamDeflate45 : Stream
        {
            private readonly DeflateStream decompressed;

            public DecompressionStreamDeflate45(Stream compressed)
            {
                this.decompressed = new DeflateStream(compressed, CompressionMode.Decompress, true);
            }

            public override bool CanRead
            {
                get { return true; }
            }

            public override bool CanSeek
            {
                get { return true; }
            }

            public override bool CanWrite
            {
                get { return false; }
            }

            public override void Flush()
            {
                this.decompressed.Close();
            }

            public override long Length
            {
                get { return this.decompressed.Length; }
            }

            public override long Position
            {
                get
                {
                    return this.decompressed.Position;
                }

                set
                {
                    throw new NotSupportedException();
                }
            }

            public override int Read(byte[] buffer, int offset, int count)
            {
                return this.decompressed.Read(buffer, offset, count);
            }

            public override long Seek(long offset, SeekOrigin origin)
            {
                throw new NotSupportedException();
            }

            public override void SetLength(long value)
            {
                throw new NotSupportedException();
            }

            public override void Write(byte[] buffer, int offset, int count)
            {
                throw new NotSupportedException();
            }

            protected override void Dispose(bool disposing)
            {
                base.Dispose(disposing);

                if (disposing)
                {
                    this.decompressed.Dispose();
                }
            }
        }
        #endregion

        #region Define Codec
        //Define the actual codec class containing the required methods for manipulating streams:
        //GetCompressedStreamOver(Stream uncompressed) and GetDecompressedStreamOver(Stream compressed).
        //Codec class uses classes for compressed and decompressed streams defined above.
        internal sealed class DeflateCodec45 : Codec
        {

            //We merely use different IMPLEMENTATIONS of Deflate, so CodecName remains "deflate"
            public static readonly string CodecName = "deflate";

            public DeflateCodec45()
                : base(CodecName)
            {
            }

            public override Stream GetCompressedStreamOver(Stream decompressed)
            {
                if (decompressed == null)
                {
                    throw new ArgumentNullException("decompressed");
                }

                return new CompressionStreamDeflate45(decompressed);
            }

            public override Stream GetDecompressedStreamOver(Stream compressed)
            {
                if (compressed == null)
                {
                    throw new ArgumentNullException("compressed");
                }

                return new DecompressionStreamDeflate45(compressed);
            }
        }
        #endregion

        #region Define modified Codec Factory
        //Define modified codec factory to be used in the reader.
        //It will catch the attempt to use "Deflate" and provide  a custom codec.
        //For all other cases, it will rely on the base class (CodecFactory).
        internal sealed class CodecFactoryDeflate45 : CodecFactory
        {

            public override Codec Create(string codecName)
            {
                if (codecName == DeflateCodec45.CodecName)
                    return new DeflateCodec45();
                else
                    return base.Create(codecName);
            }
        }
        #endregion

        #endregion

        #region Sample Class with demonstration methods
        //This class contains methods demonstrating
        //the usage of Microsoft Avro Library
        public class AvroSample
        {

            //Serializes and deserializes sample data set by using reflection and Avro object container files.
            //Serialized data is compressed with the custom compression codec (Deflate of .NET Framework 4.5).
            //
            //This sample uses memory stream for all operations related to serialization, deserialization and
            //object container manipulation, though file stream could be easily used.
            public void SerializeDeserializeUsingObjectContainersReflectionCustomCodec()
            {

                Console.WriteLine("SERIALIZATION USING REFLECTION, AVRO OBJECT CONTAINER FILES AND CUSTOM CODEC\n");

                //Path for Avro object container file
                string path = "AvroSampleReflectionDeflate45.avro";

                //Create a data set by using sample class and struct
                var testData = new List<SensorData>
                        {
                            new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } },
                            new SensorData { Value = new byte[] { 6, 7, 8, 9 }, Position = new Location { Room = 244, Floor = 1 } }
                        };

                //Serializing and saving data to file.
                //Creating a memory stream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Serializing Sample Data Set...");

                    //Create a SequentialWriter instance for type SensorData, which can serialize a sequence of SensorData objects to stream.
                    //Here the custom codec is introduced. For convenience, the next commented code line shows how to use built-in Deflate.
                    //Note that because the sample deals with different IMPLEMENTATIONS of Deflate, built-in and custom codecs are interchangeable
                    //in read-write operations.
                    //using (var w = AvroContainer.CreateWriter<SensorData>(buffer, Codec.Deflate))
                    using (var w = AvroContainer.CreateWriter<SensorData>(buffer, new DeflateCodec45()))
                    {
                        using (var writer = new SequentialWriter<SensorData>(w, 24))
                        {
                            // Serialize the data to stream using the sequential writer
                            testData.ForEach(writer.Write);
                        }
                    }

                    //Save stream to file
                    Console.WriteLine("Saving serialized data to file...");
                    if (!WriteFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }
                }

                //Reading and deserializing data.
                //Creating a memory stream buffer.
                using (var buffer = new MemoryStream())
                {
                    Console.WriteLine("Reading data from file...");

                    //Reading data from object container file
                    if (!ReadFile(buffer, path))
                    {
                        Console.WriteLine("Error during file operation. Quitting method");
                        return;
                    }

                    Console.WriteLine("Deserializing Sample Data Set...");

                    //Prepare the stream for deserializing the data
                    buffer.Seek(0, SeekOrigin.Begin);

                    //Because of SequentialReader<T> constructor signature, an AvroSerializerSettings instance is required
                    //when codec factory is explicitly specified.
                    //You may comment the line below if you want to use built-in Deflate (see next comment).
                    AvroSerializerSettings settings = new AvroSerializerSettings();

                    //Create a SequentialReader instance for type SensorData, which will deserialize all serialized objects from the given stream.
                    //It allows iterating over the deserialized objects because it implements the IEnumerable<T> interface.
                    //Here the custom codec factory is introduced.
                    //For convenience, the next commented code line shows how to use built-in Deflate
                    //(no explicit Codec Factory parameter is required in this case).
                    //Note that because the sample deals with different IMPLEMENTATIONS of Deflate, built-in and custom codecs are interchangeable
                    //in read-write operations.
                    //using (var reader = new SequentialReader<SensorData>(AvroContainer.CreateReader<SensorData>(buffer, true)))
                    using (var reader = new SequentialReader<SensorData>(
                        AvroContainer.CreateReader<SensorData>(buffer, true, settings, new CodecFactoryDeflate45())))
                    {
                        var results = reader.Objects;

                        //Finally, verify that deserialized data matches the original one
                        Console.WriteLine("Comparing Initial and Deserialized Data Sets...");
                        bool isEqual;
                        int count = 1;
                        var pairs = testData.Zip(results, (serialized, deserialized) => new { expected = serialized, actual = deserialized });
                        foreach (var pair in pairs)
                        {
                            isEqual = this.Equal(pair.expected, pair.actual);
                            Console.WriteLine("For Pair {0} result of Data Set Identity Comparison is {1}", count, isEqual.ToString());
                            count++;
                        }
                    }
                }

                //Delete the file
                RemoveFile(path);
            }
        #endregion

            #region Helper Methods

            //Comparing two SensorData objects
            private bool Equal(SensorData left, SensorData right)
            {
                return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }

            //Saving memory stream to a new file with the given path
            private bool WriteFile(MemoryStream InputStream, string path)
            {
                if (!File.Exists(path))
                {
                    try
                    {
                        using (FileStream fs = File.Create(path))
                        {
                            InputStream.Seek(0, SeekOrigin.Begin);
                            InputStream.CopyTo(fs);
                        }
                        return true;
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during creation and writing to the file "{0}"", path);
                        Console.WriteLine(e.Message);
                        return false;
                    }
                }
                else
                {
                    Console.WriteLine("Can not create file "{0}". File already exists", path);
                    return false;

                }
            }

            //Reading file content by using the given path to a memory stream
            private bool ReadFile(MemoryStream OutputStream, string path)
            {
                try
                {
                    using (FileStream fs = File.Open(path, FileMode.Open))
                    {
                        fs.CopyTo(OutputStream);
                    }
                    return true;
                }
                catch (Exception e)
                {
                    Console.WriteLine("The following exception was thrown during reading from the file "{0}"", path);
                    Console.WriteLine(e.Message);
                    return false;
                }
            }

            //Deleting file by using given path
            private void RemoveFile(string path)
            {
                if (File.Exists(path))
                {
                    try
                    {
                        File.Delete(path);
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("The following exception was thrown during deleting the file "{0}"", path);
                        Console.WriteLine(e.Message);
                    }
                }
                else
                {
                    Console.WriteLine("Can not delete file "{0}". File does not exist", path);
                }
            }
            #endregion

            static void Main()
            {

                string sectionDivider = "---------------------------------------- ";

                //Create an instance of AvroSample Class and invoke methods
                //illustrating different serializing approaches
                AvroSample Sample = new AvroSample();

                //Serialization using reflection to Avro object container file using custom codec
                Sample.SerializeDeserializeUsingObjectContainersReflectionCustomCodec();

                Console.WriteLine(sectionDivider);
                Console.WriteLine("Press any key to exit.");
                Console.Read();
            }
        }
    }
    // The example is expected to display the following output:
    // SERIALIZATION USING REFLECTION, AVRO OBJECT CONTAINER FILES AND CUSTOM CODEC
    //
    // Serializing Sample Data Set...
    // Saving serialized data to file...
    // Reading data from file...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // For Pair 1 result of Data Set Identity Comparison is True
    //For Pair 2 result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.

###<a name="Scenario6"></a>示例 6：使用 Avro 上载 Azure HDInsight 服务的数据

第六个示例演示与 Azure HDInsight 服务交互相关的一些编程技巧。包含此示例代码的样例可以从 [Azure 代码示例](https://github.com/Azure-Samples)站点下载。

该示例将执行以下操作：

* 连接到现有的 HDInsight 服务群集。
* 序列化多个 CSV 文件并将结果上载到 Azure Blob 存储。（CSV 文件随着示例一起分发，而且代表 [Infochimps](http://www.infochimps.com/) 在 1970 年到 2010 年期间提取自 AMEX 股票的历史记录数据。该示例将读取 CSV 文件数据、将记录转换为 **Stock** 类的实例，然后使用反射序列化这些实例。Stock 类型定义是使用 Microsoft Avro Library 代码生成实用工具从 JSON 架构创建的。
* 在 Hive 中创建名为 **Stocks** 的新外部表，并将它链接到前一个步骤中上载的数据。
* 使用 Hive 对 **Stocks** 表执行查询。

此外，该示例将在执行主要操作之前和之后执行清理过程。在清理期间，将删除所有相关的 Azure Blob 数据和文件夹，并删除 Hive 表。你也可以从示例命令行调用清理过程。

该示例要求满足以下先决条件：

* 有效的 Azure 订阅及其订阅 ID。
* 包含相应私钥的订阅管理证书。该证书应安装在用于运行示例的计算机上的当前用户私用存储中。
* 活动的 HDInsight 群集。
* 在先前的必要条件中链接到 HDInsight 群集的 Azure 存储帐户，以及相应的主要或辅助访问密钥。

运行示例之前，必要条件中的所有信息均应输入到示例配置文件中。要运行此操作有两个可行的方式：

* 编辑示例根目录中的 app.config 文件，然后生成示例，或
* 先生成示例，然后在生成目录中编辑 AvroHDISample.exe.config

在这两个情况下，所有编辑均应该在 **<appSettings>** 设置节中完成。请遵循文件中的注释。
执行以下命令从命令行运行该示例（其中，包含该示例的 .zip 文件假设已解压缩到 C:\\AvroHDISample；如果不是，请使用相关的文件路径）：

    AvroHDISample run C:\AvroHDISample\Data

若要清理群集，请运行以下命令：

    AvroHDISample clean

[deflate-100]: http://msdn.microsoft.com/zh-cn/library/system.io.compression.deflatestream(v=vs.100).aspx
[deflate-110]: http://msdn.microsoft.com/zh-cn/library/system.io.compression.deflatestream(v=vs.110).aspx

<!---HONumber=Mooncake_0307_2016-->