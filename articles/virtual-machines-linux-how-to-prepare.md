<properties linkid="manage-linux-howto-linux-agent" urlDisplayName="Prepare a distribution" pageTitle="Prepare a distribution of Linux in Azure" metaKeywords="Azure Git CodePlex, Azure website CodePlex, Azure website Git" description="Learn how to use Git to publish to an Azure web site, as well as enable continuous deployment from GitHub and CodePlex." metaCanonical="" services="virtual-machines" documentationCenter="" title="Prepare a Linux Virtual Machine for Azure" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 为 Azure 准备 Linux 虚拟机

Azure 中的虚拟机运行你在创建虚拟机时选择的操作系统。Azure 采用 VHD 格式（.vhd 文件）将虚拟机的操作系统存储在虚拟硬盘中。已为复制准备的操作系统的 VHD 称作映像。本文说明如何利用已安装且经过一般化的操作系统上载 .vhd 文件来创建你自己的映像。有关 Azure 中的磁盘和映像的详细信息，请参阅[管理磁盘和映像（可能为英文页面）][管理磁盘和映像（可能为英文页面）]。

**注意**：创建虚拟机时，你可以自定义操作系统设置以快速运行你的应用程序。你设置的配置存储在该虚拟机的磁盘上。有关说明，请参阅[如何创建自定义虚拟机][如何创建自定义虚拟机]。

**重要说明**：只有在使用某个认可的分发的时候也使用[本文][本文]中指定的配置详细信息时，Azure 平台 SLA 才适用于运行 Linux 操作系统的虚拟机。在 Azure 平台映像库中提供的所有 Linux 分发都是具有所需配置的认可的分发。

## 先决条件

本文假定你拥有以下项目：

-   **管理证书** - 你已为要为其上载 VHD 的订阅创建一个管理证书，并且已将该证书导出到 .cer 文件。有关创建证书的详细信息，请参阅[为 Azure 创建管理证书（可能为英文页面）][为 Azure 创建管理证书（可能为英文页面）]。

-   **安装在 .vhd 文件中的 Linux 操作系统。** - 你已将支持的 Linux 操作系统安装到某一虚拟硬盘。可使用多个工具创建 .vhd 文件。可使用虚拟化解决方案（例如 Hyper-V）创建 .vhd 文件并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机][安装 Hyper-V 角色和配置虚拟机]。

    **重要说明**：Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。

    有关认可分发的列表，请参阅 [Azure 认可的分发中的 Linux][Azure 认可的分发中的 Linux]。或者，请参阅本文最后一节中的[非认可分发的信息][非认可分发的信息]。

-   **Linux Azure 命令行工具。**如果你正在使用 Linux 操作系统创建映像，请使用此工具上载 VHD 文件。若要下载此工具，请参阅[针对 Mac 和 Linux 的 Azure 命令行工具][针对 Mac 和 Linux 的 Azure 命令行工具]。

-   **Add-AzureVhd cmdlet**，它是 Azure PowerShell 模块的一部分。若要下载该模块，请参阅 [Azure 下载（可能为英文页面）][Azure 下载（可能为英文页面）]。有关详细信息，请参阅 [Add-AzureVhd（可能为英文页面）][Add-AzureVhd（可能为英文页面）]。

对于所有分发，请注意以下几点：

Azure Linux 代理 (Waagent) 与 NetworkManager 不兼容。网络配置应使用 ifcfg-eth0 文件并且可通过 ifup/ifdown 脚本进行控制。如果检测到 NetworkManager 包，则 Waagent 将拒绝安装。

NUMA 不受支持，因为 2.6.37 版之前的 Linux 内核版本有 Bug。Waagent 的安装将自动在 Linux 内核命令行的 GRUB 配置中禁用 NUMA。

Azure Linux 代理需要安装 python-pyasn1 包。

建议你在安装时不要创建 SWAP 分区。可以通过使用 Azure Linux 代理配置 SWAP 空间。此外，建议不要将主流 Linux 内核用于不带 [Microsoft 网站（可能为英文页面）][Microsoft 网站（可能为英文页面）]上提供的修补程序（许多当前分发/内核可能已包含此修补程序）的 Azure 虚拟机。

所有 VHD 的大小必须是 1 MB 的倍数。

此任务包括下列步骤：

-   [步骤 1：准备要上载的映像][步骤 1：准备要上载的映像]
-   [步骤 2：在 Azure 中创建存储帐户][步骤 2：在 Azure 中创建存储帐户]
-   [步骤 3：准备连接到 Azure][步骤 3：准备连接到 Azure]
-   [步骤 4：向 Azure 上载映像][步骤 4：向 Azure 上载映像]

## <span id="prepimage"></span> </a>步骤 1：准备要上载的映像

### 准备 CentOS 6.2+ 操作系统

你必须在操作系统中完成特定的配置步骤才能使虚拟机在 Azure 中运行。

1.  在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2.  单击**“连接”**以打开虚拟机窗口。

3.  通过运行以下命令卸载 NetworkManager：

        rpm -e --nodeps NetworkManager

    **注意：**如果未安装此包，则该命令将失败，并显示一条错误消息。这是正常情况。

4.  在`/etc/sysconfig/` 目录中创建一个名为“network”的文件，该文件包含以下文本：

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5.  在`/etc/sysconfig/network-scripts/` 目录中创建一个名为“ifcfg-eth0”的文件，该文件包含以下文本：

        DEVICE=eth0
        ONBOOT=yes
        DHCP=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6.  通过运行以下命令启用网络服务：

        chkconfig network on

7.  CentOS 6.2 或 6.3：安装适用于 Linux Integration Services 的驱动程序。

    **注意：**该步骤仅适用于 CentOS 6.2 和 6.3。在 CentOS 6.4+ 中，Linux Integration Services 已在内核中提供。

    a) 从[下载中心][下载中心]获取包含适用于 Linux Integration Services 的驱动程序的 .iso 文件。

    b) 在 Hyper-V 管理器中的“操作”窗格中，单击“设置”。

    ![打开 Hyper-V 设置][打开 Hyper-V 设置]

    c) 在“硬件”窗格中，单击“IDE 控制器 1”。

    ![添加 DVD 驱动器与安装介质][添加 DVD 驱动器与安装介质]

    d) 在“IDE 控制器”框中，单击“DVD 驱动器”，然后单击“添加”。

    e) 选择“映像文件”，浏览到“Linux IC v3.2.iso”，然后单击“打开”。

    f) 在“设置”页中，单击“确定”。

    g) 单击“连接”以打开该虚拟机窗口。

    h) 在命令提示符窗口中键入以下命令：

        mount /dev/cdrom /media
        /media/install.sh
        reboot

8.  通过运行以下命令安装 python-pyasn1：

        sudo yum install python-pyasn1

9.  将其 /etc/yum.repos.d/CentOS-Base.repo 文件替换为以下文本

        [openlogic]
        name=CentOS-$releasever - openlogic packages for $basearch
        baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
        enabled=1
        gpgcheck=0

        [base]
        name=CentOS-$releasever - Base
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #released updates
        [updates]
        name=CentOS-$releasever - Updates
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #additional packages that may be useful
        [extras]
        name=CentOS-$releasever - Extras
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #additional packages that extend functionality of existing packages
        [centosplus]
        name=CentOS-$releasever - Plus
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

        #contrib - packages by Centos Users
        [contrib]
        name=CentOS-$releasever - Contrib
        baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
        gpgcheck=1
        enabled=0
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

10. 将下列行添加到 /etc/yum.conf

        http_caching=packages
        exclude=kernel*

11. 通过编辑文件“/etc/yum/pluginconf.d/fastestmirror.conf”禁用 yum 模块“fastestmirror”，并在 [main] 下面键入以下内容

        set enabled=0

12. 运行以下命令以便清除当前 yum 元数据：

        yum clean all

13. 对于 CentOS 6.2 和 6.3，通过运行以下命令更新正在运行的虚拟机的内核

    对于 CentOS 6.2，运行以下命令：

        sudo yum remove kernel-firmware
        sudo yum --disableexcludes=all install kernel

    对于 CentOS 6.3+，键入以下内容：

        sudo yum --disableexcludes=all install kernel

14. 在 grub 中修改内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

15. 确保你内核中安装的所有 SCSI 设备都包含 300 秒或更长时间的 I/O 超时。

16. 在 /etc/sudoers 中，如果以下行存在，则注释掉以下行：

        Defaults targetpw

17. 请确保 SSH 服务器已安装并且配置为在引导时启动

18. 通过运行以下命令来安装 Azure Linux 代理

        yum install WALinuxAgent

19. 不要在操作系统磁盘上创建交换空间

    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

20. 运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        waagent -force -deprovision
        export HISTSIZE=0
        logout

21. 在 Hyper-V 管理器中单击“关闭”。

### 准备 Ubuntu 12.04+ 操作系统

1.  在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2.  单击**“连接”**以打开虚拟机窗口。

3.  替换映像中的当前存储库以使用具有你升级 VM 所需的内核和代理包的 Azure 存储库。这些步骤可能会由于 Ubuntu 版本的不同而稍有差异。

    在编辑 /etc/apt/sources.list 之前，建议你
    备份 sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

    Ubuntu 12.04：

        sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
        sudo apt-add-repository 'http://archive.canonical.com/ubuntu precise-backports main'
        sudo apt-get update

    Ubuntu 12.10：

        sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
        sudo apt-add-repository 'http://archive.canonical.com/ubuntu quantal-backports main'
        sudo apt-get update

    Ubuntu 13.04+：

        sudo sed -i "s/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g" /etc/apt/sources.list
        sudo apt-get update

4.  通过运行以下命令将操作系统更新为最新内核：

    Ubuntu 12.04：

        sudo apt-get update
        sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-precise-virtual
        (recommended) sudo apt-get dist-upgrade
        sudo reboot

    Ubuntu 12.10：

        sudo apt-get update
        sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-quantal-virtual
        (recommended) sudo apt-get dist-upgrade
        sudo reboot

    Ubuntu 13.04 和 13.10：

        sudo apt-get update
        sudo apt-get install hv-kvp-daemon-init
        (recommended) sudo apt-get dist-upgrade
        sudo reboot

5.  在系统发生崩溃后，Ubuntu 将等待 Grub 提示用户输入。若要防止此情况发生，请完成下列步骤：

    a) 打开 /etc/grub.d/00\_header 文件。

    b) 在 **make\_timeout()** 函数中，搜索 **if [“\\${recordfail}” = 1 ]; then**

    c) 将该行下的语句更改为 **set timeout=5**。

    d) 运行 update-grub。

6.  在 Grub 或 Grub2 中修改内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

7.  在 /etc/sudoers 中，如果以下行存在，则注释掉以下行：

        Defaults targetpw

8.  请确保 SSH 服务器已安装并且配置为在引导时启动

9.  通过将以下命令与 sudo 一起运行来安装代理：

        apt-get update
        apt-get install walinuxagent 

10. 不要在操作系统磁盘上创建交换空间

    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

11. 运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        waagent -force -deprovision
        export HISTSIZE=0
        logout

12. 在 Hyper-V 管理器中单击“关闭”。

### 准备 SUSE Linux Enterprise Server 11 SP2 操作系统

**注意：** [SUSE Studio][SUSE Studio] 可轻松地为 Azure 和 Hyper-V 创建和管理你的 SLES/opeSUSE 映像。此外，SUSE Studio 库中的以下正式映像可下载或克隆到你自己的 SUSE Studio 帐户中以便轻松地进行自定义：

> -   [SUSE Studio 库上的 SLES 11 SP2 for Azure][SUSE Studio 库上的 SLES 11 SP2 for Azure]
> -   [SUSE Studio 库上的 SLES 11 SP3 for Azure][SUSE Studio 库上的 SLES 11 SP3 for Azure]

1.  在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2.  单击**“连接”**以打开虚拟机窗口。

3.  添加包含最新内核和 Azure Linux 代理的存储库。运行命令`zypper lr`。例如，对于 SLES 11 SP3，输出应如下所示：

        # | Alias                        | Name               | Enabled | Refresh
        --+------------------------------+--------------------+---------+--------
        1 | susecloud:SLES11-SP1-Pool    | SLES11-SP1-Pool    | No      | Yes
        2 | susecloud:SLES11-SP1-Updates | SLES11-SP1-Updates | No      | Yes
        3 | susecloud:SLES11-SP2-Core    | SLES11-SP2-Core    | No      | Yes
        4 | susecloud:SLES11-SP2-Updates | SLES11-SP2-Updates | No      | Yes
        5 | susecloud:SLES11-SP3-Pool    | SLES11-SP3-Pool    | Yes     | Yes
        6 | susecloud:SLES11-SP3-Updates | SLES11-SP3-Updates | Yes     | Yes

    在命令返回如下的错误消息时：

        "No repositories defined. Use the 'zypper addrepo' command to add one or more repositories."

    可能需要重新启用存储库或注册系统。这可以通过 suse\_register 实用工具完成。有关更多信息，请参见 [SLES 文档（可能为英文页面）][SLES 文档（可能为英文页面）]。

如果未启用某个相关的更新存储库，请使用以下命令启用该存储库：

        zypper mr -e [REPOSITORY NUMBER]

在上面的示例中，正确的命令应为

        zypper mr -e 1 2 3 4

1.  将内核更新为可用的最新版本：

        zypper up kernel-default

2.  安装 Azure Linux 代理：

        zypper up WALinuxAgent

    你可能会看到如下消息：

        "There is an update candidate for 'WALinuxAgent', but it is from different vendor.
        Use 'zypper install WALinuxAgent-1.2-1.1.noarch' to install this candidate."

    由于包的供应商已从“Microsoft Corporation”更改为“SUSE LINUX Products GmbH, Nuernberg, Germany”，因此必须按照消息中所述显式安装此包。

    注意：WALinuxAgent 包的版本可能会稍有不同。

3.  在 Grub 中修改内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

4.  建议你将 /etc/sysconfig/network/dhcp 或等效项从 DHCLIENT\_SET\_HOSTNAME="yes" 设置为 DHCLIENT\_SET\_HOSTNAME="no"

5.  在 /etc/sudoers 中，如果以下行存在，则注释掉以下行：

        Defaults targetpw

6.  请确保 SSH 服务器已安装并且配置为在引导时启动

7.  不要在操作系统磁盘上创建交换空间

    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

8.  运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        waagent -force -deprovision
        export HISTSIZE=0
        logout

9.  在 Hyper-V 管理器中单击“关闭”。

### 准备 openSUSE 12.3 操作系统

**注意：** [SUSE Studio][SUSE Studio] 可轻松地为 Azure 和 Hyper-V 创建和管理你的 SLES/opeSUSE 映像。此外，SUSE Studio 库中的以下正式映像可下载或克隆到你自己的 SUSE Studio 帐户中以便轻松地进行自定义：

> -   [SUSE Studio 库上的 openSUSE 12.3 for Azure][SUSE Studio 库上的 openSUSE 12.3 for Azure]

1.  在 Hyper-V 管理器的中间窗格中，选择虚拟机。

2.  单击**“连接”**以打开虚拟机窗口。

3.  将操作系统更新为可用的最新内核和修补程序

4.  在外壳程序上，运行命令 '`zypper lr`'。如果此命令返回

        # | Alias                     | Name                      | Enabled | Refresh
        --+---------------------------+---------------------------+---------+--------
        1 | Cloud:Tools_openSUSE_12.3 | Cloud:Tools_openSUSE_12.3 | Yes     | Yes
        2 | openSUSE_12.3_OSS         | openSUSE_12.3_OSS         | Yes     | Yes
        3 | openSUSE_12.3_Updates     | openSUSE_12.3_Updates     | Yes     | Yes

则按预期配置存储库，不需要进行任何调整。

如果此命令返回“未定义存储库。请使用‘zypper addrepo’命令添加一个
或多个存储库。”，则需要重新启用存储库：

        zypper ar -f http://download.opensuse.org/distribution/12.3/repo/oss openSUSE_12.3_OSS
        zypper ar -f http://download.opensuse.org/update/12.3 openSUSE_12.3_Updates

确认已通过再次调用“zypper lr”添加存储库。

如果未启用某个相关的更新存储库，请使用以下命令启用该存储库：

        zypper mr -e [NUMBER OF REPOSITORY]

1.  禁用自动 DVD ROM 探测。

2.  安装 Azure Linux 代理：

    首先，添加包含新的 WALinuxAgent 的存储库：

        zypper ar -f -r http://download.opensuse.org/repositories/Cloud:/Tools/openSUSE_12.3/Cloud:Tools.repo

    然后，运行以下命令：

        zypper up WALinuxAgent

    在运行此命令后，可能会显示如下消息：

        "There is an update candidate for 'WALinuxAgent', but it is from different vendor. 
        Use 'zypper install WALinuxAgent' to install this candidate."

    该消息是预期的消息。由于包的供应商已从“Microsoft Corporation”更改为“obs://build.opensuse.org/Cloud”，因此必须按照消息中所述显式安装此包。

3.  在 Grub 中修改内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

    并且在 /boot/grub/menu.lst 中，从内核命令行中删除以下参数（如果它们存在）：

        libata.atapi_enabled=0 reserve=0x1f0,0x8

4.  建议你将 /etc/sysconfig/network/dhcp 或等效项从 DHCLIENT\_SET\_HOSTNAME="yes" 设置为 DHCLIENT\_SET\_HOSTNAME="no"

5.  在 /etc/sudoers 中，如果以下行存在，则注释掉以下行：

        Defaults targetpw

6.  请确保 SSH 服务器已安装并且配置为在引导时启动

7.  不要在操作系统磁盘上创建交换空间

    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

8.  运行以下命令可取消对虚拟机的设置并且对其进行准备以便在 Azure 上进行设置：

        waagent -force -deprovision
        export HISTSIZE=0
        logout

9.  确保在启动时运行 Azure Linux 代理：

        systemctl enable waagent.service

10. 在 Hyper-V 管理器中单击“关闭”。

## <span id="createstorage"></span> </a>步骤 2：在 Azure 中创建存储帐户

存储帐户表示用于访问存储服务的最高级别的命名空间，并且与你的 Azure 订阅相关联。你需要在 Azure 中具有存储帐户才能将 .vhd 文件上载到可用于创建虚拟机的 Azure。可使用 Azure 管理门户创建存储帐户。

1.  登录到 Azure 管理门户。

2.  在命令栏上，单击“新建”。

    ![创建存储帐户][创建存储帐户]

3.  单击“存储帐户”，然后单击“快速创建”。

    ![快速创建存储帐户][快速创建存储帐户]

4.  如下所示填写字段：

    ![输入存储帐户详细信息][输入存储帐户详细信息]

-   在 **URL** 下，键入要在存储帐户的 URL 中使用的子域名称。输入的名称可包含 3-24 个小写字母和数字。此名称将成为用于对订阅的 Blob、队列或表资源进行寻址的 URL 中的主机名。

-   选择存储帐户的位置或地缘组。通过指定地缘组，可将同一数据中心内的云服务与你的存储放置在一起。

-   决定是否使用存储帐户的地域复制。默认情况下启用地域复制。此选项会将你的数据免费复制到辅助位置，以便在发生无法在主要位置进行处理的严重故障时将你的存储故障转移到辅助位置。将自动分配辅助位置，并且无法对其进行更改。如果法律要求或组织政策要求更加严格地控制基于云的存储的位置，则你可以关闭地域复制。但是，请注意，如果稍后你打开地域复制，则将现有数据复制到辅助位置时将向你收取一次性数据传输费用。不具有地域复制的存储服务将以优惠价提供。

1.  单击**“创建存储帐户”**。

    帐户现在列在“存储帐户”下。

    ![已成功创建存储帐户][已成功创建存储帐户]

## <span id="#connect"></span> </a>步骤 3：准备连接到 Azure

你首先需要在计算机和 Azure 中的订阅之间建立一个安全连接，然后才能上载 .vhd 文件。

1.  打开 Azure PowerShell 窗口。

2.  类型：

    `Get-AzurePublishSettingsFile`

    此命令将打开浏览器窗口，并自动下载包含信息的 .publishsettings 文件和 Azure 订阅的证书。

3.  保存 .publishsettings 文件。

4.  类型：

    `Import-AzurePublishSettingsFile <PathToFile>`

    其中`<PathToFile>` 是 .publishsettings 文件的完整路径。

    有关详细信息，请参阅 [Azure Cmdlet 入门][Azure Cmdlet 入门]

## <span id="upload"></span> </a>步骤 4：向 Azure 上载映像

在上载 .vhd 文件时，你可以将 .vhd 文件放置在 Blob 存储中的任何位置。在以下命令示例中，**BlobStorageURL** 是你在步骤 2 中创建的存储帐户的 URL，**YourImagesFolder** 是要在其中存储映像的 Blob 存储中的容器。**VHDName** 是显示在管理门户中用于标识虚拟硬盘的标签。**PathToVHDFile** 是 .vhd 文件的完整路径和名称。

执行下列操作之一：

-   从你在上一步中使用的 Azure PowerShell 窗口中，键入：

    `Add-AzureVhd -Destination <BlobStorageURL>/<YourImagesFolder>/<VHDName> -LocalFilePath <PathToVHDFile>`

    有关详细信息，请参阅 [Add-AzureVhd][Add-AzureVhd（可能为英文页面）]。

-   使用 Linux 命令行工具上载映像。可使用以下命令上载映像：

        Azure vm image create <image name> --location <Location of the data center> --OS Linux <Sourcepath to the vhd>

## <span id="nonendorsed"></span> </a>有关未认可分发的信息

从本质上说，所有正在 Azure 上运行的分发都需要满足以下先决条件才能在平台上正常运行。

此列表并未涵盖所有信息，因为每个分发都是不同的；即使你满足以下所有条件，你也可能仍需显著调整你的映像以确保其在平台上正常运行。

正是出于这个原因，建议你从某个我们的[合作伙伴认可的映像][合作伙伴认可的映像]开始操作。

下面列出的内容将替换该过程的步骤 1 来创建你自己的 VHD：

1.  你将需要确保所运行的是包含最新的适用于 Hyper V 的 LIS 驱动程序的内核，或者确保已成功编译这些驱动程序（它们已进行开源）。可在[此处][此处]找到这些驱动程序

2.  内核还应包括最新版本的 ATA PiiX 驱动程序，该驱动程序用于配置映像并向内核提交修补程序，其中提交为 cd006086fa5d91414d8ff9ff2b78fbb593878e3c，日期为:Fri May 4 22:15:11 2012 +0100 ata\_piix:默认情况下，将磁盘交由 Hyper-V 驱动程序处理

3.  已压缩的 intird 应小于 40 MB（我们一直致力于增大此数字，因此它目前可能已过时）

4.  在 Grub 或 Grub2 中修改内核引导行，以便包含以下参数。这还将确保所有控制台消息都发送到第一个串行端口，从而可以协助 Azure 支持人员调试问题：

        console=ttyS0 earlyprintk=ttyS0 rootdelay=300

5.  建议你将 /etc/sysconfig/network/dhcp 或等效项从 DHCLIENT\_SET\_HOSTNAME="yes" 设置为 DHCLIENT\_SET\_HOSTNAME="no"

6.  你应确保你内核中安装的所有 SCSI 设备都包含 300 秒或更长时间的 I/O 超时。

7.  你将需要按照 [Linux 代理指南][Linux 代理指南]中的步骤操作来安装 Azure Linux 代理。已根据 Apache 2 许可发布代理，你可以在[代理 GitHub 位置][代理 GitHub 位置]获取最新的位数

8.  在 /etc/sudoers 中，如果以下行存在，则注释掉以下行：

        Defaults targetpw

9.  请确保 SSH 服务器已安装并且配置为在引导时启动

10. 不要在操作系统磁盘上创建交换空间

    Azure Linux 代理可使用在 Azure 上设置后附加到虚拟机的本地资源磁盘自动配置交换空间。在安装 Azure Linux 代理（请参见前一步骤）后，相应地在 /etc/waagent.conf 中修改以下参数：

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

11. 你将需要运行以下命令以取消设置虚拟机：

        waagent -force -deprovision
        export HISTSIZE=0
        logout

12. 然后需要关闭虚拟机并继续上载。

  [管理磁盘和映像（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
  [如何创建自定义虚拟机]: /en-us/manage/windows/how-to-guides/custom-create-a-vm/
  [本文]: http://support.microsoft.com/kb/2805216
  [为 Azure 创建管理证书（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/gg551722.aspx
  [安装 Hyper-V 角色和配置虚拟机]: http://technet.microsoft.com/zh-cn/library/hh846766.aspx
  [Azure 认可的分发中的 Linux]: ../linux-endorsed-distributions
  [非认可分发的信息]: #nonendorsed
  [针对 Mac 和 Linux 的 Azure 命令行工具]: http://go.microsoft.com/fwlink/?LinkID=253691&clcid=0x409
  [Azure 下载（可能为英文页面）]: /en-us/develop/downloads/
  [Add-AzureVhd（可能为英文页面）]: http://msdn.microsoft.com/zh-cn/library/azure/dn205185.aspx
  [Microsoft 网站（可能为英文页面）]: http://go.microsoft.com/fwlink/?LinkID=253692&clcid=0x409
  [步骤 1：准备要上载的映像]: #prepimage
  [步骤 2：在 Azure 中创建存储帐户]: #createstorage
  [步骤 3：准备连接到 Azure]: #connect
  [步骤 4：向 Azure 上载映像]: #upload
  [下载中心]: http://www.microsoft.com/zh-cn/download/details.aspx?id=34603
  [打开 Hyper-V 设置]: ./media/virtual-machines-linux-how-to-prepare/settings.png
  [添加 DVD 驱动器与安装介质]: ./media/virtual-machines-linux-how-to-prepare/installiso.png
  [SUSE Studio]: http://www.susestudio.com
  [SUSE Studio 库上的 SLES 11 SP2 for Azure]: http://susestudio.com/a/02kbT4/sles-11-sp2-for-windows-azure
  [SUSE Studio 库上的 SLES 11 SP3 for Azure]: http://susestudio.com/a/02kbT4/sles-11-sp3-for-windows-azure
  [SLES 文档（可能为英文页面）]: https://www.suse.com/documentation/sles11/
  [SUSE Studio 库上的 openSUSE 12.3 for Azure]: http://susestudio.com/a/02kbT4/opensuse-12-3-for-windows-azure
  [创建存储帐户]: ./media/virtual-machines-linux-how-to-prepare/create.png
  [快速创建存储帐户]: ./media/virtual-machines-linux-how-to-prepare/storage-quick-create.png
  [输入存储帐户详细信息]: ./media/virtual-machines-linux-how-to-prepare/storage-create-account.png
  [已成功创建存储帐户]: ./media/virtual-machines-linux-how-to-prepare/Storagenewaccount.png
  [Azure Cmdlet 入门]: http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx
  [合作伙伴认可的映像]: http://azure.microsoft.com/zh-cn/documentation/articles/linux-endorsed-distributions/
  [此处]: http://go.microsoft.com/fwlink/p/?LinkID=254263&clcid=0x409
  [Linux 代理指南]: http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-linux-agent-user-guide/
  [代理 GitHub 位置]: http://go.microsoft.com/fwlink/p/?LinkID=250998&clcid=0x409
