<properties
    pageTitle="在 Azure 中创建和上载基于 CentOS 的 Linux VHD"
    description="了解如何创建和上载包含基于 CentOS 的 Linux 操作系统的 Azure 虚拟硬盘 (VHD)。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="szarkos"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management" />
<tags
    ms.assetid="0e518e92-e981-43f4-b12c-9cba1064c4bb"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/02/2017"
    wacn.date="03/20/2017"
    ms.author="szark" />  


# 为 Azure 准备基于 CentOS 的虚拟机
* [为 Azure 准备 CentOS 6.x 虚拟机](#centos-6x)
* [为 Azure 准备 CentOS 7.0+ 虚拟机](#centos-70)

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## 先决条件
本文假设已在虚拟硬盘安装 CentOS（或类似衍生产品）Linux 操作系统。可使用多种工具创建 .vhd 文件，如 Hyper-V 等虚拟化解决方案。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

**CentOS 安装说明**

* 另请参阅[常规 Linux 安装说明](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)，了解更多如何为 Azure 准备 Linux 的提示。
* Azure 不支持 VHDX 格式，仅支持**固定大小的 VHD**。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。如果你使用 VirtualBox，则意味着选择的是“固定大小”，而不是在创建磁盘时动态分配默认大小。
* 安装 Linux 系统时，*建议*使用标准分区而非 LVM（通常是许多安装的默认值）。这可以避免与克隆 VM 发生 LVM 名称冲突，尤其是在需要将 OS 磁盘连接到另一个同类 VM 进行故障排除时。可在数据磁盘上使用 [LVM](/documentation/articles/virtual-machines-linux-configure-lvm/) 或 [RAID](/documentation/articles/virtual-machines-linux-configure-raid/)。
* 需要装载 UDF 文件系统的内核支持。在 Azure 上首次启动时，预配配置将通过附加到来宾的 UDF 格式媒体传递到 Linux VM。Azure Linux 代理必须能够装载 UDF 文件系统才能读取其配置和预配 VM。
* 低于 2.6.37 的 Linux 内核版本不支持具有更大 VM 大小的 Hyper-V 上的 NUMA。此问题主要影响使用上游 Red Hat 2.6.32 内核的旧分发版，在 RHEL 6.6 (kernel-2.6.32-504) 中已解决。运行版本低于 2.6.37 的自定义内核的系统，或者版本低于 2.6.32-504 的基于 RHEL 的内核必须在 grub.conf 中的内核命令行上设置启动参数 `numa=off`。有关详细信息，请参阅 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。
* 不要在操作系统磁盘上配置交换分区。可以配置 Linux 代理，以便在临时资源磁盘上创建交换文件。有关此操作的详细信息，请参见以下步骤。
* 所有 VHD 的大小必须是 1 MB 的倍数。

## <a id="centos-6x"></a> CentOS 6.x

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击“连接”以打开该虚拟机的控制台窗口。

3. 在 CentOS 6 中，NetworkManager 可能会干扰 Azure Linux 代理。请运行以下命令来卸载该包：
   
        # sudo rpm -e --nodeps NetworkManager

4. 创建或编辑 `/etc/sysconfig/network` 文件，添加以下文本：
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件，添加以下文本：
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. 修改 udev 规则，以避免产生针对以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则可能会引发问题：
   
        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. 通过运行以下命令，确保网络服务将在引导时启动：
   
        # sudo chkconfig network on

8. 如果要使用 Azure 数据中心托管的 OpenLogic 镜像，请使用以下存储库替换 `/etc/yum.repos.d/CentOS-Base.repo` 文件。这还会添加 **[openlogic]** 存储库，其中包含 Azure Linux 代理等其他包：

        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
    	
        [base]
        name=CentOS-$releasever - Base
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    	
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    	
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
    	
        #contrib - packages by Centos Users
        [contrib]
        name=CentOS-$releasever - Contrib
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

    >[AZURE.NOTE]
    本指南其余部分假设用户至少使用 `[openlogic]` 存储库，用于安装以下 Azure Linux 代理。

9. 将以下行添加到 /etc/yum.conf：
    
        http_caching=packages

10. 运行以下命令，清除当前的 yum 元数据并使用最新的包更新系统：
    
        # yum clean all

    建议将所有包都更新到最新版本，除非要为旧版 CentOS 创建映像：

        # sudo yum -y update

    运行此命令后，可能需要重新启动。

11. （可选）安装适用于 Linux Integration Services (LIS) 的驱动程序。
   
    >[AZURE.IMPORTANT]
    此步骤对于 CentOS 6.3 及更低版本来说是**必需**步骤，对于更高版本来说是可选步骤。

        # sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
        # sudo yum install microsoft-hyper-v

    也可按照 [LIS 下载页](https://go.microsoft.com/fwlink/?linkid=403033)上的手动安装说明进行操作，将 RPM 安装到 VM。
 
12. 安装 Azure Linux 代理和依赖项：
    
        # sudo yum install python-pyasn1 WALinuxAgent
    
    如果没有如步骤 3 中所述删除 NetworkManager 包和 NetworkManager-gnome 包，则 WALinuxAgent 包会将其删除。

13. 在 grub 配置中修改内核引导行，使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/boot/grub/menu.lst`，并确保默认内核包含以下参数：
    
        console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    
    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。
    
    除此之外，建议*删除*以下参数：
    
        rhgb quiet crashkernel=auto
    
    图形和静默引导不适用于要将所有日志发送到串行端口的云环境。根据需要可以配置 `crashkernel` 选项，但请注意此参数会使 VM 中的可用内存量减少 128 MB 或更多，这在较小的 VM 上可能会出现问题。

    >[AZURE.IMPORTANT]
    CentOS 6.5 及更低版本还必须设置内核参数 `numa=off`。请参阅 Red Hat [KB 436883](https://access.redhat.com/solutions/436883)。

14. 请确保已安装 SSH 服务器且已将其配置为在引导时启动。这通常是默认设置。

15. 不要在 OS 磁盘上创建交换空间。
    
    Azure Linux 代理可以使用在 Azure 上预配后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是*临时*磁盘，并可能在取消预配虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 `/etc/waagent.conf` 中修改以下参数：
    
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. 运行以下命令可取消预配虚拟机，并做好准备以便在 Azure 上进行预配：
    
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

17. 在 Hyper-V 管理器中单击“操作”->“关闭”。Linux VHD 现已准备好上载到 Azure。

- - -
## <a id="centos-70"></a> CentOS 7.0+
**CentOS 7（和类似衍生产品）中的更改**

为 Azure 准备 CentOS 7 虚拟机与 CentOS 6 非常类似，但有几个值得注意的重要区别：

* NetworkManager 包不再与 Azure Linux 代理冲突。默认会安装此包，建议不要删除。
* GRUB2 现在用作默认引导加载程序，因此用于编辑内核参数的过程已更改（请参见下文）。
* XFS 现在是默认文件系统。如果需要，仍可以使用 ext4 文件系统。

**配置步骤**

1. 在 Hyper-V 管理器中，选择虚拟机。

2. 单击“连接”以打开该虚拟机的控制台窗口。

3. 创建或编辑 `/etc/sysconfig/network` 文件，添加以下文本：
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

4. 创建或编辑 `/etc/sysconfig/network-scripts/ifcfg-eth0` 文件，添加以下文本：
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

5. 修改 udev 规则，以避免产生针对以太网接口的静态规则。在 Azure 或 Hyper-V 中克隆虚拟机时，这些规则可能会引发问题：
   
        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

6. 如果要使用 Azure 数据中心托管的 OpenLogic 镜像，请使用以下存储库替换 `/etc/yum.repos.d/CentOS-Base.repo` 文件。这还会添加包含 Azure Linux 代理包的 **[openlogic]** 存储库：
   
        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0
    	
        [base]
        name=CentOS-$releasever - Base
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    	
        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    	
        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    	
        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

    >[AZURE.NOTE]
    本指南其余部分假设用户至少使用 `[openlogic]` 存储库，用于安装以下 Azure Linux 代理。

7. 运行以下命令以清除当前 yum 元数据并安装所有更新：
   
        # sudo yum clean all

    建议将所有包都更新到最新版本，除非要为旧版 CentOS 创建映像：

        # sudo yum -y update

    运行此命令后，可能需要重新启动。

8. 在 grub 配置中修改内核引导行，使其包含 Azure 的其他内核参数。为此，请在文本编辑器中打开 `/etc/default/grub` 并编辑 `GRUB_CMDLINE_LINUX` 参数，例如：
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
    这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题。此外还会关闭新的针对 NIC 的 CentOS 7 命名约定。除此之外，建议*删除*以下参数：
   
        rhgb quiet crashkernel=auto
   
    图形和静默引导不适用于要将所有日志发送到串行端口的云环境。根据需要可以配置 `crashkernel` 选项，但请注意此参数会使 VM 中的可用内存量减少 128 MB 或更多，这在较小的 VM 上可能会出现问题。

9. 按照上面所示完成编辑 `/etc/default/grub` 后，运行以下命令以重新生成 grub 配置：
   
        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

10. 如果生成的映像源自 **VMWare、VirtualBox 或 KVM：**请确保 initramfs 中包括 Hyper-V 驱动程序：
   
    编辑 `/etc/dracut.conf`，添加内容：
   
        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"
   
    重新生成 initramfs：
   
        # sudo dracut -f -v

11. 安装 Azure Linux 代理和依赖项：

        # sudo yum install python-pyasn1 WALinuxAgent
        # sudo systemctl enable waagent

12. 不要在 OS 磁盘上创建交换空间。
   
    Azure Linux 代理可以使用在 Azure 上预配后附加到虚拟机的本地资源磁盘自动配置交换空间。请注意，本地资源磁盘是*临时*磁盘，并可能在取消预配虚拟机时被清空。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 `/etc/waagent.conf` 中修改以下参数：
   
        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. 运行以下命令可取消预配虚拟机，并做好准备以便在 Azure 上进行预配：
   
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

14. 在 Hyper-V 管理器中单击“操作”->“关闭”。Linux VHD 现已准备好上载到 Azure。

## 后续步骤
现在，可以使用 CentOS Linux 虚拟硬盘在 Azure 中创建新的 Azure 虚拟机了。如果这是第一次将 .vhd 文件上载到 Azure，请参阅[创建和上载包含 Linux 操作系统的虚拟硬盘](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/)中的步骤 2 和步骤 3。

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->