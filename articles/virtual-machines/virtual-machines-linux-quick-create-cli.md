<properties
    pageTitle="使用 Azure CLI 2.0（预览版）创建 Linux VM | Azure"
    description="使用 Azure CLI 2.0（预览版）创建 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter="author: squillace"
    manager="timlt" />  

<tags
    ms.assetid="82005a05-053d-4f52-b0c2-9ae2e51f7a7e"
    ms.service="virtual-machines-linux"
    ms.devlang="NA"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="09/26/2016"
    wacn.date=""
    ms.author="rasquill" />  


# 使用 Azure CLI 2.0（预览版）创建 Linux VM

> [Azure.Note]
在 Azure 中国区使用 Azure CLI 2.0 之前，需要为其设置 Azure 上下文。运行 `az context create --cloud AzureChinaCloud --name AzureChinaCloud` 可以创建 Azure 中国区的 Azure 上下文。运行 `az context switch -n AzureChinaCloud` 可切换到 Azure 中国区环境。如果想要返回全球 Azure，可以运行 `az context switch -n default`

本文说明如何在 Azure CLI 2.0（预览版）中使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 命令在 Azure 上快速部署 Linux 虚拟机 (VM)。

> [AZURE.NOTE] 
Azure CLI 2.0 预览版是下一代的多平台 CLI。欢迎通过 [GitHub 项目页](https://github.com/Azure/azure-cli)试用该软件并提供反馈。
><p>
><p>剩余的文档使用现有的 Azure CLI。若要使用现有 Azure CLI 而不是 CLI 2.0 预览版创建 VM，请参阅 [Create a VM with the Azure CLI](/documentation/articles/virtual-machines-linux-quick-create-cli-nodejs/)（使用 Azure CLI 创建 VM）。

若要创建 VM，需要：

* 一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）
* 已安装 [Azure CLI v.2.0（预览版）](https://github.com/Azure/azure-cli#installation)
* 登录到 Azure 帐户（键入 [az login](https://docs.microsoft.com/cli/azure/#login)）

（也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)快速部署 Linux VM。）

以下示例演示如何部署 Debian VM 和附加安全外壳 (SSH) 密钥（你的参数可能与此不同；如果需要不同的映像，[可以搜索映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)）。

## 创建资源组

首先，键入 [az resource group create](https://docs.microsoft.com/cli/azure/group#create) 创建包含所有已部署资源的资源组：

    az resource group create -n myResourceGroup -l chinanorth

输出如下所示（如果需要，可以选择一个不同的 `--output` 选项）：

    {
      "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
      "location": "chinanorth",
      "name": "myResourceGroup",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null
    }

## 使用最新的 Debian 映像创建 VM

现在，可以创建 VM 及其环境。请记得将 `----public-ip-address-dns-name` 值替换为唯一值；下面的值可能已被使用。

    az vm create \
    --image credativ:Debian:8:latest \
    --admin-username ops \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --public-ip-address-dns-name mydns \
    --resource-group myResourceGroup \
    --location chinanorth \
    --name myVM

输出如下所示。请注意通过 **ssh** 连接到 VM 时使用的 `publicIpAddress` 或 `fqdn` 值。

    {
      "fqdn": "mydns.chinanorth.chinacloudapp.cn",
      "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
      "macAddress": "00-0D-3A-32-05-07",
      "privateIpAddress": "10.0.0.4",
      "publicIpAddress": "40.112.217.29",
      "resourceGroup": "myResourceGroup"
    }

使用输出中列出的公共 IP 地址登录到 VM。可以使用列出的完全限定域名 (FQDN)。

    ssh ops@mydns.chinanorth.chinacloudapp.cn

根据所选的分发版，应会显示类似于下面的输出：

    The authenticity of host 'mydns.chinanorth.chinacloudapp.cn (40.112.217.29)' can't be established.
    RSA key fingerprint is SHA256:xbVC//lciRvKild64lvup2qIRimr/GB8C43j0tSHWnY.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'mydns.chinanorth.chinacloudapp.cn,40.112.217.29' (RSA) to the list of known hosts.

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    ops@mynewvm:~$ ls /
    bin  boot  dev  etc  home  initrd.img  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  vmlinuz

## 后续步骤
使用 `az vm create` 命令可以快速部署 VM，以便可以登录到 bash shell 开始工作。但是，使用 `az vm create` 不会为用户提供广泛的控制，也不会让用户创建更复杂的环境。若要部署针对基础结构自定义的 Linux VM，可以遵循下列任一文章操作：

* [Use an Azure Resource Manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建用户自己的自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [Create an SSH Secured Linux VM on Azure using templates（使用模板在 Azure 上创建受 SSH 保护的 Linux VM）](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

如果使用 Java，请尝试 [create()](https://docs.microsoft.com/java/api/com.microsoft.azure.management.compute._virtual_machine) 方法。

<!---HONumber=Mooncake_1212_2016-->