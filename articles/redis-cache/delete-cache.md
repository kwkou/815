<properties
    pageTitle="Azure CLI 脚本示例 - 删除 Azure Redis 缓存 | Azure"
    description="Azure CLI 脚本示例 - 删除 Azure Redis 缓存"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="7beded7a-d2c9-43a6-b3b4-b8079c11de4a"
    ms.service="cache-redis"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="04/14/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="d96c11ada878b23873bb96772b43dfa808d8756d"
    ms.lasthandoff="04/22/2017" />

# <a name="delete-an-azure-redis-cache"></a>删除 Azure Redis 缓存

此方案介绍如何删除 Azure Redis 缓存。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Delete a Redis Cache named contosoCache from the Resource Group contosoGroup
    az redis delete --name contosoCache --resource-group contosoGroup


[AZURE.INCLUDE [cli-script-clean-up](../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令删除 Azure Redis 缓存实例。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az redis delete](https://docs.microsoft.com/zh-cn/cli/azure/redis#delete) | 删除 Redis 缓存实例。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Redis 缓存文档](/documentation/articles/cli-samples/)中找到其他 Azure Redis 缓存 CLI 脚本示例。