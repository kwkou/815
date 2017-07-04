<properties
    pageTitle="设计规模化的 Azure 虚拟机规模集 | Azure"
    description="了解如何设计规模化的 Azure 虚拟机规模集"
    keywords="linux 虚拟机, 虚拟机规模集"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="gatneil"
    manager="madhana"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="c27c6a59-a0ab-4117-a01b-42b049464ca1"
    ms.service="virtual-machine-scale-sets"
    ms.workload="na"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/13/2017"
    wacn.date="04/17/2017"
    ms.author="negat"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="c488f6bd675bb5746700bb83550741129b25053e"
    ms.lasthandoff="04/06/2017" />

# <a name="designing-vm-scale-sets-for-scale"></a>设计规模化的 VM 规模集
本主题讨论虚拟机规模集的设计注意事项。 有关什么是虚拟机规模集的信息，请参阅[虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)。

## <a name="storage"></a>存储

### <a name="user-managed-storage"></a>用户管理的存储
使用 Azure 非托管磁盘定义的规模集依赖于用户创建的存储帐户在规模集中存储 VM 的 OS 磁盘。 建议为每个存储帐户预配不超过 20 个 VM 的比率，以实现最大 IO，并充分利用_过度预配_（请参见下文）。 还建议存储帐户名称的开始部分字符采用字母表中的不同字符。 这样做有助于在多个不同的内部系统中分散负载。 

## <a name="overprovisioning"></a>预配过度
从 "2016-03-30" API 版本开始，VM 规模集默认设置为“过度预配”VM。 开启过度预配时，规模集实际预配的 VM 数量超过所要求的数量，在成功预配所请求数量的 VM 后删除额外的 VM。 过度预配可提高预配成功率和减少部署时间。 这些额外的 VM 不会计费，并且不会计入配额限制。

虽然过度预配可以提高预配成功率，但对于不是用于处理时而出现时而消失的额外 VM 的应用程序，会导致其行为混乱。 为了关闭预配过度，请确保你的模板中包含以下字符串："overprovision": "false"。 可在 [VM 规模集 REST API 文档](https://msdn.microsoft.com/zh-cn/library/azure/mt589035.aspx)找到更多详细信息。

如果规模集使用用户管理的存储，并且关闭了过度预配，则可为每个存储帐户预配超过 20 个 VM，但是出于 IO 性能考虑，建议不要超过 40 个 VM。 

## <a name="limits"></a>限制

使用用户管理的存储帐户配置的规模集目前限制为 100 个 VM（建议为此规模使用 5 个存储帐户）。

基于自定义映像（用户构建的映像）构建的规模集具有 100 个 VM 的容量。 如果规模集配置了用户管理的存储帐户，则必须在同一存储帐户中创建所有 OS 磁盘 VHD。 因此，基于自定义映像和用户管理的存储构建的规模集中 VM 的最大建议数目为 20。 如果关闭预配过度，最大可为 40。

对于高出这些限制所允许的 VM，需要部署多个规模集，如[此模板](https://github.com/Azure/azure-quickstart-templates/tree/master/301-custom-images-at-scale)所示。
<!--Update_Description: wording update-->