<properties
    pageTitle="使用 SAS 令牌和 PowerShell 部署 Azure 模板 | Azure"
    description="使用 Azure Resource Manager 和 Azure PowerShell 从使用 SAS 令牌保护的模板将资源部署到 Azure。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/19/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="37e738e20aab8321083fd3d31d7437a74ad2dc99"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="deploy-private-resource-manager-template-with-sas-token-and-azure-powershell"></a>使用 SAS 令牌和 Azure PowerShell 部署专用 Resource Manager 模板

如果模板驻留在存储帐户中，你可以限制对该模板的访问，并在部署过程中提供共享访问签名 (SAS) 令牌。 本主题介绍如何将 Azure PowerShell 与 Resource Manager 模板配合使用在部署过程中提供 SAS 令牌。 

## <a name="add-private-template-to-storage-account"></a>将专用模板添加到存储帐户

可以将模板添加到存储帐户，并在部署过程中使用 SAS 令牌链接到这些模板。

> [AZURE.IMPORTANT]
> 通过执行以下步骤，只有帐户所有者可以访问包含模板的 blob。 但是，如果为 blob 创建 SAS 令牌，则拥有该 URI 的任何人都可以访问 blob。 如果其他用户截获了该 URI，则此用户可以访问该模板。 使用 SAS 令牌是限制对模板的访问的好方法，但不应直接在模板中包括密码等敏感数据。
> 
> 

以下示例设置专用存储帐户容器并上传模板：

    # create a storage account for templates
    New-AzureRmResourceGroup -Name ManageGroup -Location "China East"
    New-AzureRmStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name} -Type Standard_LRS -Location "China North"
    Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}

    # create a container and upload template
    New-AzureStorageContainer -Name templates -Permission Off
    Set-AzureStorageBlobContent -Container templates -File c:\MyTemplates\storage.json

## <a name="provide-sas-token-during-deployment"></a>在部署期间提供 SAS 令牌
若要在存储帐户中部署专用模板，请生成 SAS 令牌，并将其包括在模板的 URI 中。 设置到期时间以允许足够的时间来完成部署。

    Set-AzureRmCurrentStorageAccount -ResourceGroupName ManageGroup -Name {your-unique-name}

    # get the URI with the SAS token
    $templateuri = New-AzureStorageBlobSASToken -Container templates -Blob storage.json -Permission r `
      -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

    # provide URI with SAS token during deployment
    New-AzureRmResourceGroup -Name ExampleGroup -Location "China East"
    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri $templateuri

有关将 SAS 令牌与链接模板配合使用的示例，请参阅[将已链接的模版与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。

## <a name="next-steps"></a>后续步骤
* 有关部署模板的简介，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy/)。
* 有关用于部署模板的完整示例脚本，请参阅[部署 Resource Manager 模板脚本](/documentation/articles/resource-manager-samples-powershell-deploy/)
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
* 有关企业可如何使用 Resource Manager 有效管理订阅的指南，请参阅 [Azure 企业基架 - 出于合规目的监管订阅](/documentation/articles/resource-manager-subscription-governance/)。

<!--Update_Description:new article about how to use sas token with powershell-->
