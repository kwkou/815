<properties linkid="dev-net-2-how-to-table-services" urlDisplayName="Table Service (2.0)" pageTitle="How to use table storage | Microsoft Azure" metaKeywords="Get started Azure table, Azure nosql, Azure large structured data store, Azure table, Azure table storage, Azure table .NET, Azure table storage .NET, Azure table C#, Azure table storage C#" description="Learn how to use table storage to create and delete tables and insert and query entities in a table." metaCanonical="" services="storage" documentationCenter=".NET" title="How to use the Table Storage Service" authors="" solutions="" manager="paulettm" editor="cgronlun" />

# 如何使用表存储服务

[1.7 版][] [2.0 版][]

本指南将演示如何使用 Azure 表存储服务执行常见方案。
示例是用 C\# 代码编写的且使用了 .NET API。
涉及的方案包括**创建和删除表、在表中插入和查询实体**
。有关表的详细信息，请参阅
[后续步骤][]部分。

## 目录

-   [什么是表服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [设置存储连接字符串][]
-   [如何：以编程方式访问表存储][]
-   [如何：创建表][]
-   [如何：将实体添加到表][]
-   [如何：插入一批实体][]
-   [如何：检索分区中的所有实体][]
-   [如何：检索分区中的一部分实体][]
-   [如何：检索单个实体][]
-   [如何：更新实体][]
-   [如何：查询一部分实体属性][]
-   [如何：插入或替换实体][]
-   [如何：删除实体][]
-   [如何：删除表][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-table-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 设置连接字符串设置存储连接字符串

Azure .NET 存储 API 支持
使用存储连接字符串来配置用于访问存储
服务的终结点和凭据。你可以将存储连接字符串放置在一个配置文件中，而不是
在代码中对其进行硬编码：

-   当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）来存储连接字符串。
-   在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如 `web.config` 文件）来存储连接字符串。

在上述两种情况下，你都可以使用 `CloudConfigurationManager.GetSetting` 方法检索连接字符串，本指南稍后将对此进行介绍。

### 在使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务
项目特有的，它使你能够从 Azure 管理
门户动态更改配置设置，而无需部署你的应用程序。

在 Azure 服务配置中配置连接字符串：

1.  在 Visual Studio 解决方案资源管理器内 Azure 部署
    项目的**“角色”**文件夹中，右键单击你的 Web 角色或辅助角色，
    然后单击**“属性”**。
    ![Blob5][]

2.  单击**“设置”**选项卡并按**“添加设置”**按钮。
    ![Blob6][]

    新的 **Setting1** 条目稍后将显示在设置网格中。

3.  在新的 **Setting1** 条目的**“类型”**下拉列表中，选择
    **“连接字符串”**。
    ![Blob7][]

4.  单击 **Setting1** 条目最右侧的 **...** 按钮。
    此时将打开**“存储帐户连接字符串”**对话框。

5.  选择是要定位到存储模拟器（在本地计算机上模拟的 Windows
    Azure 存储空间），还是要定位到云中的实际存储帐户。
    本指南中的代码适用于其中任一方式。
    如果你希望使用我们之前在 Azure 中创建的存储帐户
    来存储 Blob 数据，请输入从本教程前面的步骤中
    复制的**“主访问密钥”**值。
    ![Blob8][]

6.  将条目**“名称”**从 **Setting1** 更改为更友好的名称，
    例如 **StorageConnectionString**。在本指南后面的
    代码中你将引用此连接字符串。
    ![Blob9][]

### 在使用网站或虚拟机时配置连接字符串

在使用网站或虚拟机时，建议你使用 .NET 配置系统（如 `web.config`）。你可以使用 `<appSettings>` 元素存储连接字符串：

    <configuration>
    <appSettings>
    <add key="StorageConnectionString"
    value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]" />
    </appSettings>
    </configuration>

阅读[配置连接字符串][]，了解有关存储连接字符串的详细信息。

你现在即可准备执行本指南中的操作任务。

## 以编程方式访问如何：以编程方式访问表存储

在你希望在其中以编程方式访问 Azure 存储空间的任何 C\# 文件中，
将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.StorageClient;

你可以使用 **CloudStorageAccount** 类型和
**CloudConfigurationManager** 类型从
Azure 服务配置中检索你的存储连接字符串和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

## 创建表如何：创建表

利用 **CloudTableClient** 对象，你可以获得表和实体的引用对象。
以下代码将创建 **CloudTableClient** 对象
并使用它创建新表。本指南中的所有代码
都使用存储在 Azure 应用程序的服务配置中的
存储连接字符串。还有其他方法可用来
创建 **CloudStorageAccount** 对象。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 如果该表不存在，则创建它
    string tableName = "people";
    tableClient.CreateTableIfNotExist(tableName);

## 将实体添加到表如何：将实体添加到表

实体将映射到使用派生自 **TableServiceEntity** 的
自定义类的 C\# 对象。若要将实体添加到表，请先创建用于
定义实体的属性的类。以下代码定义了
将客户的名字和姓氏分别用作行键和分区键的
实体类。实体的分区键和行键共同唯一地标识
表中的实体。查询分区键相同的
实体的速度快于查询
分区键不同的实体的速度。

    public class CustomerEntity :TableServiceEntity
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

涉及实体的表操作需要使用 **TableServiceContext** 对象。
此对象可跟踪通过客户端代码创建和访问的所有表实体的
客户端状态。保留代表单个实体的客户端对象
可使写入操作更为有效，
因为执行保存操作时只会
在表服务中更新发生更改的对象。以下代码通过调用
**GetDataServiceContext** 方法创建
**TableServiceContext**
 对象。然后，该代码会创建 **CustomerEntity**
 类的实例。该代码会调用 **serviceContext.AddObject** 将新实体插入表中。
这会向
**serviceContext** 中添加实体对象，但不执行任何服务操作。最后，该代码
会在调用 **SaveChangesWithRetries** 方法
时将新实体发送到表服务。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 创建新的客户实体
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // 将新客户添加到 people 表
    serviceContext.AddObject("people", customer1);

    // 将操作提交到表服务
    serviceContext.SaveChangesWithRetries();

## 插入一批实体如何：插入一批实体

你可以通过一个写入操作将一批实体插入到表服务。
以下代码使用 **AddObject** 方法创建三个实体
对象并将每个对象都添加到服务上下文中。然后，该代码
使用 **SaveChangesOptions.Batch** 参数调用 **SaveChangesWithRetries**。
如果你省略了 **SaveChangesOptions.Batch**，将分别
调用表服务三次。有关批处理操作的一些其他
注意事项：

1.  你可以执行批处理更新、删除或插入操作。
2.  单个批处理操作最多可包含 100 个实体。
3.  单个批处理操作中的所有实体都必须具有相同的
    分区键。

<!-- -->

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
    string tableName = "people";

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 创建一个客户实体并将其添加到表中
    CustomerEntity customer = new CustomerEntity("Smith", "Jeff");
    customer.Email = "Jeff@contoso.com";
    customer.PhoneNumber = "425-555-0104";
    serviceContext.AddObject(tableName, customer);

    // 创建另一个客户实体并将其添加到表中
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.Email = "Ben@contoso.com";
    customer2.PhoneNumber = "425-555-0102";
    serviceContext.AddObject(tableName, customer2);

    // 创建一个客户实体并将其添加到表中
    CustomerEntity customer3 = new CustomerEntity("Smith", "Denise");
    customer3.Email = "Denise@contoso.com";
    customer3.PhoneNumber = "425-555-0103";
    serviceContext.AddObject(tableName, customer3);

    // 将操作提交到表服务
    serviceContext.SaveChangesWithRetries(SaveChangesOptions.Batch);

## 检索所有实体如何：检索分区中的所有实体

若要查询表以获取分区中的实体，可以使用 LINQ 查询。
可以调用 **serviceContext.CreateQuery** 从你的数据源
创建查询。以下代码指定了一个筛选器，用于筛选分区键是“Smith”
的实体。对 LINQ 查询的结果调用 **AsTableServiceQuery\<CustomerEntity\>**
以完成创建 **CloudTableQuery** 对象。
然后你可以使用在 **foreach** 循环中创建的
**partitionQuery** 对象来使用结果。此代码会将查询结果中每个
实体的字段输出到控制台。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 指定一个分区查询，使用“Smith”作为分区键
    CloudTableQuery<CustomerEntity> partitionQuery =
    (from e in serviceContext.CreateQuery<CustomerEntity>("people")
    where e.PartitionKey == "Smith"
    select e).AsTableServiceQuery<CustomerEntity>();

    // 循环访问结果，显示实体的相关信息
    foreach (CustomerEntity entity in partitionQuery)
    {
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
    entity.Email, entity.PhoneNumber);
    }

## 检索一部分实体如何：检索分区中的一部分实体

如果不想查询分区中的所有实体，则可以使
用 **CompareTo** 方法，而不是使用常见的
大于 (\>) 和小于 (\<) 运算符来指定一个范围。这是因为
后者将导致不适当的查询构造。以下代码使用
两个筛选器来获取分区“Smith”中行键（名字）以字母表中字母
“E”及其前面的字母开头的所有实体。然后，
该代码输出查询结果。如果你使用在本指南的批量插入部分中
添加到表的实体，则此次只会返回两个实体（Ben 和 Denise Smith），
而不会包括 Jeff Smith。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 指定一个分区查询，使用“Smith”作为分区键，
    // 使用最大为字母“E”的行键
    CloudTableQuery<CustomerEntity> entityRangeQuery =
    (from e in serviceContext.CreateQuery<CustomerEntity>("people")
    where e.PartitionKey == "Smith" && e.RowKey.CompareTo("E") < 0
    select e).AsTableServiceQuery<CustomerEntity>();

    // 循环访问结果，显示实体的相关信息
    foreach (CustomerEntity entity in entityRangeQuery)
    {
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
    entity.Email, entity.PhoneNumber);
    }

## 检索单个实体如何：检索单个实体

你可以编写查询以检索单个特定实体。以下代码
使用两个筛选器来指定客户“Jeff Smith”。
该代码调用 **FirstOrDefault** 而不是
调用 **AsTableServiceQuery**。此方法仅返回一个实体，而不是一个集合，
因此代码会将返回值直接分配给
**CustomerEntity** 对象。如果没有实体具有完全匹配的分区键和行键，
则会返回 null 值。在查询中同时指定分区键和行键是
从表服务中检索单个实体的最快方法。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 返回分区键为“Smith”且行键为“Jeff”的实体
    CustomerEntity specificEntity =
    (from e in serviceContext.CreateQuery<CustomerEntity>("people")
    where e.PartitionKey == "Smith" && e.RowKey == "Jeff"
    select e).FirstOrDefault();

## 更新实体如何：更新实体

若要更新实体，请从表服务中检索它，修改实体对象，
然后将更改保存回表服务。以下代码
将更改现有客户的电话号码。此代码
将调用 **UpdateObject**，而不是像执行插入时那样
调用 **AddObject**。**SaveChangesWithRetries** 方法将调用表服务，
并更新该实体，除非在此应用程序检索到该实体之后
另一个应用程序对它进行了更改。如果出现这种情况，
则会引发异常，必须再次检索、修改并保存该实体。
此重试模式在分布式存储系统中很常见。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 返回分区键为“Smith”且行键为“Jeff”的实体
    CustomerEntity specificEntity =
    (from e in serviceContext.CreateQuery<CustomerEntity>("people")
    where e.PartitionKey == "Smith" && e.RowKey == "Jeff"
    select e).FirstOrDefault();

    // 指定一个新电话号码
    specificEntity.PhoneNumber = "425-555-0105";

    // 更新实体
    serviceContext.UpdateObject(specificEntity);

    // 将操作提交到表服务
    serviceContext.SaveChangesWithRetries();

## 查询一部分属性如何：查询一部分实体属性

对表的查询可以只检索实体中的少数几个属性。此方法称为投影，
可减少带宽并提高查询性能，尤其适用于大型实体。
以下代码
中的查询只返回表中实体的电子邮件
地址。你可以在此[博客文章][]中了解有关投影的详细信息。请注意，
本地存储模拟器不支持投影，因此，此代码仅在使用表服务中的帐户时
才能运行。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 定义一个仅检索 Email 属性的投影查询
    var projectionQuery = 
    from e in serviceContext.CreateQuery<CustomerEntity>("people")
    select new
        {
    Email = e.Email
    // 你可以在此处指定其他字段
        };

    // 循环访问结果，显示 Email 值
    foreach (var person in projectionQuery)
    {
    Console.WriteLine(person.Email);
    }

## 插入或替换实体如何：插入或替换实体

很多时候，你需要将某个实体添加到表中，但又不知道该实体是否
已存在于表中。利用“插入或替换”操作，你可以提出一个以下形式的
请求：如果实体不存在，则插入一个实体；如果实体存在，
则替换现有实体。以下代码基于前面的示例
针对“Walter Harp”插入或替换实体。创建新实体后，
此代码调用 **serviceContext.AttachTo**
方法。然后，此代码会调用 **UpdateObject**，最后调用带
**SaveChangesOptions.ReplaceOnUpdate** 参数的
**SaveChangesWithRetries**。省略
**SaveChangesOptions.ReplaceOnUpdate** 参数会导致“插入或合并”
操作。请注意，本地存储模拟器不支持“插入或替换”，
因此，此代码仅在使用表服务中的帐户时才能运行。
你可以在此[博客文章][]中了解
有关“插入或替换”和“插入或合并”的更多信息。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    // 创建新的客户实体
    CustomerEntity customer5 = new CustomerEntity("Harp", "Walter");
    customer5.Email = "Walter@contoso.com";
    customer5.PhoneNumber = "425-555-0106";

    // 将此客户添加到 people 表
    serviceContext.AttachTo("people", customer5);

    // 如果此客户是新客户，则插入；如果已存在，则替换
    serviceContext.UpdateObject(customer5);

    // 将操作提交到表服务，使用 ReplaceOnUpdate 选项
    serviceContext.SaveChangesWithRetries(SaveChangesOptions.ReplaceOnUpdate);

## 删除实体如何：删除实体

你可以在检索到实体后轻松将其删除。还可以
使用 **AttachTo** 方法开始跟踪它，而无需从服务器中检索它
（请参阅上面的“插入或替换”）。在使
用 **serviceContext** 跟踪实体后，对要删除的实体
调用 **DeleteObject**。然后调用 **SaveChangesWithRetries**。以下代码
检索并删除一个客户实体。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 获取数据服务上下文
    TableServiceContext serviceContext = tableClient.GetDataServiceContext();

    CustomerEntity specificEntity =
    (from e in serviceContext.CreateQuery<CustomerEntity>("people")
    where e.PartitionKey == "Smith" && e.RowKey == "Jeff"
    select e).FirstOrDefault();

    // 删除实体
    serviceContext.DeleteObject(specificEntity);

    // 将操作提交到表服务
    serviceContext.SaveChangesWithRetries();

## 删除表如何：删除表

最后，以下代码将从存储帐户中删除一个表。
在删除表之后的一段时间
内无法重新创建它。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 如果表存在，则将其删除
    tableClient.DeleteTableIfExist("people");

## 后续步骤后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 Blob 服务参考文档，了解有关可用 API 的完整详情：
    -   [.NET 客户端库引用][]
    -   [REST API 参考][]
-   在以下位置了解使用 Azure 存储空间能够执行的更高级任务：[在 Azure 中存储和访问数据][]。
-   查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    -   使用 [Blob 存储][]来存储非结构化数据。
    -   使用 [SQL Database][] 来存储关系数据。

  [1.7 版]: /en-us/develop/net/how-to-guides/table-services-v17/ "1.7 版"
  [2.0 版]: /en-us/develop/net/how-to-guides/table-services/ "2.0 版"
  [后续步骤]: #next-steps
  [什么是表服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置存储连接字符串]: #setup-connection-string
  [如何：以编程方式访问表存储]: #configure-access
  [如何：创建表]: #create-table
  [如何：将实体添加到表]: #add-entity
  [如何：插入一批实体]: #insert-batch
  [如何：检索分区中的所有实体]: #retrieve-all-entities
  [如何：检索分区中的一部分实体]: #retrieve-range-entities
  [如何：检索单个实体]: #retrieve-single-entity
  [如何：更新实体]: #update-entity
  [如何：查询一部分实体属性]: #query-entity-properties
  [如何：插入或替换实体]: #insert-entity
  [如何：删除实体]: #delete-entity
  [如何：删除表]: #delete-table
  [howto-table-storage]: ../includes/howto-table-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [Blob5]: ./media/storage-dotnet-how-to-use-table-storage-17/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-table-storage-17/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-table-storage-17/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-table-storage-17/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-table-storage-17/blob9.png
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [博客文章]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Blob 存储]: /en-us/develop/net/how-to-guides/blob-storage/
  [SQL Database]: /en-us/develop/net/how-to-guides/sql-database/
