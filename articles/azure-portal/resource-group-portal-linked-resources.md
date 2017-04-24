<properties
    pageTitle="磁贴库中相关的资源与链接的资源"
    description="了解 Azure 预览门户的磁贴库中显示的相关资源和链接资源。"
    services="azure-portal"
    documentationCenter=""
    authors="adamabdelhamed"
    manager="wpickett"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="azure-portal"
    ms.date="07/16/2015"
    wacn.date="04/24/2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="827bb6d871fe501b50c33182e000a620cd2b9f9c"
    ms.lasthandoff="04/14/2017" />

# <a name="related-and-linked-resources-in-the-tile-gallery"></a>磁贴库中相关的资源与链接的资源

使用磁贴库可以在磁贴中查找特定的资源，并将资源拖放到你当前边栏选项卡中。 你可以使用磁贴库创建跨资源的管理视图。 对于任何指定的资源，相关资源包含其资源组中的所有资源，以及与资源相互链接的任何资源。

## <a name="linked-resources-in-resource-manager"></a>Resource Manager 中链接的资源
链接是 Resource Manager 的一项功能。  它可以让你声明资源之间的关系，即使它们不是驻留在相同的资源组中。 链接不会影响资源的运行时、计费，以及基于角色的访问。  它只是一个可用来表示关系的机制，使磁贴库等工具可以提供丰富的管理体验。  你的工具可以使用链接 API 来检查链接，并提供自定义的关系管理体验。 

## <a name="how-do-i-link-my-resources"></a>如何链接我的资源？

当你通过门户或者通过 Azure PowerShell 或 Azure CLI 部署模板来创建资源时，就会自动为某些依赖资源创建链接。 还可以使用[链接的资源 REST API](https://docs.microsoft.com/zh-cn/rest/api/resources/resourcelinks) 以编程方式链接资源。

## <a name="next-steps"></a>后续步骤
* 有关编写 Resource Manager 模板的简介，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要了解如何通过门户使用资源组，请参阅[使用 Azure 门户管理 Azure 资源](/documentation/articles/resource-group-portal/)。


<!--Update_Description:update meta properties and wording-->