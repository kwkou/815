<properties
	pageTitle="使用 REST API 创建 IoT 中心 | Azure"
	description="遵照本教程开始使用 REST API 创建 IoT 中心。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="05/31/2016"
     wacn.date="07/04/2016"/>

# 教程：使用 C# 程序和 REST API 创建 IoT 中心

[AZURE.INCLUDE [iot-hub-resource-manager-selector](../includes/iot-hub-resource-manager-selector.md)]

## 介绍

你可以通过编程方式使用 [IoT 中心资源提供程序 REST API][lnk-rest-api] 创建和管理 Azure IoT 中心。本教程说明如何使用资源提供程序 REST API 从 C# 程序创建 IoT 中心。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用资源管理器部署模型。

为了完成本教程，你需要有：

- Microsoft Visual Studio 2015。
- 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。
- [Microsoft Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

[AZURE.INCLUDE [iot-hub-prepare-resource-manager](../includes/iot-hub-prepare-resource-manager.md)]

## 准备 Visual Studio 项目

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板创建一个新的 Visual C# Windows 项目。将该项目命名为 **CreateIoTHubREST**。

2. 在解决方案资源管理器中右键单击你的项目，然后单击“管理 NuGet 包”。

3. 在 NuGet 包管理器中，选中“包括预发行版”，然后搜索 **Microsoft.Azure.Management.ResourceManager**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。

4. 在 NuGet 包管理器中，搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。

6. 在 Program.cs 中，将现有 **using** 语句替换为以下内容：

    ```
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    using Newtonsoft.Json;
    ```
    
7. 在 Program.cs 中，将占位符值替换为以下静态变量。在本教程前面的介绍中，你已记下 **ApplicationId**、**SubscriptionId**、**TenantId** 和 **Password**。**Resource group name** 是创建 IoT 中心时要使用的资源组名称，可以是现有的资源组或新资源组。**IoT 中心名称**是要创建的 IoT 中心的名称，例如 **MyIoTHub**（请注意，此名称必须全局唯一，因此应当包含你的姓名或姓名的首字母缩写）。**Deployment name** 是部署的名称，例如 **Deployment\_01**。

    ```
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";
    
    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```

[AZURE.INCLUDE [iot-hub-get-access-token](../includes/iot-hub-get-access-token.md)]

## 使用 REST API 创建 IoT 中心

使用 [IoT 中心 REST API][lnk-rest-api] 在资源组中创建新的 IoT 中心。也可以使用 REST API 更改现有的 IoT 中心。

1. 将以下方法添加到 Program.cs：
    
    ```
    static void CreateIoTHub(string token)
    {
        
    }
    ```

2. 将以下代码添加到 **CreateIoTHub** 方法，以创建 **HttpClient** 对象并在标头中指定身份验证令牌：

    ```
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```

3. 将以下代码添加到 **CreateIoTHub** 方法，以描述如何使用 IoT 中心来创建并生成 JSON 表示法：

    ```
    var description = new
    {
      name = iotHubName,
      location = "China East",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };
    
    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```

4. 将以下代码添加到 **CreateIoTHub** 方法，以向到 Azure 提交 REST 请求、检查响应，并检索可用于监视部署任务状态的 URL：

    ```
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;
      
    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }
    
    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```

5. 将以下代码添加到 **CreateIoTHub** 方法末尾，以使用上一个步骤中检索的 **asyncStatusUri** 地址来等待部署完成：

    ```
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    } while (body == "{\'Status\':\'Running\'}");
    ```

6. 将以下代码添加到 **CreateIoTHub** 方法末尾，以检索创建的 IoT 中心密钥并将其输出到控制台：

    ```
    var listKeysUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2015-08-15-preview", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;
    
    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```
    
## 完成并运行应用程序

现在，可以调用 **CreateIoTHub** 方法来完成应用程序，然后生成并运行该应用程序。

1. 在 **Main** 方法的末尾添加以下代码：

    ```
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```
    
2. 单击“生成”，然后单击“生成解决方案”。更正所有错误。

3. 单击“调试”，然后单击“开始调试”以运行应用程序。运行部署可能需要几分钟时间。

4. 若要验证应用程序中是否添加了新 IoT 中心，你可以访问[门户][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE] 本示例应用程序将添加用于对你计费的 S1 标准 IoT 中心。可以通过[门户][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## 后续步骤

既然你已使用 REST API 部署了一个 IoT 中心，接下来可以进一步进行探索：

- 探索 [IoT 中心资源提供程序 REST API][lnk-rest-api] 的功能。
- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview]。

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://manage.windowsazure.cn
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/
[lnk-rest-api]: https://msdn.microsoft.com/zh-cn/library/mt589014.aspx
[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/

<!---HONumber=Mooncake_0307_2016-->