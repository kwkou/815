<properties
	pageTitle="创建和上载 Oracle Linux VHD | Azure"
	description="了解如何创建和上载包含 Oracle Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="szarkos"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management,azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/09/2016"
	wacn.date="06/29/2016"/>

# 为 Azure 准备 Oracle Linux 虚拟机


[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## 先决条件 ##

本文假定你已在虚拟硬盘中安装了 Oracle Linux 操作系统。存在多个用于创建 .vhd 文件的工具，例如 Hyper-V 等虚拟化解决方案。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。


### Oracle Linux 安装说明

- 请同时阅读[通用 Linux 安装笔记](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)，以查看更多为 Azure 准备 Linux 的提示。

- Hyper-V 和 Azure 同时支持 Oracle 的 Red Hat 兼容内核及其 UEK3（坚不可摧企业内核）。为了获得最佳结果，请务必在准备 Oracle Linux VHD 时更新到最新内核。

- Hyper-V 和 Azure 不支持 Oracle 的 UEK2，因为它不包括所需的驱动程序。

- Azure 不支持 VHDX 格式，仅支持**固定大小的 VHD**。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。

- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid/) 可以在数据磁盘上使用。

- 由于低于 2.6.37 的 Linux 内核版本中的 bug，更大的 VM 不支持 NUMA。此问题主要影响使用上游 Red Hat 2.6.32 内核的分发。手动安装的 Azure Linux 代理 (waagent) 将自动在 Linux 内核的 GRUB 配置中禁用 NUMA。可以在下面的步骤中找到有关此内容的详细信息。

- 不要在操作系统磁盘上配置交换分区。可以配置 Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。

- 所有 VHD 的大小必须是 1 MB 的倍数。

- 确保 `Addons` 仓库是激活的。修改文件 `/etc/yum.repo.d/public-yum-ol6.repo`(Oracle Linux 6) 或者 `/etc/yum.repo.d/public-yum-ol7.repo`(Oracle Linux)，文件中在 **[ol6_addons]** 或者 **[ol7_addons]** 下，把 `enabled=0` 改成 `enabled=1`。

## <a id="oracle6"> </a> Oracle Linux 6.4+ ##

您必须在操作系统中完成特定的配置步骤才能使虚拟机在 Azure 中运行。

1. 在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2. 单击**“连接”**以打开虚拟机窗口。

3. 通过运行以下命令卸载 NetworkManager：

		# sudo rpm -e --nodeps NetworkManager

	**注意：**如果尚未安装此包，则该命令将失败，并显示一条错误消息。这是正常情况。

4.	在包含以下文本的 `/etc/sysconfig/` 目录中创建一个名为 **network** 的文件：

		NETWORKING=yes
		HOSTNAME=localhost.localdomain

5.	在包含以下文本的 `/etc/sysconfig/network-scripts/` 目录中创建一个名为 **ifcfg-eth0** 的文件：

		DEVICE=eth0
		ONBOOT=yes
		BOOTPROTO=dhcp
		TYPE=Ethernet
		USERCTL=no
		PEERDNS=yes
		IPV6INIT=no

6.	修改 udev 规则，以避免产生以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

		# sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
		# sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. 通过运行以下命令，确保网络服务将在引导时启动：

		# chkconfig network on

8. 通过运行以下命令安装 python-pyasn1：

		# sudo yum install python-pyasn1

9.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/boot/grub/menu.lst”，并确保默认内核包含以下参数：

		console=ttyS0 earlyprintk=ttyS0 rootdelay=300 numa=off

	这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。由于 Oracle 的 Red Hat 兼容内核中的 bug，这将禁用 NUMA。

	除此之外，建议删除以下参数：

		rhgb quiet crashkernel=auto

	图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。

	根据需要可以配置 `crashkernel` 选项，但请注意此参数会使虚拟机中的可用内存量减少 128MB 或更多，这在较小的虚拟机上可能会出现问题。


10.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

11. 通过运行以下命令来安装 Azure Linux 代理。最新版本是 2.0.15。

		# sudo yum install WALinuxAgent

	请注意，如果没有如步骤 2 中所述删除 NetworkManager 包和 NetworkManager-gnome 包，则安装 WALinuxAgent 包将删除它们。

12.	不要在 OS 磁盘上创建交换空间。

	Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并可能在取消设置虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

		ResourceDisk.Format=y
		ResourceDisk.Filesystem=ext4
		ResourceDisk.MountPoint=/mnt/resource
		ResourceDisk.EnableSwap=y
		ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

14. 在 Hyper-V 管理器中单击**“操作”->“关闭”**。Linux VHD 现已准备好上载到 Azure。


----------


## <a id="oracle7"> </a> Oracle Linux 7.0+ ##

**Oracle Linux 7 中的更改**

为 Azure 准备 Oracle Linux 7 虚拟机非常类似于 Oracle Linux 6，但有几个值得注意的重要区别：

 - 在 Azure 中同时支持 Red Hat 兼容内核和 Oracle 的 UEK3。建议使用 UEK3 内核。
 - NetworkManager 包不再与 Azure Linux 代理冲突。默认情况下将安装此包，建议你不要删除它。
 - GRUB2 现在用作默认引导加载程序，因此用于编辑内核参数的过程已更改（请参见下文）。
 - XFS 现在是默认文件系统。如果需要，仍可以使用 ext4 文件系统。


**配置步骤**

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击**“连接”**以打开该虚拟机的控制台窗口。

3.	在包含以下文本的 `/etc/sysconfig/` 目录中创建一个名为 **network** 的文件：

		NETWORKING=yes
		HOSTNAME=localhost.localdomain

4.	在包含以下文本的 `/etc/sysconfig/network-scripts/` 目录中创建一个名为 **ifcfg-eth0** 的文件：

		DEVICE=eth0
		ONBOOT=yes
		BOOTPROTO=dhcp
		TYPE=Ethernet
		USERCTL=no
		PEERDNS=yes
		IPV6INIT=no

5.	修改 udev 规则，以避免产生以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

		# sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

6. 通过运行以下命令，确保网络服务将在引导时启动：

		# sudo chkconfig network on

7. 通过运行以下命令安装 python-pyasn1 包：

		# sudo yum install python-pyasn1

8.	运行以下命令以清除当前 yum 元数据并安装所有更新：

		# sudo yum clean all
		# sudo yum -y update

9.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/etc/default/grub”并编辑 `GRUB_CMDLINE_LINUX` 参数，例如：

		GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"

	这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。除此之外，建议删除以下参数：

		rhgb quiet crashkernel=auto

	图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。

	根据需要可以配置 `crashkernel` 选项，但请注意此参数会使虚拟机中的可用内存量减少 128MB 或更多，这在较小的虚拟机上可能会出现问题。


10. 完成后，请按照上面所示编辑“/etc/default/grub”，运行以下命令以重新生成 grub 配置：

		# sudo grub2-mkconfig -o /boot/grub2/grub.cfg

11.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

12. 通过运行以下命令来安装 Azure Linux 代理：

		# sudo yum install WALinuxAgent
		# sudo systemctl enable waagent

13.	不要在 OS 磁盘上创建交换空间。

	Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并可能在取消设置虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

		ResourceDisk.Format=y
		ResourceDisk.Filesystem=ext4
		ResourceDisk.MountPoint=/mnt/resource
		ResourceDisk.EnableSwap=y
		ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

14.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

15. 在 Hyper-V 管理器中单击**“操作”->“关闭”**。Linux VHD 现已准备好上载到 Azure。


## 后续步骤
现在，你已准备就绪，可以使用 Oracle Linux .vhd 在 Azure 中创建新的虚拟机了。如果这是你第一次将 .vhd 文件上载到 Azure，请参阅 [创建并上载包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的步骤 2 和步骤 3。

<!---HONumber=Mooncake_0321_2016-->