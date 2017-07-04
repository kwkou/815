<properties
    pageTitle="Azure CLI 脚本示例 - 部署模板 | Azure"
    description="用于部署 Azure Resource Manager 模板的示例脚本。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/19/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="9385b7ff9b3e7a82fa5d8f8e54c74fce223dd269"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-resource-manager-template-deployment---azure-cli-script"></a>Azure Resource Manager 模板部署 - Azure CLI 脚本

此脚本将 Resource Manager 模板部署到订阅中的资源组。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>示例脚本

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
            az group create --name $resourceGroupName --location $resourceGroupLocation 1> /dev/null
        )
        else
        echo "Using existing resource group..."
    fi

    #Start deployment
    echo "Starting deployment..."
    (
        set -x
        az group deployment create --name $deploymentName --resource-group $resourceGroupName --template-file $templateFilePath --parameters $parametersFilePath
    )

    if [ $?  == 0 ];
     then
        echo "Template has been successfully deployed"
    fi

## <a name="clean-up-deployment"></a>清理部署 

运行以下命令以删除资源组及其所有资源。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建部署。 表中的每一项均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group show](https://docs.microsoft.com/zh-cn/cli/azure/group#show) | 获取资源组。 |
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az group deployment create](https://docs.microsoft.com/zh-cn/cli/azure/group/deployment#create) | 启动部署。  |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/group#delete) | 删除资源组，包括其所有资源。 |

## <a name="next-steps"></a>后续步骤
* 有关部署模板的简介，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy-cli/)。
* 有关部署需要 SAS 令牌的模板的信息，请参阅[使用 SAS 令牌部署专用模板](/documentation/articles/resource-manager-cli-sas-token/)。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
* 有关企业可如何使用 Resource Manager 有效管理订阅的指南，请参阅 [Azure 企业基架 - 出于合规目的监管订阅](/documentation/articles/resource-manager-subscription-governance/)。

<!--Update_Description:new article about deploying resource manager sample with CLI -->
