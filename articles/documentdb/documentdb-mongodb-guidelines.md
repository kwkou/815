<properties 
	pageTitle="具有 MongoDB 协议支持的 DocumentDB 帐户开发指南（预览版）| Azure" 
	description="了解具有 MongoDB 协议支持的 DocumentDB 帐户（目前以预览版提供）的预览版开发指南。" 
	services="documentdb" 
	authors="andrewhoh" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/15/2016" 
	ms.author="anhoh"
   	wacn.date="10/18/2016"/>  


# 具有 MongoDB 协议支持的 DocumentDB 帐户开发指南

可通过任何开源 MongoDB 客户端[驱动程序](https://docs.mongodb.org/ecosystem/drivers/)来与 Azure DocumentDB 通信。对 MongoDB 的协议支持假设 MongoDB 客户端驱动程序与 MongoDB 2.6 或更高版本的服务器终结点通信。DocumentDB 通过遵守 MongoDB [线路协议](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol/) 2.6 版支持此功能。

DocumentDB 支持使用核心 MongoDB API 函数来创建、读取、更新和删除 (CRUD) 数据以及查询数据库。实现的功能已根据常见平台、架构、工具和应用程序模式的需求设置优先级。

## 集合

> [AZURE.IMPORTANT] DocumentDB 在集合级别使用保留的吞吐量来提供有保证的可预测性能。因此，DocumentDB 中的集合是可计费实体。

性能保留在集合级别应用，使应用程序可以调整系统中数据容器的最低级别性能。因此，集合的价格由集合的预配吞吐量（以每秒的请求单位数来度量）和总占用存储空间（以千兆字节为单位）决定。可以在集合的整个生命周期内调整设置的吞吐量，以适应不断变化的应用程序的处理需求和访问模式。有关详细信息，请参阅 [DocumentDB performance levels](/documentation/articles/documentdb-performance-levels/)（DocumentDB 性能级别）。

默认情况下，具有 MongoDB 协议支持的 DocumentDB 集合是以 1,000 RU/秒的预配吞吐量在标准定价层创建的。

## CRUD 操作

完全支持 MongoDB 插入、读取、更新、替换和删除操作。这包括支持 MongoDB [更新运算符规范](https://docs.mongodb.org/manual/reference/operator/update/)指定的字段、数组、位和隔离更新。对于需要多个文档操作的更新运算符，DocumentDB 提供了完整的 ACID 语义与快照隔离。

## 查询操作

DocumentDB 支持 MongoDB 查询的完整语法，但有一些例外情况。除了支持 MongoDB 日期时间格式，还支持对 JSON 兼容的 [BSON 类型](https://docs.mongodb.org/manual/reference/bson-types/)集运行查询。对于需要非 JSON 类型特定运算符的查询，DocumentDB 支持 GUID 数据类型。

## 门户体验
启用 MongoDB 协议的帐户的 Azure 门户体验迎合启用 MongoDB 协议的帐户的需要。我们正在寻求拓展该体验，但需要用户提供有关哪些门户功能最有用的[反馈](mailto:askdocdb@microsoft.com?subject=DocumentDB%20Protocol%20Support%20for%20MongoDB%20Preview%20Portal%20Experience)。

## 支持矩阵


### CRUD 和查询操作

功能|支持|即将支持
---|---|---
插入|InsertOne| 
 |InsertMany| 
 |插入| 
更新|UpdateOne| 
 |UpdateMany| 
 |更新| 
字段更新|$inc、$mul、$rename、$set、$unset、$min、$max|$currentDate| 
数组更新| |-all-
位| |-all-
隔离| |-all-
将|ReplaceOne| 
删除|DeleteOne | 
 |DeleteMany| 
 |删除| 
BulkWrite| |bulkWrite()
比较|-all-| 
逻辑|-all-| 
元素查询| |-all-
计算|$mod、$regex |$text、$where
GeoSpatial|2dsphere、2d、polygon|其他
Array|$all、$size、$elemMatch|
位| |-all-
注释|-all-| 
投影| |-all-


### 数据库命令

功能|支持|即将支持
---|---|---
聚合|计数| 
GeoSpatial| |-all-
查询和写入|find、insert、update、delete、getLastError、getMore、findAndModify| 
身份验证|getnonce、logout、authenticate| 
管理|createIndex、listIndexes、dropIndexes、connectionStatus、reIndex| 
诊断|listDatabases、collStats、dbStats| 

## 后续步骤

- 了解如何对具有 MongoDB 协议支持的 DocumentDB 帐户[使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

 

<!---HONumber=Mooncake_1010_2016-->
