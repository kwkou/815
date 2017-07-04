<properties
    pageTitle="通过 .NET 开始使用 Azure 表存储 | Azure"
    description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
    services="storage"
    documentationcenter=".net"
    author="mmacy"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="fe46d883-7bed-49dd-980e-5c71df36adb3"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="04/10/2017"
    wacn.date="05/02/2017"
    ms.author="marsma"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="aa81e2c73a61786c56e0168f8087323273d500bc"
    ms.lasthandoff="04/22/2017" />

# <a name="get-started-with-azure-table-storage-using-net"></a>通过 .NET 开始使用 Azure 表存储
[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

[AZURE.INCLUDE [storage-check-out-samples-dotnet](../../includes/storage-check-out-samples-dotnet.md)]

Azure 表存储是一项用于在云中存储结构化 NoSQL 数据的服务，通过无架构设计提供键/属性存储。 因为表存储无架构，因此可以很容易地随着你的应用程序需求的发展使数据适应存储。 对于许多类型的应用程序来说，访问表存储数据速度快且经济高效，在数据量相似的情况下，其成本通常比传统 SQL 要低。

可以使用表存储来存储灵活的数据集，例如 Web 应用程序的用户数据、通讯簿、设备信息，或者你的服务需要的其他类型的元数据。 可以在表中存储任意数量的实体，并且一个存储帐户可以包含任意数量的表，直至达到存储帐户的容量极限。

### <a name="about-this-tutorial"></a>关于本教程
本教程介绍如何在某些常见的 Azure 表存储方案中使用[适用于 .NET 的 Azure 存储客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)。 这些方案通过 C# 示例来呈现，用于创建和删除表，以及插入、更新、删除和查询表数据。

## <a name="prerequisites"></a>先决条件

若要成功完成本教程，需要具备以下先决条件：

* [Microsoft Visual Studio](https://www.visualstudio.com/visual-studio-homepage-vs.aspx)
* [适用于 .NET 的 Azure 存储空间客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)
* [适用于 .NET 的 Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
* [Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-development-environment-include](../../includes/storage-development-environment-include.md)]

### <a name="add-using-directives"></a>添加 using 指令
将以下 **using** 指令添加到 `Program.cs` 文件顶部：

    using Microsoft.Azure; // Namespace for CloudConfigurationManager
    using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
    using Microsoft.WindowsAzure.Storage.Table; // Namespace for Table storage types

### <a name="parse-the-connection-string"></a>解析连接字符串
[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="create-the-table-service-client"></a>创建表服务客户端
[CloudTableClient][dotnet_CloudTableClient] 类用于检索存储在表存储中的表和实体。 下面是创建表服务客户端的一种方法：

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

现在，你已准备好编写从表存储读取数据并将数据写入表存储的代码。

## <a name="create-a-table"></a>创建表
此示例演示如何创建表（如果表已经不存在）：

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Retrieve a reference to the table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create the table if it doesn't exist.
    table.CreateIfNotExists();

## <a name="add-an-entity-to-a-table"></a>将实体添加到表
实体使用派生自 [TableEntity][dotnet_TableEntity] 的自定义类映射到 C# 对象。 若要将实体添加到表，请创建用于定义实体的属性的类。 以下代码定义将客户的名字和姓氏分别用作行键和分区键的实体类。 实体的分区键和行键共同唯一地标识表中的实体。 查询分区键相同的实体的速度快于查询分区键不同的实体的速度，但使用不同的分区键可实现更高的并行操作可伸缩性。 需要存储在表中的实体必须是受支持的类型，例如，必须派生自 [TableEntity][dotnet_TableEntity] 类。 要存储在表中的实体属性必须是相应类型的公共属性，并且允许获取和设置值。 此外，你的实体类型 *必须* 公开不带参数的构造函数。

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

涉及实体的表操作通过你先前在“创建表”部分中创建的 [CloudTable][dotnet_CloudTable] 对象执行。 用一个 [TableOperation][dotnet_TableOperation] 对象表示要执行的操作。 以下代码示例演示如何创建 [CloudTable][dotnet_CloudTable] 对象以及 **CustomerEntity** 对象。 为准备此操作，会创建一个 [TableOperation][dotnet_TableOperation] 对象以将客户实体插入该表中。 最后，通过调用 [CloudTable][dotnet_CloudTable].[Execute][dotnet_CloudTable_Execute] 执行此操作。

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

## <a name="insert-a-batch-of-entities"></a>插入一批实体
你可以通过一次写入操作将一批实体插入表中。 批处理操作的一些其他注意事项：

* 你可以在同一批处理操作中执行更新、删除和插入操作。
* 单个批处理操作最多可包含 100 个实体。
* 单次批处理操作中的所有实体都必须具有相同的分区键。
* 虽然可以将某个查询作为批处理操作执行，但该操作必须是批处理中仅有的操作。

以下代码示例创建两个实体对象，并使用 [Insert][dotnet_TableBatchOperation_Insert] 方法将其中每个对象都添加到 [TableBatchOperation][dotnet_TableBatchOperation] 中。 然后调用 [CloudTable][dotnet_CloudTable].[ExecuteBatch][dotnet_CloudTable_ExecuteBatch] 以执行此操作。

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

## <a name="retrieve-all-entities-in-a-partition"></a>检索分区中的所有实体
若要查询表以获取分区中的所有实体，请使用 [TableQuery][dotnet_TableQuery] 对象。 以下代码示例指定了一个筛选器，以筛选分区键为“Smith”的实体。 此示例会将查询结果中每个实体的字段输出到控制台。

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

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>检索分区中的一部分实体
如果不想查询分区中的所有实体，则可以通过结合使用分区键筛选器与行键筛选器来指定一个范围。 以下代码示例使用两个筛选器来获取分区“Smith”中的、行键（名字）以字母“E”前面的字母开头的所有实体，然后输出查询结果。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
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

## <a name="retrieve-a-single-entity"></a>检索单个实体
你可以编写查询以检索单个特定实体。 以下代码使用 [TableOperation][dotnet_TableOperation] 来指定客户“Ben Smith”。 此方法仅返回一个实体，而不是一个集合，并且 [TableResult][dotnet_TableResult].[Result][dotnet_TableResult_Result] 中的返回值是一个 **CustomerEntity** 对象。 在查询中指定分区键和行键是从表服务中检索单个实体的最快方法。

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
    {
        Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
    }
    else
    {
        Console.WriteLine("The phone number could not be retrieved.");
    }

## <a name="replace-an-entity"></a>替换实体
若要更新实体，请从表服务中检索它，修改实体对象，然后将更改保存回表服务。 以下代码将更改现有客户的电话号码。 此代码使用 [Replace][dotnet_TableOperation_Replace]，而不是调用 [Insert][dotnet_TableOperation_Insert]。 [Replace][dotnet_TableOperation_Replace] 将导致在服务器上完全替换该实体，除非服务器上的该实体自检索到它以后发生更改，在此情况下，该操作将失败。 操作失败将防止你的应用程序无意中覆盖应用程序的其他组件在检索与更新之间所做的更改。 正确处理此失败问题的方法是再次检索实体，进行更改（如果仍有效），然后再次执行 [Replace][dotnet_TableOperation_Replace] 操作。 下一节将为你演示如何重写此行为。

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

        // Create the Replace TableOperation.
        TableOperation updateOperation = TableOperation.Replace(updateEntity);

        // Execute the operation.
        table.Execute(updateOperation);

        Console.WriteLine("Entity updated.");
    }
    else
    {
        Console.WriteLine("Entity could not be retrieved.");
    }

## <a name="insert-or-replace-an-entity"></a>插入或替换实体
如果该实体自从服务器中检索到它以后发生更改，则 [Replace][dotnet_TableOperation_Replace] 操作将失败。 此外，必须首先从服务器中检索该实体，[Replace][dotnet_TableOperation_Replace] 操作才能成功。 但是，有时你不知道服务器上是否存在该实体以及存储在其中的当前值是否无关。 更新操作会将其全部覆盖。 为此，你应使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace] 操作。 如果该实体不存在，此操作将插入它，如果存在，则替换它，而不管上次更新是何时进行的。

在下面的代码示例中，创建了“Fred Jones”的客户实体并将其插入了“people”表中。 接下来，我们使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace] 操作通过相同的分区键 (Jones) 和行键 (Fred) 将实体保存到服务器，此次为 PhoneNumber 属性设置了其他值。 我们使用 [InsertOrReplace][dotnet_TableOperation_InsertOrReplace]，因此其所有属性值被替换。 但是，如果表中部不存在“Fred Jones”实体，则会将其插入。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable object that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Create a customer entity.
    CustomerEntity customer3 = new CustomerEntity("Jones", "Fred");
    customer3.Email = "Fred@contoso.com";
    customer3.PhoneNumber = "425-555-0106";

    // Create the TableOperation object that inserts the customer entity.
    TableOperation insertOperation = TableOperation.Insert(customer3);

    // Execute the operation.
    table.Execute(insertOperation);

    // Create another customer entity with the same partition key and row key.
    // We've already created a 'Fred Jones' entity and saved it to the
    // 'people' table, but here we're specifying a different value for the
    // PhoneNumber property.
    CustomerEntity customer4 = new CustomerEntity("Jones", "Fred");
    customer4.Email = "Fred@contoso.com";
    customer4.PhoneNumber = "425-555-0107";

    // Create the InsertOrReplace TableOperation.
    TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(customer4);

    // Execute the operation. Because a 'Fred Jones' entity already exists in the
    // 'people' table, its property values will be overwritten by those in this
    // CustomerEntity. If 'Fred Jones' didn't already exist, the entity would be
    // added to the table.
    table.Execute(insertOrReplaceOperation);

## <a name="query-a-subset-of-entity-properties"></a>查询一部分实体属性
表查询可以只检索实体中的少数几个属性而不是所有实体属性。 此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。 以下代码中的查询只返回表中实体的电子邮件地址。 这可通过使用 [DynamicTableEntity][dotnet_DynamicTableEntity] 和 [EntityResolver][dotnet_EntityResolver] 的查询来实现。 可以在[“Upsert 和查询投影介绍”博客文章][blog_post_upsert]中更加详细地了解投影。 存储模拟器不支持投影，因此，此代码仅在使用表服务中的帐户时才能运行。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Define the query, and select only the Email property.
    TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

    // Define an entity resolver to work with the entity after retrieval.
    EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue : null;

    foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
    {
        Console.WriteLine(projectedEmail);
    }

## <a name="delete-an-entity"></a>删除实体
在检索实体之后，可使用更新实体的相同演示模式轻松删除该实体。 以下代码检索并删除一个客户实体。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
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
    {
        Console.WriteLine("Could not retrieve the entity.");
    }

## <a name="delete-a-table"></a>删除表
最后，以下代码示例将从存储帐户中删除表。 在删除表之后的一段时间内无法重新创建它。

    // Retrieve the storage account from the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the table client.
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // Create the CloudTable that represents the "people" table.
    CloudTable table = tableClient.GetTableReference("people");

    // Delete the table it if exists.
    table.DeleteIfExists();

## <a name="retrieve-entities-in-pages-asynchronously"></a>以异步方式检索页中的实体
如果你正在读取大量实体，并且想要在检索进行时处理/显示实体，而非等待返回全部实体，则可以通过使用分段查询检索实体。 此示例显示如何通过 Async-Await 模式以页面形式返回结果，这样就不会在等待返回大量结果时阻止操作的执行。 有关在 .NET 中使用 Async-Await 模式的详细信息，请参阅 [使用 Async 和 Await 进行异步编程（C# 和 Visual Basic）](https://msdn.microsoft.com/zh-cn/library/hh191443.aspx)。

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

## <a name="next-steps"></a>后续步骤
现在，你已了解有关表存储的基础知识，请按照下面的链接了解更复杂的存储任务：


* 查看表服务参考文档，了解有关可用 API 的完整详情：
  * [.NET 存储客户端库参考](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
  * [REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
* 了解如何使用 [Azure WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)
* 查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
  * [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/) 来存储非结构化数据。
  * [使用.NET (C#) 连接到 SQL 数据库](/documentation/articles/sql-database-develop-dotnet-simple/)，存储关系数据。

[Download and install the Azure SDK for .NET]: /develop/net/
[Creating an Azure Project in Visual Studio]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx

[blog_post_upsert]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx

[dotnet_api_ref]: https://msdn.microsoft.com/zh-cn/library/azure/mt347887.aspx
[dotnet_CloudTableClient]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.cloudtableclient.aspx
[dotnet_CloudTable]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.cloudtable.aspx
[dotnet_CloudTable_Execute]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.cloudtable.execute.aspx
[dotnet_CloudTable_ExecuteBatch]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.cloudtable.executebatch.aspx
[dotnet_DynamicTableEntity]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.dynamictableentity.aspx
[dotnet_EntityResolver]: https://msdn.microsoft.com/zh-cn/library/jj733144.aspx
[dotnet_TableBatchOperation]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tablebatchoperation.aspx
[dotnet_TableBatchOperation_Insert]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tablebatchoperation.insert.aspx
[dotnet_TableEntity]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableentity.aspx
[dotnet_TableOperation]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableoperation.aspx
[dotnet_TableOperation_Insert]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableoperation.insert.aspx
[dotnet_TableOperation_InsertOrReplace]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableoperation.insertorreplace.aspx
[dotnet_TableOperation_Replace]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableoperation.replace.aspx
[dotnet_TableQuery]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tablequery.aspx
[dotnet_TableResult]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableresult.aspx
[dotnet_TableResult_Result]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tableresult.result.aspx
<!--Update_Description:add sample include;add links references to inline code blocks;update code for insert and delete operation;add anchors to sub titles-->