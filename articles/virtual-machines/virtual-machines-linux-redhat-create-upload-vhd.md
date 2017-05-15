<properties
    pageTitle="创建并上传可在 Azure 中使用的 Red Hat Enterprise Linux VHD | Azure"
    description="了解如何创建和上传包含 Red Hat Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="szarkos"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management" />
<tags
    ms.assetid="6c6b8f72-32d3-47fa-be94-6cb54537c69f"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/10/2017"
    wacn.date="05/15/2017"
    ms.author="szark"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="b0ea25ec3167dd65a24244ca6fc8a5f484b91027"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="prepare-a-red-hat-based-virtual-machine-for-azure"></a>为 Azure 准备基于 Red Hat 的虚拟机
在本文中，你将了解如何准备 Red Hat Enterprise Linux (RHEL) 虚拟机，以供在 Azure 中使用。 本文介绍的 RHEL 版本为 6.7+ 和 7.1+。 本文所述的用于准备工作的虚拟机监控程序为 Hyper-V、基于内核的虚拟机 (KVM) 和 VMware。 有关参与 Red Hat 云访问计划的资格要求的详细信息，请参阅 [Red Hat 的云访问网站](http://www.redhat.com/en/technologies/cloud-computing/cloud-access)和[在 Azure 上运行 RHEL](https://access.redhat.com/articles/1989673)。

## <a name="prepare-a-red-hat-based-virtual-machine-from-hyper-v-manager"></a>从 Hyper-V 管理器准备基于 Red Hat 的虚拟机

### <a name="prerequisites"></a>先决条件
本部分假定你已经从 Red Hat 网站获取 ISO 文件并将 RHEL 映像安装到虚拟硬盘 (VHD)。 有关如何使用 Hyper-V 管理器来安装操作系统映像的更多详细信息，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

**RHEL 安装说明**

* Azure 不支持 VHDX 格式。 Azure 仅支持固定 VHD。 可使用 Hyper-V 管理器将磁盘转换为 VHD 格式，也可以使用 convert-vhd cmdlet。 如果使用 VirtualBox，则选择“固定大小”，而不是在创建磁盘时默认动态分配选项。
* Azure 仅支持第 1 代虚拟机。 可以将第 1 代虚拟机从 VHDX 转换为 VHD 文件格式，从动态扩展磁盘转换为固定大小磁盘。 但无法更改虚拟机的代次。 有关详细信息，请参阅[是否应在 Hyper-V 中创建第 1 代或第 2 代虚拟机？](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)。
* VHD 允许的最大大小为 1,023 GB。
* 在安装 Linux 操作系统时，建议使用标准分区而不是逻辑卷管理器 (LVM)（通常是许多安装的默认设置）。 这种做法可以避免 LVM 名称与克隆的虚拟机冲突，尤其是当你需要将操作系统磁盘附加到另一台相同的虚拟机进行故障排除时。 [LVM](/documentation/articles/virtual-machines-linux-configure-lvm/) 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid/) 可以在数据磁盘上使用。
* 需要装载通用磁盘格式 (UDF) 文件系统的内核支持。 在 Azure 上首次启动时，附加到来宾的 UDF 格式媒体会将预配配置传递到 Linux 虚拟机。 Azure Linux 代理必须能够装载 UDF 文件系统才能读取其配置和预配虚拟机。
* 早于 2.6.37 的 Linux 内核版本不支持虚拟机比较大的 Hyper-V 上的非一致性内存访问 (NUMA)。 此问题主要影响使用上游 Red Hat 2.6.32 内核的旧分发版，在 RHEL 6.6 (kernel-2.6.32-504) 中已得到解决。 运行版本低于 2.6.37 的自定义内核的系统或者版本低于 2.6.32-504 的基于 RHEL 的内核必须在 grub.conf 中的内核命令行上设置启动参数 `numa=off`。 有关详细信息，请参阅 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。
* 不要在操作系统磁盘上配置交换分区。 可以配置 Linux 代理，然后在临时资源磁盘上创建交换文件。  可以在以下步骤中找到有关此内容的详细信息。
* 所有 VHD 的大小必须是 1 MB 的倍数。

### <a name="prepare-a-rhel-6-virtual-machine-from-hyper-v-manager"></a>从 Hyper-V 管理器准备 RHEL 6 虚拟机

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击“连接”打开该虚拟机的控制台窗口。

3. 在 RHEL 6 中，NetworkManager 可能会干扰 Azure Linux 代理。 运行以下命令卸载此包：

        # sudo rpm -e --nodeps NetworkManager

4. 创建或编辑 `/etc/sysconfig/network` 文件并添加以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. 移动（或删除）udev 规则，以避免产生以太网接口的静态规则。 在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. 通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

8. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

9. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

10. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 若要执行此修改，请在文本编辑器中打开 `/boot/grub/menu.lst`，并确保默认内核包含以下参数：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。

    除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。  如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多。 在遇到较小的虚拟机大小时，此配置可能会有问题。

    >[AZURE.IMPORTANT]
    RHEL 6.5 和更早版本还必须设置 `numa=off` 内核参数。 请参阅 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。

11. 请确保安全外壳 (SSH) 服务器已安装且已配置为在引导时启动（默认采用此配置）。 修改 /etc/ssh/sshd_config 以包含以下行：

        ClientAliveInterval 180

12. 通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent

        # sudo chkconfig waagent on

    如果步骤 3 中没有删除 NetworkManager 包和 NetworkManager-gnome 包，则安装 WALinuxAgent 包将删除它们。

13. 不要在操作系统磁盘上创建交换空间。

    Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，可能在取消预配虚拟机后被清空。 在上一步中安装 Azure Linux 代理后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

14. 通过运行以下命令取消注册订阅（如有必要）：

        # sudo subscription-manager unregister

15. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

16. 在 Hyper-V 管理器中单击“操作” > “关闭”。 Linux VHD 现已准备好上载到 Azure。

### <a name="prepare-a-rhel-7-virtual-machine-from-hyper-v-manager"></a>从 Hyper-V 管理器准备 RHEL 7 虚拟机

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击“连接”打开该虚拟机的控制台窗口。

3. 创建或编辑 `/etc/sysconfig/network` 文件并添加以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

4. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

5. 通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

6. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 若要执行此修改，请在文本编辑器中打开 `/etc/default/grub` 并编辑 `GRUB_CMDLINE_LINUX` 参数。 例如：

        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 此配置还会关闭 NIC 的新 RHEL 7 命名约定。 除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。 如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多，遇到较小的虚拟机大小时，此配置可能会有问题。

8. 完成 `/etc/default/grub` 编辑后，运行以下命令以重新生成 grub 配置：

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

9. 请确保 SSH 服务器已安装且已配置为在引导时启动（默认采用此配置）。 修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

10. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

11. 通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

12. 不要在操作系统磁盘上创建交换空间。

    Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，并可能在取消预配虚拟机时被清空。 在上一步中安装 Azure Linux 代理后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. 如果你想要取消注册订阅，运行以下命令：

        # sudo subscription-manager unregister

14. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

15. 在 Hyper-V 管理器中单击“操作” > “关闭”。 Linux VHD 现已准备好上载到 Azure。

## <a name="prepare-a-red-hat-based-virtual-machine-from-kvm"></a>从 KVM 准备基于 Red Hat 的虚拟机
### <a name="prepare-a-rhel-6-virtual-machine-from-kvm"></a>从 KVM 准备 RHEL 6 虚拟机

1. 从 Red Hat 网站上下载 RHEL 6 的 KVM 映像。

2. 设置 root 密码。

    生成加密密码，然后复制命令的输出：

        # openssl passwd -1 changeme

    使用 guestfish 设置 root 密码：

        # guestfish --rw -a <image-name>
        > <fs> run
        > <fs> list-filesystems
        > <fs> mount /dev/sda1 /
        > <fs> vi /etc/shadow
        > <fs> exit

    将 root 用户的第二个字段从“!!”更改 为加密密码。

3. 在 KVM 中通过 qcow2 映像创建虚拟机。 将磁盘类型设置为 **qcow2**，将虚拟网络接口设备型号设置为 **virtio**。 然后启动虚拟机，并以 root 身份登录。

4. 创建或编辑 `/etc/sysconfig/network` 文件并添加以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. 移动（或删除）udev 规则，以避免产生以太网接口的静态规则。 在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. 通过运行以下命令，确保网络服务将在引导时启动：

        # chkconfig network on

8. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # subscription-manager register --auto-attach --username=XXX --password=XXX

9. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 若要执行此配置，请在文本编辑器中打开 `/boot/grub/menu.lst`，并确保默认内核包含以下参数：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。

    除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。 如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多，遇到较小的虚拟机大小时，此配置可能会有问题。

    >[AZURE.IMPORTANT]
    RHEL 6.5 和更早版本还必须设置 `numa=off` 内核参数。 请参阅 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。

10. 将 Hyper-V 模块添加到 initramfs 中：  

    编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

11. 卸载 cloud-init：

        # yum remove cloud-init

12. 确保已安装 SSH 服务器且已将其配置为在引导时启动。

        # chkconfig sshd on

    修改 /etc/ssh/sshd_config 以包含以下行：

        PasswordAuthentication yes
        ClientAliveInterval 180

13. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

14. 通过运行以下命令来安装 Azure Linux 代理：

        # yum install WALinuxAgent

        # chkconfig waagent on

15. Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，并可能在取消预配虚拟机时被清空。 在上一步中安装 Azure Linux 代理后，相应地在 **/etc/waagent.conf** 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. 通过运行以下命令取消注册订阅（如有必要）：

        # subscription-manager unregister

17. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # waagent -force -deprovision

        # export HISTSIZE=0

        # logout

18. 关闭 KVM 中的虚拟机。

19. 将 qcow2 映像转换为 VHD 格式。

    首先将此映像转换为原始格式：

        # qemu-img convert -f qcow2 -O raw rhel-6.8.qcow2 rhel-6.8.raw

    请确保原始映像大小为 1 MB。 如果不是，请将大小四舍五入，使其等于 1 MB：

                # MB=$((1024*1024))

                # size=$(qemu-img info -f raw --output json "rhel-6.8.raw" | \

            gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

                # rounded_size=$((($size/$MB + 1)*$MB))

                # qemu-img resize rhel-6.8.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.8.raw rhel-6.8.vhd

### <a name="prepare-a-rhel-7-virtual-machine-from-kvm"></a>从 KVM 准备 RHEL 7 虚拟机

1. 从 Red Hat 网站上下载 RHEL 7 的 KVM 映像。 此过程以 RHEL 7 为例。

2. 设置 root 密码。

    生成加密密码，然后复制命令的输出：

        # openssl passwd -1 changeme

    使用 guestfish 设置 root 密码：

        # guestfish --rw -a <image-name>
        > <fs> run
        > <fs> list-filesystems
        > <fs> mount /dev/sda1 /
        > <fs> vi /etc/shadow
        > <fs> exit

    将 root 用户的第二个字段从“!!”更改 为加密密码。

3. 在 KVM 中通过 qcow2 映像创建虚拟机。 将磁盘类型设置为 **qcow2**，将虚拟网络接口设备型号设置为 **virtio**。 然后启动虚拟机，并以 root 身份登录。

4. 创建或编辑 `/etc/sysconfig/network` 文件并添加以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

6. 通过运行以下命令，确保网络服务将在引导时启动：

        # chkconfig network on

7. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # subscription-manager register --auto-attach --username=XXX --password=XXX

8. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 若要执行此配置，请在文本编辑器中打开 `/etc/default/grub` 并编辑 `GRUB_CMDLINE_LINUX` 参数。 例如：

        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"

    此命令还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 此命令还会关闭 NIC 的新 RHEL 7 命名约定。 除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。 如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多，遇到较小的虚拟机大小时，此配置可能会有问题。

9. 完成 `/etc/default/grub` 编辑后，运行以下命令以重新生成 grub 配置：

        # grub2-mkconfig -o /boot/grub2/grub.cfg

10. 将 Hyper-V 模块添加到 initramfs 中。

    编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

11. 卸载 cloud-init：

        # yum remove cloud-init

12. 确保已安装 SSH 服务器且已将其配置为在引导时启动。

        # systemctl enable sshd

    修改 /etc/ssh/sshd_config 以包含以下行：

        PasswordAuthentication yes
        ClientAliveInterval 180

13. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

14. 通过运行以下命令来安装 Azure Linux 代理：

        # yum install WALinuxAgent

    启用 waagent 服务：

        # systemctl enable waagent.service

15. 不要在操作系统磁盘上创建交换空间。

    Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，并可能在取消预配虚拟机时被清空。 在上一步中安装 Azure Linux 代理后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. 通过运行以下命令取消注册订阅（如有必要）：

        # subscription-manager unregister

17. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

18. 关闭 KVM 中的虚拟机。

19. 将 qcow2 映像转换为 VHD 格式。

    首先将此映像转换为原始格式：

        # qemu-img convert -f qcow2 -O raw rhel-7.3.qcow2 rhel-7.3.raw

    请确保原始映像大小为 1 MB。 如果不是，请将大小四舍五入，使其等于 1 MB：

                # MB=$((1024*1024))

                # size=$(qemu-img info -f raw --output json "rhel-7.3.raw" | \

            gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

                # rounded_size=$((($size/$MB + 1)*$MB))

                # qemu-img resize rhel-7.3.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.3.raw rhel-7.3.vhd

## <a name="prepare-a-red-hat-based-virtual-machine-from-vmware"></a>从 VMware 准备基于 Red Hat 的虚拟机
### <a name="prerequisites"></a>先决条件
本部分假设你已在 VMware 中安装了 RHEL 虚拟机。 有关如何在 VMware 中安装操作系统的详细信息，请参阅 [VMware 来宾操作系统安装指南](http://partnerweb.vmware.com/GOSIG/home.html)。

* 在安装 Linux 操作系统时，建议使用标准分区而不是 LVM，这通常是许多安装的默认设置。 这种做法可以避免 LVM 名称与克隆的虚拟机名称冲突，尤其是在需要将操作系统磁盘附加到另一台虚拟机进行故障排除时。 如果需要，可以在数据磁盘上使用 LVM 或 RAID。
* 不要在操作系统磁盘上配置交换分区。 可将 Linux 代理配置为在临时资源磁盘上创建交换文件。 可以在下面的步骤中找到有关此操作的详细信息。
* 创建虚拟硬盘时，选择“将虚拟磁盘存储为单个文件”。

### <a name="prepare-a-rhel-6-virtual-machine-from-vmware"></a>从 VMware 准备 RHEL 6 虚拟机
1. 在 RHEL 6 中，NetworkManager 可能会干扰 Azure Linux 代理。 运行以下命令卸载此包：

        # sudo rpm -e --nodeps NetworkManager

2. 在包含以下文本的 /etc/sysconfig/ 目录中创建一个名为 **network** 的文件：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

3. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

4. 移动（或删除）udev 规则，以避免产生以太网接口的静态规则。 在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则会引发问题：

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

5. 通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

6. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

8. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 为此，请在文本编辑器中打开 `/etc/default/grub` 并编辑 `GRUB_CMDLINE_LINUX` 参数。 例如：

        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"

    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 此配置还会关闭 NIC 的新 RHEL 7 命名约定。 除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。 如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多，遇到较小的虚拟机大小时，此配置可能会有问题。

9. 将 Hyper-V 模块添加到 initramfs 中：

    编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

10. 请确保 SSH 服务器已安装且已配置为在引导时启动（默认采用此配置）。 修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

11. 通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent

        # sudo chkconfig waagent on

12. 不要在操作系统磁盘上创建交换空间。

    Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，并可能在取消预配虚拟机时被清空。 在上一步中安装 Azure Linux 代理后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. 通过运行以下命令取消注册订阅（如有必要）：

        # sudo subscription-manager unregister

14. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

15. 关闭虚拟机，并将 VMDK 文件转换为 .vhd 文件。

    首先将此映像转换为原始格式：

        # qemu-img convert -f vmdk -O raw rhel-6.8.vmdk rhel-6.8.raw

    请确保原始映像大小为 1 MB。 如果不是，请将大小四舍五入，使其等于 1 MB：

                # MB=$((1024*1024))

                # size=$(qemu-img info -f raw --output json "rhel-6.8.raw" | \

            gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

                # rounded_size=$((($size/$MB + 1)*$MB))

                # qemu-img resize rhel-6.8.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.8.raw rhel-6.8.vhd

### <a name="prepare-a-rhel-7-virtual-machine-from-vmware"></a>从 VMware 准备 RHEL 7 虚拟机
1. 创建或编辑 `/etc/sysconfig/network` 文件并添加以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

2. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件并添加以下文本：

        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

3. 通过运行以下命令，确保网络服务将在引导时启动：

        # sudo chkconfig network on

4. 注册你的 Red Hat 订阅，以通过运行以下命令来启用来自 RHEL 存储库中的包的安装：

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

5. 在 grub 配置中修改内核引导行，以使其包含 Azure 的其他内核参数。 若要执行此修改，请在文本编辑器中打开 `/etc/default/grub` 并编辑 `GRUB_CMDLINE_LINUX` 参数。 例如：

        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"

    此配置还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。 此外，还会关闭 NIC 的新 RHEL 7 命名约定。 除此之外，建议删除以下参数：

        rhgb quiet crashkernel=auto

    图形引导和无人参与引导不适用于云环境，在该环境中我们想要将所有日志都发送到串行端口。 如果需要，可以保留配置的 `crashkernel` 选项。 请注意，此参数可以将虚拟机中的可用内存量减少 128 MB 或更多，遇到较小的虚拟机大小时，此配置可能会有问题。

6. 完成 `/etc/default/grub` 编辑后，运行以下命令以重新生成 grub 配置：

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

7. 将 Hyper-V 模块添加到 initramfs 中。

    编辑 `/etc/dracut.conf`，添加内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    重新生成 initramfs：

        # dracut -f -v

8. 请确保已安装 SSH 服务器且已将其配置为在引导时启动。 此设置通常是默认设置。 修改 `/etc/ssh/sshd_config` 以包含以下行：

        ClientAliveInterval 180

9. WALinuxAgent 包 `WALinuxAgent-<version>` 已推送到 Red Hat extras 存储库。 通过运行以下命令启用 extras 存储库：

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

10. 通过运行以下命令来安装 Azure Linux 代理：

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

11. 不要在操作系统磁盘上创建交换空间。

    Azure Linux 代理可使用在 Azure 上预配虚拟机后附加到虚拟机的本地资源磁盘自动配置交换空间。 请注意，本地资源磁盘是临时磁盘，并可能在取消预配虚拟机时被清空。 在上一步中安装 Azure Linux 代理后，相应地在 `/etc/waagent.conf` 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

12. 如果你想要取消注册订阅，运行以下命令：

        # sudo subscription-manager unregister

13. 运行以下命令可取消对虚拟机的预配并且对其进行准备以便在 Azure 上进行预配：

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

14. 关闭虚拟机，将 VMDK 文件转换为 VHD 格式。

    首先将此映像转换为原始格式：

        # qemu-img convert -f vmdk -O raw rhel-7.3.vmdk rhel-7.3.raw

    请确保原始映像大小为 1 MB。 如果不是，请将大小四舍五入，使其等于 1 MB：

                # MB=$((1024*1024))

                # size=$(qemu-img info -f raw --output json "rhel-7.3.raw" | \

            gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

                # rounded_size=$((($size/$MB + 1)*$MB))

                # qemu-img resize rhel-7.3.raw $rounded_size

    将原始磁盘转换为固定大小的 VHD：

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.3.raw rhel-7.3.vhd

## <a name="prepare-a-red-hat-based-virtual-machine-from-an-iso-by-using-a-kickstart-file-automatically"></a>使用 kickstart 文件自动从 ISO 准备基于 Red Hat 的虚拟机
### <a name="prepare-a-rhel-7-virtual-machine-from-a-kickstart-file"></a>从 kickstart 文件准备 RHEL 7 虚拟机

1.  创建包括以下内容的 kickstart 文件，并保存该文件。 有关 kickstart 安装的详细信息，请参阅 [Kickstart 安装指南](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html)。

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
        sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
        sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

        # Set the cmdline
        sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

        # Enable SSH keepalive
        sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

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
        NM_CONTROLLED=no
        EOF

        # Deprovision and prepare for Azure
        waagent -force -deprovision

        %end

2. 将 kickstart 文件放在安装系统可以访问的位置。

3. 在 Hyper-V 管理器中，创建新的虚拟机。 在“连接虚拟硬盘”页上，选择“稍后附加虚拟硬盘”，并完成新建虚拟机向导。

4. 打开虚拟机设置：

    a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。  将新的虚拟硬盘附加到虚拟机。 请务必选择“VHD 格式”和“固定大小”。

    b.保留“数据库类型”设置，即设置为“共享”。  将安装 ISO 附加到 DVD 光驱。

    c.  将 BIOS 设置为从 CD 启动。

5. 启动虚拟机。 当安装指南出现时，请按 **Tab** 键来配置启动选项。

6. 在启动选项的末尾输入 `inst.ks=<the location of the kickstart file>` ，然后按 **Enter**键。

7. 等待安装完成。 完成后，虚拟机将自动关闭。 Linux VHD 现已准备好上载到 Azure。

## <a name="known-issues"></a>已知问题
### <a name="the-hyper-v-driver-could-not-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>使用非 Hyper-V 虚拟机监控程序时，初始 RAM 磁盘未包含 Hyper-V 驱动程序

在某些情况下，Linux 安装程序可能无法在初始 RAM 磁盘（initrd 或 initramfs）中包含 Hyper-V 驱动程序，除非 Linux 检测到它正在 Hyper-V 环境中运行。

使用不同虚拟化系统（即 Virtualbox、Xen 等）来准备 Linux 映像时，可能需要重新生成 initrd 以确保至少 hv_vmbus 和 hv_storvsc 内核模块可在初始 RAM 磁盘上使用。 至少在基于上游 Red Hat 分发的系统上这是一个已知问题。

若要解决此问题，请将 Hyper-V 模块添加到 initramfs 并进行重新生成：

编辑 `/etc/dracut.conf` 并添加以下内容：

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

重新生成 initramfs：

        # dracut -f -v

有关详细信息，请参阅有关 [重新生成 initramfs](https://access.redhat.com/solutions/1958)的信息。

## <a name="next-steps"></a>后续步骤
现在，你可以使用 Red Hat Enterprise Linux 虚拟硬盘在 Azure 中创建新的虚拟机。 如果这是第一次将 .vhd 文件上传到 Azure，请参阅[创建和上传包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的步骤 2 和步骤 3。

有关已通过认证可运行 Red Hat Enterprise Linux 的虚拟机监控程序的更多详细信息，请参阅 [Red Hat 网站](https://access.redhat.com/certified-hypervisors)。