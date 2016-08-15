<properties 
	pageTitle="开始使用 Azure 存储空间和 Visual Studio 连接服务（WebJob 项目）"
	description="在使用 Visual Studio 连接服务连接到存储帐户后，如何开始在 Visual Studio 的 Azure WebJobs 项目中使用 Azure 表存储"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags 
	ms.service="storage"
	ms.date="05/08/2016"
	wacn.date="06/13/2016"/>

# Azure 存储入门（Azure WebJob 项目）

## 概述

本文章提供了 C# 代码示例，用于演示如何在 Azure 表存储服务中使用 Azure WebJobs SDK 版本 1.x。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 版本 1.x。

Azure 表存储服务使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。有关详细信息，请参阅[通过 .NET 开始使用 Azure 表存储](storage-dotnet-how-to-use-tables.md#create-a-table)。

一些代码段显示了手动调用（即：不是使用触发器属性之一调用）的函数中使用的 **Table** 属性。

## 如何向表中添加实体

若要将实体添加到表中，请使用具有 **ICollector<T>** 或 **IAsyncCollector<T>** 参数的 **Table** 属性，其中 **T** 指定你想要添加的实体的架构。属性构造函数使用指定表名称的字符串参数。

下面的代码示例将 **Person** 实体添加到名为 *Ingress* 的表。

		[NoAutomaticTrigger]
		public static void IngressDemo(
		    [Table("Ingress")] ICollector<Person> tableBinding)
		{
		    for (int i = 0; i < 100000; i++)
		    {
		        tableBinding.Add(
		            new Person() {
		                PartitionKey = "Test",
		                RowKey = i.ToString(),
		                Name = "Name" }
		            );
		    }
		}

通常情况下，用于 **ICollector** 的类型派生自 **TableEntity** 或者通过实现 **ITableEntity** 获得，但这不是必需的。以下 **Person** 类之一适用于前面 **Ingress** 方法中所示的代码。

		public class Person : TableEntity
		{
		    public string Name { get; set; }
		}

		public class Person
		{
		    public string PartitionKey { get; set; }
		    public string RowKey { get; set; }
		    public string Name { get; set; }
		}

如果你想要直接使用 Azure 存储 API，则可以向方法签名添加 **CloudStorageAccount** 参数。

## 实时监视

因为数据入口函数通常处理大量数据，WebJobs SDK 仪表板提供了实时监视的数据。“调用日志”部分告诉你函数是否仍在运行。

![Ingress 函数正在运行](./media/vs-storage-webjobs-getting-started-tables/ingressrunning.png)

“调用详细信息”页在运行时报告函数的进度（写入的实体数），并且为你提供中止的机会。

![Ingress 函数正在运行](./media/vs-storage-webjobs-getting-started-tables/ingressprogress.png)

该函数完成时，**调用详细信息**页会报告写入的行数。

![Ingress 函数已完成](./media/vs-storage-webjobs-getting-started-tables/ingresssuccess.png)

## 如何从表中读取多个实体

若要读取表，可将 **Table** 属性用于 **IQueryable<T>** 参数，其中类型 **T** 派生自 **TableEntity** 或者通过实现 **ITableEntity** 来获得。

下面的代码示例读取并记录 **Ingress** 表中所有行：

		public static void ReadTable(
		    [Table("Ingress")] IQueryable<Person> tableBinding,
		    TextWriter logger)
		{
		    var query = from p in tableBinding select p;
		    foreach (Person person in query)
		    {
		        logger.WriteLine("PK:{0}, RK:{1}, Name:{2}",
		            person.PartitionKey, person.RowKey, person.Name);
		    }
		}

### 如何从表中读取单个实体

有一个 **Table** 属性构造函数具有两个附加参数，当你想要绑定到单个表实体时，该参数可以用于指定分区键和行键的。

下面的代码示例基于队列消息中接收到的分区键和行键读取 **Person** 实体的表行：

		public static void ReadTableEntity(
		    [QueueTrigger("inputqueue")] Person personInQueue,
		    [Table("persontable","{PartitionKey}", "{RowKey}")] Person personInTable,
		    TextWriter logger)
		{
		    if (personInTable == null)
		    {
		        logger.WriteLine("Person not found: PK:{0}, RK:{1}",
		                personInQueue.PartitionKey, personInQueue.RowKey);
		    }
		    else
		    {
		        logger.WriteLine("Person found: PK:{0}, RK:{1}, Name:{2}",
		                personInTable.PartitionKey, personInTable.RowKey, personInTable.Name);
		    }
		}


此示例中的 **Person** 类不必实现 **ITableEntity**。

## 如何直接使用 .NET 存储 API 处理表

你还可以将 **Table** 属性和 **CloudTable** 对象一起使用，以便更灵活地处理表。

下面的代码示例使用 **CloudTable** 对象将单个实体添加到 *Ingress* 表中。

		public static void UseStorageAPI(
		    [Table("Ingress")] CloudTable tableBinding,
		    TextWriter logger)
		{
		    var person = new Person()
		        {
		            PartitionKey = "Test",
		            RowKey = "100",
		            Name = "Name"
		        };
		    TableOperation insertOperation = TableOperation.Insert(person);
		    tableBinding.Execute(insertOperation);
		}

有关如何使用 **CloudTable** 对象的详细信息，请参阅[通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)。

## 队列操作指南文章涵盖的相关主题

有关如何处理队列消息触发的表处理，或者不特定于表处理的 WebJobs SDK 方案的信息，请参阅[开始使用 Azure 队列存储和 Visual Studio 连接服务（WebJob 项目）](/documentation/articles/vs-storage-webjobs-getting-started-queues/)。



## 后续步骤

本文章提供了代码示例，演示如何处理用于操作 Azure 表的常见方案。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 文档资源](/documentation/articles/websites-webjobs-resources/)。
 
<!---HONumber=Mooncake_0606_2016-->