<properties urlDisplayName="Upload a SUSE Linux VHD" pageTitle="在 Azure 中创建和上载 SUSE Linux VHD" metaKeywords="Azure VHD, uploading Linux VHD, SUSE, SLES, openSUSE" description="了解如何创建和上载包含 SUSE Linux 操作系统的 Azure 虚拟硬盘 (VHD)。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Creating and Uploading a Virtual Hard Disk that Contains a SUSE Linux Operating System" authors="szarkos" solutions="" manager="timlt" editor="tysonn" />

<tags ms.service="virtual-machines" ms.date="05/15/2015" wacn.date="06/19/2015"/>

# 为 Azure 准备 SLES 或 openSUSE 虚拟机

- [为 Azure 准备 SLES 11 SP3 虚拟机](#sles11)
- [为 Azure 准备 openSUSE 13.1+ 虚拟机](#osuse)

## 先决条件

本文假定你已在虚拟硬盘中安装了 SUSE 或 openSUSE Linux 操作系统。存在多个用于创建 .vhd 文件的工具，例如 Hyper-V 等虚拟化解决方案。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。


**SLES/openSUSE 安装说明**

 - [SUSE Studio](http://www.susestudio.com) 可以轻松地创建和管理 Azure 和 Hyper-V 的 SLES/openSUSE 映像。这是自定义你自己的 SUSE 和 openSUSE 映像的推荐方法。SUSE Studio 库中的以下正式映像可下载或克隆到你自己的 SUSE Studio 中：

  - [SUSE Studio 库中的 SLES 11 SP3 for Azure](http://susestudio.com/a/02kbT4/sles-11-sp3-for-windows-azure)
  - [SUSE Studio 库中的 openSUSE 13.1 for Azure](https://susestudio.com/a/02kbT4/opensuse-13-1-for-windows-azure)

- Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。

- 在安装 Linux 系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid) 可以在数据磁盘上使用。

- 不要在操作系统磁盘上配置交换分区。可以配置 Linux 代理，以在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此内容的详细信息。

- 所有 VHD 的大小必须是 1 MB 的倍数。


## <a id="sles11"></a>准备 SUSE Linux Enterprise Server 11 SP3 ##

1. 在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2. 单击**“连接”**以打开虚拟机窗口。

3. 注册 SUSE Linux Enterprise 系统以允许其下载更新并安装程序包。

4. 使用最新修补程序更新系统：

		# sudo zypper update

5. 从 SLES 存储库安装 Azure Linux 代理：

		# sudo zypper install WALinuxAgent

6. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/boot/grub/menu.lst”，并确保默认内核包含以下参数：

		console=ttyS0 earlyprintk=ttyS0 rootdelay=300

	这将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。

7.	建议编辑文件“/etc/sysconfig/network/dhcp”，并将 `DHCLIENT_SET_HOSTNAME` 参数更改为以下值：

		DHCLIENT_SET_HOSTNAME="no"

8.	在“/etc/sudoers”中，注释掉或删除以下行（如果存在）：

		Defaults targetpw   # ask for the password of the target user i.e. root
		ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!

9.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

10. 不要在操作系统磁盘上创建交换空间

	Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是*临时*磁盘，并可能在取消设置虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

		ResourceDisk.Format=y
		ResourceDisk.Filesystem=ext4
		ResourceDisk.MountPoint=/mnt/resource
		ResourceDisk.EnableSwap=y
		ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

11.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

12. 在 Hyper-V 管理器中单击**“操作”->“关闭”**。Linux VHD 现已准备好上载到 Azure。


----------

## <a id="osuse"></a>准备 openSUSE 13.1+ ##

1. 在 Hyper-V 管理器的中间窗格中，选择虚拟机

2. 单击**“连接”**以打开虚拟机窗口

3. 在 shell 上，运行命令“`zypper lr`”。如果此命令返回类似于以下内容的输出（请注意版本号可能会有所不同）：

		# | Alias                 | Name                  | Enabled | Refresh
		--+-----------------------+-----------------------+---------+--------
		1 | Cloud:Tools_13.1      | Cloud:Tools_13.1      | Yes     | Yes
		2 | openSUSE_13.1_OSS     | openSUSE_13.1_OSS     | Yes     | Yes
		3 | openSUSE_13.1_Updates | openSUSE_13.1_Updates | Yes     | Yes

	则按预期配置存储库，不需要进行任何调整。

	如果该命令返回“未定义存储库...”，则使用以下命令来添加这些存储库：

		# sudo zypper ar -f http://download.opensuse.org/repositories/Cloud:Tools/openSUSE_13.1 Cloud:Tools_13.1 
		# sudo zypper ar -f http://download.opensuse.org/distribution/13.1/repo/oss openSUSE_13.1_OSS
		# sudo zypper ar -f http://download.opensuse.org/update/13.1 openSUSE_13.1_Updates

	然后，可以通过再次运行命令“`zypper lr`”来验证已添加存储库。如果未启用某个相关的更新存储库，请使用以下命令启用该存储库：

		# sudo zypper mr -e [NUMBER OF REPOSITORY]


4. 将内核更新为可用的最新版本：

		# sudo zypper up kernel-default

	或使用所有最新修补程序更新系统：

		# sudo zypper update

5.	安装 Azure Linux 代理

		# sudo zypper install WALinuxAgent

6.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/boot/grub/menu.lst”，并确保默认内核包含以下参数：

		console=ttyS0 earlyprintk=ttyS0 rootdelay=300

	这将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。此外，从内核引导行删除以下参数（如果它们存在）：

		libata.atapi_enabled=0 reserve=0x1f0,0x8

7.	建议编辑文件“/etc/sysconfig/network/dhcp”，并将 `DHCLIENT_SET_HOSTNAME` 参数更改为以下值：

		DHCLIENT_SET_HOSTNAME="no"

8.	**重要提示：**在“/etc/sudoers”中，注释掉或删除以下行（如果存在）：

		Defaults targetpw   # ask for the password of the target user i.e. root
		ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!

9.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

10.	不要在操作系统磁盘上创建交换空间

	Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是*临时*磁盘，并可能在取消设置虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

		ResourceDisk.Format=y
		ResourceDisk.Filesystem=ext4
		ResourceDisk.MountPoint=/mnt/resource
		ResourceDisk.EnableSwap=y
		ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

11.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

		# sudo waagent -force -deprovision
		# export HISTSIZE=0
		# logout

12. 确保在启动时运行 Azure Linux 代理：

		# sudo systemctl enable waagent.service

13. 在 Hyper-V 管理器中单击**“操作”->“关闭”**。Linux VHD 现已准备好上载到 Azure。

<!---HONumber=60-->