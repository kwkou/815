<properties
	pageTitle="使用 C# 部署 Azure 资源 | Azure"
	description="了解如何使用 C# 和 Azure Resource Manager 创建 Azure 资源。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="07/14/2016"
	wacn.date="08/15/2016"/>

# 使用 C 部署 Azure 资源# 

本文演示了如何使用 C# 创建 Azure 资源。

首先需要确保你已完成此操作：

- 安装 [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)
- 验证是否安装了 [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)
- 获取[身份验证令牌](/documentation/articles/resource-group-authenticate-service-principal/)

完成这些步骤大约需要 30 分钟。

## 步骤 1：创建 Visual Studio 项目并安装库

使用 NuGet 包是安装完成本教程所需的库的最简单方法。你必须安装 Azure 资源管理库、Azure Active Directory 身份验证库和计算机资源提供程序库。若要在 Visual Studio 中获取这些库，请执行以下操作：

1. 单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

4. 在搜索框中键入 *Active Directory*，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

5. 在页面顶部，选择“包括预发行版”。在搜索框中键入 *Microsoft.Azure.Management.Compute*，单击“计算 .NET 库”的“安装”，然后按照说明安装该包。

6. 在搜索框中键入 *Microsoft.Azure.Management.Network*，单击“网络 .NET 库”的“安装”，然后按照说明安装该包。

7. 在搜索框中键入 *Microsoft.Azure.Management.Storage*，单击“存储 .NET 库”的“安装”，然后按照说明安装该包。

8. 在搜索框中键入 *Microsoft.Azure.Management.ResourceManager*，并单击“资源管理库”的“安装”。

现在，你可以开始使用这些库来创建应用程序了。

## 步骤 2：创建用于对请求进行身份验证的凭据

创建 Azure Active Directory 应用程序并安装身份验证库后，你可以将应用程序信息格式化为凭据，以用于对发往 Azure Resource Manager 的请求进行身份验证。执行以下操作：

1. 打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.ResourceManager;
        using Microsoft.Azure.Management.ResourceManager.Models;
        using Microsoft.Azure.Management.Storage;
        using Microsoft.Azure.Management.Storage.Models;
        using Microsoft.Azure.Management.Network;
        using Microsoft.Azure.Management.Network.Models;
        using Microsoft.Azure.Management.Compute;
        using Microsoft.Azure.Management.Compute.Models;
        using Microsoft.Rest;

2. 将以下方法添加到 Program 类，以获取创建凭据所需的令牌：

        private static async Task<AuthenticationResult> GetAccessTokenAsync()
        {
          var cc = new ClientCredential("{client-id}", "{client-secret}");
          var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
          var token = context.AcquireTokenAsync("https://management.chinacloudapi.cn/", cc);
          if (token == null)
          {
            throw new InvalidOperationException("Could not get the token");
          }
          return token;
        }

	将 {client-id} 替换为 Azure Active Directory 应用程序的标识符，将 {client-secret} 替换为 AD 应用程序的访问密钥，并将 {tenant-id} 替换为你的订阅的租户标识符。可以通过运行 Get-AzureRmSubscription 找到租户 ID。可以使用 Azure 门户找到访问密钥。

3. 将以下代码添加到 Program.cs 文件中的 Main 方法，以创建凭据：

        var token = GetAccessTokenAsync();
        var credential = new TokenCredentials(token.Result.AccessToken);

4. 保存 Program.cs 文件。

## 步骤 3：添加代码以注册提供程序并创建资源

### 注册提供程序并创建资源组

必须在资源组中包含所有资源。将资源添加到组之前，必须将订阅注册到资源提供程序。

1. 将变量添加到 Program 类的 Main 方法，以便指定需要用于资源的名称、资源的位置（例如“中国北部”）、管理员帐户信息，以及订阅标识符：

        var groupName = "resource group name";
        var subscriptionId = "subsciption id";
        var location = "location name";
        var storageName = "storage account name";
        var ipName = "public ip name";
        var subnetName = "subnet name";
        var vnetName = "virtual network name";
        var nicName = "network interface name";
        var avSetName = "availability set name";
        var vmName = "virtual machine name";  
        var adminName = "administrator account name";
        var adminPassword = "administrator account password";
        
    将所有变量值替换为你想要使用的名称和标识符。可以通过运行 Get-AzureRmSubscription 查找订阅标识符。

2. 将以下方法添加到 Program 类，以创建资源组并注册提供程序：

        public static async Task<ResourceGroup> CreateResourceGroupAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location)
        {
          var rpResult = resourceManagementClient.Providers.Register("Microsoft.Storage");
          Console.WriteLine(rpResult.RegistrationState);
          rpResult = resourceManagementClient.Providers.Register("Microsoft.Network");
          Console.WriteLine(rpResult.RegistrationState);
          rpResult = resourceManagementClient.Providers.Register("Microsoft.Compute");
          Console.WriteLine(rpResult.RegistrationState);
          
          Console.WriteLine("Creating the resource group...");
          var resourceManagementClient = new ResourceManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var resourceGroup = new ResourceGroup { Location = location };
          return await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync(groupName, resourceGroup);
        }

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        var rgResult = CreateResourceGroupAsync(
          credential,
          groupName,
          subscriptionId,
          location);
        Console.WriteLine(rgResult.Result.Properties.ProvisioningState);
        Console.ReadLine();

### 创建存储帐户

需要一个[存储帐户](/documentation/articles/storage-create-storage-account/)来存储为虚拟机创建的虚拟硬盘文件。

1. 将以下方法添加到 Program 类，以创建存储帐户：

        public static async Task<StorageAccount> CreateStorageAccountAsync(
          TokenCredentials credential,       
          string groupName,
          string subscriptionId,
          string location,
          string storageName)
        {
          Console.WriteLine("Creating the storage account...");
          var storageManagementClient = new StorageManagementClient(credential);
            { SubscriptionId = subscriptionId };
          return await storageManagementClient.StorageAccounts.CreateAsync(
            groupName,
            storageName,
            new StorageAccountCreateParameters()
            {
              Sku = new Microsoft.Azure.Management.Storage.Models.Sku() 
                { Name = SkuName.StandardLRS},
              Kind = Kind.Storage,
              Location = location
            }
          );
        }

2. 将以下代码添加到 Program 类的 Main 方法，以调用你刚刚添加的方法：

        var stResult = CreateStorageAccountAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          storageName);
        Console.WriteLine(stResult.Result.ProvisioningState);  
        Console.ReadLine();

### 创建公共 IP 地址

与虚拟机通信需要公共 IP 地址。

1. 将以下方法添加到 Program 类，以创建虚拟机的公共 IP 地址：

        public static async Task<PublicIPAddress> CreatePublicIPAddressAsync(
          TokenCredentials credential,  
          string groupName,
          string subscriptionId,
          string location,
          string ipName)
        {
          Console.WriteLine("Creating the public ip...");
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };
          return await networkManagementClient.PublicIPAddresses.CreateOrUpdateAsync(
            groupName,
            ipName,
            new PublicIPAddress
            {
              Location = location,
              PublicIPAllocationMethod = "Dynamic"
            }
          );
        }

2. 将以下代码添加到 Program 类的 Main 方法，以调用你刚刚添加的方法：

        var ipResult = CreatePublicIPAddressAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          ipName);
        Console.WriteLine(ipResult.Result.ProvisioningState);  
        Console.ReadLine();

### 创建虚拟网络

使用 Resource Manager 部署模型创建的虚拟机必须位于虚拟网络中。

1. 将以下方法添加到 Program 类，以创建子网和虚拟网络：

        public static async Task<VirtualNetwork> CreateVirtualNetworkAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location,
          string vnetName,
          string subnetName)
        {
          Console.WriteLine("Creating the virtual network...");
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };
          
          var subnet = new Subnet
          {
            Name = subnetName,
            AddressPrefix = "10.0.0.0/24"
          };
          
          var address = new AddressSpace {
            AddressPrefixes = new List<string> { "10.0.0.0/16" }
          };
          
          return await networkManagementClient.VirtualNetworks.CreateOrUpdateAsync(
            groupName,
            vnetName,
            new VirtualNetwork
            {
              Location = location,
              AddressSpace = address,
              Subnets = new List<Subnet> { subnet }
            }
          );
        }
        
2. 将以下代码添加到 Program 类的 Main 方法，以调用你刚刚添加的方法：

        var vnResult = CreateVirtualNetworkAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          vnetName,
          subnetName);
        Console.WriteLine(vnResult.Result.ProvisioningState);  
        Console.ReadLine();
        
### 创建网络接口

虚拟机需要网络接口才能在刚创建的虚拟网络上进行通信。

1. 将以下方法添加到 Program 类，以创建网络接口：

        public static async Task<NetworkInterface> CreateNetworkInterfaceAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location,
          string subnetName,
          string vnetName,
          string ipName,
          string nicName)
        {
          Console.WriteLine("Creating the network interface...");
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var subnetResponse = await networkManagementClient.Subnets.GetAsync(
            groupName,
            vnetName,
            subnetName
          );
          var pubipResponse = await networkManagementClient.PublicIPAddresses.GetAsync(groupName, ipName);

          return await networkManagementClient.NetworkInterfaces.CreateOrUpdateAsync(
            groupName,
            nicName,
            new NetworkInterface
            {
              Location = location,
              IpConfigurations = new List<NetworkInterfaceIPConfiguration>
              {
                new NetworkInterfaceIPConfiguration
                {
                  Name = nicName,
                  PublicIPAddress = pubipResponse,
                  Subnet = subnetResponse
                }
              }
            }
          );
        }

2. 将以下代码添加到 Program 类的 Main 方法，以调用你刚刚添加的方法：

        var ncResult = CreateNetworkInterfaceAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          subnetName,
          vnetName,
          ipName,
          nicName);
        Console.WriteLine(ncResult.Result.ProvisioningState);  
        Console.ReadLine();

### 创建可用性集

可用性集可以方便你管理应用程序所使用的虚拟机的维护。

1. 将以下方法添加到 Program 类，以创建可用性集：

        public static async Task<AvailabilitySet> CreateAvailabilitySetAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location,
          string avsetName)
        {
          Console.WriteLine("Creating the availability set...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          return await computeManagementClient.AvailabilitySets.CreateOrUpdateAsync(
            groupName,
            avsetName,
            new AvailabilitySet()
            {
              Location = location
            }
          );
        }

2. 将以下代码添加到 Program 类的 Main 方法，以调用你刚刚添加的方法：

        var avResult = CreateAvailabilitySetAsync(
          credential,  
          groupName,
          subscriptionId,
          location,
          avSetName);
        Console.ReadLine();

### 创建虚拟机

创建所有支持的资源后，可以创建虚拟机。

1. 将以下方法添加到 Program 类，以创建虚拟机：

        public static async Task<VirtualMachine> CreateVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName,
          string subscriptionId,
          string location,
          string nicName,
          string avsetName,
          string storageName,
          string adminName,
          string adminPassword,
          string vmName)
        {
          var networkManagementClient = new NetworkManagementClient(credential);
            { SubscriptionId = subscriptionId };
          var nic = networkManagementClient.NetworkInterfaces.Get(groupName, nicName);

          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          var avSet = computeManagementClient.AvailabilitySets.Get(groupName, avsetName);

          Console.WriteLine("Creating the virtual machine...");
          return await computeManagementClient.VirtualMachines.CreateOrUpdateAsync(
            groupName,
            vmName,
            new VirtualMachine
            {
              Location = location,
              AvailabilitySet = new Microsoft.Azure.Management.Compute.Models.SubResource
              {
                Id = avSet.Id
              },
              HardwareProfile = new HardwareProfile
              {
                VmSize = "Standard_A0"
              },
              OsProfile = new OSProfile
              {
                AdminUsername = adminName,
                AdminPassword = adminPassword,
                ComputerName = vmName,
                WindowsConfiguration = new WindowsConfiguration
                {
                  ProvisionVMAgent = true
                }
              },
              NetworkProfile = new NetworkProfile
              {
                NetworkInterfaces = new List<NetworkInterfaceReference>
                {
                  new NetworkInterfaceReference { Id = nic.Id }
                }
              },
              StorageProfile = new StorageProfile
              {
                ImageReference = new ImageReference
                {
                  Publisher = "MicrosoftWindowsServer",
                  Offer = "WindowsServer",
                  Sku = "2012-R2-Datacenter",
                  Version = "latest"
                },
                OsDisk = new OSDisk
                {
                  Name = "mytestod1",
                  CreateOption = DiskCreateOptionTypes.FromImage,
                  Vhd = new VirtualHardDisk
                  {
                    Uri = "http://" + storageName + ".blob.core.chinacloudapi.cn/vhds/mytestod1.vhd"
                  }
                }
              }
            }
          );
          Console.WriteLine(vm.ProvisioningState);
        }

	>[AZURE.NOTE] 本教程创建运行 Windows Server 操作系统版本的虚拟机。若要详细了解如何选择其他映像，请参阅 [Navigate and select Azure virtual machine images with Windows PowerShell and the Azure CLI](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（使用 Windows PowerShell 和 Azure CLI 来导航和选择 Azure 虚拟机映像）。

2. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        var vmResult = CreateVirtualMachineAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          nicName,
          avsetName,
          storageName,
          adminName,
          adminPassword,
          vmName);
        Console.WriteLine(vmResult.Result.ProvisioningState);
        Console.ReadLine();

##步骤 4：添加代码以删除资源

由于你需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。如果要删除虚拟机和所有支持资源，只需删除资源组。

1.	将以下方法添加到 Program 类，以删除资源组：

        public static async void DeleteResourceGroupAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId)
        {
          Console.WriteLine("Deleting resource group...");
          var resourceManagementClient = new ResourceManagementClient(credential)
            { SubscriptionId = subscriptionId };
          return await resourceManagementClient.ResourceGroups.DeleteAsync(groupName);
        }

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        DeleteResourceGroupAsync(
          credential,
          groupName,
          subscriptionId);
        Console.ReadLine();

## 步骤 5：运行控制台应用程序

1. 若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

2. 在返回每个状态代码后，按 **Enter** 创建每个资源。创建虚拟机后，执行下一步骤，然后按 Enter 删除所有资源。

	控制台应用程序从头到尾完成运行大约需要 5 分钟时间。在按 Enter 开始删除资源之前，你可能需要在 Azure 门户预览版中花费几分钟时间来验证资源的创建。

3. 在 Azure 门户预览版中浏览到“审核日志”，以查看资源的状态：

	![在 Azure 门户预览版中浏览审核日志](./media/virtual-machines-windows-csharp/crpportal.png)
    
## 后续步骤

- 参考 [Deploy an Azure Virtual Machine using C# and a Resource Manager template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 C# 和 Resource Manager 模板部署 Azure 虚拟机）中的信息，利用模板创建虚拟机。
- 查看 [Manage virtual machines using Azure Resource Manager and PowerShell](/documentation/articles/virtual-machines-windows-csharp-manage/)（使用 Azure Resource Manager 和 PowerShell 管理虚拟机），了解如何管理刚创建的虚拟机。

<!---HONumber=Mooncake_0808_2016-->