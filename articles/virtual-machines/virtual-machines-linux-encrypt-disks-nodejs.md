<properties
    pageTitle="使用 Azure CLI 1.0 在 Linux VM 上加密磁盘 | Azure"
    description="如何使用 Azure CLI 1.0 和 Resource Manager 部署模型加密 Linux VM 中的磁盘"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="03/06/2017"
    wacn.date="05/15/2017"
    ms.author="iainfou"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="2e6cc0fd7dc162ba06f1581e9ca7919f13b59673"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="encrypt-disks-on-a-linux-vm-using-the-azure-cli-10"></a>使用 Azure CLI 1.0 加密 Linux VM 中的磁盘
为了增强虚拟机 (VM) 的安全性以及遵从法规，可以静态加密 Azure 中的虚拟磁盘。 磁盘是使用 Azure 密钥保管库中受保护的加密密钥加密的。 可以控制这些加密密钥，以及审核对它们的使用。 本文详细说明如何使用 Azure CLI 1.0 和 Resource Manager 部署模型加密 Linux VM 中的虚拟磁盘。

## <a name="cli-versions-to-complete-the-task"></a>用于完成任务的 CLI 版本
可使用以下 CLI 版本之一完成任务：

- [Azure CLI 1.0](#quick-commands) - 适用于经典部署模型和资源管理部署模型（本文）的 CLI
- [Azure CLI 2.0](/documentation/articles/virtual-machines-linux-encrypt-disks/) - 适用于资源管理部署模型的下一代 CLI

## <a name="quick-commands"></a>快速命令
如果需要快速完成任务，请参阅以下部分，其中详细说明了用于加密 VM 中虚拟磁盘的基本命令。 本文档的余下部分（[从此处开始](#overview-of-disk-encryption)）提供了每个步骤的更详细信息和上下文。

需要安装[最新的 Azure CLI 1.0](/documentation/articles/xplat-cli-install/)，然后按如下所示，使用 Resource Manager 模式登录：

    azure config mode arm

在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 `myResourceGroup`、`myKeyVault` 和 `myVM`。

首先，在 Azure 订阅中启用 Azure 密钥保管库提供程序，并创建一个资源组。 以下示例在 `ChinaNorth` 位置创建资源组名称 `myResourceGroup`：

    azure provider register Microsoft.KeyVault
    azure group create myResourceGroup --location ChinaNorth

创建 Azure 密钥保管库。 以下示例创建名为 `myKeyVault`的密钥保管库：

    azure keyvault create --vault-name myKeyVault --resource-group myResourceGroup \
      --location ChinaNorth

在密钥保管库中创建加密密钥，并启用该密钥进行加密磁盘。 以下示例创建名为 `myKey`的密钥：

    azure keyvault key create --vault-name myKeyVault --key-name myKey \
      --destination software
    azure keyvault set-policy --vault-name myKeyVault --resource-group myResourceGroup \
      --enabled-for-disk-encryption true

使用 Azure Active Directory 创建一个终结点，用于处理身份验证以及从密钥保管库交换加密密钥。 `--home-page` 和 `--identifier-uris` 不需要是实际的可路由地址。 为实现高级别的安全性，应使用客户端机密而不是密码。 Azure CLI 目前无法生成客户端机密。 只能在 Azure 门户中生成客户端机密。 以下示例创建名为 `myAADApp` 的 Azure Active Directory 终结点，并使用密码 `myPassword`。 指定自己的密码，如下所示：

    azure ad app create --name myAADApp \
      --home-page http://testencrypt.contoso.com \
      --identifier-uris http://testencrypt.contoso.com \
      --password myPassword

请注意上述命令的输出中显示的 `applicationId` 。 在以下步骤中使用此应用程序 ID：

    azure ad sp create --applicationId myApplicationID
    azure keyvault set-policy --vault-name myKeyVault --spn myApplicationID \
      --perms-to-keys [\"all\"] --perms-to-secrets [\"all\"]

将数据磁盘添加到现有 VM。 以下示例将数据磁盘添加到名为 `myVM`的 VM：

    azure vm disk attach-new --resource-group myResourceGroup --vm-name myVM \
      --size-in-gb 5

检查密钥保管库和所创建的密钥的详细信息。 在最后一个步骤中，需要使用密钥保管库 ID、URI 和密钥 URL。 以下示例检查名为 `myKeyVault` 的密钥保管库和名为 `myKey` 的密钥的详细信息：

    azure keyvault show myKeyVault
    azure keyvault key show myKeyVault myKey

按如下所示加密磁盘，在整个过程中请输入你自己的参数名称：

    azure vm enable-disk-encryption --resource-group myResourceGroup --name myVM \
      --aad-client-id myApplicationID --aad-client-secret myApplicationPassword \
      --disk-encryption-key-vault-url myKeyVaultVaultURI \
      --disk-encryption-key-vault-id myKeyVaultID \
      --key-encryption-key-url myKeyKID \
      --key-encryption-key-vault-id myKeyVaultID \
      --volume-type Data

在加密过程中，Azure CLI 不提供详细错误。 有关其他故障排除信息，请查看 `/var/log/azure/Microsoft.OSTCExtensions.AzureDiskEncryptionForLinux/0.x.x.x/extension.log`。 由于上述命令包含许多变量，你可能不太明白为何进程会失败，为此，我们提供了如下所示的完整命令示例：

    azure vm enable-disk-encryption --resource-group myResourceGroup --name myVM \
      --aad-client-id 147bc426-595d-4bad-b267-58a7cbd8e0b6 \
      --aad-client-secret P@ssw0rd! \
      --disk-encryption-key-vault-url https://myKeyVault.vault.azure.cn/ \ 
      --disk-encryption-key-vault-id /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myKeyVault \
      --key-encryption-key-url https://myKeyVault.vault.azure.cn/keys/myKey/6f5fe9383f4e42d0a41553ebc6a82dd1 \
      --key-encryption-key-vault-id /subscriptions/guid/resourceGroups/myResoureGroup/providers/Microsoft.KeyVault/vaults/myKeyVault \
      --volume-type Data

最后，再次检查加密状态，确认虚拟磁盘现在是否已加密。 以下示例检查 `myResourceGroup` 资源组中名为 `myVM` 的 VM 的状态：

    azure vm show-disk-encryption-status --resource-group myResourceGroup --name myVM

## <a name="overview-of-disk-encryption"></a>磁盘加密概述
Linux VM 上的虚拟磁盘是使用 [dm-crypt](https://wikipedia.org/wiki/Dm-crypt) 静态加密的。 加密 Azure 中的虚拟磁盘不会产生费用。 使用软件保护将加密密钥存储在 Azure 密钥保管库中，或者，可在已通过 FIPS 140-2 级别 2 标准认证的硬件安全模块 (HSM) 中导入或生成密钥。 你可以控制这些加密密钥，以及审核对它们的使用。 这些加密密钥用于加密和解密附加到 VM 的虚拟磁盘。 打开和关闭 VM 时，Azure Active Directory 终结点将提供一个安全机制用于颁发这些加密密钥。

加密 VM 的过程如下：

1. 在 Azure 密钥保管库中创建加密密钥。
2. 配置可用于加密磁盘的加密密钥。
3. 若要从 Azure 密钥保管库中读取加密密钥，可以使用 Azure Active Directory 以适当的权限创建一个终结点。
4. 发出加密虚拟磁盘的命令，指定要使用的 Azure Active Directory 终结点和相应的加密密钥。
5. Azure Active Directory 终结点将从 Azure 密钥保管库请求所需的加密密钥。
6. 使用提供的加密密钥加密虚拟磁盘。

## <a name="supporting-services-and-encryption-process"></a>支持服务和加密过程
磁盘加密依赖于以下附加组件：

* **Azure 密钥保管库** - 用于保护磁盘加密/解密过程中使用的加密密钥和机密。 
    * 可以使用现有的 Azure 密钥保管库（如果有）。 不需要专门使用某个密钥保管库来加密磁盘。
    * 若要将管理边界和密钥可见性隔离开来，可以创建专用的密钥保管库。
* **Azure Active Directory** - 处理所需加密密钥的安全交换，以及对请求的操作执行的身份验证。 
    * 通常，可以使用现有的 Azure Active Directory 实例来容装应用程序。 
    * 应用程序更多地充当密钥保管库和虚拟机服务的终结点，请求并获取相应的加密密钥。 实际并不需要开发与 Azure Active Directory 集成的应用程序。

## <a name="requirements-and-limitations"></a>要求和限制
磁盘加密支持的方案和要求

* 以下 Linux 服务器 SKU - Ubuntu、CentOS、SUSE、SUSE Linux Enterprise Server (SLES) 和 Red Hat Enterprise Linux。
* 所有资源（例如密钥保管库、存储帐户和 VM）必须在同一个 Azure 区域和订阅中。
* 标准 A、D 和 DS 系列 VM。

以下方案目前不支持磁盘加密：

* 基本层 VM。
* 使用经典部署模型创建的 VM。
* 在 Linux VM 上禁用 OS 磁盘加密。
* 在已加密的 Linux VM 上更新加密密钥。

## <a name="create-the-azure-key-vault-and-keys"></a>创建 Azure 密钥保管库和密钥
若要安装本指南的余下部分，需要安装[最新的 Azure CLI 1.0](/documentation/articles/xplat-cli-install/)，然后按如下所示，使用 Resource Manager 模式登录：

    azure config mode arm

在整个命令示例中，请将所有示例参数替换为自己的名称、位置和密钥值。 以下示例使用 `myResourceGroup`、`myKeyVault`、`myAADApp` 等的约定。

第一步是创建用于存储加密密钥的 Azure 密钥保管库。 Azure 密钥保管库可以存储能够在应用程序和服务中安全实现的密钥、机密或密码。 对于虚拟磁盘加密，可以使用密钥保管库来存储用于加密或解密虚拟磁盘的加密密钥。 

在 Azure 订阅中启用 Azure 密钥保管库提供程序，然后创建一个资源组。 以下示例在 `ChinaNorth` 位置创建名为 `myResourceGroup` 的资源组：

    azure provider register Microsoft.KeyVault
    azure group create myResourceGroup --location ChinaNorth

包含加密密钥和关联的计算资源（例如存储和 VM 本身）的 Azure 密钥保管库必须位于同一区域。 以下示例创建名为 `myKeyVault` 的 Azure 密钥保管库：

    azure keyvault create --vault-name myKeyVault --resource-group myResourceGroup \
      --location ChinaNorth

可以使用软件或硬件安全模型 (HSM) 保护来存储加密密钥。 使用 HSM 时需要高级密钥保管库。 与用于存储受软件保护的密钥的标准密钥保管库不同，创建高级密钥保管库会产生额外的费用。 若要创建高级密钥保管库，请在前一步骤中，将 `--sku Premium` 添加到命令。 由于我们创建的是标准密钥保管库，以下示例使用了受软件保护的密钥。 

对于这两种保护模型，在启动 VM 解密虚拟磁盘时，都需要向 Azure 平台授予请求加密密钥的访问权限。 在密钥保管库中创建加密密钥，然后启用该密钥，以便进行虚拟磁盘加密。 以下示例创建名为 `myKey` 的密钥，然后为磁盘加密启用该密钥：

    azure keyvault key create --vault-name myKeyVault --key-name myKey \
      --destination software
    azure keyvault set-policy --vault-name myKeyVault --resource-group myResourceGroup \
      --enabled-for-disk-encryption true

## <a name="create-the-azure-active-directory-application"></a>创建 Azure Active Directory 应用程序
加密或解密虚拟磁盘时，将使用一个终结点来处理身份验证，以及从密钥保管库交换加密密钥。 此终结点（Azure Active Directory 应用程序）允许 Azure 平台代表 VM 请求相应的加密密钥。 订阅中提供了一个默认的 Azure Active Directory 实例，不过，许多组织使用专用的 Azure Active Directory 目录。

由于我们不需要创建完整的 Azure Active Directory 应用程序，以下示例中的 `--home-page` 和 `--identifier-uris` 参数不需要是实际的可路由地址。 此外，以下示例还指定基于密码的机密，而不是在 Azure 门户中生成密钥。 目前，无法从 Azure CLI 生成密钥。 

创建 Azure Active Directory 应用程序。 以下示例创建名为 `myAADApp` 的应用程序并使用密码 `myPassword`。 指定自己的密码，如下所示：

    azure ad app create --name myAADApp \
      --home-page http://testencrypt.contoso.com \
      --identifier-uris http://testencrypt.contoso.com \
      --password myPassword

记下上述命令的输出中返回的 `applicationId` 。 余下的某些步骤会用到此应用程序 ID。 接下来，创建服务主体名称 (SPN)，以便能够从环境内部访问该应用程序。 若要成功加密或解密虚拟磁盘，必须将密钥保管库中存储的加密密钥的权限设置为允许 Azure Active Directory 应用程序读取密钥。 

创建 SPN，然后按如下所示设置相应的权限：

    azure ad sp create --applicationId myApplicationID
    azure keyvault set-policy --vault-name myKeyVault --spn myApplicationID \
      --perms-to-keys [\"all\"] --perms-to-secrets [\"all\"]

## <a name="add-a-virtual-disk-and-review-encryption-status"></a>添加虚拟磁盘并检查加密状态
为了实践加密一些虚拟磁盘，让我们在现有 VM 中添加一个磁盘。 按如下所示，将一个 5Gb 的数据磁盘添加到现有 VM：

    azure vm disk attach-new --resource-group myResourceGroup --vm-name myVM \
      --size-in-gb 5

这些虚拟磁盘当前未加密。 按如下所示检查 VM 的当前加密状态：

    azure vm show-disk-encryption-status --resource-group myResourceGroup --name myVM

## <a name="encrypt-virtual-disks"></a>加密虚拟磁盘
若要立即加密虚拟磁盘，可将前面的所有组件合并在一起：

1. 指定 Azure Active Directory 应用程序和密码。
2. 指定用于存储加密磁盘元数据的密钥保管库。
3. 指定用于实际加密和解密的加密密钥。
4. 指定是要加密 OS 磁盘、数据磁盘还是所有磁盘。

让我们查看 Azure 密钥保管库和所创建的密钥的详细信息，因为最后一个步骤需要先后用到密钥保管库 ID、URI 和密钥 URL：

    azure keyvault show myKeyVault
    azure keyvault key show myKeyVault myKey

按如下所示，使用 `azure keyvault show` 和 `azure keyvault key show` 命令的输出加密虚拟磁盘：

    azure vm enable-disk-encryption --resource-group myResourceGroup --name myVM \
      --aad-client-id myApplicationID --aad-client-secret myApplicationPassword \
      --disk-encryption-key-vault-url myKeyVaultVaultURI \
      --disk-encryption-key-vault-id myKeyVaultID \
      --key-encryption-key-url myKeyKID \
      --key-encryption-key-vault-id myKeyVaultID \
      --volume-type Data

由于上述命令包含许多变量，下面提供了一个完整的命令示例供参考：

    azure vm enable-disk-encryption --resource-group myResourceGroup --name myVM \
      --aad-client-id 147bc426-595d-4bad-b267-58a7cbd8e0b6 \
      --aad-client-secret P@ssw0rd! \
      --disk-encryption-key-vault-url https://myKeyVault.vault.azure.cn/ \ 
      --disk-encryption-key-vault-id /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myKeyVault \
      --key-encryption-key-url https://myKeyVault.vault.azure.cn/keys/myKey/6f5fe9383f4e42d0a41553ebc6a82dd1 \
      --key-encryption-key-vault-id /subscriptions/guid/resourceGroups/myResoureGroup/providers/Microsoft.KeyVault/vaults/myKeyVault \
      --volume-type Data

在加密过程中，Azure CLI 不提供详细错误。 有关其他故障排除信息，请在所加密的 VM 上查看 `/var/log/azure/Microsoft.OSTCExtensions.AzureDiskEncryptionForLinux/0.x.x.x/extension.log` 。

最后，让我们再次检查加密状态，确认虚拟磁盘现在是否已加密。

    azure vm show-disk-encryption-status --resource-group myResourceGroup --name myVM

## <a name="add-additional-data-disks"></a>添加更多数据磁盘
加密数据磁盘后，可将更多的虚拟磁盘添加到 VM 并将其加密。 运行 `azure vm enable-disk-encryption` 命令时，可以使用 `--sequence-version` 参数递增序列版本。 使用此序列版本参数可以针对同一个 VM 重复执行操作。

例如，按如下所示将另一个虚拟磁盘添加到 VM：

    azure vm disk attach-new --resource-group myResourceGroup --vm-name myVM \
      --size-in-gb 5

重新运行上述命令来加密虚拟磁盘，不过这次请添加 `--sequence-version` 参数，以便递增第一次运行中使用的值，如下所示：

    azure vm enable-disk-encryption --resource-group myResourceGroup --name myVM \
      --aad-client-id myApplicationID --aad-client-secret myApplicationPassword \
      --disk-encryption-key-vault-url myKeyVaultVaultURI \
      --disk-encryption-key-vault-id myKeyVaultID \
      --key-encryption-key-url myKeyKID \
      --key-encryption-key-vault-id myKeyVaultID \
      --volume-type Data
      --sequence-version 2

## <a name="next-steps"></a>后续步骤
* 有关管理 Azure 密钥保管库的详细信息，包括删除加密密钥和保管库，请参阅[使用 CLI 管理密钥保管库](/documentation/articles/key-vault-manage-with-cli/)。
