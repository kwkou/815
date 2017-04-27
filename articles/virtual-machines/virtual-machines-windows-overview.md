<properties
    pageTitle="Windows 虚拟机概述 | Azure"
    description="了解如何在 Azure 中创建和管理 Windows 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management" />
<tags 
    ms.assetid="fbae9c8e-2341-4ed0-bb20-fd4debb2f9ca"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="10/20/2016"
    wacn.date="03/28/2017"
    ms.author="davidmu" />

# Azure 中的 Windows 虚拟机概述
Azure 虚拟机 (VM) 是 Azure 提供的多种[可缩放按需分配计算资源](/documentation/articles/choose-web-site-cloud-service-vm/)之一。通常情况下，如果需要以更大的力度（相对于其他控制选项）控制计算环境，则应选择 VM。本文介绍创建 VM 之前的注意事项，以及 VM 的创建方法和管理方式。

使用 Azure VM 可以灵活进行虚拟化，而无需购买和维护运行 VM 的物理硬件。不过，仍然需要通过执行任务来维护 VM，例如，配置、修补和安装在 VM 上运行的软件。

可通过多种方式使用 Azure 虚拟机。下面是一些示例：

* **开发和测试** – 在 Azure VM 上，可以快速轻松地创建具有特定配置的计算机来满足编程和应用程序测试的需要。
* **云中的应用程序** – 由于应用程序的需求会不断变化，在 Azure 中的 VM 上运行应用程序可能会节省成本。使用 VM 时，需要支付额外的费用；关闭 VM 时，则无需付费。
* **扩展的数据中心** – Azure 虚拟网络中的虚拟机可以轻松连接到组织的网络。

可以根据需要，将应用程序使用的 VM 纵向和横向扩展为任意数目。

## 在创建 VM 之前需要考虑哪些因素？
在 Azure 中构建应用程序基础结构时，始终要全方位地考虑[设计注意事项](/documentation/articles/virtual-machines-windows-infrastructure-virtual-machine-guidelines/)。在开始之前，必须考虑到 VM 的以下重要方面：

* 应用程序资源的名称
* 资源的存储位置
* VM 的大小
* 可以创建的 VM 数目上限
* VM 运行的操作系统
* VM 在启动后的配置
* VM 所需的相关资源

### 命名
虚拟机有一个分配的[名称](/documentation/articles/virtual-machines-windows-infrastructure-naming-guidelines/)，另外，它还配置有一个在操作系统中使用的计算机名称。VM 的名称最多可包含 15 个字符。

如果使用 Azure 创建操作系统磁盘，计算机名称与虚拟机名称相同。如果[上载包含以前配置的操作系统的自有映像](/documentation/articles/virtual-machines-windows-upload-image/)并使用它来创建虚拟机，则名称可以不同。建议在上载自己的映像文件时，使操作系统中的计算机名称与虚拟机名称保持相同。

### 位置
创建 VM 时，区域通常称为**位置**。位置指定 VM 虚拟硬盘的存储位置。

下表显示了获取可用位置列表的一些方法。

| 方法 | 说明 |
| --- | --- |
| Azure 门户预览 |创建 VM 时，可从列表中选择位置。 |
| Azure PowerShell |使用 [Get-AzureRmLocation](https://msdn.microsoft.com/zh-cn/library/mt619449.aspx) 命令。 |
| REST API |使用“[列出位置](https://docs.microsoft.com/en-us/rest/api/resources/subscriptions#Subscriptions_ListLocations)”操作。 |

### VM 大小
VM 的[大小](/documentation/articles/virtual-machines-windows-sizes/)由所要运行的工作负荷决定。然后，选择的大小决定了处理能力、内存和存储容量等因素。Azure 提供各种大小来支持多种类型的用途。

Azure 根据 VM 的大小和操作系统按[小时价格](/pricing/details/virtual-machines/)计费。对于不足一小时的部分，Azure 仅根据使用的分钟数计费。存储将另行定价和收费。

### VM 限制
订阅附带默认的[配额限制](/documentation/articles/azure-subscription-service-limits/)，在为项目部署大量 VM 时，这些限制可能会造成影响。每个订阅的当前限制是每区域 20 个 VM。可以开具支持票证来请求提高限制。

### 操作系统磁盘和映像
虚拟机使用[虚拟硬盘 (VHD)](/documentation/articles/storage-about-disks-and-vhds-windows/) 来存储其操作系统 (OS) 和数据。VHD 还可用于存储映像，你可以选择某个映像来安装 OS。

Azure 提供许多应用商店映像，这些映像可配合各种版本和类型的 Windows Server 操作系统使用。应用商店映像由映像发布者、产品、SKU 和版本（通常指定为最新版本）标识。

下表显示了查找映像信息的一些方法。

| 方法 | 说明 |
| --- | --- |
| Azure 门户预览 |选择要使用的映像时，系统会自动指定值。 |
| Azure PowerShell |[Get-AzureRMVMImagePublisher](https://msdn.microsoft.com/zh-cn/library/mt603484.aspx) -Location "location"<BR>[Get-AzureRMVMImageOffer](https://msdn.microsoft.com/zh-cn/library/mt603824.aspx) -Location "location" -Publisher "publisherName"<BR>[Get-AzureRMVMImageSku](https://msdn.microsoft.com/zh-cn/library/mt619458.aspx) -Location "location" -Publisher "publisherName" -Offer "offerName" |
| REST API |[列出映像发布者](https://docs.microsoft.com/zh-cn/rest/api/compute/platformimages/platformimages-list-publishers)<BR>[列出映像产品](https://docs.microsoft.com/zh-cn/rest/api/compute/platformimages/platformimages-list-publisher-offers)<BR>[列出映像 SKU](https://docs.microsoft.com/zh-cn/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) |

可以选择[上载并使用自己的映像](/documentation/articles/virtual-machines-windows-upload-image/)，在这种情况下，无需使用发布者名称、产品和 SKU。

### 扩展
VM [扩展](/documentation/articles/virtual-machines-windows-extensions-features/)通过部署后的配置和自动化任务来增加 VM 的功能。

可以使用扩展完成以下常见任务：

* **运行自定义脚本** – 预配 VM 时，[自定义脚本扩展](/documentation/articles/virtual-machines-windows-extensions-customscript/)可以通过运行脚本，帮助在 VM 上配置工作负荷。
* **部署和管理配置** – 可以借助 [PowerShell Desired State Configuration (DSC) 扩展](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)在 VM 上设置用于管理配置和环境的 DSC。
* **收集诊断数据** – 可以借助 [Azure 诊断扩展](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/)将 VM 配置为收集诊断数据，然后，可以使用这些数据监视应用程序的运行状况。

### 相关资源
VM 使用下表中的资源，创建 VM 时，这些资源必须存在，否则要予以创建。

| 资源 | 必选 | 说明 |
| --- | --- | --- |
| [资源组](/documentation/articles/resource-group-overview/) |是 |VM 必须包含在资源组中。 |
| [存储帐户](/documentation/articles/storage-create-storage-account/) |是 |VM 需要使用存储帐户来存储其虚拟硬盘。 |
| [虚拟网络](/documentation/articles/virtual-networks-overview/) |是 |VM 必须是虚拟网络的成员。 |
| [公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/) |否 |可以向 VM 分配一个公共 IP 地址，以便远程访问它。 |
| [网络接口](/documentation/articles/virtual-network-network-interface/) |是 |VM 需要使用网络接口在网络中通信。 |
| [数据磁盘数](/documentation/articles/virtual-machines-windows-attach-disk-portal/) |否 |VM 可以包含数据磁盘，以便扩展存储功能。 |

## 如何创建第一个 VM？
可以选择多种方法创建 VM。选择哪种方法取决于所在的环境。

下表提供的信息可帮助你开始创建 VM。

| 方法 | 文章 |
| --- | --- |
| Azure 门户预览 |[使用门户创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/) |
| 模板 |[使用 Resource Manager 模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/) |
| Azure PowerShell |[使用 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-quick-create-powershell/) |
| 客户端 SDK |[使用 C# 部署 Azure 资源](/documentation/articles/virtual-machines-windows-csharp/) |
| REST API |[创建或更新 VM](https://docs.microsoft.com/zh-cn/rest/api/compute/virtualmachines/virtualmachines-create-or-update) |

问题偶尔会不期而至。如果遇到了问题，请查看 [Troubleshoot Resource Manager deployment issues with creating a Windows virtual machine in Azure](/documentation/articles/virtual-machines-windows-troubleshoot-deployment-new-vm/)（排查在 Azure 中创建 Windows 虚拟机时遇到的 Resource Manager 部署问题）。

## 如何管理创建的 VM？
可以使用基于浏览器的门户、支持脚本的命令行工具或直接通过 API 管理 VM。可能要执行的一些常见管理任务包括获取有关 VM 的信息、登录到 VM、管理可用性以及执行备份。

### 获取有关 VM 的信息
下表显示了获取有关 VM 的信息的一些方法。

| 方法 | 说明 |
| --- | --- |
| Azure 门户预览 |在中心菜单中，单击“虚拟机”，然后从列表中选择 VM。在 VM 的边栏选项卡中，可以访问概述信息、设置值以及监视指标。 |
| Azure PowerShell |有关使用 PowerShell 管理 VM 的信息，请参阅 [Manage Azure Virtual Machines using Resource Manager and PowerShell](/documentation/articles/virtual-machines-windows-ps-manage/)（使用 Resource Manager 与 PowerShell 来管理 Azure 虚拟机）。 |
| REST API |使用“[获取 VM 信息](https://docs.microsoft.com/zh-cn/rest/api/compute/virtualmachines/virtualmachines-get)”操作获取有关 VM 的信息。|
| 客户端 SDK |有关使用 C# 管理 VM 的信息，请参阅 [Manage Azure Virtual Machines using Azure Resource Manager and C#](/documentation/articles/virtual-machines-windows-csharp-manage/)（使用 Azure Resource Manager 与 C# 来管理 Azure 虚拟机）。 |

### 登录到 VM
使用 Azure 门户预览中的“连接”按钮[启动远程桌面 (RDP) 会话](/documentation/articles/virtual-machines-windows-connect-logon/)。尝试使用远程连接时，有时可能会出错。如果遇到这种情况，请查看[对运行 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)中的帮助信息。

### 管理可用性
必须知道如何[确保应用程序的高可用性](/documentation/articles/virtual-machines-windows-manage-availability/)。此配置涉及到创建多个 VM，确保至少有一个 VM 在运行。

为了使部署符合 VM 运行时间达到 99.95% 的服务级别协议，必须在[可用性集](/documentation/articles/virtual-machines-windows-infrastructure-availability-sets-guidelines/)中部署两个或更多个运行工作负荷的 VM。此配置可确保 VM 分布到多个容错域，并使用不同的维护时段部署到主机上。完整 [Azure SLA](/support/sla/virtual-machines/) 说明了 Azure 作为整体的保证可用性。

### 备份 VM
[恢复服务保管库](/documentation/articles/backup-introduction-to-azure-backup/)用于保护 Azure 备份和 Azure Site Recovery 服务中的数据与资产。可以使用恢复服务保管库，[通过 PowerShell 来部署和管理采用 Resource Manager 部署模型的 VM 的备份](/documentation/articles/backup-azure-vms-automation/)。

## 后续步骤
* 如果有意使用 Linux VM，请查看 [Azure 和 Linux](/documentation/articles/virtual-machines-linux-azure-overview/)。
* 在[示例 Azure 基础结构演练](/documentation/articles/virtual-machines-windows-infrastructure-example/)中查看有关设置基础结构的指导。
* 请务必遵循 [Best Practices for running a Windows VM on Azure](/documentation/articles/virtual-machines-windows-guidance-compute-single-vm/)（在 Azure 上运行 Windows VM 的最佳实践）。

<!---HONumber=Mooncake_1212_2016-->