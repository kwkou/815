<properties urlDisplayName="Upload an Ubuntu Linux VHD" pageTitle="在 Azure 中创建和上载 Ubuntu Linux VHD" metaKeywords="Azure VHD, uploading Linux VHD, Ubuntu" description="了解如何创建和上载包含 Ubuntu Linux 操作系统的 Azure 虚拟硬盘 (VHD)。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Creating and Uploading a Virtual Hard Disk that Contains an Ubuntu Linux Operating System" authors="szarkos" solutions="" manager="timlt" editor="tysonn" />

<tags ms.service="virtual-machines" ms.date="05/15/2015" wacn.date="06/19/2015"/>

# 为 Azure 准备 Ubuntu 虚拟机

## 先决条件##

本文假定你已在虚拟硬盘中安装了 Ubuntu Linux 操作系统。存在多个用于创建 .vhd 文件的工具，例如 Hyper-V 等虚拟化解决方案。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

**Ubuntu 安装说明**

- Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。

- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid) 可以在数据磁盘上使用。

- 不要在操作系统磁盘上配置交换分区。可以配置 Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。

- 所有 VHD 的大小必须是 1 MB 的倍数。


## <a id="ubuntu"></a>Ubuntu 12.04+ ##

1. 在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2. 单击**“连接”**以打开虚拟机窗口。

3.	替换映像中的当前存储库，以使用 Ubuntu 的 Azure 存储库。这些步骤可能会由于 Ubuntu 版本的不同而稍有差异。

	在编辑 /etc/apt/sources.list 之前，建议进行备份

		# sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

	Ubuntu 12.04：

		# sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
		# sudo apt-add-repository 'http://archive.canonical.com/ubuntu precise-backports main'
		# sudo apt-get update

	Ubuntu 12.10：

		# sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
		# sudo apt-add-repository 'http://archive.canonical.com/ubuntu quantal-backports main'
		# sudo apt-get update

	Ubuntu 14.04+：

		# sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
		# sudo apt-get update

4. 通过运行以下命令将操作系统更新为最新内核：

	Ubuntu 12.04：

		# sudo apt-get update
		# sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-precise-virtual
		(recommended) sudo apt-get dist-upgrade

		# sudo reboot

	Ubuntu 12.10：

		# sudo apt-get update
		# sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-quantal-virtual
		(recommended) sudo apt-get dist-upgrade

		# sudo reboot
	
	Ubuntu 14.04+：

		# sudo apt-get update
		# sudo apt-get install hv-kvp-daemon-init
		(recommended) sudo apt-get dist-upgrade

		# sudo reboot

5.	（可选）如果 Ubuntu 系统遇到错误并重新启动，则它通常将等待 grub 启动提示用户输入以阻止系统正常启动。若要防止此情况发生，请完成下列步骤：

	a) 打开 /etc/grub.d/00_header 文件。

	b) 在 **make_timeout()** 函数中，搜索 **if ["\${recordfail}" = 1 ]; then**

	c) 将该行下的语句更改为 **set timeout=5**。

	d) 运行“sudo update-grub”。

6. 修改 Grub 的内核引导行以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/etc/default/grub”、找到名为“`GRUB_CMDLINE_LINUX_DEFAULT`”的变量（或根据需要添加它）并对它进行编辑以使其包含以下参数：

		GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"

	保存并关闭此文件，然后再运行“`sudo update-grub`”。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 技术支持人员调试问题。

8.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

9.	安装 Azure Linux 代理：

		# sudo apt-get update
		# sudo apt-get install walinuxagent

	请注意，安装 `walinuxagent` 包时将删除 `NetworkManager` 和 `NetworkManager-gnome` 包（如果已安装它们）。

10.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

11. 在 Hyper-V 管理器中单击**“操作”->“关闭”**。Linux VHD 现已准备好上载到 Azure。

<!---HONumber=60-->