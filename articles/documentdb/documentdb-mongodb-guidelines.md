<properties 
	pageTitle="具有 MongoDB 协议支持的 DocumentDB 帐户开发指南（预览版）| Azure" 
	description="了解具有 MongoDB 协议支持的 DocumentDB 帐户（目前以预览版提供）的预览版开发指南。" 
	services="documentdb" 
	authors="stephbaron" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="05/31/2016" 
	wacn.date="08/18/2016"/>

# 具有 MongoDB 协议支持的 DocumentDB 帐户开发指南（预览版）

可通过任何开源 MongoDB 客户端[驱动程序](https://docs.mongodb.org/ecosystem/drivers/)来与 Azure DocumentDB 通信。对 MongoDB 的协议支持假设 MongoDB 客户端驱动程序与 MongoDB 2.6 或更高版本的服务器终结点通信。DocumentDB 通过遵守 MongoDB [线路协议](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol/) 2.6 版来提供此支持（请注意，线路协议 3.2 版几乎完全受支持，但某些客户端体验，例如 3.2 版 MongoDB shell 会话，可能会指出它们“即将降级到‘旧’模式”）。

DocumentDB 支持使用核心 MongoDB API 函数来创建、读取、更新和删除 (CRUD) 数据以及查询数据库。实现的功能已根据常见平台、架构、工具和应用程序模式的需求设置优先级。

## 集合

> [AZURE.IMPORTANT] DocumentDB 在集合级别使用保留的吞吐量来提供有保证的可预测性能。因此，DocumentDB 中的集合是可计费实体。

性能保留在集合级别应用，使应用程序可以调整系统中数据容器的最低级别性能。因此，集合的成本由集合的设置的吞吐量（以每秒的请求单位数来衡量）和总占用存储空间（以千兆字节为单位）决定。可以在集合的整个生命周期内调整设置的吞吐量，以适应不断变化的应用程序的处理需求和访问模式。有关详细信息，请参阅 [DocumentDB performance levels（DocumentDB 性能级别）](documentdb-performance-levels.md)。

默认情况下，具有 MongoDB 协议支持的 DocumentDB 集合是以 1,000 RU/秒的预配吞吐量在标准定价层创建的。可以根据 [Changing performance levels using the Azure Portal（使用 Azure 门户更改性能级别）](/documentation/articles/documentdb-performance-levels/#changing-performance-levels-using-the-azure-portal)中所述调整每个集合的预配吞吐量。

## CRUD 操作

完全支持 MongoDB 插入、读取、更新、替换和删除操作。这包括支持 MongoDB [更新运算符规范](https://docs.mongodb.org/manual/reference/operator/update/)指定的字段、数组、位和隔离更新。对于需要多个文档操作的更新运算符，DocumentDB 提供了完整的 ACID 语义与快照隔离。

## 查询操作

DocumentDB 支持 MongoDB 查询的完整语法，但有一些例外情况。除了支持 MongoDB 日期时间格式，还支持对 JSON 兼容的 [BSON 类型](https://docs.mongodb.org/manual/reference/bson-types/)集运行查询。对于需要非 JSON 类型特定运算符的查询，DocumentDB 支持 GUID 数据类型。下表提供了支持与不支持的 MongoDB 查询语法。

## 聚合

DocumentDB 不支持 MongoDB 聚合管道或 Map-Reduce 操作。聚合管道通常用于通过多阶段筛选和转换集处理文档，例如匹配和分组结果。DocumentDB 本机支持 JavaScript 用户定义的函数和存储过程。此外，DocumentDB 可以通过 DocumentDB [Hadoop 连接器](/documentation/articles/documentdb-run-hadoop-with-hdinsight/)使用 Azure HDInsight，轻松充当 Hive、Pig 和 MapReduce 操作的源和接收器。

## 门户体验
已启用 MongoDB 协议的帐户的 Azure 门户体验与标准 DocumentDB 帐户的门户体验稍有不同。我们正在寻求拓展体验，但目前希望你提供有关哪个门户功能对你最有用的[反馈](mailto:askdocdb@microsoft.com?subject=DocumentDB%20Protocol%20Support%20for%20MongoDB%20Preview%20Portal%20Experience)。

## 支持矩阵


### CRUD 和查询操作

功能|支持|即将支持|不支持 
---|---|---|---
插入|InsertOne| | 
 |InsertMany| | 
 |插入| | 
更新|UpdateOne| | 
 |UpdateMany| | 
 |更新| | 
字段更新|$inc、$mul、$rename、$set、$unset、$min、$max|$currentDate| 
数组更新| |-all-| 
位| |-all-| 
隔离| |-all-| 
将|ReplaceOne| |
删除|DeleteOne | |
 |DeleteMany| | 
 |删除| | 
BulkWrite| |bulkWrite()| 
比较|-all-| | 
逻辑|-all-| | 
元素查询| |-all-| 
计算|$mod|$regex、$text、$where| 
GeoSpatial|2dsphere、2d、polygon|其他| 
Array|$all、$size|$elemMatch| 
位| |-all-| 
注释|-all-| | 
投影| |-all-| 


### 数据库命令

功能|支持|即将支持|不支持 
---|---|---|---
聚合|计数| |aggregate、distinct、group、mapreduce
GeoSpatial| |-all-| 
查询和写入|find、insert、update、delete、getLastError、getMore、findAndModify| |Eval、parallelCollectionScan、getPrevError、resetError
QueryPlan 缓存| | |-all-
身份验证|getnonce、logout、authenticate| |Copydbgetnone、authschemaUpgrade
用户管理| | |-all-
角色管理| | |-all-
复制| | |-all-
管理|createIndex、listIndexes、dropIndexes、connectionStatus、reIndex| |其他命令对于索引，不支持 Unique、expireAfterSeconds、storageEngine、weights、default\_language、textIndexVersion、min、max、bucketSize
诊断|listDatabases、collStats、dbStats| |其他

## 后续步骤

- 了解如何配合具有 MongoDB 协议支持的 DocumentDB 帐户[使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

 

<!---HONumber=Mooncake_0725_2016-->