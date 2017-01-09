<properties
    pageTitle="Resource Manager 模板中的依赖关系 | Azure"
    description="介绍如何在部署期间将一个资源设置为依赖于另一个资源，以确保按正确的顺序部署资源。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="34ebaf1e-480c-4b4d-9bf6-251bd3f8f2cf"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/28/2016"
    wacn.date="01/06/2017"
    ms.author="tomfitz" />  


# 在 Azure 资源管理器模板中定义依赖关系
对于给定的资源，可能有部署资源之前必须存在的其他资源。例如，SQL Server 必须存在，才能尝试部署 SQL 数据库。可通过将一个资源标记为依赖于其他资源来定义此关系。通常会使用 **dependsOn** 元素来定义依赖关系，但也可以通过 **reference** 函数来定义。

Resource Manager 将评估资源之间的依赖关系，并根据其依赖顺序进行部署。如果资源互不依赖，Resource Manager 将并行部署资源。只需为部署在同一模板中的资源定义依赖关系。

## dependsOn
在模板中，dependsOn 元素可让你将一个资源定义为与一个或多个资源相依赖。它的值可以是一个资源名称间采用逗号进行分隔的列表。

以下示例显示了一个虚拟机规模集，该集依赖于负载均衡器、虚拟网络以及创建多个存储帐户的循环。下面的示例中未显示其他这些资源，但它们需要存在于模板的其他位置。

    {
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "name": "[variables('namingInfix')]",
      "location": "[variables('location')]",
      "apiVersion": "2016-03-30",
      "tags": {
        "displayName": "VMScaleSet"
      },
      "dependsOn": [
        "storageLoop",
        "[variables('loadBalancerName')]",
        "[variables('virtualNetworkName')]"
      ],
      ...
    }

在前面的示例中，通过复制名为 **storageLoop** 的循环创建的资源包含依赖关系。有关示例，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。

定义依赖关系时，可以包含资源提供程序命名空间和资源类型，以避免多义性。例如，为明确表示可能与其他资源同名的负载均衡器和虚拟网络，可使用以下格式：

    "dependsOn": [
      "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]",
      "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
    ] 

尽管你可能倾向使用 dependsOn 来映射资源之间的关系，但请务必了解这么做的理由，因为它会影响部署性能。例如，若要记录资源的互连方式，那么，dependsOn 方法并不合适。部署之后，无法查询 dependsOn 元素中定义的资源。通过使用 dependsOn，可以影响部署时间，因为 Resource Manager 不会并行部署两个具有依赖关系的资源。若要记录资源之间的关系，请改为使用[资源链接](/documentation/articles/resource-group-link-resources/)。

## 子资源
资源属性允许指定与所定义的资源相关的子资源。子资源总共只能定义五级。请务必注意子资源和父资源之间不能创建隐式依赖关系。如果您需要在父级资源后部署子资源，则必须使用 dependsOn 属性明确声明该依赖关系。

每个父资源仅接受特定的资源类型作为子资源。可接受的资源类型在父资源的[模板架构](https://github.com/Azure/azure-resource-manager-schemas)中指定。子资源类型的名称包含父资源类型的名称，例如 **Microsoft.Web/sites/config** 和 **Microsoft.Web/sites/extensions** 都是 **Microsoft.Web/sites** 的子资源。

以下示例演示了 SQL 服务器和 SQL 数据库。请注意，在 SQL 数据库与 SQL 服务器之间定义了显式依赖关系，尽管数据库是服务器的子级。

    "resources": [
      {
        "name": "[variables('sqlserverName')]",
        "type": "Microsoft.Sql/servers",
        "location": "[resourceGroup().location]",
        "tags": {
          "displayName": "SqlServer"
        },
        "apiVersion": "2014-04-01-preview",
        "properties": {
          "administratorLogin": "[parameters('administratorLogin')]",
          "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
        },
        "resources": [
          {
            "name": "[parameters('databaseName')]",
            "type": "databases",
            "location": "[resourceGroup().location]",
            "tags": {
              "displayName": "Database"
            },
            "apiVersion": "2014-04-01-preview",
            "dependsOn": [
              "[variables('sqlserverName')]"
            ],
            "properties": {
              "edition": "[parameters('edition')]",
              "collation": "[parameters('collation')]",
              "maxSizeBytes": "[parameters('maxSizeBytes')]",
              "requestedServiceObjectiveName": "[parameters('requestedServiceObjectiveName')]"
            }
          }
        ]
      }
    ]


## 引用函数
[引用函数](/documentation/articles/resource-group-template-functions/#reference)使表达式能够从其他 JSON 名值对或运行时资源中派生其值。引用表达式隐式声明一个资源依赖于另一个资源。

    reference('resourceName').propertyPath

可以使用此元素或 dependsOn 元素来指定依赖关系，但不需要同时使用它们用于同一依赖资源。只要可能，可使用隐式引用以避免无意添加不必要的依赖关系。

若要了解详细信息，请参阅[引用函数](/documentation/articles/resource-group-template-functions/#reference)。

## 后续步骤
* 若要了解有关创建 Azure 资源管理器模板的信息，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/)。
* 有关模板的可用函数列表，请参阅[模板函数](/documentation/articles/resource-group-template-functions/)。

<!---HONumber=Mooncake_0103_2017-->