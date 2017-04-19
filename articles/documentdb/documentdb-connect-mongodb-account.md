<properties
    pageTitle="DocumentDB 帐户的 MongoDB 连接字符串 | Azure"
    description="了解如何使用 MongoDB 连接字符串将 MongoDB 应用连接到 Azure DocumentDB 帐户。"
    keywords="mongodb 连接字符串"
    services="documentdb"
    author="AndrewHoh"
    manager="jhubbard"
    editor=""
    documentationcenter=""
    translationtype="Human Translation" />
    
<tags
    ms.assetid="e36f7375-9329-403b-afd1-4ab49894f75e"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="anhoh"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="4ecee192ca8f56d251709c078f823106165eb804"
    ms.lasthandoff="04/07/2017" />

# <a name="connect-an-application-to-documentdb-api-for-mongodb"></a>将应用程序连接到 DocumentDB：MongoDB 的 API
了解如何使用 MongoDB 连接字符串，将 MongoDB 应用连接到 Azure DocumentDB 帐户。 将 MongoDB 应用连接到 Azure DocumentDB 数据库后，可使用 DocumentDB 数据库作为 MongoDB 应用的数据存储。 

本教程提供两个用于检索连接字符串信息的方法：

- [快速启动方法](#QuickstartConnection)，适合搭配 .NET、Node.js、MongoDB Shell、Java 和 Python 驱动程序使用。
- [自定义连接字符串方法](#GetCustomConnection)：适用于其他驱动程序。

## <a name="prerequisites"></a>先决条件

- 一个 Azure 帐户。 如果没有 Azure 帐户，请立即创建 [Azure 帐户](/pricing/1rmb-trial/)。 
- 一个 DocumentDB 帐户。 有关说明，请参阅[创建与 MongoDB 应用配合使用的 DocumentDB 帐户](/documentation/articles/documentdb-create-mongodb-account/)。

## <a id="QuickstartConnection"></a>使用快速启动方法获取 MongoDB 连接字符串
1. 在 Internet 浏览器中，登录 [Azure 门户预览](https://portal.azure.cn)。
2. 在“NoSQL (DocumentDB)”边栏选项卡中，选择 DocumentDB：MongoDB 帐户的 API。 
3. 在“帐户”边栏选项卡的“左导航”栏中，单击“快速启动”。 
4. 选择平台（*.NET 驱动程序*、*Node.js 驱动程序*、*MongoDB Shell*、*Java 驱动程序*、*Python 驱动程序*）。 如果未列出驱动程序或工具，别担心，我们持续记录了更多连接代码片段。 请在下方就想要看到的内容进行评论，并阅读 [获取帐户的连接字符串信息](#GetCustomConnection) ，了解如何创建自己的连接。
5. 将代码片段复制并粘贴到 MongoDB 应用后，即已准备就绪。

    ![“快速启动”边栏选项卡的屏幕截图](./media/documentdb-connect-mongodb-account/QuickStartBlade.png)

## <a name="GetCustomConnection"></a> 获取要自定义的 MongoDB 连接字符串
1. 在 Internet 浏览器中，登录 [Azure 门户预览](https://portal.azure.cn)。
2. 在“NoSQL (DocumentDB)”边栏选项卡中，选择 DocumentDB：MongoDB 帐户的 API。 
3. 在“帐户”边栏选项卡的“左导航”栏中，单击“连接字符串”。 
4. 此时将打开“连接字符串信息”边栏选项卡，其中显示了使用 MongoDB 驱动程序连接到帐户所需的所有信息，包括预先构造的连接字符串。

    ![连接字符串边栏选项卡的屏幕截图](./media/documentdb-connect-mongodb-account/ConnectionStringBlade.png)

## <a name="connection-string-requirements"></a>连接字符串要求
> [AZURE.IMPORTANT]
> DocumentDB 具有严格的安全要求和标准。 DocumentDB 帐户需要通过 **SSL**进行身份验证和安全通信。
>
>

请务必注意，DocumentDB 支持标准 MongoDB 连接字符串 URI 格式，但要满足几个具体的要求：DocumentDB 帐户需要通过 SSL 进行身份验证和安全通信。  因此，连接字符串格式为：

    mongodb://username:password@host:port/[database]?ssl=true

上图显示了“连接字符串”边栏选项卡中提供此字符串值的位置。

- 用户名（必需）
  - DocumentDB 帐户名
- 密码（必需）
  - DocumentDB 帐户密码
- 主机（必需）
  - DocumentDB 帐户的 FQDN
- 端口（必需）
  - 10250
- 数据库（可选）
  - 连接使用的默认数据库（如果未提供数据库，则默认数据库为“test”）
- ssl=true（必需）

以上面“连接字符串信息”中显示的帐户为例。  有效的连接字符串为：

    mongodb://contoso123:0Fc3IolnL12312asdfawejunASDF@asdfYXX2t8a97kghVcUzcDv98hawelufhawefafnoQRGwNj2nMPL1Y9qsIr9Srdw==@anhohmongo.documents.azure.com:10250/mydatabase?ssl=true

## <a name="next-steps"></a>后续步骤
- 了解如何配合 DocumentDB：MongoDB 帐户的 API [使用 MongoChef](/documentation/articles/documentdb-mongodb-mongochef/)。
- 浏览 MongoDB [示例](/documentation/articles/documentdb-mongodb-samples/)的 DocumentDB: API。

<!--Update_Description: update meta properties-->