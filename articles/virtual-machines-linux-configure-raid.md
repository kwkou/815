<properties 
	pageTitle="在运行 Linux 的虚拟机上配置软件 RAID | Azure" 
	description="了解如何使用 mdadm 在 Azure 中的 Linux 上配置 RAID。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="szarkos" 
	writer="szark" 
	manager="timlt" 
	editor=""
	tag="azure-service-management,azure-resource-manager" />

<tags 
	ms.service="virtual-machines-linux" 
	ms.date="03/25/2016" 
	wacn.date="05/24/2016"/>



# 在 Linux 上配置软件 RAID
在 Azure 中的 Linux 虚拟机上使用软件 RAID 将多个附加的数据磁盘呈现为一个单一的 RAID 设备，是一种常见的情形。通常，使用这种方法可以改进性能，而且与只使用单独一块磁盘相比，吞吐量也会有所改进。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]
 

## 附加数据磁盘
配置 RAID 设备通常需要两个或更多的空数据磁盘。本文将不详细介绍如何为 Linux 虚拟机附加数据磁盘。请参阅 Azure 文章[附加磁盘](/documentation/articles/virtual-machines-linux-classic-attach-disk/#attachempty)，以详细了解如何在 Azure 上为 Linux 虚拟机附加空数据磁盘。

>[AZURE.NOTE]“特小型”VM 大小不支持将一个以上数据磁盘附加到虚拟机。请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/cloud-services-sizes-specs/)，以了解有关 VM 大小和支持的数据磁盘数量的详细信息。


## 安装 mdadm 实用程序

- **Ubuntu**

		# sudo apt-get update
		# sudo apt-get install mdadm

- **CentOS 和 Oracle Linux**

		# sudo yum install mdadm

- **SLES 和 openSUSE**

		# zypper install mdadm


## 创建磁盘分区
在此示例中，我们将在 /dev/sdc 上创建一个单一的磁盘分区。该新磁盘分区将被命名为 /dev/sdc1。

- 启动 fdisk，以开始创建分区

		# sudo fdisk /dev/sdc
		Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
		Building a new DOS disklabel with disk identifier 0xa34cb70c.
		Changes will remain in memory only, until you decide to write them.
		After that, of course, the previous content won't be recoverable.

		WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
				 switch off the mode (command 'c') and change display units to
				 sectors (command 'u').

- 在提示符处按 N 键，以创建新分区：

		Command (m for help): n

- 接下来，按 P 键，以创建主分区：

		Command action
			e   extended
			p   primary partition (1-4)
		p

- 按 1 键，以选择分区号 1：

		Partition number (1-4): 1

- 选择新分区的起始点，或者直接按 `<enter>` 键以接受默认值，将该分区置于驱动器可用空间的开始处：

		First cylinder (1-1305, default 1):
		Using default value 1

- 选择分区大小，例如，键入“+10G”以创建一个 10 GB 的分区。或者，直接按 `<enter>` 键创建一个跨整个驱动器的单一分区：

		Last cylinder, +cylinders or +size{K,M,G} (1-1305, default 1305): 
		Using default value 1305

- 接下来，将该分区的 ID 和类型从默认的 ID“83”(Linux) 更改为 ID“fd”(Linux raid auto)：

		Command (m for help): t
		Selected partition 1
		Hex code (type L to list codes): fd

- 最后，将分区表写入驱动器并退出 fdisk：

		Command (m for help): w
		The partition table has been altered!


## 创建 RAID 阵列

1. 以下示例将给位于三个不同数据磁盘（sdc1、sdd1、sde1）上的三个分区设置带区（RAID 级别 0）：

		# sudo mdadm --create /dev/md127 --level 0 --raid-devices 3 \
		  /dev/sdc1 /dev/sdd1 /dev/sde1

在此示例中，运行此命令之后，将会创建一个名为 **/dev/md127** 的新 RAID 设备。另请注意，如果这些数据磁盘以前属于另一失效的 RAID 阵列，则可能有必要将 `--force` 参数添加到 `mdadm` 命令。


2. 在新 RAID 设备上创建文件系统

	**CentOS、Oracle Linux、SLES 12、openSUSE 和 Ubuntu**

		# sudo mkfs -t ext4 /dev/md127

	**SLES 11**

		# sudo mkfs -t ext3 /dev/md127

3. **SLES 11 和 openSUSE** - 启用 boot.md 并创建 mdadm.conf

		# sudo -i chkconfig --add boot.md
		# sudo echo 'DEVICE /dev/sd*[0-9]' >> /etc/mdadm.conf

	>[AZURE.NOTE]在 SUSE 系统中完成这些更改之后，可能需要重新引导。在 SLES 12 中，此步骤*不*是必需的。


## 将新文件系统添加到 /etc/fstab

**警告：**错误地编辑 /etc/fstab 文件可能会导致系统无法引导。如果没有把握，请参考分发的文档来获取有关如何正确编辑该文件的信息。另外，建议在编辑之前创建 /etc/fstab 文件的备份。

1. 为新文件系统创建需要的装入点，例如：

		# sudo mkdir /data

2. 在编辑 /etc/fstab 时，**UUID** 应该用于引用文件系统而不是设备名称。使用 `blkid` 实用程序来确定新文件系统的 UUID：

		# sudo /sbin/blkid
		...........
		/dev/md127: UUID="aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" TYPE="ext4"

3. 在文本编辑器中打开 /etc/fstab 并为新文件系统添加新条目，例如：

		UUID=aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  /data  ext4  defaults  0  2

	或者，在 **SLES 11 和 openSUSE** 上：

		/dev/disk/by-uuid/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  /data  ext3  defaults  0  2

	然后，保存并关闭 /etc/fstab。

4. 测试该 /etc/fstab 条目是否正确：

		# sudo mount -a

	如果此命令导致错误消息，请检查 /etc/fstab 文件中的语法。

	接下来，运行 `mount` 命令以确保文件系统已装入：

		# mount
		.................
		/dev/md127 on /data type ext4 (rw)

5. （可选）故障安全引导参数

	**fstab 配置**

	许多分发版包括 `nobootwait` 或 `nofail` 装入参数，这些参数可以添加到 /etc/fstab 文件中。这些参数允许装入某特定文件系统时失败，并且允许 Linux 系统继续引导，即使它无法正确装入 RAID 文件系统也无妨。请参阅你的分发的文档，以了解有关这些参数的详细信息。

	示例 (Ubuntu)：

		UUID=aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  /data  ext4  defaults,nobootwait  0  2

	**Linux 引导参数**

	除了以上参数，还可以使用内核参数“`bootdegraded=true`”来启用系统引导功能，即使发现 RAID 已损坏或降级（例如，由于无意中从虚拟机中移除了数据驱动器而发现这种情况）也无妨。默认情况下，这样也可能会导致系统无法引导。

	请参阅你的分发的文档，了解如何正确编辑内核参数。例如，在许多分发（CentOS、Oracle Linux、SLES 11）中，可以手动将这些参数添加到“`/boot/grub/menu.lst`”文件。在 Ubuntu 上，此参数可添加到“/etc/default/grub”上的 `GRUB_CMDLINE_LINUX_DEFAULT` 变量中。

 

<!---HONumber=Mooncake_0118_2016-->