<properties
    pageTitle="使用 Azure CLI 和模板部署资源 | Azure"
    description="使用 Azure Resource Manager 和 Azure CLI 将资源部署到 Azure。资源在 Resource Manager 模板中定义。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="493b7932-8d1e-4499-912c-26098282ec95"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/10/2017"
    wacn.date="03/31/2017"
    ms.author="tomfitz" />  


# 使用 Resource Manager 模板和 Azure CLI 部署资源
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-template-deploy/)
- [Azure CLI](/documentation/articles/resource-group-template-deploy-cli/)
- [门户](/documentation/articles/resource-group-template-deploy-portal/)
- [REST API](/documentation/articles/resource-group-template-deploy-rest/)

本主题介绍如何将 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 与 Resource Manager 模板配合使用向 Azure 部署资源。你的模板可以是本地文件或是可通过 URI 访问的外部文件。如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。

## <a name="deploy"></a> 部署

* 若要快速开始部署，请使用以下命令部署带内联参数的本地模板：

        az login
        az account set --subscription {subscription-id}

        az group create --name ExampleGroup --location "China North"
        az group deployment create \
          --name ExampleDeployment \
          --resource-group ExampleGroup \
          --template-file storage.json \
          --parameters '{"storageNamePrefix":{"value":"contoso"},"storageSKU":{"value":"Standard_GRS"}}'

    部署可能需要几分钟才能完成。完成之后，你将看到一条包含以下结果的消息：

        "provisioningState": "Succeeded",

* 仅当要使用非默认订阅时，才需要 `az account set` 命令。若要查看所有订阅及其 ID，请使用：

        az account list

* 若要部署外部模板，请使用 **template-uri** 参数：

        az group deployment create \
           --name ExampleDeployment \
           --resource-group ExampleGroup \
           --template-uri "https://raw.githubusercontent.com/exampleuser/MyTemplates/master/storage.json" \
           --parameters '{"storageNamePrefix":{"value":"contoso"},"storageSKU":{"value":"Standard_GRS"}}'

* 若要在文件中传递参数值，请使用：

        az group deployment create \
           --name ExampleDeployment \
           --resource-group ExampleGroup \
           --template-file storage.json \
           --parameters @storage.parameters.json

    参数文件必须采用以下格式：

        {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "storageNamePrefix": {
                "value": "contoso"
            },
            "storageSKU": {
                "value": "Standard_GRS"
            }
         }
        }

[AZURE.INCLUDE [resource-manager-deployments](../../includes/resource-manager-deployments.md)]

若要使用完整模式，请使用 mode 参数：

        az group deployment create \
        --name ExampleDeployment \
        --mode Complete \
        --resource-group ExampleGroup \
        --template-file storage.json \
        --parameters '{"storageNamePrefix":{"value":"contoso"},"storageSKU":{"value":"Standard_GRS"}}'

## 使用 SAS 令牌从存储空间部署模板
可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT]
通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。如果其他用户截获了该 URI，则此用户可以访问该模板。使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。
> 
> 

### 将专用模板添加到存储帐户
以下示例设置专用存储帐户容器并上载模板：

    az group create --name "ManageGroup" --location "China East"
    az storage account create \
        --resource-group ManageGroup \
        --location "China East" \
        --sku Standard_LRS \
        --kind Storage \
        --name {your-unique-name}
    connection=$(az storage account show-connection-string \
        --resource-group ManageGroup \
        --name {your-unique-name} \
        --query connectionString)
    az storage container create \
        --name templates \
        --public-access Off \
        --connection-string $connection
    az storage blob upload \
        --container-name templates \
        --file vmlinux.json \
        --name vmlinux.json \
        --connection-string $connection

### 在部署期间提供 SAS 令牌
若要在存储帐户中部署专用模板，请生成 SAS 令牌，并将其包括在模板的 URI 中。设置到期时间以允许足够的时间来完成部署。

    seconds='@'$(( $(date +%s) + 1800 ))
    expiretime=$(date +%Y-%m-%dT%H:%MZ --date=$seconds)
    connection=$(az storage account show-connection-string \
        --resource-group ManageGroup \
        --name {your-unique-name} \
        --query connectionString)
    token=$(az storage blob generate-sas \
        --container-name templates \
        --name vmlinux.json \
        --expiry $expiretime \
        --permissions r \
        --output tsv \
        --connection-string $connection)
    url=$(az storage blob url \
        --container-name templates \
        --name vmlinux.json \
        --output tsv \
        --connection-string $connection)
    az group deployment create --resource-group ExampleGroup --template-uri $url?$token

有关将 SAS 令牌与链接模板配合使用的示例，请参阅[将已链接的模版与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。

## 调试

若要查看有关失败的部署操作的信息，请使用：

    az group deployment operation list --resource-group ExampleGroup --name vmlinux --query "[*].[properties.statusMessage]"

有关解决常见部署错误的提示，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。

## 完整部署脚本

以下示例显示用于部署[导出模板](/documentation/articles/resource-manager-export-template/)功能生成的模板的 Azure CLI 2.0 脚本：

    #!/bin/bash
    set -euo pipefail
    IFS=$'\n\t'

    # -e: immediately exit if any command has a non-zero exit status
    # -o: prevents errors in a pipeline from being masked
    # IFS new value is less likely to cause confusing bugs when looping arrays or arguments (e.g. $@)

    usage() { echo "Usage: $0 -i <subscriptionId> -g <resourceGroupName> -n <deploymentName> -l <resourceGroupLocation>" 1>&2; exit 1; }

    declare subscriptionId=""
    declare resourceGroupName=""
    declare deploymentName=""
    declare resourceGroupLocation=""

    # Initialize parameters specified from command line
    while getopts ":i:g:n:l:" arg; do
        case "${arg}" in
            i)
                subscriptionId=${OPTARG}
                ;;
            g)
                resourceGroupName=${OPTARG}
                ;;
            n)
                deploymentName=${OPTARG}
                ;;
            l)
                resourceGroupLocation=${OPTARG}
                ;;
            esac
    done
    shift $((OPTIND-1))

    #Prompt for parameters is some required parameters are missing
    if [[ -z "$subscriptionId" ]]; then
        echo "Subscription Id:"
        read subscriptionId
        [[ "${subscriptionId:?}" ]]
    fi

    if [[ -z "$resourceGroupName" ]]; then
        echo "ResourceGroupName:"
        read resourceGroupName
        [[ "${resourceGroupName:?}" ]]
    fi

    if [[ -z "$deploymentName" ]]; then
        echo "DeploymentName:"
        read deploymentName
    fi

    if [[ -z "$resourceGroupLocation" ]]; then
        echo "Enter a location below to create a new resource group else skip this"
        echo "ResourceGroupLocation:"
        read resourceGroupLocation
    fi

    #templateFile Path - template file to be used
    templateFilePath="template.json"

    if [ ! -f "$templateFilePath" ]; then
        echo "$templateFilePath not found"
        exit 1
    fi

    #parameter file path
    parametersFilePath="parameters.json"

    if [ ! -f "$parametersFilePath" ]; then
        echo "$parametersFilePath not found"
        exit 1
    fi

    if [ -z "$subscriptionId" ] || [ -z "$resourceGroupName" ] || [ -z "$deploymentName" ]; then
        echo "Either one of subscriptionId, resourceGroupName, deploymentName is empty"
        usage
    fi

    #login to azure using your credentials
    az account show 1> /dev/null

    if [ $? != 0 ];
    then
        az login
    fi

    #set the default subscription id
    az account set --name $subscriptionId

    set +e

    #Check for existing RG
    az group show $resourceGroupName 1> /dev/null

    if [ $? != 0 ]; then
        echo "Resource group with name" $resourceGroupName "could not be found. Creating new resource group.."
        set -e
        (
            set -x
            az resource group create --name $resourceGroupName --location $resourceGroupLocation 1> /dev/null
        )
        else
        echo "Using existing resource group..."
    fi

    #Start deployment
    echo "Starting deployment..."
    (
        set -x
        az resource group deployment create --name $deploymentName --resource-group $resourceGroupName --template-file $templateFilePath --parameters $parametersFilePath
    )

    if [ $?  == 0 ];
     then
        echo "Template has been successfully deployed"
    fi

## 后续步骤
* 有关通过 .NET 客户端库部署资源的示例，请参阅 [Deploy resources using .NET libraries and a template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 .NET 库和模板部署资源）。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
<!--* 有关将解决方案部署到不同环境的指南，请参阅 [Development and test environments in Azure](/documentation/articles/solution-dev-test-environments/)（Azure 中的开发和测试环境）。-->
* 有关使用 KeyVault 引用来传递安全值的详细信息，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter/)。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。
* 有关自动化部署的四部分系列教程，请参阅[将应用程序自动部署到 Azure 虚拟机](/documentation/articles/virtual-machines-windows-dotnet-core-1-landing/)。此系列教程介绍了应用程序体系结构、访问与安全性、可用性与缩放性，以及应用程序部署。

<!---HONumber=Mooncake_0327_2017-->
<!-- Update_Description:update meta properties; wording update; update link reference; add new feature of debug and whole deployment of Azure CLI 2.0  -->