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
	ms.date="04/08/2016"
	wacn.date="06/06/2016"/>


# 使用 CLI 在 Azure 上创建 Linux VM

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

本文说明如何使用 Azure CLI 的 `azure vm quick-create` 命令在 Azure 上快速创建 Linux 虚拟机。`quick-create` 命令可创建带基本的基础结构的 VM，你可以将其用于构建某一概念的原型并非常快速地进行测试。可以将其视为指向 Linux bash shell 的最快方法。此项目需要一个资源管理器模式下 (`azure config mode arm`) 的 Azure 帐户（[获取试用版](/pricing/1rmb-trial/)）和 [Azure CLI](/documentation/articles/xplat-cli-install)。

## 快速命令摘要

	# One command to quickly the VM that prompts for arguments
	ahmet@fedora$ azure vm quick-create -M ~/.ssh/azure_id_rsa.pub

## 详细演练

## 创建 Linux VM

在下面的命令中，你可以随意使用任何映像，但此示例使用 `canonical:ubuntuserver:14.04.2-LTS:latest` 来快速创建 VM。（若要在应用商店中找到映像，请[搜索图像](/documentation/articles/virtual-machines-linux-cli-ps-findimage) 或者你可以[上传你自己的自定义映像](/documentation/articles/virtual-machines-linux-create-upload-generic)。） 它将看起来如下所示。

在下面的命令演练中，请使用你自己环境中的值替换提示符。我们对此项目使用“示例”值

	# Create the Linux VM using prompts
	ahmet@fedora$ azure vm quick-create -M ~/.ssh/azure_id_rsa.pub
	info:    Executing command vm quick-create
	Resource group name: exampleRGname
	Virtual machine name: exampleVMname
	Location name: chinanorth
	Operating system Type [Windows, Linux]: /documentation/articles/Linux
	ImageURN (in the format of "publisherName:offer:skus:version") or a VHD link to the user image: Canonical:UbuntuServer:14.04.3-LTS:latest
	User name: ahmet
	Password: ************************************************
	Confirm password: ************************************************
	+ Looking up the VM "exampleVMname"
	info:    Verifying the public key SSH file: /home/ahmet/.ssh/azure_id_rsa.pub
	info:    Using the VM Size "Standard_D1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Looking up the storage account cli38948918364134011018
	info:    Could not find the storage account "cli38948918364134011018", trying to create new one
	+ Creating storage account "cli38948918364134011018" in "chinanorth"
	+ Looking up the storage account cli38948918364134011018
	+ Looking up the NIC "examp-westu-3894891836-nic"
	info:    An nic with given name "examp-westu-3894891836-nic" not found, creating a new one
	+ Looking up the virtual network "examp-westu-3894891836-vnet"
	info:    Preparing to create new virtual network and subnet
	| Creating a new virtual network "examp-westu-3894891836-vnet" [address prefix: "10.0.0.0/16"] with subnet "examp-westu-3894891836-snet" [address prefix: "10.+.1.0/24"]
	+ Looking up the virtual network "examp-westu-3894891836-vnet"
	+ Looking up the subnet "examp-westu-3894891836-snet" under the virtual network "examp-westu-3894891836-vnet"
	info:    Found public ip parameters, trying to setup PublicIP profile
	+ Looking up the public ip "examp-westu-3894891836-pip"
	info:    PublicIP with given name "examp-westu-3894891836-pip" not found, creating a new one
	+ Creating public ip "examp-westu-3894891836-pip"
	+ Looking up the public ip "examp-westu-3894891836-pip"
	+ Creating NIC "examp-westu-3894891836-nic"
	+ Looking up the NIC "examp-westu-3894891836-nic"
	+ Creating VM "exampleVMname"
	+ Looking up the VM "exampleVMname"
	+ Looking up the NIC "examp-westu-3894891836-nic"
	+ Looking up the public ip "examp-westu-3894891836-pip"
	data:    Id                              :/subscriptions/<**subscriptionsID**>/resourceGroups/exampleRGname/providers/Microsoft.Compute/virtualMachines/exampleVMname
	data:    ProvisioningState               :Succeeded
	data:    Name                            :exampleVMname
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
	data:        Name                        :clife36db80ae0539d2-os-1460152163612
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://<**subscriptionsID**>.blob.core.chinacloudapi.cn/vhds/clife36db80ae0539d2-os-1460152163612.vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :exampleVMname
	data:      User Name                     :ahmet
	data:      Linux Configuration:
	data:        Disable Password Auth       :false
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-33-4C-B2
	data:          Provisioning State        :Succeeded
	data:          Name                      :examp-westu-3894891836-nic
	data:          Location                  :chinanorth
	data:            Public IP address       :13.88.22.244
	data:            FQDN                    :examp-westu-3894891836-pip.chinanorth.chinacloudapp.cn
	data:
	data:    Diagnostics Profile:
	data:      BootDiagnostics Enabled       :true
	data:      BootDiagnostics StorageUri    :https://<**subscriptionsID**>.blob.core.chinacloudapi.cn/
	data:
	data:      Diagnostics Instance View:
	info:    vm quick-create command OK

你现在可以使用 SSH 通过默认的 SSH 端口 22 登录到 VM 以及进入上述输出中列出的公共 IP 地址。

	ahmet@fedora$ ssh -i ~/.ssh/azure_id_rsa ubuntu@13.88.22.244

`azure vm quick-create` 是一种快速创建 VM 的方法，因此你可以登录到 bash shell 并开始使用。使用 `vm quick-create` 并不会为你提供复杂环境的其他好处，但是，如果你想要自定义环境，则可以[使用 Azure 资源管理器模板快速创建特定部署](/documentation/articles/virtual-machines-linux-cli-deploy-templates)，或者你可以[使用 Azure CLI 命令为 Linux VM 直接创建你自己的自定义环境](/documentation/articles/virtual-machines-linux-cli-deploy-templates)。

以上示例将创建：

- 在其中部署 VM 的 Azure 资源组
- 存储作为 VM 映像的 .vhd 文件的 Azure 存储帐户
- 提供到 VM 的连接的 Azure 虚拟网络和子网
- 与网络相关联的虚拟网络接口卡 (NIC)
- 提供 Internet 地址供外部使用，然后将在该环境内创建 Linux VM 的公共 IP 地址和子域前缀。

## 后续步骤

现在，你已快速创建了一个用于测试或演示的 Linux VM。若要创建针对基础结构自定义的 Linux VM，可以遵循下列任一文章操作。

- [Create a Linux VM on Azure using Templates（使用模板在 Azure 上创建 Linux VM）](/documentation/articles/virtual-machines-linux-cli-deploy-templates)
- [Create a SSH Secured Linux VM on Azure using Templates（使用模板在 Azure 上创建受 SSH 保护的 Linux VM）](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template)
- [Create a Linux VM using the Azure CLI（使用 Azure CLI 创建 Linux VM）](/documentation/articles/virtual-machines-linux-create-cli-complete)

这些文章可帮助你开始构建 Azure 基础结构，并介绍多种专属和开源基础结构部署、配置与协调工具。

<!---HONumber=Mooncake_0509_2016-->