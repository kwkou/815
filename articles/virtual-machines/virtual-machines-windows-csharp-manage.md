<!-- ARM: tested -->

<properties
	pageTitle="使用 Azure Resource Manager 和 C# 管理 VM | Azure"
	description="使用 Azure Resource Manager 和 C# 来管理虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/17/2016"
	wacn.date="06/07/2016"/>

# 使用 Azure Resource Manager 与 C 来管理 Azure 虚拟机#  

若要完成本文中的任务，你需要：

- [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)。
- [身份验证令牌](/documentation/articles/resource-group-authenticate-service-principal/)

## 创建 Visual Studio 项目并安装包

使用 NuGet 包可以最轻松地安装本文中任务所需的库。必须安装 Azure Active Directory 身份验证库和计算机资源提供程序库。若要在 Visual Studio 中获取这些库，请执行以下操作：

1. 单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

4. 在搜索框中键入 Active Directory，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

5. 在页面顶部，选择“包括预发行版”。在搜索框中键入 Microsoft.Azure.Management.Compute，单击“计算 .NET 库”的“安装”，然后按照说明安装该包。

现在，你已准备好开始使用这些库来管理虚拟机。

## 设置项目

创建 Azure Active Directory 应用程序并安装身份验证库后，你可以将应用程序信息格式化为凭据，以用于对发往 Azure 资源管理器的请求进行身份验证。

1. 打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.Compute;
        using Microsoft.Azure.Management.Compute.Models;
        using Microsoft.Rest;
        
2. 将变量添加到 Program 类的 Main 方法，以便指定要管理的资源的名称、资源的位置（例如“中国北部”）和订阅标识符：

        var groupName = "resource group name";
        var vmName = "virtual machine name";  
        var location = "location name";
        var subscriptionId = "subsciption id";

    将所有变量值替换为你想要使用的名称和标识符。可以通过运行 Get-AzureRmSubscription 查找订阅标识符。
    
3. 将以下方法添加到 Program 类，以获取创建凭据所需的令牌。

	    private static string GetAuthorizationHeader()
	    {
          ClientCredential cc = new ClientCredential("{application-id}", "{password}");
          var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
          var result = context.AcquireTokenAsync("https://management.chinacloudapi.cn/", cc);

          if (result == null)
          {
            throw new InvalidOperationException("Failed to obtain the JWT token");
          }

          string token = result.Result.AccessToken;

          return token;
        }
	
    将 {application-id} 替换为前面记下的应用程序标识符，将 {password} 替换为你为 AD 应用程序选择的密码，将 {tenant-id} 替换为订阅的租户标识符。
    
4. 将以下代码添加到 Program.cs 中的 Main 方法，以创建凭据：

        var token = GetAuthorizationHeader();
        var credential = new TokenCredentials(token);

5. 保存 Program.cs 文件。

## 显示有关虚拟机的信息

1. 将以下方法添加到前面创建的项目中的 Program 类：

        public static void GetVirtualMachine(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Getting information about the virtual machine...");

          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          var vmResult = computeManagementClient.VirtualMachines.Get(
            groupName, 
            vmName, 
            InstanceViewTypes.InstanceView);

          Console.WriteLine("hardwareProfile");
          Console.WriteLine("   vmSize: " + vmResult.HardwareProfile.VmSize);

          Console.WriteLine("\nstorageProfile");
          Console.WriteLine("  imageReference");
          Console.WriteLine("    publisher: " + vmResult.StorageProfile.ImageReference.Publisher);
          Console.WriteLine("    offer: " + vmResult.StorageProfile.ImageReference.Offer);
          Console.WriteLine("    sku: " + vmResult.StorageProfile.ImageReference.Sku);
          Console.WriteLine("    version: " + vmResult.StorageProfile.ImageReference.Version);
          Console.WriteLine("  osDisk");
          Console.WriteLine("    osType: " + vmResult.StorageProfile.OsDisk.OsType);
          Console.WriteLine("    name: " + vmResult.StorageProfile.OsDisk.Name);
          Console.WriteLine("    createOption: " + vmResult.StorageProfile.OsDisk.CreateOption);
          Console.WriteLine("    uri: " + vmResult.StorageProfile.OsDisk.Vhd.Uri);
          Console.WriteLine("    caching: " + vmResult.StorageProfile.OsDisk.Caching);

          Console.WriteLine("\nosProfile");
          Console.WriteLine("  computerName: " + vmResult.OsProfile.ComputerName);
          Console.WriteLine("  adminUsername: " + vmResult.OsProfile.AdminUsername);
          Console.WriteLine("  provisionVMAgent: " + vmResult.OsProfile.WindowsConfiguration.ProvisionVMAgent.Value);
          Console.WriteLine("  enableAutomaticUpdates: " + vmResult.OsProfile.WindowsConfiguration.EnableAutomaticUpdates.Value);

          Console.WriteLine("\nnetworkProfile");
          foreach (NetworkInterfaceReference nic in vmResult.NetworkProfile.NetworkInterfaces)
          {
            Console.WriteLine("  networkInterface id: " + nic.Id);
          }

          Console.WriteLine("\nvmAgent");
          Console.WriteLine("  vmAgentVersion" + vmResult.InstanceView.VmAgent.VmAgentVersion);
          Console.WriteLine("    statuses");
          foreach (InstanceViewStatus stat in vmResult.InstanceView.VmAgent.Statuses)
          {
            Console.WriteLine("    code: " + stat.Code);
            Console.WriteLine("    level: " + stat.Level);
            Console.WriteLine("    displayStatus: " + stat.DisplayStatus);
            Console.WriteLine("    message: " + stat.Message);
            Console.WriteLine("    time: " + stat.Time);
          }

          Console.WriteLine("\ndisks");
          foreach (DiskInstanceView idisk in vmResult.InstanceView.Disks)
          {
            Console.WriteLine("  name: " + idisk.Name);
            Console.WriteLine("  statuses");
            foreach (InstanceViewStatus istat in idisk.Statuses)
            {
              Console.WriteLine("    code: " + istat.Code);
              Console.WriteLine("    level: " + istat.Level);
              Console.WriteLine("    displayStatus: " + istat.DisplayStatus);
              Console.WriteLine("    time: " + istat.Time);
            }
          }

          Console.WriteLine("\nVM general status");
          Console.WriteLine("  provisioningStatus: " + vmResult.ProvisioningState);
          Console.WriteLine("  id: " + vmResult.Id);
          Console.WriteLine("  name: " + vmResult.Name);
          Console.WriteLine("  type: " + vmResult.Type);
          Console.WriteLine("  location: " + vmResult.Location);
          Console.WriteLine("\nVM instance status");
          foreach (InstanceViewStatus istat in vmResult.InstanceView.Statuses)
          {
            Console.WriteLine("\n  code: " + istat.Code);
            Console.WriteLine("  level: " + istat.Level);
            Console.WriteLine("  displayStatus: " + istat.DisplayStatus);
          }
        }

2. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        GetVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
    
3. 保存 Program.cs 文件。

4. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

	当你运行此方法时，应会看到与下面类似的内容：
    
        Getting information about the virtual machine...
        hardwareProfile
          vmSize: Standard_A0

        storageProfile
          imageReference
            publisher: MicrosoftWindowsServer
            offer: WindowsServer
            sku: 2012-R2-Datacenter
            version: latest
          osDisk
            osType: Windows
            name: myosdisk
            createOption: FromImage
            uri: http://store1.blob.core.chinacloudapi.cn/vhds/myosdisk.vhd
            caching: ReadWrite

          osProfile
            computerName: vm1
            adminUsername: account1
            provisionVMAgent: True
            enableAutomaticUpdates: True

          networkProfile
            networkInterface 
              id: /subscriptions/{subscription-id}
                /resourceGroups/rg1/providers/Microsoft.Network/networkInterfaces/nc1

          vmAgent
            vmAgentVersion2.7.1198.766
            statuses
            code: ProvisioningState/succeeded
            level: Info
            displayStatus: Ready
            message: GuestAgent is running and accepting new configurations.
            time: 4/13/2016 8:35:32 PM

          disks
            name: myosdisk
            statuses
              code: ProvisioningState/succeeded
              level: Info
              displayStatus: Provisioning succeeded
              time: 4/13/2016 8:04:36 PM

          VM general status
            provisioningStatus: Succeeded
            id: /subscriptions/{subscription-id}
              /resourceGroups/rg1/providers/Microsoft.Compute/virtualMachines/vm1
            name: vm1
            type: Microsoft.Compute/virtualMachines
            location: centralus

          VM instance status
            code: ProvisioningState/succeeded
              level: Info
              displayStatus: Provisioning succeeded
            code: PowerState/running
              level: Info
              displayStatus: VM running

## 启动虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static void StartVirtualMachine(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Starting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          computeManagementClient.VirtualMachines.Start(groupName, vmName);
        }

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        StartVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

	你应会看到虚拟机的状态更改为“正在运行”。

## 停止虚拟机

可以使用两种方法停止虚拟机。可以停止虚拟机并保留其所有设置但继续支付其费用，也可以停止虚拟机并将其解除分配，这也会解除分配与其关联的所有资源，并停止该虚拟机的计费。

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static void StopVirtualMachine(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Stopping the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          computeManagementClient.VirtualMachines.PowerOff(groupName, vmName);
        }

	如果你要解除分配虚拟机，请将 PowerOff 调用更改为：

        computeManagementClient.VirtualMachines.Deallocate(groupName, vmName);

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        StopVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

    你应会看到虚拟机的状态更改为“已停止”。如果你运行了调用 Deallocate 的方法，则状态为已停止（已解除分配）。

## 重新启动正在运行的虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static void RestartVirtualMachine(
          TokenCredentials credential,
          string groupName,
          string vmName,
          string subscriptionId)
        {
          Console.WriteLine("Restarting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          computeManagementClient.VirtualMachines.Restart(groupName, vmName);
        }

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        RestartVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

## 删除虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static void DeleteVirtualMachine(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Deleting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          computeManagementClient.VirtualMachines.Delete(groupName, vmName);
        }

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        DeleteVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

## 更新虚拟机

本示例演示如何更改运行中虚拟机的大小。

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static void UpdateVirtualMachine(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Updating the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential);
          computeManagementClient.SubscriptionId = subscriptionId;
          var vmResult = computeManagementClient.VirtualMachines.Get(groupName, vmName);
          vmResult.HardwareProfile.VmSize = "Standard_A1";
          computeManagementClient.VirtualMachines.CreateOrUpdate(groupName, vmName, vmResult);
        }

3. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

        UpdateVirtualMachine(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

    你应会看到虚拟机的大小更改为 Standard\_A1

<!---HONumber=Mooncake_0425_2016-->