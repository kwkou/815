<properties
    pageTitle="加密 Azure 中 Linux VM 上的磁盘 | Azure"
    description="如何使用 Azure CLI 2.0 加密 Linux VM 上的虚拟磁盘以增强安全性"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="2a23b6fa-6941-4998-9804-8efe93b647b3"
    ms.service="virtual-machines-linux"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/23/2017"
    wacn.date="05/15/2017"
    ms.author="iainfou"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="70566157bcd2103c27a292b2eb6232a4c0ce2264"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="how-to-encrypt-virtual-disks-on-a-linux-vm"></a>如何加密 Linux VM 上的虚拟磁盘
为了增强虚拟机 (VM) 的安全性以及符合性，可以加密 Azure 中的虚拟磁盘。 磁盘是使用 Azure 密钥保管库中受保护的加密密钥加密的。 可以控制这些加密密钥，以及审核对它们的使用。 本文介绍如何使用 Azure CLI 2.0 加密 Linux VM 上的虚拟磁盘。 还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-encrypt-disks-nodejs/) 执行这些步骤。

## <a name="quick-commands"></a>快速命令
如果需要快速完成任务，请参阅以下部分，其中详细说明了用于加密 VM 中虚拟磁盘的基本命令。 本文档的余下部分（[从此处开始](#overview-of-disk-encryption)）提供了每个步骤的更详细信息和上下文。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

需要安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 并已使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。 在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 `myResourceGroup`、`myKey` 和 `myVM`。

首先，在 Azure 订阅中使用 [az provider register](https://docs.microsoft.com/zh-cn/cli/azure/provider#register) 启用 Azure Key Vault 提供程序，并使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建一个资源组。 以下示例在 `ChinaNorth` 位置创建资源组名称 `myResourceGroup`：

    az provider register -n Microsoft.KeyVault
    az group create --name myResourceGroup --location ChinaNorth

使用 [az keyvault create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#create) 创建 Azure Key Vault，并启用 Key Vaul，将其用于磁盘加密。 指定 `keyvault_name` 的唯一 Key Vault 名称，如下所示：

    keyvault_name=myUniqueKeyVaultName
    az keyvault create --name $keyvault_name --resource-group myResourceGroup \
      --location ChinaNorth --enabled-for-disk-encryption true

使用 [az keyvault key create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault/key#create) 在 Key Vault 中创建加密密钥。 以下示例创建名为 `myKey` 的密钥：

    az keyvault key create --vault-name $keyvault_name --name myKey --protection software

通过将 Azure Active Directory 与 [az ad sp create-for-rbac](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#create-for-rbac) 配合使用创建服务主体。 服务主体可处理 Key Vault 中的加密密钥的身份验证和交换。 下面的示例读入服务主体 ID 和密码的值，供稍后的命令中使用：

    read sp_id sp_password <<< $(az ad sp create-for-rbac --query [appId,password] -o tsv)

仅在创建服务主体时，输出密码。 如果需要，可查看并记录密码 (`echo $sp_password`)。 可以使用 [az ad sp list](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#list) 列出服务主体并使用 [az ad sp show](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#show) 查看特定服务主体的详细信息。

使用 [az keyvault set-policy](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#set-policy) 在 Key Vault 上设置权限。 在下面的示例中，服务主体 ID 由之前的命令提供：

    az keyvault set-policy --name $keyvault_name --spn $sp_id \
      --key-permissions all \
      --secret-permissions all

使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建 VM。 只有特定应用商店映像才支持磁盘加密。 下面的示例使用 **Ubuntu 14.04** 映像创建一个名为 `myVM` 的 VM：

    az vm create -g myResourceGroup -n myVM --image Canonical:UbuntuServer:14.04.3-LTS:latest \
      --admin-username azureuser --ssh-key-value ~/.ssh/id_rsa.pub \
      --use-unmanaged-disk

通过 SSH 连接到你的 VM。 创建分区和文件系统，然后安装数据磁盘。 有关详细信息，请参阅[连接 Linux VM 以安装新磁盘](/documentation/articles/virtual-machines-linux-add-disk/#connect-to-the-linux-vm-to-mount-the-new-disk)。 关闭 SSH 会话。

使用 [az vm encryption enable](https://docs.microsoft.com/zh-cn/cli/azure/vm/encryption#enable) 加密 VM。 下面的示例使用之前的 `ad sp create-for-rbac` 命令中的 `$sp_id` 和 `$sp_password` 变量：

    az vm encryption enable --resource-group myResourceGroup --name myVM \
      --aad-client-id $sp_id \
      --aad-client-secret $sp_password \
      --disk-encryption-keyvault $keyvault_name \
      --key-encryption-key myKey \
      --volume-type all

需要一段时间才能完成磁盘加密过程。 使用 [az vm encryption show](https://docs.microsoft.com/zh-cn/cli/azure/vm/encryption#show) 监视过程状态：

    az vm encryption show --resource-group myResourceGroup --name myVM

状态显示“EncryptionInProgress”。 请等待，直到 OS 磁盘的状态报告“VMRestartPending”，然后使用 [az vm restart](https://docs.microsoft.com/zh-cn/cli/azure/vm#restart) 重启 VM：

    az vm restart --resource-group myResourceGroup --name myVM

磁盘加密过程将在启动过程中完成，因此请等待几分钟后再使用 **az vm encryption show** 查看加密状态：

    az vm encryption show --resource-group myResourceGroup --name myVM

 现在，状态应将 OS 磁盘和数据磁盘均报告为“Encrypted”。

## <a name="overview-of-disk-encryption"></a>磁盘加密概述
Linux VM 上的虚拟磁盘是使用 [dm-crypt](https://wikipedia.org/wiki/Dm-crypt) 静态加密的。 加密 Azure 中的虚拟磁盘不会产生费用。 使用软件保护将加密密钥存储在 Azure 密钥保管库中，或者，可在已通过 FIPS 140-2 级别 2 标准认证的硬件安全模块 (HSM) 中导入或生成密钥。 你可以控制这些加密密钥，以及审核对它们的使用。 这些加密密钥用于加密和解密附加到 VM 的虚拟磁盘。 打开和关闭 VM 时，Azure Active Directory 服务主体将提供一个安全机制用于颁发这些加密密钥。

加密 VM 的过程如下：

1. 在 Azure 密钥保管库中创建加密密钥。
2. 配置可用于加密磁盘的加密密钥。
3. 若要从 Azure Key Vault 中读取加密密钥，可创建具有相应权限的 Azure Active Directory 服务主体。
4. 发出加密虚拟磁盘的命令，指定要使用的 Azure Active Directory 服务主体和相应的加密密钥。
5. Azure Active Directory 服务主体将从 Azure Key Vault 请求所需的加密密钥。
6. 使用提供的加密密钥加密虚拟磁盘。

## <a name="encryption-process"></a>加密过程
磁盘加密依赖于以下附加组件：

* **Azure 密钥保管库** - 用于保护磁盘加密/解密过程中使用的加密密钥和机密。 
    * 可以使用现有的 Azure 密钥保管库（如果有）。 不需要专门使用某个密钥保管库来加密磁盘。
    * 若要将管理边界和密钥可见性隔离开来，可以创建专用的密钥保管库。
* **Azure Active Directory** - 处理所需加密密钥的安全交换，以及对请求的操作执行的身份验证。 
    * 通常，可以使用现有的 Azure Active Directory 实例来容装应用程序。
  * 服务主体提供了安全机制，可用于请求和获取相应的加密密钥。 实际并不需要开发与 Azure Active Directory 集成的应用程序。

## <a name="requirements-and-limitations"></a>要求和限制
磁盘加密支持的方案和要求

* 以下 Linux 服务器 SKU - Ubuntu、CentOS、SUSE、SUSE Linux Enterprise Server (SLES) 和 Red Hat Enterprise Linux。
* 所有资源（例如密钥保管库、存储帐户和 VM）必须在同一个 Azure 区域和订阅中。
* 标准 A、D、DS、G 和 GS 系列 VM。

以下方案目前不支持磁盘加密：

* 基本层 VM。
* 使用经典部署模型创建的 VM。
* 在 Linux VM 上禁用 OS 磁盘加密。
* 在已加密的 Linux VM 上更新加密密钥。

## <a name="create-azure-key-vault-and-keys"></a>创建 Azure Key Vault 和密钥
需要安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 并已使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。 在整个命令示例中，请将所有示例参数替换为自己的名称、位置和密钥值。 以下示例使用 `myResourceGroup`、`myKeyVault`、`myAADApp` 等的约定。

在整个命令示例中，请将所有示例参数替换为自己的名称、位置和密钥值。 以下示例使用 `myResourceGroup`、`myKey`、`myVM` 等的约定。

第一步是创建用于存储加密密钥的 Azure 密钥保管库。 Azure 密钥保管库可以存储能够在应用程序和服务中安全实现的密钥、机密或密码。 对于虚拟磁盘加密，可以使用密钥保管库来存储用于加密或解密虚拟磁盘的加密密钥。 

在 Azure 订阅中使用 [az provider register](https://docs.microsoft.com/zh-cn/cli/azure/provider#register) 启用 Azure Key Vault 提供程序，并使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 创建一个资源组。 以下示例在 `ChinaNorth` 位置创建资源组名称 `myResourceGroup`：

    az provider register -n Microsoft.KeyVault
    az group create --name myResourceGroup --location ChinaNorth

包含加密密钥和关联的计算资源（例如存储和 VM 本身）的 Azure 密钥保管库必须位于同一区域。 使用 [az keyvault create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#create) 创建 Azure Key Vault，并启用 Key Vaul，将其用于磁盘加密。 指定 `keyvault_name` 的唯一 Key Vault 名称，如下所示：

    keyvault_name=myUniqueKeyVaultName
    az keyvault create --name $keyvault_name --resource-group myResourceGroup \
      --location ChinaNorth --enabled-for-disk-encryption true

可以使用软件或硬件安全模型 (HSM) 保护来存储加密密钥。 使用 HSM 时需要高级密钥保管库。 与用于存储受软件保护的密钥的标准密钥保管库不同，创建高级密钥保管库会产生额外的费用。 若要创建高级密钥保管库，请在前一步骤中，将 `--sku Premium` 添加到命令。 由于我们创建的是标准密钥保管库，以下示例使用了受软件保护的密钥。 

对于这两种保护模型，在启动 VM 解密虚拟磁盘时，都需要向 Azure 平台授予请求加密密钥的访问权限。 使用 [az keyvault key create](https://docs.microsoft.com/zh-cn/cli/azure/keyvault/key#create) 在 Key Vault 中创建加密密钥。 以下示例创建名为 `myKey` 的密钥：

    az keyvault key create --vault-name $keyvault_name --name myKey --protection software

## <a name="create-the-azure-active-directory-service-principal"></a>创建 Azure Active Directory 服务主体
加密或解密虚拟磁盘时，将指定一个帐户来处理身份验证，以及从 Key Vault 交换加密密钥。 此帐户（Azure Active Directory 服务主体）允许 Azure 平台代表 VM 请求相应的加密密钥。 订阅中提供了一个默认的 Azure Active Directory 实例，不过，许多组织使用专用的 Azure Active Directory 目录。

通过将 Azure Active Directory 与 [az ad sp create-for-rbac](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#create-for-rbac) 配合使用创建服务主体。 下面的示例读入服务主体 ID 和密码的值，供稍后的命令中使用：

    read sp_id sp_password <<< $(az ad sp create-for-rbac --query [appId,password] -o tsv)

只有创建服务主体时，才会显示密码。 如果需要，可查看并记录密码 (`echo $sp_password`)。 可以使用 [az ad sp list](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#list) 列出服务主体并使用 [az ad sp show](https://docs.microsoft.com/zh-cn/cli/azure/ad/sp#show) 查看特定服务主体的详细信息。

若要成功加密或解密虚拟磁盘，必须将 Key Vault 中存储的加密密钥的权限设置为允许 Azure Active Directory 服务主体读取密钥。 使用 [az keyvault set-policy](https://docs.microsoft.com/zh-cn/cli/azure/keyvault#set-policy) 在 Key Vault 上设置权限。 在下面的示例中，服务主体 ID 由之前的命令提供：

    az keyvault set-policy --name $keyvault_name --spn $sp_id \
      --key-permissions all \
      --secret-permissions all

## <a name="create-virtual-machine"></a>创建虚拟机
为了实际加密一些虚拟磁盘，让我们创建一个 VM 并添加一个数据磁盘。 使用 [az vm create](https://docs.microsoft.com/zh-cn/cli/azure/vm#create) 创建要加密的 VM。 只有特定应用商店映像才支持磁盘加密。 下面的示例使用 **Ubuntu 14.04** 映像创建一个名为 `myVM` 的 VM：

    az vm create -g myResourceGroup -n myVM --image Canonical:UbuntuServer:14.04.3-LTS:latest \
      --use-unmanaged-disk

使用 VM 的 SSH 创建分区和文件系统，然后安装数据磁盘。 有关详细信息，请参阅[连接 Linux VM 以安装新磁盘](/documentation/articles/virtual-machines-linux-add-disk/#connect-to-the-linux-vm-to-mount-the-new-disk)。 关闭 SSH 会话。

## <a name="encrypt-virtual-machine"></a>加密虚拟机
若要加密虚拟磁盘，可将前面的所有组件合并在一起：

1. 指定 Azure Active Directory 服务主体和密码。
2. 指定用于存储加密磁盘元数据的密钥保管库。
3. 指定用于实际加密和解密的加密密钥。
4. 指定是要加密 OS 磁盘、数据磁盘还是所有磁盘。

使用 [az vm encryption enable](https://docs.microsoft.com/zh-cn/cli/azure/vm/encryption#enable) 加密 VM。 下面的示例使用之前的 `ad sp create-for-rbac` 命令中的 `$sp_id` 和 `$sp_password` 变量：

    az vm encryption enable --resource-group myResourceGroup --name myVM \
      --aad-client-id $sp_id \
      --aad-client-secret $sp_password \
      --disk-encryption-keyvault $keyvault_name \
      --key-encryption-key myKey \
      --volume-type all

需要一段时间才能完成磁盘加密过程。 使用 [az vm encryption show](https://docs.microsoft.com/zh-cn/cli/azure/vm/encryption#show) 监视过程状态：

    az vm encryption show --resource-group myResourceGroup --name myVM

输出类似于下面截断的示例：

    [
      "dataDisk": "EncryptionInProgress",
      "osDisk": "EncryptionInProgress",
    ]

请等待，直到 OS 磁盘的状态报告“VMRestartPending”，然后使用 [az vm restart](https://docs.microsoft.com/zh-cn/cli/azure/vm#restart) 重启 VM：

    az vm restart --resource-group myResourceGroup --name myVM

磁盘加密过程将在启动过程中完成，因此请等待几分钟后再使用 **az vm encryption show** 查看加密状态：

    az vm encryption show --resource-group myResourceGroup --name myVM

现在，状态应将 OS 磁盘和数据磁盘均报告为“Encrypted”。

## <a name="add-additional-data-disks"></a>添加更多数据磁盘
加密数据磁盘后，可将更多的虚拟磁盘添加到 VM 并将其加密。 运行 `az vm encryption enable` 命令时，可以使用 `--sequence-version` 参数递增序列版本。 使用此序列版本参数可以针对同一个 VM 重复执行操作。

重新运行上述命令来加密虚拟磁盘，不过这次请添加 `--sequence-version` 参数，以便递增第一次运行中使用的值，如下所示：

    az vm encryption enable --resource-group myResourceGroup --name myVM \
      --aad-client-id $sp_id \
      --aad-client-secret $sp_password \
      --disk-encryption-keyvault $keyvault_name \
      --key-encryption-key myKey \
      --volume-type all \
      --sequence-version 2

## <a name="next-steps"></a>后续步骤
* 有关管理 Azure 密钥保管库的详细信息，包括删除加密密钥和保管库，请参阅 [Manage Key Vault using CLI](/documentation/articles/key-vault-manage-with-cli/)（使用 CLI 管理密钥保管库）。

<!--Update_Description: change to CLI 2.0-->