<properties 
   pageTitle="使用 InMage 进行保护时从 Azure 故障回复到 VMware 的步骤" 
   description="本文介绍了如何使用 Azure Site Recovery 和 vContinuum 工具将虚拟机故障回复到 VMware。" 
   services="site-recovery" 
   documentationCenter="" 
   authors="ruturaj" 
   manager="mkjain" 
   editor=""/>

<tags
   ms.service="site-recovery"
   ms.devlang="powershell"
   ms.tgt_pltfrm="na"
   ms.topic="article"
   ms.workload="required" 
   ms.date="04/23/2015"
   wacn.date="05/15/2015"
   ms.author="ruturajd@microsoft.com"/>

# 使用 InMage 进行保护时从 Azure 故障回复到 VMware 的步骤

在成功故障转移到 Azure 之后，虚拟机将出现在虚拟机选项卡中。当你决定故障回复时 - 下面是你需要遵循的步骤

## 概述

1.  在本地安装 vContinuum 服务器

    a.  将它配置为指向 CS

2.  在 Azure 上部署 PS

3.  本地安装 MT

4.  在本地保护保护故障的 VM 的步骤

    a.  配置注意事项

5.  监视已故障回复到本地的 VM 的保护

6.  将 VM 故障转移到本地

下面概述了需要使用以下步骤实现的安装。
在故障转移过程中已完成了部分安装。

-   蓝线是故障转移期间使用的连接。

-   红线是故障回复期间使用的连接。

-   箭头线将通过 Internet。

![](./media/site-recovery-failback-azure-to-vmware/image1.png)

## 在本地安装 vContinuum

vContinuum 安装将在http://go.microsoft.com/fwlink/?linkid=526305进行

1.  启动安装程序以开始安装 vContinuum。在显示欢迎屏幕后，请单击"下一步"开始指定设置

![](./media/site-recovery-failback-azure-to-vmware/image2.png)

2.  指定 CX 服务器 IP 地址和 CX 服务器端口。确保在复选框中选中 HTTP。

![](./media/site-recovery-failback-azure-to-vmware/image3.png)

    a.  若要发现 CX IP，请在 Azure 上转到 CS 部署并查看其仪表板。公共 IP 地址将显示在"公共虚拟 IP 地址"下。

![](./media/site-recovery-failback-azure-to-vmware/image4.png)

    b.  若要发现 CX 公用端口，请转到 VM 页中的终结点选项卡，并找到 HTTP 终结点公用端口

![](./media/site-recovery-failback-azure-to-vmware/image5.png)

3.  指定 CS 通行短语。在 CS 注册期间，你应该已记下了通行短语。在 MT 和 PS 部署期间，你也会使用通行短语。如果你不记得通行短语，可以在 Azure 上进入 CS 服务器并找到  下存储的通行短语 C:\\Program Files (x86)\\InMage Systems\\private\\connection.passphrase

    ![](./media/site-recovery-failback-azure-to-vmware/image6.png)

4.  指定安装 vContinuum 服务器的位置并开始安装

    ![](./media/site-recovery-failback-azure-to-vmware/image7.png)

5.  安装完成后，你可以启动 vContinuum 查看其工作情况。

    ![](./media/site-recovery-failback-azure-to-vmware/image8.png)

## 在 Azure 上安装 PS 服务器

需要在 Azure 上安装进程服务器，以便 Azure 中的 VM 可以将数据发回到本地 MT需要将 PS 部署在 Azure 上与配置服务器所在的同一网络中。

1.  在 Azure 中的"配置服务器"页上，选择添加新的进程服务器 ![](./media/site-recovery-failback-azure-to-vmware/image9.png)

2.  在进程服务器上配置以下设置以部署新服务器

    a.  为进程服务器指定一个名称

    b.  输入一个用户名，以管理员身份连接到虚拟机

    c.  输入登录密码

    d.  选择需要将进程服务器注册到的配置服务器。确保选择正确的配置服务器。这是用于保护和故障转移虚拟机的同一台服务器。

    e.  指定需要将进程服务器部署到的 Azure 网络。确保选择配置服务器所在的同一个网络。

    f.  指定选定子网中的某个唯一 IP 地址。

    g.  开始部署进程服务器。

![](./media/site-recovery-failback-azure-to-vmware/image10.png)

1.  将触发一个用于部署进程服务器的作业

![](./media/site-recovery-failback-azure-to-vmware/image11.png)

在 Azure 上部署进程服务器后，你可以使用指定的凭据登录到该服务器。使用正向定向保护时所用的相同步骤来注册 PS。

![](./media/site-recovery-failback-azure-to-vmware/image12.png)

在 VM 属性下，故障回复期间注册的服务器将不可见。它们只会在它们注册到的配置服务器中的"服务器"选项卡下可见。

可能需要在大约 10-15 分钟后，PS 才会列在 CS 下。

## 在本地安装 MT 服务器

根据源端虚拟机，你需要在本地安装 Linux 或 Windows 主目录服务器。

### 部署 Windows MT

vContinuum 安装程序中已捆绑了 Windows MT。当你安装 vContinuum 时，MT 也会部署在同一台计算机上，并注册到配置服务器。

1.  若要开始部署，请在 ESX 主机本地创建一个空计算机，你需要将 VM 从 Auzre 恢复到该计算机。

2.  确保至少有两个磁盘附加到 VM - 一个磁盘用于存储 OS，另一个磁盘用作保留驱动器。

3.  安装操作系统。

4.  在服务器上安装 vContinuum。这还会完成 MT 的安装。

### 部署 Linux MT

1.  若要开始部署，请在 ESX 主机本地创建一个空计算机，你需要将 VM 从 Auzre 恢复到该计算机。

2.  确保至少有两个磁盘附加到 VM - 一个磁盘用于存储 OS，另一个磁盘用作保留驱动器。

3.  安装 Linux 操作系统。

    a.  注意：Linux 主目标 (MT) 系统不应为根或保留存储空间使用 LVM。默认情况下，Linux MT 已配置为避免发现 LVM 分区/磁盘。

    b.  可以创建的分区包括
        ![](./media/site-recovery-failback-azure-to-vmware/image13.png)

4.  在开始安装 MT 之前，请执行以下安装后步骤。

#### OS 安装后步骤

若要获取 Linux 虚拟机中每个 SCSI 硬盘的 SCSI ID，你应该启用参数"disk.EnableUUID = TRUE"。

若要启用此参数，请遵循下面提供的步骤：

a. 关闭你的虚拟机。

b. 在左侧面板中右键单击 VM 对应的条目，然后选择"编辑设置"。

c. 单击"选项"选项卡。

d. 在左侧选择"高级""常规"项，然后单击右侧显示的"配置参数"。

![](./media/site-recovery-failback-azure-to-vmware/image14.png)

当计算机正在运行时，"配置参数"选项将处于停用状态。若要使此选项卡处于活动状态，请关闭计算机。

e. 查看是否存在包含 **disk.EnableUUID** 的行。

如果存在该行并且其值设置为 False，请将该值覆盖为True（True 和 False 值不区分大小写）。

如果存在该行并且其值设置为 true，请单击"取消"，然后在启动来宾操作系统后，在来宾操作系统中测试 SCSI 命令。

f. 如果不存在该行，请单击"添加行"。

在"名称"列中添加 disk.EnableUUID。

将其值设置为 TRUE

注意：添加上述值时请不要包括双引号。

![](./media/site-recovery-failback-azure-to-vmware/image15.png)

#### 下载并安装其他程序包

注意：在下载并安装其他程序包之前，请确保系统已建立 Internet 连接。

\# yum install -y xfsprogs perl lsscsi rsync wget kexec-tools

上述命令将从 CentOS 6.6 存储库下载并安装下面提到的 15 个程序包。

bc-1.06.95-1.el6.x86\_64.rpm

busybox-1.15.1-20.el6.x86\_64.rpm

elfutils-libs-0.158-3.2.el6.x86\_64.rpm

kexec-tools-2.0.0-280.el6.x86\_64.rpm

lsscsi-0.23-2.el6.x86\_64.rpm

lzo-2.03-3.1.el6\_5.1.x86\_64.rpm

perl-5.10.1-136.el6\_6.1.x86\_64.rpm

perl-Module-Pluggable-3.90-136.el6\_6.1.x86\_64.rpm

perl-Pod-Escapes-1.04-136.el6\_6.1.x86\_64.rpm

perl-Pod-Simple-3.13-136.el6\_6.1.x86\_64.rpm

perl-libs-5.10.1-136.el6\_6.1.x86\_64.rpm

perl-version-0.77-136.el6\_6.1.x86\_64.rpm

rsync-3.0.6-12.el6.x86\_64.rpm

snappy-1.1.0-1.el6.x86\_64.rpm

wget-1.12-5.el6\_6.1.x86\_64.rpm

注意：如果源计算机为根设备或引导设备使用了 Reiser 或 XFS 文件系统，则应该在保护之前，在 Linux 主目标中下载并安装以下程序包。

\# cd /usr/local

\# wget
<http://elrepo.org/linux/elrepo/el6/x86_64/RPMS/kmod-reiserfs-0.0-1.el6.elrepo.x86_64.rpm>

\# wget
<http://elrepo.org/linux/elrepo/el6/x86_64/RPMS/reiserfs-utils-3.6.21-1.el6.elrepo.x86_64.rpm>

\# rpm -ivh kmod-reiserfs-0.0-1.el6.elrepo.x86\_64.rpm
reiserfs-utils-3.6.21-1.el6.elrepo.x86\_64.rpm

\# wget
<http://mirror.centos.org/centos/6.6/os/x86_64/Packages/xfsprogs-3.1.1-16.el6.x86_64.rpm>

\# rpm -ivh xfsprogs-3.1.1-16.el6.x86\_64.rpm

#### 应用自定义配置更改

在应用自定义配置更改之前，请确保已完成安装后步骤

若要应用自定义配置更改，请遵循下述步骤：

1. 将 RHEL 6-64 统一代理二进制文件复制到新建的 OS。

2. 运行以下命令来解压缩二进制文件。

**tar -zxvf \<文件名\>**

3. 执行以下命令来指定权限。

\# **chmod 755 ./ApplyCustomChanges.sh**

4. 执行以下命令来运行脚本。

**\# ./ApplyCustomChanges.sh**

注意：仅在服务器上执行该脚本一次。成功执行上述脚本后，**重新启动**服务器。

### 开始安装 MT

[下载](http://go.microsoft.com/fwlink/?LinkID=529757) Linux 主目标服务器安装文件。

使用所选的 sftp 客户端实用工具，将下载的 Linux主目标服务器安装程序复制到 Linux 主目标虚拟机。或者，你可以登录到 Linux 主目标虚拟机，然后使用 wget 从提供的链接下载安装程序包。

使用所选的 ssh 客户端登录到 Linux 主目标服务器虚拟机。

如果你已通过 VPN 连接到部署了 Linux 主目标服务器的 Azure 网络，请使用从虚拟机仪表板获取的 Linux 主目标服务器内部 IP 地址和端口 22，通过安全 Shell 连接到 Linux 主目标服务器。

如果你要通过公共 Internet 连接来连接到 Linux 主目标服务器，请使用 Linux 主目标服务器的公共 IP 地址（从虚拟机仪表板页获取）以及为 ssh 创建的公共终结点登录到 Linux 主目标服务器。

通过从 Linux 主目标服务器安装程序复制到的目录执行

*"tar -xvzf Microsoft-ASR\_UA\_8.2.0.0\_RHEL6-64\*"*从使用 gzip 压缩的 Linux 主目标服务器安装程序 tar 存档中提取文件。

![](./media/site-recovery-failback-azure-to-vmware/image16.png)

如果将安装程序文件提取到了其他目录，请将目录切换到 tar 存档内容所提取到的目录。从此目录路径执行"sudo ./install.sh"

![](./media/site-recovery-failback-azure-to-vmware/image17.png)

当系统提示你选择此代理的主角色时，请选择 2（主目标）

将其他交互式安装选项保留为其默认值。

等待余下的安装步骤完成，然后将显示主机配置界面。适用于 Linux 主目标服务器的主机配置实用工具是一个命令行实用工具。请不要调整 ssh 客户端实用工具窗口的大小。使用箭头键选择"全局"选项，然后在键盘上按 Enter。

![](./media/site-recovery-failback-azure-to-vmware/image18.png)

![](./media/site-recovery-failback-azure-to-vmware/image19.png)

1.  对于"IP"字段，请输入从虚拟机仪表板页获取的配置服务器的内部 IP 地址，然后按 Enter。

2.  对于"端口"字段，请输入 22，然后按 Enter。

3.  将"使用 https:"保留为"是"。再次按 Enter。

4.  输入在配置服务器上生成的通行短语。如果在 Windows 计算机中使用 PuTTY 客户端与 Linux 主目标服务器建立 ssh 连接，则可以使用 Shift+Insert 来粘贴剪贴板的内容。使用 Ctrl+C 将通行短语复制到本地剪贴板，然后使用 Shift+Insert 粘贴通行短语。按 Enter

5.  使用右箭头键导航到退出按钮，然后按 Enter。

等待安装完成

![](./media/site-recovery-failback-azure-to-vmware/image20.png)

如果出于某种原因而无法将 Linux 主目标服务器注册到配置服务器，可以通过从/usr/local/ASR/Vx/bin/hostconfigcli运行主机配置实用工具来再次注册（首先需要通过以超级用户身份执行 chmod，来设置对此目录的访问权限）


#### 验证主目标服务器是否已注册到配置服务器。

可以通过访问 Azure Site Recovery 保管库中"配置服务器"页下的"服务器详细信息"页，来验证是否已成功将主目标服务器注册到配置服务器


## 开始保护要故障回复到本地的虚拟机

在将 VM 故障回复到本地之前，需要先保护要故障回复到本地的虚拟机。遵循以下步骤来保护应用程序的 VM。

注意：
-----

将 VM 故障转移到 Azure 时，将会为页面文件添加一个额外的临时驱动器。故障转移的 VM 通常不需要此额外驱动器，因为它可能已经为页面文件提供了一个专用的驱动器。

在开始反向保护虚拟机之前，需要确保将该驱动器脱机，使它不会受到保护。

为此，请执行以下操作：

1.  打开"计算机管理"（通过控制面板中的管理工具，或者在资源管理器窗口中右键单击"此电脑"，然后选择"管理"。）

2.  选择"存储管理"列出已联机并已附加到计算机的磁盘。

3.  选择附加到计算机的临时磁盘，然后选择它以使其脱机。

4.  成功将它脱机后，你可以继续反向保护虚拟机。

### VM 保护计划

在 Azure 门户中，查看虚拟机的状态并确保它们已故障转移。

![](./media/site-recovery-failback-azure-to-vmware/image21.png)

注意：在从 Azure 故障转移回到本地期间，Azure VM 被视为类似于物理 VM。故障转移到本地相当于故障转移到虚拟机。

1.  在计算机上启动 vContinuum

![](./media/site-recovery-failback-azure-to-vmware/image8.png)

1.  在"选择应用程序"设置中选择"P2V"

2.  单击"新建保护"选项以开始操作

3.  在打开的新窗口中，你可以开始保护要故障回复到本地的虚拟机。

    a.  根据你要故障回复的 VM 选择 OS 类型，然后单击"获取详细信息"

    b.  在"主服务器详细信息"中，你需要确定想要保护的虚拟机。

    c.  虚拟机将按照其在 vCenter 或 Azure 上显示的计算机主机名列出

    d.  虚拟机列在虚拟机故障转移之前所在的 vCenter 主机名下。

    e.  在确定想要保护的虚拟机后，你可以逐个选择这些 VM。

4.  在选择要保护的虚拟机时（该虚拟机已故障转移到 Azure），你将会看到一个弹出窗口，其中提供了该虚拟机的两个条目。这是因为，CS 已检测到其中注册的虚拟机的两个实例。你需要删除本地 VM 的条目，以便可以保护正确的 VM。请注意，你将会看到按计算机主机名列出的条目。

![](./media/site-recovery-failback-azure-to-vmware/image22.png)

    a.  若要选择正确的 VM，你可以参考其 IP 地址。本地 IP 地址范围将是本地 VM。

    b.  单击"删除"以删除该条目

![](./media/site-recovery-failback-azure-to-vmware/image23.png)

    c.  转到 vCenter，然后在 vCenter 上停止虚拟机

    d.  接下来，你还可以删除本地虚拟机

5.  然后，需要指定要在其上保护 VM 的本地 MT 服务器。

    a.  连接到要故障回复到的 vCenter

![](./media/site-recovery-failback-azure-to-vmware/image24.png)

a.  根据要将虚拟机恢复到的主机选择MT服务器

![](./media/site-recovery-failback-azure-to-vmware/image24.png)

1.  接下来，提供每个虚拟机的复制选项

![](./media/site-recovery-failback-azure-to-vmware/image25.png)

a.  需要选择恢复端"数据存储"- 这是 VM 要恢复到的数据存储

要为每个 VM 提供的不同选项包括

<table>
<tr><td>选项</td><td>选项建议值</td></tr>
<tr><td>进程服务器 IP</td><td>选择在 Azure 上部署的 PS</td></tr>
<tr><td>保留大小 (MB)</td><td></td></tr>
<tr><td>保留值</td><td>1</td></tr>
<tr><td>天/小时</td><td>天</td></tr>
<tr><td>一致性间隔</td><td>1</td></tr>
<tr><td>选择目标数据存储</td><td>在恢复端上可用的数据存储。此数据存储应有足够的空间，并且可用于要在其上识别虚拟机的 ESX 主机。</td></tr>
</table>


1.  接下来，可以配置故障转移到本地站点后虚拟机要获取的属性。可配置的不同属性如下

![](./media/site-recovery-failback-azure-to-vmware/image26.png)


  <table>
<tr><td>属性</td><td>如何配置</td></tr>
<tr><td>网络配置</td><td>对于检测到的每个 NIC，配置虚拟机的故障回复 IP 地址。选择"NIC"，然后单击"更改"指定 IP 地址详细信息。

</td></tr>
<tr><td>硬件配置</td><td>可以指定 VM 的"CPU"和"内存"值。可向你正在尝试保护的所有 VM 应用此设置。

若要确定"CPU"和"内存"的正确值，可以参考 IAAS VM 角色大小，并查看分配的核心数和内存。
</td></tr>
<tr><td>显示名称</td><td>在故障转移回到本地后，你可以选择重命名要在 vCenter 清单中显示的虚拟机。请注意，此处看到的默认值是虚拟机主机名。若要确定 VM 名称，可以参考保护组中的 VM 列表。</td></tr>
<tr><td>NAT 配置</td><td>下面将详细讨论</td></tr>
</table>

![](./media/site-recovery-failback-azure-to-vmware/image27.png)



> **PS NAT 设置选项**
>
> 若要启用虚拟机保护，需要建立
> 两个通信通道。
>
> 第一个通道在虚拟机和进程服务器之间
> 建立。此通道从 VM 收集数据并将数据发送到
> PS。然后，PS 会将此数据发送到主目标。如果要保护的
> 进程服务器和虚拟机位于同一个
> Azure vNet 上，则你不需要使用 NAT 设置。如果要保护的 PS 和
> 虚拟机在两个不同的 vNet 上，则你
> 需要指定 PS 的 NAT 设置，并检查第一个
> 选项。
>
> ![](./media/site-recovery-failback-azure-to-vmware/image28.png)
>
> 若要确定进程服务器的公共 IP，可以
> 转到 Azure 中的 PS 部署，然后查看其公共 IP 地址。
>
> 第二个通道在进程服务器和主目标之间
> 建立。是否选择使用 NAT 取决于是要在 MT 和 PS 之间
> 使用基于 VPN 的连接，还是要通过 Internet 进行
> 保护。如果 PS 通过 VPN 来与 MT 通信，
> 则不应选择此选项。如果主目标需要通过 Internet 来
> 与进程服务器通信，请指定 PS 的 NAT
> 设置。
>
> ![](./media/site-recovery-failback-azure-to-vmware/image29.png)
>
> 若要确定进程服务器的公共 IP，可以
> 转到 Azure 中的 PS 部署，然后查看其公共 IP 地址。
>
> ![](./media/site-recovery-failback-azure-to-vmware/image30.png)

1.  如果未按步骤 5.d 中的指定删除本地虚拟机，并且步骤 7.a 中选择的、要故障回复到的数据存储仍然包含旧 VMDK，则你还需要确保在新位置创建故障回复 VM。为此，你可以选择"高级设置"，然后在"高级设置"的"文件夹名称设置"部分中指定要还原到的备用文件夹。

![](./media/site-recovery-failback-azure-to-vmware/image31.png)

"高级设置"中的其他选项可保留默认值。
请确保将文件夹名称设置应用到所有服务器。

1.  接下来，转到保护的最后一个阶段。此时，你需要运行就绪状态检查，以确保虚拟机已准备好在本地受到保护。

![](./media/site-recovery-failback-azure-to-vmware/image32.png)

	a.  单击"就绪状态检查"并等待检查完成。

	b.  如果所有虚拟机都已准备就绪，则"就绪报告"选项卡将会显示相应的信息。

	c.  如果对所有虚拟机的就绪报告为成功，则你可以指定保护计划的名称

![](./media/site-recovery-failback-azure-to-vmware/image33.png)

	d.  为计划指定新名称，然后单击下面的按钮开始保护。


1.  随后将开始保护。

    a.  可以在 vContinuum 中查看保护进度

![](./media/site-recovery-failback-azure-to-vmware/image34.png)

	b.  完成**激活保护计划**步骤后，你可以通过 ASR 门户监视虚拟机的保护。

	c.  也可以通过 Azure Site Recovery 来监视保护。

![](./media/site-recovery-failback-azure-to-vmware/image35.png)

你还可以通过单击进入虚拟机，然后在卷复制部分下监视其进度，来查看虚拟机对的确切状态。

![](./media/site-recovery-failback-azure-to-vmware/image36.png)

## 准备故障回复计划

可以使用 vContinuum 来准备故障回复计划，使应用程序随时可以故障转移回到本地。这些恢复计划与 ASR 恢复计划非常相似。

1.  启动 vContinuum 并选择"管理计划"选项。

2.  使用子选项选择"恢复"。

![](./media/site-recovery-failback-azure-to-vmware/image37.png)

1.  你可以查看已用于保护虚拟机的所有计划列表。这是可用于恢复的相同计划。

2.  选择"保护计划"，然后选择要在其中恢复的所有 VM。

    a.  在选择每个 VM 时，你可以查看有关源 VM、VM 要恢复到的目标 ESX 服务器和源 VM 磁盘的详细信息

3.  单击"下一步"启动"恢复"向导

4.  选择要恢复的虚拟机

    a.  查看可恢复的所有虚拟机的列表

![](./media/site-recovery-failback-azure-to-vmware/image38.png)

	b.  你可以基于多个选项进行恢复，不过，我们建议使用"最新标记"这可以确保使用虚拟机中的最新数据。

	c.  选择"对所有 VM 应用"以确保为所有虚拟机选择最新标记。


1.  运行"就绪状态检查"。这将会告知是否配置了正确的参数来启用虚拟机的最新标记恢复。如果所有检查都已成功，请单击"下一步"，否则请查看日志并解决错误。

![](./media/site-recovery-failback-azure-to-vmware/image39.png)

2.  在向导的"VM 配置"步骤中，确保恢复设置正确。如果 VM 设置不符合需要，你可以选择对其进行更改。由于我们已在保护期间完成此操作，因此现在你可以选择将其忽略。

![](./media/site-recovery-failback-azure-to-vmware/image40.png)

1.  最后，查看要恢复的虚拟机列表。

    a.  指定虚拟机的恢复顺序。

注意：虚拟机是使用计算机主机名列出的。可能很难将计算机主机名映射到虚拟机。若要映射名称，可以转到 Azure IAAS 中的虚拟机仪表板，然后查看虚拟机的主机名。

![](./media/site-recovery-failback-azure-to-vmware/image41.png)

1.  指定"恢复计划名称"，然后在"恢复选项"中选择"以后恢复"。

    a.  如果你想要立即恢复，可以在"恢复选项"中选择"立即恢复"。

    b.  建议选择"以后恢复"，因为保护的初始复制可能尚未完成

    c.  最后单击"恢复"按钮，以保存计划或者根据"恢复选项"触发恢复。

2.  你可以查看恢复状态，并了解该计划是否已成功保存。

![](./media/site-recovery-failback-azure-to-vmware/image42.png)

1.  如果你选择了以后恢复，系统会通知你已创建计划，并且你可以在以后恢复。

![](./media/site-recovery-failback-azure-to-vmware/image43.png)

## 恢复虚拟机

创建计划后，你可以选择恢复虚拟机。作为先决条件，你需要确保虚拟机已完成同步。

![](./media/site-recovery-failback-azure-to-vmware/image44.png)

如果"复制状态"显示"正常"，则表示保护已完成，并且符合 RPO 阈值。若要确认复制对的运行状况，可以转到虚拟机属性并查看复制运行状况。

**在启动恢复之前，请关闭 Azure 虚拟机。这可以确保不会出现裂脑，并且能够使用应用程序的一个副本来为最终客户提供服务。仅当已成功确保关闭了 Azure VM 时，才能继续执行以下步骤。**

若要开始将虚拟机恢复到本地，你需要启动已保存的计划。

1.  在 VContinuum 上选择**监视**计划。

2.  向导将列出到目前已执行的计划。

![](./media/site-recovery-failback-azure-to-vmware/image45.png)

1.  选择"恢复"节点，然后选择要恢复的计划。

    a.  系统将通知你该计划尚未启动。

2.  单击"启动"以开始恢复。

3.  你可以监视虚拟机的恢复


![](./media/site-recovery-failback-azure-to-vmware/image46.png)

4. 在打开 VM 后，你可以在 vCenter 上连接到虚拟机。

## 故障回复后在 Azure 上重新保护

完成故障回复后，你可能需要重新保护虚拟机。以下步骤将帮助你重新配置保护

1.  检查本地虚拟机是否在正常工作，并且最终客户能够访问应用程序。

2.  在 Azure Site Recovery 中，选择虚拟机并将其删除。

    a.  选择禁用虚拟机的保护。这将确保 VM 不再受保护。

3.  转到 Azure IAAS 虚拟机，并删除故障转移的 VM。

4.  在 vSpehere 上删除旧 VM - 这是以前故障转移到 Azure 的 VM。

5.  在 ASR 门户中，选择保护最近故障转移的虚拟机。

6.  VM 受保护后，你可以将它们添加到恢复计划，并继续为其提供保护。


<!--HONumber=53-->