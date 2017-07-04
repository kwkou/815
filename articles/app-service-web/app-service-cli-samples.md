<properties
    pageTitle="Azure CLI 示例 - 应用服务 | Azure"
    description="Azure CLI 示例 - 应用服务"
    services="app-service"
    documentationcenter="app-service"
    author="syntaxc4"
    manager="erikre"
    editor="ggailey777"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="53e6a15a-370a-48df-8618-c6737e26acec"
    ms.service="app-service"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="app-service"
    ms.date="03/08/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="81886dd418d0c0f69a54d31bed23aa19b44f8776"
    ms.lasthandoff="04/14/2017" />

# <a name="azure-cli-samples"></a>Azure CLI 示例

下表包含指向使用 Azure CLI 生成的 bash 脚本的链接。

| | |
|-|-|
|**创建应用**||
| [从 GitHub 创建 Web 应用并部署代码](/documentation/articles/app-service-cli-deploy-github/)| 从公共 GitHub 存储库创建 Azure Web 应用并部署代码。 |
| [从本地 Git 存储库创建 Web 应用并部署代码](/documentation/articles/app-service-cli-deploy-local-git/) | 从本地 Git 存储库创建 Azure Web 应用并配置代码推送。 |
| [创建 Web 应用并将代码部署到过渡环境](/documentation/articles/app-service-cli-deploy-staging-environment/) | 使用用于暂存代码更改的部署槽创建 Azure Web 应用。 |
|**配置应用**||
| [将自定义域映射到 Web 应用](/documentation/articles/app-service-cli-configure-custom-domain/)| 创建 Azure Web 应用并将自定义域名映射到它。 |
| [将自定义 SSL 证书绑定到 Web 应用](/documentation/articles/app-service-cli-configure-ssl-certificate/)| 创建 Azure Web 应用并将自定义域名的 SSL 证书绑定到它。 |
|**缩放应用**||
| [手动缩放 Web 应用](/documentation/articles/app-service-cli-scale-manual/) | 创建 Azure Web 应用并将其在 2 个实例之间进行缩放。 |
| [缩放具有高可用性体系结构的全国性 Web 应用](/documentation/articles/app-service-cli-scale-high-availability/) | 在两个不同地理区域中创建两个 Azure Web 应用，并使用 Azure 流量管理器通过单个终结点使其可用。 |
|**将应用连接到资源**||
| [将 Web 应用连接到 SQL 数据库](/documentation/articles/app-service-cli-app-service-sql/)| 创建 Azure Web 应用和 SQL 数据库，然后将数据库连接字符串添加到应用设置。 |
| [将 Web 应用连接到存储帐户](/documentation/articles/app-service-cli-app-service-storage/)| 创建 Azure Web 应用和存储帐户，然后将存储连接字符串添加到应用设置。 |
| [将 Web 应用连接到 Redis 缓存](/documentation/articles/app-service-cli-app-service-redis/) | 创建 Azure Web 应用和 Redis 缓存，然后将 Redis 连接详细信息添加到应用设置。 |
| [将 Web 应用连接到 documentdb](/documentation/articles/app-service-cli-app-service-documentdb/) | 创建 Azure Web 应用和 documentdb，然后将 documentdb 连接详细信息添加到应用设置。 |
|**监视应用**||
| [使用 Web 服务器日志监视 Web 应用](/documentation/articles/app-service-cli-monitor/) | 创建 Azure Web 应用，为其启用日志记录，并将日志下载到本地计算机。 |
| | |