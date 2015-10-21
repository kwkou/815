<properties
	pageTitle="如何通过 .NET 使用表存储 | Windows Azure"
	description="了解如何使用 Windows Azure 表存储来创建和删除表，以及在表中插入和查询实体。"
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="adinah"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="08/04/2015"
	wacn.date="09/18/2015"/>



# 如何通过 .NET 使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述

本指南将演示如何使用 Azure 表存储服务执行常见方案。示例是使用 C# 代码编写的，并使用了 .NET 的 Azure 存储客户端库。涉及的方案包括**创建和删除表**，以及**使用表实体**。

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-configure-connection-string-include](../includes/storage-configure-connection-string-include.md)]

## 以编程方式访问表存储

[AZURE.INCLUDE [storage-dotnet-obtain-assembly](../includes/storage-dotnet-obtain-assembly.md)]

### 命名空间声明
在你希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Table;

确保你引用  `Microsoft.WindowsAzure.Storage.dll` 程序集。

[AZURE.INCLUDE [storage-dotnet-retrieve-conn-string](../includes/storage-dotnet-retrieve-conn-string.md)]

## 创建表

利用 **CloudTableClient** 对象，您可以获得表和实体的引用对象。以下代码将创建 **CloudTableClient** 对象并使用它创建新表。本指南中的所有代码假定将构建的应用程序是 Azure 云服务项目，并且使用存储在 Azure 应用程序的服务配置中的存储连接字符串。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the table if it doesn't exist.
    CloudTable table = tableClient.GetTableReference("people");
    table.CreateIfNotExists();

## 将实体添加到表

实体将映射到使用派生自 **TableEntity** 的自定义类的 C# 对象。若要将实体添加到表，请创建用于定义实体的属性的类。以下代码定义将客户的名字和姓氏分别用作行键和分区键的实体类。实体的分区键和行键共同唯一地标识表中的实体。查询分区键相同的实体的速度快于查询分区键不同的实体的速度，但使用不同的分区键可实现更高的并行操作可伸缩性。对于应存储在表服务中的任何属性，该属性必须是公开 `get` 和 `set` 的受支持类型的公共属性。此外，你的实体类型*必须*公开不带参数的构造函数。

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

调用实体的表操作通过你在“如何创建表”中创建的 **CloudTable** 对象执行。 用一个 **TableOperation** 对象表示要执行的操作。以下代码示例演示如何创建 **CloudTable** 对象以及 **CustomerEntity** 对象。为准备此操作，会创建一个 **TableOperation** 对象以将客户实体插入该表中。最后，通过调用 **CloudTable.Execute** 执行此操作。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
       CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
	
	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");

    // Create a new customer entity.
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // Create the TableOperation object that inserts the customer entity.
    TableOperation insertOperation = TableOperation.Insert(customer1);

    // Execute the insert operation.
    table.Execute(insertOperation);

## 插入一批实体

你可以通过一次写入操作将一批实体插入表中。批处理操作的一些其他注意事项：

-  你可以在同一批处理操作中执行更新、删除和插入操作。
-  单个批处理操作最多可包含 100 个实体。
-  单次批处理操作中的所有实体都必须具有相同的分区键。
-  可以作为批处理操作执行查询时，此操作必须是批处理中仅有的操作。

<!-- -->
以下代码示例创建两个实体对象，并使用 **Insert** 方法将其中每个对象都添加到 **TableBatchOperation** 中。然后调用 **CloudTable.Execute** 以执行此操作。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    
	// Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

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
	table.ExecuteBatch(batchOperation);

## 检索分区中的所有实体

若要查询表以获取分区中的所有实体，请使用 **TableQuery** 对象。以下代码示例指定了一个筛选器，以筛选分区键为“Smith”的实体。此示例会将查询结果中每个实体的字段输出到控制台。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // Print the fields for each customer.
    foreach (CustomerEntity entity in table.ExecuteQuery(query))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## 检索分区中的一部分实体

如果不想查询分区中的所有实体，则可以通过结合使用分区键筛选器与行键筛选器来指定一个范围。以下代码示例使用两个筛选器来获取分区“Smith”中的、行键（名字）以字母“E”前面的字母开头的所有实体，然后输出查询结果。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    //Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

	// Create the table query.
    TableQuery<CustomerEntity> rangeQuery = new TableQuery<CustomerEntity>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "E")));

    // Loop through the results, displaying information about the entity.
    foreach (CustomerEntity entity in table.ExecuteQuery(rangeQuery))
    {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
            entity.Email, entity.PhoneNumber);
    }

## 检索单个实体

你可以编写查询以检索单个特定实体。以下代码使用 **TableOperation** 来指定客户“Ben Smith”。此方法仅返回一个实体，而不是一个集合，并且 **TableResult.Result** 中的返回值是一个 **CustomerEntity** 对象。在查询中指定分区键和行键是从表服务中检索单个实体的最快方法。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the retrieve operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Print the phone number of the result.
	if (retrievedResult.Result != null)
	   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
	else
	   Console.WriteLine("The phone number could not be retrieved.");

## 替换实体

若要更新实体，请从表服务中检索它，修改实体对象，然后将更改保存回表服务。以下代码将更改现有客户的电话号码。此代码使用 **Replace**，而不是调用 **Insert**。这将导致在服务器上完全替换该实体，除非服务器上的该实体自检索到它以后发生更改，在此情况下，该操作将失败。操作失败将防止您的应用程序无意中覆盖应用程序的其他组件在检索与更新之间所做的更改。正确处理此失败问题的方法是再次检索实体，进行更改（如果仍有效），然后再次执行 **Replace** 操作。下一节将为你演示如何重写此行为。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
	{
	   // Change the phone number.
	   updateEntity.PhoneNumber = "425-555-0105";

	   // Create the InsertOrReplace TableOperation.
	   TableOperation updateOperation = TableOperation.Replace(updateEntity);

	   // Execute the operation.
	   table.Execute(updateOperation);

	   Console.WriteLine("Entity updated.");
	}

	else
	   Console.WriteLine("Entity could not be retrieved.");

## 插入或替换实体

如果该实体自从服务器中检索到它以后发生更改，则 **Replace** 操作将失败。此外，你必须首先从服务器中检索该实体，**Replace** 操作才能成功。但是，有时你不知道服务器上是否存在该实体以及存储在其中的当前值是否无关。更新操作会将其全部覆盖。为此，你应使用 **InsertOrReplace** 操作。如果该实体不存在，此操作将插入它，如果存在，则替换它，而不管上次更新是何时进行的。在以下代码示例中，仍将检索 Ben Smith 的客户实体，但稍后会使用 **InsertOrReplace** 将其保存回服务器。将覆盖在检索与更新操作之间对实体进行的任何更新。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that takes a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity object.
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
	{
	   // Change the phone number.
	   updateEntity.PhoneNumber = "425-555-1234";

	   // Create the InsertOrReplace TableOperation.
	   TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(updateEntity);

	   // Execute the operation.
	   table.Execute(insertOrReplaceOperation);

	   Console.WriteLine("Entity was updated.");
	}

	else
	   Console.WriteLine("Entity could not be retrieved.");

## 查询一部分实体属性

表查询可以只检索实体中的少数几个属性而不是所有实体属性。此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。以下代码中的查询只返回表中实体的电子邮件地址。这可通过使用 **DynamicTableEntity** 和 **EntityResolver** 的查询来实现。你可以在“Upsert 和查询投影介绍”[博客文章][]中更加详细地了解投影。注意，本地存储模拟器不支持投影，因此，此代码仅在使用表服务中的帐户时才能运行。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    //Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Define the query, and select only the Email property.
    TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

    // Define an entity resolver to work with the entity after retrieval.
    EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue : null;

    foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
    {
        Console.WriteLine(projectedEmail);
    }

## 删除实体

在检索实体之后，可使用更新实体的相同演示模式轻松删除该实体。以下代码检索并删除一个客户实体。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    //Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a retrieve operation that expects a customer entity.
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // Execute the operation.
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // Assign the result to a CustomerEntity.
    CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

    // Create the Delete TableOperation.
	if (deleteEntity != null)
	{
	   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

	   // Execute the operation.
	   table.Execute(deleteOperation);

	   Console.WriteLine("Entity deleted.");
	}

	else
	   Console.WriteLine("Could not retrieve the entity.");

## 删除表

最后，以下代码示例将从存储帐户中删除表。在删除表之后的一段时间内无法重新创建它。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    //Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Delete the table it if exists.
    table.DeleteIfExists();

## 以异步方式检索页中的实体

如果你正在读取大量实体，并且你想要如已经检索到一样处理/显示实体而不是等待返回全部实体，则可以通过使用分段查询检索实体。此示例显示如何通过 Async-Await 模式以页面形式返回结果，这样就不会在等待返回大量结果时阻止操作的执行。有关在 .NET 中使用 Async-Await 模式的详细信息，请参阅 [使用 Async 和 Await 进行异步编程（C# 和 Visual Basic）](https://msdn.microsoft.com/zh-cn/library/hh191443.aspx)。

    // Initialize a default TableQuery to retrieve all the entities in the table.
    TableQuery<CustomerEntity> tableQuery = new TableQuery<CustomerEntity>();

    // Initialize the continuation token to null to start from the beginning of the table.
    TableContinuationToken continuationToken = null;

    do
    {
        // Retrieve a segment (up to 1,000 entities).
        TableQuerySegment<CustomerEntity> tableQueryResult =
            await table.ExecuteQuerySegmentedAsync(tableQuery, continuationToken);

        // Assign the new continuation token to tell the service where to
        // continue on the next iteration (or null if it has reached the end).
        continuationToken = tableQueryResult.ContinuationToken;

        // Print the number of rows retrieved.
        Console.WriteLine("Rows retrieved {0}", tableQueryResult.Results.Count);

    // Loop until a null continuation token is received, indicating the end of the table.
    } while(continuationToken != null);

## 后续步骤

现在，你已了解有关表存储的基础知识，请按照下面的链接了解更复杂的存储任务：

<ul>
<li>查看表服务参考文档，了解有关可用 API 的完整详情：
  <ul>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx">.NET 存储客户端库参考</a>
    </li>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179355">REST API 参考</a></li>
  </ul>
</li>
<li>通过查看 <a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx">Azure 存储空间文档</a>了解可以通过 Azure 存储空间执行的更高级任务。</li>
<li>了解如何通过使用 <a href="/documentation/articles/websites-dotnet-webjobs-sdk/">Azure WebJobs SDK</a> 简化为使用 Azure 存储空间而写的代码。</li>
<li>查看更多功能指南，以了解在 Azure 中存储数据的其他方式：
  <ul>
    <li>使用 <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">Blob 存储</a>来存储非结构化数据。</li>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/">队列存储</a>来存储结构化数据。</li>
    <li>使用 <a href="/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/">SQL 数据库</a> 来存储关系数据。</li>
  </ul>
</li>
</ul>

  [下载并安装 Azure SDK for.NET]: /develop/net/
  [在 Visual Studio 中创建 Azure 项目]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx
  
  [Blob5]: ./media/storage-dotnet-how-to-use-table-storage/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-table-storage/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-table-storage/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-table-storage/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-table-storage/blob9.png
  
  [博客文章]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [空间]: http://nuget.org/packages/System.Spatial/5.0.2
<!---HONumber=70-->