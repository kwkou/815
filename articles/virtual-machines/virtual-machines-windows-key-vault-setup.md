<properties
	pageTitle="在 Azure Resource Manager 中为虚拟机设置密钥保管库 | Azure"
	description="如何设置与 Azure Resource Manager 虚拟机搭配使用的密钥保管库。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="singhkays"
	manager="drewm"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/24/2017"
	wacn.date="03/28/2017"
	ms.author="singhkay"/>

# 在 Azure Resource Manager 中为虚拟机设置密钥保管库

> [AZURE.NOTE] Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型

在 Azure Resource Manager 堆栈中，密码/证书被建模为密钥保管库资源提供程序所提供的资源。若要了解有关密钥保管库的详细信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

>[AZURE.NOTE] 
>1. 为了让密钥保管库能与 Azure Resource Manager 虚拟机搭配使用，必须将密钥保管库上的 **EnabledForDeployment** 属性设置为 true。可以在各种客户端中执行此操作。
><p>2.需要在与虚拟机相同的订阅和位置中创建密钥保管库。

## 使用 PowerShell 设置密钥保管库
若要使用 PowerShell 创建密钥保管库，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/#vault)。

对于新的密钥保管库，可以使用此 PowerShell cmdlet：

	New-AzureRmKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'China East' -EnabledForDeployment

对于现有的密钥保管库，可以使用此 PowerShell cmdlet：

	Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -EnabledForDeployment

## 使用 CLI 设置密钥保管库
若要使用命令行接口 (CLI) 来创建密钥保管库，请参阅[使用 CLI 管理密钥保管库](/documentation/articles/key-vault-manage-with-cli/#create-a-key-vault)。

使用 CLI 时，必须先创建密钥保管库，然后分配部署策略。可以使用以下命令来执行此操作：

	azure keyvault set-policy ContosoKeyVault -enabled-for-deployment true

## 使用模板设置密钥保管库
使用模板时，必须将密钥保管库资源的 `enabledForDeployment` 属性设置为 `true`。

	{
      "type": "Microsoft.KeyVault/vaults",
      "name": "ContosoKeyVault",
      "apiVersion": "2015-06-01",
      "location": "<location-of-key-vault>",
      "properties": {
        "enabledForDeployment": "true",
        ....
        ....
      }
    }

有关使用模板创建密钥保管库时可以配置的其他选项，请参阅[创建密钥保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create/)。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->