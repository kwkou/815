<properties
    pageTitle="在 Azure DocumentDB 中使用日期 | Azure"
    description="了解如何在 Azure DocumentDB 中使用日期。"
    services="documentdb"
    author="arramac"
    manager="jhubbard"
    editor="mimig"
    documentationcenter="" />
<tags
    ms.assetid="e587772f-ce9f-498c-a017-a51e7265bb23"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/21/2016"
    wacn.date="02/27/2017"
    ms.author="arramac" />  


# 在 Azure DocumentDB 中使用日期
DocumentDB 通过本机 [JSON](http://www.json.org) 数据模型提供架构灵活性和丰富的索引。所有 DocumentDB 资源（包括数据库、集合、文档和存储过程）均作为 JSON 文档进行建模和存储。JSON（和 DocumentDB）仅支持一小部分基本类型：字符串、数字、布尔值、数组、对象和 Null。这是对可移植性的要求。但是，JSON 具有相当的灵活性，允许开发人员和框架使用这些基元并将其编写为对象或数组，以便表示更复杂的类型。

除了基本类型，许多应用程序还需要表示日期和时间戳的 \[DateTime\]\(https://msdn.microsoft.com/zh-cn/library/system.datetime(v=vs.110).aspx\) 类型。本文介绍开发人员如何在 DocumentDB 中使用 .NET SDK 存储、检索和查询日期。

## 存储 DateTime
默认情况下，[DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/) 会将 DateTime 值序列化为 [ISO 8601](http://www.iso.org/iso/catalogue_detail?csnumber=40874) 字符串。由于以下原因，大多数应用程序可以使用 DateTime 的默认字符串表示形式：

- 字符串可以进行比较，而 DateTime 值的相对顺序在这些值转换为字符串时得以保留。
- 此方法不需要进行 JSON 转换所需的任何自定义代码或属性。
- 在 JSON 中存储的日期人工可读。
- 这种方法可以利用 DocumentDB 的索引执行快速查询。

例如，以下代码片段使用 .NET SDK 将 `Order` 对象存储为文档，该对象包含两个 DateTime 属性 - `ShipDate` 和 `OrderDate`：

    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await client.CreateDocumentAsync("/dbs/orderdb/colls/orders", 
        new Order 
        { 
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14), 
            Total = 113.39
        });

此文档存储在 DocumentDB 中，如下所示：

    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
    

也可将 DateTime 存储为 Unix 时间戳，即存储为数字，用于表示自 1970 年 1 月 1 日以来已过去的秒数。DocumentDB 的内部时间戳 \(`_ts`\) 属性遵循这种方法。可以使用 [UnixDateTimeConverter](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.documents.unixdatetimeconverter.aspx) 类将 DateTime 序列化为数字。

## 编制 DateTime 的索引以执行范围查询
通常可以使用 DateTime 值执行范围查询。例如，如果需要查找昨天以来创建的所有订单，或者查找过去五分钟内发运的所有订单，则要执行范围查询。若要有效执行这些查询，必须针对字符串的范围索引配置集合。

    DocumentCollection collection = new DocumentCollection { Id = "orders" };
    collection.IndexingPolicy = new IndexingPolicy(new RangeIndex(DataType.String) { Precision = -1 });
    await client.CreateDocumentCollectionAsync("/dbs/orderdb", collection);

如需详细了解如何配置索引策略，可参阅 [DocumentDB Indexing Policies](/documentation/articles/documentdb-indexing-policies/)（DocumentDB 索引策略）。

## 在 LINQ 中查询 Datetime
DocumentDB .NET SDK 自动支持通过 LINQ 查询存储在 DocumentDB 中的数据。例如，以下代码片段显示一个 LINQ 查询，该查询筛选在过去三天中发运的订单。

    IQueryable<Order> orders = client.CreateDocumentQuery<Order>("/dbs/orderdb/colls/orders")
        .Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
          
    // Translated to the following SQL statement and executed on DocumentDB
    SELECT * FROM root WHERE (root["ShipDate"] >= "2016-12-18T21:55:03.45569Z")

可以在 [Querying DocumentDB](/documentation/articles/documentdb-sql-query/)（查询 DocumentDB）中详细了解 DocumentDB 的 SQL 查询语言和 LINQ 提供程序。

本文探讨了如何在 DocumentDB 中存储和查询 DateTime 并为其编制索引。

## 后续步骤
- 下载并运行 [Github 上的代码示例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/code-samples)
- 详细了解 [DocumentDB 查询](/documentation/articles/documentdb-sql-query/)
- 详细了解 [DocumentDB 索引策略](/documentation/articles/documentdb-indexing-policies/)

<!---HONumber=Mooncake_0220_2017-->
