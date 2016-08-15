<properties 
   pageTitle="将 VMware 虚拟机和物理服务器故障回复到本地站点 | Azure"
   description="了解在将 VMware VM 和物理服务器故障转移到 Azure 后，如何故障回复到本地站点。" 
   services="site-recovery" 
   documentationCenter="" 
   authors="rayne-wiselman" 
   manager="jwhit" 
   editor=""/>

<tags
   ms.service="site-recovery"
   ms.date="01/11/2015"
   wacn.date="02/25/2016"/>

# 将 VMware 虚拟机和物理服务器故障回复到本地站点

> [AZURE.SELECTOR]
- [Enhanced](/documentation/articles/site-recovery-failback-azure-to-vmware-classic/)
- [Legacy](/documentation/articles/site-recovery-failback-azure-to-vmware-classic-legacy/)


本文介绍如何将 Azure 虚拟机从 Azure 故障回复到本地站点。



## 概述

下图显示了这种情况下的故障回复体系结构。

![](./media/site-recovery-failback-azure-to-vmware-classic/architecture.png)

下面是故障回复的工作原理：

- 故障转移到 Azure 以后，可通过以下几个阶段故障回复到本地站点：
	- **阶段 1**：对 Azure VM 进行重新保护，使之开始复制回在本地站点运行的 VMware VM。启用重新保护会将 VM 移到故障回复保护组中，该保护组是在你最初创建故障转移保护组时自动创建的。如果已将故障转移保护组添加到某个恢复计划，则也会自动将故障回复保护组添加到该计划。在重新保护期间，你需要指定如何进行故障回复计划。
	- **阶段 2**：将 Azure VM 复制到本地站点以后，可以通过运行故障转移从 Azure 进行故障回复。
	- **阶段 3**：对数据进行故障回复以后，即可对故障回复到的本地 VM 进行重新保护，使之开始复制到 Azure。




### 故障回复到原始或备用位置

如果你对某个 VMware VM 进行了故障转移，则可故障回复到同一源 VM，前提是该 VM 仍存在于本地。在这种情况下，仅对增量更改进行故障回复。请注意：

- 如果你对物理服务器进行了故障转移，则会始终故障回复到新的 VMware VM。
- 如果故障回复到原始 VM，则需满足以下条件：
	- 如果该 VM 由 vCenter 服务器管理，则本地站点中的 VM 所使用的数据存储必须与运行本地主目标服务器的 VM 相同。否则，你需要先对其进行迁移，然后才能故障回复到同一 VM。
	- 如果该 VM 位于 ESX 主机上，但不由 vCenter 管理，则该 VM 的硬盘必须与主目标 VM 的硬盘位于同一数据存储中。
	- 如果你的 VM 位于 ESX 主机上但不使用 vCenter，则应完成针对 ESX 主机的发现操作，然后才能进行重新保护。如果你要对物理服务器进行故障回复，此规则也同样适用。
	- 另一选项（如果本地 VM 存在）是在进行故障回复之前删除它。然后，故障回复就会在与主目标 ESX 主机相同的主机上创建新的 VM。
	
- 故障回复到备用位置时，会将数据恢复到与本地主目标服务器所用数据存储和 ESX 主机相同的数据存储和 ESX 主机。


## 先决条件

- 需要 VMware 环境才能对 VMware VM 和物理服务器进行故障回复。不支持故障回复到物理服务器。
- 为了进行故障回复，你应该在最初设置保护时，就已经创建 Azure 网络。故障回复需要进行 VPN 或 ExpressRoute 连接，从 Azure VM 所在的 Azure 网络连接到本地站点。
- 如果你要故障回复到的 VM 受 vCenter 服务器管理，则需确保你拥有在 vCenter 服务器上发现 VM 所必需的权限。
- 如果 VM 上存在快照，重新保护会失败。你可以删除这些快照或磁盘。 
- 故障回复之前，你需要创建多个组件：
	- **在 Azure 中创建进程服务器**。这是你需要在故障回复过程中创建并保持运行的 Azure VM。完成故障回复后，可以删除该虚拟机。
	- **创建主目标服务器**：主目标服务器发送和接收故障回复数据。在本地创建的管理服务器默认情况下已安装主目标服务器。但是，你可能需要创建单独的故障回复用主目标服务器，具体取决于故障回复流量。
	- 如果你需要创建其他在 Linux 上运行的主目标服务器，，则需先设置 Linux VM，然后再安装主目标服务器，如下所示。

## 在 Azure 中设置进程服务器

需在 Azure 中安装进程服务器，以便 Azure VM 将数据发送回本地主目标服务器。

1.  在 Site Recovery 门户 >“配置服务器”上，选择添加新的进程服务器。

	![](./media/site-recovery-failback-azure-to-vmware-classic/ps1.png)

2.  指定进程服务器名称，然后输入要使用的名称和密码，以管理员身份连接到 Azure VM。在“配置服务器”中，选择本地管理服务器，并指定应在其中部署进程服务器的 Azure 网络。这应该是 Azure VM 所在的网络。指定来自所选子网的唯一 IP 地址，然后开始部署。

	![](./media/site-recovery-failback-azure-to-vmware-classic/ps2.png)

	将触发一个用于部署进程服务器的作业

	![](./media/site-recovery-failback-azure-to-vmware-classic/ps3.png)

	在 Azure 上部署进程服务器后，你可以使用指定的凭据登录到该服务器。第一次登录时，会运行进程服务器对话框。键入本地管理服务器的 IP 地址及其密码。保留默认的端口 443 设置。你也可以保留默认的 9443 端口进行数据复制，除非你在安装本地管理服务器时特意修改了此设置。

	>[AZURE.NOTE] 服务器不会显示在“VM 属性”下。它只显示在所注册到的管理服务器中的“服务器”选项卡下。可能需要大约 10-15 分钟才能显示进程服务器。


## 在本地安装主目标服务器

主目标服务器接收故障回复数据。主目标服务器会自动安装在本地管理服务器上，但如果你要对大量数据进行故障回复，则需安装另一主目标服务器。请按如下所述执行此操作：

>[AZURE.NOTE] 如果你想要在 Linux 上安装主目标服务器，请遵循下一过程中的说明。

1. 如果你要在 Windows 上安装主目标服务器，请通过需在其中安装主目标服务器的 VM 打开“快速启动”页，然后下载 Azure Site Recovery 统一安装程序向导的安装文件。
2. 运行安装程序，然后在“开始之前”中选择“添加额外的进程服务器以横向扩展部署”。
3. 完成向导。在“配置服务器详细信息”页上，指定此主目标服务器的 IP 地址，以及访问 VM 所需的密码。

### 将 Linux VM 安装为主目标服务器
若要将运行主目标服务器的管理服务器安装为 Linux VM，至少需安装 Cent)S 6.6 操作系统，检索每个 SCSI 硬盘的 SCSI ID，安装一些其他的程序包，然后应用某些自定义更改。

#### 安装 CentOS 6.6

1.	在管理服务器 VM 上至少安装 CentOS 6.6 操作系统。将 ISO 留在 DVD 驱动器中，然后启动系统。跳过介质测试，选择“US English”作为语言，选择“基本存储设备”，确保硬盘驱动器上没有任何重要数据，然后单击“是”，放弃任何数据。输入管理服务器的主机名，然后选择服务器网络适配器。在“编辑系统”对话框中选择“自动连接”，然后添加静态 IP 地址、网络和 DNS 设置。****指定一个时区，以及一个用于访问管理服务器的根密码。 
2.	当系统询问你的安装类型时，应选择“创建自定义布局”作为分区。单击“下一步”后，选择“免费”，然后单击“创建”。通过 **FS Type:** **ext4** 创建 **/**、**/var/crash** 和 **/home 分区**。创建交换分区，格式为 **FS Type: swap**。
3.	如果发现有预先存在的设备，则会显示警告消息。单击“格式化”，将驱动器格式化为分区设置。单击“将更改写入磁盘”以应用分区更改。
4.	选择“安装引导加载程序”>“下一步”，在根分区上安装引导加载程序。
5.	安装完成后，单击“重新启动”。


#### 检索 SCSI ID

1. 安装后，检索 VM 中每个 SCSI 硬盘的 SCSI ID。为此，请先关闭管理服务器 VM，然后在 VMware 的 VM 属性中，右键单击“VM”条目 >“编辑设置”>“选项”。
2. 选择“高级”>“常规项”，然后单击“配置参数”。当计算机正在运行时，此选项将处于停用状态。若要使其处于活动状态，必须关闭计算机。
3. 如果 **disk.EnableUUID** 这一行存在，请确保将值设置为 **True**（区分大小写）。如果已进行了相应设置，则可先取消，然后在启动来宾操作系统后对其中的 SCSI 命令进行测试。 
4.	如果该行不存在，请单击“添加行”，并在添加时将值设置为 **True**。不要使用双引号。

#### 安装其他程序包

需下载和安装其他程序包。

1.	请确保主目标服务器已连接到 Internet。
2.	运行以下命令，从 CentOS 存储库下载并安装 15 个程序包：**# yum install –y xfsprogs perl lsscsi rsync wget kexec-tools**。
3.	如果所要保护的源计算机运行针对根设备或启动设备的 Linux wit Reiser 或 XFS 文件系统，则应下载和安装其他程序包，如下所示：

	- # cd /usr/local
	- # wget [http://elrepo.org/linux/elrepo/el6/x86\_64/RPMS/kmod-reiserfs-0.0-1.el6.elrepo.x86\_64.rpm](http://elrepo.org/linux/elrepo/el6/x86_64/RPMS/kmod-reiserfs-0.0-1.el6.elrepo.x86_64.rpm)
	- # wget [http://elrepo.org/linux/elrepo/el6/x86\_64/RPMS/reiserfs-utils-3.6.21-1.el6.elrepo.x86\_64.rpm](http://elrepo.org/linux/elrepo/el6/x86_64/RPMS/reiserfs-utils-3.6.21-1.el6.elrepo.x86_64.rpm)
	- # rpm –ivh kmod-reiserfs-0.0-1.el6.elrepo.x86\_64.rpm reiserfs-utils-3.6.21-1.el6.elrepo.x86\_64.rpm
	- # wget [http://mirror.centos.org/centos/6.6/os/x86\_64/Packages/xfsprogs-3.1.1-16.el6.x86\_65.rpm](http://mirror.centos.org/centos/6.6/os/x86_64/Packages/xfsprogs-3.1.1-16.el6.x86_65.rpm)
	- # rpm –ivh xfsprogs-3.1.1-16.el6.x86\_64.rpm

#### 应用自定义更改

在完成安装后步骤并安装程序包以后，请执行下述操作以应用自定义更改：

1.	将 RHEL 6-64 统一代理二进制文件复制到 VM。运行以下命令来解压缩二进制文件：**tar –zxvf <file name>**
2.	运行以下命令来指定权限：**# chmod 755 ./ApplyCustomChanges.sh**
3.	运行脚本：**# ./ApplyCustomChanges.sh**。只应运行脚本一次。成功运行脚本后，重新启动服务器。


## 运行故障回复

### 重新保护 Azure VM

1.	在 Site Recovery 门户 >“计算机”选项卡中，选择已进行故障转移的 VM，然后单击“重新保护”。
2.	在“主目标服务器”和“进程服务器”中，选择本地主目标服务器，以及 Azure VM 进程服务器。
3.	选择已配置的用于连接到 VM 的帐户。
4.	选择保护组的故障回复版本。例如，如果 VM 是在 PG1 中进行保护的，则需选择 PG1\_Failback。
5.	如果要恢复到备用位置，则请选择针对主目标服务器配置的保留驱动器和数据存储。故障回复到本地站点时，故障回复保护计划中的 VMware VM 将使用与主目标服务器相同的数据存储。如果要将副本 Azure VM 恢复到相同的本地 VM，则本地 VM 所在的数据存储应与主目标服务器的相同。如果没有本地 VM，则会在重新保护期间创建一个新的。

	![](./media/site-recovery-failback-azure-to-vmware-classic/failback1.png)

6.	单击“确定”开始重新保护以后，作业就会开始将 VM 从 Azure 复制到本地站点。你可以在“作业”选项卡上跟踪进度。

	![](./media/site-recovery-failback-azure-to-vmware-classic/failback2.png)

### 运行到本地站点的故障转移

进行重新保护以后，VM 将变为其保护组的故障回复版本，并会被自动添加到故障转移到 Azure 时使用的恢复计划（如果有）。

1.	在“恢复计划”页上，选择包含故障回复组的恢复计划，然后单击“故障转移”>“非计划的故障转移”。
2.	在“确认故障转移”中，验证故障转移方向（故障转移到 Azure），然后选择要用于故障转移（最新）的恢复点。配置复制属性时，如果启用了“多 VM”，则可恢复到与应用一致的或崩溃状态一致的最新恢复点。单击复选标记开始故障转移。
3.	故障转移期间，Site Recovery 会关闭 Azure VM。查看故障回复是否按预期完成以后，即可查看 Azure VM 是否已按预期关闭。

### 重新保护本地站点

故障回复完成以后，数据会回到本地站点，但不受保护。若要再次开始复制到 Azure，请执行以下操作：

1.	在 Site Recovery 门户 >“计算机”选项卡中，选择已进行故障回复的 VM，然后单击“重新保护”。 
2.	确保可以按预期复制到 Azure 以后，即可在 Azure 中删除已进行故障回复的 Azure VM（当前尚未运行）。


## 使用 ExpressRoute 进行故障回复

可以通过 VPN 连接或 Azure ExpressRoute 进行故障回复。若要使用 ExpressRoute，请注意：

- 应在 Azure 虚拟网络上设置 ExpressRoute，源计算机会故障转移到该网络，而故障转移完成后，Azure VM 也将位于其中。
- 在公共终结点上，数据将复制到 Azure 存储帐户。你应该在 ExpressRoute 中设置公共对等互连，以便通过进行 Site Recovery 复制的目标数据中心使用 ExpressRoute。

<!---HONumber=Mooncake_0215_2016-->