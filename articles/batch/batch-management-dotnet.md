<properties
    pageTitle="使用适用于 .NET 的客户端库管理 Batch 帐户资源 - Azure | Azure"
    description="使用 Batch Management .NET 库创建、删除和修改 Azure Batch 帐户资源。"
    services="batch"
    documentationcenter=".net"
    author="tamram"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="16279b23-60ff-4b16-b308-5de000e4c028"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="03/15/2017"
    wacn.date="04/24/2017"
    ms.author="tamram"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="e7be25b2dd6fecfd88a6f8deb6b1ef333f6bdf87"
    ms.lasthandoff="04/14/2017" />

# <a name="manage-batch-accounts-and-quotas-with-the-batch-management-client-library-for-net"></a>通过用于 .NET 的 Batch Management 客户端库管理 Batch 帐户和配额
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/batch-account-create-portal/)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet/)

可以使用 [Batch Management .NET][api_mgmt_net] 库来自动化 Batch 帐户的创建、删除、密钥管理和配额发现，从而降低 Azure Batch 应用程序的维护开销。

- 在任何区域中**创建和删除 Batch 帐户**。 例如，如果你是一家独立软件供应商 (ISV)，现在要为每个分配了不同计费 Batch 帐户的客户提供服务，则你可以将帐户创建和删除功能添加到客户门户中。
- 以编程方式为任何 Batch 帐户**检索和重新生成帐户密钥**。 这样可确保遵循安全策略，以便定期滚动更新帐户密钥或令其过期。 当各种不同的 Azure 区域中有多个 Batch 帐户时，将此滚动更新过程自动化会提高解决方案的效率。
- **检查帐户配额** 并采取试错猜测，确定哪些 Batch 帐户存在哪些限制。 在启动作业、创建池或添加计算节点之前检查帐户配额可以主动调整创建计算资源的位置或时机。 可在帐户中分配其他资源之前，确定哪些帐户需要增加配额。
- 通过在同一应用程序中使用 Batch Management .NET、[Azure Active Directory][aad_about] 和 [Azure资源管理器][resman_overview]，用户可以**结合其他 Azure 服务的功能**获得全功能管理体验。 使用这些功能及其 API 可以提供顺畅的身份验证体验、创建和删除资源组以及上述功能，以获取端到端管理解决方案。

> [AZURE.NOTE]
> 尽管本文着重介绍以编程方式管理 Batch 帐户、密钥和配额，但你也可以使用 [Azure 门户][azure_portal]执行其中的许多活动。 有关详细信息，请参阅[使用 Azure 门户创建 Azure 批处理帐户](/documentation/articles/batch-account-create-portal/)以及 [Azure 批处理服务的配额和限制](/documentation/articles/batch-quota-limit/)。
> 
> 

## <a name="create-and-delete-batch-accounts"></a>创建和删除 Batch 帐户
如上所述，Batch Management API 的主要功能之一就是在 Azure 区域中创建和删除 Batch 帐户。 为此，请使用 [BatchManagementClient.Account.CreateAsync][net_create] 和 [DeleteAsync][net_delete]，或其同步等效命令。

以下代码段将创建一个帐户，从 Batch 服务获取新建的帐户，然后将它删除。 在此代码片段以及本文的其他代码片段中，`batchManagementClient` 是完全初始化的 [BatchManagementClient][net_mgmt_client] 实例。

    // Create a new Batch account
    await batchManagementClient.Account.CreateAsync("MyResourceGroup",
        "mynewaccount",
        new BatchAccountCreateParameters() { Location = "China North" });

    // Get the new account from the Batch service
    AccountResource account = await batchManagementClient.Account.GetAsync(
        "MyResourceGroup",
        "mynewaccount");

    // Delete the account
    await batchManagementClient.Account.DeleteAsync("MyResourceGroup", account.Name);

> [AZURE.NOTE]
> 使用 Batch Management .NET 库及其 BatchManagementClient 类的应用程序需有**服务管理员**或**共同管理员**访问权限才能使用拥有要管理的 Batch 帐户的订阅。 有关详细信息，请参阅 [Azure Active Directory](#azure-active-directory) 部分和 [AccountManagement][acct_mgmt_sample] 代码示例。
> 
> 

## <a name="retrieve-and-regenerate-account-keys"></a>检索和重新生成帐户密钥
使用 [ListKeysAsync][net_list_keys]从订阅中的任何 Batch 帐户获取主要和辅助帐户密钥。 可以使用 [RegenerateKeyAsync][net_regenerate_keys] 重新生成这些密钥。

    // Get and print the primary and secondary keys
    BatchAccountListKeyResult accountKeys =
        await batchManagementClient.Account.ListKeysAsync(
            "MyResourceGroup",
            "mybatchaccount");
    Console.WriteLine("Primary key:   {0}", accountKeys.Primary);
    Console.WriteLine("Secondary key: {0}", accountKeys.Secondary);

    // Regenerate the primary key
    BatchAccountRegenerateKeyResponse newKeys =
        await batchManagementClient.Account.RegenerateKeyAsync(
            "MyResourceGroup",
            "mybatchaccount",
            new BatchAccountRegenerateKeyParameters() {
                KeyName = AccountKeyType.Primary
                });

> [AZURE.TIP]
> 可以为管理应用程序创建简化的连接工作流。 首先，获取想要使用 [ListKeysAsync][net_list_keys] 管理的批处理帐户的帐户密钥。 然后在初始化 Batch .NET 库的 [BatchSharedKeyCredentials][net_sharedkeycred] 类（初始化 [BatchClient][net_batch_client] 时使用）时使用此密钥。
> 
> 

## <a name="check-azure-subscription-and-batch-account-quotas"></a>检查 Azure 订阅和 Batch 帐户配额
Azure 订阅和类似于 Batch 的各个 Azure 服务均有默认配额，用于限制其中特定实体的数目。 有关 Azure 订阅的默认配额，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)。 有关 Batch 服务的默认配额，请参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/)。 使用 Batch 管理 .NET 库可以在应用程序中检查这些配额。 这样，你就可以在添加帐户或计算资源（如池和计算节点）之前做出分配决策。

### <a name="check-an-azure-subscription-for-batch-account-quotas"></a>检查 Azure 订阅和 Batch 帐户配额
在区域中创建 Batch 帐户之前，可以检查 Azure 订阅，看是否能将帐户添加到该区域中。

在以下的代码片段中，我们先使用 [BatchManagementClient.Account.ListAsync][net_mgmt_listaccounts] 来获取订阅中所有 Batch 帐户的集合。 获取此集合后，可以确定目标区域有多少个帐户。 然后使用 [BatchManagementClient.Subscriptions][net_mgmt_subscriptions] 来获取批处理帐户配额，并确定可以在该区域中创建多少个帐户（如果有）。

    // Get a collection of all Batch accounts within the subscription
    BatchAccountListResponse listResponse =
            await batchManagementClient.Account.ListAsync(new AccountListParameters());
    IList<AccountResource> accounts = listResponse.Accounts;
    Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}",
        creds.SubscriptionId,
        accounts.Count);

    // Get a count of all accounts within the target region
    string region = "chinanorth";
    int accountsInRegion = accounts.Count(o => o.Location == region);

    // Get the account quota for the specified region
    SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Subscriptions.GetSubscriptionQuotasAsync(region);
    Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

    // Determine how many accounts can be created in the target region
    Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
    Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);

在上面的代码片段中，`creds` 是 [TokenCloudCredentials][azure_tokencreds] 的实例。 若要查看创建此对象的示例，请参阅 GitHub 上的 [AccountManagement][acct_mgmt_sample] 代码示例。

### <a name="check-a-batch-account-for-compute-resource-quotas"></a>检查 Batch 帐户的计算资源配额
增加 Batch 解决方案中的计算资源之前，可以通过检查来确保需要分配的资源不超过帐户的配额。 在以下代码片段中，将输出名为 `mybatchaccount`的 Batch 帐户的配额信息。 在用户自己的应用程序中，可以使用此类信息来确定帐户是否可以处理要创建的其他资源。

    // First obtain the Batch account
    BatchAccountGetResponse getResponse =
        await batchManagementClient.Account.GetAsync("MyResourceGroup", "mybatchaccount");
    AccountResource account = getResponse.Resource;

    // Now print the compute resource quotas for the account
    Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
    Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
    Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);

> [AZURE.IMPORTANT]
> 尽管 Azure 订阅和服务有默认配额，但许多限制都可以通过在 [Azure 门户][azure_portal]中提出请求来提高。 例如，可以参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/)以获取有关提高 Batch 帐户配额的说明。
> 
> 

## <a name="use-azure-ad-with-batch-management-net"></a>将 Azure AD 和 Batch 管理 .NET 配合使用

Batch 管理 .NET 库是 Azure 资源提供程序客户端，与 [Azure资源管理器][resman_overview] 配合使用以编程方式管理帐户资源。 Azure AD 需要对通过任何 Azure 资源提供程序客户端（包括 Batch 管理 .NET 库）和 [Azure资源管理器][resman_overview] 发出的请求进行身份验证。

## GitHub 上的示例项目 <a name="sample"></a>

查看 GitHub 上的 [AccountManagment][acct_mgmt_sample] 示例项目，了解 Batch 管理 .NET 的操作实践。 AccountManagment 示例应用程序演示了以下操作：

1. 使用 [ADAL][aad_adal] 从 Azure AD 获取安全令牌。 如果用户尚未登录，系统将提示其输入 Azure 凭据。
2. 使用从 Azure AD 获取的安全令牌创建 [SubscriptionClient][resman_subclient]，以便在 Azure 中查询与帐户关联的订阅列表。 如果列表包含多个订阅，则用户可从中选择一个订阅。
3. 获取与所选订阅关联的凭据。
4. 使用凭据创建 [ResourceManagementClient][resman_client] 对象。
5. 使用 [ResourceManagementClient][resman_client] 对象创建资源组。
6. 使用 [BatchManagementClient][net_mgmt_client] 对象执行多个 Batch 帐户操作：
   - 在新资源组中创建 Batch 帐户。
   - 从 Batch 服务获取新建的帐户。
   - 输出新帐户的帐户密钥。
   - 重新生成帐户的新主密钥。
   - 输出帐户的配额信息。
   - 输出订阅的配额信息。
   - 输出订阅中的所有帐户。
   - 删除新建的帐户。
7. 删除该资源组。

删除新建的批处理帐户和资源组之前，可以在 [Azure 门户][azure_portal]中查看它们：

若要成功运行示例应用程序，必须首先在 Azure 管理门户中将其注册到 Azure AD 租户，并向 Azure资源管理器API 授予权限。


[aad_about]: /documentation/articles/active-directory-whatis/
[aad_adal]: /documentation/articles/active-directory-authentication-libraries/
[aad_auth_scenarios]: /documentation/articles/active-directory-authentication-scenarios/
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[api_net]: http://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_mgmt_net]: https://msdn.microsoft.com/zh-cn/library/azure/mt463120.aspx
[azure_portal]: http://portal.azure.cn
[azure_storage]: /home/features/storage/
[azure_tokencreds]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.tokencloudcredentials.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[net_batch_client]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.batchclient.aspx
[net_list_keys]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.accountoperationsextensions.listkeysasync.aspx
[net_create]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.accountoperationsextensions.createasync.aspx
[net_delete]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.accountoperationsextensions.deleteasync.aspx
[net_regenerate_keys]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.accountoperationsextensions.regeneratekeyasync.aspx
[net_sharedkeycred]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.auth.batchsharedkeycredentials.aspx
[net_mgmt_client]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.batchmanagementclient.aspx
[net_mgmt_subscriptions]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.batchmanagementclient.subscriptions.aspx
[net_mgmt_listaccounts]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.batch.iaccountoperations.listasync.aspx
[resman_api]: https://msdn.microsoft.com/zh-cn/library/azure/mt418626.aspx
[resman_client]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx
[resman_subclient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.subscriptions.subscriptionclient.aspx
[resman_overview]: /documentation/articles/resource-group-overview/

[1]: ./media/batch-management-dotnet/portal-01.png
[2]: ./media/batch-management-dotnet/portal-02.png
[3]: ./media/batch-management-dotnet/portal-03.png

<!---Update_Description: wording update -->