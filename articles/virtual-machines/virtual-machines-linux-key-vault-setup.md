<properties
	pageTitle="在 Azure Resource Manager 中为虚拟机设置密钥保管库 | Azure"
	description="如何设置与 Azure Resource Manager 虚拟机搭配使用的密钥保管库。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="singhkay"
	manager="drewm"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/31/2016"
	wacn.date="07/11/2016"/>

# 在 Azure Resource Manager 中为虚拟机设置密钥保管库

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型

在 Azure Resource Manager 堆栈中，密码/证书被建模为密钥保管库资源提供程序所提供的资源。若要了解有关 Azure 密钥保管库的详细信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

为了让密钥保管库能与 Azure Resource Manager 虚拟机搭配使用，必须将密钥保管库上的 *EnabledForDeployment* 属性设置为 true。你可以在各种客户端中执行此操作。

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

有关使用模板创建密钥保管库时可以配置的其他选项，请参阅[创建密钥保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create)。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

<!---HONumber=Mooncake_0704_2016-->