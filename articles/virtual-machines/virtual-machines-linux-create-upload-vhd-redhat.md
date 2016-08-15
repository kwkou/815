<properties
	pageTitle="创建并上载 Red Hat Enterprise Linux VHD，以供在 Azure 中使用"
	description="了解如何创建和上载包含 Red Hat Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor="tysonn"
    tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/17/2016"
	wacn.date="05/12/2016"/>


# 为 Azure 准备基于 Red Hat 的虚拟机

在本文中，你将了解如何准备 Red Hat Enterprise Linux (RHEL) 虚拟机，以供在 Azure 中使用。本文所述的 RHEL 版本为 6.7、7.1 和 7.2。本文所述的用于准备工作的虚拟机监控程序为 Hyper-V、基于内核的虚拟机 (KVM) 和 VMware。有关参与 Red Hat 云访问计划的资格要求的详细信息，请参阅 [Red Hat 的云访问网站](http://www.redhat.com/en/technologies/cloud-computing/cloud-access)和[在 Azure 上运行 RHEL](https://access.redhat.com/articles/1989673)。

[从 Hyper-V 管理器准备 RHEL 6.7 虚拟机](#rhel67hyperv)

[从 Hyper-V 管理器准备 RHEL 7.1/7.2 虚拟机](#rhel7xhyperv)

[从 KVM 准备 RHEL 6.7 虚拟机](#rhel67kvm)

[从 KVM 准备 RHEL 7.1/7.2 虚拟机](#rhel7xkvm)

[从 VMware 准备 RHEL 6.7 虚拟机](#rhel67vmware)

[从 VMware 准备 RHEL 7.1/7.2 虚拟机](#rhel7xvmware)

[从 kickstart 文件准备 RHEL 7.1/7.2 虚拟机](#rhel7xkickstart)


## 从 Hyper-V 管理器准备基于 Red Hat 的虚拟机
### 先决条件
本部分假定你已经将从 Red Hat 网站获取的 ISO 文件中的 RHEL 映像安装到虚拟硬盘 (VHD)。有关如何使用 Hyper-V 管理器来安装操作系统映像的更多详细信息，请参阅 [Install the Hyper-V Role and Configure a Virtual Machine（安装 Hyper-V 角色和配置虚拟机）](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

**RHEL 安装说明**

- Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 **convert-vhd** PowerShell cmdlet 将磁盘转换为 VHD 格式。

- 创建的 VHD 必须为“固定”形式，动态 VHD 不受支持。

- 在安装 Linux 操作系统时，建议使用标准分区而不是逻辑卷管理器 (LVM)（通常是许多安装的默认值）。这有助于避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果需要，你可以使用 LVM 或 RAID 数据磁盘。

- 不要在操作系统磁盘上配置交换分区。可将 Linux 代理配置为在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此操作的详细信息。

- 所有 VHD 的大小必须是 1 MB 的倍数。

- 使用 **qemu-img** 将磁盘映像转换成 VHD 格式时，请注意，2.2.1 及更高版本的 qemu-img 中存在一个已知的 bug。此 bug 会导致 VHD 格式不正常。我们有望在即将发布的 qemu-img 版本中解决此问题。我们建议你暂时使用 qemu-img 2.2.0 或更低版本。

### <a id="rhel67hyperv"></a>从 Hyper-V 管理器准备 RHEL 6.7 虚拟机###


1.	在 Hyper-V 管理器中，选择虚拟机。

2.	单击**“连接”**以打开该虚拟机的控制台窗口。

3.	通过运行以下命令卸载 NetworkManager：

        # sudo rpm -e --nodeps NetworkManager

    请注意，如果尚未安装此包，则该命令将失败，并显示一条错误消息。这是正常情况。

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

6.	移动（或删除）udev 规则，以避免产生以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # sudo mkdir -m 0700 /var/lib/waagent
        # sudo mv /lib/udev/rules.d/75-persistent-net-generator.rules /var/lib/waagent/
        # sudo mv /etc/udev/rules.d/70-persistent-net.rules /var/lib/waagent/

7.	通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

8.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

9.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

10.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/boot/grub/menu.lst`，并确保默认内核包含以下参数：

        console=ttyS0
        earlyprintk=ttyS0
        rootdelay=300
        numa=off

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。由于 RHEL 6 所使用的内核版本中存在 bug，因此这将禁用 NUMA。

    除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。

    可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

11.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。修改 /etc/ssh/sshd\_config 以包含以下行：

        ClientAliveInterval 180

12.	通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent
        # sudo chkconfig waagent on

    请注意，如果没有如步骤 2 中所述删除 NetworkManager 包和 NetworkManager-gnome 包，则安装 WALinuxAgent 包将删除它们。

13.	不要在 OS 磁盘上创建交换空间。
Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

14.	通过运行以下命令取消注册订阅（如有必要）：

        # sudo subscription-manager unregister

15.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

16.	在 Hyper-V 管理器中单击“操作”>“关闭”。Linux VHD 现已准备好上载到 Azure。

### <a id="rhel7xhyperv"></a>从 Hyper-V 管理器准备 RHEL 7.1/7.2 虚拟机###

1.  在 Hyper-V 管理器中，选择虚拟机。

2.	单击“连接”以打开该虚拟机的控制台窗口。

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

5.	通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

6.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/etc/default/grub` 并编辑 **GRUB\_CMDLINE\_LINUX** 参数。例如：

        GRUB_CMDLINE_LINUX="rootdelay=300
        console=ttyS0
        earlyprintk=ttyS0"

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

	图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

8.	完成编辑 `/etc/default/grub` 后，运行以下命令以重新生成 grub 配置：

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

9.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

10.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

11.	通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent
        # sudo systemctl enable waagent.service

12.	不要在 OS 磁盘上创建交换空间。Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。安装 Azure Linux 代理（请参阅上一步）后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13.	如果你想要撤消注册订阅，运行以下命令：

        # sudo subscription-manager unregister

14.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

15.	在 Hyper-V 管理器中单击“操作”>“关闭”。Linux VHD 现已准备好上载到 Azure。


## 从 KVM 准备基于 Red Hat 的虚拟机

### <a id="rhel67kvm"></a>从 KVM 准备 RHEL 6.7 虚拟机###


1.	从 Red Hat 网站上下载 RHEL 6.7 的 KVM 映像。

2.	设置 root 密码。

    生成加密密码，然后复制命令的输出：

        # openssl passwd -1 changeme

    使用 guestfish 设置根密码：

        # guestfish --rw -a <image-name>
        ><fs> run
        ><fs> list-filesystems
        ><fs> mount /dev/sda1 /
        ><fs> vi /etc/shadow
        ><fs> exit

	将 root 用户的第二个字段从“!!”更改为加密密码。

3.	在 KVM 中通过 qcow2 映像创建虚拟机，将磁盘类型设置为 **qcow2**，将虚拟网络接口设备型号设置为 **virtio**。然后启动虚拟机，并以 root 用户身份登录。

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

6.	移动（或删除）udev 规则，以避免产生以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # mkdir -m 0700 /var/lib/waagent
        # mv /lib/udev/rules.d/75-persistent-net-generator.rules /var/lib/waagent/
        # mv /etc/udev/rules.d/70-persistent-net.rules /var/lib/waagent/

7.	通过运行以下命令，确保网络服务将在引导时启动：

        # chkconfig network on

8.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # subscription-manager register --auto-attach --username=XXX --password=XXX

9.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/boot/grub/menu.lst`，并确保默认内核包含以下参数：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300 numa=off

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。由于 RHEL 6 所使用的内核版本中存在 bug，因此这将禁用 NUMA。

    除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

10. 将 Hyper-V 模块添加到 initramfs 中：

    编辑 `/etc/dracut.conf` 并添加以下内容：
    add\_drivers+="hv\_vmbus hv\_netvsc hv\_storvsc"

    重新生成 initramfs：
        # dracut -f -v

11.	卸载 cloud-init：

        # yum remove cloud-init

12.	确保已安装 SSH 服务器且已将其配置为在引导时启动。

        # chkconfig sshd on

    修改 /etc/ssh/sshd\_config 以包含以下行：

        PasswordAuthentication yes
        ClientAliveInterval 180

    重新启动 sshd：

		# service sshd restart

13.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

14.	通过运行以下命令来安装 Azure Linux 代理：

        # yum install WALinuxAgent
        # chkconfig waagent on

15.	Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 **/etc/waagent.conf** 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16.	通过运行以下命令取消注册订阅（如有必要）：

        # subscription-manager unregister

17.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # waagent -force -deprovision
        # export HISTSIZE=0
        # logout

18.	关闭 KVM 中的 VM。

19.	将 qcow2 映像转换为 VHD 格式。
    首先将此映像转换为原始格式：

         # qemu-img convert -f qcow2 -O raw rhel-6.7.qcow2 rhel-6.7.raw
    请确保原始映像大小为 1 MB。如果不是，请将大小四舍五入，使其等于 1 MB：
         # MB=$((1024*1024))
         # size=$(qemu-img info -f raw --output json "rhel-6.7.raw" | \
                  gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
         # rounded_size=$((($size/$MB + 1)*$MB))

         # qemu-img resize rhel-6.7.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

         # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.7.raw rhel-6.7.vhd


### <a id="rhel7xkvm"></a>从 KVM 准备 RHEL 7.1/7.2 虚拟机###


1.	从 Red Hat 网站上下载 RHEL 7.1（或 7.2）的 KVM 映像。我们将使用 RHEL 7.1 作为此处的示例。

2.	设置 root 密码。

    生成加密密码，然后复制命令的输出：

        # openssl passwd -1 changeme

    使用 guestfish 设置 root 密码。

        # guestfish --rw -a <image-name>
        ><fs> run
        ><fs> list-filesystems
        ><fs> mount /dev/sda1 /
        ><fs> vi /etc/shadow
        ><fs> exit

    将 root 用户的第二个字段从“!!”更改为加密密码。

3.	在 KVM 中通过 qcow2 映像创建虚拟机，将磁盘类型设置为 **qcow2**，将虚拟网络接口设备型号设置为 **virtio**。然后启动虚拟机，并以 root 用户身份登录。

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

6.	通过运行以下命令，确保网络服务将在引导时启动：

        # chkconfig network on

7.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # subscription-manager register --auto-attach --username=XXX --password=XXX

8.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/etc/default/grub` 并编辑 **GRUB\_CMDLINE\_LINUX** 参数。例如：

        GRUB_CMDLINE_LINUX="rootdelay=300
        console=ttyS0
        earlyprintk=ttyS0"

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

9.	完成编辑 `/etc/default/grub` 后，运行以下命令以重新生成 grub 配置：

        # grub2-mkconfig -o /boot/grub2/grub.cfg

10.	将 Hyper-V 模块添加到 initramfs 中：

    编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

11.	卸载 cloud-init：

        # yum remove cloud-init

12.	确保已安装 SSH 服务器且已将其配置为在引导时启动。

        # systemctl enable sshd

    修改 /etc/ssh/sshd\_config 以包含以下行：

        PasswordAuthentication yes
        ClientAliveInterval 180

    重新启动 sshd：

        systemctl restart sshd

13.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

14.	通过运行以下命令来安装 Azure Linux 代理：

        # yum install WALinuxAgent

    启用 waagent 服务：

        # systemctl enable waagent.service

15.	不要在 OS 磁盘上创建交换空间。Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。安装 Azure Linux 代理（请参阅上一步）后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16.	通过运行以下命令取消注册订阅（如有必要）：

        # subscription-manager unregister

17.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

18.	关闭 KVM 中的虚拟机。

19.	将 qcow2 映像转换为 VHD 格式。

    首先将此映像转换为原始格式：

         # qemu-img convert -f qcow2 -O raw rhel-7.1.qcow2 rhel-7.1.raw

    请确保原始映像大小为 1 MB。如果不是，请将大小四舍五入，使其等于 1 MB：

         # MB=$((1024*1024))
         # size=$(qemu-img info -f raw --output json "rhel-7.1.raw" | \
                  gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
         # rounded_size=$((($size/$MB + 1)*$MB))

         # qemu-img resize rhel-7.1.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

         # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.1.raw rhel-7.1.vhd


## 从 VMware 准备基于 Red Hat 的虚拟机
### 先决条件
本部分假设你已在 VMware 中安装了 RHEL 虚拟机。有关如何在 VMware 中安装操作系统的详细信息，请参阅 [VMware Guest Operating System Installation GuideVMware （来宾操作系统安装指南）](http://partnerweb.vmware.com/GOSIG/home.html)。

- 在安装 Linux 操作系统时，建议使用标准分区而不是 LVM（通常是许多安装的默认值）。这将避免 LVM 与克隆 VM 发生名称冲突，特别是在 OS 磁盘需要连接到另一台 VM 以进行故障排除的情况下。如果首选，LVM 或 RAID 可以在数据磁盘上使用。

- 不要在操作系统磁盘上配置交换分区。可将 Linux 代理配置为在临时资源磁盘上创建交换文件。可以在下面的步骤中找到有关此操作的详细信息。

- 当你创建虚拟硬盘时，选择“将虚拟磁盘存储为单个文件”。



### <a id="rhel67vmware"></a>从 VMware 准备 RHEL 6.7 虚拟机###

1.	通过运行以下命令卸载 NetworkManager：

         # sudo rpm -e --nodeps NetworkManager

    请注意，如果尚未安装此包，则该命令将失败，并显示一条错误消息。这是正常情况。

2.	在包含以下文本的 /etc/sysconfig/ 目录中创建一个名为 **network** 的文件：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

3.	在包含以下文本的 /etc/sysconfig/network-scripts/ 目录中创建一个名为 **ifcfg-eth0** 的文件：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

4.	移动（或删除）udev 规则，以避免产生以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # sudo mkdir -m 0700 /var/lib/waagent
        # sudo mv /lib/udev/rules.d/75-persistent-net-generator.rules /var/lib/waagent/
        # sudo mv /etc/udev/rules.d/70-persistent-net.rules /var/lib/waagent/

5.	通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

6.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

8.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开“/boot/grub/menu.lst”，并确保默认内核包含以下参数：

        console=ttyS0
        earlyprintk=ttyS0
        rootdelay=300
        numa=off

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。由于 RHEL 6 所使用的内核版本中存在 bug，因此这将禁用 NUMA。除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

9.	将 Hyper-V 模块添加到 initramfs 中：

	    Edit `/etc/dracut.conf` and add content:

	        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

	    Rebuild initramfs:

	        # dracut -f -v

10.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

11.	通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent
        # sudo chkconfig waagent on

12.	请勿在 OS 磁盘上创建交换空间：

    Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。安装 Azure Linux 代理（请参阅上一步）后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13.	通过运行以下命令取消注册订阅（如有必要）：

        # sudo subscription-manager unregister

14.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

15.	关闭 VM，并将 VMDK 文件转换为 .vhd 文件。

    首先将此映像转换为原始格式：

        # qemu-img convert -f vmdk -O raw rhel-6.7.vmdk rhel-6.7.raw

    请确保原始映像大小为 1 MB。如果不是，请将大小四舍五入，使其等于 1 MB：

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-6.7.raw" | \
                gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-6.7.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.7.raw rhel-6.7.vhd


### <a id="rhel7xvmware"></a>从 VMware 准备 RHEL 7.1/7.2 虚拟机###

1.	在包含以下文本的 /etc/sysconfig/ 目录中创建一个名为 **network** 的文件：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

2.	在包含以下文本的 /etc/sysconfig/network-scripts/ 目录中创建一个名为 **ifcfg-eth0** 的文件：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

3.	通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

4.	注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

5.	在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/etc/default/grub` 并编辑 **GRUB\_CMDLINE\_LINUX** 参数。例如：

        GRUB_CMDLINE_LINUX="rootdelay=300
        console=ttyS0
        earlyprintk=ttyS0"

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。除上述操作以外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。可以根据需要配置 crashkernel 选项，但请注意，此参数会使 VM 中的可用内存量减少 128 MB 或更多。这可能对于较小的 VM 大小有问题。

6.	完成编辑 `/etc/default/grub` 后，运行以下命令以重新生成 grub 配置：

         # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

7.	将 Hyper-V 模块添加到 initramfs 中：

    编辑 `/etc/dracut.conf`，添加内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

8.	请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

9.	WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

10.	通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent
        # sudo systemctl enable waagent.service

11.	不要在 OS 磁盘上创建交换空间。Azure Linux 代理可使用在 Azure 上预配 VM 后附加到 VM 的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是临时磁盘，并且当取消设置 VM 时可能会被清空。安装 Azure Linux 代理（请参阅上一步）后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

12.	如果你想要撤消注册订阅，运行以下命令：

        # sudo subscription-manager unregister

13.	运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

14.	关闭 VM，并将 VMDK 文件转换为 VHD 格式。

    首先将此映像转换为原始格式：

        # qemu-img convert -f vmdk -O raw rhel-7.1.vmdk rhel-7.1.raw

    请确保原始映像大小为 1 MB。如果不是，请将大小四舍五入，使其等于 1 MB：

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-7.1.raw" | \
                 gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')
        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-7.1.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.1.raw rhel-7.1.vhd


## 使用 kickstart 文件自动从 ISO 准备基于 Red Hat 的虚拟机


### <a id="rhel7xkickstart"></a>从 kickstart 文件准备 RHEL 7.1/7.2 虚拟机###


1.	创建包含以下内容的启动文件，然后保存该文件。有关 kickstart 安装的详细信息，请参阅 [Kickstart Installation Guide（Kickstart 安装指南](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html)）。



        # Kickstart for provisioning a RHEL 7 Azure VM

        # System authorization information
        auth --enableshadow --passalgo=sha512

        # Use graphical install
        text

        # Do not run the Setup Agent on first boot
        firstboot --disable

        # Keyboard layouts
        keyboard --vckeymap=us --xlayouts='us'

        # System language
        lang en_US.UTF-8

        # Network information
        network  --bootproto=dhcp

        # Root password
        rootpw --plaintext "to_be_disabled"

        # System services
        services --enabled="sshd,waagent,NetworkManager"

        # System timezone
        timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

        # Partition clearing information
        clearpart --all --initlabel

        # Clear the MBR
        zerombr

        # Disk partitioning information
        part /boot --fstype="xfs" --size=500
        part / --fstyp="xfs" --size=1 --grow --asprimary

        # System bootloader configuration
        bootloader --location=mbr

        # Firewall configuration
        firewall --disabled

        # Enable SELinux
        selinux --enforcing

        # Don't configure X
        skipx

        # Power down the machine after install
        poweroff



        %packages
        @base
        @console-internet
        chrony
        sudo
        parted
        -dracut-config-rescue

        %end

        %post --log=/var/log/anaconda/post-install.log

        #!/bin/bash

        # Register Red Hat Subscription
        subscription-manager register --username=XXX --password=XXX --auto-attach --force

        # Install latest repo update
        yum update -y

        # Enable extras repo
        subscription-manager repos --enable=rhel-7-server-extras-rpms

        # Install WALinuxAgent
        yum install -y WALinuxAgent

        # Unregister Red Hat subscription
        subscription-manager unregister

        # Enable waaagent at boot-up
        systemctl enable waagent

        # Disable the root account
        usermod root -p '!!'

        # Configure swap in WALinuxAgent
        sed -i 's/^(ResourceDisk\.EnableSwap)=[Nn]$/\1=y/g' /etc/waagent.conf
        sed -i 's/^(ResourceDisk\.SwapSizeMB)=[0-9]*$/\1=2048/g' /etc/waagent.conf

        # Set the cmdline
        sed -i 's/^(GRUB_CMDLINE_LINUX)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

        # Enable SSH keepalive
        sed -i 's/^#(ClientAliveInterval).*$/\1 180/g' /etc/ssh/sshd_config

        # Build the grub cfg
        grub2-mkconfig -o /boot/grub2/grub.cfg

        # Configure network
        cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=yes
        EOF

        # Deprovision and prepare for Azure
        waagent -force -deprovision

        %end

2.	将 kickstart 文件放在可从安装系统访问的位置。

3.	在 Hyper-V 管理器中，创建新的 VM。在“连接虚拟硬盘”页上，选择“稍后附加虚拟硬盘”，并完成新建虚拟机向导。

4.	打开 VM 设置：

    a.将新的虚拟硬盘附加到 VM。请务必选择“VHD 格式”和“固定大小”。

    b.将安装 ISO 附加到 DVD 光驱。

    c.将 BIOS 设置为从 CD 启动。

5.	启动 VM。当安装指南出现时，请按 **Tab** 键来配置启动选项。

6.	在启动选项的末尾输入 `inst.ks=<the location of the kickstart file>`，然后按 **Enter** 键。

7.	等待安装完成。完成后，VM 将自动关闭。Linux VHD 现已准备好上载到 Azure。

## 已知问题
当你在 Hyper-V 和 Azure 中使用 RHEL 7.1 时，会遇到多个已知问题。

### 磁盘 I/O 冻结

在 Hyper-V 和 Azure 中使用 RHEL 7.1 时，频繁的存储磁盘 I/O 活动期间可能会出现此问题。

重现率：

此问题是间歇性的。但在 Hyper-V 和 Azure 中进行频繁的磁盘 I/O 操作过程中出现更加频繁。


[AZURE.NOTE] 此已知问题已被 Red Hat 解决。若要安装关联的修补程序，请运行以下命令：

    # sudo yum update

### 使用非 Hyper-V 虚拟机监控程序时，初始 RAM 磁盘未包含 Hyper-V 驱动程序

在某些情况下，Linux 安装程序可能无法在初始 RAM 磁盘（initrd 或 initramfs）中包含 Hyper-V 驱动程序，除非它检测到它正在运行 Hyper-V 环境。

使用不同虚拟化系统（即 Virtualbox、Xen 等）来准备 Linux 映像时，可能需要重新生成 initrd 以确保至少 hv\_vmbus 和 hv\_storvsc 内核模块可在初始 RAM 磁盘上使用。至少在基于上游 Red Hat 分发的系统上这是一个已知问题。

若要解决此问题，需要将 Hyper-V 模块添加到 initramfs 并将它重新生成：

编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

重新生成 initramfs：

        # dracut -f -v

有关详细信息，请参阅有关[重新生成 initramfs](https://access.redhat.com/solutions/1958) 的信息。

## 后续步骤
现在，你可以使用 Red Hat Enterprise Linux 虚拟硬盘在 Azure 中创建新的 Azure 虚拟机。如果这是你第一次将 .vhd 文件上载到 Azure，请参阅 [Creating and uploading a virtual hard disk that contains the Linux operating system（创建并上载包含 Linux 操作系统的虚拟硬盘）](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的步骤 2 和步骤 3。

有关已通过认证可运行 Red Hat Enterprise Linux 的虚拟机监控程序的更多详细信息，请参阅 [Red Hat 网站](https://access.redhat.com/certified-hypervisors)。

<!---HONumber=Mooncake_0503_2016-->