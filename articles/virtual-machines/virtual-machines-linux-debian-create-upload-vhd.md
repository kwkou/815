<properties
	pageTitle="准备 Debian Linux VHD | Azure"
	description="了解如何创建 Debian 7 和 8 的 VHD 文件，以便在 Azure 中进行部署。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="szarkos"
	manager="timlt"
	editor=""
    tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/09/2016"
	wacn.date="06/29/2016"/>




# 为 Azure 准备 Debian VHD

## 先决条件
本部分假定你已经将从 [Debian 网站](https://www.debian.org/distrib/)下载的 .iso 文件中的 Debian Linux 操作系统安装到虚拟硬盘。可以使用多种现有的工具来创建 .vhd 文件；Hyper-V 只是一个示例。有关 Hyper-V 的使用说明，请参阅[安装 Hyper-V 角色和配置虚拟机](https://technet.microsoft.com/zh-cn/library/hh846766.aspx)。


## 安装说明

- 请同时阅读[通用 Linux 安装笔记](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)，以查看更多为 Azure 准备 Linux 的提示。
- Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 **convert-vhd** cmdlet 将磁盘转换为 VHD 格式。
- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 RAID 可以在数据磁盘上使用。
- 不要在操作系统磁盘上配置交换分区。可以配置 Azure Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。
- 所有 VHD 的大小必须是 1 MB 的倍数。


## 使用 Azure-Manage 来创建 Debian VHD

有工具可用于生成适用于 Azure 的 Debian VHD，如来自 [credativ](http://www.credativ.com/) 的 [azure-manage](https://gitlab.credativ.com/de/azure-manage) 脚本。这是建议的方法，而不是从头开始创建映像。例如，要创建 Debian 8 VHD，运行以下命令来下载 azure-manage（以及依赖关系），并运行 azure\_build\_image 脚本：

	# sudo apt-get update
	# sudo apt-get install git qemu-utils mbr kpartx debootstrap

	# sudo apt-get install python3-pip
	# sudo pip3 install azure-storage azure-servicemanagement-legacy pytest pyyaml
	# git clone https://gitlab.credativ.com/de/azure-manage.git
	# cd azure-manage
	# sudo pip3 install .

	# sudo azure_build_image --option release=jessie --option image_size_gb=30 --option image_prefix=debian-jessie-azure section


## 手动准备 Debian VHD

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击“连接”以打开该虚拟机的控制台窗口。

3. 如果你针对 ISO 文件设置了 VM，则请注释掉 `/etc/apt/source.list` 中对应于 “deb cdrom” 的行。

4. 编辑 `/etc/default/grub` 文件并按如下方式修改 “GRUB\_CMDLINE\_LINUX” 参数，以包括用于 Azure 的其他内核参数。

        GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200 earlyprintk=ttyS0,115200 rootdelay=30"

5. 重新生成 grub 并运行：

        # sudo update-grub

6. 将 Debian 的 Azure 存储库添加到 Debian 7 或 8 的 /etc/apt/sources.list 中：

	**Debian 7.x "Wheezy"**

		deb http://debian-archive.trafficmanager.cn/debian wheezy-backports main
		deb-src http://debian-archive.trafficmanager.cn/debian wheezy-backports main
		deb http://debian-archive.trafficmanager.cn/debian-azure wheezy main
		deb-src http://debian-archive.trafficmanager.cn/debian-azure wheezy main


	**Debian 8.x "Jessie"**

		deb http://debian-archive.trafficmanager.cn/debian jessie-backports main
		deb-src http://debian-archive.trafficmanager.cn/debian jessie-backports main
		deb http://debian-archive.trafficmanager.cn/debian-azure jessie main
		deb-src http://debian-archive.trafficmanager.cn/debian-azure jessie main


7. 安装 Azure Linux 代理：

		# sudo apt-get update
		# sudo apt-get install waagent

8. 对于 Debian 7，需要从 wheezy-backports 存储库运行基于 3.16 的内核。首先使用以下内容创建名为 /etc/apt/preferences.d/linux.pref 的文件：

		Package: linux-image-amd64 initramfs-tools
		Pin: release n=wheezy-backports
		Pin-Priority: 500

	然后运行“sudo apt-get install linux-image-amd64”以安装新的内核。

8. 取消对虚拟机的预配并对其进行准备，以便在 Azure 上进行预配并运行：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

9. 在 Hyper-V 管理器中单击“操作”>“关闭”。Linux VHD 现已准备好上载到 Azure。


## 后续步骤

现在，你可以使用 Debian 虚拟硬盘在 Azure 中创建新的 Azure 虚拟机了。如果这是你第一次将 .vhd 文件上载到 Azure，请参阅[创建并上载包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的步骤 2 和步骤 3。

<!---HONumber=Mooncake_0509_2016-->