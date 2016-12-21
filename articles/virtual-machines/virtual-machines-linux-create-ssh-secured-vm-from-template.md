<properties
    pageTitle="使用 Azure 模板创建 Linux VM | Azure"
    description="使用 Azure Resource Manager 模板在 Azure 上创建 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager" />  

<tags
    ms.assetid="721b8378-9e47-411e-842c-ec3276d3256a"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.date="10/24/2016"
    wacn.date="12/20/2016"
    ms.author="v-livech" />  


# 使用 Azure 模板创建 Linux VM
本文说明如何使用 Azure 模板在 Azure 上快速部署 Linux 虚拟机。本文要求满足以下条件：

* 一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）
* 已使用 `azure login -e AzureChinaCloud` 登录 [Azure CLI](/documentation/articles/xplat-cli-install/)。
* Azure CLI *必须处于* Azure Resource Manager 模式`azure config mode arm`。

也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)快速部署 Linux VM 模板。

## 快速命令摘要

    azure group create \
        -n myResourceGroup \
        -l chinanorth \
        --template-file /path/to/azuredeploy.json

## 详细演练
使用模板可以在 Azure 上创建具有所需设置的 VM，这些设置可在启动过程中自定义，例如用户名和主机名。对于本文，我们将推出一个利用 Ubuntu VM 和网络安全组 (NSG) 并打开端口 22 用于 SSH 的 Azure 模板。

Azure Resource Manager 模板是可用于一次性简易任务（如启动 Ubuntu VM）的 JSON 文件，如本文所述。Azure 模板还可用于构造整个环境的复杂 Azure 配置，如测试、开发或生产部署堆栈。

## 创建 Linux VM
下面的代码示例演示如何调用 `azure group create` 创建资源组，并同时使用[此 Azure Resource Manager 模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json)部署受 SSH 保护的 Linux VM。请记住，在示例中必须使用环境中唯一的名称。此示例使用 `myResourceGroup` 作为资源组名称，使用 `myVM` 作为 VM 名称。

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

    azure group create \
        --name myResourceGroup \
        --location chinanorth \
        --template-file /path/to/azuredeploy.json

输出应类似于以下输出块：

    info:    Executing command group create
    + Getting resource group myResourceGroup
    + Creating resource group myResourceGroup
    info:    Created resource group myResourceGroup
    info:    Supply values for the following parameters
    sshKeyData: ssh-rsa AAAAB3Nza<..ssh public key text..>VQgwjNjQ== myAdminUser@myVM
    + Initializing template configurations and parameters
    + Creating a deployment
    info:    Created template deployment "azuredeploy"
    data:    Id:                  /subscriptions/<..subid text..>/resourceGroups/myResourceGroup
    data:    Name:                myResourceGroup
    data:    Location:            chinanorth
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK

该示例使用 `--template-file` 参数（模板文件的路径为自变量）部署了一个 VM。如果你确信该模板适用于 Azure 中国区，也可以使用 `--template-uri` 直接从 github 原始文件部署该模板。Azure CLI 将提示你输入模板所需的参数。

## 后续步骤
搜索[模板库](https://github.com/Azure/azure-quickstart-templates/)以发现下一步要部署哪些应用框架。

<!---HONumber=Mooncake_1212_2016-->