<properties
	pageTitle="创建并上载自定义 Linux 映像 | Azure"
	description="使用 Resource Manager 部署模型创建包含自定义 Linux 映像的虚拟硬盘 (VHD) 并将其上载到 Azure。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-linux"
	ms.date="07/15/2016"
	wacn.date="09/12/2016"/>  


# 上载自定义磁盘映像并从其创建 Linux VM

本文说明如何使用 Resource Manager 部署模型将虚拟硬盘 (VHD) 上载到 Azure，并从此自定义映像创建 Linux VM。此功能可让你安装并配置 Linux 分发以满足你的需求，然后使用该 VHD 快速创建 Azure 虚拟机 (VM)。

## 快速命令
确保已登录 [Azure CLI](/documentation/articles/xplat-cli-install/) 并使用 Resource Manager 模式 (`azure config mode arm`)。

首先，创建一个资源组：

	azure group create TestRG --location "ChinaNorth"

创建一个存储帐户以存放虚拟磁盘：

	azure storage account create testuploadedstorage --resource-group TestRG \
	--location "ChinaNorth" --kind Storage --sku-name PLRS

列出创建的存储帐户的存储密钥，并记下 `key1`：

	azure storage account keys list testuploadedstorage --resource-group TestRG

使用刚获取的存储密钥在存储帐户内创建容器：


	azure storage container create --account-name testuploadedstorage \
	--account-key <key1> --container vm-images

最后，将 VHD 上载到创建的容器：



	azure storage blob upload --blobtype page --account-name testuploadedstorage \
	--account-key <key1> --container vm-images /path/to/disk/yourdisk.vhd

现在，可以[使用 Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd)从上载的虚拟磁盘创建 VM。为此，也可以在 CLI 中指定磁盘的 URI，如下所示：

	azure vm create TestVM -l "ChinaNorth" --resource-group TestRG \
	-Q https://testuploadedstorage.blob.core.chinacloudapi.cn/vm-images/yourdisk.vhd

目标存储帐户必须与虚拟磁盘上载到的位置相同。还需要指定或根据提示输入 `azure vm create` 命令所需的所有其他参数，例如虚拟网络、公共 IP 地址、用户名和 SSH 密钥。阅读有关[可用 CLI Resource Manager 参数](/documentation/articles/azure-cli-arm-commands/#azure-vm-commands-to-manage-your-azure-virtual-machines)的详细信息。


## 详细步骤
在准备自定义 Linux 映像并将其上载到 Azure 的过程中，需执行多个步骤。本文的其余部分提供前面的一系列快速命令中所述的每个步骤的更多相关详细信息。


## 要求
若要完成以下步骤，需要：

- **安装在 .vhd 文件中的 Linux 操作系统** — 将 [Azure 认可的 Linux 分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)（或参阅[关于未认可分发的信息](/documentation/articles/virtual-machines-linux-create-upload-generic/)）安装在 VHD 格式的虚拟磁盘中。可使用多种工具创建 VM 和 VHD：
	- 安装并配置 [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) 或 [KVM](http://www.linux-kvm.org/page/RunningKVM)，并小心使用 VHD 作为你的映像格式。如有需要，可以使用 `qemu-img convert` [转换映像](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats)。
	- 也可以在 [Windows 10](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install) 或 [Windows Server 2012/2012 R2](https://technet.microsoft.com/zh-cn/library/hh846766.aspx) 上使用 Hyper-V。

> [AZURE.NOTE] Azure 不支持更新的 VHDX 格式。创建 VM 时，请将 VHD 指定为映像格式。如有需要，可以使用 [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) 或 [`Convert-VHD`](https://technet.microsoft.com/zh-cn/library/hh848454.aspx) PowerShell cmdlet 将 VHDX 磁盘转换为 VHD。此外，Azure 不支持上载动态 VHD，因此，上载之前，你需要将此类磁盘转换为静态 VHD。可以使用 [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go) 等工具在上载到 Azure 的过程中转换动态磁盘。

- 从自定义映像创建的 VM 必须位于映像本身所在的存储帐户中
	- 创建存储帐户和容器，以存放自定义映像以及所创建的 VM
	- 创建所有 VM 之后，可以安全地删除映像


## <a id="prepimage"> </a> 准备要上载的映像

Azure 支持各种 Linux 分发版（请参阅 [Endorsed Distributions](/documentation/articles/virtual-machines-linux-endorsed-distros/)（认可的分发版））。以下文章将指导你如何准备 Azure 上支持的各种 Linux 分发：

- **[基于 CentOS 的分发](/documentation/articles/virtual-machines-linux-create-upload-centos/)**
- **[Debian Linux](/documentation/articles/virtual-machines-linux-debian-create-upload-vhd/)**
- **[Oracle Linux](/documentation/articles/virtual-machines-linux-oracle-create-upload-vhd/)**
- **[Red Hat Enterprise Linux](/documentation/articles/virtual-machines-linux-redhat-create-upload-vhd/)**
- **[SLES 和 openSUSE](/documentation/articles/virtual-machines-linux-suse-create-upload-vhd/)**
- **[Ubuntu](/documentation/articles/virtual-machines-linux-create-upload-ubuntu/)**
- **[其他 - 非认可分发](/documentation/articles/virtual-machines-linux-create-upload-generic/)**

另请参阅 **[Linux 安装说明](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)**，以获取更多有关如何为 Azure 准备 Linux 映像的一般提示。

> [AZURE.NOTE] 只有在使用某个认可的分发的时候也使用 [Azure 认可的分发中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distros/) 中“支持的版本”下指定的配置详细信息时，[Azure 平台 SLA](/support/sla/virtual-machines/) 才适用于运行 Linux 的 VM。


## 创建资源组
资源组以逻辑方式将所有 Azure 资源（例如虚拟网络和存储）聚集在一起，以支持虚拟机。从此处了解有关 [Azure 资源组](/documentation/articles/resource-group-overview/)的详细信息。在上载自定义磁盘映像和创建 VM 之前，首先需要创建一个资源组：

	azure group create TestRG --location "ChinaNorth"

## 创建存储帐户
VM 以页 Blob 形式存储在存储帐户中。从此处了解有关 [Azure Blob 存储](/documentation/articles/storage-introduction/#blob-storage)的详细信息。为自定义磁盘映像和 VM 创建存储帐户。从自定义磁盘映像创建的所有 VM 都必须位于该映像所在的存储帐户中。

在创建的资源组中创建存储帐户：

	azure storage account create testuploadedstorage --resource-group TestRG \
		--location "ChinaNorth" --kind Storage --sku-name PLRS

## 列出存储帐户密钥
Azure 将为每个存储帐户生成两个 512 位的访问密钥。在向存储帐户进行身份验证以执行操作（例如执行写入操作）时，将使用这些访问密钥。从此处了解有关[管理对存储的访问](/documentation/articles/storage-create-storage-account/#manage-your-storage-account)的详细信息。你可以使用 `azure storage account keys list` 命令查看访问密钥。

查看创建的存储帐户的访问密钥：

	azure storage account keys list testuploadedstorage --resource-group TestRG

输出类似于：

	info:    Executing command storage account keys list
	+ Getting storage account keys
	data:    Name  Key                                                                                       Permissions
	data:    ----  ----------------------------------------------------------------------------------------  -----------
	data:    key1  d4XAvZzlGAgWdvhlWfkZ9q4k9bYZkXkuPCJ15NTsQOeDeowCDAdB80r9zA/tUINApdSGQ94H9zkszYyxpe8erw==  Full
	data:    key2  Ww0T7g4UyYLaBnLYcxIOTVziGAAHvU+wpwuPvK4ZG0CDFwu/mAxS/YYvAQGHocq1w7/3HcalbnfxtFdqoXOw8g==  Full
	info:    storage account keys list command OK

记下 `key1`，因为你将在后续步骤中使用它与存储帐户进行交互。

## 创建存储容器
在存储帐户内创建容器以整理虚拟磁盘和映像，其方式与创建各种目录以便通过逻辑方式整理本地文件系统相同。一个存储帐户可以包含任意数目的容器。

创建一个新容器，并指定在上一步中获取的访问密钥：

	azure storage container create --account-name testuploadedstorage \
		--account-key <key1> --container vm-images

## 上载 VHD
现在可以真正地上载自定义磁盘映像。与 VM 所用的所有虚拟磁盘一样，需要上载自定义磁盘映像并将其作为页 Blob 存储。

指定访问密钥、在上一步中创建的容器，以及自定义磁盘映像在本地计算机上的路径：

	azure storage blob upload --blobtype page --account-name testuploadedstorage \
		--account-key <key1> --container vm-images /path/to/disk/yourdisk.vhd

## 从自定义映像创建 VM
从自定义磁盘映像创建 VM 时，需指定磁盘映像的 URI。确保目标存储帐户与用于存储自定义磁盘映像的位置相匹配。可以使用 Azure CLI 或 Resource Manager JSON 模板创建 VM。


### 使用 Azure CLI 创建 VM
在 `azure vm create` 命令中指定 `--image-urn`（或简称 `-Q`）参数，以指向自定义磁盘映像。确保 `--storage-account-name`（或 `-o`）与用于存储自定义磁盘映像的存储帐户匹配。不需要使用与自定义磁盘映像相同的容器来存储 VM。上载自定义磁盘映像之前，请确保使用前面步骤中所述的相同方式创建任何附加容器。

从自定义磁盘映像创建 VM：


	azure vm create TestVM -l "ChinaNorth" --resource-group TestRG \
	-Q https://testuploadedstorage.blob.core.chinacloudapi.cn/vm-images/yourdisk.vhd
	-o testuploadedstorage
仍需要指定或根据提示输入 `azure vm create` 命令所需的所有其他参数，例如虚拟网络、公共 IP 地址、用户名和 SSH 密钥。阅读有关[可用 CLI Resource Manager 参数](/documentation/articles/azure-cli-arm-commands/#azure-vm-commands-to-manage-your-azure-virtual-machines)的详细信息。

### 使用 JSON 模板创建 VM
Azure Resource Manager 模板是用于定义你希望生成的环境的 JavaScript 对象表示法 (JSON) 文件。这些模板细分为不同的资源提供程序，如计算或网络。你可以使用现有模板，也可以编写自己的模板。阅读有关[使用 Resource Manager 和模板](/documentation/articles/resource-group-overview/)的详细信息。

在模板的 `Microsoft.Compute/virtualMachines` 提供程序中有一个 `storageProfile` 节点，其中包含 VM 的配置详细信息。需要编辑的两个主要参数为 `image` 和 `vhd` URI，它们指向自定义磁盘映像和新 VM 的虚拟磁盘。下面显示了使用自定义磁盘映像的 JSON 示例：

	"storageProfile": {
          "osDisk": {
            "name": "TestVM",
            "osType": "Linux",
            "caching": "ReadWrite",
			"createOption": "FromImage",
            "image": {
              "uri": "https://testuploadedstorage.blob.core.chinacloudapi.cn/vm-images/yourdisk.vhd"
            },
            "vhd": {
              "uri": "https://testuploadedstorage.blob.core.chinacloudapi.cn/vhds/newvmname.vhd"
            }
          }

可以使用[此现有模板从自定义映像创建 VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)，或阅读有关[创建自己的 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)的信息。

一旦配置了模板，就可以使用 `azure group deployment create` 命令创建 VM。使用 `--template-uri` 参数指定 JSON 模板的 URI：

	azure group deployment create --resource-group TestTemplateRG
		--template-uri https://uri.to.template/yourtemplate.json

如果在计算机上以本地方式存储了一个 JSON 文件，则可以改用 `--template-file` 参数：

	azure group deployment create --resource-group TestTemplateRG
		--template-file /path/to/yourtemplate.json


## 后续步骤
准备好并上载自定义虚拟磁盘之后，可以阅读有关[使用 Resource Manager 和模板](/documentation/articles/resource-group-overview/)的详细信息。你可能还需要向新 VM [添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。如果需要访问在 VM 上运行的应用程序，请务必[打开端口和终结点](/documentation/articles/virtual-machines-linux-nsg-quickstart/)。

<!---HONumber=Mooncake_0905_2016-->