<properties 
	pageTitle="Azure 资源管理器中的链接资源" 
	description="在 Azure 资源管理器的不同资源组中的各个资源之间创建链接。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="01/26/2016" 
	wacn.date="03/21/2016"/>

# Azure 资源管理器中的链接资源

部署后，您可能想要查询资源之间的关系或链接。依赖关系会通知部署，但其生命周期会在部署时结束。部署完成后，依赖资源之间没有任何标识关系。

相反，Azure 资源管理器提供了一个称为资源链接的新功能来建立和查询资源之间的关系。您可以确定哪些资源将链接到某个资源或哪些资源将链接自某个资源。

资源链接的范围可以是订阅、资源组或特定的资源。这意味着资源链接可以记录跨资源组的关系。当您开始将您的解决方案分解为多个模板和多个资源组时，使用某种机制来标识这些资源链接经证明是非常有用的。例如，将数据库的生命周期驻留在一个资源组中，将应用程序的不同于前者的生命周期驻留在另一个资源组中是一种常见现象。该应用程序连接到数据库时，就会在不同的资源组中的各个资源之间建立链接。

所有链接资源必须属于同一订阅。每个资源可以链接到 50 个其他资源。如果删除或移动任何链接资源，链接所有者必须清理剩余的链接。

## 模板中的链接

若要在模板中的资源之间定义链接，请参阅[资源链接 - 模板架构](/documentation/articles/resource-manager-template-links/)。

## 使用 REST API 进行链接

若要定义已部署的资源之间的链接，请运行：

    PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{provider-namespace}/{resource-type}/{resource-name}/providers/Microsoft.Resources/links/{link-name}?api-version={api-version}

将 {subscription-id} 替换为订阅 id。将 {resource-group}、{provider-namespace}、{resource-type} 和 {resource-name} 替换为标识链接中的第一个资源的值。将 {link-name} 替换为要创建的链接名称。将 2015-01-01 用于 api-version。

在该请求中，包括定义链接中第二个资源的一个对象：

    {
        "name": "{new-link-name}",
        "properties": {
            "targetId": "subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{provider-namespace}/{resource-type}/{resource-name}/",
            "notes": "{link-description}",
        }
    }

Properties 元素包含第二个资源的标识符。

有关更多示例，包括如何检索有关链接的信息，请参阅[链接资源](https://msdn.microsoft.com/zh-cn/library/azure/mt238499.aspx)。

## 后续步骤

- 您还可以使用标记来组织您的资源。若要了解有关标记资源的信息，请参阅[使用标记来组织你的资源](/documentation/articles/resource-group-using-tags/)。
- 有关如何创建模板并定义要部署的资源的说明，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/)。

<!---HONumber=Mooncake_0314_2016-->