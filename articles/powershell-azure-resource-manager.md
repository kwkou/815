<properties 
	pageTitle="将 Azure PowerShell 与资源管理器配合使用 | Azure" 
	description="介绍如何使用 Azure PowerShell 将作为资源组的多个资源部署到 Azure。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="10/16/2015" 
	wacn.date="01/21/2016"/>

# 将 Azure PowerShell 与 Azure 资源管理器配合使用

> [AZURE.SELECTOR]
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager)

Azure 资源管理器引入了一种考虑您的 Azure 资源的全新方法。首先应该构想整个解决方案，而不是创建和管理各个资源，如博客、照片库、SharePoint 门户或 wiki。可以使用模板（解决方案的声明性表示形式）创建包含支持该解决方案所需资源的资源组。然后，可以将该资源组作为一个逻辑单元进行管理和部署。

在本教程中，你将了解如何将 Azure PowerShell 与 Azure 资源管理器配合使用。本教程将指导您逐步完成通过 SQL 数据库创建和部署 Azure 托管的 Web 应用的资源组的过程，其中充分使用了支持该过程所需的所有资源。

## 先决条件

若要完成本教程，你需要：

- 一个 Azure 帐户
  + 可以[免费建立一个 Azure 帐户](/pricing/1rmb-trial/)：获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如 Web 应用）。你的信用卡将永远不会付费，除非你显式更改设置并要求付费。
  
- Azure PowerShell

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

本教程专为 PowerShell 新手设计，但它假定您了解基本概念，如模块、cmdlet 和会话。有关 Windows PowerShell 的详细信息，请参阅 [Windows PowerShell 入门](http://technet.microsoft.com/zh-cn/library/hh857337.aspx)。

## 将部署的内容

在本教程中，你将使用 Azure PowerShell 来部署 Web 应用和 SQL 数据库。但是，此 Web 应用和 SQL 数据库解决方案由多个共同协作的资源类型构成。将要部署的实际资源包括：

- SQL 服务器 - 用于托管数据库
- SQL 数据库 - 用于存储数据
- 防火墙规则 - 允许 Web 应用连接到数据库
- App Service 计划 - 用于定义 Web 应用的功能和成本
- Web 应用 - 用于运行 Web 应用
- Web 配置 - 用于存储数据库的连接字符串 

## 获取有关 cmdlet 的帮助

若要获得您在此教程中看到的任何 cmdlet 的详细帮助，请使用 Get-Help cmdlet。

	Get-Help <cmdlet-name> -Detailed

例如，若要获取有关 Get-AzureRmResource cmdlet 的帮助，请键入：

	Get-Help Get-AzureRmResource -Detailed

若要获取带有帮助摘要的资源模块中的 cmdlet 列表，请键入：

    PS C:\> Get-Command -Module AzureRM.Resources | Get-Help | Format-Table Name, Synopsis

输出与以下摘要类似：

	Name                                   Synopsis
	----                                   --------
	Find-AzureRmResource                   Searches for resources using the specified parameters.
	Find-AzureRmResourceGroup              Searches for resource group using the specified parameters.
	Get-AzureRmADGroup                     Filters active directory groups.
	Get-AzureRmADGroupMember               Get a group members.
	...

若要获得 cmdlet 的完整帮助，请使用以下格式键入命令：

	Get-Help <cmdlet-name> -Full
  
## 登录到你的 Azure 帐户

在处理解决方案之前，你必须登录到你的帐户。

若要登录到你的 Azure 帐户，请使用 **Login-AzureRmAccount** cmdlet。在低于 1.0 预览版的 Azure PowerShell 版本中，请使用 **Add-AzureAccount** 命令。

    PS C:\> Login-AzureRmAccount

该 cmdlet 将提示您提供您的 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。

帐户设置会过期，因此您需要不时刷新它们。若要刷新帐户设置，请再次运行 **Login-AzureRmAccount**。

>[AZURE.NOTE]资源管理器模块要求使用 Login-AzureRmAccount。一个发布设置文件是不够的。

## 获取资源类型的位置

在部署资源时，必须指定资源的托管位置。并非每个区域都支持每种资源类型。在部署 Web 应用和 SQL 数据库之前，必须确定哪些区域支持这些类型。资源组可以包含位于不同区域的资源；但是，只要可能，你就应该在同一位置创建资源，以优化性能。具体而言，请确保你的数据库位于应用所访问的同一位置。

若要获取支持每种资源类型的位置，需要使用 **Get-AzureRmResourceProvider** cmdlet。首先，让我们看看此命令的返回内容：

    PS C:\> Get-AzureRmResourceProvider -ListAvailable

    ProviderNamespace               RegistrationState ResourceTypes
    -----------------               ----------------- -------------
    Microsoft.ApiManagement         Unregistered      {service, validateServiceName, checkServiceNameAvailability}
    Microsoft.AppService            Registered        {apiapps, appIdentities, gateways, deploymenttemplates...}
    Microsoft.Batch                 Registered        {batchAccounts}
    ...

ProviderNamespace 表示相关资源类型的集合。这些命名空间通常与你要在 Azure 中创建的服务完全匹配。如果你想要使用已列作“已取消注册”的资源提供程序，可以通过运行 **Register-AzureRmResourceProvider** cmdlet 并指定要注册的提供程序命名空间，来注册该资源提供程序。要在本教程中使用的资源提供程序很可能已由你的订阅注册。

可以通过指定命名空间来获取有关提供程序的更多详细信息：

    PS C:\> Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Sql

    ProviderNamespace RegistrationState ResourceTypes                                 Locations
    ----------------- ----------------- -------------                                 ---------
    Microsoft.Sql     Registered        {operations}                                  {China North, China East
    Microsoft.Sql     Registered        {locations}                                   {China North, China East
    Microsoft.Sql     Registered        {locations/capabilities}                      {China North, China East
    ...

若要将输出限制为特定资源类型（例如 Web 应用）的支持位置，请使用：

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
    
输出结果将会类似于：

    China East
    China North

你看到的位置可能与上面的结果略有不同。结果存在差异的原因是你组织中的管理员创建的策略限制了可在订阅中使用的区域，或者可能存在与你所在国家/地区的税收政策相关的限制。

让我们对数据库运行同一命令：

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Sql).ResourceTypes | Where-Object ResourceTypeName -eq servers).Locations
    China East
    China North

看起来这些资源在许多区域可用。在本主题中，我们将使用“美国西部”，但你可以指定任何支持的区域。

## 创建资源组

本教程部分将指导你完成创建资源组的整个过程。资源组将充当解决方案中具有相同生命周期的所有资源的容器。在后面的教程部分，你要向此资源组部署 Web 应用和 SQL 数据库。

若要创建资源组，请使用 **New-AzureRmResourceGroup** cmdlet。

该命令使用 **Name** 参数来指定资源组的名称，并使用 **Location** 参数来指定其位置。根据上一部分中所述，我们将使用“美国西部”作为位置。

    PS C:\> New-AzureRmResourceGroup -Name TestRG1 -Location "China East"
    
    ResourceGroupName : TestRG1
    Location          : China East
    ProvisioningState : Succeeded
    Tags              :
    Permissions       :
                    Actions  NotActions
                    =======  ==========
                    *

    ResourceId        : /subscriptions/{guid}/resourceGroups/TestRG1

现已成功创建资源组。


## 获取资源的可用 API 版本

部署模板时，必须指定要用于创建资源的 API 版本。可用 API 版本对应于资源提供程序发布的 REST API 操作版本。资源提供程序启用新功能时，将会发布 REST API 的新版本。因此，在模板中指定的 API 版本会影响你在创建模板时可用的属性。通常，在创建新模板时，你需要选择最新的 API 版本。对于现有模板，你可以决定是要继续使用你已知不会更改部署的 API 版本，还是要选择最新版本来更新模板以利用新功能。

此步骤可能看起来令人困惑，但发现资源可用的 API 版本并不困难。你将再次使用 **Get-AzureRmResourceProvider** 命令。

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
    2015-08-01
    2015-07-01
    2015-06-01
    2015-05-01
    2015-04-01
    2015-02-01
    2014-11-01
    2014-06-01
    2014-04-01-preview
    2014-04-01

可以看到，此 API 经常会更新。通常，相同的 API 版本号适用于资源提供程序中的所有资源。唯一的例外情况是在某个时间点添加或删除了资源。我们假设相同的 API 版本适用于 serverFarms 资源；但是，你可以反复检查任何资源的可用 API 版本列表是否不同。

对于数据库，你将看到：

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Sql).ResourceTypes | Where-Object ResourceTypeName -eq servers/databases).ApiVersions
    2014-04-01-preview
    2014-04-01 

## 创建模板

本主题不会说明如何创建模板，也不会介绍模板的结构。有关此类信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。下面显示了你要部署的模板。请注意，该模板使用了你在上一部分中检索的 API 版本。为确保所有资源都驻留在同一区域中，我们将使用模板表达式 **resourceGroup ().location** 来使用该资源组位置。

另请注意参数部分。此部分定义部署资源时可以提供的值。本教程后面的部分将会到用到这些值。

你可以复制该模板，并将它作为.json 文件保存在本地。在本教程中，我们假设模板已保存到 c:\\Azure\\Templates\\azuredeploy.json，但你可以根据需要，使用好记的名称将它保存在任何方便访问的位置。

    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "hostingPlanName": {
                "type": "string"
            },
            "serverName": {
                "type": "string"
            },
            "databaseName": {
                "type": "string"
            },
            "administratorLogin": {
                "type": "string"
            },
            "administratorLoginPassword": {
                "type": "securestring"
            }
        },
        "variables": {
            "siteName": "[concat('ExampleSite', uniqueString(resourceGroup().id))]"
        },
        "resources": [
            {
                "name": "[parameters('serverName')]",
                "type": "Microsoft.Sql/servers",
                "location": "[resourceGroup().location]",
                "apiVersion": "2014-04-01",
                "properties": {
                    "administratorLogin": "[parameters('administratorLogin')]",
                    "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
                    "version": "12.0"
                },
                "resources": [
                    {
                        "name": "[parameters('databaseName')]",
                        "type": "databases",
                        "location": "[resourceGroup().location]",
                        "apiVersion": "2014-04-01",
                        "dependsOn": [
                            "[concat('Microsoft.Sql/servers/', parameters('serverName'))]"
                        ],
                        "properties": {
                            "edition": "Basic",
                            "collation": "SQL_Latin1_General_CP1_CI_AS",
                            "maxSizeBytes": "1073741824",
                            "requestedServiceObjectiveName": "Basic"
                        }
                    },
                    {
                        "name": "AllowAllWindowsAzureIps",
                        "type": "firewallrules",
                        "location": "[resourceGroup().location]",
                        "apiVersion": "2014-04-01",
                        "dependsOn": [
                            "[concat('Microsoft.Sql/servers/', parameters('serverName'))]"
                        ],
                        "properties": {
                            "endIpAddress": "0.0.0.0",
                            "startIpAddress": "0.0.0.0"
                        }
                    }
                ]
            },
            {
                "apiVersion": "2015-08-01",
                "type": "Microsoft.Web/serverfarms",
                "name": "[parameters('hostingPlanName')]",
                "location": "[resourceGroup().location]",
                "sku": {
                    "tier": "Free",
                    "name": "f1",
                    "capacity": 0
                },
                "properties": {
                    "numberOfWorkers": 1
                }
            },
            {
                "apiVersion": "2015-08-01",
                "name": "[variables('siteName')]",
                "type": "Microsoft.Web/sites",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[concat('Microsoft.Web/serverFarms/', parameters('hostingPlanName'))]"
                ],
                "properties": {
                    "serverFarmId": "[parameters('hostingPlanName')]"
                },
                "resources": [
                    {
                        "name": "web",
                        "type": "config",
                        "apiVersion": "2015-08-01",
                        "dependsOn": [
                            "[concat('Microsoft.Web/Sites/', variables('siteName'))]"
                        ],
                        "properties": {
                            "connectionStrings": [
                                {
                                    "ConnectionString": "[concat('Data Source=tcp:', reference(concat('Microsoft.Sql/servers/', parameters('serverName'))).fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('databaseName'), ';User Id=', parameters('administratorLogin'), '@', parameters('serverName'), ';Password=', parameters('administratorLoginPassword'), ';')]",
                                    "Name": "DefaultConnection",
                                    "Type": 2
                                }
                            ]
                        }
                    }
                ]
            }
        ]
    }


## 部署模板

在创建资源组和模板后，可以将模板中定义的基础结构部署到资源组。你可以使用 **New-AzureRmResourceGroupDeployment** cmdlet 部署资源。基本语法如下：

    PS C:\> New-AzureRmResourceGroupDeployment -ResourceGroupName TestRG1 -TemplateFile c:\Azure\Templates\azuredeploy.json

指定资源组及模板的位置。如果模板不在本地，你可以使用 -TemplateUri 参数并指定模板的 URI。

###动态模板参数

如果你熟悉 PowerShell 的话，就知道你可以通过键入减号 (-) 并按 TAB 键来切换 cmdlet 的可用参数。对于模板中定义的参数，同样也可以使用此功能。只要你键入模板名称，该 cmdlet 就会提取该模板、对其进行分析并将模板参数动态地添加到该命令。这使指定模板参数值变得非常轻松。而且，如果你忘记了必需的参数值，PowerShell 会提示你输入该值。

下面是包含参数的完整命令。对于资源的名称，你可以提供自己的值。

    PS C:\> New-AzureRmResourceGroupDeployment -ResourceGroupName TestRG1 -TemplateFile c:\Azure\Templates\azuredeploy.json -hostingPlanName freeplanwest -serverName exampleserver -databaseName exampledata -administratorLogin exampleadmin

当你输入该命令时，将提示你输入遗漏的必需参数 **administratorLoginPassword**。并且，当您键入密码时，将遮盖安全字符串值。此策略可以消除以纯文本格式提供密码的风险。

    cmdlet New-AzureRmResourceGroupDeployment at command pipeline position 1
    Supply values for the following parameters:
    (Type !? for Help.)
    administratorLoginPassword: ********

创建资源时，该命令将会运行并返回消息。最终，你将看到部署结果。

    DeploymentName    : azuredeploy
    ResourceGroupName : TestRG1
    ProvisioningState : Succeeded
    Timestamp         : 10/16/2015 12:55:50 AM
    Mode              : Incremental
    TemplateLink      :
    Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    hostingPlanName  String                     freeplanwest
                    serverName       String                     exampleserver
                    databaseName     String                     exampledata
                    administratorLogin  String                  exampleadmin
                    administratorLoginPassword  SecureString

    Outputs           :

在几个步骤中，我们将创建和部署复杂 Web 应用所需的资源。

## 获取有关资源组的信息

创建资源组之后，可以使用资源管理器模块中的 cmdlet 来管理该资源组。

- 若要获取你订阅中的所有资源组，请使用 **Get-AzureRmResourceGroup** cmdlet：

		PS C:\>Get-AzureRmResourceGroup

		ResourceGroupName : TestRG
		Location          : China East
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/{guid}/resourceGroups/TestRG
		
		...

- 若要获取资源组中的资源，请使用 **Get-AzureRmResource** cmdlet 及其 ResourceGroupName 参数。若不带参数，则 Get-AzureRmResource 获取在你的 Azure 订阅中的所有资源。

		PS C:\> Get-AzureRmResource -ResourceGroupName TestRG1
		
		Name              : exampleserver
                ResourceId        : /subscriptions/{guid}/resourceGroups/TestRG1/providers/Microsoft.Sql/servers/tfserver10
                ResourceName      : exampleserver
                ResourceType      : Microsoft.Sql/servers
                Kind              : v12.0
                ResourceGroupName : TestRG1
                Location          : China East
                SubscriptionId    : {guid}
                
                ...
	        

## 添加到资源组

若要将资源添加到资源组，可以使用 **New-AzureRmResource** cmdlet。但是，以这种方式添加资源可能会在将来造成混淆，因为模板中不存在新资源。如果重新部署旧模板，则会部署不完整的解决方案。如果你经常要部署模板，就会发现更方便且更可靠的方法是将新资源添加到模板，然后重新部署模板。

## 移动资源

可以将现有资源移动到新的资源组。有关示例，请参阅[将资源移动到新的资源组或订阅中](/documentation/articles/resource-group-move-resources)。

## 删除资源组

- 若要从资源组中删除资源，请使用 **Remove-AzureRmResource** cmdlet。此 cmdlet 将删除该资源，但不会删除该资源组。

	此命令从 TestRG 资源组中删除 TestSite Web 应用。

		Remove-AzureRmResource -Name TestSite -ResourceGroupName TestRG1 -ResourceType "Microsoft.Web/sites" -ApiVersion 2015-08-01

- 若要删除资源组，请使用 **Remove-AzureRmResourceGroup** cmdlet。此 cmdlet 将删除资源组及其资源。

		PS C:\> Remove-AzureRmResourceGroup -Name TestRG1
		
		Confirm
		Are you sure you want to remove resource group 'TestRG1'
		[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y



## 后续步骤

- 若要了解如何创建资源管理器模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。

<!---HONumber=79-->