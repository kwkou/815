<properties
    pageTitle="DocumentDB 自动化 - Azure CLI 2.0 | Microsoft 文档"
    description="使用 Azure CLI 2.0 管理 DocumentDB 数据库帐户。 DocumentDB 是用于 JSON 数据的云端 NoSQL 数据库。"
    services="documentdb"
    author="dmakwana"
    manager="jhubbard"
    editor=""
    tags="azure-resource-manager"
    documentationcenter=""
    translationtype="Human Translation" />
<tags
    ms.assetid="6158c27f-6b9a-404e-a234-b5d48c4a5b29"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.date="04/04/2017"
    wacn.date="05/08/2017"
    ms.author="dimakwan"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="2ec439e7c6790a3830a8c7303cde5013196b2f94"
    ms.lasthandoff="04/28/2017" />

# <a name="automate-azure-documentdb-account-management-using-azure-cli-20"></a>使用 Azure CLI 2.0 自动管理 Azure DocumentDB 帐户
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/documentdb-create-account/)
- [Azure CLI 1.0](/documentation/articles/documentdb-automation-resource-manager-cli-nodejs/)
- [Azure CLI 2.0](/documentation/articles/documentdb-automation-resource-manager-cli/)
- [Azure PowerShell](/documentation/articles/documentdb-manage-account-with-powershell/)

以下指南介绍了使用 Azure CLI 2.0 中提供的 DocumentDB 预览版命令自动管理 DocumentDB 数据库帐户的命令。 还介绍了用于管理[多区域数据库帐户][scaling-globally]中的帐户密钥和故障转移优先级的命令。 更新数据库帐户可以修改一致性策略以及添加/删除区域。 对于 DocumentDB 数据库帐户的跨平台管理，可使用 [Azure Powershell](/documentation/articles/documentdb-manage-account-with-powershell/)、[资源提供程序 REST API][rp-rest-api] 或 [Azure 门户预览](/documentation/articles/documentdb-create-account/)。

## <a name="getting-started"></a>入门

按照[如何安装和配置 Azure CLI 2.0][install-az-cli2] 中的说明使用 Azure CLI 2.0 设置开发环境。

执行以下命令并按照屏幕上的步骤登录到 Azure 帐户。

    az login -e AzureChinaCloud

如果目前没有[资源组](/documentation/articles/resource-group-overview/#resource-groups/)，请创建资源组：

    az group create --name <resourcegroupname> --location <resourcegrouplocation>
    az group list

`<resourcegrouplocation>` 必须是已正式推出 DocumentDB 的区域之一。 [Azure 区域页面](https://azure.microsoft.com/regions/#services)提供当前的区域列表。

### <a name="notes"></a>说明

- 执行“az documentdb -h”可获取可用命令的完整列表，或访问[参考页][az-documentdb-ref]。
- 执行“az documentdb <command> -h”可获取每个命令的必需和可选参数的详细信息列表。

## <a id="create-documentdb-account-cli"></a> 创建 DocumentDB 数据库帐户

此命令可创建 DocumentDB 数据库帐户。 可以将新数据库帐户配置为具有特定[一致性策略](/documentation/articles/documentdb-consistency-levels/)的单区域或[多区域][scaling-globally]。 

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.
        --default-consistency-level   : Default consistency level of the DocumentDB database account.
                                        Allowed values: BoundedStaleness, Eventual, Session, Strong.
        --ip-range-filter             : Firewall support. Specifies the set of IP addresses or IP
                                        address ranges in CIDR form to be included as the allowed list
                                        of client IPs for a given database account. IP addresses/ranges
                                        must be comma separated and must not contain any spaces.
        --kind                        : The type of DocumentDB database account to create.  Allowed
                                        values: GlobalDocumentDB, MongoDB, Parse.  Default:
                                        GlobalDocumentDB.
        --locations                   : Space separated locations in 'regionName=failoverPriority'
                                        format. E.g "China East"=0 "China North"=1. Failover priority values
                                        are 0 for write regions and greater than 0 for read regions. A
                                        failover priority value must be unique and less than the total
                                        number of regions. Default: single region account in the
                                        location of the specified resource group.
        --max-interval                : When used with Bounded Staleness consistency, this value
                                        represents the time amount of staleness (in seconds) tolerated.
                                        Accepted range for this value is 1 - 100.  Default: 5.
        --max-staleness-prefix        : When used with Bounded Staleness consistency, this value
                                        represents the number of stale requests tolerated. Accepted
                                        range for this value is 1 - 2,147,483,647.  Default: 100.

示例: 

    az documentdb create -g rg-test -n docdb-test
    az documentdb create -g rg-test -n docdb-test --kind MongoDB
    az documentdb create -g rg-test -n docdb-test --locations "East US"=0 "West US"=1 "South Central US"=2
    az documentdb create -g rg-test -n docdb-test --ip-range-filter "13.91.6.132,13.91.6.1/24"
    az documentdb create -g rg-test -n docdb-test --locations "East US"=0 "West US"=1 --default-consistency-level BoundedStaleness --max-interval 10 --max-staleness-prefix 200

### <a name="notes"></a>说明
- 位置必须是已正式推出 DocumentDB 的区域。 [Azure 区域页面](https://azure.microsoft.com/regions/#services)提供当前的区域列表。
- 若要启用门户访问，请在 ip-range-filter 中包含你所在区域的 Azure 门户预览的 IP 地址（按照[配置 IP 访问控制策略](/documentation/articles/documentdb-firewall-support/#configure-ip-policy/)中的指定）。

## <a id="update-documentdb-account-cli"></a> 更新 DocumentDB 数据库帐户

此命令可更新 DocumentDB 数据库帐户属性。 这包括一致性策略和数据库帐户所在的位置。

> [AZURE.NOTE]
> 此命令可添加和删除区域，但不可修改故障转移优先级。 若要修改故障转移优先级，请参阅[以下内容](#modify-failover-priority-powershell)。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.
        --default-consistency-level   : Default consistency level of the DocumentDB database account.
                                        Allowed values: BoundedStaleness, Eventual, Session, Strong.
        --ip-range-filter             : Firewall support. Specifies the set of IP addresses or IP address
                                        ranges in CIDR form to be included as the allowed list of client
                                        IPs for a given database account. IP addresses/ranges must be comma
                                        separated and must not contain any spaces.
        --locations                   : Space separated locations in 'regionName=failoverPriority' format.
                                        E.g "China East"=0. Failover priority values are 0 for write regions
                                        and greater than 0 for read regions. A failover priority value must
                                        be unique and less than the total number of regions.
        --max-interval                : When used with Bounded Staleness consistency, this value represents
                                        the time amount of staleness (in seconds) tolerated. Accepted range
                                        for this value is 1 - 100.
        --max-staleness-prefix        : When used with Bounded Staleness consistency, this value represents
                                        the number of stale requests tolerated. Accepted range for this
                                        value is 1 - 2,147,483,647.

示例: 

    az documentdb update -g rg-test -n docdb-test --locations "East US"=0 "West US"=1 "South Central US"=2
    az documentdb update -g rg-test -n docdb-test --ip-range-filter "13.91.6.132,13.91.6.1/24"
    az documentdb update -g rg-test -n docdb-test --default-consistency-level BoundedStaleness --max-interval 10 --max-staleness-prefix 200

## <a id="add-remove-region-documentdb-account-cli"></a> 在 DocumentDB 数据库帐户中添加/删除区域

若要在现有 DocumentDB 数据库帐户中添加或删除区域，请使用带 `--locations` 标志的 [update](#update-documentdb-account-cli) 命令。 以下示例演示如何创建新帐户，随后在该帐户中添加和删除区域。

示例：

    az documentdb create -g rg-test -n docdb-test --locations "East US"=0 "West US"=1
    az documentdb update -g rg-test -n docdb-test --locations "East US"=0 "North Europe"=1 "South Central US"=2


## <a id="delete-documentdb-account-cli"></a> 删除 DocumentDB 数据库帐户

此命令可删除现有 DocumentDB 数据库帐户。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.

示例：

    az documentdb delete -g rg-test -n docdb-test

## <a id="get-documentdb-properties-cli"></a> 获取 DocumentDB 数据库帐户的属性

此命令可获取现有 DocumentDB 数据库帐户的属性。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.

示例：

    az documentdb show -g rg-test -n docdb-test

## <a id="list-account-keys-cli"></a> 列出帐户密钥

当你创建 DocumentDB 帐户时，服务生成两个主访问密钥，用于访问 DocumentDB 帐户时的身份验证。 通过提供两个访问密钥，DocumentDB 允许你在不中断 DocumentDB 帐户连接的情况下重新生成密钥。 还提供用于对只读操作进行身份验证的只读密钥。 有两个读写密钥（主密钥和辅助密钥）和两个只读密钥（主密钥和辅助密钥）。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.

示例：

    az documentdb list-keys -g rg-test -n docdb-test

## <a id="list-connection-strings-cli"></a> 列出连接字符串

对于 MongoDB 帐户，可以使用以下命令检索将 MongoDB 应用连接到数据库帐户的连接字符串。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.

示例：

    az documentdb list-connection-strings -g rg-test -n docdb-test

## <a id="regenerate-account-key-cli"></a> 重新生成帐户密钥

你应定期将访问密钥更改为你的 DocumentDB 帐户，使你的连接更安全。 分配了两个访问密钥，你可以使用一个访问密钥保持与 DocumentDB 帐户的连接，同时，你可以重新生成另一个访问密钥。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.
        --key-kind          [Required]: The access key to regenerate.  Allowed values: primary, primaryReadonly,
                                        secondary, secondaryReadonly.

示例：

    az documentdb regenerate-key -g rg-test -n docdb-test --key-kind secondary

## <a id="modify-failover-priority-cli"></a> 修改 DocumentDB 数据库帐户的故障转移优先级

对于多区域数据库帐户，可以更改 DocumentDB 数据库帐户所在的各个区域的故障转移优先级。 若要深入了解 DocumentDB 数据库帐户中的故障转移，请参阅[使用 DocumentDB 全局分配数据](/documentation/articles/documentdb-distribute-data-globally/)。

    Arguments
        --name -n           [Required]: Name of the DocumentDB database account.
        --resource-group -g [Required]: Name of the resource group.
        --failover-policies [Required]: Space separated failover policies in 'regionName=failoverPriority' format.
                                        E.g "China East"=0 "China North"=1.

示例：

    az documentdb failover-priority-change "East US"=1 "West US"=0 "South Central US"=2

## <a name="next-steps"></a>后续步骤
现在你已经有了 DocumentDB 帐户，下一步是创建 DocumentDB 数据库。 你可以使用下面其中一项来创建数据库：

- Azure 门户预览，如[使用 Azure 门户预览创建 DocumentDB 集合和数据库](/documentation/articles/documentdb-create-collection/)中所述。
- C# .NET 示例，位于 GitHub 上 [azure-documentdb-dotnet](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples) 存储库的 [DatabaseManagement](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples/DatabaseManagement) 项目中。
- [DocumentDB SDK](/documentation/articles/documentdb-sdk-dotnet/)。 DocumentDB 有 .NET、Java、Python、Node.js 和 JavaScript API SDK。

创建数据库后，必须向数据库[添加一个或多个集合](/documentation/articles/documentdb-create-collection/)，然后向集合[添加文档](/documentation/articles/documentdb-view-json-document-explorer/)。


当集合中有文档后，可以使用门户中的[查询资源管理器](/documentation/articles/documentdb-query-collections-query-explorer/)、[REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn781481.aspx) 或某个 [SDK](https://msdn.microsoft.com/zh-cn/library/azure/dn781482.aspx)针对文档使用 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/) [执行查询](/documentation/articles/documentdb-sql-query/#ExecutingSqlQueries/)。

若要详细了解 DocumentDB，请浏览以下资源：

- [DocumentDB 资源模型和概念](/documentation/articles/documentdb-resources/)


<!--Reference style links - using these makes the source content way more readable than using inline links-->
[scaling-globally]:/documentation/articles/documentdb-distribute-data-globally/#scaling-across-the-planet/
[install-az-cli2]: https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2
[az-documentdb-ref]: https://docs.microsoft.com/zh-cn/cli/azure/documentdb
[az-documentdb-create-ref]: https://docs.microsoft.com/zh-cn/cli/azure/documentdb#create
[rp-rest-api]: https://docs.microsoft.com/zh-cn/rest/api/documentdbresourceprovider/

<!--Update_Description: wording update-->