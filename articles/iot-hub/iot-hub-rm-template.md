<properties
    pageTitle="使用模板创建 Azure IoT 中心 (.NET) | Azure"
    description="如何使用 Azure Resource Manager 模板和 C# 程序创建 IoT 中心。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="a447b40c-c728-487e-875d-db554db5adc3"
    ms.service="iot-hub"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/08/2017"
    wacn.date="03/10/2017"
    ms.author="dobett" />  


# 使用 Azure Resource Manager 模板创建 IoT 中心 \(.NET\)
[AZURE.INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## 介绍
可以使用 Azure Resource Manager 以编程方式创建和管理 Azure IoT 中心。本教程说明如何使用 Azure Resource Manager 模板通过 C\# 程序创建 IoT 中心。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[Azure Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Azure Resource Manager 部署模型。

若要完成本教程，您需要以下各项：

- Microsoft Visual Studio 2015。
* 有效的 Azure 帐户。<br/>如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
* 可用于存储 Azure Resource Manager 模板文件的 [Azure 存储帐户][lnk-storage-account]。
- [Microsoft Azure PowerShell 1.0][lnk-powershell-install] 或更高版本。

[AZURE.INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## 准备 Visual Studio 项目
1. 在 Visual Studio 中，使用**控制台应用程序**项目模板创建一个新的 Visual C\# Windows 项目。将项目命名为 **CreateIoTHub**。
2. 在解决方案资源管理器中右键单击你的项目，然后单击“管理 NuGet 包”。

3. 在 NuGet 包管理器中，选中“包括预发行版”，然后搜索 **Microsoft.Azure.Management.ResourceManager**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。

4. 在 NuGet 包管理器中，搜索 **Microsoft.IdentityModel.Clients.ActiveDirectory**。单击“安装”，在“审阅更改”中单击“确定”，然后单击“我接受”以接受许可证。


6. 在 Program.cs 中，将现有 **using** 语句替换为以下内容：

    
	    using System;
	    using Microsoft.Azure.Management.ResourceManager;
	    using Microsoft.Azure.Management.ResourceManager.Models;
	    using Microsoft.IdentityModel.Clients.ActiveDirectory;
	    using Microsoft.Rest;
    
    
6. 在 Program.cs 中，将占位符值替换为以下静态变量。在本教程前面的介绍中，已记下 **ApplicationId**、**SubscriptionId**、**TenantId** 和 **Password**。**Your Azure Storage account name** 是要在其中存储 Azure Resource Manager 模板文件的 Azure 存储帐户的名称。**Resource group name** 是创建 IoT 中心时使用的资源组的名称，它可以是现有的资源组，也可以是新资源组。**Deployment name** 是部署的名称，例如 **Deployment\_01**。

    
	    static string applicationId = "{Your ApplicationId}";
	    static string subscriptionId = "{Your SubscriptionId}";
	    static string tenantId = "{Your TenantId}";
	    static string password = "{Your application Password}";
	    static string storageAddress = "https://{Your storage account name}.blob.core.chinacloudapi.cn";
	    static string rgName = "{Resource group name}";
	    static string deploymentName = "{Deployment name}";
    

[AZURE.INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## 提交用于创建 IoT 中心的 Azure Resource Manager 模板
使用 JSON 模板和参数文件在资源组中创建 IoT 中心。还可以使用 Azure Resource Manager 模板更改现有 IoT 中心。

1. 在解决方案资源管理器中右键单击项目，然后依次单击“添加”和“新建项”。将名为 **template.json** 的 JSON 文件添加到项目。

2. 将 **template.json** 的内容替换为以下资源定义，以便在**中国东部**区域添加新的标准 IoT 中心：

    
	    {
	      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
	      "contentVersion": "1.0.0.0",
	      "parameters": {
	        "hubName": {
	          "type": "string"
	        }
	      },
	      "resources": [
	      {
	        "apiVersion": "2016-02-03",
	        "type": "Microsoft.Devices/IotHubs",
	        "name": "[parameters('hubName')]",
	        "location": "China East",
	        "sku": {
	          "name": "S1",
	          "tier": "Standard",
	          "capacity": 1
	        },
	        "properties": {
	          "location": "China East"
	        }
	      }
	      ],
	      "outputs": {
	        "hubKeys": {
	          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
	          "type": "object"
	        }
	      }
	    }
    

3. 在解决方案资源管理器中右键单击项目，然后依次单击“添加”和“新建项”。将名为 **parameters.json** 的 JSON 文件添加到项目。
4. 将 **parameters.json** 的内容替换为以下参数信息，以便将新 IoT 中心的名称设置为 **{your initials}mynewiothub** 等。IoT 中心名称必须全局唯一，因此，应包含姓名或姓名首字母缩写：

    
	    {
	      "$schema": "https://schema.management.azure.cn/schemas/2015-01-01/deploymentParameters.json#",
	      "contentVersion": "1.0.0.0",
	      "parameters": {
	        "hubName": { "value": "mynewiothub" }
	      }
	    }
    

5. 在**服务器资源管理器**中，连接到 Azure 订阅，并在 Azure 存储帐户中创建名为 **templates** 的容器。在“属性”面板中，将 **templates** 容器的“公共读取访问权限”权限设置为“Blob”。
6. 在**服务器资源管理器**中，右键单击 **templates** 容器，然后单击“查看 Blob 容器”。单击“上传 Blob”按钮，选择 **parameters.json** 和 **templates.json** 这两个文件，然后单击“打开”，将 JSON 文件上传到 **templates** 容器。包含 JSON 数据的 Blob 的 URL 如下：

    
	    https://{Your storage account name}.blob.core.chinacloudapi.cn/templates/parameters.json
	    https://{Your storage account name}.blob.core.chinacloudapi.cn/templates/template.json
    

7. 将以下方法添加到 Program.cs：
   
    
	    static void CreateIoTHub(ResourceManagementClient client)
	    {
   
	    }
    

8. 在 **CreateIoTHub** 方法中添加以下代码，向 Azure Resource Manager 提交模板和参数文件：

    
	    var createResponse = client.Deployments.CreateOrUpdate(
	        rgName,
	        deploymentName,
	        new Deployment()
	        {
	          Properties = new DeploymentProperties
	          {
	            Mode = DeploymentMode.Incremental,
	            TemplateLink = new TemplateLink
	            {
	              Uri = storageAddress + "/templates/template.json"
	            },
	            ParametersLink = new ParametersLink
	            {
	              Uri = storageAddress + "/templates/parameters.json"
	            }
	          }
	        });
    

9. 在 **CreateIoTHub** 方法中添加以下代码，显示新 IoT 中心的状态和密钥：

    
	    string state = createResponse.Properties.ProvisioningState;
	    Console.WriteLine("Deployment state: {0}", state);
   
	    if (state != "Succeeded")
	    {
	      Console.WriteLine("Failed to create iothub");
	    }
	    Console.WriteLine(createResponse.Properties.Outputs);
    

## 完成并运行应用程序
现在，可以调用 **CreateIoTHub** 方法来完成应用程序，然后生成并运行该应用程序。

1. 在 **Main** 方法末尾添加以下代码：

    
	    CreateIoTHub(client);
	    Console.ReadLine();
    
    
2. 依次单击“生成”和“生成解决方案”。更正所有错误。
3. 单击“调试”，然后单击“开始调试”，运行应用程序。运行部署可能需要几分钟时间。

4. 若要验证应用程序中是否添加了新 IoT 中心，可以访问[门户预览][lnk-azure-portal]并查看资源列表，或使用 **Get-AzureRmResource** PowerShell cmdlet。

> [AZURE.NOTE] 此示例应用程序添加用于计费的 S1 标准 IoT 中心。可以通过 [Azure 门户预览][lnk-azure-portal]删除该 IoT 中心，或者在完成后使用 **Remove-AzureRmResource** PowerShell cmdlet。

## 后续步骤
现在，已使用 Azure Resource Manager 模板和 C\# 程序部署了一个 IoT 中心，接下来可以进一步进行探索：


既然你已使用 REST API 和 C# 程序部署了一个 IoT 中心，接下来可以进一步进行探索：


- 有关 Azure Resource Manager 功能的详细信息，请参阅 [Azure Resource Manager 概述][lnk-azure-rm-overview]。

若要深入了解如何开发 IoT 中心，请参阅以下内容：

- [C SDK 简介][lnk-c-sdk]
- [Azure IoT SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-azure-portal]: https://portal.azure.cn/
[lnk-powershell-install]: /documentation/articles/powershell-install-configure/

[lnk-azure-rm-overview]: /documentation/articles/resource-group-overview/
[lnk-storage-account]: /documentation/articles/storage-create-storage-account/
[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update code and wording-->