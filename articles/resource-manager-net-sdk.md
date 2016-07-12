<properties
   pageTitle="用于 .Net 的资源管理器 SDK | Azure"
   description="概述资源管理器的 .Net SDK 身份验证和用例"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="03/10/2016"
   wacn.date="04/11/2016"/>

# Azure Resource Manager SDK for .Net  
Azure Resource Manager (ARM) 预览版 SDK 适用于多种语言和平台。每种语言中实现可通过其生态系统包管理器和 GitHub 来使用。

其中每个 SDK 中的代码是从 [Azure RESTful API 规范](https://github.com/azure/azure-rest-api-specs)生成的。这些规范是开源代码，基于 Swagger v2 规范。SDK 代码是通过名为 [AutoRest](https://github.com/azure/autorest) 的开源项目生成的。AutoRest 将这些 RESTful API 规范转换成采用多种语言的客户端库。如果你想要改进 SDK 中生成的代码的任何方面，你可以使用开放的、免费提供的、基于广泛采用 API 规范格式的整个工具集。

Azure SDK for .NET 作为一组 NuGet 包提供，可帮助你调用由 Azure Resource Manager 公开的大多数 API。如果该 SDK 未公开所需的功能，你可以轻松地将该 SDK 与在后台对 ARM REST API 的常规调用相结合。

本文档并非旨在介绍 Azure SDK for .NET、Azure ARM API 或 Visual Studio 的所有方面，而是为你提供一种快速入门方法。

可在[此处](https://github.com/dx-ted-emea/Azure-Resource-Manager-Documentation/tree/master/ARM/SDKs/Samples/Net)找到完整的可下载示例项目，以下所有代码片段均取自其中。

## 身份验证
对 ARM 的身份验证由 Azure Active Directory (AD) 处理。若要连接到任何 API，首先需要使用 Azure AD 进行身份验证，以接收可传递给每个请求的身份验证令牌。若要获取此令牌，首先需要创建所谓的 Azure AD 应用程序和一个用于登录的服务主体。有关分步说明，请参阅[创建 Azure AD 应用程序和服务主体](/documentation/articles/resource-group-create-service-principal-portal/)。

创建服务主体后，你应会获得：
* 客户端 ID (GUID)
* 客户端机密（字符串）
* 租户 ID (GUID) 或域名（字符串）

### 从代码接收 AccessToken
使用下面几行代码，只需传入你的 Azure AD 租户 ID、Azure AD 应用程序客户端 ID 和 Azure AD 应用程序客户端机密即可轻松获得身份验证令牌。保存多个请求的令牌，因为默认情况下它的有效时间为 1 小时。

```csharp
private static AuthenticationResult GetAccessToken(string tenantId, string clientId, string clientSecret)
{
    Console.WriteLine("Aquiring Access Token from Azure AD");
    AuthenticationContext authContext = new AuthenticationContext
        ("https://login.windows.net/" /* AAD URI */
            + $"{tenantId}.onmicrosoft.com" /* Tenant ID or AAD domain */);

    var credential = new ClientCredential(clientId, clientSecret);

    AuthenticationResult token = authContext.AcquireToken("https://management.azure.cn/", credential);

    Console.WriteLine($"Token: {token.AccessToken}");
    return token;
}
```

### 查询附加到经过验证的应用程序的 Azure 订阅
要执行的首要任务之一是查询哪些 Azure 订阅与刚经过验证的应用程序相关联。必须将目标订阅的订阅 ID 传递给从现在起执行的每个 API 调用。

下面的示例代码将直接使用 REST API（即不使用 Azure SDK for .NET 中的任何功能）查询 Azure API。

```csharp
async private static Task<List<string>> GetSubscriptionsAsync(string token)
{
    Console.WriteLine("Querying for subscriptions");
    const string apiVersion = "2015-01-01";

    var client = new HttpClient();
    client.BaseAddress = new Uri("https://management.azure.cn/");
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await client.GetAsync($"subscriptions?api-version={apiVersion}");

    var jsonResponse = response.Content.AsString();

    var subscriptionIds = new List<string>();
    dynamic json = JsonConvert.DeserializeObject(jsonResponse);

    for (int i = 0; i < json.value.Count; i++)
    {
        subscriptionIds.Add(json.value[i].subscriptionId.Value);
    }

    Console.WriteLine($"Found {subscriptionIds.Count} subscription(s)");
    return subscriptionIds;
}
```

请注意，我们确实从 Azure 收到 JSON 响应，然后从该响应中提取订阅 ID，以便返回 ID 列表。在本文档中对 Azure ARM API 的所有后续调用只使用单个 Azure 订阅 ID，因此，如果你的应用程序与多个订阅关联，只需挑选最适合的一个，并将其作为参数向前传递。

在此处，我们针对 Azure API 进行的每个调用将使用 Azure SDK for .NET ，因此代码看起来会略有不同。

### 将令牌包装为 TokenCredentials 对象
以下所有 API 调用都需要从 Azure AD 收到的“TokenCredentials”对象格式的令牌。只需将原始令牌作为参数传递给此类的构造函数即可轻松创建此类对象。

```csharp
var credentials = new TokenCredentials(token);
```

## 创建资源组
Azure 中的所有操作都围绕着资源组进行，因此，让我们首先创建一个资源组。常规资源和资源组由 *ResourceManagementClient* 处理，由于我们将使用以下任何更专用的管理客户端，你需要提供凭据以及订阅 ID 来确定要使用哪个订阅。

```csharp
private static async Task<ResourceGroup> CreateResourceGroupAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location)
{
    Console.WriteLine($"Creating Resource Group {resourceGroup}");
    var resourceClient = new ResourceManagementClient(credentials) { SubscriptionId = subscriptionId };
    return await resourceClient.ResourceGroups.CreateOrUpdateAsync(resourceGroup,
        new ResourceGroup
        {
            Location = location
        });
}
```

## 手动创建资源，或使用模板创建
可通过多种不同方式与 Azure Resource Manager API 交互，但执行此操作主要有两种方法：

* 手动（通过手动调用特定资源提供程序）或
* 使用 Azure Resource Manager 模板（又称为“ARM 模板”）

使用 ARM 模板具有以下优点：

* 你可以以声明方式指定你希望最终结果显示为什么样，而不是指定应如何实现
* 你不必手动处理部署的并行执行，ARM 会为你执行该操作
* 你不必为了部署 ARM 模板而学习 C# 或任何其他语言，虽然你可以使用任何语言来启动模板化部署
* 模板中使用的域特定语言（即 DSL ）是使用 JSON 生成的，使用过 JSON 的任何人都很容易理解

即使使用模板有所有这些好处，我们也将首先向你演示如何手动调用 API。

### 逐步创建虚拟机
现在我们已有订阅和资源组。如果我们想要部署虚拟机，我们需要确定哪些部件实际组成虚拟机，结果发现相当多的部件：

* 1 个或多个存储帐户，用于存储持久性磁盘
* 可从 Internet 访问的 1 个或多个公共 IP 地址，即 PIP（包括 DNS 名称）
* 1 个或多个虚拟网络（即 VNET），用于资源之间的内部通信
* 1 个或多个网络接口卡（即 NIC），用于允许 VM 进行通信
* 1 个或多个虚拟机（即 VM），用于运行软件

另一个有趣的部分也是其中一些资源可以并行创建（而其他资源却不能并行创建）的方式。例如：

* NIC，依赖于 PIP 和 VNet
* VM，依赖于 NIC 和存储帐户

你需要确保在创建了必需的依赖项之前，不会尝试实例化任何资源。本文档提供的完整[示例](https://github.com/dx-ted-emea/Azure-Resource-Manager-Documentation/tree/master/ARM/SDKs/Samples/Net)演示如何有效地并行创建资源，同时仍随时跟踪已创建的内容。

#### 创建存储帐户
需要使用存储帐户来存储虚拟机的虚拟 VHD。如果你有现有存储帐户，则可以将它用于多个 VM，但请记住使用多个存储帐户分担负载不要达到限制。请记住，存储帐户的类型及其位置可限制可以选择的 VM 大小，因为并非所有 VM 大小在所有区域和/或对于所有存储帐户类型都可用。

```csharp
private static async Task<StorageAccount> CreateStorageAccountAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string storageAccountName, AccountType accountType = AccountType.StandardLRS)
{
    Console.WriteLine("Creating Storage Account");
    var storageClient = new StorageManagementClient(credentials) { SubscriptionId = subscriptionId };
    return await storageClient.StorageAccounts.CreateAsync(resourceGroup, storageAccountName,
        new StorageAccountCreateParameters
        {
            Location = location,
            AccountType = accountType,
        });
}
```

#### 创建公共 IP 地址（即 PIP）
公共 IP 地址是使 Azure 中的资源可从 Internet 访问的 IP 地址。创建 IP 地址时还会为你分配完全限定域名（即 FQDN），你可以使用它进行更轻松的访问。

```csharp
private static Task<PublicIPAddress> CreatePublicIPAddressAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string pipAddressName, string pipDnsName)
{
    Console.WriteLine("Creating Public IP");
    var networkClient = new NetworkManagementClient(credentials) { SubscriptionId = subscriptionId };
    var createPipTask = networkClient.PublicIPAddresses.CreateOrUpdateAsync(resourceGroup, pipAddressName,
        new PublicIPAddress
        {
            Location = location,
            DnsSettings = new PublicIPAddressDnsSettings { DomainNameLabel = pipDnsName },
            PublicIPAllocationMethod = "Dynamic" // This sample doesn't support Static IP Addresses but could be extended to do so
        });

    return createPipTask;
}
```

#### 创建虚拟网络（即 VNET）
使用 ARM API 创建的每个 VM 都需要属于一个虚拟网络，即使该 VM 单独在其中，也是如此。
虚拟网络必须至少包含一个子网，但你可以划分多个子网并在其中保护资源。

```csharp
private static Task<VirtualNetwork> CreateVirtualNetworkAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string vNetName, string vNetAddressPrefix, Subnet[] subnets)
{
    Console.WriteLine("Creating Virtual Network");
    var networkClient = new NetworkManagementClient(credentials) { SubscriptionId = subscriptionId };
    var createVNetTask = networkClient.VirtualNetworks.CreateOrUpdateAsync(resourceGroup, vNetName,
        new VirtualNetwork
        {
            Location = location,
            AddressSpace = new AddressSpace(new[] { vNetAddressPrefix }),
            Subnets = subnets
        });

    return createVNetTask;
}
```

#### 创建网络接口卡（即 NIC）
网络接口卡（即 NIC）是将你的 VM 与它所在的虚拟网络连接的设备。
一个 VM 可以有多个 NIC，因此可以与多个虚拟网络相关联。本示例假定你只将你的 VM 连接到一个 VNET。

```csharp
private static Task<NetworkInterface> CreateNetworkInterfaceAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string nicName, string nicIPConfigName, PublicIPAddress pip, Subnet subnet)
{
    Console.WriteLine("Creating Network Interface");
    var networkClient = new NetworkManagementClient(credentials) { SubscriptionId = subscriptionId };
    var createNicTask = networkClient.NetworkInterfaces.CreateOrUpdateAsync(resourceGroup, nicName,
        new NetworkInterface()
        {
            Location = location,
            IpConfigurations = new[] {
                new NetworkInterfaceIPConfiguration
                {
                    Name = nicIPConfigName,
                    PrivateIPAllocationMethod = "Dynamic",
                    PublicIPAddress = pip,
                    Subnet = subnet
                }
            }
        });

    return createNicTask;
}
```

#### 创建虚拟机
最后，可以创建实际的虚拟机了。VM 直接或间接地依赖于前面创建的所有资源，因此你需要等待上述所有资源准备就绪，然后才能尝试预配 VM。
预配 VM 需要上述资源的最长时间，因此希望你的应用程序可以等待一段时间，以便可以执行此操作。

```csharp
private static async Task<VirtualMachine> CreateVirtualMachineAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string storageAccountName, string vmName, string vmSize, string vmAdminUsername, string vmAdminPassword, string vmImagePublisher, string vmImageOffer, string vmImageSku, string vmImageVersion, string vmOSDiskName, string nicId)
{
    Console.WriteLine("Creating Virtual Machine (this may take a while)");
    var computeClient = new ComputeManagementClient(credentials) { SubscriptionId = subscriptionId };
    var vm = await computeClient.VirtualMachines.CreateOrUpdateAsync(resourceGroup, vmName,
        new VirtualMachine
        {
            Location = location,
            HardwareProfile = new HardwareProfile(vmSize),
            OsProfile = new OSProfile(vmName, vmAdminUsername, vmAdminPassword),
            StorageProfile = new StorageProfile(
                new ImageReference
                {
                    Publisher = vmImagePublisher,
                    Offer = vmImageOffer,
                    Sku = vmImageSku,
                    Version = vmImageVersion
                },
                new OSDisk
                {
                    Name = vmOSDiskName,
                    Vhd = new VirtualHardDisk($"http://{storageAccountName}.blob.core.windows.net/vhds/{vmOSDiskName}.vhd"),
                    Caching = "ReadWrite",
                    CreateOption = "FromImage"
                }),
            NetworkProfile = new NetworkProfile(
                new[] { new NetworkInterfaceReference { Id = nicId } }),
            DiagnosticsProfile = new DiagnosticsProfile(
                new BootDiagnostics
                {
                    Enabled = true,
                    StorageUri = $"http://{storageAccountName}.blob.core.chinacloudapp.cn"
                })
        });

    return vm;
}
```

### 使用模板化部署
有关如何部署模板的详细说明，请阅读并遵循[使用 .NET 库和模板部署 Azure 资源](/documentation/articles/virtual-machines-windows-csharp-template/#step-4-create-the-credentials-that-are-used-to-authenticate-requests)教程。

简单地说，部署模板比手动预配资源要容易得多，以下代码演示如何通过指向包含模板和参数文件的 URI 来执行此操作。

```csharp
private static async Task<DeploymentExtended> CreateTemplatedDeployment(TokenCredentials credentials, string subscriptionId, string resourceGroup, string templateUri, string parametersUri)
{
    var resourceClient = new ResourceManagementClient(credentials) { SubscriptionId = subscriptionId };

    return await resourceClient.Deployments.BeginCreateOrUpdateAsync(resourceGroup, "mytemplateddeployment", new Deployment(
        new DeploymentProperties()
        {
            Mode = DeploymentMode.Incremental,
            TemplateLink = new TemplateLink(templateUri),
            ParametersLink = new ParametersLink(parametersUri)
        }));

}
```


 
   

<!---HONumber=Mooncake_0405_2016-->