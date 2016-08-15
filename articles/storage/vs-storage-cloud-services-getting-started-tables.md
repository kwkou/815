<properties
    pageTitle="表存储和 Visual Studio 连接服务（云服务）入门 | Azure"
	description="在使用 Visual Studio 连接服务连接到存储帐户后，如何开始在 Visual Studio 的云服务项目中使用 Azure 表存储"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
    ms.date="05/08/2016"
	wacn.date="06/13/2016"/>

# 开始使用 Azure 表存储和 Visual Studio 连接服务（云服务项目）

##概述

本文介绍通过使用 Visual Studio 中的“添加连接服务”对话框在云服务项目中创建或引用 Azure 存储帐户之后，如何开始在 Visual Studio 中使用 Azure 表存储。执行“添加连接服务”操作会安装相应的 NuGet 程序包，以访问项目中的 Azure 存储，并将存储帐户的连接字符串添加到项目配置文件中。

Azure 表存储服务使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。

若要开始，首先需要在存储帐户中创建表。我们将展示如何使用代码创建 Azure 表，以及如何执行基本的表和实体操作，例如添加、修改、读取和删除表实体。示例是用 C# 代码编写的，并使用了 [Azure .NET 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。

**注意：**执行 Azure 存储调用的一些 API 是异步的。有关详细信息，请参阅[使用 Async 和 Await 进行异步编程](http://msdn.microsoft.com/zh-cn/library/hh191443.aspx)。下面的代码假定正在使用异步编程方法。

- 有关以编程方式操作表的详细信息，请参阅[通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)。
- 有关 Azure 存储空间的常规信息，请参阅[存储空间文档](/documentation/services/storage)。
- 有关 Azure 云服务的常规信息，请参阅[云服务文档](/documentation/services/cloud-services)。
- 有关对 ASP.NET 应用程序进行编程的详细信息，请参阅 [ASP.NET](http://www.asp.net)。

## 使用代码访问表

若要在云服务项目中访问表，需要在访问 Azure 表存储的任何 C# 源文件中包含以下项。

1. 请确保 C# 文件顶部的命名空间声明包括这些 **using** 语句。

		using Microsoft.Framework.Configuration;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Table;
		using System.Threading.Tasks;
		using LogLevel = Microsoft.Framework.Logging.LogLevel;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。

		 CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		   CloudConfigurationManager.GetSetting("<storage account name>
         _AzureStorageConnectionString"));
> [AZURE.NOTE]在下列示例中，在代码的前面使用上述全部代码。

3. 获取 **CloudTableClient** 对象，以引用存储帐户中的表对象。

         // Create the table client.
         CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

4. 获取 **CloudTable** 引用对象，以引用特定的表和实体。

    	// Get a reference to a table named "peopleTable".
	    CloudTable peopleTable = tableClient.GetTableReference("peopleTable");

## 使用代码创建表

若要创建 Azure 表，只需在获取 **CloudTable** 对象后添加对 **CreateIfNotExistsAsync** 的调用，如“使用代码访问表”一节中所述。

	// Create the CloudTable if it does not exist.
	await peopleTable.CreateIfNotExistsAsync();

## 将实体添加到表

若要将实体添加到表，请创建用于定义实体的属性的类。以下代码定义了将客户的名字和姓氏分别用作行键和分区键的 **CustomerEntity** 实体类。

	public class CustomerEntity : TableEntity
	{
	    public CustomerEntity(string lastName, string firstName)
	    {
	        this.PartitionKey = lastName;
	        this.RowKey = firstName;
	    }

	    public CustomerEntity() { }

	    public string Email { get; set; }

	    public string PhoneNumber { get; set; }
	}

将使用之前在“使用代码访问表”中创建的 **CloudTable** 对象完成涉及实体的表操作。 **TableOperation** 对象表示将完成的操作。以下代码示例演示如何创建 **CloudTable** 对象以及 **CustomerEntity** 对象。为准备此操作，会创建一个 **TableOperation** 以将客户实体插入该表中。最后，将通过调用 **CloudTable.ExecuteAsync** 执行此操作。

	// Create a new customer entity.
	CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
	customer1.Email = "Walter@contoso.com";
	customer1.PhoneNumber = "425-555-0101";

	// Create the TableOperation that inserts the customer entity.
	TableOperation insertOperation = TableOperation.Insert(customer1);

	// Execute the insert operation.
	await peopleTable.ExecuteAsync(insertOperation);


## 插入一批实体

您可以通过单个写入操作将多个实体插入表中。以下代码示例将创建两个实体对象（“Jeff Smith”和“Ben Smith”），使用 Insert 方法将它们添加到 **TableBatchOperation** 对象，然后通过调用 **CloudTable.ExecuteBatchAsync** 启动操作。

    // Create the batch operation.
    TableBatchOperation batchOperation = new TableBatchOperation();

    // Create a customer entity and add it to the table.
    CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
    customer1.Email = "Jeff@contoso.com";
    customer1.PhoneNumber = "425-555-0104";

    // Create another customer entity and add it to the table.
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.Email = "Ben@contoso.com";
    customer2.PhoneNumber = "425-555-0102";

    // Add both customer entities to the batch insert operation.
    batchOperation.Insert(customer1);
    batchOperation.Insert(customer2);

    // Execute the batch operation.
    await peopleTable.ExecuteBatchAsync(batchOperation);

## 获取分区中的所有实体

若要查询表以获取分区中的所有实体，请使用 **TableQuery** 对象。以下代码示例指定了一个筛选器，以筛选分区键为“Smith”的实体。此示例会将查询结果中每个实体的字段输出到控制台。

    // Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
        .Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // Print the fields for each customer.
    TableContinuationToken token = null;
    do
    {
    	TableQuerySegment<CustomerEntity> resultSegment = await peopleTable.ExecuteQuerySegmentedAsync(query, token);
		token = resultSegment.ContinuationToken;

		foreach (CustomerEntity entity in resultSegment.Results)
    	{
    		Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
    		entity.Email, entity.PhoneNumber);
        }
    } while (token != null);

    return View();


## 获取单个实体

您可以编写查询以获取单个特定实体。以下代码使用 **TableOperation** 对象来指定名为“Ben Smith”的客户。此方法仅返回一个实体，而不是一个集合，并且 **TableResult.Result** 中的返回值是一个 **CustomerEntity** 对象。在查询中同时指定分区键和行键是从**表**服务中检索单个实体的最快方法。

	// Create a retrieve operation that takes a customer entity.
	TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

	// Execute the retrieve operation.
	TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);

	// Print the phone number of the result.
	if (retrievedResult.Result != null)
	   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
	else
	   Console.WriteLine("The phone number could not be retrieved.");

## 删除实体
您可以在找到实体后将其删除。以下代码将查找名为“Ben Smith”的客户实体，如果找到，会将其删除。

	// Create a retrieve operation that expects a customer entity.
	TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

	// Execute the operation.
	TableResult retrievedResult = peopleTable.Execute(retrieveOperation);

	// Assign the result to a CustomerEntity object.
	CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

	// Create the Delete TableOperation and then execute it.
	if (deleteEntity != null)
	{
	   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

	   // Execute the operation.
	   await peopleTable.ExecuteAsync(deleteOperation);

	   Console.WriteLine("Entity deleted.");
	}

	else
	   Console.WriteLine("Couldn't delete the entity.");

## 后续步骤

[AZURE.INCLUDE [vs-storage-dotnet-tables-next-steps](../includes/vs-storage-dotnet-tables-next-steps.md)]

<!---HONumber=Mooncake_0606_2016-->