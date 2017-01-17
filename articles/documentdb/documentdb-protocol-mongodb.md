<properties
    pageTitle="什么是 DocumentDB 的 MongoDB 协议支持？| Azure"
    description="什么是 DocumentDB 的 MongoDB 协议支持？ 借助它，可将 Azure DocumentDB（托管的基于云的服务）用作为 MongoDB 编写的应用的数据存储。"
    keywords="什么是 MongoDB"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter="" />
<tags
    ms.assetid="4afaf40d-c560-42e0-83b4-a64d94671f0a"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/23/2016"
    wacn.date="01/16/2017"
    ms.author="anhoh" />  


# 什么是 DocumentDB 的 MongoDB 协议支持？

现在可以将 DocumentDB 数据库用作为 MongoDB 编写的应用的数据存储。这意味着，通过使用 MongoDB 数据库的现有[驱动程序](https://docs.mongodb.org/ecosystem/drivers/)，为 MongoDB 编写的应用程序现在可以与 DocumentDB 通信，并能够使用 DocumentDB 数据库（而不使用 MongoDB 数据库）。在许多情况下，只需更改连接字符串，即可从使用 MongoDB 切换为使用 DocumentDB。借助此功能，客户可利用 DocumentDB 的完全托管且可缩放的 NoSQL 数据库，使用熟悉的技能和 MongoDB 工具，在 Azure 云中轻松生成和运行 MongoDB 数据库应用程序。

## 使用 DocumentDB 的 MongoDB 协议支持有什么好处？
**无服务器管理**：DocumentDB 是完全托管的服务，这意味着自己无需管理任何基础结构或虚拟机。DocumentDB 已在 20 多个 [Azure 区域](https://azure.microsoft.com/regions/)推出。

**无限缩放**：可单独灵活地缩放吞吐量和存储。可轻松添加容量，每秒为数百万个请求提供服务。

**企业级**：DocumentDB 支持多个本地副本，可在本地或区域出现故障时，提供 99.99% 的可用性和数据保护。DocumentDB 具有企业级[合规性认证](https://www.microsoft.com/trustcenter)和安全功能。

**MongoDB 兼容性**：DocumentDB 的 MongoDB 协议支持专为兼容 MongoDB 而设计。可将现有代码、应用程序、驱动程序和工具与 DocumentDB 配合使用。

## 如何入门？
在 [Azure 门户预览](https://portal.azure.cn)中创建具有 MongoDB 协议支持的 DocumentDB 帐户，并将连接交换到新帐户。

*就这么简单！*

有关更多详细说明，请参阅[创建帐户](/documentation/articles/documentdb-create-mongodb-account/)和[连接到帐户](/documentation/articles/documentdb-connect-mongodb-account/)。

## 后续步骤
- 请参阅[创建具有 MongoDB 协议支持的 DocumentDB 帐户](/documentation/articles/documentdb-create-mongodb-account/)教程，了解如何创建 DocumentDB 帐户。
- 请参阅[连接到具有 MongoDB 协议支持的 DocumentDB 帐户](/documentation/articles/documentdb-connect-mongodb-account/)教程，了解如何获取帐户连接字符串信息。
- 请参阅[将 MongoChef 与具有 MongoDB 协议支持的 DocumentDB 帐户配合使用](/documentation/articles/documentdb-mongodb-mongochef/)教程，了解如何在 MongoChef 中的 DocumentDB 数据库和 MongoDB 应用之间创建连接。
- 浏览具有 MongoDB 协议支持的 DocumentDB [示例](/documentation/articles/documentdb-mongodb-samples/)。

<!---HONumber=Mooncake_0109_2017-->
