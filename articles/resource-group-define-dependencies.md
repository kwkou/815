<properties
   pageTitle="在 Azure 资源管理器模板中定义依赖关系"
   description="介绍如何在部署期间将一个资源设置为依赖于另一个资源。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="mmercuri"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/15/2015"
   wa.date="10/3/2015"/>

# 在 Azure 资源管理器模板中定义依赖关系

对于给定的资源，可以有多个上游和子依赖关系，它们对于能否拓扑成功至关重要。可以使用资源的 **dependsOn** 和 **resources** 属性定义其他资源的依赖关系。此外可以使用**引用**函数指定依赖关系。

    {
        "name": "<name-of-the-resource>",
        "type": "<resource-provider-namespace/resource-type-name>",
        "apiVersion": "<supported-api-version-of-resource>",
        "location": "<location-of-resource>",
        "tags": { <name-value-pairs-for-resource-tagging> },
        "dependsOn": [ <array-of-related-resource-names> ],
        "properties": { <settings-for-the-resource> },
        "resources": { <dependent-resources> }
    }

 另外，还有些资源链接可以定义资源间的关系并且可以跨资源组支持定义这些关系。

## dependsOn

对于给定的虚拟机，您可能依赖于已成功设置的数据库资源。在另一种情况下，您可能依赖于在使用群集管理工具部署虚拟机之前在群集中安装的多个节点。

在您的模板中，dependsOn 属性提供对资源定义此依赖关系的功能。它的值可以是一个资源名称间采用逗号进行分隔的列表。将会评估资源之间的依赖关系，并按资源的依赖顺序来部署资源。如果资源不相互依赖，则会尝试并行部署资源。

尽管您可能倾向使用 dependsOn 来映射您的资源之间的依赖关系，但请务必了解您这么做的理由，因为它会影响您的部署性能。例如，如果您这么做是因为想要记录资源的互连方式，那么，dependsOn 方法并不合适。DependsOn 的生命周期仅适合于部署而不适合于部署后。一旦部署完成，就无法查询这些依赖关系。如果您无意中阻止部署引擎使用应有的并行功能，则使用 dependsOn 将会给性能带来不利影响。若要对资源之间的关系进行记录并提供查询功能，则应改用[资源链接](/documentation/articles/resource-group-link-resources)。

如果将引用函数用于获取某个资源的表示形式，则不需要此元素，因为引用对象意味着依赖于该资源。事实上，如果有一个选项要使用引用和 dependsOn，则该指南将使用引用函数并具有隐式引用。此处的基本原理同样是性能。引用将定义在模版中引用时所需的已知的隐式依赖关系。根据其状态显示，它们是相关，避免再次优化性能并避免阻止部署引擎执行不必要地避免并行性带来的潜在风险。

## 资源

资源属性允许指定与所定义的资源相关的子资源。子资源总共只能定义 5 级。请务必注意子资源和父资源之间不能创建隐式依赖关系。如果您需要在父级资源后部署子资源，则必须使用 dependsOn 属性明确声明该依赖关系。

## 引用函数

引用函数使表达式能够从其他 JSON 名值对或运行时资源中派生其值。引用表达式隐式声明一个资源依赖于另一个资源。以下 **propertyPath** 表示的属性是可选的，如果未指定数据，则引用将指向该资源。

    reference('resourceName').propertyPath

可以使用此元素或 dependsOn 元素来指定依赖关系，但不需要同时使用它们用于同一依赖资源。本指南将使用隐式引用以避免无意中使某个不必要的 dependsOn 元素阻止部署引擎执行并行部署方面操作的风险。

若要了解详细信息，请参阅[引用函数](/documentation/articles/resource-group-template-functions#reference)。

## 后续步骤

- 若要了解有关创建 Azure 资源管理器模板的信息，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates)。 
- 有关模板的可用函数列表，请参阅[模板函数](/documentation/articles/resource-group-template-functions)。

<!---HONumber=71-->