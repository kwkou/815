<properties linkid="hdinsight-dotnet-avro-serialization" urlDisplayName="HDInsight Microsoft Avro Library Serialization" pageTitle="Serialize data with Microsoft Avro Library | Azure" metaKeywords="" description="Learn how Azure HDInsight uses Avro to serialize big data." metaCanonical="" services="hdinsight" documentationCenter="" title="Serialize data with Microsoft Avro Library " authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# 使用 Microsoft Avro Library 序列化数据

## 概述

本主题演示如何使用 Microsoft Avro Library 将对象及其他数据结构序列化为流，以便将它们持久保存到内存、数据库或文件中，同时还演示如何对这些流进行反序列化以恢复原始对象。

### Apache Avro

Microsoft Avro Library 针对 Microsoft.NET 环境实现了 Apache Avro 数据序列化系统。Apache Avro 为序列化提供了一种紧凑的二进制数据交换格式。它使用 [JSON][] 定义与语言无关的架构，以支持语言互操作性。以一种语言序列化的数据可以用另一种语言读取。目前支持 C、C++、C\#、Java、PHP、Python 和 Ruby。有关格式的详细信息可以在 [Apache Avro 规范][]中找到。请注意，Microsoft Avro Library 的当前版本不支持此规范的远程过程调用 (RPC) 部分。

Avro 系统中的对象的序列化表示形式由两部分组成：架构和实际值。Avro 架构使用 JSON 描述已序列化数据的与语言无关的数据模型。它与数据的二进制表示形式并排显示。将架构与二进制表示形式分离，使写入每个对象时没有针对值的开销，从而实现快速序列化和较小的表示形式。

### Hadoop 应用场景

Apache Avro 序列化格式广泛应用于 Azure HDInsight 及其他 Apache Hadoop 环境中。Avro 提供了简便的方法来表示 Hadoop MapReduce 作业内的复杂数据结构。Avro 文件格式已设计为支持分布式 MapReduce 编程模型。实现分布的关键是文件必须是“可拆分的”，也就是说，用户可以在文件中随机设置一个点，然后即可从某一特定块开始读取。

### Microsoft Avro Library 中的序列化

Microsoft Avro Library 支持通过两种方式序列化对象：

-   **反射**：自动从要序列化的 .NET 类型的数据协定特性生成这些类型的 JSON 架构。
-   **通用记录**：当没有 .NET 类型可以用来描述要序列化的数据的架构时，系统会在以 [**AvroRecord**][] 类表示的记录中显式指定 JSON 架构。

当流的写入器和读取器都知道数据架构时，可以发送没有架构的数据。但如果不是这种情况，则必须使用 Avro 容器文件共享架构。可以指定其他参数，例如用于数据压缩的编解码器。这些情况将在下面的代码示例中进一步详述和说明。

### Microsoft Avro Library 必备组件

-   [Microsoft .NET Framework v4.0][]
-   [Newtonsoft Json.NET][]（5.0.5 或更高版本）

请注意，在安装 Microsoft Avro Library 时将自动下载 Newtonsoft.Json.dll 依赖项，其过程在下一节中提供。

### Microsoft Avro Library 安装

Microsoft .NET Library for Avro 以 NuGet 包的形式分发，可以使用以下过程从 Visual Studio 安装：

-   选择“项目” 选项卡 -\>“管理 NuGet 包...” 
-   在“联机搜索” 框中，搜索“Microsoft.Hadoop.Avro”。
-   单击 **Microsoft .NET Library for Avro** 旁边的“安装” 按钮。

请注意，Newtonsoft.Json.dll (\>= .5.0.5) 依赖项也将随 Microsoft Avro Library 一起自动下载。

## 示例指南

本主题中提供的五个示例每个都说明了 Microsoft Avro Library 所支持的不同方案。

前两个示例显示如何使用反射和通用记录将数据序列化到内存流缓冲区，以及如何进行反序列化。这两个示例中的架构假定在带外读取器和写入器之间共享，这样架构就不需要在 Avro 容器文件中与数据一起序列化。

第三个和第四个示例显示如何使用 Avro 对象容器文件，通过反射和通用记录将数据序列化到内存流缓冲区，以及如何进行反序列化。当数据存储在 Avro 容器文件中时，其架构始终随之一起存储，因为必须共享架构才能进行反序列化。

包含前四个示例的样例可以从 [Azure 代码示例][]站点下载。

第五个也是最后一个示例演示如何将自定义压缩编解码器用于对象容器文件。包含此示例代码的样例可以从 [Azure 代码示例][1]站点下载。

Microsoft Avro Library 设计为可处理任何流。在这些示例中，为保持简单性和一致性，是使用内存流（而不是文件流或数据库）来操作数据的。在生产环境中所采取的方法将取决于实际的方案要求、数据源和卷、性能限制及其他因素。

-   [**通过反射进行序列化**][]：自动从数据协定特性生成要序列化的类型的 JSON 架构。
-   [**通过通用记录进行序列化**][]：当没有可用于反射的 .NET 类型时，在记录中显式指定 JSON 架构。
-   [**使用对象容器文件通过反射进行序列化**][]：JSON 架构隐式地与数据一起序列化，并使用 Avro 容器文件进行共享。
-   [**使用对象容器文件通过通用记录进行序列化**][]：JSON 架构显式地与数据一起序列化，并使用 Avro 容器文件进行共享。
-   [**使用对象容器文件通过自定义压缩编解码器进行序列化**][]：JSON 架构与数据一起序列化，并使用 Avro 容器文件（包含 deflate 数据压缩编解码器的自定义 .NET 实现）进行共享。

## 通过反射进行序列化

Microsoft Avro Library 可以使用反射从要序列化的 C\# 对象的数据协定特性自动生成类型的 JSON 架构。Microsoft Avro Library 将创建一个 [**IAvroSeralizer**][] 以标识要序列化的字段。

在此示例中，将对象（具有成员 **Location** 结构的 **SensorData** 类）序列化到内存流，继而又将此流反序列化。然后，将结果与初始实例进行比较，以确认恢复的 **SensorData** 对象与原始对象相同。

此示例中的架构假定在读取器与写入器之间共享，因此无需采用 Avro 对象容器格式。有关在架构必须与数据一起序列化时，如何使用反射和对象容器格式将数据序列化到内存缓冲区，以及如何对内存缓冲区中的数据进行反序列化的示例，请参阅[使用对象容器文件通过反射进行序列化][**使用对象容器文件通过反射进行序列化**]。

    namespace Microsoft.Hadoop.Avro.Sample
    {
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Runtime.Serialization;
    using Microsoft.Hadoop.Avro.Container;

    //序列化示例中使用的示例类
    [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
    internal class SensorData
        {
    [DataMember(Name = "Location")]
    public Location Position { get; set; }

    [DataMember(Name = "Value")]
    public byte[] Value { get; set; }
        }

    //序列化示例中使用的示例结构
    [DataContract]
    internal struct Location
        {
    [DataMember]
    public int Floor { get; set; }

    [DataMember]
    public int Room { get; set; }
        }

    //此类包含所有演示
    //Microsoft Avro Library 用法的方法
    public class AvroSample
        {

    //使用反射序列化和反序列化表示为对象的示例数据集
    //无需显式架构定义 - 自动生成所序列化对象的架构
    public void SerializeDeserializeObjectUsingReflection()
            {

    Console.WriteLine("SERIALIZATION USING REFLECTION\n");
    Console.WriteLine("Serializing Sample Data Set...");

    //创建一个新的 AvroSerializer 实例，并指定自定义序列化策略 AvroDataContractResolver，
    //仅序列化使用 DataContract/DateMember 特性化的属性
    var avroSerializer = AvroSerializer.Create<SensorData>();

    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    //使用示例类和结构创建数据集 
    var expected = new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } };

    //将数据序列化到指定的流
    avroSerializer.Serialize(buffer, expected);


    Console.WriteLine("Deserializing Sample Data Set...");

    //准备用于反序列化数据的流
    buffer.Seek(0, SeekOrigin.Begin);

    //反序列化流中的数据，并将其转换为用于序列化的同一类型
    var actual = avroSerializer.Deserialize(buffer);

    Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

    //最后，验证反序列化后的数据是否与原始数据匹配
    bool isEqual = this.Equal(expected, actual);

    Console.WriteLine("Result of Data Set Identity Comparison is {0}", isEqual);

                }
            }

            //
    //帮助器方法
            //

    //比较两个 SensorData 对象
    private bool Equal(SensorData left, SensorData right)
            {
    return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }



    static void Main()
            {

    string sectionDivider = "---------------------------------------- ";

    //创建 AvroSample 类的一个实例，并调用
    //说明不同序列化方案的方法
    AvroSample Sample = new AvroSample();

    //使用反射序列化到内存
    Sample.SerializeDeserializeObjectUsingReflection();

    Console.WriteLine(sectionDivider);
    Console.WriteLine("Press any key to exit.");
    Console.Read();
            }
        }
    }
    // 此示例应显示以下输出： 
    // SERIALIZATION USING REFLECTION
    //
    // Serializing Sample Data Set...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // Result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.

## 通过通用记录进行序列化

当数据无法使用具有数据协定的 .NET 类表示而导致不能使用反射时，可以在通用记录中显式指定 JSON 架构。此方法通常比使用反射和用于特定 C\# 类的序列化程序速度慢。在这种情况下，数据架构也可能是动态的，因为在编译之前它是未知的。以逗号分隔值 (CSV) 文件表示的数据（在运行时转换为 Avro 格式之前，其架构一直是未知的）是这种动态方案的一个示例。

此示例演示如何创建 [**AvroRecord**][2] 并使用它显式指定 JSON 架构，如何为其填充数据，然后对其进行序列化和反序列化。然后，将结果与初始实例进行比较，以确认恢复的记录与原始记录相同。

此示例中的架构假定在读取器与写入器之间共享，因此无需采用 Avro 对象容器格式。有关在架构必须包含在已序列化的数据中时，如何使用通用记录和对象容器格式将数据序列化到内存缓冲区，以及对内存缓冲区中的数据进行反序列化的示例，请参阅[使用对象容器文件通过通用记录进行序列化][**使用对象容器文件通过通用记录进行序列化**]。

    namespace Microsoft.Hadoop.Avro.Sample
    {
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Runtime.Serialization;
    using Microsoft.Hadoop.Avro.Container;
    using Microsoft.Hadoop.Avro.Schema;

    //此类包含所有演示
    //Microsoft Avro Library 用法的方法
    public class AvroSample
    {

    //使用通用记录序列化和反序列化示例数据集。
    //通用记录是一种使用 JSON 显式定义架构的特殊类。
    //所有已序列化的数据应映射到通用记录的相应字段，
    //然后，反过来对其进行反序列化。
    public void SerializeDeserializeObjectUsingGenericRecords()
        {
    Console.WriteLine("SERIALIZATION USING GENERIC RECORD\n");
    Console.WriteLine("Defining the Schema and creating Sample Data Set...");

    //使用 JSON 定义架构
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

    //根据架构创建通用序列化程序
    var serializer = AvroSerializer.CreateGeneric(Schema);
    var rootSchema = serializer.WriterSchema as RecordSchema;

    //创建一个内存流缓冲区
    using (var stream = new MemoryStream())
            {
    //创建用于表示数据的通用记录
    dynamic location = new AvroRecord(rootSchema.GetField("Location").TypeSchema);
    location.Floor = 1;
    location.Room = 243;

    dynamic expected = new AvroRecord(serializer.WriterSchema);
    expected.Location = location;
    expected.Value = new byte[] { 1, 2, 3, 4, 5 };

    Console.WriteLine("Serializing Sample Data Set...");

    //对数据进行序列化
    serializer.Serialize(stream, expected);

    stream.Seek(0, SeekOrigin.Begin);

    Console.WriteLine("Deserializing Sample Data Set...");

    //将数据反序列化到通用记录
    dynamic actual = serializer.Deserialize(stream);

    Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

    //最后，验证结果
    bool isEqual = expected.Location.Floor.Equals(actual.Location.Floor);
    isEqual = isEqual && expected.Location.Room.Equals(actual.Location.Room);
    isEqual = isEqual && ((byte[])expected.Value).SequenceEqual((byte[])actual.Value);
    Console.WriteLine("Result of Data Set Identity Comparison is {0}", isEqual);
            }
        }

    static void Main()
        {

    string sectionDivider = "---------------------------------------- ";

    //创建 AvroSample 类的一个实例，并调用
    //说明不同序列化方案的方法
    AvroSample Sample = new AvroSample();

    //使用通用记录序列化到内存
    Sample.SerializeDeserializeObjectUsingGenericRecords();

    Console.WriteLine(sectionDivider);
    Console.WriteLine("Press any key to exit.");
    Console.Read();
        }
    }
    }
    // 此示例应显示以下输出： 
    // SERIALIZATION USING GENERIC RECORD
    //
    // Defining the Schema and creating Sample Data Set...
    // Serializing Sample Data Set...
    // Deserializing Sample Data Set...
    // Comparing Initial and Deserialized Data Sets...
    // Result of Data Set Identity Comparison is True
    // ----------------------------------------
    // Press any key to exit.

## 使用对象容器文件进行序列化与使用反射进行序列化

此示例类似于[第一个示例][**通过反射进行序列化**]中的方案，都是使用反射隐式地指定架构，不同的是，此处未假定对架构进行反序列化的读取器知道架构。要序列化的 **SensorData** 对象及其隐式指定的架构存储在由 [**AvroContainer**][] 类表示的对象容器文件中。

在此示例中，数据使用 [**SequentialWriter**][] 进行序列化，使用 [**SequentialReader**][**SequentialWriter**] 进行反序列化。然后，将结果与初始实例比较，以确保相同。

对象容器文件中的数据是使用 .NET Framework 4.0 中的默认 [**Deflate**][] 压缩编解码器压缩的。请参阅本主题中的[上一个示例][**使用对象容器文件通过自定义压缩编解码器进行序列化**]，了解如何使用 .NET Framework 4.5 中提供的更新的 [**Deflate**][3] 压缩编解码器高级版。

    namespace Microsoft.Hadoop.Avro.Sample
    {
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Runtime.Serialization;
    using Microsoft.Hadoop.Avro.Container;

    //序列化示例中使用的示例类
    [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
    internal class SensorData
        {
    [DataMember(Name = "Location")]
    public Location Position { get; set; }

    [DataMember(Name = "Value")]
    public byte[] Value { get; set; }
        }

    //序列化示例中使用的示例结构
    [DataContract]
    internal struct Location
        {
    [DataMember]
    public int Floor { get; set; }

    [DataMember]
    public int Room { get; set; }
        }

    //此类包含所有演示
    //Microsoft Avro Library 用法的方法
    public class AvroSample
        {

    //使用反射和 Avro 对象容器文件序列化和反序列化示例数据集
    //使用 Deflate 编解码器压缩已序列化的数据
    public void SerializeDeserializeUsingObjectContainersReflection()
            {

    Console.WriteLine("SERIALIZATION USING REFLECTION AND AVRO OBJECT CONTAINER FILES\n");

    //Avro 对象容器文件的路径
    string path = "AvroSampleReflectionDeflate.avro";

    //使用示例类和结构创建数据集
    var testData = new List<SensorData>
                        {
    new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } },
    new SensorData { Value = new byte[] { 6, 7, 8, 9 }, Position = new Location { Room = 244, Floor = 1 } }
                        };

    //序列化数据并将其保存到文件中
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Serializing Sample Data Set...");

    //为类型 SensorData 创建一个 SequentialWriter 实例，该实例可以将 SensorData 对象序列序列化到流
    //将使用 Deflate 编解码器压缩数据
    using (var w = AvroContainer.CreateWriter<SensorData>(buffer, Codec.Deflate))
                    {
    using (var writer = new SequentialWriter<SensorData>(w, 24))
                        {
    // 使用序列写入器将数据序列化到流
    testData.ForEach(writer.Write);
                        }
                    }

    //将流保存到文件中
    Console.WriteLine("Saving serialized data to file...");
    if (!WriteFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }
                }

    //读取数据并对数据进行反序列化
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Reading data from file...");

    //从对象容器文件中读取数据
    if (!ReadFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }

    Console.WriteLine("Deserializing Sample Data Set...");

    //准备用于反序列化数据的流
    buffer.Seek(0, SeekOrigin.Begin);

    //为类型 SensorData 创建一个 SequentialReader，该项将对给定流中的所有已序列化对象进行反序列化
    //它允许遍历反序列化后的对象，因为它实现了 IEnumerable <T> 接口
    using (var reader = new SequentialReader<SensorData>(
    AvroContainer.CreateReader<SensorData>(buffer, true)))
                    {
    var results = reader.Objects;

    //最后，验证反序列化后的数据是否与原始数据匹配
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

    //删除文件
    RemoveFile(path);
            }

            //
    //帮助器方法
            //

    //比较两个 SensorData 对象
    private bool Equal(SensorData left, SensorData right)
            {
    return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }

    //将内存流保存到具有给定路径的新文件
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
    Console.WriteLine("The following exception was thrown during creation and writing to the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                    }
                }
    else
                {
    Console.WriteLine("Can not create file \"{0}\".File already exists", path);
    return false;

                }
            }

    //使用给定路径将文件内容读取到内存流
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
    Console.WriteLine("The following exception was thrown during reading from the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                }
            }

    //使用给定路径删除文件
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
    Console.WriteLine("The following exception was thrown during deleting the file \"{0}\"", path);
    Console.WriteLine(e.Message);
                    }
                }
    else
                {
    Console.WriteLine("Can not delete file \"{0}\".File does not exist", path);
                }
            }

    static void Main()
            {

    string sectionDivider = "---------------------------------------- ";

    //创建 AvroSample 类的一个实例，并调用
    //说明不同序列化方案的方法
    AvroSample Sample = new AvroSample();

    //使用反射序列化到 Avro 对象容器文件
    Sample.SerializeDeserializeUsingObjectContainersReflection();

    Console.WriteLine(sectionDivider);
    Console.WriteLine("Press any key to exit.");
    Console.Read();
            }
        }
    }
    // 此示例应显示以下输出：
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

## 使用对象容器文件进行序列化与使用通用记录进行序列化

此示例类似于[第二个示例][**通过通用记录进行序列化**]中的方案，都是使用 JSON 显式指定架构，不同的是，此处未假定对架构进行反序列化的读取器知道架构。

测试数据集将使用显式定义的 JSON 架构收集到 [**AvroRecord**][2] 对象列表中，然后存储在由 [**AvroContainer**][] 类表示的对象容器文件中。此容器文件将创建一个写入器，该写入器用于将未压缩的数据序列化到内存流，然后将该内存流保存到文件中。在创建读取器时，使用 [**Codex.Null**][] 参数指定不对此数据进行压缩。

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

    //此类包含所有演示
    //Microsoft Avro Library 用法的方法
    public class AvroSample
        {

    //使用通用记录和 Avro 对象容器文件序列化和反序列化示例数据集
    //未对序列化后的数据进行压缩
    public void SerializeDeserializeUsingObjectContainersGenericRecord()
            {
    Console.WriteLine("SERIALIZATION USING GENERIC RECORD AND AVRO OBJECT CONTAINER FILES\n");

    //Avro 对象容器文件的路径
    string path = "AvroSampleGenericRecordNullCodec.avro";

    Console.WriteLine("Defining the Schema and creating Sample Data Set...");

    //使用 JSON 定义架构
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

    //根据架构创建通用序列化程序
    var serializer = AvroSerializer.CreateGeneric(Schema);
    var rootSchema = serializer.WriterSchema as RecordSchema;

    //创建用于表示数据的通用记录
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

    //序列化数据并将其保存到文件中
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Serializing Sample Data Set...");

    //为类型 SensorData 创建一个 SequentialWriter 实例，该实例可以将 SensorData 对象序列序列化到流
    //将不压缩数据（Null 压缩编解码器）
    using (var writer = AvroContainer.CreateGenericWriter(Schema, buffer, Codec.Null))
                    {
    using (var streamWriter = new SequentialWriter<object>(writer, 24))
                        {
    // 使用序列写入器将数据序列化到流
    testData.ForEach(streamWriter.Write);
                        }
                    }

    Console.WriteLine("Saving serialized data to file...");

    //将流保存到文件中
    if (!WriteFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }
                }

    //读取数据并对数据进行反序列化
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Reading data from file...");

    //从对象容器文件中读取数据
    if (!ReadFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }

    Console.WriteLine("Deserializing Sample Data Set...");

    //准备用于反序列化数据的流
    buffer.Seek(0, SeekOrigin.Begin);

    //为类型 SensorData 创建一个 SequentialReader，该项将对给定流中的所有已序列化对象进行反序列化
    //它允许遍历反序列化后的对象，因为它实现了 IEnumerable <T> 接口
    using (var reader = AvroContainer.CreateGenericReader(buffer))
                    {
    using (var streamReader = new SequentialReader<object>(reader))
                        {
    var results = streamReader.Objects;

    Console.WriteLine("Comparing Initial and Deserialized Data Sets...");

    //最后，验证结果
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

    //删除文件
    RemoveFile(path);
            }

            //
    //帮助器方法
            //

    //将内存流保存到具有给定路径的新文件
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
    Console.WriteLine("The following exception was thrown during creation and writing to the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                    }
                }
    else
                {
    Console.WriteLine("Can not create file \"{0}\".File already exists", path);
    return false;

                }
            }

    //使用给定路径将文件内容读取到内存流
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
    Console.WriteLine("The following exception was thrown during reading from the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                }
            }

    //使用给定路径删除文件
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
    Console.WriteLine("The following exception was thrown during deleting the file \"{0}\"", path);
    Console.WriteLine(e.Message);
                    }
                }
    else
                {
    Console.WriteLine("Can not delete file \"{0}\".File does not exist", path);
                }
            }

    static void Main()
            {

    string sectionDivider = "---------------------------------------- ";

    //创建 AvroSample 类的一个实例，并调用
    //说明不同序列化方案的方法
    AvroSample Sample = new AvroSample();

    //使用通用记录序列化到 Avro 对象容器文件
    Sample.SerializeDeserializeUsingObjectContainersGenericRecord();

    Console.WriteLine(sectionDivider);
    Console.WriteLine("Press any key to exit.");
    Console.Read();
            }
        }
    }
    // 此示例应显示以下输出：
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

## 使用对象容器文件通过自定义压缩编解码器进行序列化

下面的示例演示如何将自定义压缩编解码器用于 Avro 对象容器文件。[Avro 规范][]允许使用可选的压缩编解码器（除了 **Null** 和 **Deflate** 默认压缩编解码器外）。此示例未完全实现类似 Snappy（在 [Avro 规范][4]中作为支持的可选编解码器提及）的新编解码器。它演示如何使用 [**Deflate**][3] 编解码器的 .NET Framework 4.5 实现，后者基于 [zlib][] 压缩库提供比默认的 .NET Framework 4.0 版本更好的压缩算法。

    // 
    // 编译此代码时，需要将参数 Target Framework 设为“.NET Framework 4.5”
    // 以确保使用所需的 Deflate 压缩算法实现
    // 请确保已相应地设置 C# 项目
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

    #region 定义要序列化的对象
    //序列化示例中使用的示例类
    [DataContract(Name = "SensorDataValue", Namespace = "Sensors")]
    internal class SensorData
        {
    [DataMember(Name = "Location")]
    public Location Position { get; set; }

    [DataMember(Name = "Value")]
    public byte[] Value { get; set; }
        }

    //序列化示例中使用的示例结构
    [DataContract]
    internal struct Location
        {
    [DataMember]
    public int Floor { get; set; }

    [DataMember]
    public int Room { get; set; }
        }
    #endregion

    #region 基于 .NET Framework V.4.5 Deflate 定义自定义编解码器
    //Avro.NET 编解码器类包含两个方法： 
    //GetCompressedStreamOver(Stream uncompressed) 和 GetDecompressedStreamOver(Stream compressed)
    //这两个方法是用于数据压缩的主要方法。
    //若要启用自定义编解码器，需要为所需的编解码器实现这些方法

    #region 定义压缩和解压缩流
    //DeflateStream（来自 System.IO.Compression 命名空间的用于实现 Deflate 算法的类）
    //不能直接用于 Avro，因为它不支持查找等重要操作。
    //因此，需要实现两个从 Stream 继承的类
    //（一个用于压缩流，一个用于解压缩流）
    //这两个类均使用 Deflate 压缩，并实现所有所需的功能 
    internal sealed class CompressionStreamDeflate45 :Stream
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

    internal sealed class DecompressionStreamDeflate45 :Stream
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

    #region 定义编解码器
    //定义实际的编解码器类，该类包含操作流所需的方法：
    //GetCompressedStreamOver(Stream uncompressed) 和 GetDecompressedStreamOver(Stream compressed)
    //编解码器类使用上面定义的用于压缩和解压缩流的类
    internal sealed class DeflateCodec45 :Codec
        {

    //我们只是使用 Deflate 的不同实现，因此 CodecName 仍为“deflate”
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

    #region 定义已修改的编解码器工厂
    //定义要在读取器中使用的已修改的编解码器工厂
    //它将捕获使用“deflate”的尝试，并提供自定义编解码器 
    //对于所有其他情况，它将依赖于基类 (CodecFactory)
    internal sealed class CodecFactoryDeflate45 :CodecFactory
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

    #region 包含演示方法的示例类
    //此类包含演示
    //Microsoft Avro Library 用法的方法
    public class AvroSample
        {

    //使用反射和 Avro 对象容器文件序列化和反序列化示例数据集
    //使用自定义压缩编解码器（.NET Framework 4.5 的 Deflate 实现）压缩已序列化的数据
            //
    //虽然可以轻松使用文件流，但此示例使用内存流进行与序列化、反序列化和
    //对象容器操作相关的所有操作。
    public void SerializeDeserializeUsingObjectContainersReflectionCustomCodec()
            {

    Console.WriteLine("SERIALIZATION USING REFLECTION, AVRO OBJECT CONTAINER FILES AND CUSTOM CODEC\n");

    //Avro 对象容器文件的路径
    string path = "AvroSampleReflectionDeflate45.avro";

    //使用示例类和结构创建数据集
    var testData = new List<SensorData>
                        {
    new SensorData { Value = new byte[] { 1, 2, 3, 4, 5 }, Position = new Location { Room = 243, Floor = 1 } },
    new SensorData { Value = new byte[] { 6, 7, 8, 9 }, Position = new Location { Room = 244, Floor = 1 } }
                        };

    //序列化数据并将其保存到文件中
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Serializing Sample Data Set...");

    //为类型 SensorData 创建一个 SequentialWriter 实例，该实例可以将 SensorData 对象序列序列化到流
    //此处将介绍自定义编解码器。为方便起见，下一个注释的代码行将演示如何使用内置 Deflate。
    //请注意，由于此示例处理 Deflate 的不同实现，因此内置编解码器和自定义编解码器在读-写操作中
    //是可互换的
    //using (var w = AvroContainer.CreateWriter<SensorData>(buffer, Codec.Deflate))
    using (var w = AvroContainer.CreateWriter<SensorData>(buffer, new DeflateCodec45()))
                    {
    using (var writer = new SequentialWriter<SensorData>(w, 24))
                        {
    // 使用序列写入器将数据序列化到流
    testData.ForEach(writer.Write);
                        }
                    }

    //将流保存到文件中
    Console.WriteLine("Saving serialized data to file...");
    if (!WriteFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }
                }

    //读取数据并对数据进行反序列化
    //创建一个内存流缓冲区
    using (var buffer = new MemoryStream())
                {
    Console.WriteLine("Reading data from file...");

    //从对象容器文件中读取数据
    if (!ReadFile(buffer, path))
                    {
    Console.WriteLine("Error during file operation.Quitting method");
    return;
                    }

    Console.WriteLine("Deserializing Sample Data Set...");

    //准备用于反序列化数据的流
    buffer.Seek(0, SeekOrigin.Begin);

    //由于使用了 SequentialReader <T> 构造函数签名，在显式指定编解码器工厂时，
    //AvroSerializerSettings 实例是必需的
    //如果你要使用内置 Deflate，则可以注释掉下面一行（请参见下一条注释）
    AvroSerializerSettings settings = new AvroSerializerSettings();

    //为类型 SensorData 创建一个 SequentialReader，该项将对给定流中的所有已序列化对象进行反序列化
    //它允许遍历反序列化后的对象，因为它实现了 IEnumerable <T> 接口
    //此处将介绍自定义编解码器工厂。
    //为方便起见，下一个注释的代码行将演示如何使用内置 Deflate
    //（在此示例中，不需要使用显式“编解码器工厂”参数）。
    //请注意，由于此示例处理 Deflate 的不同实现，因此内置编解码器和自定义编解码器在读-写操作中
    //是可互换的
    //using (var reader = new SequentialReader<SensorData>(AvroContainer.CreateReader<SensorData>(buffer, true)))
    using (var reader = new SequentialReader<SensorData>(
    AvroContainer.CreateReader<SensorData>(buffer, true, settings, new CodecFactoryDeflate45())))
                    {
    var results = reader.Objects;

    //最后，验证反序列化后的数据是否与原始数据匹配
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

    //删除文件
    RemoveFile(path);
            }
    #endregion

    #region 帮助器方法

    //比较两个 SensorData 对象
    private bool Equal(SensorData left, SensorData right)
            {
    return left.Position.Equals(right.Position) && left.Value.SequenceEqual(right.Value);
            }

    //将内存流保存到具有给定路径的新文件
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
    Console.WriteLine("The following exception was thrown during creation and writing to the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                    }
                }
    else
                {
    Console.WriteLine("Can not create file \"{0}\".File already exists", path);
    return false;

                }
            }

    //使用给定路径将文件内容读取到内存流
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
    Console.WriteLine("The following exception was thrown during reading from the file \"{0}\"", path);
    Console.WriteLine(e.Message);
    return false;
                }
            }

    //使用给定路径删除文件
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
    Console.WriteLine("The following exception was thrown during deleting the file \"{0}\"", path);
    Console.WriteLine(e.Message);
                    }
                }
    else
                {
    Console.WriteLine("Can not delete file \"{0}\".File does not exist", path);
                }
            }
    #endregion

    static void Main()
            {

    string sectionDivider = "---------------------------------------- ";

    //创建 AvroSample 类的一个实例，并调用
    //说明不同序列化方案的方法
    AvroSample Sample = new AvroSample();

    //使用自定义编解码器通过反射序列化到 Avro 对象容器文件
    Sample.SerializeDeserializeUsingObjectContainersReflectionCustomCodec();

    Console.WriteLine(sectionDivider);
    Console.WriteLine("Press any key to exit.");
    Console.Read();
            }
        }
    }
    // 此示例应显示以下输出：
    // SERIALIZATION USING REFLECTION, AVRO OBJECT CONTAINER FILES AND CUSTOM CODEC
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

  [JSON]: http://www.json.org
  [Apache Avro 规范]: http://avro.apache.org/docs/current/spec.html
  [**AvroRecord**]: http://msdn.microsoft.com/en-us/library/microsoft.hadoop.avro.avrorecord.aspx
  [Microsoft .NET Framework v4.0]: http://www.microsoft.com/zh-cn/download/details.aspx?id=17851
  [Newtonsoft Json.NET]: http://james.newtonking.com/json
  [Azure 代码示例]: http://code.msdn.microsoft.com/windowsazure/Serialize-data-with-the-86055923
  [1]: http://code.msdn.microsoft.com/windowsazure/Serialize-data-with-the-67159111
  [**通过反射进行序列化**]: #Scenario1
  [**通过通用记录进行序列化**]: #Scenario2
  [**使用对象容器文件通过反射进行序列化**]: #Scenario3
  [**使用对象容器文件通过通用记录进行序列化**]: #Scenario4
  [**使用对象容器文件通过自定义压缩编解码器进行序列化**]: #Scenario5
  [**IAvroSeralizer**]: http://msdn.microsoft.com/zh-cn/library/dn627341.aspx
  [2]: http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.avrorecord.aspx
  [**AvroContainer**]: http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.container.avrocontainer.aspx
  [**SequentialWriter**]: http://msdn.microsoft.com/zh-cn/library/dn627340.aspx
  [**Deflate**]: http://msdn.microsoft.com/zh-cn/library/system.io.compression.deflatestream(v=vs.100).aspx
  [3]: http://msdn.microsoft.com/zh-cn/library/system.io.compression.deflatestream(v=vs.110).aspx
  [**Codex.Null**]: http://msdn.microsoft.com/zh-cn/library/microsoft.hadoop.avro.container.codec.null.aspx
  [Avro 规范]: http://avro.apache.org/docs/current/spec.html#Required+Codecs
  [4]: http://avro.apache.org/docs/current/spec.html#snappy
  [zlib]: http://zlib.net/
