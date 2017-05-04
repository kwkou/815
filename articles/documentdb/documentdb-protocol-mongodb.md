<properties
    pageTitle="什么是 DocumentDB：MongoDB 的 API？ | Azure"
    description="Learn about DocumentDB: API for MongoDB and how you can easily run existing MongoDB applications in the Azure cloud"
    keywords="什么是 MongoDB"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid="4afaf40d-c560-42e0-83b4-a64d94671f0a"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/14/2017"
    wacn.date="05/02/2017"
    ms.author="anhoh"
    ms.sourcegitcommit="75890c3ffb1d1757de64a8b8344e9f2569f26273"
    ms.openlocfilehash="ed854d2b3b01a88e429dc297bbfa2220d9b40d75"
    ms.lasthandoff="04/25/2017" />

# <a name="what-is-documentdb-api-for-mongodb"></a>什么是 DocumentDB：MongoDB 的 API？

现在可以将 DocumentDB 数据库用作为 MongoDB 编写的应用的数据存储。 这意味着，通过使用 MongoDB 数据库的现有 [驱动程序](https://docs.mongodb.org/ecosystem/drivers/) ，为 MongoDB 编写的应用程序现在可以与 DocumentDB 通信，并能够使用 DocumentDB 数据库（而不使用 MongoDB 数据库）。 在许多情况下，只需更改连接字符串，即可从使用 MongoDB 切换为使用 DocumentDB。 使用此功能，客户可以在 Azure 云中轻松生成和运行 MongoDB 数据库应用程序（利用 DocumentDB 的完全托管且可缩放的 NoSQL 数据库），并且可继续使用熟悉的技能和 MongoDB 工具。

## <a name="what-is-the-benefit-of-using-documentdb-api-for-mongodb"></a>使用 DocumentDB：MongoDB 的 API 有什么好处？
**无服务器管理** - DocumentDB 是完全托管的服务，这意味着无需自己管理任何基础结构或虚拟机。 DocumentDB 在 30 多个 [Azure 区域](https://azure.microsoft.com/regions/services/)中可用。

**无限缩放** - 可以独立和弹性缩放吞吐量和存储。 可轻松添加容量，每秒为数百万个请求提供服务。

**企业级** ：DocumentDB 支持多个本地副本，可在本地或区域出现故障时，提供 99.99% 的可用性和数据保护。 DocumentDB 具有企业级[合规性认证](https://www.microsoft.com/trustcenter)和安全功能。 

**MongoDB 兼容性** - DocumentDB：MongoDB 的 API 旨在提供与 MongoDB 的兼容性。 可将现有代码、应用程序、驱动程序和工具与 DocumentDB 配合使用。 

## <a name="how-to-get-started"></a>如何入门？
在 [Azure 门户预览](https://portal.azure.cn)中创建 DocumentDB：MongoDB 帐户的 API，并将连接交换到新帐户。 

*这就是所有的操作！*

有关详细说明，请参阅[创建帐户](/documentation/articles/documentdb-create-mongodb-account/)和[连接到帐户](/documentation/articles/documentdb-connect-mongodb-account/)。

## <a name="next-steps"></a>后续步骤

有关 DocumentDB：MongoDB 的 API 的信息已集成到整个 DocumentDB 文档中，但以下是几个帮助你入门的指南：
- 若要了解如何获取帐户连接字符串信息，请参阅[连接到 MongoDB 帐户](/documentation/articles/documentdb-connect-mongodb-account/)教程。
- 若要了解如何在 MongoChef 中创建 DocumentDB 数据库和 MongoDB 应用之间的连接，请参阅[将 MongoChef 与 DocumentDB 配合使用](/documentation/articles/documentdb-mongodb-mongochef/)教程。
- 按照[将数据迁移到具有 MongoDB 协议支持的 DocumentDB](/documentation/articles/documentdb-mongodb-migrate/) 教程将数据导入到 MongoDB 的 API 数据库。
- 使用 [Node.js](/documentation/articles/documentdb-mongodb-samples/) 生成第一个 MongoDB 的 API 应用。
- 使用 .[NET](/documentation/articles/documentdb-mongodb-application/) 生成第一个 MongoDB 的 API Web 应用。
- 使用 [Robomongo](/documentation/articles/documentdb-mongodb-robomongo/) 连接到 MongoDB 的 API 帐户。
- 了解你的操作将多少 RU 用于 [GetLastRequestStatistics 命令和 Azure 门户预览指标](/documentation/articles/documentdb-request-units/#GetLastRequestStatistics/)。
- 了解如何[配置全局分布的应用的读取首选项](/documentation/articles/documentdb-distribute-data-globally/#ReadPreferencesAPIforMongoDB/)。

<!---Update_Description: wording update -->