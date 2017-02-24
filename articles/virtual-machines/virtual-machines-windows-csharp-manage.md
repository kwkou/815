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
	ms.workload="na"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/27/2016"
	wacn.date="12/16/2016"
	ms.author="davidmu"/>  


# 使用 Azure Resource Manager 与 C 来管理 Azure 虚拟机#  

本文中的任务显示了如何管理虚拟机，如启动、停止和更新。资源组中必须有虚拟机才能完成本文中的任务。

若要完成本文中的任务，你需要：

- [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)
- [身份验证令牌](/documentation/articles/resource-group-authenticate-service-principal/)

## 创建 Visual Studio 项目并安装包

使用 NuGet 包可以最轻松地安装本文中任务所需的库。针对本文安装的库是 Azure Active Directory 身份验证库和计算资源提供程序库。请完成以下步骤来获取 Visual Studio 中的库：

1. 依次单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

4. 在搜索框中键入 *Active Directory*，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

5. 在页面顶部，选择“包括预发行版”。在搜索框中键入 *Microsoft.Azure.Management.Compute*，单击“计算 .NET 库”的“安装”，然后按照说明安装该包。

现在，你已准备好开始使用这些库来管理虚拟机。

## 设置项目

现已创建应用程序并安装了库，即可使用应用程序信息创建令牌。此令牌用于验证发至 Azure Resource Manager 的请求。

1. 打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.Compute;
        using Microsoft.Azure.Management.Compute.Models;
        using Microsoft.Rest;
        
2. 将变量添加到 Program 类的 Main 方法，指定资源组名称、虚拟机名称和订阅标识符：

        var groupName = "resource group name";
        var vmName = "virtual machine name";  
        var subscriptionId = "subsciption id";

    可以通过运行 Get-AzureRmSubscription 查找订阅标识符。
    
3. 若要获取创建凭据所需的令牌，请将以下方法添加到 Program 类：

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
	
    将 {client-id} 替换为 Azure Active Directory 应用程序的标识符，将 {client-secret} 替换为 AD 应用程序的访问密钥，并将 {tenant-id} 替换为你的订阅的租户标识符。可以通过运行 Get-AzureRmSubscription 找到租户 ID。可使用 Azure 门户预览找到访问密钥。
    
4. 若要创建凭据，请将以下代码添加到 Program.cs 中的 Main 方法：

        var token = GetAccessTokenAsync();
        var credential = new TokenCredentials(token.Result.AccessToken);

5. 保存 Program.cs 文件。

## 显示有关虚拟机的信息

1. 将以下方法添加到前面创建的项目中的 Program 类：

        public static async void GetVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Getting information about the virtual machine...");

          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(
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

2. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        GetVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
    
3. 保存 Program.cs 文件。

4. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

	运行此方法时，应会显示与下例类似的内容：
    
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
            location: chinaeast

          VM instance status
            code: ProvisioningState/succeeded
              level: Info
              displayStatus: Provisioning succeeded
            code: PowerState/running
              level: Info
              displayStatus: VM running

## 停止虚拟机

可以使用两种方法停止虚拟机。可停止虚拟机并保留其所有设置，但需继续付费；还可停止虚拟机并解除分配。解除分配虚拟机时，也会解除分配与其关联的所有资源并将停止计费。

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void StopVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Stopping the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.PowerOffAsync(groupName, vmName);
        }

	若要解除分配虚拟机，请将 PowerOff 调用更改为以下代码：

        computeManagementClient.VirtualMachines.Deallocate(groupName, vmName);

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        StopVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

    虚拟机的状态应会显示更改为“已停止”。如果你运行了调用 Deallocate 的方法，则状态为已停止（已解除分配）。

## 启动虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void StartVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Starting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.StartAsync(groupName, vmName);
        }

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        StartVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

	你应会看到虚拟机的状态更改为“正在运行”。

## 重新启动正在运行的虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void RestartVirtualMachineAsync(
          TokenCredentials credential,
          string groupName,
          string vmName,
          string subscriptionId)
        {
          Console.WriteLine("Restarting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.RestartAsync(groupName, vmName);
        }

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        RestartVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

## 重设虚拟机的大小

本示例演示如何更改运行中虚拟机的大小。

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void UpdateVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Updating the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(groupName, vmName);
          vmResult.HardwareProfile.VmSize = "Standard_A1";
          await computeManagementClient.VirtualMachines.CreateOrUpdateAsync(groupName, vmName, vmResult);
        }

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        UpdateVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

    虚拟机的大小应会显示更改为 Standard\_A1。

## 将数据磁盘添加到虚拟机

此示例演示了如何向正在运行的虚拟机添加数据磁盘。

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void AddDataDiskAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Adding the disk to the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(groupName, vmName);
          vmResult.StorageProfile.DataDisks.Add(
            new DataDisk
              {
                Lun = 0,
                Name = "mydatadisk1",
                Vhd = new VirtualHardDisk
                  {
                    Uri = "https://mystorage1.blob.core.chinacloudapi.cn/vhds/mydatadisk1.vhd"
                  },
                CreateOption = DiskCreateOptionTypes.Empty,
                DiskSizeGB = 2,
                Caching = CachingTypes.ReadWrite
              });
          await computeManagementClient.VirtualMachines.CreateOrUpdateAsync(groupName, vmName, vmResult);
        }

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        AddDataDiskAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

## 删除虚拟机

1. 注释掉前面已添加到 Main 方法的任何代码（用于获得凭据的代码除外）。

2. 将以下方法添加到 Program 类：

        public static async void DeleteVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Deleting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.DeleteAsync(groupName, vmName);
        }

3. 若要调用刚添加的方法，请将以下代码添加到 Main 方法：

        DeleteVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();

4. 保存 Program.cs 文件。

5. 在 Visual Studio 中单击“启动”，然后使用订阅所用的相同用户名和密码登录到 Azure AD。

## 后续步骤

如果部署出现问题，请参阅[使用 Azure 门户预览排除资源组部署故障](/documentation/articles/resource-manager-deployment-operations/)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->