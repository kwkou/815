<properties
    pageTitle="Azure CLI 脚本示例 - 创建 Azure Redis 缓存 | Azure"
    description="Azure CLI 脚本示例 - 创建 Azure Redis 缓存"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="afd7f6e0-9297-4c98-a95e-597be939cef7"
    ms.service="cache-redis"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="04/14/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="a87e3f84151ef383f7696b2494357239e4a53d77"
    ms.lasthandoff="04/22/2017" />

# <a name="create-an-azure-redis-cache"></a>创建 Azure Redis 缓存

此方案介绍如何创建 Azure Redis 缓存。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Creates a Resource Group named contosoGroup, and creates a Redis Cache in that group named contosoCache

    # Create a Resource Group 
    az group create --name contosoGroup --location chinaeast

    # Create a Redis Cache
    az redis create --name contosoCache --resource-group contosoGroup --location chinaeast --sku-capacity 0 --sku-family C --sku-name Basic


[AZURE.INCLUDE [cli-script-clean-up](../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组和 Redis 缓存。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az redis create](https://docs.microsoft.com/zh-cn/cli/azure/redis#create) | 创建 Redis 缓存实例。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Redis 缓存文档](/documentation/articles/cli-samples/)中找到其他 Azure Redis 缓存 CLI 脚本示例。