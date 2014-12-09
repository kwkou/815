<properties pageTitle="将 Windows PowerShell 与资源管理器一起使用" metaKeywords="ResourceManager, PowerShell, Azure PowerShell" description="使用 Windows PowerShell 来创建资源组" metaCanonical="" services="" documentationCenter="" title="将 Windows PowerShell 与资源管理器一起使用" authors="juneb" solutions="" manager="mbaldwin" editor="mollybos" />

# 将 Windows PowerShell 与资源管理器一起使用

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/powershell-azure-resource-manager.md" title="Windows PowerShell" class="current">Windows PowerShell</a><a href="/zh-cn/documentation/articles/xplat-cli-azure-resource-manager.md" title="跨平台 CLI">跨平台 CLI</a></div>

资源管理器引入了一种考虑您的 Azure 资源的全新方法。您首先设想一项复杂的服务，而不是创建和管理各个资源，如博客、照片库、SharePoint 门户或 wiki。您使用模板（服务的资源模型）来创建具有您支持服务所需要的资源的资源组。然后，可以将该资源组作为一个逻辑单元进行管理和部署。

在本教程中，您将了解如何将 Windows PowerShell 与 Microsoft Azure 资源管理器一起使用。本教程将指导您逐步完成通过 SQL Database 创建和部署 Azure 托管网站（或 Web 应用程序）的资源组的过程，其中充分使用了支持该过程所需的所有资源。

**估计完成时间：**15 分钟

## 先决条件

在您可以将 Windows PowerShell 与资源管理器一起使用之前，您必须具有：

-   Windows PowerShell 3.0 版或 4.0 版。若要查找 Windows PowerShell 版本，请键入：`$PSVersionTable`并验证 `PSVersion` 的值是 3.0 或 4.0。若要安装兼容版本，请参阅 [Windows Management Framework 3.0][Windows Management Framework 3.0] 或 [Windows Management Framework 4.0][Windows Management Framework 4.0]。

-   Azure PowerShell 0.8.0 版或更高版本。若要安装最新版本，并将其与您的 Azure 订阅关联，请参阅[如何安装和配置 Windows Azure PowerShell][如何安装和配置 Windows Azure PowerShell]。

本教程专为 Windows PowerShell 新手设计，但它假定您了解基本概念，如模块、cmdlet 和会话。有关 Windows PowerShell 的详细信息，请参阅 [Windows PowerShell 入门][Windows PowerShell 入门]。

若要获得您在此教程中看到的任何 cmdlet 的详细帮助，请使用 Get-Help cmdlet。

    Get-Help <cmdlet-name> -Detailed

例如，若要获得有关 Add-AzureAccount cmdlet 的帮助，请键入：

    Get-Help Add-AzureAccount -Detailed

## 本教程的内容

-   [有关 Azure Powershell 模块][有关 Azure Powershell 模块]
-   [创建资源组][创建资源组]
-   [管理资源组][管理资源组]
-   [资源组疑难解答][资源组疑难解答]
-   [后续步骤][后续步骤]

## <span id="about"></span></a>有关 Azure PowerShell 模块

从版本 0.8.0 开始，Azure PowerShell 安装包括三个 Windows PowerShell 模块：

-   **Azure**：包括用于管理单个资源（如存储帐户、网站、数据库、虚拟机和媒体服务）的传统 cmdlet。有关详细信息，请参阅 [Azure 服务管理 Cmdlet][Azure 服务管理 Cmdlet]。

-   **AzureResourceManager**：包括可用于创建、管理和部署资源组的 Azure 资源的 cmdlet。有关详细信息，请参阅 [Azure 资源管理器 Cmdlet][Azure 资源管理器 Cmdlet]。

-   **AzureProfile**：包括上述两个模块公用的 cmdlet，如 Add-AzureAccount、Get-AzureSubscription 和 Switch-AzureMode。有关详细信息，请参阅 [Azure 配置文件 Cmdlet][Azure 配置文件 Cmdlet]。

> [WACOM.NOTE] Azure 资源管理器模块目前处于预览状态。它可能未提供与 Azure 模块相同的管理功能。

Azure 模块和 Azure 资源管理器模块不能在同一 Windows PowerShell 会话中使用。为了在这两个模块之间轻松切换，我们已经将新的 cmdlet（**Switch-AzureMode**）添加到 Azure 配置文件模块。

当使用 Azure PowerShell 时，默认情况下将导入 Azure 模块中的 cmdlet。若要切换到 Azure 资源管理器模块，请使用 Switch-AzureMode cmdlet。它将从您的会话中删除 Azure 模块，并导入 Azure 资源管理器模块和 Azure 配置文件模块。

若要切换到 AzureResoureManager 模块，请键入：

    PS C:PS C:\> Switch-AzureMode -Name AzureResourceManagergt; Switch-AzureMode -Name AzureResourceManager

若要切换回 Azure 模块，请键入：

    PS C:PS C:\> Switch-AzureMode -Name AzureServiceManagementgt; Switch-AzureMode -Name AzureServiceManagement

默认情况下，Switch-AzureMode 只影响当前会话。若要使切换在所有 Windows PowerShell 会话中生效，请使用 Switch-AzureMode 的 **Global** 参数。

有关 Switch-AzureMode cmdlet 的帮助，请键入：`Get-Help Switch-AzureMode` 或查看 [Switch-AzureMode][Switch-AzureMode]。

若要获取带有帮助摘要的 AzureResourceManager 模块中的 cmdlet 列表，请键入：

    PS C:\> Get-Command -Module AzureResourceManager | Get-Help | Format-Table Name, Synopsis

    Name                                   Synopsis
    ----                                   --------
    Get-AzureLocation                      Gets the Azure data center locations and the resource types that they support
    Get-AzureResource                      Gets Azure resources
    Get-AzureResourceGroup                 Gets Azure resource groups
    Get-AzureResourceGroupDeployment       Gets the deployments in a resource group.
    Get-AzureResourceGroupGalleryTemplate  Gets resource group templates in the gallery
    Get-AzureResourceGroupLog              Gets the deployment log for a resource group
    New-AzureResource                      Creates a new resource in a resource group
    New-AzureResourceGroup                 Creates an Azure resource group and its resources
    New-AzureResourceGroupDeployment       Add an Azure deployment to a resource group.
    Remove-AzureResource                   Deletes a resource
    Remove-AzureResourceGroup              Deletes a resource group.
    Save-AzureResourceGroupGalleryTemplate Saves a gallery template to a JSON file
    Set-AzureResource                      Changes the properties of an Azure resource.
    Stop-AzureResourceGroupDeployment      Cancels a resource group deployment
    Test-AzureResourceGroupTemplate        Detects errors in a resource group template or template parameters

若要获得 cmdlet 的完整帮助，请使用以下格式键入命令：

    Get-Help <cmdlet-name> -Full

例如，

    Get-Help Get-AzureLocation -Full

# <span id="create"></span></a>创建资源组

教程的本部分将指导您借助 SQL Database 完成创建和部署网站的资源组的过程。

您不必成为 Azure、SQL、网站或资源管理方面的专家，即可执行此任务。这些模板提供了具有您可能会需要的所有资源的资源组模型。而且，因为我们使用 Windows PowerShell 来自动执行任务，您可以将这些过程作为参照，以便为大规模任务撰写脚本。

## 步骤 1：切换到 Azure 资源管理器

1.  启动 Windows PowerShell。您可以使用任何您喜欢的主机程序，如 Windows PowerShell 控制台或 Windows PowerShell ISE。

2.  使用 **Switch-AzureMode** cmdlet 导入 AzureResourceManager 和 AzureProfile 模块中的 cmdlet。

    `PS C:PS C:\>Switch-AzureMode AzureResourceManager`gt;Switch-AzureMode AzureResourceManager</code>

3.  若要将您的 Azure 帐户添加到 Windows PowerShell 会话中，请使用 **Add-AzureAccount** cmdlet。

    `PS C:PS C:\> Add-AzureAccount`gt; Add-AzureAccount</code>

该 cmdlet 会提示您输入电子邮件地址和密码。然后它会下载您的帐户设置，以便这些信息可供 Windows PowerShell 使用。

帐户设置会过期，因此您需要不时刷新它们。若要刷新帐户设置，请再次运行 **Add-AzureAccount**。

> [WACOM.NOTE] AzureResourceManager 模块需要 Add-AzureAccount。一个发布设置文件是不够的。

## 步骤 2：选择库模板

创建资源组及其资源的方法有多种，但最简单的方法是使用资源组模板。*资源组模板*是定义资源组中的资源的 JSON 字符串。该字符串包含称为“参数”的占位符以用于用户定义的值，如名称和大小。

Azure 托管资源组模板库，而且可以从头开始创建您自己的模板，也可以通过编辑库模板进行创建。在本教程中，我们将使用库模板。

若要搜索 Azure 资源组模板库中的模板，请使用 **Get-AzureResourceGroupGalleryTemplate** cmdlet。

在 Windows Powershell 提示符处，键入：

    PS C:PS C:\> Get-AzureResourceGroupGalleryTemplategt; Get-AzureResourceGroupGalleryTemplate

该 cmdlet 返回具有发布服务器和标识属性的库模板的列表。使用 **Identity** 属性来确定命令中的模板。

    Publisher       Identity
    ---------       --------
    Ghost           Ghost.Ghost.0.1.0-preview1
    Joomla          Joomla.Joomla.0.1.0-preview1
    Microsoft       Microsoft.ASPNETEmptySite.0.1.0-preview1
    Microsoft       Microsoft.ASPNETstarterSite.0.1.0-preview1
    Microsoft       Microsoft.Bakery.0.1.0-preview1
    Microsoft       Microsoft.Boilerplate.0.1.0-preview1
    ...

提示：若要重新调用最后一个命令，请按向上箭头键。

Microsoft.WebSiteSQLDatabase.0.1.0-preview1 模板看起来很有趣。若要获取有关库模板的详细信息，请使用 **Identity** 参数。Identity 参数的值是该模板的标识。

    PS C:PS C:\> Get-AzureResourceGroupGalleryTemplate -Identity Microsoft.WebSiteSQLDatabase.0.1.0-preview1gt; Get-AzureResourceGroupGalleryTemplate -Identity Microsoft.WebSiteSQLDatabase.0.1.0-preview1

该 cmdlet 返回一个包含有关该模板的详细信息（包括描述）的对象。

    <p>Windows Azure Web Sites offers secure and flexible development, 
    deployment and scaling options for any sized web application. Leverage 
    your existing tools to create and deploy applications without the hassle 
    of managing infrastructure.</p>

此模板看起来会满足我们的需要。让我们将其保存到磁盘并更密切地查看它。

## 步骤 3：检查模板

让我们将该模板保存到磁盘上的 JSON 文件。此步骤不是必需的，但可以更容易地查看模板。若要保存该模板，请使用 **Save-AzureResourceGroupGalleryTemplate** cmdlet。使用其 **Identity** 参数来指定模板，并使用 **Path** 参数来指定磁盘上的路径。

Save-AzureResourceGroupGalleryTemplate 将保存模板，并返回 JSON 模板文件的文件名的路径。

    PS C:\> Save-AzureResourceGroupGalleryTemplate -Identity Microsoft.WebSiteSQLDatabase.0.1.0-preview1 -Path D:\Azure\Templates

    Path
    ----
    D:\Azure\Templates\Microsoft.WebSite.0.1.0-preview1.json

您可以在诸如记事本之类的文本编辑器中查看模板文件。每个模板都有**资源**部分和**参数**部分。

模板的**资源**部分列出了该模板将创建的资源。该模板创建 SQL Database 服务器和 SQL Database、服务器场和网站，以及多个管理设置。

每个资源的定义包括其属性（如名称、类型和位置）以及用户定义的值的参数。例如，模板的此部分定义 SQL Database。它包括数据库名称（[parameters('databaseName')]）、数据库服务器位置 [parameters('serverLocation')] 和排序规则属性 [parameters('collation')] 的参数。

        {
          "name": "[parameters('databaseName')]",
          "type": "databases",
          "location": "[parameters('serverLocation')]",
          "apiVersion": "2.0",
          "dependsOn": [
            "[concat('Microsoft.Sql/servers/', parameters('serverName'))]"
          ],
          "properties": {
            "edition": "Web",
            "collation": "[parameters('collation')]",
            "maxSizeBytes": "1073741824"
          }
        },

模板的**参数**部分是在所有的资源中都定义的参数的集合。它包括 databaseName、serverLocation 和排序规则属性。

    "parameters": {
    ...    

    "serverLocation": {
      "type": "string"
    }, 
    ...

    "databaseName": {
      "type": "string"
    },
    "collation": {
      "type": "string",
      "defaultValue": "SQL_Latin1_General_CP1_CI_AS"
    }

某些参数具有默认值。当使用该模板时，不需要提供这些参数的值。如果不指定一个值，则使用默认值。

    "collation": {
          "type": "string",
          "defaultValue": "SQL_Latin1_General_CP1_CI_AS"
        }

当参数具有枚举值时，将列出带有该参数的有效值。例如，**sku** 参数可以采用值“Free”、“Shared”、“Basic”或“Standard”。如果没有指定 **sku** 参数的值，则它使用默认值“Free”。

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

我们已基本准备好使用此模板，但在执行此操作之前，我们需要找到每种资源的位置。

## 步骤 4：获取资源类型的位置

大多数模板要求您为资源组中的每种资源指定位置。所有资源都位于 Azure 数据中心，但不是每个 Azure 数据中心都支持每种资源类型。

选择支持资源类型的任何位置。资源组中的资源不需要处于同一位置，而且他们不需要处于与资源组或订阅相同的位置。

若要获取支持每种资源类型的位置，请使用 **Get-AzureLocation** cmdlet。下面是输出的摘录。（此输出与您的输出可能会不同。详细信息可能会随时间而变化。）

    Name                                 Locations
    ----                                 ---------
    ResourceGroup                        East Asia, South East Asia, East US, West US, North Central US,
                                         South Central US, Central US, North Europe, West Europe

    Microsoft.Sql/servers/databases      Brazil South, Central US, East Asia, East US, East US 2, Japan
                                         East, Japan West, North Central US, North Europe, South Central US,
                                         Southeast Asia, West Europe, West US

    Microsoft.Web/serverFarms            Brazil South, East Asia, East US, Japan East, Japan West, North
                                         Central US, North Europe, West Europe, West US

    Microsoft.Web/sites                  Brazil South, East Asia, East US, Japan East, Japan West, North
                                         Central US, North Europe, West Europe, West US

现在，我们有了需要创建资源组的信息。

## 步骤 5：创建资源组

在此步骤中，我们将使用资源组模板来创建资源组。作为参考，请打开磁盘上的 Microsoft.WebSiteSQLDatabase.0.1.0-preview1 JSON 文件并按照该文件操作。

若要创建资源组，请使用 **New-AzureResourceGroup** cmdlet。

该命令使用 **Name** 参数来指定资源组的名称，并使用 **Location** 参数来指定其位置。使用 **Get-AzureLocation** 的输出来选择资源组的位置。它使用 **GalleryTemplateIdentity** 参数来指定库模板。

    PS C:\> New-AzureResourceGroup ` 
            -Name TestRG1 `
            -Location "China East" `
            -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1 `
            ....

只要您键入模板名称，New-AzureResourceGroup 就会提取该模板、对其进行分析并将模板参数动态地添加到该命令。这使指定模板参数值变得非常轻松。而且，如果您忘记了必需的参数值，Windows PowerShell 会提示您输入该值。

**动态模板参数**

若要获取这些参数，请键入减号 (-) 指示参数名称，然后按 TAB 键。或者，键入参数名称的前几个字母，例如“siteName”，然后按 TAB 键。

        PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1
        -si<TAB>

Windows PowerShell 会完成参数名称。若要循环显示参数名称，请重复按 TAB。

        PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1
        -siteName 

输入网站的名称，并为每个参数重复 TAB 过程。具有默认值的参数是可选的。若要接受默认值，请在命令中省略此参数。

当模板参数具有枚举值（如此模板中的 sku 参数）以循环显示这些参数值时，请按 TAB 键。

        PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1
        -siteName TestSite -sku <TAB>

        PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1
        -siteName TestSite -sku Free<TAB>

        PS C:\> New-AzureResourceGroup -Name TestRG1 -Location "China East" -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1
        -siteName TestSite -sku Basic<TAB>

下面是 New-AzureResourceGroup 命令的示例，此命令仅指定所需模板参数和 **Verbose** 通用参数。请注意，省略了 **administratorLoginPassword**。（反引号 (\`) 是 Windows PowerShell 行继续符。）

    PS C:\> New-AzureResourceGroup 
    -Name TestRG `
    -Location "China East" `
    -GalleryTemplateIdentity Microsoft.WebSiteSQLDatabase.0.1.0-preview1 `
    -siteName TestSite `
    -hostingPlanName TestPlan `
    -siteLocation "China East" `
    -serverName testserver `
    -serverLocation "China East" `
    -administratorLogin Admin01 `
    -databaseName TestDB `
    -Verbose

当您输入该命令时，将提示您输入遗漏的必需参数 **administratorLoginPassword**。并且，当您键入密码时，将遮盖安全字符串值。此策略可以消除以纯文本格式提供密码的风险。

    cmdlet New-AzureResourceGroup at command pipeline position 1
    Supply values for the following parameters:
    (Type !? for Help.)
    administratorLoginPassword: **********

**New-AzureResourcGroup** 返回其创建和部署的资源组。下面是命令的输出，包括详细输出。

    VERBOSE: 3:47:30 PM - Create resource group 'TestRG' in location 'China East'
    VERBOSE: 3:47:30 PM - Template is valid.
    VERBOSE: 3:47:31 PM - Create template deployment 'Microsoft.WebSiteSQLDatabase.0.1.0-preview1'
    using template https://gallerystoreprodch.blob.core.chinacloudapi.cn/prod-microsoft-windowsazure-gallery/8D6B920B-10F4-4B5A-B3DA-9D398FBCF3EE.PUBLICGALLERYITEMS.MICROSOFT.WEBSITESQLDATABASE.0.1.0-PREVIEW1/DeploymentTemplates/Website_NewHostingPlan_SQL_NewDB-Default.json.
    VERBOSE: 3:47:43 PM - Resource Microsoft.Sql/servers 'testserver' provisioning status is succeeded
    VERBOSE: 3:47:43 PM - Resource Microsoft.Web/serverFarms 'TestPlan' provisioning status is
    succeeded
    VERBOSE: 3:47:47 PM - Resource Microsoft.Sql/servers/databases 'testserver/TestDB' provisioning status is succeeded
    VERBOSE: 3:47:47 PM - Resource microsoft.insights/autoscalesettings 'TestPlan-TestRG'
    provisioning status is succeeded
    VERBOSE: 3:47:47 PM - Resource Microsoft.Sql/servers/firewallrules
    'testserver/AllowAllWindowsAzureIps' provisioning status is succeeded
    VERBOSE: 3:47:50 PM - Resource Microsoft.Web/Sites 'TestSite' provisioning status is succeeded
    VERBOSE: 3:47:54 PM - Resource Microsoft.Web/Sites/config 'TestSite/web' provisioning status is succeeded
    VERBOSE: 3:47:54 PM - Resource microsoft.insights/alertrules 'ServerErrors-TestSite' provisioning
    status is succeeded
    VERBOSE: 3:47:57 PM - Resource microsoft.insights/components 'TestSite' provisioning status is succeeded


    ResourceGroupName : TestRG
    Location          : China East
    ProvisioningState : Succeeded
    Resources         :
                    Name                   Type                                  Location
                    =====================  ====================================  =========
                    ServerErrors-TestSite  microsoft.insights/alertrules         China East
                    TestPlan-TestRG        microsoft.insights/autoscalesettings  China East
                    TestSite               microsoft.insights/components         China East
                    testserver             Microsoft.Sql/servers                 China East
                    TestDB                 Microsoft.Sql/servers/databases       China East
                    TestPlan               Microsoft.Web/serverFarms             China East
                    TestSite               Microsoft.Web/sites                   China East

只需几个步骤，我们就已创建和部署了一个复杂网站所需的资源。
库模板提供了我们执行此任务所需要的几乎所有信息。
而且，此任务很容易实现自动化。

# <span id="manage"></span></a>管理资源组

创建资源组之后，可以使用 AzureResourceManager 模块中的 cmdlet 来管理该资源组、对其进行更改、向其添加资源以及将其删除。

-   若要获取您订阅中的资源组，请使用 **Get-AzureResourceGroup** cmdlet：

        PS C:>Get-AzureResourceGroup

        ResourceGroupName : TestRG
        Location          : China East
        ProvisioningState : Succeeded
        Resources         :
                        Name                   Type                                  Location
                        =====================  ====================================  =========
                        ServerErrors-TestSite  microsoft.insights/alertrules         China East
                        TestPlan-TestRG        microsoft.insights/autoscalesettings  China East
                        TestSite               microsoft.insights/components         China East
                        testserver             Microsoft.Sql/servers                 China East
                        TestDB                 Microsoft.Sql/servers/databases       China East
                        TestPlan               Microsoft.Web/serverFarms             China East
                        TestSite               Microsoft.Web/sites                   China East

-   若要获取资源组中的资源，请使用 **GetAzureResource** cmdlet 及其 ResourceGroupName 参数。若不带参数，则 Get-AzureResource 获取在您的 Azure 订阅中的所有资源。

        PS C:\> Get-AzureResource -ResourceGroupName TestRG

        Name                   ResourceType                          Location
        ----                   ------------                          --------
        ServerErrors-TestSite  microsoft.insights/alertrules         China East
        TestPlan-TestRG        microsoft.insights/autoscalesettings  China East
        TestSite               microsoft.insights/components         China East
        testserver             Microsoft.Sql/servers                 China East
        TestDB                 Microsoft.Sql/servers/databases       China East
        TestPlan               Microsoft.Web/serverFarms             China East
        TestSite               Microsoft.Web/sites                   China East

-   若要将资源添加到资源组，请使用 **New-AzureResource** cmdlet。此命令将新的网站添加到 TestRG 资源组。此命令稍微有些复杂，因为它不使用模板。

        PS C:\>New-AzureResource -Name TestSite2 `
        -Location "China East" `
        -ResourceGroupName TestRG `
        -ResourceType "Microsoft.Web/sites" `
        -ApiVersion 2004-04-01 `
        -PropertyObject @{"name" = "TestSite2"; "siteMode"= "Limited"; "computeMode" = "Shared"}

-   若要将新的基于模板的部署添加到资源组，请使用 **New-AzureResourceGroupDeployment** 命令。

        PS C:\>New-AzureResourceGroupDeployment ` 
        -ResourceGroupName TestRG `
        -GalleryTemplateIdentity Microsoft.WebSite.0.1.0-preview1 `
        -siteName TestWeb2 `
        -hostingPlanName TestDeploy2 `
        -siteMode Limited `
        -computeMode Dedicated `
        -siteLocation "China East" `
        -subscriptionID "9b14a38b-4b93-4554-8bb0-3cefb47abcde" `
        -resourceGroup TestRG

-   若要从资源组中删除资源，请使用 **Remove-AzureResource** cmdlet。此 cmdlet 将删除该资源，但不会删除该资源组。

    此命令从 TestRG 资源组中删除 TestSite2 网站。

        Remove-AzureResource -Name TestSite2 `
            -Location "China East" `
            -ResourceGroupName TestRG `
            -ResourceType "Microsoft.Web/sites" `
            -ApiVersion 2004-04-01

-   若要删除资源组，请使用 **Remove-AzureResourceGroup** cmdlet。此 cmdlet 将删除资源组及其资源。

        PS C:\ps-test> Remove-AzureResourceGroup -Name TestRG

        Confirm
        Are you sure you want to remove resource group 'TestRG'
        [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y

# <span id="troubleshoot"></span></a>资源组疑难解答

当您在 AzureResourceManager 模块中试用 cmdlet 时，您很可能会遇到错误。请使用本部分中的提示来解决它们。

## 防止错误

AzureResourceManager 模块包括帮助您防止错误的 cmdlet。

-   **Get-AzureLocation**：此 cmdlet 获取支持每种类型的资源的位置。输入资源的位置之前，请使用此 cmdlet 验证该位置是否支持此资源类型。

-   **Test-AzureResourceGroupTemplate**：先测试您的模板和模板参数，然后再使用它们。输入自定义或库模板，以及您打算使用的模板参数值。此 cmdlet 测试该模板是否内部一致以及参数值的设置是否与该模板匹配。

## 修复错误

-   **Get-AzureResourceGroupLog**：此 cmdlet 获取资源组中每个部署的日志中的条目。如果出现问题，则从检查部署日志开始。

-   **详细和调试**：AzureResourceManager 模块中的 cmdlet 调用执行实际工作的 REST API。若要查看 API 返回的消息，请将 $DebugPreference 变量设置为“Continue”，并在您的命令中使用 Verbose 通用参数。这些消息经常提供有关任何失败原因的重要线索。

-   **您的 Azure 凭据尚未设置，或已过期**：若要刷新 Windows PowerShell 会话中的凭据，请使用 Add-AzureAccount cmdlet。发布设置文件中的凭据不足以满足 AzureResourceManager 模块中的 cmdlet 的需要。

# <span id="next"></span></a> 后续步骤

若要了解有关将 Windows PowerShell 与资源管理器一起使用的详细信息：

-   [Azure 资源管理器 Cmdlet][1]：了解如何在 AzureResourceManager 模块中使用这些 cmdlet。
-   [使用资源组管理 Azure 资源][使用资源组管理 Azure 资源]：了解如何创建和管理 Azure 管理门户中的资源组。
-   [将 Azure 跨平台命令行界面与资源管理器一起使用][将 Azure 跨平台命令行界面与资源管理器一起使用]：了解如何使用在许多操作系统平台上工作的命令行工具来创建和管理资源组。
-   [Azure 博客][Azure 博客]：了解 Azure 中的新功能。
-   [Windows PowerShell 博客][Windows PowerShell 博客]：了解 Windows PowerShell 中的新功能。
-   [“您好，脚本编写专家！”博客][“您好，脚本编写专家！”博客]：从 Windows PowerShell 社区获取实用提示和技巧。

  [Windows Management Framework 3.0]: http://www.microsoft.com/zh-cn/download/details.aspx?id=34595
  [Windows Management Framework 4.0]: http://www.microsoft.com/zh-cn/download/details.aspx?id=40855
  [如何安装和配置 Windows Azure PowerShell]: http://azure.microsoft.com/zh-cn/documentation/articles/install-configure-powershell/
  [Windows PowerShell 入门]: http://technet.microsoft.com/zh-cn/library/hh857337.aspx
  [有关 Azure Powershell 模块]: #about
  [创建资源组]: #create
  [管理资源组]: #manage
  [资源组疑难解答]: #troubleshoot
  [后续步骤]: #next
  [Azure 服务管理 Cmdlet]: http://msdn.microsoft.com/zh-cn/library/jj152841.aspx
  [Azure 资源管理器 Cmdlet]: http://msdn.microsoft.com/zh-cn/library/azure/dn654592.aspx
  [Azure 配置文件 Cmdlet]: http://go.microsoft.com/fwlink/?LinkID=394766
  [Switch-AzureMode]: http://go.microsoft.com/fwlink/?LinkID=394398
  [1]: http://go.microsoft.com/fwlink/?LinkID=394765&clcid=0x409
  [使用资源组管理 Azure 资源]: http://azure.microsoft.com/zh-cn/documentation/articles/azure-preview-portal-using-resource-groups
  [将 Azure 跨平台命令行界面与资源管理器一起使用]: http://www.windowsazure.com/zh-cn/documentation/articles/xplat-cli-azure-resource-manager/
  [Azure 博客]: http://blogs.msdn.com/windowsazure
  [Windows PowerShell 博客]: http://blogs.msdn.com/powershell
  [“您好，脚本编写专家！”博客]: http://blogs.technet.com/b/heyscriptingguy/
