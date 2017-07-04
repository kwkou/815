<properties
    pageTitle="如何使用表存储 (C++) | Azure"
    description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
    services="storage"
    documentationcenter=".net"
    author="seguler"
    manager="jahogg"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="f191f308-e4b2-4de9-85cb-551b82b1ea7c"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/28/2017"
    wacn.date="04/24/2017"
    ms.author="seguler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="64e9f5024c07c82e242a3fd2678cf5dadbd1d3a4"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-use-table-storage-from-c"></a>如何通过 C++ 使用表存储
[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]


## <a name="overview"></a>概述
本指南将演示如何使用 Azure 表存储服务执行常见方案。 示例采用 C++ 编写，并使用了[适用于 C++ 的 Azure 存储客户端库](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)。 涉及的方案包括**创建和删除表**以及**使用表实体**。

> [AZURE.NOTE]
> 本指南主要面向适用于 C++ 的 Azure 存储空间客户端库 1.0.0 版及更高版本。 推荐版本：存储客户端库 2.2.0（可通过 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](https://github.com/Azure/azure-storage-cpp/) 获得）。
> 
> 

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>创建 C++ 应用程序
在本指南中，将使用可在 C++ 应用程序内运行的存储功能。 为此，需要安装适用于 C++ 的 Azure 存储客户端库，并在 Azure 订阅中创建 Azure 存储帐户。  

若要安装适用于 C++ 的 Azure 存储客户端库，可使用以下方法：

* **Linux：**按照适用于 C++ 的 [Azure 存储客户端库自述文件](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)页中提供的说明进行操作。  
* **Windows：**在 Visual Studio 中，单击“工具”>“NuGet 包管理器”>“程序包管理器控制台”。 在 [NuGet 包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)中，键入以下命令，然后按 Enter。  

     Install-Package wastorage

## <a name="configure-your-application-to-access-table-storage"></a>配置应用程序以访问表存储
将以下 include 语句添加到要在其中使用 Azure 存储 API 访问表的 C++ 文件的顶部：  

    #include <was/storage_account.h>
    #include <was/table.h>

## <a name="set-up-an-azure-storage-connection-string"></a>设置 Azure 存储连接字符串
Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。 运行客户端应用程序时，必须提供以下格式的存储连接字符串。 使用 [Azure 门户](https://portal.azure.cn)中列出的存储帐户的存储帐户名称和存储访问密钥作为 AccountName 和 AccountKey 值。 有关存储帐户和访问密钥的信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。此示例演示如何声明一个静态字段以保存连接字符串：

	// Define the connection string with your values.
	const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key;EndpointSuffix=core.chinacloudapi.cn"));

若要在基于 Windows 的本地计算机中测试应用程序，可以使用随 [Azure SDK](/downloads/) 一起安装的 Azure [存储模拟器](/documentation/articles/storage-use-emulator/)。 存储模拟器是一种用于模拟本地开发计算机上提供的 Azure Blob、队列和表服务的实用程序。 以下示例演示如何声明一个静态字段以将连接字符串保存到你的本地存储模拟器：  

	// Define the connection string with Azure storage emulator.
	const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  

若要启动 Azure 存储模拟器，请单击“开始”按钮或按 Windows 键。开始键入“Azure 存储模拟器”，然后从应用程序列表中选择“Azure 存储模拟器”。

下面的示例假定使用了这两个方法之一来获取存储连接字符串。  

## <a name="retrieve-your-connection-string"></a>检索你的连接字符串
可使用 **cloud_storage_account** 类来表示你的存储帐户信息。 若要从存储连接字符串中检索你的存储帐户信息，你可以使用 parse 方法。

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

接下来，获取对 **cloud_table_client** 类的引用，因为使用它可以获取表存储服务中存储的表和实体的引用对象。 以下代码使用我们在上面检索到的存储帐户对象创建 **cloud_table_client** 对象：  

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

## <a name="create-a-table"></a>创建表
使用 **cloud_table_client** 对象可获得表和实体的引用对象。 以下代码将创建 **cloud_table_client** 对象并使用它创建新表。

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);  

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Retrieve a reference to a table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table if it doesn't exist.
	table.create_if_not_exists();  

## <a name="add-an-entity-to-a-table"></a>将实体添加到表
若要将实体添加到表，请创建一个新的 **table_entity** 对象并将其传递到 **table_operation::insert_entity**。 以下代码使用客户的名字作为行键，并使用姓氏作为分区键。 条目的分区键和行键共同唯一地标识表中的条目。 查询分区键相同的条目的速度快于查询分区键不同的条目的速度，但使用不同的分区键可实现更高的并行操作可伸缩性。 有关详细信息，请参阅 [Azure 存储性能和可伸缩性核对清单](/documentation/articles/storage-performance-checklist/)。

下面的代码创建一个 **table_entity** 新实例，其中包含要进行存储的部分客户数据。 接下来，该代码调用 **table_operation::insert_entity** 来创建一个 **table_operation** 对象，以便将实体插入表中，并将新的表实体与之关联。 最后，该代码调用 **cloud_table** 对象的 execute 方法。 并且新的 **table_operation** 向表服务发送请求，以此将新的客户实体插入“people”表中。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Retrieve a reference to a table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table if it doesn't exist.
	table.create_if_not_exists();

	// Create a new customer entity.
	azure::storage::table_entity customer1(U("Harp"), U("Walter"));

	azure::storage::table_entity::properties_type& properties = customer1.properties();
	properties.reserve(2);
	properties[U("Email")] = azure::storage::entity_property(U("Walter@contoso.com"));

	properties[U("Phone")] = azure::storage::entity_property(U("425-555-0101"));

	// Create the table operation that inserts the customer entity.
	azure::storage::table_operation insert_operation = azure::storage::table_operation::insert_entity(customer1);

	// Execute the insert operation.
	azure::storage::table_result insert_result = table.execute(insert_operation);

## <a name="insert-a-batch-of-entities"></a>插入一批实体
你可以通过一次写入操作将一批实体插入到表服务。 以下代码创建一个 **table_batch_operation** 对象，然后向其中添加三个插入操作。 每个插入操作的添加方法如下：创建一个新的实体对象，对其设置值，然后对 **table_batch_operation** 对象调用 insert 方法来将实体与新的插入操作相关联。 然后调用 **cloud_table.execute** 来执行此操作。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Define a batch operation.
	azure::storage::table_batch_operation batch_operation;

	// Create a customer entity and add it to the table.
	azure::storage::table_entity customer1(U("Smith"), U("Jeff"));

	azure::storage::table_entity::properties_type& properties1 = customer1.properties();
	properties1.reserve(2);
	properties1[U("Email")] = azure::storage::entity_property(U("Jeff@contoso.com"));
	properties1[U("Phone")] = azure::storage::entity_property(U("425-555-0104"));

	// Create another customer entity and add it to the table.
	azure::storage::table_entity customer2(U("Smith"), U("Ben"));

	azure::storage::table_entity::properties_type& properties2 = customer2.properties();
	properties2.reserve(2);
	properties2[U("Email")] = azure::storage::entity_property(U("Ben@contoso.com"));
	properties2[U("Phone")] = azure::storage::entity_property(U("425-555-0102"));

	// Create a third customer entity to add to the table.
	azure::storage::table_entity customer3(U("Smith"), U("Denise"));

	azure::storage::table_entity::properties_type& properties3 = customer3.properties();
	properties3.reserve(2);
	properties3[U("Email")] = azure::storage::entity_property(U("Denise@contoso.com"));
	properties3[U("Phone")] = azure::storage::entity_property(U("425-555-0103"));

	// Add customer entities to the batch insert operation.
	batch_operation.insert_or_replace_entity(customer1);
	batch_operation.insert_or_replace_entity(customer2);
	batch_operation.insert_or_replace_entity(customer3);

	// Execute the batch operation.
	std::vector<azure::storage::table_result> results = table.execute_batch(batch_operation);

批处理操作的注意事项如下：

* 在单次批处理操作中最多可以执行 100 个插入、删除、合并、替换、插入或合并以及插入或替换操作（可以是这些操作的任意组合）。  
* 批处理操作也可以包含检索操作，但前提是检索操作是批处理中仅有的操作。  
* 单次批处理操作中的所有条目都必须具有相同的分区键。  
* 批处理操作的数据负载限制为 4MB。  

## <a name="retrieve-all-entities-in-a-partition"></a>检索分区中的所有实体
若要查询表以获取分区中的所有实体，请使用 **table_query** 对象。 以下代码示例指定了一个筛选器，以筛选分区键为“Smith”的实体。 此示例会将查询结果中每个条目的字段输出到控制台。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Construct the query operation for all customer entities where PartitionKey="Smith".
	azure::storage::table_query query;

	query.set_filter_string(azure::storage::table_query::generate_filter_condition(U("PartitionKey"), azure::storage::query_comparison_operator::equal, U("Smith")));

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Print the fields for each customer.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		const azure::storage::table_entity::properties_type& properties = it->properties();

		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
			<< U(", Property1: ") << properties.at(U("Email")).string_value()
			<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
	}  

此示例中的查询将检索出与筛选条件匹配的所有条目。 如果有大型表并需要经常下载表条目，建议改为将数据存储在 Azure 存储 Blob 中。

## <a name="retrieve-a-range-of-entities-in-a-partition"></a>检索分区中的一部分条目
如果不想查询分区中的所有条目，则可以通过结合使用分区键筛选器与行键筛选器来指定一个范围。 以下代码示例使用两个筛选器来获取分区“Smith”中的、行键（名字）以字母“E”前面的字母开头的所有条目，然后输出查询结果。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table query.
	azure::storage::table_query query;

	query.set_filter_string(azure::storage::table_query::combine_filter_conditions(
		azure::storage::table_query::generate_filter_condition(U("PartitionKey"),
		azure::storage::query_comparison_operator::equal, U("Smith")),
		azure::storage::query_logical_operator::op_and,
		azure::storage::table_query::generate_filter_condition(U("RowKey"), azure::storage::query_comparison_operator::less_than, U("E"))));

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Loop through the results, displaying information about the entity.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		const azure::storage::table_entity::properties_type& properties = it->properties();

		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
			<< U(", Property1: ") << properties.at(U("Email")).string_value()
			<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
	}  

## <a name="retrieve-a-single-entity"></a>检索单个条目
你可以编写查询以检索单个特定实体。 以下代码使用 **table_operation::retrieve_entity** 来指定客户“Jeff Smith”。 此方法只返回一个实体，而不是一个集合，并且返回的值在 **table_result** 中。 在查询中指定分区键和行键是从表服务中检索单个实体的最快方法。  

	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Output the entity.
	azure::storage::table_entity entity = retrieve_result.entity();
	const azure::storage::table_entity::properties_type& properties = entity.properties();

	std::wcout << U("PartitionKey: ") << entity.partition_key() << U(", RowKey: ") << entity.row_key()
		<< U(", Property1: ") << properties.at(U("Email")).string_value()
		<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;

## <a name="replace-an-entity"></a>替换条目
若要替换条目，请从表服务中检索它，修改条目对象，然后将更改保存回表服务。 以下代码更改现有客户的电话号码和电子邮件地址。 此代码不是调用 **table_operation::insert_entity**，而是使用 **table_operation::replace_entity**。 这将导致在服务器上完全替换该实体，除非服务器上的该实体自检索到它以后发生更改，在此情况下，该操作将失败。 操作失败将防止你的应用程序无意中覆盖应用程序的其他组件在检索与更新之间所做的更改。 正确处理此失败的方法是再次检索实体，进行更改（如果仍有效），然后执行另一个 **table_operation::replace_entity** 操作。 下一节将为你演示如何重写此行为。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Replace an entity.
	azure::storage::table_entity entity_to_replace(U("Smith"), U("Jeff"));
	azure::storage::table_entity::properties_type& properties_to_replace = entity_to_replace.properties();
	properties_to_replace.reserve(2);

	// Specify a new phone number.
	properties_to_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0106"));

	// Specify a new email address.
	properties_to_replace[U("Email")] = azure::storage::entity_property(U("JeffS@contoso.com"));

	// Create an operation to replace the entity.
	azure::storage::table_operation replace_operation = azure::storage::table_operation::replace_entity(entity_to_replace);

	// Submit the operation to the Table service.
	azure::storage::table_result replace_result = table.execute(replace_operation);

## <a name="insert-or-replace-an-entity"></a>插入或替换实体
如果该实体自从服务器中检索到它以后发生更改，则 **table_operation::replace_entity** 操作将失败。 此外，必须首先从服务器中检索该实体，**table_operation::replace_entity** 才会成功。 但是，有时你不知道服务器上是否存在该实体以及存储在其中的当前值是否无关 - 更新操作应将其全部覆盖。 为此，应使用 **table_operation::insert_or_replace_entity** 操作。 如果该实体不存在，此操作将插入它，如果存在，则替换它，而不管上次更新是何时进行的。 在以下代码示例中，仍将检索 Jeff Smith 的客户实体，但稍后会通过 **table_operation::insert_or_replace_entity** 将其保存回服务器。 将覆盖在检索与更新操作之间对实体进行的任何更新。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Insert-or-replace an entity.
	azure::storage::table_entity entity_to_insert_or_replace(U("Smith"), U("Jeff"));
	azure::storage::table_entity::properties_type& properties_to_insert_or_replace = entity_to_insert_or_replace.properties();

	properties_to_insert_or_replace.reserve(2);

	// Specify a phone number.
	properties_to_insert_or_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0107"));

	// Specify an email address.
	properties_to_insert_or_replace[U("Email")] = azure::storage::entity_property(U("Jeffsm@contoso.com"));

	// Create an operation to insert-or-replace the entity.
	azure::storage::table_operation insert_or_replace_operation = azure::storage::table_operation::insert_or_replace_entity(entity_to_insert_or_replace);

	// Submit the operation to the Table service.
	azure::storage::table_result insert_or_replace_result = table.execute(insert_or_replace_operation);

## <a name="query-a-subset-of-entity-properties"></a>查询条目属性的子集
对表的查询可以只检索实体中的少数几个属性。 以下代码中的查询使用 **table_query::set_select_columns** 方法，仅返回表中实体的电子邮件地址。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Define the query, and select only the Email property.
	azure::storage::table_query query;
	std::vector<utility::string_t> columns;

	columns.push_back(U("Email"));
	query.set_select_columns(columns);

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Display the results.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key();

		const azure::storage::table_entity::properties_type& properties = it->properties();
		for (auto prop_it = properties.begin(); prop_it != properties.end(); ++prop_it)
		{
			std::wcout << ", " << prop_it->first << ": " << prop_it->second.str();
		}

		std::wcout << std::endl;
	}

> [AZURE.NOTE]
> 查询实体的几个属性是比检索所有属性更高效的操作。
> 
> 

## <a name="delete-an-entity"></a>删除条目
可以在检索到实体后轻松将其删除。 检索到实体后，对要删除的实体调用 **table_operation::delete_entity**。 然后调用 **cloud_table.execute** 方法。 以下代码检索并删除分区键为“Smith”、行键为“Jeff”的实体。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Create an operation to delete the entity.
	azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

	// Submit the delete operation to the Table service.
	azure::storage::table_result delete_result = table.execute(delete_operation);  

## <a name="delete-a-table"></a>删除表
最后，以下代码示例将从存储帐户中删除表。 在删除表之后的一段时间内无法重新创建它。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Create an operation to delete the entity.
	azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

	// Submit the delete operation to the Table service.
	azure::storage::table_result delete_result = table.execute(delete_operation);

## <a name="next-steps"></a>后续步骤
现在，你已了解表存储的基础知识，请打开以下链接了解有关 Azure 存储空间的详细信息：  

* [如何通过 C++ 使用 Blob 存储](/documentation/articles/storage-c-plus-plus-how-to-use-blobs/)
* [如何通过 C++ 使用队列存储](/documentation/articles/storage-c-plus-plus-how-to-use-queues/)
* [使用 C++ 列出 Azure 存储资源](/documentation/articles/storage-c-plus-plus-enumeration/)
* [适用于 C++ 的存储客户端库参考](http://azure.github.io/azure-storage-cpp)
* [Azure 存储文档](/documentation/services/storage/)
<!--Update_Description: wording update; add anchors to H2 titles-->