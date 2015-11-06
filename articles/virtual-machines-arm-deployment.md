<properties
	pageTitle="使用计算、网络和存储 .NET 库部署 Azure 资源"
	description="了解如何使用计算、存储和网络 .NET 库中的某些可用客户端来创建和删除 Microsoft Azure 中的资源。"
	services="virtual-machines,virtual-network,storage"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines"
	ms.date="07/28/2015"
	wacn.date="09/15/2015"/>

# 使用计算、网络和存储 .NET 库部署 Azure 资源

本教程演示了如何使用计算、存储和网络 .NET 库中的某些可用客户端来创建和删除 Windows Azure 中的资源。它还演示了如何使用 Azure Active Directory 对发往 Azure 资源管理器的请求进行身份验证。

<!--AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)-->

若要完成本教程，你还需要：

- [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)
- [Azure 存储帐户](/documentation/articles/storage-create-storage-account)
- [Windows Management Framework 3.0](http://www.microsoft.com/zh-cn/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/zh-cn/download/details.aspx?id=40855)
- [Azure PowerShell](/documentation/articles/install-configure-powershell)

完成这些步骤大约需要 30 分钟。

## 步骤 1：将应用程序添加到 Azure AD 并设置权限

若要使用 Azure AD 要对发往 Azure 资源管理器的请求进行身份验证，必须在默认目录中添加一个应用程序。执行以下操作可以添加应用程序：

1. 打开 Azure PowerShell 提示符，然后运行以下命令：

        Switch-AzureMode –Name AzureResourceManager

2. 设置你要用于本教程的 Azure 帐户。运行以下命令，并在出现提示时输入订阅的凭据：

	    Add-AzureAccount -Environment AzureChinaCloud

3. 在以下命令中的 {password} 替换为要使用的密码，然后运行以下命令创建应用程序：

	    New-AzureADApplication -DisplayName "My AD Application 1" -HomePage "https://myapp1.com" -IdentifierUris "https://myapp1.com"  -Password "{password}"

4. 记录前一步骤的响应中提供的 ApplicationId 值。本教程后面的步骤会用到此值：

	![创建 AD 应用程序](./media/virtual-machines-arm-deployment/azureapplicationid.png)

	>[AZURE.NOTE]你也可以在管理门户上，从应用程序的客户端 id 字段中找到应用程序标识符。

5. 将 {application-id} 替换为刚才记下的标识符，然后创建该应用程序的服务主体：

        New-AzureADServicePrincipal -ApplicationId {application-id}

6. 设置应用程序的使用权限：

	    New-AzureRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName "https://myapp1.com"

## 步骤 2：创建 Visual Studio 项目并安装这些库

使用 NuGet 包是安装完成本教程所需的库的最简单方法。你必须安装 Azure 资源管理库、Azure Active Directory 身份验证库和计算机资源提供程序库。若要在 Visual Studio 中获取这些库，请执行以下操作：

1. 单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

4. 在搜索框中键入 *Active Directory*，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

5. 在页面顶部，选择“包括预发行版”。在搜索框中键入 *Azure 计算*，单击“计算 .NET 库”的**“安装”**，然后按照说明安装该包。

6. 在搜索框中键入*“Azure 网络”*，单击“网络 .NET 库”的**“安装”**，然后按照说明安装该包。

7. 在搜索框中键入*“Azure 存储空间”*，单击“网络 .NET 库”的**“安装”**，然后按照说明安装该包。

8. 在搜索框中键入*“Azure 资源管理”*，并单击“资源管理库”的**“安装”**。

现在，你可以开始使用这些库来创建应用程序了。

## 步骤 3：创建用于对请求进行身份验证的凭据

创建 Azure Active Directory 应用程序并安装身份验证库后，你可以将应用程序信息格式化为凭据，以用于对发往 Azure 资源管理器的请求进行身份验证。请执行以下操作：

1.	打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
		using Microsoft.Azure.Management.Resources;
		using Microsoft.Azure.Management.Resources.Models;
		using Microsoft.Azure.Management.Storage;
		using Microsoft.Azure.Management.Storage.Models;
		using Microsoft.Azure.Management.Network;
		using Microsoft.Azure.Management.Network.Models;
		using Microsoft.Azure.Management.Compute;
		using Microsoft.Azure.Management.Compute.Models;


2. 将以下方法添加到 Program 类，以获取创建凭据所需的令牌：

		private static string GetAuthorizationHeader()
        {
          ClientCredential cc = new ClientCredential("{application-id}", "{password}");
            var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
            var result = context.AcquireToken("https://management.azure.com/", cc);

          if (result == null)
          {
            throw new InvalidOperationException("Failed to obtain the JWT token");
          }

          string token = result.AccessToken;

          return token;
        }

	将 {application-id} 替换为前面记下的应用程序标识符，将 {password} 替换为你为 AD 应用程序选择的密码，将 {tenant-id} 替换为订阅的租户标识符。可以通过运行 Get-AzureSubscription 找到租户 ID。

3.	将以下代码添加到 Program.cs 文件中的 Main 方法，以创建凭据：

		var token = GetAuthorizationHeader();
		var credential = new TokenCloudCredentials("{subscription-id}", token);

	将 {subscription-id} 替换为订阅标识符。

4.	保存 Program.cs 文件。

## 步骤 4：添加代码以注册提供程序并创建资源

### 注册提供程序并创建资源组

资源始终部署到资源组。可以使用 [ResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.models.resourcegroup.aspx) 和 [ResourceManagementClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx) 类来创建要将资源部署到的资源组。

1.	将以下方法添加到 Program 类，以创建资源组：

		public async static void CreateResourceGroup(TokenCloudCredentials credential)
		{
		  Console.WriteLine("Creating the resource group...");

          using (var resourceManagementClient = new ResourceManagementClient(credential))
		  {
		    var rgResult = await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync("mytestrg1", new ResourceGroup { Location = "West US" });
            Console.WriteLine(rgResult.StatusCode);
            var rpResult = await resourceManagementClient.Providers.RegisterAsync("Microsoft.Storage");
            Console.WriteLine(rpResult.StatusCode);
            rpResult = await resourceManagementClient.Providers.RegisterAsync("Microsoft.Network");
            Console.WriteLine(rpResult.StatusCode);
            rpResult = await resourceManagementClient.Providers.RegisterAsync("Microsoft.Compute");
            Console.WriteLine(rpResult.StatusCode);
		  }
		}

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateResourceGroup(credential);
		Console.ReadLine();

###创建存储帐户

需要一个存储帐户来存储为虚拟机创建的虚拟硬盘文件。

1.	将以下方法添加到 Program 类，以创建存储帐户：

		public async static void CreateStorageAccount(TokenCloudCredentials credential)
        {
          Console.WriteLine("Creating the storage account...");

          using (var storageManagementClient = new StorageManagementClient(credential))
          {
            var saResult = await storageManagementClient.StorageAccounts.CreateAsync(
              "mytestrg1",
              "mytestsa1",
              new StorageAccountCreateParameters()
              { AccountType = AccountType.StandardLRS, Location = "West US" } );
            Console.WriteLine(saResult.StatusCode);
          }
        }

3.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateStorageAccount(credential);
		Console.ReadLine();

###创建虚拟网络

虚拟机在添加到虚拟网络时最为高效。

1.	将以下方法添加到 Program 类，以创建子网、公共 IP 地址和虚拟网络：

		public async static void CreateNetwork(TokenCloudCredentials credential)
        {
          Console.WriteLine("Creating the virtual network...");
          using (var networkClient = new NetworkResourceProviderClient(credential))
          {
            var subnet = new Subnet
            {
              Name = "mytestsn1",
              AddressPrefix = "10.0.0.0/24"
            };

            var vnResult = await networkClient.VirtualNetworks.CreateOrUpdateAsync(
              "mytestrg1",
              "mytestvn1",
              new VirtualNetwork
              {
                Location = "West US",
                AddressSpace = new AddressSpace { AddressPrefixes = new List<string> { "10.0.0.0/16" } },
                Subnets = new List<Subnet> { subnet }
              });
            Console.WriteLine(vnResult.StatusCode);

            var subnetResponse = await networkClient.Subnets.GetAsync(
              "mytestrg1",
              "mytestvn1",
              "mytestsn1"
            );

            Console.WriteLine("Creating the public ip...");
            vnResult = await networkClient.PublicIpAddresses.CreateOrUpdateAsync(
              "mytestrg1",
              "mytestip1",
              new PublicIpAddress
              {
                Location = "West US",
                PublicIpAllocationMethod = "Dynamic"
              });
            Console.WriteLine(vnResult.StatusCode);

            var pubipResponse = await networkClient.PublicIpAddresses.GetAsync("mytestrg1", "mytestip1");

            Console.WriteLine("Updating the network with the nic...");
            vnResult = await networkClient.NetworkInterfaces.CreateOrUpdateAsync(
              "mytestrg1",
              "mytestnic1",
              new NetworkInterface
              {
                Name = "mytestnic1",
                Location = "West US",
                IpConfigurations = new List<NetworkInterfaceIpConfiguration>
                {
                  new NetworkInterfaceIpConfiguration
                  {
                    Name = "nicconfig1",
                    PublicIpAddress = new ResourceId
                    {
                      Id = pubipResponse.PublicIpAddress.Id
                    },
                    Subnet = new ResourceId
                    {
                      Id = subnetResponse.Subnet.Id
                    }
                  }
                }
              });
            Console.WriteLine(vnResult.StatusCode);
          }
        }

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateNetwork(credential);
		Console.ReadLine();

###创建虚拟机

创建所有支持的资源后，可以创建虚拟机。

1.	将以下方法添加到 Program 类，以创建虚拟机：

		public async static void CreateVirtualMachine(TokenCloudCredentials credential)
        {
          using (var computeClient = new ComputeManagementClient(credential))
          {
            Console.WriteLine("Creating the availability set...");
            var avSetResponse = await computeClient.AvailabilitySets.CreateOrUpdateAsync(
              "mytestrg1",
              new AvailabilitySet
              {
                Name = "mytestav1",
                Location = "West US"
              } );
            Console.WriteLine(avSetResponse.StatusCode);

            var networkClient = new NetworkResourceProviderClient(credential);
            var nicResponse = await networkClient.NetworkInterfaces.GetAsync("mytestrg1", "mytestnic1");

            Console.WriteLine("Creating the virtual machine...");
            var putVMResponse = await computeClient.VirtualMachines.CreateOrUpdateAsync(
              "mytestrg1",
              new VirtualMachine
              {
                Name = "mytestvm1",
                Location = "West US",
                AvailabilitySetReference = new AvailabilitySetReference
                {
                  ReferenceUri = avSetResponse.AvailabilitySet.Id
                },
                HardwareProfile = new HardwareProfile
                {
                  VirtualMachineSize = "Standard_A0"
                },
                OSProfile = new OSProfile
                {
                  AdminUsername = "mytestuser1",
                  AdminPassword = "Mytestpass1",
                  ComputerName = "mytestsv1",
                  WindowsConfiguration = new WindowsConfiguration
                  {
                    ProvisionVMAgent = true
                  }
                },
                NetworkProfile = new NetworkProfile
                {
                  NetworkInterfaces = new List<NetworkInterfaceReference>
                  {
                    new NetworkInterfaceReference
                    {
                      ReferenceUri = nicResponse.NetworkInterface.Id
                    }
                  }
                },
                StorageProfile = new StorageProfile
                {
                  SourceImage = new SourceImageReference
                  {
                    ReferenceUri = "/{subscription-id}/services/images/a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-201502.01-en.us-127GB.vhd"
                  },
                  OSDisk = new OSDisk
                  {
                    Name = "myosdisk1",
                    CreateOption = "FromImage",
                    VirtualHardDisk = new VirtualHardDisk
                    {
                      Uri = "http://mytestsa1.blob.core.chinacloudapi.cn/vhds/myosdisk1.vhd"
                    }
                  }
                }
              } );
            Console.WriteLine(putVMResponse.StatusCode);
          }
        }

	>[AZURE.NOTE]映像库中的映像 vhd 名称经常发生变化，因此你需要获取当前映像名称以部署虚拟机。为此，请参阅[使用 Windows PowerShell 管理映像 (Windows)](https://msdn.microsoft.com/zh-cn/library/azure/dn790330.aspx)，然后将 {source-image-name} 替换为要使用的 vhd 文件名。例如，“a699494373c04fc0bc8f2bb1389d6106\_\_Windows-Server-2012-R2-201411.01-en.us-127GB.vhd”。

	将 {subscription-id} 替换为订阅的标识符。

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateVirtualMachine(credential);
        Console.ReadLine();

##步骤 5：添加代码以删除资源

由于你需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。如果要删除虚拟机和所有支持资源，只需删除资源组。

1.	将以下方法添加到 Program 类，以删除资源组：

		public async static void DeleteResourceGroup(TokenCloudCredentials credential)
        {
          Console.WriteLine("Deleting resource group...");
          using (var resourceGroupClient = new ResourceManagementClient(credential))
          {
            var rgResult = await resourceGroupClient.ResourceGroups.DeleteAsync("mytestrg1");
            Console.WriteLine(rgResult.StatusCode);
          }
        }

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		DeleteResourceGroup(credential);
        Console.ReadLine();

##步骤 6：运行控制台应用程序

1.	若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

2.	在返回每个状态代码后，按 **Enter** 创建每个资源。创建虚拟机后，执行下一步骤，然后按 Enter 删除所有资源。

	控制台应用程序从头到尾完成运行大约需要 5 分钟时间。在按 Enter 开始删除资源之前，你可能需要在 Azure 预览版门户中花费几分钟时间来验证资源的创建。

3. 在 Azure 预览版门户浏览到“审核日志”，以查看资源的状态：

	![创建 AD 应用程序](./media/virtual-machines-arm-deployment/crpportal.png)

<!---HONumber=69-->