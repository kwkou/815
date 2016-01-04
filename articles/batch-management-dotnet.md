<properties
	pageTitle="使用批处理 ( Batch )  Management .NET 管理帐户 | Windows Azure"
	description="使用批处理 ( Batch )  Management .NET 库在应用程序中创建、删除和修改 Azure 批处理 ( Batch ) 帐户。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="batch"
	ms.date="11/10/2015"
	wacn.date="12/31/2015"/>

# 使用批处理 ( Batch ) Management .NET 管理 Azure批处理 ( Batch ) 帐户和配额

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/batch-account-create-portal)
- [Batch Management .NET](/documentation/articles/batch-management-dotnet)

使用 [Batch Management .NET][api_mgmt_net] 库来自动化批处理 ( Batch )  帐户的创建、删除、密钥管理和配额发现可以降低 Azure 批处理 ( Batch )  应用程序的维护开销。

- 在任何区域中**创建和删除批处理 ( Batch ) 帐户**。例如，如果你是一家独立软件供应商 (ISV)，现在要为每个分配了不同计费批处理 ( Batch ) 帐户的客户提供服务，则你可以将帐户创建和删除功能添加到客户门户中。
- 以编程方式为任何批处理 ( Batch ) 帐户**检索和重新生成帐户密钥**。这尤其有利于保持符合可能对帐户密钥强制实施定期滚动更新或过期的安全策略。当各种不同的 Azure 区域中有许多批处理 ( Batch ) 帐户时，将此滚动更新过程自动化会提高解决方案的效率。
- **检查帐户配额**并采取试错猜测，确定哪些批处理 ( Batch ) 帐户存在哪些限制。在启动作业、创建池或添加计算节点之前检查帐户配额可以主动调整创建计算资源的位置或时机。你可以在帐户中分配其他资源之前，确定哪些帐户需要增加配额。
- 通过在同一个应用程序中利用批处理 ( Batch ) Management .NET、[Azure Active Directory][aad_about] 和 [Azure 资源管理器][resman_overview]，**结合其他 Azure 服务的功能**以获得全功能管理体验。使用这些功能及其 API 可以提供顺畅的身份验证体验、创建和删除资源组以及上述功能，以获取端到端管理解决方案。

> [AZURE.NOTE]尽管本文着重介绍以编程方式管理批处理 ( Batch ) 帐户、密钥和配额，但你也可以使用 [Azure 门户][azure_portal]执行其中的许多活动。有关详细信息，请参阅[在 Azure 门户中创建和管理 Azure批处理 ( Batch ) 帐户](/documentation/articles/batch-account-create-portal)以及 [Azure批处理 ( Batch ) 服务的配额和限制](/documentation/articles/batch-quota-limit)。

## 创建和删除批处理 ( Batch ) 帐户

如上所述，Batch Management API 的主要功能之一就是在 Azure 区域中创建和删除批处理 ( Batch ) 帐户。为此，你将要使用 [BatchManagementClient.Accounts.CreateAsync][net_create] 和 [DeleteAsync][net_delete]，或其同步等效命令。

以下代码段将创建一个帐户，从批处理 ( Batch ) 服务获取新建的帐户，然后将它删除。在此部分与在本文的其他代码段中，`batchManagementClient` 是完全初始化的 [BatchManagementClient][net_mgmt_client] 实例。

```
// Create a new Batch account
await batchManagementClient.Accounts.CreateAsync("MyResourceGroup",
	"mynewaccount",
	new BatchAccountCreateParameters() { Location = "China North" });

// Get the new account from the Batch service
BatchAccountGetResponse getResponse = await batchManagementClient.Accounts.GetAsync("MyResourceGroup", "mynewaccount");
AccountResource account = getResponse.Resource;

// Delete the account
await batchManagementClient.Accounts.DeleteAsync("MyResourceGroup", account.Name);
```

> [AZURE.NOTE]使用Batch Management .NET 库及其 BatchManagementClient 的应用程序需有**服务管理员**或**共同管理员**访问权限才能使用拥有要管理的批处理 ( Batch ) 帐户的订阅。有关详细信息，请参阅下面的 [Azure Active Directory](#aad) 部分和 [AccountManagement][acct_mgmt_sample] 代码示例。

## 检索和重新生成帐户密钥

使用 [ListKeysAsync][net_list_keys] 从订阅中的任何批处理 ( Batch ) 帐户获取主要和辅助帐户密钥，然后使用 [RegenerateKeyAsync][net_regenerate_keys] 重新生成这些密钥。

```
// Get and print the primary and secondary keys
BatchAccountListKeyResponse accountKeys = await batchManagementClient.Accounts.ListKeysAsync("MyResourceGroup", "mybatchaccount");
Console.WriteLine("Primary key:   {0}", accountKeys.PrimaryKey);
Console.WriteLine("Secondary key: {0}", accountKeys.SecondaryKey);

// Regenerate the primary key
BatchAccountRegenerateKeyResponse newKeys = await batchManagementClient.Accounts.RegenerateKeyAsync(
	"MyResourceGroup",
	"mybatchaccount",
	new BatchAccountRegenerateKeyParameters() { KeyName = AccountKeyType.Primary });
```

> [AZURE.TIP]可以为管理应用程序创建简化的连接工作流。首先，获取想要使用 [ListKeysAsync][net_list_keys] 管理的批处理 ( Batch ) 帐户的帐户密钥，然后在初始化批处理 ( Batch ) .NET 库的 [BatchSharedKeyCredentials][net_sharedkeycred]（初始化时 [BatchClient][net_batch_client] 时使用）时使用此密钥。

## 检查 Azure 订阅和批处理 ( Batch ) 帐户配额

Azure 订阅和类似于批处理 ( Batch ) 的各个 Azure 服务均有默认配额，用于限制其中特定实体的数目。Azure 订阅的默认配额可以在 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits)中找到，而批处理 ( Batch ) 服务的默认配额可以在 [Azure 批处理 ( Batch ) 服务的配额和限制](/documentation/articles/batch-quota-limit)中找到。Batch Management .NET 库提供在应用程序中检查这些配额的功能，让你在添加帐户或计算资源（如池和计算节点）之前做出分配决策。

### 检查 Azure 订阅和批处理 ( Batch ) 帐户配额

在区域中创建批处理 ( Batch ) 帐户之前，可以检查 Azure 订阅，以查看是否能够将帐户添加到该区域中。

在以下的代码段中，我们先使用 [BatchManagementClient.Accounts.ListAsync][net_mgmt_listaccounts] 来获取订阅中所有批处理 ( Batch ) 帐户的集合。我们获取此集合后，我们确定目标区域有多少个帐户，然后使用 [BatchManagementClient.Subscriptions][net_mgmt_subscriptions] 来获取批处理 ( Batch ) 帐户配额，并确定可以在该区域中创建多少个帐户（如果有）。

```
// Get a collection of all Batch accounts within the subscription
BatchAccountListResponse listResponse = await batchManagementClient.Accounts.ListAsync(new AccountListParameters());
IList<AccountResource> accounts = listResponse.Accounts;
Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}", creds.SubscriptionId, accounts.Count);

// Get a count of all accounts within the target region
string region = "westus";
int accountsInRegion = accounts.Count(o => o.Location == region);

// Get the account quota for the specified region
SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Subscriptions.GetSubscriptionQuotasAsync(region);
Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

// Determine how many accounts can be created in the target region
Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);
```

在上面的代码段中，`creds` 是 [TokenCloudCredentials][azure_tokencreds] 的实例。若要查看创建此对象的示例，请参阅 GitHub 上的 [AccountManagement][acct_mgmt_sample] 代码示例。

### 检查批处理 ( Batch ) 帐户的计算资源配额

在增加批处理 ( Batch ) 解决方案内的计算资源之前，你可以检查以确保想要分配的资源不会超出当前指定的帐户配额。在以下代码段中，我们只是要输出名为 `mybatchaccount` 的批处理 ( Batch ) 帐户的配额信息，但在你自己的应用程序中，你可以使用此类信息来确定帐户是否可以处理你要创建的其他资源。

```
// First obtain the Batch account
BatchAccountGetResponse getResponse = await batchManagementClient.Accounts.GetAsync("MyResourceGroup", "mybatchaccount");
AccountResource account = getResponse.Resource;

// Now print the compute resource quotas for the account
Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);
```

> [AZURE.IMPORTANT]尽管 Azure 订阅和服务有默认配额，但许多限制都可以通过在 [Azure 门户][azure_portal]中提出请求来提高。例如，可以参阅 [Azure 批处理 ( Batch ) 服务的配额和限制](/documentation/articles/batch-quota-limit)以获取有关提高批处理 ( Batch ) 帐户配额的说明。

##批处理 ( Batch ) Management .NET、AAD 和资源管理器

使用批处理 ( Batch ) Management .NET 库时，你通常会利用 [Azure Active Directory][aad_about] (AAD) 和 [Azure 资源管理器][resman_overview]的功能。下面讨论的示例项目将在演示批处理 ( Batch ) Management .NET API 时同时运用 Azure Active Directory 和资源管理器。

### <a name="aad"></a>Azure Active Directory

Azure 本身使用 Azure Active Directory (AAD) 来对其客户、服务管理员和组织用户进行身份验证。在批处理 ( Batch ) Management .NET 的上下文中，你将使用它来对订阅管理员或共同管理员进行身份验证，然后管理库便可以查询批处理 ( Batch ) 服务并执行本文中所述的操作。

在下面讨论的示例项目中，[Azure Active Directory 身份验证库][aad_adal] (ADAL) 用于提示用户输入他们的 Microsoft ID 凭据。提供服务管理员或共同管理员凭据后，应用程序便可以查询 Azure 订阅的列表，以及创建和删除资源组与批处理 ( Batch ) 帐户。

### 资源管理器

使用批处理 ( Batch ) Management .NET 库创建批处理 ( Batch ) 帐户时，通常会在[资源组][resman_overview]中创建帐户。你可使用[资源管理器 .NET][resman_api] 库中提供的 [ResourceManagementClient][resman_client] 以编程方式创建资源组，或者将帐户添加到以前使用 [Azure 门户][azure_portal]创建的现有资源组。

## <a name="sample"></a>GitHub 上的示例项目

查看 GitHub 上的 [AccountManagment][acct_mgmt_sample] 示例项目，以了解批处理 ( Batch ) Management .NET 库的操作实践。此控制台应用程序将显示 [BatchManagementClient][net_mgmt_client] 和 [ResourceManagementClient][resman_client] 的创建与使用方式，并演示两个客户端所需的 [Azure Active Directory 身份验证库][aad_adal] (ADAL) 使用方式。

> [AZURE.IMPORTANT]若要成功运行该示例应用程序，你必须先使用 Azure 管理门户将它注册到 Azure Active Directory。查看[将应用程序与 Azure Active Directory 集成][aad_integrate]中的**添加应用程序**，然后遵循该文章中的步骤，在你自己的帐户中注册示例应用程序。

该示例应用程序演示了以下操作：

1. 使用 [ADAL][aad_adal] 从 Azure Active Directory (AAD) 获取安全令牌。如果用户尚未登录，系统将提示其输入 Azure 凭据。
2. 使用从 AAD 获取的安全令牌创建 [SubscriptionClient][resman_subclient]，以便在 Azure 中查询与帐户关联的订阅列表，如果找到多个订阅，将允许用户选择其中一个。
3. 创建与所选订阅关联的新凭据对象
4. 使用新凭据创建 [ResourceManagementClient][resman_client]
5. 使用 [ResourceManagementClient][resman_client] 创建新资源组
6. 使用 [BatchManagementClient][net_mgmt_client] 可以执行许多批处理 ( Batch ) 帐户操作：
  - 在新建的资源组中创建新的批处理 ( Batch ) 帐户
  - 从批处理 ( Batch ) 服务获取新建的帐户
  - 输出新帐户的帐户密钥
  - 重新生成帐户的新主密钥
  - 输出帐户的配额信息
  - 输出订阅的配额信息
  - 输出订阅中的所有帐户
  - 删除新建的帐户
7. 删除资源组

删除新建的批处理 ( Batch ) 帐户和资源组之前，可以在 [Azure 门户][azure_portal]中检查这两项：

![显示资源组和批处理 ( Batch ) 帐户的 Azure 门户][1] <br /> *显示新资源组和批处理 ( Batch ) 帐户的 Azure 门户*

[aad_about]: /documentation/articles/active-directory-whatis
[aad_adal]: /documentation/articles/active-directory-authentication-libraries
[aad_auth_scenarios]: /documentation/articles/active-directory/active-directory-authentication-scenarios
[aad_integrate]: /documentation/articles/active-directory-integrating-applications
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_mgmt_net]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[azure_portal]: https://manage.windowsazure.cn
[azure_storage]: /services/storage/
[azure_tokencreds]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.tokencloudcredentials.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_list_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.listkeysasync.aspx
[net_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.createasync.aspx
[net_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.deleteasync.aspx
[net_regenerate_keys]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.accountoperationsextensions.regeneratekeyasync.aspx
[net_sharedkeycred]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.auth.batchsharedkeycredentials.aspx
[net_mgmt_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.aspx
[net_mgmt_subscriptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.batchmanagementclient.subscriptions.aspx
[net_mgmt_listaccounts]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.batch.iaccountoperations.listasync.aspx
[resman_api]: https://msdn.microsoft.com/library/azure/mt418626.aspx
[resman_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx
[resman_subclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.subscriptions.subscriptionclient.aspx
[resman_overview]: /documentation/articles/resource-group-overview

[1]: ./media/batch-management-dotnet/portal-01.png

<!---HONumber=Mooncake_1221_2015-->