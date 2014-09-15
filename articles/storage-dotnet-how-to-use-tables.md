<properties linkid="dev-net-how-to-table-services" urlDisplayName="Table Service" pageTitle="How to use table storage  from .NET | Microsoft Azure" metaKeywords="Get started Azure table   Azure nosql   Azure large structured data store   Azure table   Azure table storage   Azure table .NET   Azure table storage .NET   Azure table C#   Azure table storage C#" description="Learn how to use Microsoft Azure Table storage to create and delete tables and insert and query entities in a table." services="storage" documentationCenter=".NET" metaCanonical="" disqusComments="1" umbracoNaviHide="1" title="How to use Microsoft Azure Table storage" authors="tamram" />

# 如何通过 .NET 使用表存储

本指南将演示如何使用 Azure 表存储服务执行常见方案。
示例是用 C\# 代码编写的并使用了
Azure .NET 存储客户端库。涉及的方案包括**创建和删除表**
，以及**使用表实体**。有关表的详细信息，请参阅
[后续步骤][]部分。

> [WACOM.NOTE] 本指南适用于 Azure .NET 存储客户端库 2.x 及更高版本。建议使用的版本是存储客户端库 3.x，可通过 NuGet 或 Azure SDK for .NET 2.3 获得。有关如何获取存储客户端库的详细信息，请参阅[如何：以编程方式访问表存储][]。

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
-   [如何：替换实体][]
-   [如何：插入或替换实体][]
-   [如何：查询一部分实体属性][]
-   [如何：删除实体][]
-   [如何：删除表][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-table-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 设置连接字符串设置存储连接字符串

Azure .NET 存储客户端库支持使用存储连接字符
串来配置终结点和用于访问存
储服务的凭据。你可以将存储连接字符串放置在一个配置文件中，而不是
在代码中对其进行硬编码：

-   当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）来存储连接字符串。
-   在使用 Azure 网站、Azure 虚拟机时或者构建准备在 Azure 外部运行的 .NET 应用程序时，建议你使用 .NET 配置系统（如 `web.config` 或 `app.config` 文件）来存储连接字符串。

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

### 使用 .NET 配置来配置连接字符串

如果你正在编写不是 Azure 云服务的应用程序（参见上一部分），则建议你使用 .NET 配置系统（如 `web.config` 或 `app.config`）。这包括 Azure 网站或 Azure 虚拟机，以及设计为在 Azure 外部运行的应用程序。你可以使用 `<appSettings>` 元素存储连接字符串，如下所示：

    <configuration>
    <appSettings>
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]" />
    </appSettings>
    </configuration>

阅读[配置连接字符串][]，了解有关存储连接字符串的详细信息。

你现在即可准备执行本指南中的操作任务。

## 以编程方式访问如何：以编程方式访问表存储

### 获得程序集

你可以使用 NuGet 来获得 `Microsoft.WindowsAzure.Storage.dll` 程序集。在**“解决方案资源管理器”**中，右键单击你的项目并选择**“管理 NuGet 包”**。在线搜索“WindowsAzure.Storage”，然后单击**“安装”**以安装 Azure 存储包和依赖项。

Azure SDK for .NET 中也包括了 `Microsoft.WindowsAzure.Storage.dll`，可从 [.NET 开发人员中心][]下载该版本。该程序集将安装到 `%Program Files%\Microsoft SDKs\Windows Azure\.NET SDK\<sdk-version>\ref\` 目录中。

### 命名空间声明

在你希望在其中以编程方式访问 Azure 存储空间的任何 C\# 文件中，
将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Table;

确保你引用了 `Microsoft.WindowsAzure.Storage.dll` 程序集。

### 检索连接字符串

可以使用 **CloudStorageAccount** 类型来表示你的存储
帐户信息。如果你使用的
是 Azure 项目模板并且/或者引用了
Microsoft.WindowsAzure.CloudConfigurationManager 命名空间，
则可以使用 **CloudConfigurationManager** 类型
从 Azure 服务配置中检索你的存储连接字符串和
存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于 `web.config` 或 `app.config` 中（如上所示），则你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

    using System.Configuration;
    ...
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

### ODataLib 依赖项

.NET 存储客户端库中的 ODataLib 依赖项可通过在 NuGet （而非 WCF 数据服务）上获得的 ODataLib（5.0.2 版）包来解析。ODataLib 库可直接下载或者通过 NuGet 由代码项目引用。特定的 ODataLib 包为 [OData][]、[Edm][] 和 [Spatial][]。

## 创建表如何：创建表

利用 **CloudTableClient** 对象，你可以获得表和实体的引用对象。
以下代码将创建 **CloudTableClient** 对象
并使用它创建新表。本指南中的所有代码假定将构建的应用程序
是 Azure 云服务项目，并且使用存储在 Azure 应用
程序的服务配置中的存储连接字符串。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 如果该表不存在，则创建它。
    CloudTable table = tableClient.GetTableReference("people");
    table.CreateIfNotExists();

## 将实体添加到表如何：将实体添加到表

实体将映射到使用派生自 **TableEntity** 的
自定义类的 C\# 对象。若要将实体添加到表，请创建用于定义
实体的属性的类。以下代码定义了
将客户的名字和姓氏分别用作行键和分区键的
实体类。实体的分区键和行键共同唯一地标识
表中的实体。查询分区键相同的实体的
速度快于查询分区键不同的实体的速度，
但使用不同的分区键可实现更高的并行操作
可伸缩性。对于应存储在表服务中的任何属性，该属性必须是
公开 `get` 和 `set` 的受支持类型的公共属性。
另外，你的实体类型*必须* 公开不带参数的构造函数。

    public class CustomerEntity :TableEntity
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

可使用你在“如何：创建表”中创建的 **CloudTable** 对象
执行涉及实体的表操作。要执行的操作
由 **TableOperation** 对象表示。以下代码示例演示如何创建 **CloudTable** 对象以及 **CustomerEntity** 对象。为准备此操作，会创建一个 **TableOperation** 以将客户实体插入该表中。最后，将通过调用 **CloudTable.Execute** 执行此操作。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建新客户实体。
    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
    customer1.Email = "Walter@contoso.com";
    customer1.PhoneNumber = "425-555-0101";

    // 创建插入客户实体的 TableOperation。
    TableOperation insertOperation = TableOperation.Insert(customer1);

    // 执行插入操作。
    table.Execute(insertOperation);

## 插入一批实体如何：插入一批实体

你可以通过一个写入操作将一批实体插入表中。
有关批处理操作的一些其他
注意事项：

1.  你可以在同一批处理操作中执行更新、删除和插入操作。
2.  单个批处理操作最多可包含 100 个实体。
3.  单个批处理操作中的所有实体都必须具有相同的
    分区键。
4.  可以作为批处理操作执行查询时，此操作必须是批处理中仅有的操作。

以下代码示例创建两个实体对象，并使用 **Insert** 方法
将其中每个对象都添加到 **TableBatchOperation** 中。然后调用 **CloudTable.Execute** 以执行此操作。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建批处理操作。
    TableBatchOperation batchOperation = new TableBatchOperation();

    // 创建一个客户实体，并将其添加到表中。
    CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
    customer1.Email = "Jeff@contoso.com";
    customer1.PhoneNumber = "425-555-0104";
            
    // 创建另一个客户实体，并将其添加到表中。
    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
    customer2.Email = "Ben@contoso.com";
    customer2.PhoneNumber = "425-555-0102";
            
    // 将两个客户实体添加到批处理插入操作中。
    batchOperation.Insert(customer1);
    batchOperation.Insert(customer2);

    // 执行批处理操作。
    table.ExecuteBatch(batchOperation);

## 检索所有实体如何：检索分区中的所有实体

若要查询表以获取分区中的所有实体，请使用 **TableQuery** 对象。
以下代码示例指定了一个筛选器，以筛选分区键
为“Smith”的实体。此示例会将查询结果中每个实体的
字段输出到控制台。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 构造所有客户实体的查询操作，其中 PartitionKey="Smith"。
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // 打印每个客户的字段。
    foreach (CustomerEntity entity in table.ExecuteQuery(query))
    {
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
    entity.Email, entity.PhoneNumber);
    }

## 检索一部分实体如何：检索分区中的一部分实体

如果不想查询分区中的所有实体，则可以通过结合使用分区键筛选器
与行键筛选器来指定一个范围。以下代码示例使用
两个筛选器来获取分区“Smith”中行键（名字）以字母表中字母
“E”前面的字母开头的所有实体，
然后输出查询结果。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建表查询。
    TableQuery<CustomerEntity> rangeQuery = new TableQuery<CustomerEntity>().Where(
    TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"),
    TableOperators.And,
    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "E")));

    // 循环访问结果，显示实体的有关信息。
    foreach (CustomerEntity entity in table.ExecuteQuery(rangeQuery))
    {
    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
    entity.Email, entity.PhoneNumber);
    }

## 检索单个实体如何：检索单个实体

你可以编写查询以检索单个特定实体。
以下代码使用 **TableOperation** 来指定客户“Ben Smith”。
此方法将仅返回一个实体，而不是返回一个集合，
并且 **TableResult.Result** 中的返回值是一个 **CustomerEntity**。
在一个查询中同时指定分区键和行键是从表服务
检索单个实体的最快方法。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建取客户实体的检索操作。
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // 执行检索操作。
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // 打印结果的电话号码。
    if (retrievedResult.Result != null)
    Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
    else
    Console.WriteLine("The phone number could not be retrieved.");

## 替换实体如何：替换实体

若要更新实体，请从表服务中检索它，修改实体对象，然后
将更改保存回表服务。以下代码
将更改现有客户的电话号码。此代码
使用 **Replace**，而不是调用 **Insert**。
这将导致在服务器上完全替换该实体，
除非服务器上的该实体自检索到它以后发生更改（
这种情况下，该操作将失败）。操作失败将防止你的应用程序
无意中覆盖应用程序的其他组件
在检索与更新之间所做的更改。正确处理此失败问题的方法是
再次检索实体，进行更改（如果仍有效），
然后再次执行 **Replace** 操作。下一节将为你演示
如何重写此行为。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建取客户实体的检索操作。
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // 执行操作。
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // 将结果赋给一个 CustomerEntity 对象。
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
    {
    // 更改电话号码。
    updateEntity.PhoneNumber = "425-555-0105";

    // 创建 InsertOrReplace TableOperation
    TableOperation updateOperation = TableOperation.Replace(updateEntity);

    // 执行操作。
    table.Execute(updateOperation);

    Console.WriteLine("Entity updated.");
    }

    else
    Console.WriteLine("Entity could not be retrieved.");

## 插入或替换实体如何：插入或替换实体

如果实体在从服务器中检索到它以后发生更改，则 **Replace** 操作
将失败。此外，要使 **Replace** 成功，必须
首先从服务器中检索实体。不过，有时你不知道
实体是否存在于服务器上以及其中
存储的当前值是否不相关，
因此你的更新应当将它们全部覆盖。为此，你应使用 **InsertOrReplace**
 操作。如果该实体不存在，此操作将插入它，如果存在，则替换它，
而不管上次更新是何时进行的。在以下代码示例中，
仍将检索 Ben Smith 的客户实体，但稍后会使用 **InsertOrReplace** 将其保存回服务器。
将覆盖在检索与更新操作之间对实体进行的任何更新。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable 对象。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建取客户实体的检索操作。
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // 执行操作。
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // 将结果赋给一个 CustomerEntity 对象。
    CustomerEntity updateEntity = (CustomerEntity)retrievedResult.Result;

    if (updateEntity != null)
    {
    // 更改电话号码。
    updateEntity.PhoneNumber = "425-555-1234";

    // 创建 InsertOrReplace TableOperation
    TableOperation insertOrReplaceOperation = TableOperation.InsertOrReplace(updateEntity);

    // 执行操作。
    table.Execute(insertOrReplaceOperation);

    Console.WriteLine("Entity was updated.");
    }

    else
    Console.WriteLine("Entity could not be retrieved.");

## 查询一部分属性如何：查询一部分实体属性

表查询可以只检索实体中的少数几个属性而不是所有实体属性。此方法称为投影，可减小带宽并提高查询性能，尤其是对于大型实体。以下代码
中的查询只返回表中实体的电子邮件
地址。这可通过使用 **DynamicTableEntity** 和 **EntityResolver** 的
查询来实现。你可以在此[博客文章][]中了解有关投影的详细信息。请注意，本地存储模拟器不支持投影，因此，此代码仅在使用表服务中的帐户时才能运行。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable。
    CloudTable table = tableClient.GetTableReference("people");

    // 定义查询，并且仅选择 Email 属性
    TableQuery<DynamicTableEntity> projectionQuery = new TableQuery<DynamicTableEntity>().Select(new string[] { "Email" });

    // 定义一个实体解析器以便在检索后使用实体。
    EntityResolver<string> resolver = (pk, rk, ts, props, etag) => props.ContainsKey("Email") ? props["Email"].StringValue :null;

    foreach (string projectedEmail in table.ExecuteQuery(projectionQuery, resolver, null, null))
    {
    Console.WriteLine(projectedEmail);
    }

## 删除实体如何：删除实体

在检索实体之后，可使用所示的用于更新实体的同一模式轻松删除该实体。
以下代码
检索并删除一个客户实体。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable。
    CloudTable table = tableClient.GetTableReference("people");

    // 创建要对客户实体执行的检索操作。
    TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

    // 执行操作。
    TableResult retrievedResult = table.Execute(retrieveOperation);

    // 将结果赋给一个 CustomerEntity。
    CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;

    // 创建 Delete TableOperation。
    if (deleteEntity != null)
    {
    TableOperation deleteOperation = TableOperation.Delete(deleteEntity);

    // 执行操作。
    table.Execute(deleteOperation);

    Console.WriteLine("Entity deleted.");
    }

    else
    Console.WriteLine("Could not retrieve the entity.");

## 删除表如何：删除表

最后，以下代码示例将从存储帐户中删除表。
在删除表之后的一段时间
内无法重新创建它。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建表客户端。
    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    // 创建表示“people”表的 CloudTable。
    CloudTable table = tableClient.GetTableReference("people");

    // 如果表存在，则删除它。
    table.DeleteIfExists();

## 后续步骤后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 Blob 服务参考文档，了解有关可用 API 的完整详情：
    -   [.NET 存储客户端库参考][]
    -   [REST API 参考][]
-   在以下位置了解使用 Azure 存储空间能够执行的更高级任务：[在 Azure 中存储和访问数据][]。
-   查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    -   使用 [Blob 存储][]来存储非结构化数据。
    -   使用[队列存储][]来存储结构化数据。
    -   使用 [SQL Database][] 来存储关系数据。

  [后续步骤]: #next-steps
  [如何：以编程方式访问表存储]: #configure-access
  [什么是表服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置存储连接字符串]: #setup-connection-string
  [如何：创建表]: #create-table
  [如何：将实体添加到表]: #add-entity
  [如何：插入一批实体]: #insert-batch
  [如何：检索分区中的所有实体]: #retrieve-all-entities
  [如何：检索分区中的一部分实体]: #retrieve-range-entities
  [如何：检索单个实体]: #retrieve-single-entity
  [如何：替换实体]: #replace-entity
  [如何：插入或替换实体]: #insert-or-replace-entity
  [如何：查询一部分实体属性]: #query-entity-properties
  [如何：删除实体]: #delete-entity
  [如何：删除表]: #delete-table
  [howto-table-storage]: ../includes/howto-table-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [Blob5]: ./media/storage-dotnet-how-to-use-table-storage/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-table-storage/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-table-storage/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-table-storage/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-table-storage/blob9.png
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 开发人员中心]: http://azure.microsoft.com/zh-cn/develop/net/
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
  [博客文章]: http://blogs.msdn.com/b/windowsazurestorage/archive/2011/09/15/windows-azure-tables-introducing-upsert-and-query-projection.aspx
  [.NET 存储客户端库参考]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Blob 存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-blobs/
  [队列存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-queues/
  [SQL Database]: /en-us/documentation/articles/sql-database-dotnet-how-to-use/
