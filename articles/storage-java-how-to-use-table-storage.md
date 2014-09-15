<properties linkid="dev-java-how-to-use-table-storage" urlDisplayName="Table Service" pageTitle="How to use table storage (Java) | Microsoft Azure" metaKeywords="Azure table storage service, Azure table service Java, table storage Java" description="Learn how to use the table storage service in Azure. Code samples are written in Java code." metaCanonical="" services="storage" documentationCenter="Java" title="How to use the Table storage service from Java" authors="" solutions="" manager="" editor="" />

# 如何通过 Java 使用表存储服务

本指南将演示如何使用 Azure 表存储服务执行常见方案。
示例是用 Java 代码编写的。
涉及的方案包括**创建和删除表、在表中插入和查询实体**
。有关表的详细信息，请参阅
[后续步骤][]部分。

## 目录

-   [什么是表存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Java 应用程序][]
-   [创建用于访问表存储的应用程序][]
-   [设置 Azure 存储连接字符串][]
-   [如何：创建表][]
-   [如何：将实体添加到表][]
-   [如何：插入一批实体][]
-   [如何：检索分区中的所有实体][]
-   [如何：检索分区中的一部分实体][]
-   [如何：检索单个实体][]
-   [如何：修改实体][]
-   [如何：查询一部分实体属性][]
-   [如何：插入或替换实体][]
-   [如何：删除实体][]
-   [如何：删除表][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-table-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Java 应用程序

在本指南中，你将使用存储功能，这些功能可在本地 Java 应用
程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码
来运行。我们假定你已下载并安装了 Java 开发工具包 (JDK)，已按照
[Azure SDK for Java][] 中的说明安装了
Azure Libraries for Java 和 Azure SDK，并已在你的 Azure 订阅中
创建了一个 Azure 存储帐户。

你可以使用任何开发工具（包括“记事本”）创建应用程序。
你只要能够编译 Java 项目并引用 Azure Libraries
for Java 即可。

## 配置应用程序以访问表存储

将下列 import 语句添加到需要在其中使用 Azure 存储 API 来
访问表的 Java 文件的顶部：

    // 包括下列 import 语句以使用表 API
    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.table.client.*;
    import com.microsoft.windowsazure.services.table.client.TableQuery.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务
的终结点和凭据。在客户端应用程序中运行时，
必须提供以下格式的存储连接字符串，
并对**“AccountName”和**“AccountKey”值使用
管理门户中列出的存储帐户的名称
和存储帐户的主访问密钥。此示例演示了如何
声明一个静态字段来保存连接字符串：

    // 使用你的值定义连接字符串
    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_storage_account;" + 
    "AccountKey=your_storage_account_key";

在 Azure 中的某个角色内运行的应用程序中，此字符串可
存储在服务配置文件 ServiceConfiguration.cscfg 中，
并可通过调用 **RoleEnvironment.getConfigurationSettings**
方法进行访问。下面是从服务配置文件中
名为 StorageConnectionString 的**设置**
元素中获取连接字符串的示例**：

    // 通过连接字符串检索存储帐户
    String storageConnectionString = 
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

下面的示例假定你使用了这两个定义之一来获取存储连接字符串。

## 如何：创建表

利用 **CloudTableClient** 对象，你可以获得表和实体的引用对象。
以下代码将创建 **CloudTableClient** 对象
并使用它创建新表。本指南中的所有代码
都使用存储在 Azure 应用程序的服务配置中的
存储连接字符串。还有其他方法可用来
创建 **CloudStorageAccount** 对象。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 如果该表不存在，则创建它。
    String tableName = "people";
    tableClient.createTableIfNotExists(tableName);

## 如何：将实体添加到表

实体将映射到使用实现了 **TableEntity** 的
自定义类的 Java 对象。为方便起见，**TableServiceEntity** 类实现了
**TableEntity**，并使用反射将属性映射到为
属性指定的 getter 和 setter 方法。若要将实体添加到表，
请先创建用于定义实体的属性的类。
以下代码定义了将客户的名字和姓氏
分别用作行键和分区键的实体类。
实体的分区键和行键共同唯一地标识表中的实体。
查询分区键相同的实体的速度快于查询分区键
不同的实体的速度。

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

涉及实体的表操作需要使用 **TableOperation** 对象。此对象
用于定义要对实体执行的操作，该操作可使
用 **CloudTableClient** 对象执行。以下代码
创建一个包含要存储的某些客户数据的 **CustomerEntity** 类
的新实例。接下来，该代码调用 **TableOperation.insert**
创建一个 **TableOperation** 对象，以便将实体
插入表中，并将新的 **CustomerEntity** 与之
关联。最后，该代码对 **CloudTableClient** 调用
**execute** 方法，并指定了“people”表和新的
**TableOperation**，后者随后向存储服务发送将新客户实体
插入“people”表的请求。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 创建新客户实体。
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.setEmail("Walter@contoso.com");
    customer1.setPhoneNumber("425-555-0101");

    // 创建一个操作来将新客户添加到 people 表。
    TableOperation insertCustomer1 = TableOperation.insert(customer1);

    // 将操作提交到表服务。
    tableClient.execute("people", insertCustomer1);

## 如何：插入一批实体

你可以通过一个写入操作将一批实体插入到表服务。
以下代码创建一个 **TableBatchOperation** 对象，
然后向其中添加三个插入操作。每个插入操作的添加方法如下：
创建一个新的实体对象，设置它的值，然后
对 **TableBatchOperation** 对象调用 **insert** 方法
以将实体与新的插入操作相关联。然后，该代码对 **CloudTableClient** 调
用 **execute**，并指定“people”表和 **TableBatchOperation** 对象，
后者将在一个请求中向存储服务发送一批表操作。
下面是批处理操作的一些注意事项：

1.  你在单个批处理操作中最多可以执行 100 个插入、删除、合并、
    替换、插入或合并以及插入或替换操作（可以是这些操作的
    任意组合）。
2.  批处理操作可以包含检索操作，但前提是检索操作是批处理中的
    唯一操作。
3.  单个批处理操作中的所有实体都必须具有相同的
    分区键。
4.  批处理操作的数据负载限制为 4MB。

<!-- -->

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 定义批处理操作。
    TableBatchOperation batchOperation = new TableBatchOperation();

    // 创建一个要添加到表中的客户实体。
    CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    customer.setEmail("Jeff@contoso.com");
    customer.setPhoneNumber("425-555-0104");
    batchOperation.insert(customer);

    // 创建另一个要添加到表中的客户实体。
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.setEmail("Ben@contoso.com");
    customer2.setPhoneNumber("425-555-0102");
    batchOperation.insert(customer2);

    // 创建第三个要添加到表中的客户实体。
    CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
    customer3.setEmail("Denise@contoso.com");
    customer3.setPhoneNumber("425-555-0103");
    batchOperation.insert(customer3);

    // 对“people”表执行批处理操作。
    tableClient.execute("people", batchOperation);

## 如何：检索分区中的所有实体

若要从表中查询分区中的实体，可以使用 **TableQuery**。
调用 **TableQuery.from** 可创建一个针对特定表
的查询，该查询将返回指定的结果类型。以下代码指定了
一个筛选器，用于筛选其中的分区键是“Smith”的实体。
**TableQuery.generateFilterCondition** 是一个用于创建查询筛选器
的帮助器方法。对 **TableQuery.from** 方法返回的引用调用
**where** 可将对查询应用筛选器。当通过
对 **CloudTableClient** 对象调用 **execute** 来执行查询时，
该查询将返回指定了 **CustomerEntity** 结果类型
的 **Iterator**。然后，你可以利用在 for each 循环中返回
的 **Iterator** 来使用结果。此代码会将查询结果中每个实体的字段
输出到控制台。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 创建一个分区键为“Smith”的筛选条件。
    String partitionFilter = TableQuery.generateFilterCondition(
    TableConstants.PARTITION_KEY, 
    QueryComparisons.EQUAL,
    "Smith");

    // 指定一个分区查询，使用“Smith”作为分区键筛选器。
    TableQuery<CustomerEntity> partitionQuery =
    TableQuery.from("people", CustomerEntity.class)
    .where(partitionFilter);

    // 循环访问结果，显示实体的有关信息。
    for (CustomerEntity entity :tableClient.execute(partitionQuery)) {
    System.out.println(entity.getPartitionKey() + " " + entity.getRowKey() + 
    "\t" + entity.getEmail() + "\t" + entity.getPhoneNumber());
    }

## 如何：检索分区中的一部分实体

如果不想查询分区中的所有实体，则可以在筛选器中使用比较
运算符来指定一个范围。以下代码
组合使用两个筛选器来获取分区“Smith”中行键（名字）以字母表中字母
“E”前面的字母开头的所有实体，
然后输出查询结果。如果你使用在本指南的批量插入部分中
添加到表的实体，则此次只会返回两个实体（Ben 和 Denise Smith），
而不会包括 Jeff Smith。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 创建一个分区键为“Smith”的筛选条件。
    String partitionFilter = TableQuery.generateFilterCondition(
    TableConstants.PARTITION_KEY, 
    QueryComparisons.EQUAL,
    "Smith");

    // 创建一个行键小于字母“E”的筛选条件。
    String rowFilter = TableQuery.generateFilterCondition(
    TableConstants.ROW_KEY, 
    QueryComparisons.LESS_THAN,
    "E");

    // 将两个条件组合到一个筛选器表达式中。
    String combinedFilter = TableQuery.combineFilters(partitionFilter, 
    Operators.AND, rowFilter);

    // 指定一个范围查询，使用“Smith”作为分区键，
    // 使用最大为字母“E”的行键。
    TableQuery<CustomerEntity> rangeQuery =
    TableQuery.from("people", CustomerEntity.class)
    .where(combinedFilter);

    // 循环访问结果，显示实体的相关信息
    for (CustomerEntity entity :tableClient.execute(rangeQuery)) {
    System.out.println(entity.getPartitionKey() + " " + entity.getRowKey() + 
    "\t" + entity.getEmail() + "\t" + entity.getPhoneNumber());
    }

## 如何：检索单个实体

你可以编写查询以检索单个特定实体。以下代码
使用分区键和行键参数调用 **TableOperation.retrieve** 来
指定客户“Jeff Smith”，而不是创建 **TableQuery** 并
使用筛选器来执行同一操作。当执行时，
检索操作将只返回一个实体，而不会返回一个集合。
**getResultAsType** 方法会将结果强制转换为分配目标的
类型 - **CustomerEntity** 对象。如果此类型与
为查询指定的类型不兼容，则会引发异常。
如果没有实体具有完全匹配的分区键和行键，
则会返回 null 值。在查询中同时指定分区键和行键是从表服务中
检索单个实体的最快方法。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 检索分区键为“Smith”且行键为“Jeff”的实体
    TableOperation retrieveSmithJeff = 
    TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // 将操作提交到表服务并获取特定的实体。
    CustomerEntity specificEntity =
    tableClient.execute("people", retrieveSmithJeff).getResultAsType();

## 如何：修改实体

若要修改实体，请从表服务中检索它，对实体对象进行更改，
然后通过替换或合并操作将更改
保存回表服务。以下代码将更改现有客户的
电话号码。此代码将调用 **TableOperation.replace**
，而不是像执行插入时那样调用 **TableOperation.insert**
。
**CloudTableClient.execute** 方法将调用表服务，并替换该实体，
除非在此应用程序检索到该实体之后另一个
应用程序对它进行了更改。如果出现这种情况，
则会引发异常，必须再次检索、修改并保存该实体。
此乐观并发重试模式在分布式存储系统中
很常见。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 检索分区键为“Smith”且行键为“Jeff”的实体。
    TableOperation retrieveSmithJeff = 
    TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // 将操作提交到表服务并获取特定的实体。
    CustomerEntity specificEntity =
    tableClient.execute("people", retrieveSmithJeff).getResultAsType();
        
    // 指定一个新电话号码。
    specificEntity.setPhoneNumber("425-555-0105");

    // 创建一个操作来替换实体。
    TableOperation replaceEntity = TableOperation.replace(specificEntity);

    // 将操作提交到表服务。
    tableClient.execute("people", replaceEntity);

## 如何：查询一部分实体属性

对表的查询可以只检索实体中的少数几个属性。此方法称为投影，
可减少带宽并提高查询性能，尤其适用于大型实体。
以下代码中的查询
使用 **select** 方法，仅返回表中实体的
电子邮件地址。返回结果在 **EntityResolver**
（用于对从服务器返回的实体执行类型转换）的帮助下
投影到一个 **String** 集合中。你
可以在此[博客文章][]中了解有关投影的详细信息。
请注意，本地存储模拟器不支持投影，因此，
此代码仅在使用表服务中的帐户时才能运行。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 定义一个仅检索 Email 属性的投影查询
    TableQuery<CustomerEntity> projectionQuery = 
    TableQuery.from("people", CustomerEntity.class)
    .select(new String[] {"Email"});

    // 定义一个实体解析器以将实体投影到 Email 值。
    EntityResolver<String> emailResolver = new EntityResolver<String>() {
    @Override
    public String resolve(String PartitionKey, String RowKey, Date timeStamp,
    HashMap<String, EntityProperty> properties, String etag) {
    return properties.get("Email").getValueAsString();
        }
    };

    // 循环访问结果，显示 Email 值。
    for (String projectedString : 
    tableClient.execute(projectionQuery, emailResolver)) {
    System.out.println(projectedString);
    }

## 如何：插入或替换实体

很多时候，你需要将某个实体添加到表中，但又不知道该实体是否
已存在于表中。利用“插入或替换”操作，你可以提出一个以下形式的
请求：如果实体不存在，则插入一个实体；如果实体存在，
则替换现有实体。以下代码基于前面的示例
针对“Walter Harp”插入或替换实体。创建
新实体后，此代码调用
**TableOperation.insertOrReplace** 方法。此代码随后使用表
和插入或替换表操作作为参数对
**CloudTableClient** 调用 **execute**。若只更新实体的一部分，
则可以改用 **TableOperation.insertOrMerge** 方法。
请注意，本地存储模拟器不支持插入或替换，
因此，此代码仅在使用表服务中的帐户时
才能运行。你可以在此[博客文章][]中了解
有关“插入或替换”和“插入或合并”的更多信息。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 创建新客户实体。
    CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
    customer5.setEmail("Walter@contoso.com");
    customer5.setPhoneNumber("425-555-0106");

    // 创建一个操作来将新客户添加到 people 表。
    TableOperation insertCustomer5 = TableOperation.insertOrReplace(customer5);

    // 将操作提交到表服务。
    tableClient.execute("people", insertCustomer5);

## 如何：删除实体

你可以在检索到实体后轻松将其删除。检索到实体后，
对要删除的实体调用 **TableOperation.delete**。
然后对 **CloudTableClient** 调用 **execute**。以下代码
检索并删除一个客户实体。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 创建一个操作来检索分区键为“Smith”且行键为“Jeff”的实体。
    TableOperation retrieveSmithJeff = TableOperation.retrieve("Smith", "Jeff", CustomerEntity.class);

    // 检索分区键为“Smith”且行键为“Jeff”的实体。
    CustomerEntity entitySmithJeff =
    tableClient.execute("people", retrieveSmithJeff).getResultAsType();

    // 创建一个操作来删除实体。
    TableOperation deleteSmithJeff = TableOperation.delete(entitySmithJeff);

    // 将删除操作提交到表服务。
    tableClient.execute("people", deleteSmithJeff);

## 如何：删除表

最后，以下代码将从存储帐户中删除一个表。
表在删除之后的一段时间内（通常小于四十秒）
无法重新创建。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);
     
    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.createCloudTableClient();
     
    // 如果表存在，则删除它及其所有数据。
    tableClient.deleteTableIfExists("people");

## 后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Windows Azure 中存储和访问
    数据]
-   访问 [Azure 存储空间团队博客][]

  [后续步骤]: #NextSteps
  [什么是表存储]: #what-is
  [概念]: #Concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 Java 应用程序]: #CreateApplication
  [创建用于访问表存储的应用程序]: #ConfigureStorage
  [设置 Azure 存储连接字符串]: #ConnectionString
  [如何：创建表]: #CreateTable
  [如何：将实体添加到表]: #AddEntity
  [如何：插入一批实体]: #InsertBatch
  [如何：检索分区中的所有实体]: #RetrieveEntities
  [如何：检索分区中的一部分实体]: #RetrieveRange
  [如何：检索单个实体]: #RetriveSingle
  [如何：修改实体]: #ModifyEntity
  [如何：查询一部分实体属性]: #QueryProperties
  [如何：插入或替换实体]: #InsertOrReplace
  [如何：删除实体]: #DeleteEntity
  [如何：删除表]: #DeleteTable
  [howto-table-storage]: ../includes/howto-table-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
  [博客文章]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
