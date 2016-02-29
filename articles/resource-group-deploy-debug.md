<properties
   pageTitle="资源组部署故障排除 | Microsoft Azure"
   description="描述部署使用资源管理器部署模型创建的资源时出现的常见问题，并说明如何检测和修复这些问题。"
   services="azure-resource-manager,virtual-machines"
   documentationCenter=""
   tags="top-support-issue"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
	ms.service="azure-resource-manager"
	ms.date="01/06/2016"
	wacn.date=""/>

# Azure 中的资源组部署故障排除

在部署期间遇到问题时，你需要了解何处出现了问题。资源管理器提供了两种方式来查明发生了什么情况以及发生的原因。你可以使用部署命令，为资源组获取有关特定部署的信息。或者，可以使用审计日志来获取有关对某个资源组所执行全部操作的信息。利用这些信息，可以解决问题并恢复解决方案中的操作。

本主题主要介绍使用部署命令来排除部署问题。有关使用审计日志来跟踪对资源所执行全部操作的信息，请参阅[使用资源管理器审计操作](/documentation/articles/resource-group-audit)。

本主题介绍如何通过 Azure PowerShell、Azure CLI 和 REST API 获取故障排除信息。

本主题还描述了用户常见错误的解决方案。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)]经典部署模型。不能使用经典部署模型创建资源组。


## 使用 PowerShell 进行故障排除

[AZURE.INCLUDE [powershell-preview-inline-include](../includes/powershell-preview-inline-include.md)]

可以用 **Get-AzureRmResourceGroupDeployment** 命令获取部署的总体状态。在下面的示例中，部署已失败。

    PS C:\> Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName ExampleDeployment

    DeploymentName    : ExampleDeployment
    ResourceGroupName : ExampleGroup
    ProvisioningState : Failed
    Timestamp         : 8/27/2015 8:03:34 PM
    Mode              : Incremental
    TemplateLink      :
    Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    siteName         String                     ExampleSite
                    hostingPlanName  String                     ExamplePlan
                    siteLocation     String                     China North
                    sku              String                     Free
                    workerSize       String                     0

    Outputs           :

每个部署通常由多个操作组成，而每个操作表示部署过程中的一个步骤。为了查明部署何处出现问题，通常需要查看部署操作的详细信息。可以用 **Get-AzureRmResourceGroupDeploymentOperation** 查看操作的状态。

    PS C:\> Get-AzureRmResourceGroupDeploymentOperation -DeploymentName ExampleDeployment -ResourceGroupName ExampleGroup
    Id                        OperationId          Properties         
    -----------               ----------           -------------
    /subscriptions/xxxxx...   347A111792B648D8     @{ProvisioningState=Failed; Timestam...
    /subscriptions/xxxxx...   699776735EFC3D15     @{ProvisioningState=Succeeded; Times...

它显示部署中的两个操作。其中一个的预配状态为“失败”，另一个的预配状态为“成功”。

你可以用以下命令获取状态信息：

    PS C:\> (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName ExampleDeployment -ResourceGroupName ExampleGroup).Properties.StatusMessage

    Code       : Conflict
    Message    : Website with given name mysite already exists.
    Target     :
    Details    : {@{Message=Website with given name mysite already exists.}, @{Code=Conflict}, @{ErrorEntity=}}
    Innererror :

## 使用 Azure CLI 进行故障排除

可以用 **azure group deployment show** 命令获取部署的总体状态。在下面的示例中，部署已失败。

    azure group deployment show ExampleGroup ExampleDeployment

    info:    Executing command group deployment show
    + Getting deployments
    data:    DeploymentName     : ExampleDeployment
    data:    ResourceGroupName  : ExampleGroup
    data:    ProvisioningState  : Failed
    data:    Timestamp          : 2015-08-27T20:03:34.9178576Z
    data:    Mode               : Incremental
    data:    Name             Type    Value
    data:    ---------------  ------  ------------
    data:    siteName         String  ExampleSite
    data:    hostingPlanName  String  ExamplePlan
    data:    siteLocation     String  China North
    data:    sku              String  Free
    data:    workerSize       String  0
    info:    group deployment show command OK


可以在审计日志中找出有关部署失败原因的更多信息。若要查看审计日志，请运行 **azure group log show** 命令。可以包含 **--last-deployment** 选项以只获取最近部署的日志。

    azure group log show ExampleGroup --last-deployment

**azure group log show** 命令可以返回很多信息。进行故障排除，你通常希望专注于失败的操作。以下脚本使用 **--json** 选项和 **jq**，在日志中搜索部署失败。若要了解有关 **jq** 等工具的信息，请参阅[与 Azure 交互的有用工具](#useful-tools-to-interact-with-azure)

    azure group log show ExampleGroup --json | jq '.[] | select(.status.value == "Failed")'

    {
      "claims": {
        "aud": "https://management.core.chinacloudapi.cn/",
        "iss": "https://sts.chinacloudapi.cn/<guid>/",
        "iat": "1442510510",
        "nbf": "1442510510",
        "exp": "1442514410",
        "ver": "1.0",
        "http://schemas.microsoft.com/identity/claims/tenantid": "<guid>",
        "http://schemas.microsoft.com/identity/claims/objectidentifier": "<guid>",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress": "someone@example.com",
        "puid": "XXXXXXXXXXXXXXXX",
        "http://schemas.microsoft.com/identity/claims/identityprovider": "example.com",

        "altsecid": "1:example.com:XXXXXXXXXXX",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "<hash string>",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "Tom",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "FitzMacken",
        "name": "Tom FitzMacken",
        "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
        "groups": "<guid>",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "example.com#someone@example.com",
        "wids": "<guid>",
        "appid": "<guid>",
        "appidacr": "0",
        "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
        "http://schemas.microsoft.com/claims/authnclassreference": "1",
        "ipaddr": "000.000.000.000"
      },
      "properties": {
        "statusCode": "Conflict",
        "statusMessage": "{"Code":"Conflict","Message":"Website with given name mysite already exists.","Target":null,"Details":[{"Message":"Website with given name
          mysite already exists."},{"Code":"Conflict"},{"ErrorEntity":{"Code":"Conflict","Message":"Website with given name mysite already exists.","ExtendedCode":
          "54001","MessageTemplate":"Website with given name {0} already exists.","Parameters":["mysite"],"InnerErrors":null}}],"Innererror":null}"
      },
    ...

你可以看到 **properties** 在 json 中包含有关失败操作的信息。

可以使用 **--verbose** 和 **-vv** 选项，在日志中查看更多信息。使用 **--verbose** 选项显示操作在 `stdout` 上进行的步骤。要查看完整的请求历史记录，请使用 **-vv** 选项。这些消息经常提供有关任何失败原因的重要线索。

## 使用 REST API 进行故障排除

资源管理器 REST API 提供若干 URI 来获取有关部署的信息、部署的操作以及有关特定操作的详细信息。有关这些命令的完整描述，请参阅：

- [获取有关模板部署的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790565.aspx)
- [列出所有模板部署操作](https://msdn.microsoft.com/zh-cn/library/azure/dn790518.aspx)
- [获取有关模板部署操作的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790519.aspx)


## 刷新过期的凭据

如果你的 Azure 凭据已过期或者尚未登录你的 Azure 帐户，则你的部署将失败。如果会话打开时间过长，则你的凭据可能会过期。你可以使用以下选项刷新你的凭据：

- 对于 PowerShell，使用 **Login-AzureRmAccount** cmdlet（PowerShell 1.0 预览版以前的版本使用 **Add-AzureAccount**）。发布设置文件中的凭据不足以满足 AzureResourceManager 模块中的 cmdlet 的需要。
- 对于 Azure CLI，使用 **azure login**。若要获取有关身份验证错误的帮助，请确保[正确配置 Azure CLI](/documentation/articles/xplat-cli-connect)。

## 检查模板和参数的格式

如果模板或参数文件的格式不正确，则你的部署会失败。在执行部署之前，可以测试模板和参数的有效性。

### PowerShell

对于 PowerShell，使用 **Test-AzureRmResourceGroupDeployment**（PowerShell 1.0 预览版以前的版本使用 **Test-AzureResourceGroupTemplate**）。

    PS C:\> Test-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile c:\Azure\Templates\azuredeploy.json -TemplateParameterFile c:\Azure\Templates\azuredeploy.parameters.json
    VERBOSE: 12:55:32 PM - Template is valid.

### Azure CLI

对于 Azure CLI，使用 **azure group template validate <resource group>**

下面的示例演示了如何验证模板及任何需要的参数。Azure CLI 提示你输入需要的参数值。

        azure group template validate \
        > --template-uri "https://contoso.com/templates/azuredeploy.json" \
        > resource-group
        info:    Executing command group template validate
        info:    Supply values for the following parameters
        adminUserName: UserName
        adminPassword: PassWord
        + Initializing template configurations and parameters
        + Validating the template
        info:    group template validate command OK

### REST API

对于 REST API，请参阅[验证模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790547.aspx)。

## 检查哪些位置支持资源

在为一个资源指定位置时，必须使用支持资源的位置之一。在输入资源的位置之前，请使用以下命令之一验证该位置是否支持此资源类型。

### PowerShell

对于 PowerShell 1.0 预览版之前的版本，可以使用 **Get-AzureLocation** 命令查看资源和位置的完整列表。

    PS C:\> Get-AzureLocation

    Name                                    Locations                               LocationsString
    ----                                    ---------                               ---------------
    ResourceGroup                           {China East, china North}				{China East, china North}
    Microsoft.AppService/apiapps            {China East, china North}				{China East, china North}
    ...

你可以用以下命令指定特定类型的资源：

    PS C:\> Get-AzureLocation | Where-Object Name -eq "Microsoft.Compute/virtualMachines" | Format-Table Name, LocationsString -Wrap

    Name                                                        LocationsString
    ----                                                        ---------------
    Microsoft.Compute/virtualMachines                           China East, China North

### Azure CLI

对于 Azure CLI，可以使用 **Azure 位置列表**。因为位置列表可能很长，而且有多个提供程序，所以你可以先使用工具来检查提供程序和位置，再使用尚未使用的位置。以下脚本使用 **jq** 发现 Azure 虚拟机资源提供程序的可用位置。

    azure location list --json | jq '.[] | select(.name == "Microsoft.Compute/virtualMachines")'
    {
      "name": "Microsoft.Compute/virtualMachines",
      "location": "China East,China North"
    }

### REST API

对于 REST API，请参阅[获取有关资源提供程序的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790534.aspx)。

## 创建唯一的资源名称

对某些资源，尤其是存储帐户、数据库服务器和网站，必须为其提供一个名称，这个名称在所有 Azure 中都是唯一的。目前，没有方法来测试某个名称是否唯一。我们建议使用其他组织不太可能使用的命名约定。

## 身份验证、订阅、角色和配额问题

可能有一个或多个问题防止部署成功，包含身份验证和授权以及 Azure Active Directory。无论你如何管理 Azure 资源组，你用于登录到你的帐户的标识必须是一个 Azure Active Directory 对象。此标识可以是你创建的或分配到你的工作或学校帐户，或者也可以为应用程序创建一个服务主体。

但是 Azure Active Directory 可让你或你的系统管理员非常精确地控制哪些标识可以访问哪些资源。如果你的部署失败，请检查请求本身是否存在身份验证或授权问题的迹象，并检查资源组的部署日志。你可能会发现，当你拥有某些资源的权限时，就没有其他资源的权限。使用 Azure CLI 可以通过 `azure ad` 命令检查 Azure Active Directory 租户和用户。（有关 Azure CLI 命令的完整列表，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理器配合使用](/documentation/articles/azure-cli-arm-commands)。）

当部署达到默认配额（可能是根据资源组、订阅、帐户和其他范围指定的）时，你也可能会遇到问题。确认你拥有可用于正确部署的资源。有关完整的配额信息，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits)。

若要检查你自己的订阅的核心配额，在 Azure CLI 中，你应使用 `azure vm list-usage` 命令，在 PowerShell 中，你应使用 **Get-AzureVMUsage** cmdlet。下面显示了 Azure CLI 中的命令，说明一个试用帐户的核心配额为 4：

    azure vm list-usage
    info:    Executing command vm list-usage
    Location: chinanorth
    data:    Name   Unit   CurrentValue  Limit
    data:    -----  -----  ------------  -----
    data:    Cores  Count  0             4
    info:    vm list-usage command OK

如果你打算尝试部署一个模板，该模板在中国北部区域的上述订阅中创建多于 4 个的核心，则你会遇到与以下错误类似的部署错误（在门户中或通过调查部署日志）：

    statusCode:Conflict
    serviceRequestId:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    statusMessage:{"error":{"code":"OperationNotAllowed","message":"Operation results in exceeding quota limits of Core. Maximum allowed: 4, Current in use: 4, Additional requested: 2."}}

在这种情况中，你应前往门户并提交一份支持问题以增加你在要部署区域内的配额。

> [AZURE.NOTE] 请记住，对于资源组，配额针对每个单独的区域，而不是针对整个订阅。如果你需要在中国北部部署 30 个核心，则必须在中国北部寻求 30 个资源管理器核心。如果你需要在你有权访问的任何区域内部署总共 30 个核心，则应在所有区域内寻求总共 30 个核心。
<!-- -->
例如，若要明确核心，你可以通过使用以下命令来检查你应为其申请相应配额的区域，该命令传递到 **jq** 以供 json 解析。
<!-- -->
        azure provider show Microsoft.Compute --json | jq '.resourceTypes[] | select(.name == "virtualMachines") | { name,apiVersions, locations}'
        {
          "name": "virtualMachines",
          "apiVersions": [
            "2015-05-01-preview",
            "2014-12-01-preview"
          ],
          "locations": [
            "China East",
            "China North"
          ]
        }


## 检查资源提供程序注册

资源由资源提供程序管理，你可以启用帐户或订阅来使用特定的提供程序。如果你可以使用某个提供程序，则还必须注册该提供程序才能使用。Azure 管理门户或正在使用的命令行接口会自动注册大多数提供程序；但非全部。

### PowerShell

若要获取资源提供程序和你的注册状态的列表，对于 PowerShell 1.0 预览版以前的版本，请使用 **Get-AzureProvider**。

    PS C:\> Get-AzureProvider

    ProviderNamespace                       RegistrationState                       ResourceTypes
    -----------------                       -----------------                       -------------
    Microsoft.AppService                    Registered                              {apiapps, appIdentities, gateways, d...
    Microsoft.Batch                         Registered                              {batchAccounts}
    microsoft.cache                         Registered                              {Redis, checkNameAvailability, opera...
    ...

若要注册一个提供程序，请使用 **Register-AzureProvider**。

对于 Powershell 1.0 预览版，请使用 **Get-AzureRmResourceProvider**。

    PS C:\> Get-AzureRmResourceProvider -ListAvailable

	ProviderNamespace         RegistrationState ResourceTypes                                                                               Locations
	-----------------         ----------------- -------------                                                                               ---------
	microsoft.backup          Registering       {BackupVault}                                                                               {China East, China North}
	Microsoft.Batch           Registered        {batchAccounts, locations, locations/quotas}                                                {China North, China East}
	microsoft.cache           Registered        {Redis, checkNameAvailability, operations, RedisConfigDefinition...}                        {China North, China East}
	...

若要注册一个提供程序，请使用 **Register-AzureRmResourceProvider**。

### Azure CLI

若要使用 Azure CLI 查看是否已注册提供程序供使用，请使用 `azure provider list` 命令（以下是截断的输出示例）。

        azure provider list
		info:    Executing command provider list
		+ Getting ARM registered providers
		data:    Namespace                  Registered
		data:    -------------------------  -----------
		data:    microsoft.backup           Registering
		data:    Microsoft.Batch            Registered
		data:    microsoft.cache            Registered
		data:    Microsoft.ClassicCompute   Registered
		data:    Microsoft.ClassicNetwork   Registered
		data:    Microsoft.ClassicStorage   Registered
		data:    Microsoft.Insights         Registered
		data:    Microsoft.KeyVault         Registered
		data:    Microsoft.SiteRecovery     Registered
		data:    Microsoft.Sql              Registered
		data:    Microsoft.StreamAnalytics  Registered
		data:    Microsoft.Web              Registered
		data:    Microsoft.Authorization    Registered
		data:    Microsoft.Features         Registered
		data:    Microsoft.Resources        Registered
		data:    Microsoft.Scheduler        Registered
		info:    provider list command OK

再次指出，如果你需要有关提供程序的更多信息，包括它们的区域可用性，请键入 `azure provider list --json`。以下仅选择列表中的第一个来查看：

        azure provider list --json | jq '.[0]'
        {
          "resourceTypes": [
            {
              "apiVersions": [
                "2014-02-14"
              ],
              "locations": [
                "China East",
                "China North"
              ],
              "properties": {},
              "name": "service"
            }
          ],
          "id": "/subscriptions/<guid>/providers/Microsoft.ApiManagement",
          "namespace": "Microsoft.ApiManagement",
          "registrationState": "Registered"
        }

如果提供程序需要注册，请使用 `azure provider register <namespace>` 命令，其中 *namespace* 值来自前面的列表。

### REST API

若要获取注册状态，请参阅[获取有关资源提供程序的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790534.aspx)。

若要注册提供程序，请参阅[使用资源提供程序注册订阅](https://msdn.microsoft.com/zh-cn/library/azure/dn790548.aspx)。


## 了解自定义模板的部署何时成功

如果你正在使用自己创建的模板，请务必了解 Azure 资源管理器系统会在成功从部署返回所有提供程序时报告部署成功。这意味着已部署所有模板项供你使用。

不过请注意，这不一定表示你的资源组“处于活动状态且可供用户使用”。例如，大多数部署会请求部署下载升级文件、等待其他非模板资源，或者安装复杂的脚本或 Azure 不知道的其他某个可执行活动（因为它不是提供程序正在跟踪的活动）。在这些情况下，可能要在一段时间后，你的资源才可供实际使用。因此，你应该预料到部署状态成功一段时间后，才能使用部署。

不过，创建自定义模板的自定义脚本（例如，使用 [CustomScriptExtension](http://azure.microsoft.com/blog/2014/08/20/automate-linux-vm-customization-tasks-using-customscript-extension/)），即可防止 Azure 报告部署成功，而此自定义脚本知道如何监视整个部署以了解系统是否准备就绪，并只在用户可以与整个部署交互时才会成功返回。如果你想要确保最后才运行你的扩展，请在模板中使用 **dependsOn** 属性。[创建模板部署](https://msdn.microsoft.com/zh-cn/library/azure/dn790564.aspx)时可以查看一个示例。

## 与 Azure 交互的有用工具
当你从命令行处理你的 Azure 资源时，你需要收集帮助你完成工作的工具。Azure 资源组模板为 JSON 文件，而 Azure 资源管理器 API 接受并返回 JSON，因此 JSON 解析工具是帮助你浏览资源的相关信息，以及设计模板和模板参数文件或与之交互的首选工具。

### Mac、Linux 和 Windows 工具
如果使用适用于 Mac、Linux 和 Windows 的 Azure 命令行接口，则可能你熟悉标准下载工具，例如 **[curl](http://curl.haxx.se/)** 和 **[wget](https://www.gnu.org/software/wget/)** 或 **[Resty](https://github.com/beders/Resty)**）和 JSON 实用工具，例如 **[jq](http://stedolan.github.io/jq/download/)**、**[jsawk](https://github.com/micha/jsawk)**，以及处理 JSON 的语言库。（其中的许多工具也有针对 Windows 的端口，例如 [wget](http://gnuwin32.sourceforge.net/packages/wget.htm)；事实上，还可以通过多种方法获取 Linux 以及在 Windows 上运行的其他开源软件工具）。

本主题包括一些你可以与 **jq** 配合使用的 Azure CLI 命令，以更有效地准确获取所需的信息。你应该选择熟悉的工具集，以帮助了解 Azure 资源的使用情况。

### PowerShell

PowerShell 提供了一些基本命令用于执行相同的过程。

- 使用 **[Invoke-WebRequest cmdlet](https://technet.microsoft.com/zh-cn/library/hh849901%28v=wps.640%29)** 可下载资源组模板或参数 JSON 文件等文件。
- 使用 **[ConvertFrom-Json](https://technet.microsoft.com/zh-cn/library/hh849898%28v=wps.640%29.aspx)** cmdlet 可将 JSON 字符串转换成自定义对象 ([PSCustomObject](https://msdn.microsoft.com/zh-cn/library/windows/desktop/system.management.automation.pscustomobject%28v=vs.85%29.aspx))，该对象具有 JSON 字符串中每个字段的属性。


## 后续步骤

若要掌握模板的创建，请阅读[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)，并浏览 [Azure 快速启动模板存储库](https://github.com/Azure/azure-quickstart-templates)以了解可部署的示例。[使用多个 NIC 和可访问的 RDP 创建虚拟机](https://github.com/Azure/azure-quickstart-templates/tree/master/201-1-vm-loadbalancer-2-nics)是 **dependsOn** 属性的一个例子。

<!--Image references-->

<!--Reference style links - using these makes the source content way more readable than using inline links-->

<!---HONumber=Mooncake_0215_2016-->