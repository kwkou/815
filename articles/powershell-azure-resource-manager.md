<properties 
	pageTitle="将 Azure PowerShell 与 Azure 资源管理器配合使用" 
	description="使用 Azure PowerShell 将作为资源组的多个资源部署到 Azure。" 
	services="azure-resource-manager" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="azure-resource-manager" 
	ms.date="07/15/2015" 
	wacn.date="10/3/2015"/>

# 将 Azure PowerShell 与 Azure 资源管理器配合使用

> [AZURE.SELECTOR]
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager)
- [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager)

Azure 资源管理器引入了一种考虑您的 Azure 资源的全新方法。您首先设想一项复杂的服务，而不是创建和管理各个资源，如博客、照片库、SharePoint 门户或 wiki。您使用模板（服务的资源模型）来创建具有您支持服务所需要的资源的资源组。然后，可以将该资源组作为一个逻辑单元进行管理和部署。

在本教程中，您将了解如何将 Azure PowerShell 与 Microsoft Azure 的 Azure 资源管理器一起使用。本教程将指导您逐步完成通过 SQL 数据库创建和部署 Azure 托管的 Web 应用的资源组的过程，其中充分使用了支持该过程所需的所有资源。

## 先决条件

若要完成本教程，您必须具有 Azure PowerShell 版本 0.8.0 或更高版本。若要安装最新版本并将其与 Azure 订阅相关联，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

本教程专为 PowerShell 新手设计，但它假定您了解基本概念，如模块、cmdlet 和会话。有关 Windows PowerShell 的详细信息，请参阅 [Windows PowerShell 入门](http://technet.microsoft.com/zh-cn/library/hh857337.aspx)。

若要获得您在此教程中看到的任何 cmdlet 的详细帮助，请使用 Get-Help cmdlet。

	Get-Help <cmdlet-name> -Detailed

例如，若要获得有关 Add-AzureAccount cmdlet 的帮助，请键入：

	Get-Help Add-AzureAccount -Detailed

## 有关 Azure PowerShell 模块
从版本 0.8.0 开始，Azure PowerShell 安装包括多个 PowerShell 模块。您必须明确确定是否要使用 Azure 模块中的还是 Azure 资源管理器模块中的可用命令。为了在这两个模块之间轻松切换，我们已经将新的 cmdlet（**Switch-AzureMode**）添加到 Azure 配置文件模块。

当使用 Azure PowerShell 时，默认情况下将导入 Azure 模块中的 cmdlet。若要切换到 Azure 资源管理器模块，请使用 Switch-AzureMode cmdlet。它将从您的会话中删除 Azure 模块，并导入 Azure 资源管理器模块和 Azure 配置文件模块。

若要切换到 AzureResoureManager 模块，请键入：

    PS C:\> Switch-AzureMode -Name AzureResourceManager

若要切换回 Azure 模块，请键入：

    PS C:\> Switch-AzureMode -Name AzureServiceManagement

默认情况下，Switch-AzureMode 只影响当前会话。若要使切换在所有 PowerShell 会话中生效，请使用 Switch-AzureMode 的 **Global** 参数。

有关 Switch-AzureMode cmdlet 的帮助，请键入：`Get-Help Switch-AzureMode`，或查看 [Switch-AzureMode](http://go.microsoft.com/fwlink/?LinkID=394398)。
  
若要获取带有帮助摘要的 AzureResourceManager 模块中的 cmdlet 列表，请键入：

    PS C:\> Get-Command -Module AzureResourceManager | Get-Help | Format-Table Name, Synopsis

输出与以下摘要类似：

	Name                                   Synopsis
	----                                   --------
	Add-AlertRule                          Adds or updates an alert rule of either metric, event, o...
	Add-AzureAccount                       Adds the Azure account to Windows PowerShell
	Add-AzureEnvironment                   Creates an Azure environment
	Add-AzureKeyVaultKey                   Creates a key in a vault or imports a key into a vault.
        ...

若要获得 cmdlet 的完整帮助，请使用以下格式键入命令：

	Get-Help <cmdlet-name> -Full

例如，

	Get-Help Get-AzureLocation -Full

有关完整的 Azure 资源管理器命令集的信息，请参阅 [Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt125356.aspx)。
  
## 创建资源组

教程的本部分将指导您借助 SQL 数据库完成创建和部署 web 应用的资源组的过程。

您不必成为 Azure、SQL、Web 应用或资源管理方面的专家，即可执行此任务。这些模板提供了具有您可能会需要的所有资源的资源组模型。而且，因为我们使用 Windows PowerShell 来自动执行任务，您可以将这些过程作为参照，以便为大规模任务撰写脚本。

### 步骤 1：切换到 Azure 资源管理器 
1. 启动 PowerShell。您可以使用任何您喜欢的主机程序，如 Azure PowerShell 控制台或 Windows PowerShell ISE。

2. 使用 **Switch-AzureMode** cmdlet 导入 AzureResourceManager 和 AzureProfile 模块中的 cmdlet。

        PS C:\> Switch-AzureMode AzureResourceManager

3. 若要将您的 Azure 帐户添加到 Windows PowerShell 会话中，请使用 **Add-AzureAccount** cmdlet。

        PS C:\> Add-AzureAccount -Environment AzureChinaCloud

该 cmdlet 将提示您提供您的 Azure 帐户的登录凭据。登录后它会下载您的帐户设置，以便这些信息可供 Windows PowerShell 使用。

帐户设置会过期，因此您需要不时刷新它们。若要刷新帐户设置，请再次运行 **Add-AzureAccount**。

>[AZURE.NOTE]AzureResourceManager 模块需要 Add-AzureAccount -Environment AzureChinaCloud。一个发布设置文件是不够的。

### 步骤 2：选择库模板

创建资源组及其资源的方法有多种，但最简单的方法是使用资源组模板。*资源组模板*是定义资源组中的资源的 JSON 字符串。该字符串包含称为“参数”的占位符以用于用户定义的值，如名称和大小。

Azure 托管资源组模板库，而且可以从头开始创建您自己的模板，也可以通过编辑库模板进行创建。在本教程中，我们将使用库模板。

若要查看 Azure 资源组模板库中的所有模板，请使用 **Get AzureResourceGroupGalleryTemplate** cmdle；但是，此命令返回大量的模板。若要查看更易于管理的多个模板，请指定发布者参数。

在 Powershell 提示符处，键入：
    
    PS C:\> Get-AzureResourceGroupGalleryTemplate -Publisher Microsoft

该 cmdlet 返回由 Microsoft 发布的库模板列表。使用 **Identity** 属性来确定命令中的模板。

Microsoft.WebSiteSQLDatabase.0.2.6-preview 模板看起来很有趣。当您运行该命令时，因为已发布了新版本，所以模板版本可能略有不同。使用模版的最新版本。若要获取有关库模板的详细信息，请使用 **Identity** 参数。Identity 参数的值是该模板的标识。

    PS C:\> Get-AzureResourceGroupGalleryTemplate -Identity Microsoft.WebSiteSQLDatabase.0.2.6-preview

该 cmdlet 返回一个包含有关该模板的详细信息（包括摘要和描述）的对象。

此模板看起来会满足我们的需要。让我们将其保存到磁盘并更密切地查看它。

### 步骤 3：检查模板

让我们将该模板保存到磁盘上的 JSON 文件。此步骤不是必需的，但可以更容易地查看模板。若要保存该模板，请使用 **Save-AzureResourceGroupGalleryTemplate** cmdlet。使用其 **Identity** 参数来指定模板，并使用 **Path** 参数来指定磁盘上的路径。

Save-AzureResourceGroupGalleryTemplate 将保存模板，并返回 JSON 模板文件的文件名的路径。

	PS C:\> Save-AzureResourceGroupGalleryTemplate -Identity Microsoft.WebSiteSQLDatabase.0.2.6-preview -Path C:\Azure\Templates\New_WebSite_And_Database.json

	Path
	----
	C:\Azure\Templates\New_WebSite_And_Database.json


您可以在诸如记事本之类的文本编辑器中查看模板文件。每个模板都有**参数**部分和**资源**部分。

模板的**参数**部分是在所有的资源中都定义的参数的集合。它包括您在设置资源组时可提供的属性值。

    "parameters": {
      "siteName": {
        "type": "string"
      },
      "hostingPlanName": {
        "type": "string"
      },
      "siteLocation": {
        "type": "string"
      },
      ...
    }

某些参数具有默认值。当使用该模板时，不需要提供这些参数的值。如果不指定一个值，则使用默认值。

    "collation": {
      "type": "string",
      "defaultValue": "SQL_Latin1_General_CP1_CI_AS"
    },

当参数具有枚举值时，将列出带有该参数的有效值。例如，**sku** 参数可以采用值“免费”、“共享”、“基本”或“Standard标准”。如果没有指定 **sku** 参数的值，则它使用默认值“免费”。

    "sku": {
      "type": "string",
      "allowedValues": [
        "Free",
        "Shared",
        "Basic",
        "Standard"
      ],
      "defaultValue": "Free"
    },


请注意，**administratorLoginPassword** 参数使用安全字符串，而非纯文本。当您提供安全字符串的值时，该值将被遮盖。

	"administratorLoginPassword": {
      "type": "securestring"
    },

模板的**资源**部分列出了该模板将创建的资源。该模板创建 SQL 数据库服务器和 SQL 数据库、服务器场和网站，以及多个管理设置。
  
每个资源的定义包括其属性（如名称、类型和位置）以及用户定义的值的参数。例如，模板的此部分定义 SQL 数据库。它包括数据库名称（[parameters('databaseName')]）、数据库服务器位置 [parameters('serverLocation')] 和排序规则属性 [parameters('collation')] 的参数。

    {
        "name": "[parameters('databaseName')]",
        "type": "databases",
        "location": "[parameters('serverLocation')]",
        "apiVersion": "2.0",
        "dependsOn": [
          "[concat('Microsoft.Sql/servers/', parameters('serverName'))]"
        ],
        "properties": {
          "edition": "[parameters('edition')]",
          "collation": "[parameters('collation')]",
          "maxSizeBytes": "[parameters('maxSizeBytes')]",
          "requestedServiceObjectiveId": "[parameters('requestedServiceObjectiveId')]"
        }
    },


我们已基本准备好使用此模板，但在执行此操作之前，我们需要找到每种资源的位置。

### 步骤 4：获取资源类型的位置

大多数模板要求您为资源组中的每种资源指定位置。所有资源都位于 Azure 数据中心，但不是每个 Azure 数据中心都支持每种资源类型。

选择支持资源类型的任何位置。不需要在资源组中的同一位置处创建所有资源；但是，在有条件的情况下，您可以将要优化性能的资源创建在同一位置处。特别是，您可以确保您的数据库是在应用程序访问它所在的位置。

若要获取支持每种资源类型的位置，请使用 **Get-AzureLocation** cmdlet。若要限制您对特定类型资源的输出（例如 ResourceGroup），请使用：

    Get-AzureLocation | Where-Object Name -eq "ResourceGroup" | Format-Table Name, LocationsString -Wrap

输出结果将会类似于：

    Name                                 LocationsString
    ----                                 ---------------
    ResourceGroup                        China East, China North

现在，我们有了需要创建资源组的信息。

### 步骤 5：创建资源组
 
在此步骤中，我们将使用资源组模板来创建资源组。作为参考，请打开磁盘上的 New\_WebSite\_And\_Database.json 文件并按照该文件操作。模板文件对于确定要传递的参数值（如资源的 ApiVersion 正确值）非常有帮助。

若要创建资源组，请使用 **New-AzureResourceGroup** cmdlet。

该命令使用 **Name** 参数来指定资源组的名称，并使用 **Location** 参数来指定其位置。使用 **Get-AzureLocation** 的输出来选择资源组的位置。它使用 **GalleryTemplateIdentity** 参数来指定库模板。

	PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview
            ....

只要您键入模板名称，New-AzureResourceGroup 就会提取该模板、对其进行分析并将模板参数动态地添加到该命令。这使指定模板参数值变得非常轻松。而且，如果您忘记了必需的参数值，Windows PowerShell 会提示您输入该值。

**动态模板参数**

若要获取这些参数，请键入减号 (-) 指示参数名称，然后按 TAB 键。或者，键入参数名称的前几个字母，例如“siteName”，然后按 TAB 键。

    PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -si<TAB>

PowerShell 会完成参数名称。若要循环显示参数名称，请重复按 TAB。

    PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -siteName 

输入网站的名称，并为每个参数重复 TAB 过程。具有默认值的参数是可选的。若要接受默认值，请在命令中省略此参数。

当模板参数具有枚举值（如此模板中的 sku 参数）以循环显示这些参数值时，请按 TAB 键。

    PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -siteName TestSite -sku <TAB>

    PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -siteName TestSite -sku Basic<TAB>

    PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -siteName TestSite -sku Free<TAB>

下面是 New-AzureResourceGroup 命令的示例，此命令仅指定所需模板参数和 **Verbose** 通用参数。请注意，省略了 **administratorLoginPassword**。

	PS C:\> New-AzureResourceGroup -Name TestRG -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.2.6-preview -siteName TestSite -hostingPlanName TestPlan -siteLocation "East Asia" -serverName testserver -serverLocation "East Asia" -administratorLogin Admin01 -databaseName TestDB -Verbose

当您输入该命令时，将提示您输入遗漏的必需参数 **administratorLoginPassword**。并且，当您键入密码时，将遮盖安全字符串值。此策略可以消除以纯文本格式提供密码的风险。

	cmdlet New-AzureResourceGroup at command pipeline position 1
	Supply values for the following parameters:
	(Type !? for Help.)
	administratorLoginPassword: **********

**New-AzureResourceGroup** 返回其创建和部署的资源组。

在几个步骤中，我们将创建和部署复杂网站所需的资源。库模板提供了几乎所有我们执行此任务所需的信息。并且，该任务可自动轻松地执行操作。

## 获取有关资源组的信息

创建资源组之后，可以使用 AzureResourceManager 模块中的 cmdlet 来管理该资源组。

- 若要获取您订阅中的所有资源组，请使用 **Get-AzureResourceGroup** cmdlet：

		PS C:\>Get-AzureResourceGroup

		ResourceGroupName : TestRG
		Location          : China East
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG
		
		...

- 若要获取资源组中的资源，请使用 **Get-AzureResource** cmdlet 及其 ResourceGroupName 参数。若不带参数，则 Get-AzureResource 获取在您的 Azure 订阅中的所有资源。

		PS C:\> Get-AzureResource -ResourceGroupName TestRG
		
				Name                   Type                          Location
		----                   ------------                          --------
		ServerErrors-TestSite  microsoft.insights/alertrules         China East
	    TestPlan-TestRG        microsoft.insights/autoscalesettings  China East
	    TestSite               microsoft.insights/components         China East
	    testserver             Microsoft.Sql/servers                 China East
	    TestDB                 Microsoft.Sql/servers/databases       China East
	    TestPlan               Microsoft.Web/serverFarms             China East
	    TestSite               Microsoft.Web/sites                   China East
		ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG

## 添加到资源组

- 若要将资源添加到资源组，请使用 **New-AzureResource** cmdlet。此命令将新的网站添加到 TestRG 资源组。此命令稍微有些复杂，因为它不使用模板。 

		PS C:\>New-AzureResource -Name TestSite2 `
		-Location "China East" `
		-ResourceGroupName TestRG `
		-ResourceType "Microsoft.Web/sites" `
		-ApiVersion 2004-04-01 `
		-PropertyObject @{"name" = "TestSite2"; "siteMode"= "Limited"; "computeMode" = "Shared"}

- 若要将新的基于模板的部署添加到资源组，请使用 **New-AzureResourceGroupDeployment** 命令。

		PS C:\>New-AzureResourceGroupDeployment ` 
		-ResourceGroupName TestRG `
		-GalleryTemplateIdentity Microsoft.WebSite.0.2.6-preview `
		-siteName TestWeb2 `
		-hostingPlanName TestDeploy2 `
		-siteLocation "North Europe" 

## 移动资源

可以将现有资源移动到新的资源组。有关示例，请参阅[将资源移动到新的资源组或订阅中](/documentation/articles/resource-group-move-resources)。

## 删除资源组

- 若要从资源组中删除资源，请使用 **Remove-AzureResource** cmdlet。此 cmdlet 将删除该资源，但不会删除该资源组。

	此命令从 TestRG 资源组中删除 TestSite2 网站。

		Remove-AzureResource -Name TestSite2 -ResourceGroupName TestRG -ResourceType "Microsoft.Web/sites" -ApiVersion 2014-06-01

- 若要删除资源组，请使用 **Remove-AzureResourceGroup** cmdlet。此 cmdlet 将删除资源组及其资源。

		PS C:\ps-test> Remove-AzureResourceGroup -Name TestRG
		
		Confirm
		Are you sure you want to remove resource group 'TestRG'
		[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


## 资源组疑难解答
当您在 AzureResourceManager 模块中试用 cmdlet 时，您很可能会遇到错误。请使用本部分中的提示来解决它们。

### 防止错误

AzureResourceManager 模块包括帮助您防止错误的 cmdlet。


- **Get-AzureLocation**：此 cmdlet 获取支持每种资源类型的位置。输入资源的位置之前，请使用此 cmdlet 验证该位置是否支持此资源类型。


- **Test-AzureResourceGroupTemplate**：在你使用模板和模板参数之前对其进行测试。输入自定义或库模板，以及您打算使用的模板参数值。此 cmdlet 测试该模板是否内部一致以及参数值的设置是否与该模板匹配。



### 修复错误

- **Get-AzureResourceGroupLog**：此 cmdlet 获取资源组中每个部署的日志中的条目。如果出现问题，则从检查部署日志开始。 

- **详细和调试**：AzureResourceManager 模块中的 cmdlet 调用执行实际工作的 REST API。若要查看 API 返回的消息，请将 $DebugPreference 变量设置为“Continue”，并在您的命令中使用 Verbose 通用参数。这些消息经常提供有关任何失败原因的重要线索。

- **你的 Azure 凭据尚未设置，或已过期**：若要刷新 Windows PowerShell 会话中的凭据，请使用 Add-AzureAccount cmdlet。发布设置文件中的凭据不足以满足 AzureResourceManager 模块中的 cmdlet 的需要。


## 后续步骤

- 若要了解如何创建资源管理器模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)。
<!--
- 若要了解部署模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy)。
- 有关部署项目的详细示例，请参阅[按可预见的方式在 Azure 中部署微服务](/documentation/articles/app-service-web/app-service-deploy-complex-application-predictably)。
-->
<!---HONumber=71-->