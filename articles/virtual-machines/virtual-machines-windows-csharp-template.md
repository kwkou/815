<properties
    pageTitle="使用 C# 和 Resource Manager 模板部署 VM | Azure"
    description="了解如何使用 C# 和 Resource Manager 模板部署 Azure VM。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="bfba66e8-c923-4df2-900a-0c2643b81240"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/01/2017"
    wacn.date="04/17/2017"
    ms.author="davidmu"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="1a5a4a2b0addd26dab5da2ae6892cb0119b02bca"
    ms.lasthandoff="04/06/2017" />

# <a name="deploy-an-azure-virtual-machine-using-c-and-a-resource-manager-template"></a>使用 C# 和 Resource Manager 模板部署 Azure 虚拟机
本文介绍如何使用 C# 部署 Azure Resource Manager 模板。 此[模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json)在包含单个子网的新虚拟网络中部署运行 Windows Server 的单个虚拟机。

有关虚拟机资源的详细说明，请参阅 [Azure Resource Manager 模板中的虚拟机](/documentation/articles/virtual-machines-windows-template-description/)。<!-- resource-manager-template-walkthrough oboslete -->

完成这些步骤大约需要 10 分钟。

## <a name="step-1-create-a-visual-studio-project"></a>步骤 1：创建 Visual Studio 项目

在此步骤中，请确保已安装 Visual Studio 并已创建用于部署模板的控制台应用程序。

1. 如果尚未安装，请安装 [Visual Studio](https://www.visualstudio.com/)。
2. 在 Visual Studio 中，单击“文件” > “新建” > “项目”。
3. 在“模板” > “Visual C#”中，选择“控制台应用程序”，输入项目的名称和位置，然后单击“确定”。

## <a name="step-2-install-libraries"></a>步骤 2：安装库

使用 NuGet 包可以最轻松地安装完成这些步骤所需的库。 需要 Azure Resource Manager 库和 Active Directory 身份验证库以创建资源。 若要在 Visual Studio 中获取这些库，请执行以下步骤：

1. 在解决方案资源管理器中右键单击项目名称，然后依次单击“管理 NuGet 包”和“浏览”。
2. 在搜索框中键入 *Microsoft.IdentityModel.Clients.ActiveDirectory*，单击“安装”，然后按照说明安装该包。
3. 在页面顶部，选择“包括预发行版”。 在搜索框中键入 *Microsoft.Azure.Management.ResourceManager*，单击“安装”，然后按照说明安装该包。

现在，你可以开始使用这些库来创建应用程序了。

## <a name="step-3-create-credentials-used-to-authenticate-requests"></a>步骤 3：创建用于对请求进行身份验证的凭据

在开始此步骤之前，请确保能够访问 [Active Directory 服务主体](/documentation/articles/resource-group-authenticate-service-principal/)。 从服务主体中，将获取对 Azure Resource Manager 请求进行身份验证的令牌。

1. 打开你为项目创建的 Program.cs 文件，然后在该文件的顶部添加以下 using 语句：

        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.ResourceManager;
        using Microsoft.Azure.Management.ResourceManager.Models;
        using Microsoft.Rest;
        using System.IO;

2. 将以下方法添加到 Program 类，以获取创建凭据所需的令牌：

        private static async Task<AuthenticationResult> GetAccessTokenAsync()
        {
          var cc = new ClientCredential("{client-id}", "{client-secret}");
          var context = new AuthenticationContext("https://login.chinacloudapi.cn/{tenant-id}");
          var token = await context.AcquireTokenAsync("https://management.chinacloudapi.cn/", cc);
          if (token == null)
          {
            throw new InvalidOperationException("Could not get the token.");
          } 
          return token;
        }

    替换以下值：

    - 将 *{client-id}* 替换为 Azure Active Directory 应用程序的标识符。 在 AD 应用程序的“属性”边栏选项卡上，可找到此标识符。 若要在 Azure 门户预览中查找 AD 应用程序，请在资源菜单中单击“Azure Active Directory”，然后单击“应用注册”。
    - 将 *{client-secret}* 替换为 AD 应用程序的访问密钥。 在 AD 应用程序的“属性”边栏选项卡上，可找到此标识符。
    - 将 *{tenant-id}* 替换为订阅的租户标识符。 在 Azure 门户预览中 Azure Active Directory 的“属性”边栏选项卡上，可找到租户标识符。 它被标记为目录 ID。

3. 若要调用刚刚添加的方法，请将以下代码添加到 Main 方法：

        var token = GetAccessTokenAsync();
        var credential = new TokenCredentials(token.Result.AccessToken);

4. 保存 Program.cs 文件。

## <a name="step-4-create-a-resource-group"></a>步骤 4：创建资源组

虽然可以从模板创建资源组，但是未使用库中的模板创建资源组。 在此步骤中，添加代码来创建资源组。

1. 若要指定应用程序的值，请将变量添加到 Program 类的 Main 方法中：

        var groupName = "myResourceGroup";
        var subscriptionId = "subsciptionId";
        var deploymentName = "deploymentName;
        var location = "location";

    替换以下值：

    - 将 *myResourceGroup* 替换为要创建的资源组的名称。
    - 将 *subscriptionId* 替换为订阅标识符。 在 Azure 门户预览的“订阅”边栏选项卡上，可找到订阅标识符。
    - 将 *deploymentName* 替换为部署的名称。
    - 将 *location* 替换为想要在其中创建资源的 Azure 区域。

2. 将以下方法添加到 Program 类，以创建资源组：

        public static async Task<ResourceGroup> CreateResourceGroupAsync(
          TokenCredentials credential,
          string groupName,
          string subscriptionId,
          string location)
        {
          var resourceManagementClient = new ResourceManagementClient(credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Creating the resource group...");
          var resourceGroup = new ResourceGroup { Location = location };
          return await resourceManagementClient.ResourceGroups.CreateOrUpdateAsync(
            groupName, 
            resourceGroup);
        }

3. 若要调用刚刚添加的方法，请将以下代码添加到 Main 方法：

        var rgResult = CreateResourceGroupAsync(
          credential,
          groupName,
          subscriptionId,
          location);
        Console.WriteLine(rgResult.Result.Properties.ProvisioningState);
        Console.ReadLine();

## <a name="step-5-create-a-parameters-file"></a>步骤 5：创建参数文件

若要为模板中定义的资源参数指定值，请创建包含值的参数文件。 部署模板时使用参数文件。 从库中使用的模板需要 adminUserName、adminPassword 和dnsLabelPrefix 参数的值。

在 Visual Studio 中执行以下步骤：

1. 在解决方案资源管理器中右键单击项目名称，然后单击“添加” > “新建项”。
2. 单击“Web”，选择“JSON 文件”，输入名称 Parameters.json，然后单击“添加”。
3. 打开 Parameters.json 文件，然后添加以下 JSON 内容：

        {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminUserName": { "value": "mytestacct1" },
            "adminPassword": { "value": "mytestpass1" },
            "dnsLabelPrefix": { "value": "mydns1" }
          }
        }

    将参数值替换为适合你的环境的值。

4. 保存 Parameters.json 文件。

## <a name="step-6-deploy-a-template"></a>步骤 6：部署模板

在本示例中，从 Azure 模板库中部署模板，并使用创建的本地文件向此模板提供参数值。 

1. 若要部署模板，请将以下方法添加到 Program 类：

        public static async Task<DeploymentExtended> CreateTemplateDeploymentAsync(
          TokenCredentials credential,
          string groupName,
          string deploymentName,
          string subscriptionId)
        {

          var resourceManagementClient = new ResourceManagementClient(new Uri("https://management.chinacloudapi.cn/"), credential)
            { SubscriptionId = subscriptionId };

          Console.WriteLine("Creating the template deployment...");
          var deployment = new Deployment();
          deployment.Properties = new DeploymentProperties
            {
              Mode = DeploymentMode.Incremental,
              TemplateLink = new TemplateLink("https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-simple-windows/azuredeploy.json"),
              Parameters = File.ReadAllText("..\\..\\Parameters.json")
            };

          return await resourceManagementClient.Deployments.CreateOrUpdateAsync(
            groupName,
            deploymentName,
            deployment
          );
        }

    你还可以使用 Template 属性，而不是 TemplateLink 属性从本地文件夹中部署模板。

2. 若要调用刚刚添加的方法，请将以下代码添加到 Main 方法：

        var dpResult = CreateTemplateDeploymentAsync(
          credential,
          groupName,
          deploymentName,
          subscriptionId);
        Console.WriteLine(dpResult.Result.Properties.ProvisioningState);
        Console.ReadLine();

## <a name="step-7-delete-the-resources"></a>步骤 7：删除资源

由于需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。 你不需要从资源组中分别删除每个资源， 删除资源组就会自动删除其所有资源。

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

2. 若要调用刚刚添加的方法，请将以下代码添加到 Main 方法：

        DeleteResourceGroupAsync(
          credential,
          groupName,
          subscriptionId);
        Console.ReadLine();

## <a name="step-8-run-the-console-application"></a>步骤 8：运行控制台应用程序

控制台应用程序从头到尾完成运行大约需要五分钟时间。 

1. 若要运行控制台应用程序，请在 Visual Studio 中单击“启动”，然后使用用于订阅的相同凭据登录到 Azure AD。

2. 显示“已成功”状态后按 **Enter**。 

    还可以在 Azure 门户预览中资源组的“概览”边栏选项卡上的“部署”下面看到“1 个成功”。

3. 在按 **Enter** 开始删除资源之前，可能需要在 Azure 门户预览中花几分钟时间来验证资源的创建。 单击部署状态以查看有关部署的信息。

## <a name="next-steps"></a>后续步骤
* 如果部署出现问题，后续措施是参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。
* 若要了解如何部署虚拟机及其支持的资源，请查看[使用 C# 部署 Azure 虚拟机](/documentation/articles/virtual-machines-windows-csharp/)。
* 查看[使用 Azure Resource Manager 和 C# 管理 Azure 虚拟机](/documentation/articles/virtual-machines-windows-csharp-manage/)，了解如何管理创建的虚拟机。
<!--Update_Description: wording update-->