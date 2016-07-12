<properties 
	pageTitle="DocumentDB 的 MongoDB 协议支持 | Azure" 
	description="了解 DocumentDB 的 MongoDB 协议支持（目前以公共预览版提供）。" 
	keywords="mongodb"
	services="documentdb" 
	authors="stephbaron" 
	manager="jhubbard" 
	editor="" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="05/31/2016" 
	wacn.date="06/29/2016"/>

# DocumentDB 的 MongoDB 协议支持

## 什么是 Azure DocumentDB？
Azure DocumentDB 是一个完全托管的 NoSQL 数据库服务，其构建目的是为了实现快速且可预测的性能、高可用性、自动扩展和易于开发。该数据库的灵活的数据模型、一贯的低延迟和丰富的查询功能使其非常适合用于 Web、移动、游戏和 IoT 应用程序，以及其他许多需要无缝扩展的应用程序。有关详细信息，请阅读 [DocumentDB 简介](/documentation/articles/documentdb-introduction/)。

## 什么是 DocumentDB 的 MongoDB 协议支持？
现在可以将 DocumentDB 数据库用作为 MongoDB 编写的应用的数据存储。使用 MongoDB 的现有[驱动程序](https://docs.mongodb.org/ecosystem/drivers/)，在许多情况下，只需更改连接字符串，应用程序就能以透明方式轻松与 DocumentDB 通信。使用此预览功能，客户可以在 Azure 云中轻松构建和运行应用程序 - 利用 DocumentDB 的完全托管且可缩放的 NoSQL 数据库 - 同时继续使用熟悉的技能和 MongoDB 工具。

核心 MongoDB API 函数可以借助 DocumentDB 的 MongoDB 协议支持来创建、读取、更新和删除 (CRUD) 数据以及查询数据库。当前实现的功能已根据常用平台、框架、工具和想要评估 Azure 是否适合用作云平台的广大 MongoDB 客户群体的需求设置了优先级。
  

## 后续步骤


- 了解如何[创建](/documentation/articles/documentdb-create-mongodb-account/)具有 MongoDB 协议支持的 DocumentDB 帐户。
- 了解如何[连接](/documentation/articles/documentdb-connect-mongodb-account/)到具有 MongoDB 协议支持的 DocumentDB 帐户。
- 了解如何配合具有 MongoDB 协议支持的 DocumentDB 帐户[使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

 

<!---HONumber=Mooncake_0627_2016-->