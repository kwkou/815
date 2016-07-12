<properties
	pageTitle="DocumentDB 自动化 — 资源管理器 — CLI | Azure"
	description="使用 Azure Resource Manager 模板或 CLI 来部署 DocumentDB 数据库帐户。DocumentDB 是用于 JSON 数据的云端 NoSQL 数据库。"
	services="documentdb"
	authors="mimig1"
	manager="jhubbard"
	editor=""
    tags="azure-resource-manager"
	documentationCenter=""/>


<tags 
	ms.service="documentdb" 
	ms.date="03/04/2016" 
	wacn.date="07/01/2016"/>

# 使用 Azure Resource Manager 模板和 Azure CLI 自动创建 DocumentDB 帐户

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/documentdb-create-account/)
- [Azure CLI 和 ARM](/documentation/articles/documentdb-automation-resource-manager-cli/)

本文说明如何使用 Azure Resource Manager 模板或 Azure 命令行界面 (CLI) 来创建 DocumentDB 帐户。若要使用 Azure 门户预览创建 DocumentDB 帐户，请参阅[使用 Azure 门户预览创建 DocumentDB 数据库帐户](/documentation/articles/documentdb-create-account/)。

- [使用 CLI 创建 DocumentDB 帐户](#quick-create-documentdb-account)
- [使用 ARM 模板创建 DocumentDB 帐户](#deploy-documentdb-from-a-template)

DocumentDB 数据库帐户是目前唯一可以使用 ARM 模板和 Azure CLI 创建的 DocumentDB 资源。

## 做好准备

必须拥有正确的 Azure CLI 版本和 Azure 帐户，才能将 Azure CLI 与 Azure 资源组配合使用。如果没有 Azure CLI，[请安装](/documentation/articles/xplat-cli-install/)。

### 更新 Azure CLI 版本

在命令提示符处，键入 `azure --version` 即可查看安装的是否为版本 0.9.11 或更高版本。

	azure --version
    0.9.11 (node: 0.12.7)

如果你的版本不是 0.9.11 或更高版本，则需要使用某个本机安装程序[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 或进行更新，或者通过在 **npm** 中键入 `npm update -g azure-cli` 进行更新或键入 `npm install -g azure-cli` 进行安装。

### 设置你的 Azure 帐户和订阅

如果你还没有 Azure 订阅，请注册获取[试用版](https://www.azure.cn/pricing/1rmb-trial/)。

需有一个工作或学校帐户或者一个 Microsoft 帐户标识，才能使用 Azure 资源管理模板。如果你有其中一个帐户，请键入以下命令。

	azure login

这将生成以下输出：

    info:    Executing command login
    |info:    To sign in, use a web browser to open the page https://aka.ms/devicelogin. 
    Enter the code E1A2B3C4D to authenticate. If you're signing in as an Azure
    AD application, use the --username and --password parameters.

> [AZURE.NOTE] 如果你没有 Azure 帐户，则会看到一条错误消息，指出你需要不同类型的帐户。若要从当前 Azure 帐户创建一个帐户，请参阅[在 Azure Active Directory 中创建工作或学校标识](/documentation/articles/virtual-machines-windows-create-aad-work-id/)。

在浏览器中打开 [https://aka.ms/devicelogin](https://aka.ms/devicelogin)，然后输入命令输出中提供的代码。

![屏幕截图：显示 Azure CLI 的设备登录屏幕](./media/documentdb-automation-resource-manager-cli/azure-cli-login-code.png)

输入代码后，便可选择想要在浏览器中使用的标识，并根据需要提供用户名和密码。

![屏幕截图：显示可在其中选择与要使用的 Azure 订阅关联的 Microsoft 标识帐户的位置](./media/documentdb-automation-resource-manager-cli/identity-cli-login.png)

成功登录之后，将会收到下列确认屏幕，接着就能关闭浏览器窗口。

![屏幕截图：显示确认登录 Azure 跨平台命令行界面](./media/documentdb-automation-resource-manager-cli/login-confirmation.png)

命令行界面还会提供下列输出。

    /info:    Added subscription Visual Studio Ultimate with MSDN
    info:    Setting subscription "Visual Studio Ultimate with MSDN" as default
    +
    info:    login command OKK

除了此处所述的交互式登录方法之外，还有一些其他的 Azure CLI 登录方法可供使用。有关其他方法的详细信息以及处理多个订阅的相关信息，请参阅[从 Azure 命令行界面 (Azure CLI) 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

### 切换到 Azure CLI 资源组模式

默认情况下，Azure CLI 在服务管理模式下启动（**asm** 模式）。键入以下内容，切换到资源组模式。

	azure config mode arm

这将提供以下输出：

    info:    Executing command config mode
    info:    New mode is arm
    info:    config mode command OK

键入 `azure config mode asm` 可切换回到默认的命令集。

## <a id="quick-create-documentdb-account"></a>任务：使用 Azure CLI 创建 DocumentDB 帐户

遵循本节中的说明，使用 Azure CLI 来创建 DocumentDB 帐户。

### 步骤 1：创建或检索资源组

若要创建 DocumentDB 帐户，首先需要一个资源组。如果你已经知道想要使用的资源组名称，则跳到[步骤 2](#create-documentdb-account-cli)。

若要查看列有你当前所有的资源组的列表，请运行下列命令，并记下你想要使用的资源组名称：

    azure group list

若要创建新的资源组，请运行下列命令，并指定要创建的新资源组的名称，以及要在其中创建资源组的区域：

	azure group create <resourcegroupname> <resourcegrouplocation>

 - `<resourcegroupname>` 只能使用字母数字字符、句点、下划线、“-”字符和括号，且不能以句点结尾。 
 - `<resourcegrouplocation>` 必须是已正式推出 DocumentDB 的区域之一。[Azure 区域页面](https://azure.microsoft.com/regions/#services)会提供当前的区域列表。

输入示例：

	azure group create new_res_group westus

这将生成以下输出：

    info:    Executing command group create
    + Getting resource group new_res_group
    + Creating resource group new_res_group
    info:    Created resource group new_res_group
    data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/new_res_group
    data:    Name:                new_res_group
    data:    Location:            westus
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK

如果遇到错误，请参阅[故障排除](#troubleshooting)。

### <a id="create-documentdb-account-cli"></a>步骤 2：使用 CLI 创建 DocumentDB 帐户

在命令提示符处输入下列命令，于新的或现有的资源组中创建 DocumentDB 帐户：

> [AZURE.TIP] 如果在 Azure PowerShell 或 Windows PowerShell 中运行此命令，将收到关于意外的令牌的错误。请改为在 Windows 命令提示符处运行此命令。

    azure resource create -g <resourcegroupname> -n <databaseaccountname> -r "Microsoft.DocumentDB/databaseAccounts" -o "2015-04-08" -l <databaseaccountlocation> -p "{"databaseAccountOfferType":"Standard"}" 

 - `<resourcegroupname>` 只能使用字母数字字符、句点、下划线、“-”字符和括号，且不能以句点结尾。 
 - `<databaseaccountname>` 只能使用小写字母、数字及“-”字符，且长度必须为 3 到 50 个字符。
 - `<databaseaccountlocation>` 必须是已正式推出 DocumentDB 的区域之一。[Azure 区域页面](https://azure.microsoft.com/regions/#services)会提供当前的区域列表。

输入示例：

    azure resource create -g new_res_group -n samplecliacct -r "Microsoft.DocumentDB/databaseAccounts" -o 2015-04-08  -l westus -p "{"databaseAccountOfferType":"Standard"}"

若已预配新的帐户，这将生成以下输出：

    info:    Executing command resource create
    + Getting resource samplecliacct
    + Creating resource samplecliacct
    info:    Resource samplecliacct is updated
    data:
    data:    Id:        /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/new_res_group/providers/Microsoft.DocumentDB/databaseAccounts/samplecliacct
    data:    Name:      samplecliacct
    data:    Type:      Microsoft.DocumentDB/databaseAccounts
    data:    Parent:
    data:    Location:  West US
    data:    Tags:
    data:
    info:    resource create command OK

如果遇到错误，请参阅[故障排除](#troubleshooting)。

在此命令返回之后，在帐户更改为**在线**状态以准备好可供使用之前，该帐户将会进入**正在创建**状态数分钟的时间。你可以在 [Azure 门户预览](https://portal.azure.cn)中的“DocumentDB 帐户”边栏选项卡上检查帐户的状态。

## <a id="deploy-documentdb-from-a-template"></a>任务：使用 ARM 模板创建 DocumentDB 帐户

遵循本节中的说明，使用 Azure Resource Manager (ARM) 模板和可选参数文件（这两者都是 JSON 文件）来创建 DocumentDB 帐户。使用模板可以准确描述所需的信息，并可重复使用而不会出现任何错误。

### 了解 ARM 模板和资源组

大多数应用程序是通过不同资源类型的组合（例如，一个或多个 DocumentDB 帐户或存储帐户、一个虚拟网络或内容传送网络）构建而成的。默认 Azure 服务管理 API 和 Azure 门户预览使用基于服务的方法代表这些项。这种方法需要你单独部署和管理各个服务（或查找其他具备相同功能的工具），而不是当作单个逻辑部署单元。

你可以利用 Azure 资源管理器模板将这些不同的资源声明为一个逻辑部署单元，然后进行部署和管理。请不要以命令方式告知 Azure 逐一部署命令，而应该在 JSON 文件中描述整个部署 - 所有资源及关联的设置以及部署参数 - 然后告诉 Azure 将这些资源视为一个组进行部署。

可在 [Azure 资源管理器概述](/documentation/articles/resource-group-overview/)中了解有关 Azure 资源组及其功能的详细信息。如果你想要了解如何创作模板，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

### 步骤 1：创建模板和参数文件

创建含有下列内容的本地模板文件。将文件命名为 azuredeploy.json。

    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "databaseAccountName": {
                "type": "string"
            }
        },
        "variables": { },
        "resources": [
            {
                "apiVersion": "2015-04-08",
                "type": "Microsoft.DocumentDb/databaseAccounts",
                "name": "[parameters('databaseAccountName')]",
                "location": "[resourceGroup().location]",
                "properties": {
                    "name": "[parameters('databaseAccountName')]",
                    "databaseAccountOfferType": "Standard"
                }
            }
        ]
    }

此模板只需要一个参数：要创建的数据库帐户名称。新数据库帐户的位置会设置为与资源组相同的位置。

由于模板只采用一个参数，因此，你可以在命令行中输入值，也可以创建参数文件来指定值。

若要创建参数文件，请将下列内容复制到新文件中，然后将文件命名为 azuredeploy.parameters.json。如果计划在命令提示符处指定数据库帐户名称，就可以继续操作而不需创建此文件。

    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "databaseAccountName": {
                "value": "samplearmacct"
            }
        }
    }

在 azuredeploy.parameters.json 文件中，将值“samplearmacct”更新为你想要使用的数据库名称，然后保存文件。`<databaseAccountName>` 只能使用小写字母、数字及“-”字符，且长度必须为 3 到 50 个字符。

### 步骤 2：创建或检索资源组

若要创建 DocumentDB 帐户，首先需要一个资源组。如果已经知道想要使用的资源组的名称，请确保其位置是[已正式推出 DocumentDB 的区域](/regions/#services)，然后跳到[步骤 3](#create-account-from-template)。在模板中，帐户的位置会创建在与资源组相同的区域中，因此尝试在无法使用 DocumentDB 的区域中创建帐户会导致部署错误。

若要查看列有你当前所有的资源组的列表，请运行下列命令，并记下你想要使用的资源组名称：

    azure group list

若要创建新的资源组，请运行下列命令，并指定要创建的新资源组的名称，以及要在其中创建资源组的区域：

	azure group create <resourcegroupname> <databaseaccountlocation>

 - `<resourcegroupname>` 只能使用字母数字字符、句点、下划线、“-”字符和括号，且不能以句点结尾。 
 - `<databaseaccountlocation>` 必须是已正式推出 DocumentDB 的区域之一。[Azure 区域页面](https://azure.microsoft.com/regions/#services)会提供当前的区域列表。

输入示例：

	azure group create new_res_group westus

这将生成以下输出：

    info:    Executing command group create
    + Getting resource group new_res_group
    + Creating resource group new_res_group
    info:    Created resource group new_res_group
    data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/new_res_group
    data:    Name:                new_res_group
    data:    Location:            West US
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK

如果遇到错误，请参阅[故障排除](#troubleshooting)。

### <a id="create-account-from-template"></a>步骤 3：使用 ARM 模板创建 DocumentDB 帐户

若要在资源组中创建 DocumentDB 帐户，请运行下列命令，并提供模板文件的路径、参数文件的路径或参数值、要部署于其中的资源组名称，以及部署名称（-n 可选）。

使用参数文件：

    azure group deployment create -f <PathToTemplate> -e <PathToParameterFile> -g <resourcegroupname> -n <deploymentname>

 - `<PathToTemplate>` 是步骤 1 中创建的 azuredeploy.json 文件的路径。如果路径名称含有空格，请使用双引号括住此参数。
 - `<PathToParameterFile>` 是步骤 1 中创建的 azuredeploy.parameters.json 文件的路径。如果路径名称含有空格，请使用双引号括住此参数。
 - `<resourcegroupname>` 是要在其中添加 DocumentDB 数据库帐户的现有资源组的名称。 
 - `<deploymentname>` 是部署的可选名称。

输入示例：

    azure group deployment create -f azuredeploy.json -e azuredeploy.parameters.json -g new_res_group -n azuredeploy

或者，若要指定数据库帐户名称参数而不使用参数文件，并且改为让系统提示输入值，请运行下列命令：

    azure group deployment create -f <PathToTemplate> -g <resourcegroupname> -n <deploymentname>

输入示例，其中显示提示以及名为 new\_db\_acct 的数据库帐户条目：

    azure group deployment create -f azuredeploy.json -g new_res_group -n azuredeploy
    info:    Executing command group deployment create
    info:    Supply values for the following parameters
    databaseAccountName: samplearmacct

预配帐户时，你将收到下列信息：

    info:    Executing command group deployment create
    + Initializing template configurations and parameters
    + Creating a deployment
    info:    Created template deployment "azuredeploy"
    + Waiting for deployment to complete
    data:    DeploymentName     : azuredeploy
    data:    ResourceGroupName  : new_res_group
    data:    ProvisioningState  : Succeeded
    data:    Timestamp          : 2015-11-30T18:50:23.6300288Z
    data:    Mode               : Incremental
    data:    Name                 Type    Value
    data:    -------------------  ------  ------------------
    data:    databaseAccountName  String  samplearmacct
    data:    location             String  West US
    info:    group deployment create command OK

如果遇到错误，请参阅[故障排除](#troubleshooting)。

在此命令返回之后，在帐户更改为**在线**状态以准备好可供使用之前，该帐户将会进入**正在创建**状态数分钟的时间。你可以在 [Azure 门户预览](https://portal.azure.cn)中的“DocumentDB 帐户”边栏选项卡上检查帐户的状态。

## <a name="troubleshooting"></a>故障排除

如果在创建资源组或数据库帐户时收到错误（例如 `Deployment provisioning state was not successful`），有几个故障排除选项可供使用。

> [AZURE.NOTE] 在数据库帐户名称中提供不正确的字符，或提供无法使用 DocumentDB 的位置将导致部署错误。数据库帐户名称只能使用小写字母、数字及“-”字符，且长度必须为 3 到 50 个字符。[Azure 区域页面](/regions/#services)上列出了所有有效的数据库帐户位置。

- 如果你的输出包含下列 `Error information has been recorded to C:\Users\wendy\.azure\azure.err`，则查看 azure.err 文件中的错误信息。

- 可在资源组的日志文件中找到有用的信息。若要查看日志文件，请运行下列命令：

    	azure group log show <resourcegroupname> --last-deployment

    输入示例：

    	azure group log show new_res_group --last-deployment

    

- Azure 门户预览中也会提供错误信息，如下列屏幕截图所示。导航到错误信息的步骤：单击跳转栏中的“资源组”，选择发生错误的资源组，接着在“资源组”边栏选项卡的“概要”区域中单击“上次部署”的日期，然后在“部署历史记录”边栏选项卡中选择失败的部署，之后在“部署”边栏选项卡中单击带有红色感叹号的“操作详细信息”。失败部署的状态消息显示在“操作详细信息”边栏选项卡中。

    ![Azure 门户预览屏幕截图：显示如何导航到部署错误消息](./media/documentdb-automation-resource-manager-cli/portal-troubleshooting-deploy.png)

## 后续步骤

现在你已经有了 DocumentDB 帐户，下一步是创建 DocumentDB 数据库。你可以使用下面其中一项来创建数据库：

- Azure 门户预览，如[使用 Azure 门户预览创建 DocumentDB 数据库](/documentation/articles/documentdb-create-database/)中所述。
- C# .NET 示例，位于 GitHub 上 [azure-documentdb-dotnet](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples) 存储库的 [DatabaseManagement](https://github.com/Azure/azure-documentdb-net/tree/master/samples/code-samples/DatabaseManagement) 项目中。
- [DocumentDB SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx)。DocumentDB 有 .NET、Java、Python、Node.js 和 JavaScript API SDK。 

创建数据库后，你必须向数据库[添加一个或多个集合](/documentation/articles/documentdb-create-collection/)，然后向集合[添加文档](/documentation/articles/documentdb-view-json-document-explorer/)。

当集合中有文档后，你就可以利用门户预览中的[查询资源管理器](/documentation/articles/documentdb-query-collections-query-explorer/)、[REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 或某个 [SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx)，来针对文档使用 [DocumentDB SQL](/documentation/articles/documentdb-sql-query/) [执行查询](/documentation/articles/documentdb-sql-query/#executing-queries)。

若要详细了解 DocumentDB，请浏览以下资源：

-	[DocumentDB 资源模型和概念](/documentation/articles/documentdb-resources/)



<!---HONumber=Mooncake_0425_2016-->