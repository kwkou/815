<!-- not suitable for Mooncake -->


<properties
   pageTitle="使用 Azure Resource Manager 模板在 HDInsight 中创建基于 Windows 的 Hadoop 群集 | Azure"
   	description="了解如何使用 Azure Resource Manager 模板创建 Azure HDInsight 的群集。"
   services="hdinsight"
   documentationCenter=""
   tags="azure-portal"
   authors="mumian"
   manager="jhubbard"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="07/25/2016"
   wacn.date="02/06/2017"
   ms.author="jgao"/>  


# 使用 Azure Resource Manager 模板在 HDInsight 中创建基于 Windows 的 Hadoop 群集

[AZURE.INCLUDE [选择器](../../includes/hdinsight-selector-create-clusters.md)]

了解如何使用 Azure Resource Manager 模板创建 HDInsight 群集。有关详细信息，请参阅 [Deploy an application with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署应用程序）。有关其他群集创建工具和功能，请单击本页顶部的相应选项卡，或参阅[群集创建方法](/documentation/articles/hdinsight-provision-clusters/#cluster-creation-methods)。

##先决条件：

[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]


在开始按照本文中的说明操作之前，你必须具有以下内容：

- [Azure 订阅](/pricing/1rmb-trial/)。
- Azure PowerShell 或 Azure CLI

    [AZURE.INCLUDE [use-latest-version](../../includes/hdinsight-use-latest-powershell-and-cli.md)]

### 访问控制要求

[AZURE.INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## Resource Manager 模板

使用 Resource Manager 模板可以轻松地通过单个协调操作为应用程序创建 HDInsight 群集、其所依赖的资源（如默认存储帐户）和其他资源（例如，要使用 Apache Sqoop 的 Azure SQL 数据库）。在模板中，定义应用程序所需的资源，并指定部署参数以针对不同的环境输入值。模板中包含可用于构造部署值的 JSON 和表达式。

可以在[附录 A](#appx-a-arm-template) 中找到用于创建 HDInsight 群集及所依赖的 Azure 存储帐户的 Resource Manager 模板。使用文本编辑器将模板保存到工作站上的文件中。你将学习如何使用各种工具调用模板。

有关 Resource Manager 模板的详细信息，请参阅

- [创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)
- [使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)


## <a name="deploy-with-powershell"></a> 使用 PowerShell 进行部署

以下过程创建 HDInsight 群集。

**使用 Resource Manager 模板部署群集**

1. 将[附录 A](#appx-a-arm-template) 中的 json 文件保存到工作站。
2. 如果需要，请设置参数。
3. 使用以下 PowerShell 脚本运行模板：

        ####################################
        # Set these variables
        ####################################
        #region - used for creating Azure service names
        $nameToken = "<Enter an Alias>" 
        $templateFile = "C:\HDITutorials-ARM\hdinsight-arm-template.json"
        #endregion

        ####################################
        # Service names and varialbes
        ####################################
        #region - service names
        $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

        $resourceGroupName = $namePrefix + "rg"
        $hdinsightClusterName = $namePrefix + "hdi"
        $defaultStorageAccountName = $namePrefix + "store"
        $defaultBlobContainerName = $hdinsightClusterName

        $location = "China East"

        $armDeploymentName = $namePrefix
        #endregion

        ####################################
        # Connect to Azure
        ####################################
        #region - Connect to Azure subscription
        Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
        try{Get-AzureRmContext}
        catch{Login-AzureRmAccount}
        #endregion

        # Create a resource group
        New-AzureRmResourceGroup -Name $resourceGroupName -Location $Location

        # Create cluster and the dependent storage accounge
        $parameters = @{clusterName="$hdinsightClusterName";clusterStorageAccountName="$defaultStorageAccountName"}

        New-AzureRmResourceGroupDeployment `
            -Name $armDeploymentName `
            -ResourceGroupName $resourceGroupName `
            -TemplateFile $templateFile `
            -TemplateParameterObject $parameters

        # List cluster
        Get-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName 

	PowerShell 脚本仅配置群集名称和存储帐户名称。可以在 ARM 模板中设置其他值。
	
有关详细信息，请参阅 [Deploy with PowerShell](/documentation/articles/resource-group-template-deploy/)（使用 PowerShell 进行部署）。

## <a name="deploy-with-azure-cli"></a> 使用 Azure CLI 进行部署

以下示例将通过调用 ARM 模板创建一个群集及其依赖的存储帐户和容器：

	azure login
	azure config mode arm
    azure group create -n hdi1229rg -l "China East"
    azure group deployment create "hdi1229rg" "hdi1229" --template-file "C:\HDITutorials-ARM\hdinsight-arm-template.json" -p "{"clusterName":{"value":"hdi1229win"},"clusterStorageAccountName":{"value":"hdi1229store"},"location":{"value":"China East"},"clusterLoginPassword":{"value":"Pass@word1"}}"





## 使用 REST API 进行部署

请参阅 [Deploy with the REST API](/documentation/articles/resource-group-template-deploy-rest/)（使用 REST API 进行部署）。

## 使用 Visual Studio 进行部署

使用 Visual Studio，可以创建资源组项目，并通过用户界面将其部署到 Azure。可选择要在项目中包括的资源类型，这些资源将自动添加到资源管理器模板中。该项目还提供了用于部署模板的 PowerShell 脚本。

有关将 Visual Studio 用于资源组的简介，请参阅 [Creating and deploying Azure resource groups through Visual Studio](/documentation/articles/vs-azure-tools-resource-groups-deployment-projects-create-deploy/)（通过 Visual Studio 创建和部署 Azure 资源组）。

##后续步骤
在本文中，你已经学习了几种创建 HDInsight 群集的方法。若要了解更多信息，请参阅下列文章：


- 有关通过 .NET 客户端库部署资源的示例，请参阅 [Deploy resources using .NET libraries and a template](/documentation/articles/virtual-machines-windows-csharp-template/)（使用 .NET 库和模板部署资源）。
- 有关部署应用程序的详细示例，请参阅 [Provision and deploy microservices predictably in Azure](/documentation/articles/app-service-deploy-complex-application-predictably/)（按可预见的方式在 Azure 中预配和部署微服务）。
- 有关将解决方案部署到不同环境的指南，请参阅 [Development and test environments in Azure](/documentation/articles/solution-dev-test-environments/)（Azure 中的开发和测试环境）。
- 若要了解 Azure Resource Manager 模板的节，请参阅 [Authoring templates](/documentation/articles/resource-group-authoring-templates/)（创作模板）。
- 有关可在 Azure Resource Manager 模板中使用的函数列表，请参阅 [Template functions](/documentation/articles/resource-group-template-functions/)（模板函数）。



## <a name="appx-a-arm-template"></a>附录 A：Resource Manager 模板

以下 Azure Resource Manager 模板使用依赖的 Azure 存储帐户创建基于 Windows 的 Hadoop 群集。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
        "type": "string",
        "defaultValue": "China East",
        "allowedValues": [
            "China North",
            "China East"
        ],
        "metadata": {
            "description": "The location where all azure resources will be deployed."
        }
        },
        "clusterName": {
        "type": "string",
        "metadata": {
            "description": "The name of the HDInsight cluster to create."
        }
        },
        "clusterLoginUserName": {
        "type": "string",
        "defaultValue": "admin",
        "metadata": {
            "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
        }
        },
        "clusterLoginPassword": {
        "type": "securestring",
        "metadata": {
            "description": "The password for the cluster login."
        }
        },
        "clusterStorageAccountName": {
        "type": "string",
        "metadata": {
            "description": "The name of the storage account to be created and be used as the cluster's storage."
        }
        },
        "clusterStorageType": {
        "type": "string",
        "defaultValue": "Standard_LRS",
        "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS"
        ]
        },
        "clusterWorkerNodeCount": {
        "type": "int",
        "defaultValue": 4,
        "metadata": {
            "description": "The number of nodes in the HDInsight cluster."
        }
        }
    },
        "variables": {},
        "resources": [
            {
            "name": "[parameters('clusterStorageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[parameters('location')]",
            "apiVersion": "2015-05-01-preview",
            "dependsOn": [],
            "tags": {},
            "properties": {
                "accountType": "[parameters('clusterStorageType')]"
            }
            },
            {
            "name": "[parameters('clusterName')]",
            "type": "Microsoft.HDInsight/clusters",
            "location": "[parameters('location')]",
            "apiVersion": "2015-03-01-preview",
            "dependsOn": [
                "[concat('Microsoft.Storage/storageAccounts/',parameters('clusterStorageAccountName'))]"
            ],
            "tags": {},
            "properties": {
                "clusterVersion": "3.2",
                "osType": "Windows",
                "clusterDefinition": {
                "kind": "hadoop",
                "configurations": {
                    "gateway": {
                    "restAuthCredential.isEnabled": true,
                    "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
                    "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
                    }
                }
                },
                "storageProfile": {
                "storageaccounts": [
                    {
                    "name": "[concat(parameters('clusterStorageAccountName'),'.blob.core.chinacloudapi.cn')]",
                    "isDefault": true,
                    "container": "[parameters('clusterName')]",
                    "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('clusterStorageAccountName')), '2015-05-01-preview').key1]"
                    }
                ]
                },
                "computeProfile": {
                "roles": [
                    {
                    "name": "headnode",
                    "targetInstanceCount": "1",
                    "hardwareProfile": {
                        "vmSize": "Large"
                    }
                    },
                    {
                    "name": "workernode",
                    "targetInstanceCount": "[parameters('clusterWorkerNodeCount')]",
                    "hardwareProfile": {
                        "vmSize": "Large"
                    }
                    }
                ]
                }
            }
            }
        ],
        "outputs": {
            "cluster": {
            "type": "object",
            "value": "[reference(resourceId('Microsoft.HDInsight/clusters',parameters('clusterName')))]"
            }
        }
    }

<!---HONumber=Mooncake_1107_2016-->