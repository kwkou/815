<properties
   pageTitle="用于 .NET 的 Resource Manager SDK | Azure"
   description="概述 Resource Manager .NET SDK 身份验证和用例"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="06/21/2016"
   wacn.date="08/15/2016"/>

# 用于 .NET 的 Azure Resource Manager SDK  
Azure Resource Manager SDK 适用于多种语言和平台。每种语言实现可通过其生态系统包管理器和 GitHub 来使用。

其中每个 SDK 中的代码是从 [Azure RESTful API 规范](https://github.com/azure/azure-rest-api-specs)生成的。这些规范是开源代码，基于 Swagger 2.0 规范。SDK 代码是通过名为 [AutoRest](https://github.com/azure/autorest) 的开源项目生成的。AutoRest 将这些 RESTful API 规范转换成采用多种语言的客户端库。如果你想要改进 SDK 中生成的代码的任何方面，则可以使用整个工具集来创建 SDK，该工具集属于开放式工具集，免费提供且基于广泛采用的 API 规范格式。

[用于 .NET 的 Azure SDK](/downloads/) 是一组 NuGet 包，可用于调用大多数 Azure Resource Manager API。如果该 SDK 未提供所需的功能，你可以轻松地将该 SDK 与在后台对 Resource Manager REST API 的常规调用相结合。

本文不会介绍用于 .NET 的 Azure SDK、Azure Resource Manager API 或 Visual Studio 的所有方面，而只介绍快速入门方法。

以下代码段全都摘自[可下载示例项目](https://github.com/dx-ted-emea/Azure-Resource-Manager-Documentation/tree/master/ARM/SDKs/Samples/Net)。

## 安装 NuGet 包

本文中的示例需要两个 NuGet 包（除用于 .NET 的 Azure SDK 之外）。若要安装这些包，请执行以下操作：

1. 在 Visual Studio 中，右键单击项目，然后选择“管理 NuGet 包”。
2. 搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**，然后安装该包的最新稳定版本。
3. 搜索 **Microsoft.Azure.Management.ResourceManager**，然后选择“包括预发行版”。安装最新预览版（例如 1.1.2-preview）。

## 身份验证
Azure Active Directory (Azure AD) 负责处理 Resource Manager 的身份验证。若要连接到任何 API，首先需要使用 Azure AD 进行身份验证，以接收可传递给每个请求的访问令牌。若要获取此令牌，需创建 Azure AD 应用程序和一个用于登录的服务主体。有关分步说明，请参阅下面的某篇文章：

- [使用 Azure PowerShell 创建可访问资源的 Active Directory 应用程序](/documentation/articles/resource-group-authenticate-service-principal/)
- [使用 Azure CLI 创建可访问资源的 Active Directory 应用程序](/documentation/articles/resource-group-authenticate-service-principal-cli/)
- [使用 Azure 门户创建可访问资源的 Active Directory 应用程序](/documentation/articles/resource-group-create-service-principal-portal/)

创建服务主体后，你会获得：

- 客户端 ID 或应用程序 ID (GUID)
- 客户端机密或密码（字符串）
- 租户 ID (GUID) 或域名（字符串）

### 从代码接收访问令牌
使用下面几行代码，只需传入你的 Azure AD 租户 ID、Azure AD 应用程序客户端 ID 和 Azure AD 应用程序客户端机密即可轻松获得访问令牌。保存多个请求的令牌，因为默认情况下它的有效时间为一小时。


    private static async Task<AuthenticationResult> GetAccessTokenAsync(string tenantId, string clientId, string clientSecret)
    {
        Console.WriteLine("Acquiring Access Token from Azure AD");
        AuthenticationContext authContext = new AuthenticationContext
            ("https://login.windows.net/" /* Azure AD URI */
                + $"{tenantId}" /* Tenant ID */);
    
        var credential = new ClientCredential(clientId, clientSecret);
    
        AuthenticationResult token = authContext.AcquireToken("https://management.chinacloudapi.cn/", credential);
    
        Console.WriteLine($"Token: {token.AccessToken}");
        return token;
    }


你可以使用 Azure AD 域而不是租户 ID 来登录，如以下代码所示。使用此方法时，需要更改方法签名，使之包括域名而非租户 ID。


    AuthenticationContext authContext = new AuthenticationContext
        ("https://login.chinacloudapi.cn/" /* Azure AD URI */
        + $"{domain}.onmicrosoft.com");


你可以使用以下代码获取 Azure AD 应用程序的访问令牌，以便使用证书进行身份验证：


    private static async Task<AuthenticationResult> GetAccessTokenFromCertAsync(string tenantId, string clientId, string certName)
    {
        Console.WriteLine("Acquiring Access Token from Azure AD");
        AuthenticationContext authContext = new AuthenticationContext
            ("https://login.chinacloudapi.cn/" /* Azure AD URI */
            + $"{tenantId}" /* Tenant ID or Azure AD domain */);
    
        X509Certificate2 cert = null;
        X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
    
        try
        {
            store.Open(OpenFlags.ReadOnly);
            var certCollection = store.Certificates;
            var certs = certCollection.Find(X509FindType.FindBySubjectName, certName, false);
            cert = certs[0];
        }
        finally
        {
            store.Close();
        }
    
        var certCredential = new ClientAssertionCertificate(clientId, cert);
    
        var token = await authContext.AcquireTokenAsync("https://management.chinacloudapi.cn/", certCredential);
    
        Console.WriteLine($"Token: {token.AccessToken}");
        return token;
    }


### 查询附加到经过身份验证的应用程序的 Azure 订阅
你可能需要查询与刚经过验证的应用程序相关联的 Azure 订阅。必须将目标订阅的订阅 ID 传递给从现在起执行的每个 API 调用。

下面的示例代码通过 REST API 来直接查询 Azure API。也就是说，此代码不使用用于 .NET 的 Azure SDK 中的任何功能。


    async private static Task<List<string>> GetSubscriptionsAsync(string token)
    {
        Console.WriteLine("Querying for subscriptions");
        const string apiVersion = "2015-01-01";
    
        var client = new HttpClient();
        client.BaseAddress = new Uri("https://management.chinacloudapi.cn/");
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


请注意，你会获得 Azure 发出的 JSON 响应。然后，你需要从此响应中提取订阅 ID，然后就会返回 ID 列表。在本文中，对 Azure Resource Manager API 的所有后续调用都使用单个 Azure 订阅 ID。因此，如果应用程序与多个订阅相关联，则可直接选取其中一个合适的，然后将其作为参数传递下去即可。

在这里，针对 Azure API 进行的每个调用都会使用用于 .NET 的 Azure SDK。因此，代码看起来会有点不同。

### 将令牌包装为 TokenCredentials 对象
以下所有 API 调用都需要从 Azure AD 收到的 TokenCredentials 对象格式的令牌。只需将原始令牌作为参数传递给此类的构造函数即可轻松创建此类对象。


    var credentials = new TokenCredentials(token);


如果你使用的是早期版本的 Resource Manager NuGet 包（其名称为 Microsoft.Azure.Management.Resources），则需使用以下代码：


    var credentials = new TokenCloudCredentials(subscriptionId, token.AccessToken);


## 创建资源组
Azure 中的所有操作都围绕着资源组进行，因此，让我们创建一个资源组。*ResourceManagementClient* 处理常规资源和资源组。不管使用以下哪个更专用的管理客户端，你都需要提供凭据以及订阅 ID 来确定所要使用的订阅。


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


## 手动创建资源，或使用模板创建
可通过多种方式与 Azure Resource Manager API 交互，但主要通过以下两种方式：

* 手动（通过手动调用特定资源提供程序）
* 使用 Azure Resource Manager 模板

使用 Resource Manager 模板具有以下优点：

* 你可以以声明方式指定你希望结果显示为什么样，而不是指定应如何实现。
* 你不必手动处理部署的并行执行，Resource Manager 会为你执行该操作。
* 你不必为了部署 Resource Manager 模板而学习 C# 或任何其他语言，虽然你可以使用任何语言来启动模板化部署。
* 在模板中使用的域特定语言 (DSL) 是通过 JSON 构建的。使用过 JSON 的人都会觉得其简单易懂。

虽然使用模板有上述好处，但我们仍会首先介绍如何手动调用 API。

### 逐步创建虚拟机
你已经有了订阅和资源组。若要部署虚拟机 (VM)，需了解其组成：

* 一个或多个存储帐户，用于存储持久性磁盘
* 一个或多个公共 IP 地址（包括 DNS 名称），以便通过 Internet 访问 Azure 中的资源
* 一个或多个虚拟网络，用于资源之间的内部通信
* 一个或多个网络接口卡（即 NIC），方便 VM 通信
* 一个或多个虚拟机，用于运行软件

这些资源中，某些可以并行创建，某些则不可以。例如：

* NIC 依赖于公共 IP 地址和虚拟网络。
* VM 依赖于 NIC 和存储帐户。

在创建必需的依赖项之前，请勿尝试实例化任何资源。完整[示例](https://github.com/dx-ted-emea/Azure-Resource-Manager-Documentation/tree/master/ARM/SDKs/Samples/Net)演示了如何有效地并行创建资源，同时仍随时跟踪已创建的内容。

#### 创建存储帐户
需要使用存储帐户来存储虚拟机的虚拟硬盘。如果你已经有了一个存储帐户，则可将其用于多个 VM。但请牢记将负载分布到多个存储帐户中，以免超出限制。另请记住，存储帐户的类型及其位置可能会限制可以选择的 VM 大小，因为并非所有 VM 大小都适合所有区域或所有存储帐户类型。


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


#### 创建公共 IP 地址
公共 IP 地址是使 Azure 中的资源可从 Internet 访问的 IP 地址。除了 IP 地址，系统还会为你分配完全限定域名（即 FQDN），方便你进行访问。


    private static Task<PublicIPAddress> CreatePublicIPAddressAsync(TokenCredentials credentials, string subscriptionId, string resourceGroup, string location, string pipAddressName, string pipDnsName)
    {
        Console.WriteLine("Creating Public IP");
        var networkClient = new NetworkManagementClient(credentials) { SubscriptionId = subscriptionId };
        var createPipTask = networkClient.PublicIPAddresses.CreateOrUpdateAsync(resourceGroup, pipAddressName,
            new PublicIPAddress
            {
                Location = location,
                DnsSettings = new PublicIPAddressDnsSettings { DomainNameLabel = pipDnsName },
                PublicIPAllocationMethod = "Dynamic" // This sample doesn't support static IP addresses but can be extended to do so
            });
    
        return createPipTask;
    }


#### 创建虚拟网络
使用 Resource Manager API 创建的每个 VM 都需要属于一个虚拟网络，即允许该虚拟网络只包含一个 VM。虚拟网络必须至少包含一个子网，但你可以通过多个子网来隔离并保护你的资源。


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


#### 创建网络接口卡
NIC 是将 VM 与所在虚拟网络连接到一起的设备。一个 VM 可以有多个 NIC，因此可以与多个虚拟网络相关联。此示例假定你只将 VM 连接到一个虚拟网络。


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


#### 创建虚拟机
最后，可以创建虚拟机了。VM（直接或间接地）依赖于前面创建的所有资源，因此你需要等待所有资源准备就绪，然后才能尝试预配 VM。预配 VM 比创建其他资源需要的时间更长，因此应用程序需要等待一段时间才能执行此操作。


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
                        StorageUri = $"http://{storageAccountName}.blob.core.chinacloudapi.cn"
                    })
            });
    
        return vm;
    }


### 使用模板化部署
有关如何部署模板的详细说明，请阅读[使用 .NET 库和模板部署 Azure 资源](/documentation/articles/virtual-machines-windows-csharp-template/)一文。

简而言之，部署模板比手动预配资源容易得多。以下代码演示如何通过指向包含模板和参数文件的 URI 来执行该操作。


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


<!---HONumber=Mooncake_0808_2016-->