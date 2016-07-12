<properties
	pageTitle="在 Azure 中创建和上载 Linux VHD"
	description="了解如何创建和上载包含 Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="szarkos"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/09/2016"
	wacn.date="06/13/2016"/>

# <a id="nonendorsed"> </a>有关未认可分发的信息 #

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


**重要提示**：仅当使用某个[认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)时，Azure 平台 SLA 才适用于运行 Linux 操作系统的虚拟机。在 Azure 平台映像库中提供的所有 Linux 分发都是具有所需配置的认可的分发。

- [Azure 上的 Linux - 认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)
- [Azure 中对 Linux 映像的支持](http://support2.microsoft.com/kb/2941892)

所有正在 Azure 上运行的分发都需要满足多个先决条件才能在平台上正常运行。本文并未涵盖所有信息，因为每个分发都是不同的；即使你满足以下所有条件，你也可能仍需显著调整你的 Linux 系统以确保其在平台上正常运行。

正是出于这个原因，建议你如果可能，从某个我们的 [Azure 上的 Linux - 认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)开始操作。以下文章将指导你完成如何准备 Azure 上支持的各种认可的 Linux 分发：

- **[基于 CentOS 的分发](/documentation/articles/virtual-machines-linux-create-upload-centos/)**
- **[Debian Linux](/documentation/articles/virtual-machines-linux-debian-create-upload-vhd/)**
- **[Oracle Linux](/documentation/articles/virtual-machines-linux-oracle-create-upload-vhd/)**
- **[Red Hat Enterprise Linux](/documentation/articles/virtual-machines-linux-redhat-create-upload-vhd/)**
- **[SLES 和 openSUSE](/documentation/articles/virtual-machines-linux-suse-create-upload-vhd/)**
- **[Ubuntu](/documentation/articles/virtual-machines-linux-create-upload-ubuntu/)**

本文的其余部分将重点介绍有关在 Azure 上运行 Linux 分发的一般准则。


## <a id="linuxinstall" name="general-linux-installation-notes"> </a>常规 Linux 安装说明 ##

- Azure 不支持 VHDX 格式，仅支持**固定大小的 VHD**。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。如果你使用 VirtualBox，则意味着选择的是“固定大小”，而不是在创建磁盘时动态分配默认大小。

- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果需要，可以在数据磁盘上使用 [LVM](/documentation/articles/virtual-machines-linux-configure-lvm/) 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid/)。

- 由于低于 2.6.37 的 Linux 内核版本中的 bug，更大的 VM 不支持 NUMA。此问题主要影响使用上游 Red Hat 2.6.32 内核的分发。手动安装的 Azure Linux 代理 (waagent) 将自动在 Linux 内核的 GRUB 配置中禁用 NUMA。

- 不要在操作系统磁盘上配置交换分区。可以配置 Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。

- 所有 VHD 的大小必须是 1 MB 的倍数。


### 安装不带 Hyper-V 的 Linux ###

在某些情况下，Linux 安装程序可能无法在初始 ramdisk（initrd 或 initramfs）中包含 Hyper-V 驱动程序，除非它检测到它正在运行 Hyper-V 环境。使用不同虚拟化系统（即 Virtualbox、KVM 等）来准备 Linux 映像时，可能需要重新生成 initrd 以确保至少 `hv_vmbus` 和 `hv_storvsc` 内核模块可在初始 ramdisk 上使用。至少在基于上游 Red Hat 分发的系统上这是一个已知问题。

重新生成 initrd 或 initramfs 映像的机制可能会因分发而有所不同。请查阅分发的文档或相应过程的支持。下面是有关如何使用 `mkinitrd` 实用工具重新生成 initrd 的示例：

首先，备份现有 initrd 映像：

	# cd /boot
	# sudo cp initrd-`uname -r`.img  initrd-`uname -r`.img.bak

接下来，使用 `hv_vmbus` 和 `hv_storvsc` 内核模块重新生成 initrd：

	# sudo mkinitrd --preload=hv_storvsc --preload=hv_vmbus -v -f initrd-`uname -r`.img `uname -r`


### 调整 VHD 大小 ###

Azure 上的 VHD 映像必须已将虚拟大小调整为 1MB。通常情况下，使用 Hyper-V 创建的 VHD 应已正确调整。如果未正确调整 VHD，则在你尝试基于 VHD 创建映像时，可能会收到如下错误消息：

	"The VHD http://<mystorageaccount>.blob.core.chinacloudapi.cn/vhds/MyLinuxVM.vhd has an unsupported virtual size of 21475270656 bytes. The size must be a whole number (in MBs)."

若要修正此问题，可使用 Hyper-V 管理器控制台或 [Resize-VHD](http://technet.microsoft.com/zh-cn/library/hh848535.aspx) Powershell cmdlet 调整 VM 大小。如果你未在 Windows 环境中运行，则建议使用 qemu-img 转换（如果需要）并调整 VHD 大小。

> [AZURE.NOTE] qemu-img 版本（>=2.2.1）中有一个已知 bug，会导致 VHD 格式不正确。我们会在即将发布的 qemu-img 版本中解决此问题。现在建议使用 qemu-img 版本 2.2.0 或较低版本。参考：https://bugs.launchpad.net/qemu/+bug/1490611


 1. 直接使用工具（如 `qemu-img` 或 `vbox-manage`）调整 VHD 大小可能会生成无法启动的 VHD。因此，建议先将 VHD 转换为 RAW 磁盘映像。如果已将 VM 映像创建为 RAW 磁盘映像（对于 KVM 等某些虚拟机监控程序，这是默认设置），则可以跳过此步骤：

		# qemu-img convert -f vpc -O raw MyLinuxVM.vhd MyLinuxVM.raw

 2. 计算磁盘映像所需的大小，以确保虚拟大小已调整为 1MB。以下 bash shell 脚本可以对此有帮助。此脚本使用“`qemu-img info`”来确定磁盘映像的虚拟大小，然后将大小计算到下个 1MB：

		rawdisk="MyLinuxVM.raw"
		vhddisk="MyLinuxVM.vhd"

		MB=$((1024*1024))
		size=$(qemu-img info -f raw --output json "$rawdisk" | \
		       gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

		rounded_size=$((($size/$MB + 1)*$MB))
		echo "Rounded Size = $rounded_size"

 3. 使用 $rounded\_size 调整原始磁盘大小，如上述脚本所设置：

		# qemu-img resize MyLinuxVM.raw $rounded_size

 4. 现在，将 RAW 磁盘重新转换为固定大小 VHD：

		# qemu-img convert -f raw -o subformat=fixed -O vpc MyLinuxVM.raw MyLinuxVM.vhd



##<a name="linux-kernel-requirements"></a> Linux 内核要求 ##

Hyper-V 和 Azure 的 Linux 集成服务 (LIS) 驱动程序会直接影响上游 Linux 内核。包括最新 Linux 内核版本（即 3.x）在内的许多分发已提供这些驱动程序，或以其他方式为其内核提供了这些驱动程序的向后移植版本。这些驱动程序会不断地在上游内核中使用新的修补程序和功能进行更新，因此，如果可能，请运行[认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)以包含这些修补程序和更新。

如果你正在运行 Red Hat Enterprise Linux 版本 **6.0-6.3** 的一个变体，则需要为 Hyper-V 安装最新的 LIS 驱动程序。可[在此处](http://www.microsoft.com/zh-cn/download/search.aspx?q=linux%20integration%20services)找到这些驱动程序。从 RHEL **6.4+**（和派生产品）开始，LIS 驱动程序已包含在内核中，因此，无需其他安装包即在 Azure 上运行这些系统。

如果需要自定义内核，建议使用较新的内核版本（即 **3.8+**）。对于这些分发或维护自己的内核的供应商，将需要执行一些操作，以便定期将 LIS 驱动程序从上游内核向后移植到自定义内核。即使你已运行相对较新的内核版本，也强烈建议你跟踪 LIS 驱动程序中的任何上游修复，并根据需要向后移植这些修复。LIS 驱动程序源文件的位置可在 Linux 内核源树中的 [MAINTAINER](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/MAINTAINERS) 文件中找到：

	F:	arch/x86/include/asm/mshyperv.h
	F:	arch/x86/include/uapi/asm/hyperv.h
	F:	arch/x86/kernel/cpu/mshyperv.c
	F:	drivers/hid/hid-hyperv.c
	F:	drivers/hv/
	F:	drivers/input/serio/hyperv-keyboard.c
	F:	drivers/net/hyperv/
	F:	drivers/scsi/storvsc_drv.c
	F:	drivers/video/fbdev/hyperv_fb.c
	F:	include/linux/hyperv.h
	F:	tools/hv/

至少，缺少以下修补程序已知会在 Azure 上导致问题，因此必须在内核中包含这些修补程序。对于所有分发而言，此列表绝并非详尽或完整：

- [ata\_piix：默认情况下，将磁盘交由 Hyper-V 驱动程序处理](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/ata/ata_piix.c?id=cd006086fa5d91414d8ff9ff2b78fbb593878e3c)
- [storvsc：解释 RESET 路径中传输中的数据包](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/drivers/scsi/storvsc_drv.c?id=5c1b10ab7f93d24f29b5630286e323d1c5802d5c)
- [storvsc：避免使用 WRITE\_SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=3e8f4f4065901c8dfc51407e1984495e1748c090)
- [storvsc：对 RAID 和虚拟主机适配器驱动程序禁用 WRITE SAME](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=54b2b50c20a61b51199bedb6e5d2f8ec2568fb43)
- [storvsc：NULL 指针取消引用修补程序](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=b12bb60d6c350b348a4e1460cd68f97ccae9822e)
- [storvsc：环形缓冲区故障可能会导致 I/O 冻结](https://git.kernel.org/cgit/linux/kernel/git/next/linux-next.git/commit/drivers/scsi/storvsc_drv.c?id=e86fb5e8ab95f10ec5f2e9430119d5d35020c951)


## Azure Linux 代理 ##

[Azure Linux 代理](/documentation/articles/virtual-machines-linux-agent-user-guide/) (waagent) 是在 Azure 中正确设置 Linux 虚拟机所必需的。你可以在 [Linux 代理 GitHub 存储库](https://github.com/Azure/WALinuxAgent)中获取最新版本、文件问题或提交拉取请求。

- 根据 Apache 2.0 许可证发布 Linux 代理。许多分发已为该代理提供 RPM 或 deb 包，因此，在某些情况下不费吹灰之力即可安装和更新该代理。

- Azure Linux 代理需要 Python v2.6 以上版本。

- 该代理还需要 python-pyasn1 模块。大多数分发提供此模块作为可以安装的单独包。

- 在某些情况下，Azure Linux 代理可能与 NetworkManager 不兼容。分发提供的许多 RPM/Deb 包会将 NetworkManager 配置为与 waagent 包冲突，因此当你安装 Linux 代理包时将卸载 NetworkManager。


## 一般 Linux 系统要求 ##

- 修改 GRUB 或 GRUB2 中的内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

		console=ttyS0 earlyprintk=ttyS0 rootdelay=300

	这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。

	除此之外，建议删除以下参数（如果存在）：

		rhgb quiet crashkernel=auto

	图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。

	根据需要可以配置 `crashkernel` 选项，但请注意此参数会使虚拟机中的可用内存量减少 128MB 或更多，这在较小的虚拟机上可能会出现问题。

- 安装 Azure Linux 代理

	Azure Linux 代理是在 Azure 上设置 Linux 映像所必需的。许多分发将该代理提供为 RPM 或 Deb 包（该包通常称为“WALinuxAgent”或“walinuxagent”）。还可以按照 [Linux 代理指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)中的步骤手动安装该代理。

- 请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

- 不要在操作系统磁盘上创建交换空间

	Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并可能在取消设置虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

		ResourceDisk.Format=y
		ResourceDisk.Filesystem=ext4
		ResourceDisk.MountPoint=/mnt/resource
		ResourceDisk.EnableSwap=y
		ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

- 在“/etc/sudoers”中，必须删除或注释掉以下行（如果存在）：

		Defaults targetpw
		ALL    ALL=(ALL) ALL

- 最后一步，请运行以下命令以取消设置虚拟机：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

	>[AZURE.NOTE] 运行 'waagent -force -deprovision' 之后，你可以在 Virtualbox 上看到以下错误：`[Errno 5] Input/output error`。此错误消息并不关键，可以忽略。

- 然后，需要关闭虚拟机并将 VHD 上载到 Azure。

<!---HONumber=Mooncake_0606_2016-->