<properties
	pageTitle="准备 Debian Linux VHD | Azure"
	description="了解如何创建 Debian 7 和 8 的 VHD 文件，以便在 Azure 中进行部署。"
	services="virtual-machines"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.date="12/01/2015"
	wacn.date="01/29/2016"/>




#为 Azure 准备 Debian VHD

##先决条件
本部分假定你已经将从 [Debian 网站](https://www.debian.org/distrib/)下载的 .iso 文件中的 Debian Linux 操作系统安装到虚拟硬盘。可以使用多种现有的工具来创建 .vhd 文件；Hyper-V 只是一个示例。有关 Hyper-V 的使用说明，请参阅[安装 Hyper-V 角色和配置虚拟机](https://technet.microsoft.com/zh-cn/library/hh846766.aspx)。


## 安装说明

- Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 **convert-vhd** cmdlet 将磁盘转换为 VHD 格式。
- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 RAID 可以在数据磁盘上使用。
- 不要在操作系统磁盘上配置交换分区。可以配置 Azure Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。
- 所有 VHD 的大小必须是 1 MB 的倍数。


##Debian 7.x 和 8.x

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击**“连接”**以打开该虚拟机的控制台窗口。

3. 如果你针对 ISO 文件设置了 VM，则请注释掉 `/etc/apt/source.list` 中对应于 **deb cdrom** 的行。

4. 编辑 `/etc/default/grub` 文件并将 **GRUB\_CMDLINE\_LINUX** 参数修改如下，以便包括用于 Azure 的其他内核参数。

        GRUB_CMDLINE_LINUX="console=ttyS0 earlyprintk=ttyS0 rootdelay=300"

5. 重新生成 grub 并运行：

        # sudo update-grub 

6. 安装 Azure Linux 代理的依赖项包：

        # apt-get install -y git parted

7.	根据[指南](/documentation/articles/virtual-machines-linux-update-agent)安装 Github 提供的 Azure Linux 代理，选择版本 2.0.14：

			# wget https://raw.githubusercontent.com/Azure/WALinuxAgent/WALinuxAgent-2.0.14/waagent
			# chmod +x waagent
			# cp waagent /usr/sbin
			# /usr/sbin/waagent -install -verbose

8. 取消对虚拟机的预配并对其进行准备，以便在 Azure 上进行预配并运行：

        # sudo waagent –force -deprovision
        # export HISTSIZE=0
        # logout
 
9. 在 Hyper-V 管理器中单击“操作”->“关闭”。Linux VHD 现已准备好上载到 Azure。

##使用 Credativ 脚本创建 Debian VHD

Credativ 网站上提供了一个脚本，该脚本可以帮助你自动生成 Debian VHD。你可以在[此处](https://gitlab.credativ.com/de/azure-manage)下载该脚本并将其安装在 Linux VM 中。若要创建 Debian VHD（例如 Debian 7），请运行：

        # azure_build_image --option release=wheezy --option image_prefix=lilidebian7 --option image_size_gb=30 SECTION

如果使用此脚本时出现问题，请单击[此处](https://gitlab.credativ.com/groups/de/issues)将问题报告给 Credativ。

## 后续步骤

现在，你已准备就绪，可以使用 Debian .vhd 来创建新的 Azure 虚拟机了。

<!---HONumber=Mooncake_0118_2016-->