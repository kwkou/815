<!-- ARM: tested -->

<properties
   pageTitle="使用 CLI 在 Azure 上创建 Linux VM | Azure"
   description="使用 CLI 在 Azure 上创建 Linux VM。"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="vlivech"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/03/2016"
	wacn.date="06/27/2016"/>


# 使用 CLI 在 Azure 上创建 Linux VM

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

本文说明如何使用 Azure CLI 的 `azure vm quick-create` 命令在 Azure 上快速部署 Linux 虚拟机。`quick-create` 命令部署周围具有基本基础结构的 VM，可让你快速创建原型或测试概念（可以将它视为实现 Linux bash shell 的最快方式）。本文中的操作需要一个 Azure 帐户（[获取试用帐户](/pricing/1rmb-trial/)）、已登录 (`azure login -e AzureChinaCloud`) 且处于 Resource Manager 模式 (`azure config mode arm`) 的 [Azure CLI](/documentation/articles/xplat-cli-install/)。也可以使用 [Azure 门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)快速部署 Linux VM。

## 快速命令摘要

使用一个命令来部署 CoreOS VM 并附加 SSH 密钥：

	azure vm quick-create -M ~/.ssh/azure_id_rsa.pub -Q OpenLogic:Centos:7.2:latest

## 部署 Linux VM

下面显示了使用如上所示的命令时出现的每个提示以及预期会显示的输出。

## 使用 ImageUR

下表列出了 Linux 分发版（从 Azure CLI 0.9 版起）。

| 发布者 | 产品 | SKU | 版本 |
|:----------|:-------------|:------------|:--------|
| OpenLogic | Centos | 7\.2 | 最新 |
| SUSE | openSUSE | 13\.2 | 最新 |
| Canonical | UbuntuServer | 14\.04.3-LTS | 最新 |



对于 **ImageURN** 选项 (`-Q`)，我们将使用 `Canonical:UbuntuServer:14.04.3-LTS:latest` 来部署 Canonical Ubuntu 14.04.3-LTS。（这 3 个映像代表 Azure 上可用 OS 的一小部分；通过在应用商店中[搜索映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)来查找更多映像，或者[上载自己的自定义映像](/documentation/articles/virtual-machines-linux-create-upload-generic/)。）

在下面的命令演练中，请将提示替换为你自己环境中的值。我们将使用“示例”值。

遵循提示并输入你自己的名称

	azure vm quick-create -M ~/.ssh/id_rsa.pub -Q Canonical:UbuntuServer:14.04.3-LTS:latest

输出应类似于以下输出块。

	info:    Executing command vm quick-create
	Resource group name: rhel-quick
	Virtual machine name: rhel
	Location name: chinanorth
	Operating system Type [Windows, Linux]: linux
	User name: ops
	+ Listing virtual machine sizes available in the location "chinanorth"
	+ Looking up the VM "rhel"
	info:    Verifying the public key SSH file: /Users/ops/.ssh/id_rsa.pub
	info:    Using the VM Size "Standard_D1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Looking up the storage account cli1630678171193501687
	info:    Could not find the storage account "cli1630678171193501687", trying to create new one
	+ Creating storage account "cli1630678171193501687" in "chinanorth"
	+ Looking up the storage account cli1630678171193501687
	+ Looking up the NIC "rhel-china-1630678171-nic"
	info:    An nic with given name "rhel-china-1630678171-nic" not found, creating a new one
	+ Looking up the virtual network "rhel-china-1630678171-vnet"
	info:    Preparing to create new virtual network and subnet
	+ Creating a new virtual network "rhel-china-1630678171-vnet" [address prefix: "10.0.0.0/16"] with subnet "rhel-china-1630678171-snet" [address prefix: "10.0.1.0/24"]
	+ Looking up the virtual network "rhel-china-1630678171-vnet"
	+ Looking up the subnet "rhel-china-1630678171-snet" under the virtual network "rhel-china-1630678171-vnet"
	info:    Found public ip parameters, trying to setup PublicIP profile
	+ Looking up the public ip "rhel-china-1630678171-pip"
	info:    PublicIP with given name "rhel-china-1630678171-pip" not found, creating a new one
	+ Creating public ip "rhel-china-1630678171-pip"
	+ Looking up the public ip "rhel-china-1630678171-pip"
	+ Creating NIC "rhel-china-1630678171-nic"
	+ Looking up the NIC "rhel-china-1630678171-nic"
	+ Looking up the storage account clisto909893658rhel
	+ Creating VM "rhel"
	+ Looking up the VM "rhel"
	+ Looking up the NIC "rhel-china-1630678171-nic"
	+ Looking up the public ip "rhel-china-1630678171-pip"
	data:    Id                              :/subscriptions/<guid>/resourceGroups/rhel-quick/providers/Microsoft.Compute/virtualMachines/rhel
	data:    ProvisioningState               :Succeeded
	data:    Name                            :rhel
	data:    Location                        :chinanorth
	data:    Type                            :Microsoft.Compute/virtualMachines
	data:
	data:    Hardware Profile:
	data:      Size                          :Standard_D1
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
	data:        Name                        :clic5abbc145c0242c1-os-1462425492101
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://cli1630678171193501687.blob.core.chinacloudapi.cn/vhds/clic5abbc145c0242c1-os-1462425492101.vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :rhel
	data:      User Name                     :ops
	data:      Linux Configuration:
	data:        Disable Password Auth       :true
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-32-0F-DD
	data:          Provisioning State        :Succeeded
	data:          Name                      :rhel-china-1630678171-nic
	data:          Location                  :chinanorth
	data:            Public IP address       :104.42.236.196
	data:            FQDN                    :rhel-china-1630678171-pip.chinanorth.chinacloudapp.cn
	data:
	data:    Diagnostics Profile:
	data:      BootDiagnostics Enabled       :true
	data:      BootDiagnostics StorageUri    :https://clisto909893658rhel.blob.core.chinacloudapi.cn/
	data:
	data:      Diagnostics Instance View:
	info:    vm quick-create command OK

现在，可以使用默认的 SSH 端口 22 和上述输出中列出的完全限定域名 (FQDN) 通过 SSH 连接到 VM。（也可以使用所列的 IP 地址。）

	ssh ops@rhel-china-1630678171-pip.chinanorth.chinacloudapp.cn

登录过程应如下所示：

	The authenticity of host 'rhel-china-1630678171-pip.chinanorth.chinacloudapp.cn (104.42.236.196)' can't be established.
	RSA key fingerprint is 0e:81:c4:36:2d:eb:3c:5a:dc:7e:65:8a:3f:3e:b0:cb.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'rhel-china-1630678171-pip.chinanorth.chinacloudapp.cn,104.42.236.196' (RSA) to the list of known hosts.
	[ops@rhel ~]$ ls -a
	.  ..  .bash_logout  .bash_profile  .bashrc  .cache  .config  .ssh

## 后续步骤

`azure vm quick-create` 是一种快速部署 VM 的方法，因此你可以登录到 bash shell 并开始使用。使用 `vm quick-create` 不会在复杂的环境中带来其他好处。若要部署针对基础结构自定义的 Linux VM，可以遵循下列任一文章操作。

- [Use an Azure resource manager template to create a specific deployment（使用 Azure Resource Manager 模板创建特定部署）](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)
- [Create your own custom environment for a Linux VM using Azure CLI commands directly（直接使用 Azure CLI 命令为 Linux VM 创建自定义环境）](/documentation/articles/virtual-machines-linux-create-cli-complete/)
- [Create a SSH Secured Linux VM on Azure using Templates（使用模板在 Azure 上创建受 SSH 保护的 Linux VM）](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

这些文章可帮助你开始构建 Azure 基础结构，并介绍多种专属和开源基础结构部署、配置与协调工具。

<!---HONumber=Mooncake_0620_2016-->