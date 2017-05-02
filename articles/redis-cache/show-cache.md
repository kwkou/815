<properties
    pageTitle="Azure CLI 脚本示例 - 获取 Azure Redis 缓存的详细信息 | Azure"
    description="Azure CLI 脚本示例 - 获取 Azure Redis 缓存的详细信息"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="155924e6-00d5-4a8c-ba99-5189f300464a"
    ms.service="cache-redis"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="04/14/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="0ae467a14ccc1586a4116a1915739ddbbb53ede6"
    ms.lasthandoff="04/22/2017" />

# <a name="get-details-of-an-azure-redis-cache"></a>获取 Azure Redis 缓存的详细信息

在此方案中，你将了解如何检索 Azure Redis 缓存实例的详细信息，包括其预配状态。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Retrieve the details for an Azure Redis Cache instance named contosoCache in the Resource Group contosoGroup
    # This script shows details such as hostname, ports, and provisioning status
    az redis show --name contosoCache --resource-group contosoGroup 

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令检索 Azure Redis 缓存实例的详细信息。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az redis show](https://docs.microsoft.com/zh-cn/cli/azure/redis#show) | 检索 Azure Redis 缓存实例的详细信息。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure Redis 缓存文档](/documentation/articles/cli-samples/)中找到其他 Azure Redis 缓存 CLI 脚本示例。