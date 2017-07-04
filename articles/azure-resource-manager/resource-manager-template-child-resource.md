<properties
    pageTitle="在 Azure 模板中定义子资源 | Azure"
    description="演示如何在 Azure Resource Manager 模板中设置子资源的资源类型和名称"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/02/2017"
    wacn.date="03/31/2017"
    ms.author="tomfitz" />  


# 在 Resource Manager 模板中设置子资源的名称和类型
创建模板时，需频繁地包括与父资源相关的子资源。例如，模板可能包括 SQL Server 和数据库。SQL Server 是父资源，数据库是子资源。

子资源类型的格式为：`{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`

子资源名称的格式为：`{parent-resource-name}/{child-resource-name}`

但是，在模板中指定资源类型和名称的方式并不相同，具体取决于该资源是嵌套在父资源中，还是独立存在于顶级目录中。本主题演示如何使用这两种方法。

## 嵌套式子资源
若要定义子资源，最简单的方式是将其嵌套在父资源中。以下示例演示了嵌套在 SQL Server 中的 SQL 数据库。

    {
      "name": "exampleserver",
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2014-04-01",
      ...
      "resources": [
        {
          "name": "exampledatabase",
          "type": "databases",
          "apiVersion": "2014-04-01",
          ...
        }
      ]
    }

对于子资源来说，类型设置为 `databases`，但其资源类型的完整名称为 `Microsoft.Sql/servers/databases`。不提供 `Microsoft.Sql/servers/` 是因为它取自父资源类型。子资源名称设置为 `exampledatabase`，但完整名称包括父名称。不提供 `exampleserver` 是因为它取自父资源。

## 顶级子资源
可以定义顶级子资源。使用此方法的前提是：父资源未在同一模板中部署，或者你需要使用 `copy` 创建多个子资源。使用此方法时，必须提供完整的资源类型，并将父资源名称包括在子资源名称中。

    {
      "name": "exampleserver",
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2014-04-01",
      "resources": [ 
      ],
      ...
    },
    {
      "name": "exampleserver/exampledatabase",
      "type": "Microsoft.Sql/servers/databases",
      "apiVersion": "2014-04-01",
      ...
    }

数据库是服务器的子资源，即使二者是在模板的同一级别定义的。

## 后续步骤
* 有关如何创建模板的建议，请参阅 [Best practices for creating Azure Resource Manager templates](/documentation/articles/resource-manager-template-best-practices/)（创建 Azure Resource Manager 模板的最佳做法）。
* 如需创建多个子资源的示例，请参阅[在 Azure Resource Manager 模板中部署资源的多个实例](/documentation/articles/resource-group-create-multiple/)。

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description:new article about how to add child resource in template-->