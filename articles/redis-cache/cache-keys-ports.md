<properties
    pageTitle="Azure CLI 脚本示例 - 获取 Azure Redis 缓存的主机名、端口和密钥 | Azure"
    description="Azure CLI 脚本示例 - 获取 Azure Redis 缓存实例的主机名、端口和密钥"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="761eb24e-2ba7-418d-8fc3-431153e69a90"
    ms.service="cache"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="04/14/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="8f3c01ff7ef37eedae66496ab4891bfb2cb7023a"
    ms.lasthandoff="04/22/2017" />

# <a name="get-the-hostname-ports-and-keys-for-azure-redis-cache"></a>获取 Azure Redis 缓存的主机名、端口和密钥

本方案介绍如何检索用于连接到 Azure Redis 缓存实例的主机名、端口和密钥。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Retrieve the hostname, ports, and keys for contosoCache located in contosoGroup

    # Retrieve the hostname and ports for an Azure Redis Cache instance
    redis=($(az redis show --name contosoCache --resource-group contosoGroup --query [hostName,enableNonSslPort,port,sslPort] --output tsv))

    # Retrieve the keys for an Azure Redis Cache instance
    keys=($(az redis list-keys --name contosoCache --resource-group contosoGroup --query [primaryKey,secondaryKey] --output tsv))

    # Display the retrieved hostname, keys, and ports
    echo "Hostname:" ${redis[0]}
    echo "Non SSL Port:" ${redis[2]}
    echo "Non SSL Port Enabled:" ${redis[1]}
    echo "SSL Port:" ${redis[3]}
    echo "Primary Key:" ${keys[0]}
    echo "Secondary Key:" ${keys[1]}

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令检索 Azure Redis 缓存实例的主机名、密钥和端口。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az redis show](https://docs.microsoft.com/zh-cn/cli/azure/redis#show) | 检索 Azure Redis 缓存实例的详细信息。 |
| [az redis list-keys](https://docs.microsoft.com/zh-cn/cli/azure/redis#list-keys) | 检索 Azure Redis 缓存实例的访问密钥。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Redis 缓存文档](/documentation/articles/cli-samples/)中找到其他 Azure Redis 缓存 CLI 脚本示例。