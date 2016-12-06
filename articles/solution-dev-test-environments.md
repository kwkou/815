<properties
   pageTitle="开发和测试环境 | Azure"
   description="了解如何使用 Azure 资源管理器模板快速一致性地创建和删除开发与测试环境。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>  

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/22/2016"
   wacn.date="11/28/2016"
   ms.author="tomfitz"/>


# Azure 中的开发和测试环境

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

自定义应用程序在部署到生产环境之前，通常会先部署到多个开发和测试环境。在本地创建环境时，需要为每个应用程序的每个环境采购或分配计算资源。环境通常包含多个物理或虚拟机，这些虚拟机采用手动部署的特定配置或包含复杂的自动化脚本。部署通常需要数小时，并且会在环境中导致不一致的配置。

## 方案 ##

当你在 Azure 中预配开发和测试环境时，只需支付资源的使用费。本文说明如何使用 Azure 资源管理器模板和参数文件按如下所示快速一致地创建、维护和删除开发及测试环境。

![方案](./media/solution-dev-test-environments/scenario.png)

上面显示了三个开发和测试环境。每个环境具有模板文件中指定的 Web 应用程序和 SQL 数据库资源。每个环境中的应用程序和数据库名称各不相同，且在每个环境中唯一的参数文件中指定。

如果你不熟悉 Azure 资源管理器的概念，建议在阅读本文之前先阅读 [Azure 资源管理器概述](/documentation/articles/resource-group-overview/)一文。

你也可以先完成本文中所列的步骤而不阅读任何参考文章，以快速体验 Azure 资源管理器模板的使用。完成这些步骤后，可以利用这些步骤体验并阅读参考文章，以获得首次使用时遇到的大多数问题的解答。

## 规划 Azure 资源的使用
在拥有应用程序高级设计经验后，你可以定义：

- 应用程序所要包含的 Azure 资源。你可以构建应用程序并将它部署为包含 Azure SQL 数据库的 Azure Web 应用。可以在虚拟机中使用 PHP 和 MySQL 或 IIS 和 SQL Server 或其他组件构建应用程序。[Azure App Service、云服务与虚拟机的比较](/documentation/articles/choose-web-site-cloud-service-vm/)一文可帮助你确定想要用于应用程序的 Azure 资源。
- 应用程序应符合哪些服务级别要求，例如可用性、安全性和缩放性。

## 下载现有模板
Azure 资源管理器模板定义应用程序使用的所有 Azure 资源。你可以直接在 Azure 门户预览中部署多个现有的模板，或者使用应用程序代码在源代码管理系统中下载、修改和保存这些模板。完成以下步骤以下载现有的模板。

1. 可以在 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/) GitHub 存储库中浏览现有模板。在列表中，你将看到“[201-web-app-sql-database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)”文件夹。由于许多自定义应用程序包含 Web 应用程序和 SQL 数据库，此模板可作为本文其余部分的示例，帮助你了解如何使用模板。 <!-- 本文未提供有关创建和配置此模板的完整内容，但是，如果你打算使用它在组织中创建实际环境，请阅读[预配包含 SQL 数据库的 Web 应用](/documentation/articles/app-service-web-arm-with-sql-database-provision/)一文以全面了解该模板。 -->
2. 单击 201-web-app-sql-databas 文件夹中的 [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-sql-database/azuredeploy.json) 文件以查看其内容。这是 Azure 资源管理器模板文件。 
3. 在视图模式中，单击“原始”按钮。[](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/3f24f7b7e1e377538d1d548eaa6eab2851a21810/201-web-app-sql-database/azuredeploy.json)
4. 使用鼠标选择此文件的内容，然后将内容以“TestApp1-Template.json”文件名保存到计算机。
5. 检查模板的内容，并注意以下事项：
 - **Resources** 部分：此部分定义此模板创建的 Azure 资源的类型。在其他资源类型中，此模板将创建 [Azure Web 应用](/documentation/articles/app-service-web-overview/)和 [Azure SQL 数据库](/documentation/articles/sql-database-technical-overview/)资源。如果偏好在虚拟机中运行和管理 Web 与 SQL 服务器，可以使用“[iis-2vm-sql-1vm](https://github.com/Azure/azure-quickstart-templates/tree/master/iis-2vm-sql-1vm)”或“[lamp-app](https://github.com/Azure/azure-quickstart-templates/tree/master/lamp-app)”模板；但本文中的说明基于 [201-web-app-sql-database](https://github.com/Azure/azure-quickstart-templates/tree/3f24f7b7e1e377538d1d548eaa6eab2851a21810/201-web-app-sql-database) 模板。
 - **Parameters** 部分：此部分定义可用于配置每个资源的参数。在模板中指定的一些参数带有“defaultValue”属性，而其他一些参数则没有该属性。使用模板部署 Azure 资源时，必须将值提供给模板中所有未指定 defaultValue 属性的参数。如果未向带有 defaultValue 属性的参数提供值，则会使用模板中为 defaultValue 参数指定的值。

模板定义所创建的 Azure 资源，以及可用来配置每个资源的参数。

## 下载和自定义现有参数文件

尽管可能希望在每个环境中创建*相同的* Azure 资源，但也可能希望在每个环境中设置*不同的*资源配置。这就是参数文件的作用。完成以下步骤，在每个环境中创建包含唯一值的参数文件。

1. 查看 201-web-app-sql-database 文件夹中 [azuredeploy.parameters.json](https://github.com/Azure/azure-quickstart-templates/tree/3f24f7b7e1e377538d1d548eaa6eab2851a21810/201-web-app-sql-database/azuredeploy.parameters.json) 文件的内容。这是在前一部分中保存的模板文件的参数文件。
2. 在视图模式中，单击“原始”按钮。[](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/3f24f7b7e1e377538d1d548eaa6eab2851a21810/201-web-app-sql-database/azuredeploy.parameters.json)
3. 使用鼠标选择此文件的内容，然后使用以下名称将其保存在计算机上的三个不同文件中：
 - TestApp1-Parameters-Development.json
 - TestApp1-Parameters-Test.json
 - TestApp1-Parameters-Pre-Production.json

3. 使用任何文本或 JSON 编辑器来编辑步骤 3 中创建的开发环境参数文件，并将列在文件中**参数**右侧的值替换为列在以下参数右侧的值： 
 - **siteName**：TestApp1DevApp
 - **hostingPlanName**：TestApp1DevPlan
 - **siteLocation**：China East
 - **serverName**：testapp1devsrv
 - **serverLocation**：China East
 - **administratorLogin**：testapp1Admin
 - **administratorLoginPassword**：替换为你的密码
 - **databaseName**：testapp1devdb

4. 使用任何文本或 JSON 编辑器来编辑步骤 3 中创建的测试环境参数文件，并将列在文件中**参数**右侧的值替换为列在以下参数右侧的值：
 - **siteName**：TestApp1TestApp
 - **hostingPlanName**：TestApp1TestPlan
 - **siteLocation**：China East
 - **serverName**：testapp1testsrv
 - **serverLocation**：China East
 - **administratorLogin**：testapp1Admin
 - **administratorLoginPassword**：替换为你的密码
 - **databaseName**：testapp1testdb

5. 使用任何文本或 JSON 编辑器，编辑在步骤 3 中创建的预生产参数文件。将文件的整个内容替换为以下内容：

	    {
    	  "$schema" : "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    	  "contentVersion" : "1.0.0.0",
    	  "parameters" : {
    	"administratorLogin" : {
    	  "value" : "testApp1Admin"
    	},
    	"administratorLoginPassword" : {
    	  "value" : "replace with your password"
    	},
    	"databaseName" : {
    	  "value" : "testapp1preproddb"
    	},
    	"hostingPlanName" : {
    	  "value" : "TestApp1PreProdPlan"
    	},
    	"serverLocation" : {
    	  "value" : "China East"
    	},
    	"serverName" : {
    	  "value" : "testapp1preprodsrv"
    	},
    	"siteLocation" : {
    	  "value" : "China East"
    	},
    	"siteName" : {
    	  "value" : "TestApp1PreProdApp"
    	},
    	"sku" : {
    	  "value" : "Standard"
    	},
    		"requestedServiceObjectiveName" : {
    		  "value" : "S1"
    	}
    	  }
    	}

在上述预生产参数文件中，**sku** 和 **requestedServiceObjectiveName** 参数为 *added*，而它们并未添加到开发和测试参数文件。这是因为模板中为这些参数指定了默认值，并且默认值也用于开发和测试环境中，但在预生产环境中用于这些参数的是非默认值。

在预生产环境中将非默认值用于这些参数的原因是要测试生产环境可能偏好的这些参数的值，以便也可以测试这些参数。这些参数全都与应用程序使用的 Azure Web 应用托管计划、**sku** 和 [Azure SQL 数据库](/pricing/details/sql-database/)或 **requestedServiceObjectiveName** 相关。不同的 sku 和服务目标名称有不同的成本和功能，并支持不同的服务级别度量量。

下表列出了模板中指定参数的默认值，以及在预生产参数文件中替代默认值的值。

| 参数 | 默认值 | 参数文件值 |
|---|---|---|
| **sku** | 免费 | 标准 |
| **requestedServiceObjectiveName** | S0 | S1 |

## 创建环境
所有 Azure 资源必须在 [Azure 资源组](/documentation/articles/resource-group-portal#create-resource-group-and-resources)中创建。资源组可让你将 Azure 资源分组，以便可以统一管理这些资源。[权限](/documentation/articles/role-based-access-built-in-roles/)可以分配给资源组，使组织中的特定人员可以创建、修改、删除或查看这些组及其包含的资源。可以在 [Azure 门户预览](https://portal.azure.cn)中查看资源组中资源的警报和计费信息。资源组在 Azure 区域中创建。

使用以下方法之一为每个环境创建资源组。所有方法都可实现相同的结果。

###Azure 命令行界面 (CLI)

确保 Windows、OS X 或 Linux 计算机上[已安装](/documentation/articles/xplat-cli-install/) CLI，并将 [Azure AD 帐户](/documentation/articles/active-directory-how-subscriptions-associated-directory/)（也称为工作帐户或学校帐户）[连接到](/documentation/articles/xplat-cli-connect/) Azure 订阅。在 CLI 命令行中键入以下命令以创建开发环境的资源组。

	azure group create "TestApp1-Development" "China East"

如果命令成功，将返回以下信息：

	info:    Executing command group create
	+ Getting resource group TestApp1-Development
	+ Creating resource group TestApp1-Development
	info:    Created resource group TestApp1-Development
	data:    Id:                  /subscriptions/uuuuuuuu-vvvv-wwww-xxxx-yyyy-zzzzzzzzzzzz/resourceGroups/TestApp1-Development
	data:    Name:                TestApp1-Development
	data:    Location:            chinaeast
	data:    Provisioning State:  Succeeded
	data:    Tags: null
	data:
	info:    group create command OK

若要创建测试环境的资源组，请键入以下命令：

	azure group create "TestApp1-Test" "China East"

若要创建预生产环境的资源组，请键入以下命令：

	azure group create "TestApp1-Pre-Production" "China East"

###PowerShell

确保已按 [How to install and configure Azure PowerShell（如何安装和配置 Azure PowerShell）](/documentation/articles/powershell-install-configure/) 一文中的详述，在 Windows 计算机上安装了 PowerShell 1.01 或更高版本，并将你的 [Azure AD 帐户](/documentation/articles/active-directory-how-subscriptions-associated-directory/)（也称为工作帐户或学校帐户）连接到你的订阅。在 PowerShell 命令提示符下，键入以下命令以创建开发环境的资源组。

	New-AzureRmResourceGroup -Name TestApp1-Development -Location "China East"

如果命令成功，将返回以下信息：

	ResourceGroupName : TestApp1-Development
	Location          : chinaeast
	ProvisioningState : Succeeded
	Tags              :
	ResourceId        : /subscriptions/uuuuuuuu-vvvv-wwww-xxxx-yyyy-zzzzzzzzzzzz/resourceGroups/TestApp1-Development

若要创建测试环境的资源组，请键入以下命令：

	New-AzureRmResourceGroup -Name TestApp1-Test -Location "China East"

若要创建预生产环境的资源组，请键入以下命令：

	New-AzureRmResourceGroup -Name TestApp1-Pre-Production -Location "China East"

###Azure 门户预览

1. 使用 [Azure AD](/documentation/articles/active-directory-how-subscriptions-associated-directory/)（也称为工作或学校）帐户登录到 [Azure 门户预览](https://portal.azure.cn)。单击“新建”-->“管理”-->“资源组”，在“资源组名称”框中输入“TestApp1-Development”、选择你的订阅，然后在“资源组位置”框中选择“美国中部”，如下图所示。
   ![门户](./media/solution-dev-test-environments/rgcreate.png)
2. 单击“创建”按钮以创建资源组。
3. 单击“浏览”并在列表中向下滚动到“资源组”，然后单击“资源组”，如下所示。
   ![门户](./media/solution-dev-test-environments/rgbrowse.png)
4. 单击“资源组”后，会看到新资源组的“资源组”边栏选项卡。
   ![门户](./media/solution-dev-test-environments/rgview.png)
5. 使用前面创建 TestApp1-Development 资源组的相同方式创建 TestApp1-Test 和 TestApp1-Pre-Production 资源组。

<a name="deploy-resources-to-environments"></a>
##将资源部署到环境

使用解决方案的模板文件，将 Azure 资源部署到每个环境的资源组，并使用下列任一方法将其部署到每个环境的参数文件。两种方法都可实现相同的结果。

###Azure 命令行界面 (CLI)

在 CLI 命令行中键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为开发环境创建的资源组。

	azure group deployment create -g TestApp1-Development -n Deployment1 -f [path]TestApp1-Template.json -e [path]TestApp1-Parameters-Development.json 

在看到“等待部署完成”消息数分钟之后，如果部署成功，命令将返回以下消息：

	info:    Executing command group deployment create
	+ Initializing template configurations and parameters
	+ Creating a deployment
	info:    Created template deployment "Deployment1"
	+ Waiting for deployment to complete
	data:    DeploymentName     : Deployment1
	data:    ResourceGroupName  : TestApp1-Development
	data:    ProvisioningState  : Succeeded
	data:    Timestamp          : XXXX-XX-XXT20:20:23.5202316Z
	data:    Mode               : Incremental
	data:    Name                           Type          Value
	data:    -----------------------------  ------------  ----------------------------
	data:    siteName                       String        TestApp1DevApp
	data:    hostingPlanName                String        TestApp1DevPlan
	data:    siteLocation                   String        China East
	data:    sku                            String        Free
	data:    workerSize                     String        0
	data:    serverName                     String        testapp1devsrv
	data:    serverLocation                 String        China East
	data:    administratorLogin             String        testapp1Admin
	data:    administratorLoginPassword     SecureString  undefined
	data:    databaseName                   String        testapp1devdb
	data:    collation                      String        SQL_Latin1_General_CP1_CI_AS
	data:    edition                        String        Standard
	data:    maxSizeBytes                   String        1073741824
	data:    requestedServiceObjectiveName  String        S0
	info:    group deployment create command OKx

如果命令未成功，请解决任何错误消息并重试。常见的问题是使用了未遵循 Azure 资源命名约束的参数值。

在 CLI 命令行中键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为测试环境创建的资源组。

	azure group deployment create -g TestApp1-Test -n Deployment1 -f [path]TestApp1-Template.json -e [path]TestApp1-Parameters-Test.json

在 CLI 命令行中键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为预生产环境创建的资源组。

	azure group deployment create -g TestApp1-Pre-Production -n Deployment1 -f [path]TestApp1-Template.json -e [path]TestApp1-Parameters-Pre-Production.json
  
###PowerShell

在 Azure PowerShell（版本 1.01 或更高）命令提示符下键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为开发环境创建的资源组。

	New-AzureRmResourceGroupDeployment -ResourceGroupName TestApp1-Development -TemplateFile [path]TestApp1-Template.json -TemplateParameterFile [path]TestApp1-Parameters-Development.json -Name Deployment1 

光标闪烁数分钟之后，如果部署成功，命令将返回以下消息：

	DeploymentName    : Deployment1
	ResourceGroupName : TestApp1-Development
	ProvisioningState : Succeeded
	Timestamp         : XX/XX/XXXX 2:44:48 PM
	Mode              : Incremental
	TemplateLink      : 
	Parameters        : 
	                    Name             Type                       Value     
	                    ===============  =========================  ==========
	                    siteName         String                     TestApp1DevApp
	                    hostingPlanName  String                     TestApp1DevPlan
	                    siteLocation     String                     China East
	                    sku              String                     Free      
	                    workerSize       String                     0         
	                    serverName       String                     testapp1devsrv
	                    serverLocation   String                     China East
	                    administratorLogin  String                     testapp1Admin
	                    administratorLoginPassword  SecureString                         
	                    databaseName     String                     testapp1devdb
	                    collation        String                     SQL_Latin1_General_CP1_CI_AS
	                    edition          String                     Standard  
	                    maxSizeBytes     String                     1073741824
	                    requestedServiceObjectiveName  String                     S0        
	                    
	Outputs           :

  如果命令未成功，请解决任何错误消息并重试。常见的问题是使用了未遵循 Azure 资源命名约束的参数值。其他故障排除提示可在 [Azure 中的资源组部署故障排除](/documentation/articles/resource-group-deploy-debug/)一文中找到。

  在 PowerShell 命令提示符下键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为测试环境创建的资源组。

	New-AzureRmResourceGroupDeployment -ResourceGroupName TestApp1-Test -TemplateFile [path]TestApp1-Template.json -TemplateParameterFile [path]TestApp1-Parameters-Test.json -Name Deployment1

  在 PowerShell 命令提示符下键入以下命令（将 [path] 替换为前面步骤中的文件保存路径），将资源部署到为预生产环境创建的资源组。

	New-AzureRmResourceGroupDeployment -ResourceGroupName TestApp1-Pre-Production -TemplateFile [path]TestApp1-Template.json -TemplateParameterFile [path]TestApp1-Parameters-Pre-Production.json -Name Deployment1

在源代码管理系统中可以使用应用程序代码控制模板和参数文件的版本和进行维护。还可以将上述命令保存到脚本文件，以及将它们与代码一起保存。


## 维护环境
在整个开发过程中，不同环境中的 Azure 资源配置可能会出现有意或无意的不一致性更改。这可能会在应用程序开发周期中造成不必要的故障排除和问题解决。

1. 打开 [Azure 门户预览](https://portal.azure.cn)以更改环境。 
2. 使用完成上述步骤时所用的同一帐户登录。 
3. 如下图所示，单击“浏览”-->“资源组”（可能需要向下滚动才能看到资源组）。
   ![门户](./media/solution-dev-test-environments/rgbrowse.png)
4. 单击上图中所示的资源组之后，你将看到“资源组”边栏选项卡，以及在上一步骤中创建的三个资源组，如下图所示。单击 TestApp1-Development 资源组之后，你将看到一个边栏选项卡，其中列出了模板在上一步骤完成的 TestApp1-Development 资源组部署中所创建的资源。单击“TestApp1-Development 资源组”边栏选项卡中的“TestApp1DevApp”，删除 TestApp1DevApp Web 应用资源，然后单击“TestApp1DevApp Web 应用”边栏选项卡中的“删除”。
   ![门户](./media/solution-dev-test-environments/portal2.png)
5. 当门户提示是否确定要删除该资源时，请单击“是”。如果关闭“TestApp1-Development 资源组”边栏选项卡并将它重新打开，显示的内容不会出现刚刚删除的 Web 应用。资源组的内容现在与其应有内容不同。你可以从多个资源组删除多个资源来进一步试验，甚至可以更改某些资源的配置设置。如果不使用 Azure 门户预览从资源组删除资源，你可以使用 PowerShell [Remove-AzureResource](https://msdn.microsoft.com/zh-cn/library/azure/dn757676.aspx) 命令或者从 CLI 使用“azure resource delete”命令来完成相同的任务。
6. 若要让所有应该位于资源组中的所有资源和配置恢复到其应有状态，请使用[将资源部署到环境](#deploy-resources-to-environments)部分中的相同命令，将环境重新部署到资源组，但需要将“Deployment1”替换为“Deployment2”。
7.  如步骤 4 中图示的 TestApp1-Development 边栏选项卡中的“摘要”部分，你将看到在上一步骤中删除的 Web 应用以及其他删除的资源再次出现。如果你更改了任何资源的配置，则还可以验证这些配置是否已设置回到参数文件中的值。使用 Azure 资源管理器模板部署环境的优点之一是可以随时轻松地将环境重新部署回到已知状态。
8. 如果你单击下图中“上次部署”下面的文本，将会看到边栏选项卡显示资源组的部署历史记录。由于你已将名称“Deployment1”用于第一个部署，并已将“Deployment2”用于第二个部署，因此会有两个条目。单击某个部署时会显示一个边栏选项卡，其中显示每个部署的结果。
   ![门户](./media/solution-dev-test-environments/portal3.png)



## 删除环境
使用完某个环境后，你可以将它删除，以免支付不再使用的 Azure 资源的使用费。删除环境比创建环境还要容易。在上一步骤中，你已为每个环境分别创建了 Azure 资源组，然后将资源部署到了资源组中。

使用下列任一方法删除环境。所有方法都可实现相同的结果。

### Azure CLI

在 CLI 提示符下键入以下命令：

	azure group delete "TestApp1-Development"

出现提示时，请输入 y 并按 Enter 以删除开发环境及其所有资源。几分钟后，命令将返回以下内容：

	info:    group delete command OK

在 CLI 提示符下键入以下命令，以删除剩余的环境：

	azure group delete "TestApp1-Test"
	azure group delete "TestApp1-Pre-Production"
  
### PowerShell

在 Azure PowerShell（版本 1.01 或更高）命令提示符下，键入以下命令以删除资源组及其所有内容。

	Remove-AzureRmResourceGroup -Name TestApp1-Development

当系统提示你是否确定要删除该资源组时，请输入 y，然后按 Enter 键。

键入以下命令以删除剩余的环境：

	Remove-AzureRmResourceGroup -Name TestApp1-Test
	Remove-AzureRmResourceGroup -Name TestApp1-Pre-Production

### Azure 门户预览

1. 在 Azure 门户预览中浏览到“资源组”，如同上一步骤所述。 
2. 选择“TestApp1-Development 资源组”，然后单击“TestApp1-Development 资源组”边栏选项卡中的“删除”。随即显示新的边栏选项卡。输入资源组名称，然后单击“删除”按钮。
![门户](./media/solution-dev-test-environments/rgdelete.png)
3. 使用删除 TestApp1-Development 资源组的相同方式删除 TestApp1-Test 和 TestApp1-Pre-Production 资源组。

不管使用哪种方法，资源组及其包含的所有资源都将不再存在，但你不再需要支付资源的使用费。

若要在应用程序开发过程中最大程度地减少 Azure 资源的使用费，你可以使用 [Azure 自动化](/documentation/articles/automation-intro/)来计划作业，以便：

- 在每天结束时停止虚拟机并在每天开始时重新启动它们。
- 在每天结束时删除整个环境并在每天开始时重新创建它们。
 
现在，你已了解如何轻松创建、维护和删除开发与测试环境，接下来可以通过进一步执行上述步骤并阅读本文中包含的参考内容，来深入了解刚刚学习的知识。

## 后续步骤

- 通过将 Azure AD 组或用户分配到能够对 Azure 资源执行部分操作的特定角色，向每个环境中的不同资源[委派管理控制](/documentation/articles/role-based-access-control-configure/)。
- 向每个环境的资源组和/或单个资源[分配标记](/documentation/articles/resource-group-using-tags/)。你可以将“Environment”标记添加到资源组，并将其值设置为与环境名称相对应。当您需要组织资源以进行计费或管理时，标记会特别有用。
- 在 [Azure 门户预览](https://portal.azure.cn)中监视资源组中资源的警报和计费。

<!---HONumber=Mooncake_1121_2016-->