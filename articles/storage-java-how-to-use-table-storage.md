<properties 
	pageTitle="如何通过 Java 使用表存储 | Azure" 
	description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
	services="storage" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="storage" 
	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>


# 如何通过 Java 使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述

本指南将演示如何使用 Azure 表存储服务执行常见方案。这些示例用 Java 编写并使用 [Azure Storage SDK for Java][]。涉及的方案包括“创建”、“列出”和“删除”表，以及“插入”、“查询”、“修改”和“删除”表中的实体。有关表的详细信息，请参阅[后续步骤](#Next-Steps)部分。

注意：为在 Android 设备上使用 Azure 存储的开发人员提供了 SDK。有关详细信息，请参阅 [Azure Storage SDK for Android][]。

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Java 应用程序

在本指南中，你将使用存储功能，这些功能可在本地 Java 应用程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码来运行。

为此，你将需要安装 Java 开发工具包 (JDK)，并在你的 Azure 订阅中创建一个 Azure 存储帐户。完成此操作后，你将需要验证开发系统满足最低要求和 GitHub 上的 [Azure Storage SDK for Java][] 存储库中列出的依赖项。如果你的系统满足这些要求，你可以按照说明下载和安装系统中该存储库的 Azure Storage Libraries for Java。完成这些任务后，您将能够创建一个 Java 应用程序，以便使用本文中的示例。

## 配置应用程序以访问表存储

将以下导入语句添加到要在其中使用 Azure 存储 API 访问表的 Java 文件的顶部：

    // Include the following imports to use table APIs
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.table.*;
    import com.microsoft.azure.storage.table.TableQuery.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并对 AccountName 和 AccountKey 值使用管理门户中列出的存储帐户的名称和存储帐户的主访问密钥。此示例演示如何声明一个静态字段以保存连接字符串：

    // Define the connection-string with your values.
    public static final String storageConnectionString = 
        "DefaultEndpointsProtocol=http;" + 
        "AccountName=your_storage_account;" + 
        "AccountKey=your_storage_account_key;" +
	"EndpointSuffix=core.chinacloudapi.cn";

在 Azure 的角色中运行的应用程序中，此字符串可存储在服务配置文件 *ServiceConfiguration.cscfg* 中，并可通过调用 **RoleEnvironment.getConfigurationSettings** 方法进行访问。下面是从服务配置文件中名为 *StorageConnectionString* 的 **Setting** 元素中获取连接字符串的示例：

    // Retrieve storage account from connection-string.
    String storageConnectionString = 
        RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

下面的示例假定你使用了这两个方法之一来获取存储连接字符串。

## 如何：创建表

利用 **CloudTableClient** 对象，您可以获得表和实体的引用对象。以下代码将创建 **CloudTableClient** 对象并使用它创建新的 **CloudTable** 对象，用于表示名为“people”的表。（注意：还有其他方式来创建 **CloudStorageAccount** 对象；有关详细信息，请参阅 [Azure 存储客户端 SDK 参考]中的 **CloudStorageAccount**。）

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

	   // Create the table client.
	   CloudTableClient tableClient = storageAccount.createCloudTableClient();

	   // Create the table if it doesn't exist.
	   String tableName = "people";
	   CloudTable cloudTable = tableClient.getTableReference(tableName);
	   cloudTable.createIfNotExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：列出表

若要获取表的列表，请调用 **CloudTableClient.listTables()** 方法来检索表名称的迭代列表。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();

    	// Loop through the collection of table names.
    	for (String table : tableClient.listTables())
    	{
		  // Output each table name.
		  System.out.println(table);
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：向表中添加实体

实体将映射到使用实现了 **TableEntity** 的自定义类的 Java 对象。为方便起见，**TableServiceEntity** 类实现 **TableEntity**，并使用反射将属性映射到以它们本身命名的 getter 和 setter 方法。若要将实体添加到表，首先要创建用于定义实体的属性的类。以下代码定义了将客户的名字和姓氏分别用作行键和分区键的实体类。实体的分区键和行键共同唯一地标识表中的实体。查询分区键相同的实体的速度可以快于查询分区键不同的实体的速度。

    public class CustomerEntity extends TableServiceEntity {
        public CustomerEntity(String lastName, String firstName) {
            this.partitionKey = lastName;
            this.rowKey = firstName;
        }

        public CustomerEntity() { }

        String email;
        String phoneNumber;
        
        public String getEmail() {
            return this.email;
        }
        
        public void setEmail(String email) {
            this.email = email;
        }
        
        public String getPhoneNumber() {
            return this.phoneNumber;
        }
        
        public void setPhoneNumber(String phoneNumber) {
            this.phoneNumber = phoneNumber;
        }
    }

涉及实体的表操作需要 **TableOperation** 对象。此对象用于定义要对实体执行的操作，该操作可使用 **CloudTable** 对象执行。以下代码创建了包含要存储的某些客户数据的 **CustomerEntity** 类的新实例。接下来，该代码调用 **TableOperation.insertOrReplace** 创建一个 **TableOperation** 对象，以便将实体插入表中，并将新的 **CustomerEntity** 与之关联。最后，该代码对 **CloudTable** 对象调用 **execute** 方法，并指定了“people”表和新的 **TableOperation**，后者随后向存储服务发送将新客户实体插入“people”表或在实体已存在的情况下替换实体的请求。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();
			
    	// Create a cloud table object for the table.
    	CloudTable cloudTable = tableClient.getTableReference("people");
			
    	// Create a new customer entity.
    	CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    	customer1.setEmail("Walter@contoso.com");
    	customer1.setPhoneNumber("425-555-0101");
			
    	// Create an operation to add the new customer to the people table.
    	TableOperation insertCustomer1 = TableOperation.insertOrReplace(customer1);

    	// Submit the operation to the table service.
    	cloudTable.execute(insertCustomer1);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：插入一批实体

你可以通过一次写入操作将一批实体插入到表服务。以下代码创建一个 **TableBatchOperation** 对象，然后向其中添加三个插入操作。每个插入操作的添加方法如下：创建一个新的实体对象，设置它的值，然后对 **TableBatchOperation** 对象调用 **insert** 方法以将实体与新的插入操作相关联。然后，该代码对 **CloudTable** 调用 **execute**，并指定“people”表和 **TableBatchOperation** 对象，后者将在一个请求中向存储服务发送一批表操作。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();

    	// Define a batch operation.
    	TableBatchOperation batchOperation = new TableBatchOperation();

    	// Create a cloud table object for the table.
    	CloudTable cloudTable = tableClient.getTableReference("people");

    	// Create a customer entity to add to the table.
    	CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    	customer.setEmail("Jeff@contoso.com");
    	customer.setPhoneNumber("425-555-0104");
    	batchOperation.insertOrReplace(customer);

	   // Create another customer entity to add to the table.
	   CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
	   customer2.setEmail("Ben@contoso.com");
	   customer2.setPhoneNumber("425-555-0102");
	   batchOperation.insertOrReplace(customer2);

	   // Create a third customer entity to add to the table.
	   CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
	   customer3.setEmail("Denise@contoso.com");
	   customer3.setPhoneNumber("425-555-0103");
	   batchOperation.insertOrReplace(customer3);

	   // Execute the batch of operations on the "people" table.
	   cloudTable.execute(batchOperation);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

批处理操作的注意事项如下：

- 您在单次批处理操作中最多可以执行 100 个插入、删除、合并、替换、插入或合并以及插入或替换操作（可以是这些操作的任意组合）。
- 批处理操作也可以包含检索操作，但前提是检索操作是批处理中仅有的操作。
- 单次批处理操作中的所有实体都必须具有相同的分区键。
- 批处理操作的数据负载限制为 4MB。

## 如何：检索分区中的所有实体

若要从表中查询分区中的实体，可以使用 **TableQuery**。调用 **TableQuery.from** 可创建一个针对特定表的查询，该查询将返回指定的结果类型。以下代码指定了一个筛选器，用于筛选其中的分区键是“Smith”的实体。**TableQuery.generateFilterCondition** 是一个用于创建查询筛选器的帮助器方法。对 **TableQuery.from** 方法返回的引用调用 **where** 可对查询应用筛选器。当通过对 **CloudTable** 对象调用 **execute** 来执行查询时，该查询将返回指定了 **CustomerEntity** 结果类型的 **Iterator**。然后，你可以利用在 for each 循环中返回的 **Iterator** 来使用结果。此代码会将查询结果中每个实体的字段打印到控制台。

    try
    {
    	// Define constants for filters.
    	final String PARTITION_KEY = "PartitionKey";
    	final String ROW_KEY = "RowKey";
    	final String TIMESTAMP = "Timestamp";

    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();
			
	   // Create a cloud table object for the table.
	   CloudTable cloudTable = tableClient.getTableReference("people");

    	// Create a filter condition where the partition key is "Smith".
    	String partitionFilter = TableQuery.generateFilterCondition(
	       PARTITION_KEY, 
	       QueryComparisons.EQUAL,
	       "Smith");

	   // Specify a partition query, using "Smith" as the partition key filter.
	   TableQuery<CustomerEntity> partitionQuery =
	       TableQuery.from(CustomerEntity.class)
	       .where(partitionFilter);

        // Loop through the results, displaying information about the entity.
        for (CustomerEntity entity : cloudTable.execute(partitionQuery)) {
            System.out.println(entity.getPartitionKey() +
                " " + entity.getRowKey() + 
                "\t" + entity.getEmail() +
                "\t" + entity.getPhoneNumber());
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：检索分区中的一部分实体

如果不想查询分区中的所有实体，则可以在筛选器中使用比较运算符来指定一个范围。以下代码组合了两个筛选器来获取分区“Smith”中的、行键（名字）以字母“E”及字母“E”前面的字母开头的所有实体。然后，该代码打印了查询结果。如果使用添加到本指南的批量插入部分的表的实体，则此次只返回两个实体（Ben 和 Denise Smith），而不会包括 Jeff Smith。

    try
    {
    	// Define constants for filters.
    	final String PARTITION_KEY = "PartitionKey";
    	final String ROW_KEY = "RowKey";
    	final String TIMESTAMP = "Timestamp";
			
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

	   // Create the table client.
	   CloudTableClient tableClient = storageAccount.createCloudTableClient();

	   // Create a cloud table object for the table.
	   CloudTable cloudTable = tableClient.getTableReference("people");

    	// Create a filter condition where the partition key is "Smith".
    	String partitionFilter = TableQuery.generateFilterCondition(
	       PARTITION_KEY, 
	       QueryComparisons.EQUAL,
	       "Smith");

    	// Create a filter condition where the row key is less than the letter "E".
    	String rowFilter = TableQuery.generateFilterCondition(
	       ROW_KEY, 
	       QueryComparisons.LESS_THAN,
	       "E");

    	// Combine the two conditions into a filter expression.
    	String combinedFilter = TableQuery.combineFilters(partitionFilter, 
	        Operators.AND, rowFilter);

    	// Specify a range query, using "Smith" as the partition key,
    	// with the row key being up to the letter "E".
    	TableQuery<CustomerEntity> rangeQuery =
	       TableQuery.from(CustomerEntity.class)
	       .where(combinedFilter);

    	// Loop through the results, displaying information about the entity
        for (CustomerEntity entity : cloudTable.execute(rangeQuery)) {
            System.out.println(entity.getPartitionKey() +
                " " + entity.getRowKey() +
                "\t" + entity.getEmail() +
                "\t" + entity.getPhoneNumber());
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：检索单个实体

你可以编写查询以检索单个特定实体。以下代码使用分区键和行键参数调用 **TableOperation.retrieve** 来指定客户“Jeff Smith”，而不是创建 **TableQuery** 并使用筛选器来执行同一操作。执行的检索操作将只返回一个实体，而不会返回一个集合。**getResultAsType** 方法会将结果强制转换为分配目标的类型 - **CustomerEntity** 对象。如果此类型与为查询指定的类型不兼容，则会引发异常。如果没有实体具有完全匹配的分区键和行键，则会返回 null 值。在查询中指定分区键和行键是从表服务中检索单个实体的最快方法。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();

    	// Create a cloud table object for the table.
    	CloudTable cloudTable = tableClient.getTableReference("people");

    	// Retrieve the entity with partition key of "Smith" and row key of "Jeff"
    	TableOperation retrieveSmithJeff = 
	       TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

	   // Submit the operation to the table service and get the specific entity.
	   CustomerEntity specificEntity =
    		cloudTable.execute(retrieveSmithJeff).getResultAsType();
			
    	// Output the entity.
    	if (specificEntity != null)
    	{
            System.out.println(specificEntity.getPartitionKey() +
                " " + specificEntity.getRowKey() +
                "\t" + specificEntity.getEmail() +
                "\t" + specificEntity.getPhoneNumber());
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：修改实体

若要修改实体，请从表服务中检索它，对实体对象进行更改，然后通过替换或合并操作将更改保存回表服务。以下代码将更改现有客户的电话号码。此代码将调用 **TableOperation.replace**，而不是像执行插入时那样调用 **TableOperation.insert**。**CloudTable.execute** 方法将调用表服务，并替换该实体，除非在此应用程序检索到该实体之后另一个应用程序对它进行了更改。如果出现这种情况，则会引发异常，必须再次检索、修改并保存该实体。此乐观并发重试模式在分布式存储系统中很常见。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount =
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();

    	// Create a cloud table object for the table.
    	CloudTable cloudTable = tableClient.getTableReference("people");

    	// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
    	TableOperation retrieveSmithJeff = 
	       TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    	// Submit the operation to the table service and get the specific entity.
    	CustomerEntity specificEntity =
		  cloudTable.execute(retrieveSmithJeff).getResultAsType();

    	// Specify a new phone number.
    	specificEntity.setPhoneNumber("425-555-0105");

    	// Create an operation to replace the entity.
    	TableOperation replaceEntity = TableOperation.replace(specificEntity);

    	// Submit the operation to the table service.
    	cloudTable.execute(replaceEntity);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：查询实体属性子集

对表的查询可以只检索实体中的少数几个属性。此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。以下代码中的查询使用 **select** 方法，仅返回表中实体的电子邮件地址。返回结果在 **EntityResolver**（用于对从服务器返回的实体执行类型转换）的帮助下投影到一个 **String** 集合中。你可以在[ Azure 表：Upsert 和查询投影简介][]中更加详细地了解投影。请注意，本地存储模拟器不支持投影，因此，此代码仅在使用表服务中的帐户时才能运行。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount =
            CloudStorageAccount.parse(storageConnectionString);

    	// Create the table client.
    	CloudTableClient tableClient = storageAccount.createCloudTableClient();

    	// Create a cloud table object for the table.
    	CloudTable cloudTable = tableClient.getTableReference("people");

    	// Define a projection query that retrieves only the Email property
    	TableQuery<CustomerEntity> projectionQuery = 
	       TableQuery.from(CustomerEntity.class)
	       .select(new String[] {"Email"});

    	// Define a Entity resolver to project the entity to the Email value.
    	EntityResolver<String> emailResolver = new EntityResolver<String>() {
            @Override
            public String resolve(String PartitionKey, String RowKey, Date timeStamp, HashMap<String, EntityProperty> properties, String etag) {
                return properties.get("Email").getValueAsString();
            }
        };

        // Loop through the results, displaying the Email values.
        for (String projectedString : 
            cloudTable.execute(projectionQuery, emailResolver)) {
                System.out.println(projectedString);
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：插入或替换实体

你经常需要将某个实体添加到表中，但又不知道该实体是否已存在于表中。利用插入或替换操作，您可以提出一个以下形式的请求：如果实体不存在，则插入一个实体；如果实体存在，则将其替换为现有实体。以下代码基于前面的示例针对“Walter Harp”插入或替换实体。创建新实体后，此代码调用 **TableOperation.insertOrReplace** 方法。此代码随后使用表和插入或替换表操作作为参数对 **CloudTable** 对象调用 **execute**。若要只更新实体的一部分，则可以改用 **TableOperation.insertOrMerge** 方法。请注意，本地存储仿真程序不支持插入或替换，因此，此代码仅在使用表服务中的帐户时才能运行。你可以在此[ Azure 表：Upsert 和查询投影简介][]中更加详细地了插入或替换和插入或合并。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount =
            CloudStorageAccount.parse(storageConnectionString);

        // Create the table client.
        CloudTableClient tableClient = storageAccount.createCloudTableClient();

        // Create a cloud table object for the table.
        CloudTable cloudTable = tableClient.getTableReference("people");

        // Create a new customer entity.
        CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
        customer5.setEmail("Walter@contoso.com");
        customer5.setPhoneNumber("425-555-0106");

        // Create an operation to add the new customer to the people table.
        TableOperation insertCustomer5 = TableOperation.insertOrReplace(customer5);

        // Submit the operation to the table service.
        cloudTable.execute(insertCustomer5);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：删除实体

你可以在检索到实体后轻松将其删除。检索到实体后，对要删除的实体调用 **TableOperation.delete**。然后对 **CloudTable** 对象调用 **execute**。以下代码检索并删除一个客户实体。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount =
            CloudStorageAccount.parse(storageConnectionString);

        // Create the table client.
        CloudTableClient tableClient = storageAccount.createCloudTableClient();

        // Create a cloud table object for the table.
        CloudTable cloudTable = tableClient.getTableReference("people");

        // Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
        TableOperation retrieveSmithJeff = TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

        // Retrieve the entity with partition key of "Smith" and row key of "Jeff".
        CustomerEntity entitySmithJeff =
            cloudTable.execute(retrieveSmithJeff).getResultAsType();

        // Create an operation to delete the entity.
        TableOperation deleteSmithJeff = TableOperation.delete(entitySmithJeff);

        // Submit the delete operation to the table service.
        cloudTable.execute(deleteSmithJeff);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：删除表

最后，以下代码将从存储帐户中删除一个表。表在删除之后的一小段时间（通常小于四十秒）内无法重新创建。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount =
            CloudStorageAccount.parse(storageConnectionString);

        // Create the table client.
        CloudTableClient tableClient = storageAccount.createCloudTableClient();

        // Delete the table and all its data if it exists.
        CloudTable cloudTable = new CloudTable("people",tableClient);
        cloudTable.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

- [Azure Storage SDK for Java][]
- [Azure 存储客户端 SDK 参考][]
- [Azure 存储 REST API][]
- [Azure 存储团队博客][]

有关详细信息，另请参阅 [Java 开发人员中心](/develop/java/)。


[Azure SDK for Java]: /develop/java/
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Azure 存储客户端 SDK 参考]: http://azure.github.io/azure-storage-java/
[Azure 存储 REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx
[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[ Azure 表：Upsert 和查询投影简介]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx

<!---HONumber=Mooncake_0530_2016-->