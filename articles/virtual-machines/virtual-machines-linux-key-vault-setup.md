<properties
    pageTitle="为 Linux VM 设置 Azure Key Vault | Azure"
    description="如何使用 CLI 2.0 设置用于 Azure Resource Manager 虚拟机的 Key Vault。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="bccdd5ab-5ccf-4760-9039-92c6eafb15bd"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/24/2017"
    wacn.date="04/24/2017"
    ms.author="singhkay"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="3431fa0d82be47a97a49a91e54bb0743f9c801d0"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-set-up-key-vault-for-virtual-machines-with-the-azure-cli-20"></a>如何使用 Azure CLI 2.0 为虚拟机设置 Key Vault

在 Azure Resource Manager 堆栈中，密码/证书被建模为 Key Vault 所提供的资源。 若要了解有关 Azure 密钥保管库的详细信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/) 为了让 Key Vault 能与 Azure Resource Manager VM 搭配使用，必须将 Key Vault 上的 *EnabledForDeployment* 属性设置为 true。 本文说明如何通过 Azure CLI 2.0 设置用于 Azure 虚拟机 (VM) 的 Key Vault。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-key-vault-setup-cli-nodejs/) 执行这些步骤。

若要执行这些步骤，需要安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-a-key-vault"></a>创建密钥保管库
使用 [az keyvault create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#create) 创建密钥保管库并分配部署策略。 以下示例在 `myResourceGroup` 资源组中创建名为 `myKeyVault` 的密钥保管库：

    az keyvault create -l chinanorth -n myKeyVault -g myResourceGroup --enabled-for-deployment true

## <a name="update-a-key-vault-for-use-with-vms"></a>更新用于 VM 的 Key Vault
使用 [az keyvault update](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#update) 在现有的密钥保管库上设置部署策略。 以下命令在 `myResourceGroup` 资源组中更新名为 `myKeyVault` 的密钥保管库：

    az keyvault update -n myKeyVault -g myResourceGroup --set properties.enabledForDeployment=true

## <a name="use-templates-to-set-up-key-vault"></a>使用模板设置密钥保管库
使用模板时，必须将 Key Vault 资源的 `enabledForDeployment` 属性设置为 `true`，如下所示：

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

## <a name="next-steps"></a>后续步骤
有关使用模板创建 Key Vault 时可以配置的其他选项，请参阅[创建密钥保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create/)。

>[AZURE.NOTE]
> 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。 例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。
<!--Update_Description: change from CLI 1.0 to CLI 2.0-->