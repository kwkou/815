<properties
    pageTitle="在 Azure 中创建 Linux VM 的不同方式 | Azure"
    description="介绍在 Azure 上创建 Linux 虚拟机的不同方法，并提供每种方法的工具和教程的链接。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="iainfoulds"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="f38f8a44-6c88-4490-a84a-46388212d24c"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure-services"
    ms.date="01/03/2016"
    wacn.date="04/10/2017"
    ms.author="iainfou" />

# 创建 Linux VM 的不同方式，包括 Azure CLI 2.0（预览版）
在 Azure 中，可以使用习惯的工具和工作流灵活创建 Linux 虚拟机 (VM)。本文汇总了这些方法的差异，并举例说明如何创建 Linux VM。

## Azure CLI
可使用以下 CLI 版本之一在 Azure 中创建 VM：

- [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-creation-choices-nodejs/)：用于经典部署模型和资源管理部署模型的 CLI
- Azure CLI 2.0（预览版）- 用于资源管理部署模型（本文）的下一代 CLI

[Azure CLI 2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)可通过 npm 包、发行版所提供的程序包或 Docker 容器跨平台使用。为环境安装最适合的内部版本，然后使用 [az login](https://docs.microsoft.com/cli/azure/#login) 登录到 Azure 帐户

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

以下示例使用 Azure CLI 2.0（预览版）。阅读每篇文章，详细了解所示命令。也可查找相关示例，了解使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-creation-choices-nodejs/) 时的 Linux 创建选项。

* [使用 Azure CLI 2.0（预览版）创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)
  
    * 以下示例使用 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建名为 `myResourceGroup` 的资源组：

          az group create --name myResourceGroup --location chinanorth

    * 以下示例使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create)、最新 Debian 映像、Azure 非托管磁盘以及名为 `id_rsa.pub` 的公钥创建名为 `myVM` 的 VM：

            az vm create \
            --image credativ:Debian:8:latest \
            --admin-username azureuser \
            --ssh-key-value ~/.ssh/id_rsa.pub \
            --public-ip-address-dns-name myPublicDNS \
            --resource-group myResourceGroup \
            --location chinanorth \
            --name myVM \
            --use-unmanaged-disk

* [使用 Azure 模板创建受保护的 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
  
    * 以下示例使用 [az group deployment create](https://docs.microsoft.com/cli/azure/group/deployment#create) 通过存储在 GitHub 上的模板创建 VM：

          az group deployment create --resource-group myResourceGroup \ 
            --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json \
            --parameters @myparameters.json

* [使用 Azure CLI 创建完整的 Linux 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)
  
    * 包括在可用性集中创建负载均衡器和多个 VM。

* [将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)

## Azure 门户预览
在 [Azure 门户预览](https://portal.azure.cn)中可以快速创建 VM，因为不需要在系统上安装任何组件。使用 Azure 门户预览创建 VM：

* [使用 Azure 门户预览创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
* [使用 Azure 门户预览附加磁盘](/documentation/articles/virtual-machines-linux-attach-disk-portal/)

## 操作系统和映像选项
创建 VM 时，可根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括预安装的应用程序和工具。你也可以上载自己的某个映像（请参阅[下一部分](#use-your-own-image)）。

### Azure 映像
使用 [az vm image](https://docs.microsoft.com/cli/azure/vm/image) 命令可按发布者、发行版本和内部版本查看可用内容。

列出可用的发布者：

    az vm image list-publishers --location ChinaNorth

列出给定发布者的可用产品（服务）：

    az vm image list-offers --publisher-name Canonical --location ChinaNorth

列出给定产品/服务的可用 SKU（发行版本）：

    az vm image list-skus --publisher-name Canonical --offer UbuntuServer --location ChinaNorth

列出给定发行版的所有可用映像：

    az vm image list --publisher Canonical --offer UbuntuServer --sku 16.04.0-LTS --location ChinaNorth

有关浏览和使用可用映像的更多示例，请参阅[使用 Azure CLI 导航并选择 Azure 虚拟机映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。

**az vm create** 命令具有一些别名，可用于快速访问较常见的分发版及其最新版本。使用别名通常比每次创建 VM 时指定发布者、产品、SKU 和版本更加快捷：

| 别名 | 发布者 | 产品 | SKU | 版本 |
|:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |Centos |7\.2 |最新 |
| CoreOS |CoreOS |CoreOS |Stable |最新 |
| Debian |credativ |Debian |8 |最新 |
| openSUSE |SUSE |openSUSE |13\.2 |最新 |
| RHEL |Redhat |RHEL |7\.2 |最新 |
| SLES |SLES |SLES |12-SP1 |最新 |
| UbuntuLTS |Canonical |UbuntuServer |14\.04.3-LTS |最新 |

### <a name="use-your-own-image"></a> 使用你自己的映像
若要进行具体的自定义，可以通过*捕获*现有 Azure VM 来使用基于该 VM 的映像。也可以上载本地创建的映像。有关受支持的发行版以及如何使用你自己的映像的详细信息，请参阅以下文章：

* [Azure endorsed distributions（Azure 认可的分发版）](/documentation/articles/virtual-machines-linux-endorsed-distros/)
* [Information for non-endorsed distributions（有关未认可分发版的信息）](/documentation/articles/virtual-machines-linux-create-upload-generic/)
* [How to capture a Linux virtual machine as a Resource Manager template](/documentation/articles/virtual-machines-linux-capture-image/)（如何捕获用作 Resource Manager 模板的 Linux 虚拟机）。
  
    * 快速入门 **az vm** 示例命令，可捕获使用非托管磁盘的现有 VM：

          az vm deallocate --resource-group myResourceGroup --name myVM
          az vm generalize --resource-group myResourceGroup --name myVM
          az vm capture --resource-group myResourceGroup --name myVM --vhd-name-prefix myCapturedVM

## 后续步骤
* 通过[门户](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 [Azure Resource Manager 模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)创建 Linux VM。
* 创建 Linux VM 后[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。
* [重置密码或 SSH 密钥和管理用户](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)的快速步骤

<!---HONumber=Mooncake_0320_2017-->