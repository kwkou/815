<properties
    pageTitle="设置 Azure 资源的部署顺序 | Azure"
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
    ms.date="01/03/2017"
    wacn.date="03/03/2017"
    ms.author="tomfitz" />  

# 定义在 Azure Resource Manager 模板中部署资源的顺序
对于给定的资源，可能有部署资源之前必须存在的其他资源。例如，SQL Server 必须存在，才能尝试部署 SQL 数据库。可通过将一个资源标记为依赖于其他资源来定义此关系。使用 **dependsOn** 元素或 **reference** 函数定义依赖项。

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
        "[variables('loadBalancerName')]",
        "[variables('virtualNetworkName')]",
        "storageLoop",
      ],
      ...
    }

在前面的示例中，通过复制名为 **storageLoop** 的循环创建的资源包含依赖关系。有关示例，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。

定义依赖关系时，可以包含资源提供程序命名空间和资源类型，以避免多义性。例如，为明确表示可能与其他资源同名的负载均衡器和虚拟网络，可使用以下格式：

    "dependsOn": [
      "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]",
      "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
    ]

尽管你可能倾向使用 dependsOn 来映射资源之间的关系，但请务必了解这么做的理由。例如，若要记录资源的互连方式，那么，dependsOn 方法并不合适。部署之后，无法查询 dependsOn 元素中定义的资源。通过使用 dependsOn，可以影响部署时间，因为 Resource Manager 不会并行部署两个具有依赖关系的资源。若要记录资源之间的关系，请改为使用[资源链接](/documentation/articles/resource-group-link-resources/)。

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
[引用函数](/documentation/articles/resource-group-template-functions/#reference)使表达式能够从其他 JSON 名值对或运行时资源中派生其值。引用表达式隐式声明一个资源依赖于另一个资源。常规格式为：

    reference('resourceName').propertyPath

在以下示例中，CDN 终结点显式依赖于 CDN 配置文件，隐式依赖于 Web 应用。

    {
        "name": "[variables('endpointName')]",
        "type": "endpoints",
        "location": "[resourceGroup().location]",
        "apiVersion": "2016-04-02",
        "dependsOn": [
                "[variables('profileName')]"
        ],
        "properties": {
            "originHostHeader": "[reference(variables('webAppName')).hostNames[0]]",
            ...
        }

可以使用此元素或 dependsOn 元素来指定依赖关系，但不需要同时使用它们用于同一依赖资源。只要可能，可使用隐式引用以避免添加不必要的依赖项。

若要了解详细信息，请参阅[引用函数](/documentation/articles/resource-group-template-functions/#reference)。

## 关于设置依赖项的建议

在决定要设置的依赖项时，请遵循以下准则：

* 尽可能少设置依赖项。
* 将子资源设置为依赖于其父资源。
* 使用 **reference** 函数在需要共享属性的资源之间设置隐式依赖项。在已经定义隐式依赖项的情况下，请勿添加显式依赖项 (**dependsOn**)。此方法降低了设置不必要依赖项的风险。
* 如果没有其他资源提供的功能某项资源就无法**创建**，则可设置依赖项。如果资源仅在部署后进行交互，请勿设置依赖项。
* 让依赖项级联，无需对其进行显式设置。例如，虚拟机依赖于虚拟网络接口，虚拟网络接口依赖于虚拟网络和公共 IP 地址。因此，虚拟机在所有这三个资源之后部署，但请勿将虚拟机显式设置为依赖于所有这三个资源。此方法阐明了依赖顺序，在以后更改模板会更容易。
* 如果某个值可以在部署之前确定，请尝试在没有依赖项的情况下部署资源。例如，如果某个配置值需要另一资源的名称，则可能不需要依赖项。本指南并非始终适用，因为某些资源可验证其他资源是否存在。如果收到错误，请添加一个依赖项。

Resource Manager 可在模板验证过程中确定循环依赖项。如果收到的错误指出存在循环依赖项，可重新评估模板，看是否存在不需要且可删除的依赖项。如果删除依赖项不起作用，则可将一些部署操作移至子资源中来避免循环依赖项（这些子资源是在具有循环依赖项的资源之后部署的）。例如，假设要部署两个虚拟机，但必须在每个虚拟机上设置引用另一虚拟机的属性。可以按下述顺序部署这两个虚拟机：

1. vm1
2. vm2
3. vm1 上的扩展依赖于 vm1 和 vm2。扩展在 vm1 上设置的值是从 vm2 获取的。
4. vm2 上的扩展依赖于 vm1 和 vm2。扩展在 vm2 上设置的值是从 vm1 获取的。

若要了解如何评估部署顺序以及如何解决依赖项错误，请参阅[检查部署顺序](/documentation/articles/resource-manager-common-deployment-errors/#check-deployment-sequence)。

## 后续步骤
* 若要了解如何在部署期间排查依赖项故障，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。
* 若要了解有关创建 Azure 资源管理器模板的信息，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/)。
* 有关模板的可用函数列表，请参阅[模板函数](/documentation/articles/resource-group-template-functions/)。

<!---HONumber=Mooncake_0227_2017-->
<!-- Update_Description: update meta properties; wording update -->