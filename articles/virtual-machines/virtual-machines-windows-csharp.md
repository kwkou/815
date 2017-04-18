<properties
    pageTitle="使用 C# 部署 Azure 虚拟机 | Azure"
    description="使用 C# 和 Azure Resource Manager 部署虚拟机及其所有支持资源。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="87524373-5f52-4f4b-94af-50bf7b65c277"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/01/2017"
    wacn.date="04/17/2017"
    ms.author="davidmu"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="700a318a8ac7362ab65695d491dc6c7a7a64715b"
    ms.lasthandoff="04/06/2017" />

# <a name="deploy-an-azure-virtual-machine-using-c"></a>使用 C 部署 Azure 虚拟机# #

这篇文章介绍如何使用 C# 创建 Azure 虚拟机及其支持资源。

完成这些步骤大约需要 20 分钟。

## <a name="step-1-create-a-visual-studio-project"></a>步骤 1：创建 Visual Studio 项目

在此步骤中，请确保已安装 Visual Studio 并创建用于创建资源的控制台应用程序。

1. 如果尚未安装，请安装 [Visual Studio](https://www.visualstudio.com/)。
2. 在 Visual Studio 中，单击“文件” > “新建” > “项目”。
3. 在“模板” > “Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

## <a name="step-2-install-libraries"></a>步骤 2：安装库

使用 NuGet 包可以最轻松地安装完成这些步骤所需的库。 若要在 Visual Studio 中获取所需的库，请执行以下步骤：

1. 在解决方案资源管理器中右键单击项目名称，然后依次单击“管理 NuGet 包”和“浏览”。
2. 在搜索框中键入 *Microsoft.IdentityModel.Clients.ActiveDirectory*，单击“安装”，然后按照说明安装该包。
3. 在页面顶部，选择“包括预发行版”。 在搜索框中键入 *Microsoft.Azure.Management.Compute*，单击“安装”，然后按照说明安装该包。
4. 在搜索框中键入 *Microsoft.Azure.Management.Network*，单击“安装”，然后按照说明安装该包。
5. 在搜索框中键入 *Microsoft.Azure.Management.Storage*，单击“安装”，然后按照说明安装该包。
6. 在搜索框中键入 *Microsoft.Azure.Management.ResourceManager*，单击“安装”，然后按照说明安装该包。

现在，你可以开始使用这些库来创建应用程序了。

## <a name="step-3-create-the-credentials-used-to-authenticate-requests"></a>步骤 3：创建用于对请求进行身份验证的凭据

在开始此步骤之前，请确保能够访问 [Active Directory 服务主体](/documentation/articles/resource-group-authenticate-service-principal/)。 从服务主体中，将获取对 Azure Resource Manager 请求进行身份验证的令牌。

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

2. 若要获取创建凭据所需的令牌，将以下方法添加到 Program 类：

        private static async Task<AuthenticationResult> GetAccessTokenAsync()
        {
          var cc = new ClientCredential("{client-id}", "{client-secret}");
          var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
          var token = await context.AcquireTokenAsync("https://management.chinacloudapi.cn/", cc);
          if (token == null)
          {
            throw new InvalidOperationException("Could not get the token");
          }
          return token;
        }

    替换以下值：

    - 将 *{client-id}* 替换为 Azure Active Directory 应用程序的标识符。 在 AD 应用程序的“属性”边栏选项卡上，可找到此标识符。 若要在 Azure 门户预览中查找 AD 应用程序，请在资源菜单中单击“Azure Active Directory”，然后单击“应用注册”。
    - 将 *{client-secret}* 替换为 AD 应用程序的访问密钥。 在 AD 应用程序的“属性”边栏选项卡上，可找到此标识符。
    - 将 *{tenant-id}* 替换为订阅的租户标识符。 在 Azure 门户预览中 Azure Active Directory 的“属性”边栏选项卡上，可找到租户标识符。 它被标记为目录 ID。

3. 若要调用前面添加的方法，请将以下代码添加到 Program.cs 文件中的 Main 方法：

        var token = GetAccessTokenAsync();
        var credential = new TokenCredentials(token.Result.AccessToken);

4. 保存 Program.cs 文件。

## <a name="step-3-create-the-resources"></a>步骤 3：创建资源

### <a name="register-the-providers-and-create-a-resource-group"></a>注册提供程序并创建资源组

必须在资源组中包含所有资源。 将资源添加到组之前，必须将订阅注册到资源提供程序。

1. 将变量添加到 Program 类的 Main 方法，指定要用于资源的名称：

        var groupName = "myResourceGroup";
        var subscriptionId = "subsciptionId";
        var storageName = "myStorageAccount";
        var ipName = "myPublicIP";
        var subnetName = "mySubnet";
        var vnetName = "myVnet";
        var nicName = "myNIC";
        var avSetName = "myAVSet";
        var vmName = "myVM";  
        var location = "location";
        var adminName = "adminName";
        var adminPassword = "adminPassword";

    替换以下值：

    - 将以 *my* 开头的所有资源名称替换为对你的环境有意义的名称。
    - 将 *subscriptionId* 替换为订阅标识符。 在 Azure 门户预览的“订阅”边栏选项卡上，可找到订阅标识符。
    - 将 *location* 替换为想要在其中创建资源的 [Azure 区域](https://azure.microsoft.com/regions/)。
    - 将 *adminName* 替换为虚拟机上管理员帐户的名称。
    - 将 *adminPassword* 替换为管理员帐户密码。

2. 若要创建资源组并注册提供程序，请将以下方法添加到 Program 类：

        public static async Task<ResourceGroup> CreateResourceGroupAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location)
        {
          var resourceManagementClient = new ResourceManagementClient(new Uri("https://management.chinacloudapi.cn/"), credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Registering the providers...");
          var rpResult = resourceManagementClient.Providers.Register("Microsoft.Storage");
          Console.WriteLine(rpResult.RegistrationState);
          rpResult = resourceManagementClient.Providers.Register("Microsoft.Network");
          Console.WriteLine(rpResult.RegistrationState);
          rpResult = resourceManagementClient.Providers.Register("Microsoft.Compute");
          Console.WriteLine(rpResult.RegistrationState);

          Console.WriteLine("Creating the resource group...");
          var resourceGroup = new ResourceGroup { Location = location };
          return await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync(
            groupName, 
            resourceGroup);
        }

3. 若要调用之前添加的方法，请将此代码添加到 Main 方法：

        var rgResult = CreateResourceGroupAsync(
          credential,
          groupName,
          subscriptionId,
          location);
        Console.WriteLine(rgResult.Result.Properties.ProvisioningState);
        Console.ReadLine();

### <a name="create-a-storage-account"></a>创建存储帐户

使用非托管磁盘时，需要[存储帐户](/documentation/articles/storage-create-storage-account/)来存储为虚拟机创建的虚拟硬盘文件。

1. 若要创建存储帐户，请将以下方法添加到 Program 类：

        public static async Task<StorageAccount> CreateStorageAccountAsync(
          TokenCredentials credential,       
          string groupName,
          string subscriptionId,
          string location,
          string storageName)
        {
          var storageManagementClient = new StorageManagementClient(credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Creating the storage account...");

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

2. 若要调用前面添加的方法，请将以下代码添加到 Main 方法：

        var stResult = CreateStorageAccountAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          storageName);
        Console.WriteLine(stResult.Result.ProvisioningState);  
        Console.ReadLine();

### <a name="create-a-public-ip-address"></a>创建公共 IP 地址

与虚拟机通信需要公共 IP 地址。

1. 若要创建虚拟机的公共 IP 地址，请将以下方法添加到 Program 类：

        public static async Task<PublicIPAddress> CreatePublicIPAddressAsync(
          TokenCredentials credential,  
          string groupName,
          string subscriptionId,
          string location,
          string ipName)
        {
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Creating the public ip...");

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

2. 若要调用前面添加的方法，请将以下代码添加到 Main 方法：

        var ipResult = CreatePublicIPAddressAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          ipName);
        Console.WriteLine(ipResult.Result.ProvisioningState);  
        Console.ReadLine();

### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 Resource Manager 部署模型创建的虚拟机必须位于虚拟网络中。

1. 若要创建子网和虚拟网络，请将以下方法添加到 Program 类：

        public static async Task<VirtualNetwork> CreateVirtualNetworkAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location,
          string vnetName,
          string subnetName)
        {
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };

          var subnet = new Subnet
            {
              Name = subnetName,
              AddressPrefix = "10.0.0.0/24"
            };

          var address = new AddressSpace 
            {
              AddressPrefixes = new List<string> { "10.0.0.0/16" }
            };

          Console.WriteLine("Creating the virtual network...");
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

2. 若要调用前面添加的方法，请将以下代码添加到 Main 方法：

        var vnResult = CreateVirtualNetworkAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          vnetName,
          subnetName);
        Console.WriteLine(vnResult.Result.ProvisioningState);  
        Console.ReadLine();

### <a name="create-a-network-interface"></a>创建网络接口

虚拟机需要使用网络接口在虚拟网络上通信。

1. 若要创建网络接口，请将以下方法添加到 Program 类：

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
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var subnet = await networkManagementClient.Subnets.GetAsync(
            groupName,
            vnetName,
            subnetName
          );
          var publicIP = await networkManagementClient.PublicIPAddresses.GetAsync(
            groupName, 
            ipName
          );

          Console.WriteLine("Creating the network interface...");
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

2. 若要调用前面添加的方法，请将以下代码添加到 Main 方法：

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

### <a name="create-an-availability-set"></a>创建可用性集

可用性集可以方便你管理应用程序所使用的虚拟机的维护。

1. 若要创建可用性集，请将以下方法添加到 Program 类：

        public static async Task<AvailabilitySet> CreateAvailabilitySetAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location,
          string avsetName)
        {
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Creating the availability set...");
          return await computeManagementClient.AvailabilitySets.CreateOrUpdateAsync(
            groupName,
            avsetName,
            new AvailabilitySet()
              { Location = location }
          );
        }

2. 若要调用前面添加的方法，请将以下代码添加到 Main 方法：

        var avResult = CreateAvailabilitySetAsync(
          credential,  
          groupName,
          subscriptionId,
          location,
          avSetName);
        Console.ReadLine();

### <a name="create-a-virtual-machine"></a>创建虚拟机

创建所有支持资源后，即可创建虚拟机。

1. 若要创建虚拟机，请将以下方法添加到 Program 类：

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
          var networkManagementClient = new NetworkManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };  
          var nic = await networkManagementClient.NetworkInterfaces.GetAsync(
            groupName, 
            nicName);
          var avSet = await computeManagementClient.AvailabilitySets.GetAsync(
            groupName, 
            avsetName);

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
        }

    > [AZURE.NOTE]
    > 本教程创建运行 Windows Server 操作系统版本的虚拟机。 若要详细了解如何选择其他映像，请参阅 [Navigate and select Azure virtual machine images with Windows PowerShell and the Azure CLI](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（使用 Windows PowerShell 和 Azure CLI 来导航和选择 Azure 虚拟机映像）。
    > 
    >

2. 若要调用之前添加的方法，请将此代码添加到 Main 方法：

        var vmResult = CreateVirtualMachineAsync(
          credential,
          groupName,
          subscriptionId,
          location,
          nicName,
          avSetName,
          storageName,
          adminName,
          adminPassword,
          vmName);
        Console.WriteLine(vmResult.Result.ProvisioningState);
        Console.ReadLine();

## <a name="step-4-delete-the-resources"></a>步骤 4：删除资源

由于需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。 如果要删除虚拟机和所有支持资源，只需删除资源组。

1. 若要删除资源组，请将此方法添加到 Program 类：

        public static async void DeleteResourceGroupAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId)
        {
          Console.WriteLine("Deleting resource group...");
          var resourceManagementClient = new ResourceManagementClient(new Uri("https://management.chinacloudapi.cn/"), credential)
            { SubscriptionId = subscriptionId };
          await resourceManagementClient.ResourceGroups.DeleteAsync(groupName);
        }

2. 若要调用之前添加的方法，请将此代码添加到 Main 方法：

        DeleteResourceGroupAsync(
          credential,
          groupName,
          subscriptionId);
        Console.ReadLine();

## <a name="step-5-run-the-console-application"></a>步骤 5：运行控制台应用程序

1. 若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

2. 显示“已成功”状态后按 **Enter**。 

3. 在创建虚拟机之后、按 **Enter** 开始删除资源之前，可能需要在 Azure 门户预览中花几分钟时间来检查资源。

## <a name="next-steps"></a>后续步骤
* 参考 [Deploy an Azure Virtual Machine using C# and a Resource Manager template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 C# 和 Resource Manager 模板部署 Azure 虚拟机）中的信息，利用模板创建虚拟机。
* 查看[使用 Azure Resource Manager 和 C# 管理 Azure 虚拟机](/documentation/articles/virtual-machines-windows-csharp-manage/)，了解如何管理创建的虚拟机。
<!--Update_Description: wording update-->