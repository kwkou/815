<properties
    pageTitle="请求单位和估计吞吐量 - DocumentDB | Azure"
    description="了解如何理解、指定和估计 DocumentDB 中的请求单位要求。"
    services="documentdb"
    author="syamkmsft"
    manager="jhubbard"
    editor="mimig"
    documentationcenter="" />
<tags
    ms.assetid="d0a3c310-eb63-4e45-8122-b7724095c32f"
    ms.service="DocumentDB"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/10/2017"
    wacn.date="05/31/2017"
    ms.author="syamk"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="0ff68caedd0229b19b59f035e36382cb638d1672"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="request-units-in-azure-documentdb"></a>DocumentDB 中的请求单位
现已推出：DocumentDB [请求单位计算器](https://www.documentdb.com/capacityplanner)。 了解[估计吞吐量需求](/documentation/articles/documentdb-request-units/#estimating-throughput-needs/)。

![吞吐量计算器][5]

## <a name="introduction"></a>介绍
[DocumentDB](/home/features/documentdb/) 是由 Microsoft 提供的全球分布式多模型数据库。 使用 DocumentDB，无需租用虚拟机、部署软件或监视数据库。 DocumentDB 由 Microsoft 顶级工程师操作和持续监视，以提供一流的可用性、性能和数据保护。 你可以使用所选的 API 访问数据，因为它原生就支持 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/)（文档）、MongoDB（文档）、[Azure 表存储](/home/features/storage/)（键-值）和 [Gremlin](https://tinkerpop.apache.org/gremlin.html)（图形）。 DocumentDB 的计费依据是请求单位 (RU)。 如果使用 RU，则无需保留读取、写入容量或者预配 CPU、内存和 IOPS。

DocumentDB 支持从读取、写入，到复杂的图形查询等一系列操作使用 API。 并非所有请求都是相同的，因此系统会根据请求所需的计算量为它们分配规范化数量的**请求单位**。 操作的请求单位数是确定性的，可以通过响应标头跟踪 DocumentDB 中的任何操作消耗的请求单位数。 

若要提供可预测的性能，需要以 100 RU/秒为单位保留吞吐量。 对于每个 100 RU/秒的块，可以附加 1,000 RU/每分钟的块。 将每秒和每分钟预配吞吐量相组合可以带来极其有利的结果，因为不需要为峰值预配容量，并且与只使用每秒预配的任何服务相比，成本最高可节省 75%。

阅读本文后，将能够回答以下问题：  

- 什么是请求单位和请求费用？
- 如何指定集合的请求单位容量？
- 如何评估应用程序的请求单位需求？
- 如果超过集合的请求单位容量会发生什么情况？

由于 DocumentDB 是多模型数据库，因此必须注意，我们会提到文档 API 的集合/文档、图形 API 的图形/节点，以及表 API 的表/实体。 在整份文档中，我们将它们统称为容器/项。

## <a name="request-units-and-request-charges"></a>请求单位和请求费用
DocumentDB 通过*保留*资源提供快速且可预测的性能，以满足应用程序的吞吐量需求。  由于应用程序加载和访问模式会随着时间推移而更改，DocumentDB 使你可以轻松增加或减少保留供应用程序使用的吞吐量。

通过 DocumentDB，可根据每秒或每分钟（附加）请求单位处理指定保留的吞吐量。  可以将请求单位视为吞吐量货币，因此，可以*保留*每秒或每分钟可用于应用程序的定量有保障请求单位。  DocumentDB 中的每个操作（编写文档、执行查询、更新文档）都会消耗 CPU、内存和 IOPS。  也就是说，每个操作都会产生请求费用（用请求单位表示）。  你要了解影响请求单位费用的因素，以及你的应用程序吞吐量要求，才能尽可能有效地运行你的应用程序。 查询资源管理器也是一个可以测试查询核心的强大工具。

## <a name="specifying-request-unit-capacity"></a>指定请求单位容量
创建 DocumentDB 集合时，可以指定希望为集合保留的每秒请求单位的数量 (RU)。  创建集合之后，将保留指定的 RU 的完整分配供集合使用。  保证每个集合具有专用的和隔离的吞吐量特征。  

如果为集合预配的请求单位数大于或等于 2,500，DocumentDB 要求指定分区键。 以后将集合的吞吐量扩展到 2,500 个请求单位以上时，也需要使用分区键。 因此，我们强烈建议在创建吞吐量容器时配置[分区键](/documentation/articles/documentdb-partition-data/)，不管初始吞吐量有多大。 由于数据可能需要跨多个分区拆分，因此需要选择一个基数较高（几百到几百万个非重复值）的分区键，以便 DocumentDB 能够统一缩放集合/表/图形与请求。 

> [AZURE.NOTE]
> 分区键是一个逻辑边界而不是物理边界。 因此，不需要限制非重复分区键值的数目。 事实上，分区键值宁多勿少，因为 DocumentDB 提供的负载均衡选项较多。

以下代码片段使用 .NET SDK 创建每秒 3,000 个请求单位的集合：

    DocumentCollection myCollection = new DocumentCollection();
    myCollection.Id = "coll";
    myCollection.PartitionKey.Paths.Add("/deviceId");

    await client.CreateDocumentCollectionAsync(
        UriFactory.CreateDatabaseUri("db"),
        myCollection,
        new RequestOptions { OfferThroughput = 3000 });

DocumentDB 运行一个保留模型来预配吞吐量。 也就是说，用户需要根据*保留的*吞吐量付费，不管实际*使用的*吞吐量是多少。 随着应用程序的负载、数据和使用情况模式的更改，可以通过 SDK 或使用 [Azure 门户](https://portal.azure.cn)轻松扩展和缩减保留的 RU 数量。

每个集合/表/图形映射到 DocumentDB 中的 `Offer` 资源，该资源包含有关预配吞吐量的元数据。 可以通过查找容器的相应服务资源，然后使用新的吞吐量值来对它进行更新，来更改分配的吞吐量。 以下代码片段使用 .NET SDK 将集合的吞吐量更改为每秒 5,000 个请求单位：

    // Fetch the resource to be updated
    Offer offer = client.CreateOfferQuery()
                    .Where(r => r.ResourceLink == collection.SelfLink)    
                    .AsEnumerable()
                    .SingleOrDefault();

    // Set the throughput to 5000 request units per second
    offer = new OfferV2(offer, 5000);

    // Now persist these changes to the database by replacing the original resource
    await client.ReplaceOfferAsync(offer);

更改吞吐量不会影响容器的可用性。 通常，新的保留吞吐量在几秒内就会在应用程序上生效。

## <a name="request-unit-considerations"></a>请求单位注意事项
在估计为 DocumentDB 容器保留的请求单位数量时，务必要考虑以下变量：

- **项大小**。 随着大小的增加，用来读取或写入数据的单位数也随之增加。
- **项属性计数**。 假设默认为所有属性创建索引，用于写入文档/节点/实体的单位数将随着属性计数的增加而增加。
- **数据一致性**。 当使用“强”或“有限过时”的数据一致性级别时，将占用更多单位数来读取项。
- **已创建索引的属性**。 每个容器的索引策略都可确定默认情况下要进行索引的属性类别。 通过限制已创建索引的属性数目或通过启用延迟索引编制，可以减少请求单位消耗。
- **文档索引**。 默认情况下，将自动为每个项创建索引，如果你选择不为其中一些项创建索引，则将占用更少的请求单位。
- **查询模式**。 查询的复杂性会影响操作使用的请求单位数量。 谓词数、谓词性质、投影、UDF 数和源数据集的大小都会影响查询操作的成本。
- **脚本使用情况**。  正如查询一样，存储过程和触发器也是根据所执行的操作的复杂性来使用请求单位的。 在开发应用程序时，检查请求费用标头，以更好地了解每个操作消耗请求单位容量的方式。

## <a name="estimating-throughput-needs"></a>估计吞吐量需求
请求单位是请求处理成本的规范化的度量。 单个请求单位用于表示读取（通过自链接或 ID）一个包含 10 个唯一属性值（系统属性除外）的 1KB 项所需的处理容量。 创建（插入）、替换或删除同一个项的请求要占用服务的更多处理，因此需要更多请求单位。   

> [AZURE.NOTE]
> 用于 1KB 项的 1 个请求单位基线通过自链接或项的 ID 与简单的 GET 对应。
> 
> 

例如，下表显示了三种不同的项大小（1KB、4KB 和 64KB）和两个不同的性能级别（500 次读取/秒 + 100 次写入/秒和 500 次读取/秒 + 500 次写入/秒）所要预配的请求单位数。 数据一致性配置为会话级别，索引策略设置为无。

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td valign="top"><p><strong>项大小</strong></p></td>
            <td valign="top"><p><strong>读取数/秒</strong></p></td>
            <td valign="top"><p><strong>写入数/秒</strong></p></td>
            <td valign="top"><p><strong>请求单位</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>1 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 × 1) + (100 × 5) = 1,000 RU/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>1 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 * 1) + (500 * 5) = 3,000 RU/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>4 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 × 1.3) + (100 × 7) = 1,350 RU/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>4 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 × 1.3) + (500 × 7) = 4,150 RU/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>64 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>100</p></td>
            <td valign="top"><p>(500 × 10) + (100 × 48) = 9,800 RU/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>64 KB</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>500</p></td>
            <td valign="top"><p>(500 × 10) + (500 × 48) = 29,000 RU/秒</p></td>
        </tr>
    </tbody>
</table>

### <a name="use-the-request-unit-calculator"></a>使用请求单位计算器
若要帮助客户微调其吞吐量估算，可以使用一个基于 Web 的[请求单位计算器](https://www.documentdb.com/capacityplanner)来帮助估计典型操作的请求单位要求，包括：

- 项创建（写入）
- 项读取
- 项删除
- 项更新

此工具还支持根据提供的示例项预估数据存储需求。

该工具易于使用：

1. 上传一个或多个有代表性的项。
   
    ![将项上传到请求单位计算器][2]
2. 若要预估数据存储需求，请输入你预期要存储的项总数。
3. 输入所需的项创建、读取、更新和删除操作数目（以秒为单位）。 若要估计项更新操作的请求单位费用，请上传上述步骤 1 中包含典型字段更新的示例项的副本。  例如，如果项更新通常会修改名为 lastLogin 和 userVisits 的两个属性，则只要复制示例项、更新这两个属性的值，并上传复制的项。
   
    ![在请求单位计算器中输入吞吐量要求][3]
4. 单击“计算”，然后查看结果。
   
    ![请求单位计算器结果][4]

> [AZURE.NOTE]
> 如果有多种项类型，它们的索引属性大小和数目截然不同，则将每种*类型*的典型项的示例上传到该工具，然后计算结果。
> 
> 

### <a name="use-the-azure-documentdb-request-charge-response-header"></a>使用 DocumentDB 请求费用响应标头
每个来自 DocumentDB 服务的响应都包含一个自定义标头 (`x-ms-request-charge`)，该标头包含请求消耗的请求单位数。 此标头也可通过 DocumentDB SDK 访问。 在 .NET SDK 中，RequestCharge 是 ResourceResponse 对象的属性。  对于查询，在 Azure 门户中的 DocumentDB 查询资源管理器提供了用于执行的查询的请求费用信息。

![检查查询资源管理器中的 RU 费用][1]

基于这一点，有一种方法可以估计应用程序所需的保留的吞吐量：记录与针对应用程序所使用的代表性项运行典型操作相关联的请求单位费用，然后估计你预计每秒执行的操作数。  也要确保测算并包含典型查询和 DocumentDB 脚本使用情况。

> [AZURE.NOTE]
> 如果有多种项类型，它们的索引属性大小和数目截然不同，则记录与每种*类型*的典型项相关联的适用操作请求单位费用。
> 
> 

例如：

1. 记录创建（插入）典型项的请求单位费用。 
2. 记录读取典型项的请求单位费用。
3. 记录更新典型项的请求单位费用。
4. 记录典型、常见项查询的请求单位费用。
5. 记录应用程序使用的任何自定义脚本（存储过程、触发器、用户定义的函数）的请求单位费用。
6. 根据给定的预计每秒运行的操作估计数计算所需的请求单位。

### <a id="GetLastRequestStatistics"></a>使用 API for MongoDB 的 GetLastRequestStatistics 命令
API for MongoDB 支持使用自定义命令 *getLastRequestStatistics* 来检索指定操作的请求费用。

例如，在 Mongo Shell 中，执行所需的操作来验证请求费用。

    > db.sample.find()

接下来，执行命令 *getLastRequestStatistics*。

    > db.runCommand({getLastRequestStatistics: 1})
    {
        "_t": "GetRequestStatisticsResponse",
        "ok": 1,
        "CommandName": "OP_QUERY",
        "RequestCharge": 2.48,
        "RequestDurationInMilliSeconds" : 4.0048
    }

基于这一点，有一种方法可以估计应用程序所需的保留的吞吐量：记录与针对应用程序所使用的代表性项运行典型操作相关联的请求单位费用，然后估计你预计每秒执行的操作数。

> [AZURE.NOTE]
> 如果有多种项类型，它们的索引属性大小和数目截然不同，则记录与每种*类型*的典型项相关联的适用操作请求单位费用。
> 
> 

## <a name="use-api-for-mongodbs-portal-metrics"></a>使用 API for MongoDB 的门户指标
准确估算 API for MongoDB 数据库请求单位费用的最简单方法是使用 [Azure 门户](https://portal.azure.cn)指标。 使用“请求数”和“请求费用”图表，可以估算每个操作消耗的请求单位数，以及每个操作相对于其他操作消耗的请求单位数。

![API for MongoDB 门户指标][6]

## <a name="a-request-unit-estimation-example"></a>请求单位估计示例
请考虑以下 ~1 KB 文档：

    {
     "id": "08259",
      "description": "Cereals ready-to-eat, KELLOGG, KELLOGG'S CRISPIX",
      "tags": [
        {
          "name": "cereals ready-to-eat"
        },
        {
          "name": "kellogg"
        },
        {
          "name": "kellogg's crispix"
        }
      ],
      "version": 1,
      "commonName": "Includes USDA Commodity B855",
      "manufacturerName": "Kellogg, Co.",
      "isFromSurvey": false,
      "foodGroup": "Breakfast Cereals",
      "nutrients": [
        {
          "id": "262",
          "description": "Caffeine",
          "nutritionValue": 0,
          "units": "mg"
        },
        {
          "id": "307",
          "description": "Sodium, Na",
          "nutritionValue": 611,
          "units": "mg"
        },
        {
          "id": "309",
          "description": "Zinc, Zn",
          "nutritionValue": 5.2,
          "units": "mg"
        }
      ],
      "servings": [
        {
          "amount": 1,
          "description": "cup (1 NLEA serving)",
          "weightInGrams": 29
        }
      ]
    }

> [AZURE.NOTE]
> 文档在 DocumentDB 中已缩小，所以系统计算的上述文档的大小略小于 1KB。
> 
> 

下表显示了此项的典型操作的请求单位大概费用（假定帐户一致性级别设置为“会话”且所有项都自动编制索引）：

| 操作 | 请求单位费用 |
| --- | --- |
| 创建项 |~15 RU |
| 读取项 |~1 RU |
| 按 ID 查询项 |~2.5 RU |

此外，此表还显示了应用程序所用的典型查询的请求单位大概费用：

| 查询 | 请求单位费用 | 返回的项数 |
| --- | --- | --- |
| 按 ID 选择食品 |~2.5 RU |1 |
| 按制造商选择食品 |~7 RU |7 |
| 按食品组选择并按重量排序 |~70 RU |100 |
| 选择食品组中的前 10 个食品 |~10 RU |10 |

> [AZURE.NOTE]
> RU 费用取决于返回的项数。
> 
> 

使用此信息，我们可以根据预计的每秒操作和查询数量来估计此应用程序的 RU 需求：

| 操作/查询 | 每秒的估计数量 | 所需的 RU |
| --- | --- | --- |
| 创建项 |10 |150 |
| 读取项 |100 |100 |
| 按制造商选择食物 |25 |175 |
| 按食品组进行选择 |10 |700 |
| 选择前 10 个 |15 |总计 150 |

在此示例中，我们认为平均吞吐量需求为 1,275 RU/s。  舍入到最接近的百位数，我们会将此应用程序的集合设置为 1,300 RU/s。

## <a id="RequestRateTooLarge"></a> 超过 DocumentDB 中的保留吞吐量限制
前面提到，如果每分钟请求单位数已禁用或者预算为空，则请求单位消耗以每秒速率进行评估。 对于超过为集合预配的请求单位速率的应用程序，将限制对该容器的请求数，直到速率降低到保留级别之下。 受到限制时，服务器将抢先结束请求、引发 RequestRateTooLargeException（HTTP 状态代码 429）并返回 x-ms-retry-after-ms 标头，该标头指示重试请求前用户必须等待的时间（以毫秒为单位）。

    HTTP Status 429
    Status Line: RequestRateTooLarge
    x-ms-retry-after-ms :100

如果使用的是 .NET 客户端 SDK 和 LINQ 查询，则大多数情况下都不需要处理此异常，因为 .NET 客户端 SDK 的当前版本会隐式捕获此响应，并会遵循服务器指定的 retry-after 标头，然后重试请求。 除非多个客户端同时访问你的帐户，否则下次重试就会成功。

如果存在多个高于请求速率的请求操作，则默认重试行为可能无法满足需要，这时客户端就会向应用程序引发 DocumentClientException，其状态代码为 429。 在这种情况下，你可以考虑处理重试行为和应用程序错误处理例程中的逻辑，或为容器增加保留的吞吐量。

## <a id="RequestRateTooLargeAPIforMongoDB"></a>API for MongoDB 中超过保留的吞吐量限制
超过为集合预配的请求单位数的应用程序将受到限制，直到比率下降到保留级别以下。 受限制时，后端将提前结束请求并返回 *16500* 错误代码 -“请求过多”。 默认情况下，在返回“请求过多”错误代码之前，API for MongoDB 将自动重试最多 10 次。 如果收到大量的“请求过多”错误代码，可以考虑在应用程序的错误处理例程中添加重试行为，或者[提高集合的保留吞吐量](/documentation/articles/documentdb-set-throughput/)。

## <a name="next-steps"></a>后续步骤
若要了解有关 DocumentDB 数据库的保留吞吐量的详细信息，请浏览以下资源：

- [DocumentDB 定价](/pricing/details/documentdb/)
- [将 DocumentDB 中的数据分区](/documentation/articles/documentdb-partition-data/)

有关 DocumentDB 的详细信息，请参阅 DocumentDB [文档](/documentation/services/documentdb/)。 

若要开始使用 DocumentDB 进行规模和性能测试，请参阅[使用 DocumentDB 进行性能和规模测试](/documentation/articles/documentdb-performance-testing/)。

[1]: ./media/documentdb-request-units/queryexplorer.png 
[2]: ./media/documentdb-request-units/RUEstimatorUpload.png
[3]: ./media/documentdb-request-units/RUEstimatorDocuments.png
[4]: ./media/documentdb-request-units/RUEstimatorResults.png
[5]: ./media/documentdb-request-units/RUCalculator2.png
[6]: ./media/documentdb-request-units/api-for-mongodb-metrics.png
<!-- Update_Description: wording update -->
