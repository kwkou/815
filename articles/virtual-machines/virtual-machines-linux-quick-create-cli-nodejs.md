<properties
    pageTitle="使用 Azure CLI 创建 Linux VM | Azure"
    description="使用 Azure CLI 在 Azure 上为 NodeJs 创建 Linux VM。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="vlivech"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="facb1115-2b4e-4ef3-9905-330e42beb686"
    ms.service="virtual-machines-linux"
    ms.devlang="NA"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="12/15/2016"
    wacn.date="04/06/2017"
    ms.author="v-livech" />  


# Create a Linux VM using the Azure CLI（使用 Azure CLI 创建 Linux VM）

本文说明如何使用 Azure 命令行接口 (CLI) 的 `azure vm quick-create` 命令在 Azure 上快速部署 Linux 虚拟机 (VM)。`quick-create` 命令在基本的安全基础结构内部署 VM，用户可以将其用于快速构建某一概念的原型或进行测试。

本文要求满足以下条件：

- [一个 Azure 帐户](/pricing/1rmb-trial/)

- [SSH 公钥和私钥文件](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)

也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)快速部署 Linux VM。

## 快速命令

以下示例演示如何部署 CoreOS VM 并附加安全外壳 (SSH) 密钥（你的参数可能与此不同）：

    azure vm quick-create -M ~/.ssh/id_rsa.pub -Q CoreOS

## 详细演练

下面逐步讲解如何部署 UbuntuLTS VM，并解释每个步骤的具体操作。

## VM quick-create 别名

选择分发的便捷方法是使用映射到最常见 OS 分发的 Azure CLI 别名。下表列出了别名（截止到 Azure 0.10 版）。使用 `quick-create` 的所有部署默认为部署到由固态硬盘 (SSD) 存储提供支持的 VM，这些 VM 提供更快的预配性能和高性能磁盘访问。（这些别名表示 Azure 上的一小部分可用分发。在 Azure 应用商店中查找更多映像（可以[在 PowerShell 中搜索映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)），或者[上载自己的自定义映像](/documentation/articles/virtual-machines-linux-create-upload-generic/)。）

| 别名 | 发布者 | 产品 | SKU | 版本 |
|:--- |:--- |:--- |:--- |:--- |
| CentOS |OpenLogic |CentOS |7\.2 |最新 |
| CoreOS |CoreOS |CoreOS |Stable |最新 |
| Debian |credativ |Debian |8 |最新 |
| openSUSE |SUSE |openSUSE |13\.2 |最新 |
| UbuntuLTS |Canonical |Ubuntu Server |14\.04.3-LTS |最新 |

以下各节对 **ImageURN** 选项 (`-Q`) 使用 `UbuntuLTS` 别名来部署 Ubuntu 14.04.3 LTS Server。

上一个 `quick-create` 示例在禁用 SSH 密码时仅调出 `-M` 标志来标识要上载的 SSH 公钥，因此系统将提示输入以下参数：

* 资源组名称（通常适用于第一个 Azure 资源组的任何字符串）
* VM 名称
* 位置（`chinanorth` 或 `westeurope` 都是合理的默认值）
* linux（为了让 Azure 知道用户需要哪个 OS）
* username

以下示例会指定所有值，因此不会再有进一步的提示。只要使用 `~/.ssh/id_rsa.pub` 作为 ssh-rsa 格式公钥文件，它就会正常运行：

    azure vm quick-create \
      --resource-group myResourceGroup \
      --name myVM \
      --location chinanorth \
      --os-type Linux \
      --admin-username myAdminUser \
      --ssh-publickey-file ~/.ssh/id_rsa.pub \
      --image-urn UbuntuLTS

输出应类似于以下输出块：

    info:    Executing command vm quick-create
    + Listing virtual machine sizes available in the location "chinanorth"
    + Looking up the VM "myVM"
    info:    Verifying the public key SSH file: /Users/ahmet/.ssh/id_rsa.pub
    info:    Using the VM Size "Standard_DS1"
    info:    The [OS, Data] Disk or image configuration requires storage account
    + Looking up the storage account cli16330708391032639673
    + Looking up the NIC "examp-china-1633070839-nic"
    info:    An nic with given name "examp-china-1633070839-nic" not found, creating a new one
    + Looking up the virtual network "examp-china-1633070839-vnet"
    info:    Preparing to create new virtual network and subnet
    / Creating a new virtual network "examp-china-1633070839-vnet" [address prefix: "10.0.0.0/16"] with subnet "examp-china-1633070839-snet" [address prefix: "10.+.1.0/24"]
    + Looking up the virtual network "examp-china-1633070839-vnet"
    + Looking up the subnet "examp-china-1633070839-snet" under the virtual network "examp-china-1633070839-vnet"
    info:    Found public ip parameters, trying to setup PublicIP profile
    + Looking up the public ip "examp-china-1633070839-pip"
    info:    PublicIP with given name "examp-china-1633070839-pip" not found, creating a new one
    + Creating public ip "examp-china-1633070839-pip"
    + Looking up the public ip "examp-china-1633070839-pip"
    + Creating NIC "examp-china-1633070839-nic"
    + Looking up the NIC "examp-china-1633070839-nic"
    + Looking up the storage account clisto1710997031examplev
    + Creating VM "myVM"
    + Looking up the VM "myVM"
    + Looking up the NIC "examp-china-1633070839-nic"
    + Looking up the public ip "examp-china-1633070839-pip"
    data:    Id                              :/subscriptions/2<--snip-->d/resourceGroups/exampleResourceGroup/providers/Microsoft.Compute/virtualMachines/exampleVMName
    data:    ProvisioningState               :Succeeded
    data:    Name                            :exampleVMName
    data:    Location                        :chinanorth
    data:    Type                            :Microsoft.Compute/virtualMachines
    data:
    data:    Hardware Profile:
    data:      Size                          :Standard_DS1
    data:
    data:    Storage Profile:
    data:      Image reference:
    data:        Publisher                   :Canonical
    data:        Offer                       :UbuntuServer
    data:        Sku                         :14.04.3-LTS
    data:        Version                     :latest
    data:
    data:      OS Disk:
    data:        OSType                      :Linux
    data:        Name                        :clic7fadb847357e9cf-os-1473374894359
    data:        Caching                     :ReadWrite
    data:        CreateOption                :FromImage
    data:        Vhd:
    data:          Uri                       :https://cli16330708391032639673.blob.core.chinacloudapi.cn/vhds/clic7fadb847357e9cf-os-1473374894359.vhd
    data:
    data:    OS Profile:
    data:      Computer Name                 :myVM
    data:      User Name                     :myAdminUser
    data:      Linux Configuration:
    data:        Disable Password Auth       :true
    data:
    data:    Network Profile:
    data:      Network Interfaces:
    data:        Network Interface #1:
    data:          Primary                   :true
    data:          MAC Address               :00-0D-3A-33-42-FB
    data:          Provisioning State        :Succeeded
    data:          Name                      :examp-china-1633070839-nic
    data:          Location                  :chinanorth
    data:            Public IP address       :138.91.247.29
    data:            FQDN                    :examp-china-1633070839-pip.chinanorth.chinacloudapp.cn
    data:
    data:    Diagnostics Profile:
    data:      BootDiagnostics Enabled       :true
    data:      BootDiagnostics StorageUri    :https://clisto1710997031examplev.blob.core.chinacloudapi.cn/
    data:
    data:      Diagnostics Instance View:
    info:    vm quick-create command OK

## 登录到新 VM
使用输出中列出的公共 IP 地址登录到 VM。可以使用列出的完全限定域名 (FQDN)：

    ssh -i ~/.ssh/id_rsa.pub ahmet@138.91.247.29

登录过程应如以下输出块所示：

    Warning: Permanently added '138.91.247.29' (ECDSA) to the list of known hosts.
    Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.19.0-65-generic x86_64)

     * Documentation:  https://help.ubuntu.com/

      System information as of Thu Sep  8 22:50:57 UTC 2016

      System load: 0.63              Memory usage: 2%   Processes:       81
      Usage of /:  39.6% of 1.94GB   Swap usage:   0%   Users logged in: 0

      Graph this data and manage this system at:
        https://landscape.canonical.com/

      Get cloud support with Ubuntu Advantage Cloud Guest:
        http://www.ubuntu.com/business/services/cloud

    0 packages can be updated.
    0 updates are security updates.


    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.

    myAdminUser@myVM:~$

## 后续步骤
使用 `azure vm quick-create` 命令可以快速部署 VM，以便可以登录到 bash shell 开始工作。但是，使用 `vm quick-create` 不会为用户提供广泛的控制，也不会让用户创建更复杂的环境。若要部署针对基础结构自定义的 Linux VM，可以遵循下列任一文章操作：

* [Use an Azure Resource Manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
* [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建用户自己的自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
* [Create an SSH Secured Linux VM on Azure using templates（使用模板在 Azure 上创建受 SSH 保护的 Linux VM）](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description: update meta properties & wording update-->