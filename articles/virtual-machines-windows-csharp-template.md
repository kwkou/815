<!-- rename to virtual-machines-windows-csharp-template -->

<properties
	pageTitle="使用模板部署 Azure 资源 | Azure"
	description="学习如何使用 Azure 资源管理库中的某些可用客户端来部署虚拟机、虚拟网络和存储帐户。"
	services="virtual-machines-windows,virtual-networks,storage"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="01/06/2016"
	wacn.date="06/07/2016"/>

# 使用 .NET 库和模板部署 Azure 资源

>[AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](documentation/articles/resource-manager-deployment-model)。这篇文章介绍如何使用资源管理器部署模型，Microsoft 建议大多数新部署使用资源管理器模型替代经典部署模型。

使用资源组和模板，可以统一管理为你的应用程序提供支持的所有资源。本教程说明如何使用 Azure 资源管理库中的某些可用客户端，以及如何生成模板用于部署虚拟机、虚拟网络和存储帐户。

[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]

若要完成本教程，你还需要：

- [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)
- [Azure 存储帐户](/documentation/articles/storage-create-storage-account)
- [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)

>[AZURE.NOTE] 你此时创建的存储帐户用于存储模板。当你部署用来存储虚拟机磁盘的模板时，会创建另一个存储帐户。在此存储帐户中创建一个名为模板的容器。

[AZURE.INCLUDE [powershell 预览](../includes/powershell-preview-inline-include.md)]

完成这些步骤大约需要 30 分钟。

## 步骤 1：将应用程序添加到 Azure AD 并设置权限

若要使用 Azure AD 要对发往 Azure 资源管理器的请求进行身份验证，必须在默认目录中添加一个应用程序。执行以下操作可以添加应用程序：

1. 打开 Azure PowerShell 命令提示符，运行以下命令，然后在出现提示时输入订阅的凭据：

			Login-AzureRmAccount

2. 在以下命令中的 {password} 替换为要使用的密码，然后运行以下命令创建应用程序：

			New-AzureRmADApplication -DisplayName "My AD Application 1" -HomePage "https://myapp1.com" -IdentifierUris "https://myapp1.com"  -Password "{password}"

	>[AZURE.NOTE] 请记下创建应用程序之后返回的应用程序标识符，因为下一步需要使用它。你也可以在 Azure 门户的 Active Directory 部分，从应用程序的客户端 ID 字段中找到应用程序标识符。

3. 将 {application-id} 替换为刚才记下的标识符，然后创建该应用程序的服务主体：

			New-AzureRmADServicePrincipal -ApplicationId {application-id}

4. 设置应用程序的使用权限：

			New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName "https://myapp1.com"

## 步骤 2：创建 Visual Studio 项目、模板文件和参数文件

### 创建模板文件

借助 Azure 资源管理器模板，你可以使用资源和关联部署参数的 JSON 描述来统一部署和管理 Azure 资源。在本教程中生成的模板非常类似于可在模板库中找到的模板。

在 Visual Studio 中执行以下操作：

1. 单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“添加”>“新建项”。

4. 在“添加新项”窗口中，选择“文本文件”，输入 *VirtualMachineTemplate.json* 作为名称，然后单击“添加”。

5. 打开 VirtualMachineTemplate.json 文件，然后添加左括号和右括号、所需的架构元素以及所需的 contentVersion 元素：

	```
	{
		"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
		"contentVersion": "1.0.0.0",
	}
	```

6. [参数](/documentation/articles/resource-group-authoring-templates#parameters)并不总是必需的，但它们有助于简化模板管理。它们描述了值的类型、所需的默认值，有时还描述参数的允许值。对于本教程，用于创建虚拟机、存储帐户和虚拟网络的参数已添加到模板中。

    在 ContentVersion 元素后面添加 parameters 元素及其子元素：

	```
	{
		"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"newStorageAccountName": { "type": "string" },
			"adminUserName": { "type": "string" },
			"adminPassword": { "type": "securestring" },
			"dnsNameForPublicIP": { "type": "string" },
			"windowsOSVersion": {
				"type": "string",
				"defaultValue": "2012-R2-Datacenter",
				"allowedValues": [ "2008-R2-SP1", "2012-Datacenter", "2012-R2-Datacenter" ]
			}
		},
 	}
	```

7. 可以在模板中使用[变量](/documentation/articles/resource-group-authoring-templates#variables)来指定可能经常发生变化的值，或者需要从参数值的组合创建的值。

    在 parameters 节的后面添加 variables 元素：

	```
	{
		"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"newStorageAccountName": { "type": "string" },
			"adminUsername": { "type": "string" },
			"adminPassword": { "type": "securestring" },
			"dnsNameForPublicIP": { "type": "string" },
			"windowsOSVersion": {
				"type": "string",
				"defaultValue": "2012-R2-Datacenter",
				"allowedValues": ["2008-R2-SP1", "2012-Datacenter", "2012-R2-Datacenter"]
			}
		},
		"variables": {
			"location": "China North",
			"imagePublisher": "MicrosoftWindowsServer",
			"imageOffer": "WindowsServer",
			"OSDiskName": "osdiskforwindowssimple",
			"nicName": "myVMnic1",
			"addressPrefix": "10.0.0.0/16",
			"subnetName": "Subnet",
			"subnetPrefix": "10.0.0.0/24",
			"storageAccountType": "Standard_LRS",
			"publicIPAddressName": "myPublicIP",
			"publicIPAddressType": "Dynamic",
			"vmStorageAccountContainerName": "vhds",
			"vmName": "MyWindowsVM",
			"vmSize": "Standard_A2",
			"virtualNetworkName": "MyVNET",
			"vnetID":"[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
			"subnet1Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]"
		},
	}
	```

8. 接下来将在模板中定义[资源](/documentation/articles/resource-group-authoring-templates#resources)，例如虚拟机、虚拟网络和存储帐户。

    在 variables 节的后面添加 resources 节：

	```
	{
		"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"newStorageAccountName": { "type": "string" },
			"adminUsername": { "type": "string" },
			"adminPassword": { "type": "securestring" },
			"dnsNameForPublicIP": { "type": "string" },
			"windowsOSVersion": {
				"type": "string",
				"defaultValue": "2012-R2-Datacenter",
				"allowedValues": [ "2008-R2-SP1", "2012-Datacenter", "2012-R2-Datacenter"]
			}
		},
		"variables": {
			"location": "China North",
			"imagePublisher": "MicrosoftWindowsServer",
			"imageOffer": "WindowsServer",
			"OSDiskName": "osdiskforwindowssimple",
			"nicName": "myVMnic1",
			"addressPrefix": "10.0.0.0/16",
			"subnetName": "Subnet",
			"subnetPrefix": "10.0.0.0/24",
			"storageAccountType": "Standard_LRS",
			"publicIPAddressName": "myPublicIP",
			"publicIPAddressType": "Dynamic",
			"vmStorageAccountContainerName": "vhds",
			"vmName": "MyWindowsVM",
			"vmSize": "Standard_A2",
			"virtualNetworkName": "MyVNET",
			"vnetID":"[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
			"subnet1Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]"
		},
		"resources": [
			{
  			"apiVersion": "2015-05-01-preview",
				"type": "Microsoft.Storage/storageAccounts",
				"name": "[parameters('newStorageAccountName')]",
				"location": "[variables('location')]",
				"properties": {
					"accountType": "[variables('storageAccountType')]"
				}
			},
			{
				"apiVersion": "2015-05-01-preview",
				"type": "Microsoft.Network/publicIPAddresses",
				"name": "[variables('publicIPAddressName')]",
				"location": "[variables('location')]",
				"properties": {
					"publicIPAllocationMethod": "[variables('publicIPAddressType')]",
					"dnsSettings": {
						"domainNameLabel": "[parameters('dnsNameForPublicIP')]"
					}
				}
			},
			{
				"apiVersion": "2015-05-01-preview",
				"type": "Microsoft.Network/virtualNetworks",
				"name": "[variables('virtualNetworkName')]",
				"location": "[variables('location')]",
				"properties": {
					"addressSpace": {
						"addressPrefixes": [ "[variables('addressPrefix')]" ]
					},
					"subnets": [ {
						"name": "[variables('subnetName')]",
						"properties": {
							"addressPrefix": "[variables('subnetPrefix')]"
						}
					} ]
				}
			},
			{
				"apiVersion": "2015-05-01-preview",
				"type": "Microsoft.Network/networkInterfaces",
				"name": "[variables('nicName')]",
				"location": "[variables('location')]",
				"dependsOn": [
					"[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
					"[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
				],
				"properties": {
					"ipConfigurations": [ {
						"name": "ipconfig1",
						"properties": {
							"privateIPAllocationMethod": "Dynamic",
							"publicIPAddress": {
								"id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]"
							},
							"subnet": {
								"id": "[variables('subnetRef')]"
							}
						}
					} ]
				}
			},
			{
				"apiVersion": "2015-06-15",
				"type": "Microsoft.Compute/virtualMachines",
				"name": "[variables('vmName')]",
				"location": "[variables('location')]",
				"dependsOn": [
					"[concat('Microsoft.Storage/storageAccounts/', parameters('newStorageAccountName'))]",
					"[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
				],
				"properties": {
					"hardwareProfile": {
						"vmSize": "[variables('vmSize')]"
					},
					"osProfile": {
						"computerName": "[variables('vmName')]",
						"adminUsername": "[parameters('adminUsername')]",
						"adminPassword": "[parameters('adminPassword')]",
					},
					"storageProfile": {
						"imageReference": {
							"publisher": "[variables('imagePublisher')]",
							"offer": "[variables('imageOffer')]",
							"sku": "[parameters('windowsOSVersion')]",
							"version" : "latest"
						},
						"osDisk": {
							"name": "osdisk",
							"vhd": {
								"uri": "[concat('http://',parameters('newStorageAccountName'),'.blob.core.chinacloudapi.cn/',variables('vmStorageAccountContainerName'),'/',variables('OSDiskName'),'.vhd')]"
							},
							"caching": "ReadWrite",
							"createOption": "FromImage"
						}
					},
					"networkProfile": {
						"networkInterfaces" : [ {
							"id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
						} ]
					}
				}
			} ]
		}
	```

9. 保存创建的模板文件。

### 创建参数文件

若要为模板中定义的资源参数指定值，请创建一个包含值的参数文件，并连同模板一起将它提交到资源管理器。在 Visual Studio 中执行以下操作：

1. 在解决方案资源管理器中右键单击项目名称，然后单击“添加”>“新建项”。

2. 在“添加新项”窗口中，选择“文本文件”，输入 *Parameters.json* 作为名称，然后单击“添加”。

3. 打开 parameters.json 文件，然后添加以下 JSON 内容：

	```
	{
		"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
		"contentVersion": "1.0.0.0",
		"parameters": {
			"newStorageAccountName": { "value": "mytestsa1" },
			"adminUserName": { "value": "mytestacct1" },
			"adminPassword": { "value": "mytestpass1" },
			"dnsNameForPublicIP": { "value": "mytestdns1" }
		}
	}
	```

    >[AZURE.NOTE] 本教程创建运行 Windows Server 操作系统版本的虚拟机。若要详细了解如何选择其他映像，请参阅[使用 Windows PowerShell 和 Azure CLI 来导航和选择 Azure 虚拟机映像](/documentation/articles/virtual-machines-windows-cli-ps-findimage)。


4. 保存创建的参数文件。

### 上载文件

Azure 资源管理器将从 Azure 存储帐户访问模板文件和参数文件。若要将文件放置在你创建的第一个存储中，请执行以下操作：

1. 打开服务器资源管理器，然后导航到存储帐户中要放置文件的容器。对于本教程，用于放置模板的容器名为 templates。

2. 在 templates 容器窗格的右上角，单击“上载 Blob”图标，浏览到你创建的 VirtualMachineTemplate.json 文件，然后单击“打开”。

3. 再次单击“上载 Blob”图标，浏览到你创建的 Parameters.json 文件，然后单击“打开”。

## 步骤 3：安装库

使用 NuGet 包是安装完成本教程所需的库的最简单方法。你必须安装 Azure 资源管理库和 Azure Active Directory 身份验证库。若要在 Visual Studio 中获取这些库，请执行以下操作：

1. 在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

2. 在搜索框中键入 *Active Directory*，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

3. 在页面顶部，选择“包括预发行版”。在搜索框中键入 *Microsoft.Azure.Management.Resources*，单击“Azure 资源管理库”旁边的“安装”，然后根据说明安装该包。

现在，你可以开始使用这些库来创建应用程序了。

##步骤 4：创建用于对请求进行身份验证的凭据

安装 Azure Active Directory 应用程序并安装身份验证库后，你可以将应用程序信息格式化为凭据，以用于对发往 Azure 资源管理器的请求进行身份验证。请执行以下操作：

1. 打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

	```
	using Microsoft.Azure;
	using Microsoft.IdentityModel.Clients.ActiveDirectory;
	using Microsoft.Azure.Management.Resources;
	using Microsoft.Azure.Management.Resources.Models;
	using Microsoft.Rest;
	```

2.	将以下方法添加到 Program 类，以获取创建凭据所需的令牌：

	```
	private static string GetAuthorizationHeader()
	{
		ClientCredential cc = new ClientCredential("{application-id}", "{password}");
		var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
		var result = context.AcquireTokenAsync("https://management.azure.cn/", cc);
		if (result == null)
		{
			throw new InvalidOperationException("Failed to obtain the JWT token");
		}

		string token = result.AccessToken;

		return token;
	}
	```

	将 {application-id} 替换为前面记下的应用程序标识符，将 {password} 替换为你为 AD 应用程序选择的密码，将 {tenant-id} 替换为订阅的租户标识符。可以通过运行 Get-AzureSubscription 找到租户 ID。

3. 将以下代码添加到 Program.cs 文件中的 Main 方法，以创建凭据：

	```
	var token = GetAuthorizationHeader();
	var credential = new TokenCredentials(token);
	```

4. 保存 Program.cs 文件。


## 步骤 5：添加代码以部署模板

资源始终从模板部署到资源组。可以使用 [ResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.models.resourcegroup.aspx) 和 [ResourceManagementClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx) 类来创建要将资源部署到的资源组。

1. 将以下方法添加到 Program 类，以创建资源组：

	```
	public static void CreateResourceGroup(
		TokenCredentials credential,
		string groupName,
		string subscriptionId,
		string location)
	{
		Console.WriteLine("Creating the resource group...");
		var resourceManagementClient = new ResourceManagementClient(credential);
		resourceManagementClient.SubscriptionId = subscriptionId;
		var resourceGroup = new ResourceGroup {
			Location = location
		};
		var rgResult = resourceManagementClient.ResourceGroups.CreateOrUpdate(groupName, resourceGroup);
		Console.WriteLine(rgResult.StatusCode.Properties.ProvisioningState);
	}
	```

2. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

	```
	CreateResourceGroup(
		credential,
		"{group-name}",
		"{subscription-id}",
		"{location}");
		Console.ReadLine();
	```

	将 {group-name} 替换为你想要用于资源组的名称。将 {subscription-id} 替换为订阅标识符，你可以通过运行 Get-AzureSubscription 获取此标识符。将位置替换为你想要创建资源的区域（例如“中国北部”）。

3. 将以下方法添加到 Program 类，以使用你定义的模板将资源部署到资源组：

	```
	public static void CreateTemplateDeployment(
		TokenCredentials credential,
		string groupName,
		string storageName,
		string deploymentName,
		string subscriptionId)
	{
		Console.WriteLine("Creating the template deployment...");
		var deployment = new Deployment();
		deployment.Properties = new DeploymentProperties
		{
			Mode = DeploymentMode.Incremental,
			TemplateLink = new TemplateLink
			{
				Uri = "https://{storage-name}.blob.core.chinacloudapi.cn/templates/VirtualMachineTemplate.json"
			},
			ParametersLink = new ParametersLink
			{
				Uri = "https://{storage-name}.blob.core.chinacloudapi.cn/templates/Parameters.json"
			}
		};
		var resourceManagementClient = new ResourceManagementClient(credential);
		resourceManagementClient.SubscriptionId = subscriptionId;
		var dpResult = resourceManagementClient.Deployments.CreateOrUpdate(
			groupName,
			deploymentName,
			deployment);
		Console.WriteLine(dpResult.Properties.ProvisioningState);
	}
	```

	将 {storage-name} 替换为前面放置文件的帐户的名称。

4. 将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

	```
	CreateTemplateDeployment(
		credential,
		"{group-name}",
		"{storage-name}",
		"{deployment-name}",
		"{subscription-id}");
	Console.ReadLine();
	```

    将 {group-name} 替换为你创建的资源组的名称。将 {storage-name} 替换为放置模板文件的存储帐户的名称。将 {deployment-name} 替换为你想要用来识别资源部署集的名称。将 {subscription-id} 替换为订阅标识符，你可以通过运行 Get-AzureSubscription 获取此标识符。

##步骤 6：添加代码以删除资源

由于你需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。你不需要从资源组中分别删除每个资源，而是可以删除资源组，这样就会自动删除它包含的所有资源。

1.	将以下方法添加到 Program 类，以删除资源组：

	```
	public static void DeleteResourceGroup(
		TokenCredentials credential,
		string groupName,
		string subscriptionId)
	{
		Console.WriteLine("Deleting resource group...");
		var resourceGroupClient = new ResourceManagementClient(credential);
		resourceGroupClient.ResourceGroups.DeleteAsync(groupName);
	}
	```

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

	```
	DeleteResourceGroup(
		credential,
		"{group-name}",
		"{subscription-id}");
	Console.ReadLine();
	```
    将 {group-name} 替换为你创建的资源组的名称。将 {subscription-id} 替换为订阅标识符，你可以通过运行 Get-AzureSubscription 获取此标识符。

##步骤 7：运行控制台应用程序

1.	若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

2.	在显示“已接受”状态之后按 “Enter”。

	控制台应用程序从头到尾完成运行大约需要 5 分钟时间。在按 Enter 开始删除资源之前，你可能需要在 Azure 门户中花几分钟时间来验证资源的创建。

3. 在 Azure 门户中浏览到“审核日志”，以查看资源的状态：

	![创建 AD 应用程序](./media/virtual-machines-windows-csharp-template/crpportal.png)

<!---HONumber=Mooncake_0411_2016-->