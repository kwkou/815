<properties
    pageTitle="Azure CLI 脚本示例 - 创建批处理帐户 | Azure"
    description="Azure CLI 脚本示例 - 创建批处理帐户"
    services="batch"
    documentationcenter=""
    author="annatisch"
    manager="daryls"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="batch"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="multiple"
    ms.workload="na"
    ms.date="03/20/2017"
    wacn.date="05/15/2017"
    ms.author="antisch"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="e6dfccdfd73aaed384463924977e786164f0a6d2"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-batch-account-with-the-azure-cli"></a>使用 Azure CLI 创建批处理帐户

此脚本创建 Azure 批处理帐户，并演示如何查询和更新该帐户的各个属性。

如果需要，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 登录到 Azure。

`az login` 默认是登录到 azure global 环境，要登录到 azure China 环境，请先运行 `az configure` 命令，根据提示去修改相应的文件才可以。 Windows 环境下需要修改 `C:\\Users\\userName\\.azure\\config` 文件，将 `name = AzureCloud` 改为 `name = AzureChinaCloud`。

## <a name="batch-account-sample-script"></a>批处理帐户示例脚本

创建批处理帐户时，默认情况下，其计算节点由批处理服务在内部分配。 分配的计算节点受到单独的核心配额的限制。可以通过共享密钥凭据或 Azure Active Dirctory 令牌对帐户进行身份验证。

    #!/bin/bash

    # Authenticate CLI session.
    az login

    # Create a resource group.
    az group create --name myresourcegroup --location westeurope

    # Create a Batch account.
    az batch account create -g myresourcegroup -n mybatchaccount -l westeurope

    # Now we can display the details of our created account.
    az batch account show -g myresourcegroup -n mybatchaccount

    # Let's add a storage account reference to the Batch account for use as 'auto-storage'
    # for applications. We'll start by creating the storage account.
    az storage account create -g myresourcegroup -n mystorageaccount -l westeurope --sku Standard_LRS

    # And then update the Batch account with the either the name (if they exist in
    # the same resource group) or the full resource ID of the storage account.
    az batch account set -g myresourcegroup -n mybatchaccount --storage-account mystorageaccount

    # We can view the access keys to the Batch Account for future client authentication.
    az batch account keys list

    # Or we can authenticate against the account directly for further CLI interaction.
    az batch account login -g myresourcegroup -n mybatchaccount --shared-key-auth

## <a name="batch-account-using-user-subscription-sample-script"></a>使用用户订阅创建批处理帐户的示例脚本

也可以选择让批处理在你自己的 Azure 订阅中创建其计算节点。
用于将计算节点分配到订阅的帐户必须通过 Azure Active Directory 令牌进行身份验证，分配的计算节点数将计入订阅配额。 若要在此模式下创建帐户，必须在创建帐户时指定 Key Vault 引用。

    #!/bin/bash

    # Authenticate CLI session.
    az login

    # Create a resource group.
    az group create --name myresourcegroup --location westeurope

    # A Batch account that will allocate pools in the user's subscription must be configured
    # with a Key Vault located in the same region. Let's create this first.
    az keyvault create \
        --resource-group myresourcegroup \
        --name mykevault \
        --location westeurope \
        --enabled-for-deployment true \
        --enabled-for-disk-encryption true \
        --enabled-for-template-deployment true

    # We will add an access-policy to the Key Vault to allow access by the Batch Service.
    az keyvault set-policy \
        --resource-group myresourcegroup \
        --name mykevault \
        --object-id f520d84c-3fd3-4cc8-88d4-2ed25b00d27a \
        --key-permissions all \
        --secret-permissions all

    # Now we can create the Batch account, referencing the Key Vault either by name (if they
    # exist in the same resource group) or by its full resource ID.
    az batch account create \
        --resource-group myresourcegroup \
        --name mybatchaccount \
        --location westeurope \
        --keyvault mykevault

    # We can now authenticate directly against the account for further CLI interaction.
    # Note that Batch accounts that allocate pools in the user's subscription must be
    # authenticated via an Azure Active Directory token.
    az batch account login -g myresourcegroup -n mybatchaccount

## <a name="clean-up-deployment"></a>清理部署

运行上述任一示例脚本后，请运行以下命令以删除资源组和所有相关的资源（包括批处理帐户、Azure 存储帐户和 Azure Key Vault）。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、批处理帐户和所有相关资源。 表中的每条命令链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az batch account create](https://docs.microsoft.com/zh-cn/cli/azure/batch/account#create) | 创建批处理帐户。  |
| [az batch account set](https://docs.microsoft.com/zh-cn/cli/azure/batch/account#set) | 更新批处理帐户的属性。  |
| [az batch account show](https://docs.microsoft.com/zh-cn/cli/azure/batch/account#show) | 检索指定批处理帐户的详细信息。  |
| [az batch account keys list](https://docs.microsoft.com/zh-cn/cli/azure/batch/account/keys#list) | 检索指定批处理帐户的访问密钥。  |
| [az batch account login](https://docs.microsoft.com/zh-cn/cli/azure/batch/account#login) | 针对指定的批处理帐户进行身份验证，以便进一步进行 CLI 交互。  |
| [az storage account create](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#create) | 创建存储帐户。 |
| [az keyvault create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#create) | 创建密钥保管库。 |
| [az keyvault set-policy](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#set-policy) | 更新指定 Key Vault 的安全策略。 |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/group#delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 批处理 CLI 文档](/documentation/articles/batch-cli-samples/)中找到其他批处理 CLI 脚本示例。

