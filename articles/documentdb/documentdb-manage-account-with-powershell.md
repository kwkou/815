<properties
    pageTitle="Azure DocumentDB 自动化 - 使用 Powershell 管理 | Azure"
    description="使用 Azure Powershell 管理 DocumentDB 数据库帐户。DocumentDB 是用于 JSON 数据的云端 NoSQL 数据库。"
    services="documentdb"
    author="dmakwana"
    manager="jhubbard"
    editor=""
    tags="azure-resource-manager"
    documentationcenter="" />
<tags
    ms.assetid=""
	ms.service: documentdb
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/27/2016"
    wacn.date="01/12/2017"
    ms.author="dimakwan" />  


# 使用 Azure Powershell 自动执行 Azure DocumentDB 帐户管理
>[AZURE.SELECTOR]
[Azure portal](/documentation/articles/documentdb-create-account/)
[Azure CLI and ARM](/documentation/articles/documentdb-automation-resource-manager-cli/)
[Azure Powershell](/documentation/articles/documentdb-manage-account-with-powershell/)

以下指南介绍了使用 Azure Powershell 自动执行 DocumentDB 数据库帐户管理的命令。它还包括用于管理[多区域数据库帐户][scaling-globally]的帐户密钥和故障转移优先级的命令。通过更新数据库帐户，可以修改一致性策略和添加/删除区域。若要实现 DocumentDB 数据库帐户的跨平台管理，可以使用 [Azure CLI](/documentation/articles/documentdb-automation-resource-manager-cli/)、[资源提供程序 REST API][rp-rest-api] 或 [Azure 门户预览](/documentation/articles/documentdb-create-account/)。

## 入门

请按照[如何安装和配置 Azure PowerShell][powershell-install-configure] 中的说明在 PowerShell 中安装 Azure资源管理器帐户并登录到该帐户。

### 说明

- 如果想要执行以下命令而无需用户确认，请向命令追加 `-Force` 标志。
- 以下所有命令都是同步执行的。

## <a id="create-documentdb-account-powershell"></a> 创建 DocumentDB 数据库帐户

此命令可创建 DocumentDB 数据库帐户。使用特定[一致性策略](/documentation/articles/documentdb-consistency-levels/)将新的数据库帐户配置为单区域或[多区域][scaling-globally]。

    $locations = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0}, @{"locationName"="<read-region-location>"; "failoverPriority"=1})
    $consistencyPolicy = @{"defaultConsistencyLevel"="<default-consistency-level>"; "maxIntervalInSeconds"="<max-interval>"; "maxStalenessPrefix"="<max-staleness-prefix>"}
    $DocumentDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy}
    New-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName <resource-group-name>  -Location "<resource-group-location>" -Name <database-account-name> -PropertyObject $DocumentDBProperties
    
- `<write-region-location>` 数据库帐户的写入区域位置名称。此位置的故障转移优先级值需要为 0。每个数据库帐户必须有且只有一个写入区域。
- `<read-region-location>` 数据库帐户的读取区域位置名称。此位置的故障转移优先级值需要大于 0。每个数据库帐户可以有多个读取区域。
- `<default-consistency-level>` DocumentDB 帐户的默认一致性级别。有关详细信息，请参阅 [Consistency Levels in DocumentDB](/documentation/articles/documentdb-consistency-levels/)（DocumentDB 中的一致性级别）。
- `<max-interval>` 与有限过期一致性一起使用时，此值表示允许的过期时间量（以秒为单位）。此值的接受范围为 1-100。
- `<max-staleness-prefix>` 与有限过期一致性一起使用时，此值表示允许的过期请求数。此值的接受范围为 1 - 2,147,483,647。
- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<resource-group-location>` 新的 DocumentDB 数据库帐户所属的 Azure 资源组的位置。
- `<database-account-name>` 要创建的 DocumentDB 数据库帐户的名称。它只能使用小写字母、数字及“-”字符，且长度必须为 3 到 50 个字符。

示例：

    $locations = @(@{"locationName"="China North"; "failoverPriority"=0}, @{"locationName"="China East"; "failoverPriority"=1})
    $consistencyPolicy = @{"defaultConsistencyLevel"="BoundedStaleness"; "maxIntervalInSeconds"=5; "maxStalenessPrefix"=100}
    $DocumentDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy}
    New-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Location "China North" -Name "docdb-test" -PropertyObject $DocumentDBProperties

### 说明
- 前面的示例使用两个区域创建数据库帐户。还可以使用一个区域（指定为写入区域并具有故障转移优先级值 0）或两个以上区域创建数据库帐户。有关详细信息，请参阅[多区域数据库帐户][scaling-globally]。
- 这些位置必须是已正式推出 DocumentDB 的区域。[“Azure 区域”页](https://azure.microsoft.com/regions/)提供了当前的区域列表。

## <a id="update-documentdb-account-powershell"></a> 更新 DocumentDB 数据库帐户

此命令可更新 DocumentDB 数据库帐户属性。这包括一致性策略和数据库帐户所在的位置。

> [AZURE.NOTE]
此命令可添加和删除区域，但不可修改故障转移优先级。若要修改故障转移优先级，请参阅[下文](#modify-failover-priority-powershell)。

    $locations = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0}, @{"locationName"="<read-region-location>"; "failoverPriority"=1})
    $consistencyPolicy = @{"defaultConsistencyLevel"="<default-consistency-level>"; "maxIntervalInSeconds"="<max-interval>"; "maxStalenessPrefix"="<max-staleness-prefix>"}
    $DocumentDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy}
    Set-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName <resource-group-name> -Name <database-account-name> -PropertyObject $DocumentDBProperties
    
- `<write-region-location>` 数据库帐户的写入区域位置名称。此位置的故障转移优先级值需要为 0。每个数据库帐户必须有且只有一个写入区域。
- `<read-region-location>` 数据库帐户的读取区域位置名称。此位置的故障转移优先级值需要大于 0。每个数据库帐户可以有多个读取区域。
- `<default-consistency-level>` DocumentDB 帐户的默认一致性级别。有关详细信息，请参阅 [Consistency Levels in DocumentDB](/documentation/articles/documentdb-consistency-levels/)（DocumentDB 中的一致性级别）。
- `<max-interval>` 与有限过期一致性一起使用时，此值表示允许的过期时间量（以秒为单位）。此值的接受范围为 1-100。
- `<max-staleness-prefix>` 与有限过期一致性一起使用时，此值表示允许的过期请求数。此值的接受范围为 1 - 2,147,483,647。
- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<resource-group-location>` 新的 DocumentDB 数据库帐户所属的 Azure 资源组的位置。
- `<database-account-name>` 要更新的 DocumentDB 数据库帐户的名称。

示例：

    $locations = @(@{"locationName"="China North"; "failoverPriority"=0}, @{"locationName"="China East"; "failoverPriority"=1})
    $consistencyPolicy = @{"defaultConsistencyLevel"="BoundedStaleness"; "maxIntervalInSeconds"=5; "maxStalenessPrefix"=100}
    $DocumentDBProperties = @{"databaseAccountOfferType"="Standard"; "locations"=$locations; "consistencyPolicy"=$consistencyPolicy}
    Set-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -PropertyObject $DocumentDBProperties

## <a id="delete-documentdb-account-powershell"></a> 删除 DocumentDB 数据库帐户

此命令可删除现有 DocumentDB 数据库帐户。

    Remove-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"
    
- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<database-account-name>` 要删除的 DocumentDB 数据库帐户的名称。

示例：

    Remove-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="get-documentdb-properties-powershell"></a> 获取 DocumentDB 数据库帐户的属性

此命令可获取现有 DocumentDB 数据库帐户的属性。

    Get-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"

- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<database-account-name>` DocumentDB 数据库帐户的名称。

示例：

    Get-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="update-tags-powershell"></a> 更新 DocumentDB 数据库帐户的标记

以下示例说明如何为 DocumentDB 数据库帐户设置 [Azure 资源标记][azure-resource-tags]。

> [AZURE.NOTE]
通过为相应参数追加 `-Tags` 标志，此命令可以与创建或更新命令组合使用。

示例：

    $tags = @{"dept" = "Finance”; environment = “Production”}
    Set-AzureRmResource -ResourceType “Microsoft.DocumentDB/databaseAccounts”  -ResourceGroupName "rg-test" -Name "docdb-test" -Tags $tags

## <a id="list-account-keys-powershell"></a> 列出帐户密钥

当创建 DocumentDB 帐户时，服务生成两个主访问密钥，用于访问 DocumentDB 帐户时的身份验证。通过提供两个访问密钥，DocumentDB 允许你在不中断 DocumentDB 帐户连接的情况下重新生成密钥。还提供用于对只读操作进行身份验证的只读密钥。有两个读写密钥（主密钥和辅助密钥）和两个只读密钥（主密钥和辅助密钥）。

    $keys = Invoke-AzureRmResourceAction -Action listKeys -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>"

- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<database-account-name>` DocumentDB 数据库帐户的名称。

示例：

    $keys = Invoke-AzureRmResourceAction -Action listKeys -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test"

## <a id="regenerate-account-key-powershell"></a> 重新生成帐户密钥

你应定期将访问密钥更改为你的 DocumentDB 帐户，使你的连接更安全。分配了两个访问密钥，你可以使用一个访问密钥保持与 DocumentDB 帐户的连接，同时，你可以重新生成另一个访问密钥。

    Invoke-AzureRmResourceAction -Action regenerateKey -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>" -Parameters @{"keyKind"="<key-kind>"}

- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<database-account-name>` DocumentDB 数据库帐户的名称。
- `<key-kind>` 要重新生成的四种类型的密钥之一：["Primary"|"Secondary"|"PrimaryReadonly"|"SecondaryReadonly"]。

示例：

    Invoke-AzureRmResourceAction -Action regenerateKey -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -Parameters @{"keyKind"="Primary"}

## <a id="modify-failover-priority-powershell"></a> 修改 DocumentDB 数据库帐户的故障转移优先级

对于多区域数据库帐户，可以更改 DocumentDB 数据库帐户所在的各个区域的故障转移优先级。有关 DocumentDB 数据库帐户中的故障转移的详细信息，请参阅[使用 DocumentDB 全局分发数据][distribute-data-globally]。

    $failoverPolicies = @(@{"locationName"="<write-region-location>"; "failoverPriority"=0},@{"locationName"="<read-region-location>"; "failoverPriority"=1})
    Invoke-AzureRmResourceAction -Action failoverPriorityChange -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "<resource-group-name>" -Name "<database-account-name>" -Parameters @{"failoverPolicies"=$failoverPolicies}

- `<write-region-location>` 数据库帐户的写入区域位置名称。此位置的故障转移优先级值需要为 0。每个数据库帐户必须有且只有一个写入区域。
- `<read-region-location>` 数据库帐户的读取区域位置名称。此位置的故障转移优先级值需要大于 0。每个数据库帐户可以有多个读取区域。
- `<resource-group-name>` 新的 DocumentDB 数据库帐户所属的 [Azure 资源组][azure-resource-groups]的名称。
- `<database-account-name>` DocumentDB 数据库帐户的名称。

示例：

    $failoverPolicies = @(@{"locationName"="China East"; "failoverPriority"=0},@{"locationName"="China North"; "failoverPriority"=1})
    Invoke-AzureRmResourceAction -Action failoverPriorityChange -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName "rg-test" -Name "docdb-test" -Parameters @{"failoverPolicies"=$failoverPolicies}

<!--Reference style links - using these makes the source content way more readable than using inline links-->

[powershell-install-configure]: https://docs.microsoft.com/zh-cn/azure/powershell-install-configure
[scaling-globally]: /documentation/articles/documentdb-distribute-data-globally/#scaling-across-the-planet/
[distribute-data-globally]: https://docs.microsoft.com/zh-cn/azure/documentdb/documentdb-distribute-data-globally
[azure-resource-groups]: https://docs.microsoft.com/zh-cn/azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-resource-tags]: https://docs.microsoft.com/zh-cn/azure/azure-resource-manager/resource-group-using-tags
[rp-rest-api]: https://docs.microsoft.com/zh-cn/rest/api/documentdbresourceprovider/

<!---HONumber=Mooncake_0103_2017-->
