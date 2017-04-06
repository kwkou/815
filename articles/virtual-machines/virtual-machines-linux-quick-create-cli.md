<properties
    pageTitle="使用 Azure CLI 2.0（预览版）创建 Linux VM | Azure"
    description="使用 Azure CLI 2.0（预览版）创建 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="squillace"
    manager="timlt"
    editor="" />
<tags 
    ms.assetid="82005a05-053d-4f52-b0c2-9ae2e51f7a7e"
    ms.service="virtual-machines-linux"
    ms.devlang="NA"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="01/13/2016"
    wacn.date="03/24/2017"
    ms.author="rasquill" />

# 使用 Azure CLI 2.0 预览版 (az.py) 创建 Linux VM
本文介绍如何通过托管磁盘和本机存储帐户中的磁盘在 Azure CLI 2.0（预览版）中使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#create) 命令在 Azure 上快速部署 Linux 虚拟机 (VM)。

> [AZURE.NOTE] 
> Azure CLI 2.0 预览版是下一代的多平台 CLI。[试用。](https://docs.microsoft.com/cli/azure/install-az-cli2)
><p>
> 若要使用现有 Azure CLI 1.0 而不是 Azure CLI 2.0 预览版创建 VM，请参阅[使用 Azure CLI 创建 VM](/documentation/articles/virtual-machines-linux-quick-create-cli-nodejs/)。

若要创建 VM，需要：

* 一个 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）
* 已安装 [Azure CLI v.2.0（预览版）](https://docs.microsoft.com/cli/azure/install-az-cli2)
* 登录到 Azure 帐户（键入 [az login](https://docs.microsoft.com/cli/azure/#login)）

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

（也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)部署 Linux VM。）

以下示例演示了如何部署 Debian VM 并使用安全外壳 (SSH) 密钥与其进行连接。你的参数可能有所不同；如果需要其他映像，[可搜索一个](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。

## 使用托管磁盘

若要使用 Azure 托管磁盘，必须使用支持此类磁盘的区域。首先，键入 [az group create](https://docs.microsoft.com/cli/azure/group#create) 创建包含所有已部署资源的资源组：

     az group create -n myResourceGroup -l chinanorth

输出如下所示（如果需要，可以指定一个不同的 `--output` 选项来查看其他格式）：

    {
      "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
      "location": "chinanorth",
      "managedBy": null,
      "name": "myResourceGroup",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null
    }

### 创建 VM 
现在，可以创建 VM 及其环境。请记得将 `--public-ip-address-dns-name` 值替换为唯一值；下面的值可能已被使用。

    az vm create \
    --image credativ:Debian:8:latest \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --public-ip-address-dns-name manageddisks \
    --resource-group myResourceGroup \
    --location chinanorth \
    --name myVM

输出如下所示。请注意通过 **ssh** 连接到 VM 时使用的 `publicIpAddress` 或 `fqdn` 值。

    {
      "fqdn": "manageddisks.chinanorth.chinacloudapp.cn",
      "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
      "macAddress": "00-0D-3A-32-E9-41",
      "privateIpAddress": "10.0.0.4",
      "publicIpAddress": "104.42.127.53",
      "resourceGroup": "myResourceGroup"
    }

使用输出中列出的公共 IP 地址或完全限定域名 (FQDN) 登录到 VM。

    ssh ops@manageddisks.chinanorth.chinacloudapp.cn

根据所选的分发版，应会显示类似于下面的输出：

    The authenticity of host 'manageddisks.chinanorth.chinacloudapp.cn (134.42.127.53)' can't be established.
    RSA key fingerprint is c9:93:f5:21:9e:33:78:d0:15:5c:b2:1a:23:fa:85:ba.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'manageddisks.chinanorth.chinacloudapp.cn' (RSA) to the list of known hosts.
    Enter passphrase for key '/home/ops/.ssh/id_rsa':

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Fri Jan 13 14:44:21 2017 from net-37-117-240-123.cust.vodafonedsl.it
    ops@myVM:~$ 

请参阅[后续步骤](#next-steps)，了解可以使用托管磁盘通过新 VM 完成的其他事项。

## 使用非托管磁盘 

使用非托管存储磁盘的 VM 具有非托管存储帐户。首先请键入 [az group create](https://docs.microsoft.com/cli/azure/group#create)，创建包含所有已部署资源的资源组：

    az group create --name nativedisks --location chinanorth

输出如下所示（如果需要，可以选择一个不同的 `--output` 选项）：

    {
      "id": "/subscriptions/<guid>/resourceGroups/nativedisks",
      "location": "chinanorth",
      "managedBy": null,
      "name": "nativedisks",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null
    }

### 创建 VM 

现在，可以创建 VM 及其环境。请记得将 `--public-ip-address-dns-name` 值替换为唯一值；下面的值可能已被使用。

    az vm create \
    --image credativ:Debian:8:latest \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --public-ip-address-dns-name nativedisks \
    --resource-group nativedisks \
    --location chinanorth \
    --name myVM \
    --use-native-disk

输出如下所示。请注意通过 **ssh** 连接到 VM 时使用的 `publicIpAddress` 或 `fqdn` 值。

    {
      "fqdn": "nativedisks.chinanorth.chinacloudapp.cn",
      "id": "/subscriptions/<guid>/resourceGroups/nativedisks/providers/Microsoft.Compute/virtualMachines/myVM",
      "macAddress": "00-0D-3A-33-24-3C",
      "privateIpAddress": "10.0.0.4",
      "publicIpAddress": "13.91.91.195",
      "resourceGroup": "nativedisks"
    }

使用在上述输出中列出的公共 IP 地址或完全限定域名 (FQDN) 登录到 VM。

    ssh ops@nativedisks.chinanorth.chinacloudapp.cn

根据所选的分发版，应会显示类似于下面的输出：

    The authenticity of host 'nativedisks.chinanorth.chinacloudapp.cn (13.91.93.195)' can't be established.
    RSA key fingerprint is 3f:65:22:b9:07:c9:ef:7f:8c:1b:be:65:1e:86:94:a2.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'nativedisks.chinanorth.chinacloudapp.cn,13.91.93.195' (RSA) to the list of known hosts.
    Enter passphrase for key '/home/ops/.ssh/id_rsa':

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    ops@myVM:~$ ls /
    bin  boot  dev  etc  home  initrd.img  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  vmlinuz

## <a name="next-steps"></a> 后续步骤
使用 `az vm create` 命令可以快速部署 VM，以便可以登录到 bash shell 开始工作。但是，使用 `az vm create` 不会为用户提供广泛的控制，也不会让用户创建更复杂的环境。若要部署针对基础结构自定义的 Linux VM，可以遵循下列任一文章操作：

* [Use an Azure Resource Manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建用户自己的自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [Create an SSH Secured Linux VM on Azure using templates（使用模板在 Azure 上创建受 SSH 保护的 Linux VM）](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

<!---HONumber=Mooncake_0320_2017-->