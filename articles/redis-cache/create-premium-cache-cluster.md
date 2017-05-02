<properties
    pageTitle="Azure CLI 脚本示例 - 使用群集功能创建高级 Azure Redis 缓存 | Azure"
    description="Azure CLI 脚本示例 - 使用群集功能创建高级层 Azure Redis 缓存"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="07bcceae-2521-4fe3-b88f-ed833104ddd2"
    ms.service="cache-redis"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="04/14/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="ec0ea9a6a66f72ddef45779178245effeab71945"
    ms.lasthandoff="04/22/2017" />

# <a name="create-a-premium-azure-redis-cache-with-clustering"></a>使用群集功能创建高级 Azure Redis 缓存

本方案介绍如何创建大小为 6 GB、启用了群集功能且包含两个分片的高级层 Azure Redis 缓存。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Creates a Resource Group named contosoGroup, and creates a Premium Redis Cache with clustering in that group named contosoCache

    # Create a Resource Group 
    az group create --name contosoGroup --location chinaeast

    # Create a Redis Cache
    az redis create --name contosoCache --resource-group contosoGroup --location chinaeast --sku-capacity 1 --sku-family P --sku-name Premium --shard-count 2


[AZURE.INCLUDE [cli-script-clean-up](../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组以及启用了群集功能的高级层 Redis 缓存。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az redis create](https://docs.microsoft.com/zh-cn/cli/azure/redis#create) | 创建 Redis 缓存实例。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Redis 缓存文档](/documentation/articles/cli-samples/)中找到其他 Azure Redis 缓存 CLI 脚本示例。