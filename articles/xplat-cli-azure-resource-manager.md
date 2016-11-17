
<properties
	pageTitle="使用 Azure CLI 管理资源 | Azure"
	description="使用 Azure 命令行接口 (CLI) 管理 Azure 资源和组"
	editor=""
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services="azure-resource-manager"/>

<tags
	ms.service="azure-resource-manager"
	ms.workload="multiple"
	ms.tgt_pltfrm="vm-multiple"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/22/2016"
	wacn.date="10/10/2016"
	ms.author="danlep"/>

# 使用 Azure CLI 管理 Azure 资源和资源组


> [AZURE.SELECTOR]
- [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager/)
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager/)
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager/)
- [REST API](/documentation/articles/resource-manager-rest-api/)


Azure 命令行接口 (Azure CLI) 是可以配合 Resource Manager 部署和管理资源的多种工具之一。本文介绍在 Resource Manager 模式下使用 Azure CLI 管理 Azure 资源和资源组的常见方式。有关使用 CLI 部署资源的信息，请参阅 [Deploy resources with Resource Manager templates and Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)（使用 Resource Manager 模板和 Azure CLI 部署资源）。有关 Azure 资源和 Resource Manager 的背景信息，请参阅 [Azure Resource Manager Overview](/documentation/articles/resource-group-overview/)（Azure Resource Manager 概述）。

>[AZURE.NOTE] 若要使用 Azure CLI 管理 Azure 资源，需要[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 并使用 `azure login -e AzureChinaCloud` 命令[登录 Azure](/documentation/articles/xplat-cli-connect/)。请确保 CLI 处于 Resource Manager 模式（运行 `azure config mode arm`）。如果已做好了这些准备，你便可以开始了。



## 获取资源组和资源

### 资源组

若要获取订阅中所有资源组的列表及其位置，请运行以下命令。

    azure group list
    

### 资源
 若要列出组中的所有资源，例如名为 *testRG* 的资源，请使用以下命令。

	azure resource list testRG

若要查看组中的单个资源，例如名为 *MyUbuntuVM* 的 VM，请使用如下命令。

	azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15"
    
注意 **Microsoft.Compute/virtualMachines** 参数。此参数表示要请求其信息的资源的类型。
    
>[AZURE.NOTE]使用除 **list** 命令以外的 **azure resource** 命令时，必须使用 **-o** 参数指定资源的 API 版本。如果不确定要使用哪个 API 版本，请查阅模板文件并查找资源的 apiVersion 字段。有关 Resource Manager 中 API 版本的详细信息，请参阅 [Resource Manager providers, regions, API versions, and schemas](/documentation/articles/resource-manager-supported-services/)（Resource Manager 提供程序、区域、API 版本和架构）。

在资源上查看详细信息时，通常使用 `--json` 参数会很有用。此参数使输出更易于阅读，因为某些值为嵌套的结构或集合。以下示例演示了将 **show** 命令的结果返回为 JSON 文档。

	azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15" --json

>[AZURE.NOTE] 可以将 JSON 数据保存到文件，使用 &gt; 字符将输出定向到文件即可。例如：
>
> `azure resource show testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15" --json > myfile.json`

### 标记

为了帮助组织资源，可将[标记](/documentation/articles/resource-group-using-tags/)添加到资源和资源组。若要查看已应用哪些标记，只需使用 **azure group show** 获取资源组及其资源即可。

    azure group show -n tag-demo-group
    
此命令返回有关资源组的元数据，包括任何应用到其中的标记。
    
    info:    Executing command group show
    + Listing resource groups
    + Listing resources for the group
    data:    Id:                  /subscriptions/{guid}/resourceGroups/tag-demo-group
    data:    Name:                tag-demo-group
    data:    Location:            chinaeast
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production
    data:    Resources:
    data:
    data:      Id      : /subscriptions/{guid}/resourceGroups/tag-demo-group/providers/Microsoft.Sql/servers/tfsqlserver
    data:      Name    : tfsqlserver
    data:      Type    : servers
    data:      Location: eastus2
    data:      Tags    : Dept=Finance;Environment=Production
    ...

若要仅获取资源组的标记，请使用 JSON 实用工具，例如 [jq](http://stedolan.github.io/jq/download/)。

    azure group show -n tag-demo-group --json | jq ".tags"
    
此命令返回该资源组的标记。
    
    {
      "Dept": "Finance",
      "Environment": "Production" 
    }

使用 **azure resource show** 查看特定资源的标记。

    azure resource show -g tag-demo-group -n tfsqlserver -r Microsoft.Sql/servers -o 2014-04-01-preview --json | jq ".tags"
    
此命令返回以下信息。
    
    {
      "Dept": "Finance",
      "Environment": "Production"
    }
    
使用如下所示的命令检索具有特定标记的所有资源。

    azure resource list --json | jq ".[] | select(.tags.Dept == "Finance") | .name"
    
此命令返回具有该标记的资源的名称。
    
    "tfsqlserver"
    "tfsqlserver/tfsqldata"

标记作为一个整体进行更新，因此，如果向已标记的资源添加一个标记，需要检索要保留的现有标记。若要为资源组设置标记值，请使用 **azure group set** 并提供该资源组的所有标记。

    azure group set -n tag-demo-group -t Dept=Finance;Environment=Production;Project=Upgrade
    
将返回带新标记的资源组的摘要。
    
    info:    Executing command group set
    ...
    data:    Name:                tag-demo-group
    data:    Location:            chinaeast
    data:    Provisioning State:  Succeeded
    data:    Tags: Dept=Finance;Environment=Production;Project=Upgrade
    ...
    
可以使用 **azure tag list** 列出订阅中的现有标记，使用 **azure tag create** 添加标记。若要从订阅的分类中删除某个标记，请先从它已应用到的所有资源中将它删除，然后使用 **azure tag delete** 删除它。

## 管理资源


若要将存储帐户等资源添加到资源组，请运行如下所示的命令：

    azure resource create testRG MyStorageAccount "Microsoft.Storage/storageAccounts" "chinaeast" -o "2015-06-15" -p "{"accountType": "Standard_LRS"}"
    
除了使用 **-o** 参数指定资源的 API 版本以外，请使用 **-p** 参数传递包含任何必需或其他属性的 JSON 格式字符串。
    
    
若要删除现有资源（例如虚拟机资源），请使用如下命令。

	azure resource delete testRG MyUbuntuVM Microsoft.Compute/virtualMachines -o "2015-06-15"

若要将现有资源移到另一个资源组或订阅，请使用 **azure resource move** 命令。以下示例演示如何将一个 Redis 缓存移到新的资源组。在 **-i** 参数中，提供要移动的资源 ID 的逗号分隔列表。


    azure resource move -i "/subscriptions/{guid}/resourceGroups/OldRG/providers/Microsoft.Cache/Redis/examplecache" -d "NewRG"

## 控制对资源的访问

可以使用 Azure CLI 来创建和管理策略，控制对 Azure 资源的访问。有关策略定义以及将策略分配给资源的背景信息，请参阅 [Use policy to manage resources and control access](/documentation/articles/resource-manager-policy/)（使用策略来管理资源和控制访问）。

例如，定义以下策略来拒绝所有位置不在中国东部或中国北部的请求，并将该策略保存到策略定义文件 policy.json 中：

    {
    "if" : {
        "not" : {
        "field" : "location",
        "in" : ["chinaeast" ,  "chinanorth"]
        }
    },
    "then" : {
        "effect" : "deny"
    }
    }

然后运行 **policy definition create** 命令：

    azure policy definition create MyPolicy -p c:\temp\policy.json
    
此命令显示如下所示的输出。

    + Creating policy definition MyPolicy
    data:    PolicyName:             MyPolicy
    data:    PolicyDefinitionId:     /subscriptions/########-####-####-####-############/providers/Microsoft.Authorization/policyDefinitions/MyPolicy

    data:    PolicyType:             Custom
    data:    DisplayName:            undefined
    data:    Description:            undefined
    data:    PolicyRule:             field=location, in=[chinaeast, chinanorth], effect=deny

 若要在所需的范围内分配策略，请使用前一命令返回的 **PolicyDefinitionId**。在以下示例中，此范围是订阅，但可以将范围设置为资源组或单个资源：

    azure policy assignment create MyPolicyAssignment -p /subscriptions/########-####-####-####-############/providers/Microsoft.Authorization/policyDefinitions/MyPolicy -s /subscriptions/########-####-####-####-############/

可以使用 **policy definition show**、**policy definition set** 和 **policy definition delete** 命令来获取、更改或删除策略定义。

同样地，可以使用 **policy assignment show**、**policy assignment set** 和 **policy assignment delete** 命令来获取、更改或删除策略分配。


## 将资源组导出为模板

对于现有资源组，你可以查看资源组的 Resource Manager 模板。导出模板有两个好处：

1. 由于模板中定义了所有基础结构，因此将来可以轻松地自动完成解决方案的部署。

2. 可以查看代表解决方案的 JSON，以此熟悉模板语法。

使用 Azure CLI，可以导出表示资源组当前状态的模板，或下载特定部署所用的模板。

* **导出资源组的模板** - 已更改资源组并需要检索其当前状态的 JSON 表示法时，此操作很有用。但是，生成的模板只包含最少的参数数目，但不包含任何变量。模板中大部分的值为硬编码。在部署所生成的模板之前，你可能想要将更多的值转换成参数，以便针对不同的环境自定义部署。

    若要将资源组模板导出到本地目录中，请运行 `azure group export` 命令，如以下示例所示。（替换为适用于你操作系统环境的本地目录。）

        azure group export testRG ~/azure/templates/

* **针对特定部署导出模板** - 需要查看用于部署资源的实际模板时，此操作很有用。模板包含针对原始部署定义的所有参数和变量。但是，如果组织中的某人更改了不在模板定义中的资源组，此模板不会显示资源组的当前状态。

    若要将用于特定部署的模板下载到本地目录，请运行 `azure group deployment template download` 命令。例如：

        azure group deployment template download TestRG testRGDeploy ~/azure/templates/downloads/
 
>[AZURE.NOTE] 模板导出功能处于预览状态，并非所有的资源类型目前都支持导出模板。尝试导出模板时，你可能会看到一个错误，指出未导出某些资源。如果需要，可以在下载模板之后，在模板中手动定义这些资源。



## 后续步骤

* 若要获取部署操作的详细信息并使用 Azure CLI 排查部署错误，请参阅 [View deployment operations with Azure CLI](/documentation/articles/resource-manager-troubleshoot-deployments-cli/)（使用 Azure CLI 查看部署操作）。
* 若要使用 CLI 设置一个应用程序或脚本来访问资源，请参阅 [Use Azure CLI to create a service principal to access resources](/documentation/articles/resource-group-authenticate-service-principal-cli/)（使用 Azure CLI 创建服务主体来访问资源）。

<!---HONumber=Mooncake_0926_2016-->