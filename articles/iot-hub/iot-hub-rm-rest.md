<properties
    pageTitle="使用资源提供程序 REST API 创建 Azure IoT 中心 | Azure"
    description="如何使用资源提供程序 REST API 创建 IoT 中心。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="52814ee5-bc10-4abe-9eb2-f8973096c2d8"
    ms.service="iot-hub"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date: 04/03/2017
    wacn.date="05/08/2017"
    ms.author="dobett"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="219fb7ccffe772adf0207877c42c363e245b9b81"
    ms.lasthandoff="04/14/2017" />

# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>使用资源提供程序 REST API 创建 IoT 中心 (.NET)
[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>介绍
你可以通过编程方式使用 IoT 中心资源提供程序 REST API 创建和管理 Azure IoT 中心。 本教程介绍如何使用 IoT 中心资源提供程序 REST API 通过 C# 程序创建 IoT 中心。

> [AZURE.NOTE]
> Azure 提供了用于创建和使用资源的两个不同部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。  本文介绍了如何使用 Azure Resource Manager 部署模型。
> 
> 

若要完成本教程，您需要以下各项：

* Visual Studio 2015 或 Visual Studio 2017。
- 有效的 Azure 帐户。 <br/>如果没有帐户，只需花费几分钟就能创建一个 [帐户][lnk-free-trial] 。
- [Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

[AZURE.INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>准备 Visual Studio 项目
1. 在 Visual Studio 中，使用“控制台应用(.NET Framework)”项目模板创建 Visual C# Windows 经典桌面项目。 将该项目命名为 **CreateIoTHubREST**。
2. 在解决方案资源管理器中右键单击你的项目，然后单击“**管理 NuGet 包**”。
3. 在 NuGet 包管理器中，选中“包括预发行版”，然后在“浏览”页上搜索 **Microsoft.Azure.Management.ResourceManager**。 选择该包，单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。
4. 在 NuGet 包管理器中，搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**。  单击“**安装**”，在“**审阅更改**”中单击“**确定**”，然后单击“**我接受**”以接受许可证。
5. 在 Program.cs 中，将现有 **using** 语句替换为以下代码：
   
    
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
    
    
6. 在 Program.cs 中，将占位符值替换为以下静态变量。 在本教程前面的介绍中，你已记下 **ApplicationId**、**SubscriptionId**、**TenantId** 和 **Password**。 **资源组名称**是创建 IoT 中心时要使用的资源组名称，可以是现有的资源组或新资源组。 **IoT 中心名称**是要创建的 IoT 中心的名称，例如 **MyIoTHub**（此名称必须全局唯一，因此应当包含你的姓名或姓名的首字母缩写）。 **部署名称**是部署的名称，例如 **Deployment_01**。

    
	    static string applicationId = "{Your ApplicationId}";
	    static string subscriptionId = "{Your SubscriptionId}";
	    static string tenantId = "{Your TenantId}";
	    static string password = "{Your application Password}";
   
	    static string rgName = "{Resource group name}";
	    static string iotHubName = "{IoT Hub name including your initials}";
    

[AZURE.INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="use-the-resource-provider-rest-api-to-create-an-iot-hub"></a>使用资源提供程序 REST API 创建 IoT 中心
在资源组中使用 IoT 中心资源提供程序 REST API 创建 IoT 中心。 还可使用资源提供程序 REST API 更改现有的 IoT 中心。

1. 将以下方法添加到 Program.cs：
    
    
	    static void CreateIoTHub(string token)
	    {
        
	    }
    

2. 将以下代码添加到 **CreateIoTHub** 方法。该代码创建一个 **HttpClient** 对象，在标头中使用身份验证令牌：

    
	    HttpClient client = new HttpClient();
	    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    

3. 将以下代码添加到 **CreateIoTHub** 方法。 此代码描述要创建的 IoT 中心，并生成 JSON 表示形式。 有关支持 IoT 中心的位置的最新列表，请参阅 [Azure 状态][lnk-status]：

    
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
    

4. 将以下代码添加到 **CreateIoTHub** 方法中。 该代码向 Azure 提交 REST 请求、检查响应，并检索可用于监视部署任务状态的 URL：

    
	    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
	    var requestUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
	    var result = client.PutAsync(requestUri, content).Result;
      
	    if (!result.IsSuccessStatusCode)
	    {
	      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
	      return;
	    }
    
	    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    

5. 将以下代码添加到 **CreateIoTHub** 方法的末尾。该代码使用在上一步检索的 **asyncStatusUri** 地址等待部署完成：

    
	    string body;
	    do
	    {
	      Thread.Sleep(10000);
	      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
	      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    	    } while (body == "{\"status\":\"Running\"}");
    

6. 将以下代码添加到 **CreateIoTHub** 方法的末尾。该代码检索用户创建的 IoT 中心的密钥并将其输出到控制台：

        var listKeysUri = string.Format("https://management.chinacloudapi.cn/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
        var keysresults = client.PostAsync(listKeysUri, null).Result;

        Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);

## <a name="complete-and-run-the-application"></a>完成并运行应用程序
现在，可以调用 **CreateIoTHub** 方法来完成应用程序，然后生成并运行该应用程序。

1. 将以下代码添加到 **Main** 方法末尾：

        CreateIoTHub(token.AccessToken);
        Console.ReadLine();

2. 单击“**生成**”，然后单击“**生成解决方案**”。 更正所有错误。
3. 单击“**调试**”，然后单击“**开始调试**”以运行应用程序。 运行部署可能需要几分钟时间。
4. 若要验证应用程序中是否添加了新 IoT 中心，可以访问 [Azure 门户][lnk-azure-portal] 并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE]
> 本示例应用程序将添加用于对你计费的 S1 标准 IoT 中心。 完成操作后，可以通过 [Azure 门户预览][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet 删除。

## <a name="next-steps"></a>后续步骤
现已使用资源提供程序 REST API 部署了 IoT 中心，接下来可更进一步探索：

- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview] 。

若要详细了解如何开发 IoT 中心，请参阅以下文章：

- [C SDK 简介][lnk-c-sdk]
- [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

- [使用 IOT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/
[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/

[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!--Update_Description:update wording-->