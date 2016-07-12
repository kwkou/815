<!-- Remove arm-troubleshoot-portal -->
<properties
   pageTitle="常见的 Azure 部署错误 | Azure"
   description="介绍如何解决使用 Azure Resource Manager 部署时遇到的常见错误。"
   services="azure-resource-manager"
   documentationCenter=""
   tags=""
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.date="04/19/2016"
   wacn.date="06/20/2016"/>

# 解决使用 Azure Resource Manager 将资源部署到 Azure 时的常见错误

本主题介绍如何解决将资源部署到 Azure 时可能遇到的一些常见错误。<!-- 有关排查部署问题的信息，请参阅 [Troubleshooting resource group deployments（对资源组部署进行故障排除）](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)。-->

如果部署之前先验证模板和参数，则可以避免一些错误。有关验证模板的示例，请参阅 [Deploy resources with Azure Resource Manager template（使用 Azure Resource Manager 模板部署资源）](/documentation/articles/resource-group-template-deploy/)。

## 无效的模板或资源

如果收到的错误指出模板或资源的属性无效，则你的模板可能缺少字符。使用模板表达式时容易出此错误，因为该表达式括在引号中，因此 JSON 仍然进行验证，而你的编辑器可能未检测到此错误。例如，存储帐户的以下名称分配包含一组方括号、三个函数、三组圆括号、一组单引号和一个属性：

    "name": "[concat('storage', uniqueString(resourceGroup().id))]",

如果未提供所有匹配的语法，该模板将生成一个与你的意图有很大差异的值。

你将收到一个错误，指出模板或资源无效，具体取决于缺少的字符在模板中的位置。该错误也可能会指出部署过程无法处理模板语言表达式。当你收到此类错误时，请仔细检查表达式语法。

## 资源名称已存在

对某些资源，尤其是存储帐户、数据库服务器和网站，必须为其提供一个名称，这个名称在所有 Azure 中都是唯一的。可将命名约定与 [uniqueString](/documentation/articles/resource-group-template-functions/#uniquestring) 函数的结果连接起来创建一个唯一名称。
 
    "name": "[concat('contosostorage', uniqueString(resourceGroup().id))]", 
    "type": "Microsoft.Storage/storageAccounts", 

## 部署期间找不到资源

如果可能，Resource Manager 将通过并行创建资源来优化部署。如果一个资源必须在另一个资源之后部署，则你需要在模板中使用 **dependsOn** 元素创建与其他资源的依赖关系。例如，在部署 Web 应用时，App Service 计划必须存在。如果未指定该 Web 应用与 App Service 计划的依赖关系，Resource Manager 将同时创建这两个资源。你将收到一条错误消息，指出找不到 App Service 计划资源，因为尝试在 Web 应用上设置属性时它尚不存在。在 Web 应用中设置依赖关系可避免此错误。

    {
      "apiVersion": "2015-08-01",
      "type": "Microsoft.Web/sites",
      ...
      "dependsOn": [
        "[variables('hostingPlanName')]"
      ],
      ...
    }

## 资源的位置不可用

在为一个资源指定位置时，必须使用支持资源的位置之一。在输入资源的位置之前，请使用以下命令之一验证该位置是否支持此资源类型。

### PowerShell

使用 **Get-AzureRmResourceProvider** 获取特定资源提供程序支持的类型和位置。

    Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web

你将获到该资源提供程序的资源类型列表。

    ProviderNamespace RegistrationState ResourceTypes               Locations
    ----------------- ----------------- -------------               ---------
    Microsoft.Web     Registered        {sites/extensions}          {China East, ...
    Microsoft.Web     Registered        {sites/slots/extensions}    {China East, ...
    Microsoft.Web     Registered        {sites/instances}           {China East, ...
    ...

可以使用以下命令来专注于特定类型的资源：

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations

这将返回支持的位置：

    China East
    China North



### Azure CLI

对于 Azure CLI，可以使用 **Azure 位置列表**。因为位置列表可能很长，而且有多个提供程序，所以你可以先使用工具来检查提供程序和位置，再使用尚未使用的位置。以下脚本使用 **jq** 发现 Azure 虚拟机资源提供程序的可用位置。

    azure location list --json | jq '.[] | select(.name == "Microsoft.Compute/virtualMachines")'
    
这将返回支持的位置：
    
    {
      "name": "Microsoft.Compute/virtualMachines",
      "location": "East US,East US 2,West US,Central US,South Central US,North Europe,West Europe,East Asia,Southeast Asia,Japan East,Japan West"
    }

### REST API

对于 REST API，请参阅[获取有关资源提供程序的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790534.aspx)。

## 超出配额

当部署超出配额（可能是根据资源组、订阅、帐户和其他范围指定的）时，你可能会遇到问题。例如，订阅可能配置为限制某个区域的核心数目。如果你尝试部署超过允许核心数目的虚拟机，将收到指出超过配额的错误消息。有关完整的配额信息，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)。

若要检查订阅的核心配额，可以使用 Azure CLI 中的 `azure vm list-usage` 命令。以下示例演示了核心配额为 4 的免费试用帐户：

    azure vm list-usage
    
将返回：
    
    info:    Executing command vm list-usage
    Location: westus
    data:    Name   Unit   CurrentValue  Limit
    data:    -----  -----  ------------  -----
    data:    Cores  Count  0             4
    info:    vm list-usage command OK

如果你打算尝试部署一个模板，该模板在美国西部地区的上述订阅中创建多于 4 个的核心，则你会遇到与以下错误类似的部署错误（在门户中或通过调查部署日志）：

    statusCode:Conflict
    serviceRequestId:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    statusMessage:{"error":{"code":"OperationNotAllowed","message":"Operation results in exceeding quota limits of Core. Maximum allowed: 4, Current in use: 4, Additional requested: 2."}}

或者在 PowerShell 中，可以使用 **Get-AzureRmVMUsage** cmdlet。

    Get-AzureRmVMUsage
    
将返回：

    ...
    CurrentValue : 0
    Limit        : 4
    Name         : {
                     "value": "cores",
                     "localizedValue": "Total Regional Cores"
                   }
    Unit         : null
    ...

在这种情况中，你应前往门户并提交一份支持问题以增加你在要部署区域内的配额。

> [AZURE.NOTE] 请记住，对于资源组，配额针对每个单独的区域，而不是针对整个订阅。如果你需要在美国西部部署 30 个核心，则必须在美国西部寻求 30 个资源管理器核心。如果你需要在你有权访问的任何区域内部署总共 30 个核心，则应在所有区域内寻求总共 30 个核心。


## 授权失败

你可能在部署期间收到错误，因为尝试部署资源的帐户或服务主体没有执行这些操作的访问权限。Azure Active Directory 可让你或你的系统管理员非常精确地控制哪些标识可以访问哪些资源。例如，如果帐户分配有“读取者”角色，它将无法创建新资源。在此情况下，你应会看到错误消息，指出授权失败。

除了基于角色的访问控制以外，部署操作可能还受限于订阅的策略。通过策略，管理员可以对订阅中所有资源部署强制实施约定。例如，管理员可以要求为某个资源类型提供特定的标记值。如果不满足策略要求，你将在部署期间收到错误。

## 检查资源提供程序注册

资源由资源提供程序管理，你必须注册帐户或订阅才能使用特定的提供程序。Azure 门户或正在使用的命令行接口会自动注册大多数提供程序；但非全部。

### PowerShell

若要查看注册状态，请使用 **Get-AzureRmResourceProvider**。

    Get-AzureRmResourceProvider -ListAvailable

这将返回所有可用的资源提供程序和注册状态：

    ProviderNamespace               RegistrationState ResourceTypes
    -----------------               ----------------- -------------
    Microsoft.ApiManagement         Unregistered      {service, validateServiceName, checkServiceNameAvailability}
    Microsoft.AppService            Registered        {apiapps, appIdentities, gateways, deploymenttemplates...}
    Microsoft.Batch                 Registered        {batchAccounts}

若要注册提供程序，请使用 **Register-AzureRmResourceProvider**，并提供想要注册的资源提供程序的名称。

    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Cdn

系统将请求你确认注册，然后返回状态。

    Confirm
    Are you sure you want to register the provider 'Microsoft.Cdn'
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y

    ProviderNamespace RegistrationState ResourceTypes
    ----------------- ----------------- -------------
    Microsoft.Cdn     Registering       {profiles, profiles/endpoints,...

### Azure CLI

若要使用 Azure CLI 查看是否已注册提供程序供使用，请使用 `azure provider list` 命令（以下是截断的输出示例）。

    azure provider list
        
这将返回所有可用的资源提供程序和注册状态：
        
    info:    Executing command provider list
    + Getting ARM registered providers
    data:    Namespace                        Registered
    data:    -------------------------------  -------------
    data:    Microsoft.Compute                Registered
    data:    Microsoft.Network                Registered  
    data:    Microsoft.Storage                Registered
    data:    microsoft.visualstudio           Registered
    ...
    info:    provider list command OK

若要注册资源提供程序，使用 `azure provider register` 命令，并指定要注册的*命名空间*。

    azure provider register Microsoft.Cdn

### REST API

若要获取注册状态，请参阅[获取有关资源提供程序的信息](https://msdn.microsoft.com/zh-cn/library/azure/dn790534.aspx)。

若要注册提供程序，请参阅[使用资源提供程序注册订阅](https://msdn.microsoft.com/zh-cn/library/azure/dn790548.aspx)。

## 自定义脚本扩展错误

如果在部署虚拟机时遇到自定义脚本扩展错误，请参阅 [Troubleshooting Azure Windows VM extension failures（排查 Azure Windows VM 扩展失败错误）](/documentation/articles/virtual-machines-windows-extensions-troubleshoot/)或 [Troubleshooting Azure Linux VM extension failures（排查 Azure Linux VM 扩展失败错误）](/documentation/articles/virtual-machines-linux-extensions-troubleshoot/)。

## 了解部署何时准备就绪 

Azure Resource Manager 部署成功返回所有提供程序时，将报告部署成功。但是，这不一定表示你的资源组“处于活动状态且可供用户使用”。例如，部署可能需要下载升级文件、等待其他非模板资源，或者安装复杂的脚本或 Azure 不知道的其他某个可执行活动（因为它不是提供程序正在跟踪的活动）。在这些情况下，可能要在一段时间后，你的资源才可供实际使用。因此，你应该预料到部署状态成功一段时间后，才能使用部署。

不过，创建自定义模板的自定义脚本（例如，使用 [CustomScriptExtension](https://azure.microsoft.com/blog/2014/08/20/automate-linux-vm-customization-tasks-using-customscript-extension/)），即可防止 Azure 报告部署成功，而此自定义脚本知道如何监视整个部署以了解系统是否准备就绪，并只在用户可以与整个部署交互时才会成功返回。如果你想要确保最后才运行你的扩展，请在模板中使用 **dependsOn** 属性。

## 后续步骤

- 若要了解审核操作，请参阅 [Audit operations with Resource Manager（使用 Resource Manager 执行审核操作）](/documentation/articles/resource-group-audit/)。
- 若要了解部署期间为确定错误执行哪些操作，请参阅 [Troubleshooting resource group deployments（对资源组部署进行故障排除）](/documentation/articles/resource-manager-troubleshoot-deployments-portal/)。

<!---HONumber=Mooncake_0620_2016-->