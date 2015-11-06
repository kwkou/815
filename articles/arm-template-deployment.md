<properties 
	pageTitle="使用模板部署 Azure 资源" 
	description="学习如何使用 Azure 资源管理库中的某些可用客户端来部署虚拟机、虚拟网络和存储帐户" 
	services="virtual-machines,virtual-networks,storage" 
	documentationCenter="" 
	authors="davidmu1" 
	manager="timlt" 
	editor="tysonn" 
	tags="azure-resource-manager"/>

<tags
	ms.service="multiple"
	wacn.date="09/15/2015"
	ms.date="06/15/2015"/>

# 使用 .NET 库和模板部署 Azure 资源

使用资源组和模板，可以统一管理为你的应用程序提供支持的所有资源。本教程说明如何使用 Azure 资源管理库中的某些可用客户端，以及如何生成模板用于部署虚拟机、虚拟网络和存储帐户。

[AZURE.INCLUDE [trial-note](../includes/free-trial-note.md)]

若要完成本教程，你还需要：

- [Visual Studio](http://msdn.microsoft.com/zh-cn/library/dd831853.aspx)
- [Azure 存储帐户](/documentation/articles/storage-create-storage-account)
- [Windows Management Framework 3.0](http://www.microsoft.com/zn-ch/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/zh-cn/download/details.aspx?id=40855)
- [Azure PowerShell](/documentation/articles/powershell-install-configure)

完成这些步骤大约需要 30 分钟。

## 步骤 1：将应用程序添加到 Azure AD 并设置权限

若要使用 Azure AD 要对发往 Azure 资源管理器的请求进行身份验证，必须在默认目录中添加一个应用程序。执行以下操作可以添加应用程序：

1. 打开 Azure PowerShell 命令提示符，然后运行以下命令：

        Switch-AzureMode –Name AzureResourceManager

2. 设置你要用于本教程的 Azure 帐户。运行以下命令，并在出现提示时输入订阅的凭据：

	    Add-AzureAccount -Environment AzureChinaCloud

3. 在以下命令中的 {password} 替换为要使用的密码，然后运行以下命令创建应用程序：

	    New-AzureADApplication -DisplayName "My AD Application 1" -HomePage "https://myapp1.com" -IdentifierUris "https://myapp1.com"  -Password "{password}"

4. 记下前一步骤的响应中提供的 ApplicationId 值。本教程后面的步骤会用到此值：

	![创建 AD 应用程序](./media/arm-template-deployment/azureapplicationid.png)

	>[AZURE.NOTE]你也可以在管理门户上，从应用程序的客户端 id 字段中找到应用程序标识符。

5. 将 {application-id} 替换为刚才记下的标识符，然后创建该应用程序的服务主体：

        New-AzureADServicePrincipal -ApplicationId {application-id}

6. 设置应用程序的使用权限：

	    New-AzureRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName "https://myapp1.com"

## 步骤 2：创建 Visual Studio 项目、模板文件和参数文件

###创建模板文件

借助 Azure 资源管理器模板，你可以使用资源和关联部署参数的 JSON 描述来统一部署和管理 Azure 资源。在 Visual Studio 中执行以下操作：

1. 单击“文件”>“新建”>“项目”。

2. 在“模板”>“Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

3. 在解决方案资源管理器中右键单击项目名称，然后单击“添加”>“新建项”。

4.	在“添加新项”窗口中，选择“文本文件”，输入 *VirtualMachineTemplate.json* 作为名称，然后单击“添加”。

5.	打开 VirtualMachineTemplate.json 文件，然后添加左括号和右括号、所需的架构元素以及所需的 contentVersion 元素：

        {
            "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
            "contentVersion": "1.0.0.0",
        }

6. [参数](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx#parameters)并不总是必需的，但它们有助于简化模板管理。它们描述了值的类型、所需的默认值，有时还描述参数的允许值。对于本教程，用于创建虚拟机、存储帐户和虚拟网络的参数已添加到模板中。

    在 ContentVersion 元素后面添加 parameters 元素及其子元素：

		{
		  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
		  "contentVersion": "1.0.0.0",
		  "parameters": {
		    "location": { "type": "String", "defaultValue" : "China North", "allowedValues": [ "China North", "China East" ] },
            "newStorageAccountName": { "type": "string" },
            "storageAccountType": { "type": "string", "defaultValue": "Standard_LRS", "allowedValues": [ "Standard_LRS", "Standard_GRS" ] },
            "publicIPAddressName": { "type": "string" },
            "publicIPAddressType" : { "type" : "string", "defaultValue" : "Dynamic", "allowedValues" : [ "Dynamic" ] },
            "vmStorageAccountContainerName": { "type": "string", "defaultValue": "vhds" },
            "vmName": { "type": "string" },
            "vmSize": { "type": "string", "defaultValue": "Standard_A0", "allowedValues" : [ "Standard_A0", "Standard_A1" ] },
            "vmSourceImageName": { "type": "string" },
            "adminUserName": { "type": "string" },
            "adminPassword": { "type": "securestring" },
            "virtualNetworkName":{ "type" : "string" },
            "addressPrefix": { "type" : "string", "defaultValue" : "10.0.0.0/16" },
            "subnet1Name": { "type" : "string", "defaultValue" : "Subnet-1" },
            "subnet2Name": { "type" : "string", "defaultValue" : "Subnet-2" },
            "subnet1Prefix" : { "type" : "string", "defaultValue" : "10.0.0.0/24" },
            "subnet2Prefix" : { "type" : "string", "defaultValue" : "10.0.1.0/24" },
            "dnsName" : { "type" : "string" },
            "subscriptionId": { "type" : "string" },
            "nicName": { "type" : "string" }
          },
        }

7.	可以在模板中使用[变量](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx#variables)来指定可能经常发生变化的值，或者需要从参数值的组合创建的值。

    在 parameters 节的后面添加 variables 元素：

		{
		  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
		  "contentVersion": "1.0.0.0",
		  "parameters": {
		    "location": { "type": "String", "defaultValue" : "China North", "allowedValues": [ "China North", "China East" ] },
            "newStorageAccountName": { "type": "string" },
            "storageAccountType": { "type": "string", "defaultValue": "Standard_LRS", "allowedValues": [ "Standard_LRS", "Standard_GRS" ] },
            "publicIPAddressName": { "type": "string" },
            "publicIPAddressType" : { "type" : "string", "defaultValue" : "Dynamic", "allowedValues" : [ "Dynamic" ] },
            "vmStorageAccountContainerName": { "type": "string", "defaultValue": "vhds" },
            "vmName": { "type": "string" },
            "vmSize": { "type": "string", "defaultValue": "Standard_A0", "allowedValues" : [ "Standard_A0", "Standard_A1" ] },
            "vmSourceImageName": { "type": "string" },
            "adminUserName": { "type": "string" },
            "adminPassword": { "type": "securestring" },
            "virtualNetworkName":{ "type" : "string" },
            "addressPrefix": { "type" : "string", "defaultValue" : "10.0.0.0/16" },
            "subnet1Name": { "type" : "string", "defaultValue" : "Subnet-1" },
            "subnet2Name": { "type" : "string", "defaultValue" : "Subnet-2" },
            "subnet1Prefix" : { "type" : "string", "defaultValue" : "10.0.0.0/24" },
            "subnet2Prefix" : { "type" : "string", "defaultValue" : "10.0.1.0/24" },
            "dnsName" : { "type" : "string" },
            "subscriptionId": { "type" : "string" },
            "nicName": { "type" : "string" }
          },
          "variables": {
            "sourceImageName" : "[concat('/',parameters('subscriptionId'),'/services/images/',parameters('vmSourceImageName'))]",
            "vnetID":"[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
            "subnet1Ref" : "[concat(variables('vnetID'),'/subnets/',parameters('subnet1Name'))]"
          },
        }

8.	接下来将在模板中定义[资源](https://msdn.microsoft.com/zh-cn/library/azure/dn835138.aspx#resources)，例如虚拟机、虚拟网络和存储帐户。

    在 variables 节的后面添加 resources 节：

		{
		  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
		  "contentVersion": "1.0.0.0",
		  "parameters": {
		    "location": { "type": "String", "defaultValue" : "China North", "allowedValues": [ "China North", "China East" ] },
            "newStorageAccountName": { "type": "string" },
            "storageAccountType": { "type": "string", "defaultValue": "Standard_LRS", "allowedValues": [ "Standard_LRS", "Standard_GRS" ] },
            "publicIPAddressName": { "type": "string" },
            "publicIPAddressType" : { "type" : "string", "defaultValue" : "Dynamic", "allowedValues" : [ "Dynamic" ] },
            "vmStorageAccountContainerName": { "type": "string", "defaultValue": "vhds" },
            "vmName": { "type": "string" },
            "vmSize": { "type": "string", "defaultValue": "Standard_A0", "allowedValues" : [ "Standard_A0", "Standard_A1" ] },
            "vmSourceImageName": { "type": "string" },
            "adminUserName": { "type": "string" },
            "adminPassword": { "type": "securestring" },
            "virtualNetworkName":{ "type" : "string" },
            "addressPrefix": { "type" : "string", "defaultValue" : "10.0.0.0/16" },
            "subnet1Name": { "type" : "string", "defaultValue" : "Subnet-1" },
            "subnet2Name": { "type" : "string", "defaultValue" : "Subnet-2" },
            "subnet1Prefix" : { "type" : "string", "defaultValue" : "10.0.0.0/24" },
            "subnet2Prefix" : { "type" : "string", "defaultValue" : "10.0.1.0/24" },
            "dnsName" : { "type" : "string" },
            "subscriptionId": { "type" : "string" },
            "nicName": { "type" : "string" }
          },
          "variables": {
            "sourceImageName" : "[concat('/',parameters('subscriptionId'),'/services/images/',parameters('vmSourceImageName'))]",
            "vnetID":"[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
            "subnet1Ref" : "[concat(variables('vnetID'),'/subnets/',parameters('subnet1Name'))]"
          },
          "resources": [ {
            "apiVersion": "2014-12-01-preview",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('newStorageAccountName')]",
            "location": "[parameters('location')]",
            "properties": { "accountType": "[parameters('storageAccountType')]" }
          },
          {
            "apiVersion": "2014-12-01-preview",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[parameters('publicIPAddressName')]",
            "location": "[parameters('location')]",
            "properties": {
              "publicIPAllocationMethod": "[parameters('publicIPAddressType')]",
              "dnsSettings": { "domainNameLabel": "[parameters('dnsName')]" }
            }
          },
          {
            "apiVersion": "2014-12-01-preview",
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[parameters('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "properties": {
              "addressSpace": { "addressPrefixes": [ "[parameters('addressPrefix')]" ] },
              "subnets": [ {
                "name": "[parameters('subnet1Name')]",
                "properties": { "addressPrefix": "[parameters('subnet1Prefix')]" }
              },
              {
                "name": "[parameters('subnet2Name')]",
                "properties": { "addressPrefix": "[parameters('subnet2Prefix')]" }
              } ]
            }
          },
          {
            "apiVersion": "2014-12-01-preview",
            "type": "Microsoft.Network/networkInterfaces",
            "name": "[parameters('nicName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
              "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
            ],
            "properties": {
              "ipConfigurations": [ {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": {
                    "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIPAddressName'))]"
                  },
                  "subnet": { "id": "[variables('subnet1Ref')]" }
                }
              } ]
            }
          },
          {
            "apiVersion": "2014-12-01-preview",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[parameters('vmName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
              "[concat('Microsoft.Storage/storageAccounts/', parameters('newStorageAccountName'))]",
              "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]"
            ],
            "properties": {
              "hardwareProfile": { "vmSize": "[parameters('vmSize')]" },
              "osProfile": {
                "computername": "[parameters('vmName')]",
                "adminUsername": "[parameters('adminUsername')]",
                "adminPassword": "[parameters('adminPassword')]",
                "windowsProfile": { "provisionVMAgent": "true" }
              },
              "storageProfile": {
                "sourceImage": { "id": "[variables('sourceImageName')]" },
                "destinationVhdsContainer" : "[concat('http://',parameters('newStorageAccountName'),'.blob.core.chinacloudapi.cn/',parameters('vmStorageAccountContainerName'),'/')]"
              },
              "networkProfile": {
                "networkInterfaces" : [ {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                } ]
              }
            }
          } ]
        }

9.	保存创建的模板文件。

###创建参数文件

若要为模板中定义的资源参数指定值，请创建一个包含值的参数文件，并连同模板一起将它提交到资源管理器。在 Visual Studio 中执行以下操作：

1.	在解决方案资源管理器中右键单击项目名称，然后单击“添加”>“新建项”。

2.	在“添加新项”窗口中，选择“文本文件”，输入 *Parameters.json* 作为名称，然后单击“添加”。

3.	打开 parameters.json 文件，然后添加以下 JSON 内容：

        {
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vmName": { "value": "mytestvm1" },
            "newStorageAccountName": { "value": "mytestsa1" },
            "storageAccountType": { "value": "Standard_LRS" },
            "publicIPaddressName": { "value": "mytestip1" },
            "location": { "value": "China North" },
            "vmStorageAccountContainerName": { "value": "vhds" },
            "vmSize": { "value": "Standard_A1" },
            "subscriptionId": { "value": "{subscription-id}" },
            "vmSourceImageName": { "value": "{source-image-name}" },
            "adminUserName": { "value": "mytestacct1" },
            "adminPassword": { "value": "mytestpass1" },
            "virtualNetworkName": { "value": "mytestvn1" },
            "dnsName": { "value": "mytestdns1" },
            "nicName": { "value": "mytestnic1" }
          }
        }

    >[AZURE.NOTE]映像库中的映像 vhd 名称经常发生变化，因此你需要获取当前映像名称以部署虚拟机。为此，请参阅[使用 Windows PowerShell 管理映像 (Windows)](https://msdn.microsoft.com/zh-cn/library/azure/dn790330.aspx)，然后将 {source-image-name} 替换为要使用的 vhd 文件名。例如“a699494373c04fc0bc8f2bb1389d6106\_\_Windows-Server-2012-R2-201412.01-en.us-127GB.vhd”。将 {subscription-id} 替换为订阅的标识符。


4.	保存创建的参数文件。

Azure 资源管理器将从 Azure 存储帐户访问模板文件和参数文件。若要将这些文件放入存储，请执行以下操作：

1.	打开服务器资源管理器，然后导航到存储帐户中要放置文件的容器。对于本教程，用于放置模板的容器名为 templates。

2.	在 templates 容器窗格的右上角，单击“上载 Blob”图标，浏览到你创建的 VirtualMachineTemplate.json 文件，然后单击“打开”。

3. 再次单击“上载 Blob”图标，浏览到你创建的 Parameters.json 文件，然后单击“打开”。

##步骤 3：安装库

使用 NuGet 包是安装完成本教程所需的库的最简单方法。你必须安装 Azure 资源管理库和 Azure Active Directory 身份验证库。若要在 Visual Studio 中获取这些库，请执行以下操作：

1.	在解决方案资源管理器中右键单击项目名称，然后单击“管理 NuGet 包”。

2.	在搜索框中键入 *Active Directory*，单击“Active Directory 身份验证库”包旁边的“安装”，然后根据说明安装该包。

3.	在页面顶部，选择“包括预发行版”。在搜索框中键入 *Azure Resource Management*，单击“Microsoft Azure 资源管理库”包旁边的“安装”，然后根据说明安装该包。

现在，你可以开始使用这些库来创建应用程序了。

##步骤 4：创建用于对请求进行身份验证的凭据

安装 Azure Active Directory 应用程序并安装身份验证库后，你可以将应用程序信息格式化为凭据，以用于对发往 Azure 资源管理器的请求进行身份验证。请执行以下操作：

1.	打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.Resources;
        using Microsoft.Azure.Management.Resources.Models;

2.	将以下方法添加到 Program 类，以获取创建凭据所需的令牌：

        private static string GetAuthorizationHeader()
        {
          ClientCredential cc = new ClientCredential("{application-id}", "{password}");
            var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
            var result = context.AcquireToken("https://management.windowsazure.cn/", cc);
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

4.	保存 Program.cs 文件。


##步骤 5：添加代码以部署模板

资源始终从模板部署到资源组。可以使用 [ResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.models.resourcegroup.aspx) 和 [ResourceManagementClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.management.resources.resourcemanagementclient.aspx) 类来创建要将资源部署到的资源组。

1.	将以下方法添加到 Program 类，以创建资源组：

		public async static void CreateResourceGroup(TokenCloudCredentials credential)
		{
		  Console.WriteLine("Creating the resource group...");
		  var resourceGroup = new ResourceGroup { Location = "China North" };
		  using (var resourceManagementClient = new ResourceManagementClient(credential))
		  {
		    var rgResult = await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync("mytestrg1", resourceGroup);
		    Console.WriteLine(rgResult.StatusCode);
		  }
		}

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateResourceGroup(credential);
		Console.ReadLine();

3.	将以下方法添加到 Program 类，以使用你定义的模板将资源部署到资源组：

		public async static void CreateTemplateDeployment(TokenCloudCredentials credential)
		{
		  Console.WriteLine("Creating the template deployment...");
		  var deployment = new Deployment();
          deployment.Properties = new DeploymentProperties
		  {
		    Mode = DeploymentMode.Incremental,
		    TemplateLink = new TemplateLink
		    {
		      Uri = new Uri("https://{storage-account-name}.blob.core.chinacloudapi.cn/templates/VirtualMachineTemplate.json")
			},
			ParametersLink = new ParametersLink
			{
			  Uri = new Uri("https://{storage-account-name}.blob.core.chinacloudapi.cn/templates/Parameters.json")
			}
		  };
		  using (var templateDeploymentClient = new ResourceManagementClient(credential))
		  {
		    var dpResult = await templateDeploymentClient.Deployments.CreateOrUpdateAsync("mytestrg1", "mytestdp1", deployment);
			Console.WriteLine(dpResult.StatusCode);
		  }
		}

	将 {storage-account-name} 替换为前面将文件所放置到的帐户名。

4.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		CreateTemplateDeployment(credential);
		Console.ReadLine();

##步骤 6：添加代码以删除资源

由于你需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。你不需要单独从资源组中删除每个资源，而是可以删除资源组，这样就会自动删除它包含的所有资源。

1.	将以下方法添加到 Program 类，以删除资源组：

		public async static void DeleteResourceGroup(TokenCloudCredentials credential)
		{
		  using (var resourceGroupClient = new ResourceManagementClient(credential))
		  {
		    var rgResult = await resourceGroupClient.ResourceGroups.DeleteAsync("mytestrg1");
			Console.WriteLine(rgResult.StatusCode);
		  }
		}

2.	将以下代码添加到 Main 方法，以调用你刚刚添加的方法：

		DeleteResourceGroup(credential);
		Console.ReadLine();

##步骤 7：运行控制台应用程序

1.	若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同用户名和密码登录到 Azure AD。

2.	在返回每个状态代码后，按 **Enter** 创建每个资源。创建虚拟机后，执行下一步骤，然后按 Enter 删除所有资源。

	控制台应用程序从头到尾完成运行大约需要 5 分钟时间。在按 Enter 开始删除资源之前，你可能需要在 Azure 预览版门户中花费几分钟时间来验证资源的创建。

3. 在 Azure 预览版门户浏览到“审核日志”，以查看资源的状态：

	![创建 AD 应用程序](./media/arm-template-deployment/crpportal.png)

<!---HONumber=69-->