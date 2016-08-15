<properties 
	pageTitle="在运行 Linux 的虚拟机上配置 LVM | Azure" 
	description="了解如何在 Azure 中的 Linux 上配置 LVM。" 
	services="virtual-machines-linux" 
	documentationCenter="na" 
	authors="szarkos"  
	manager="timlt" 
	editor="tysonn"
	tag="azure-service-management,azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/06/2016"
	wacn.date="06/13/2016"/>


# 在 Azure 中的 Linux VM 上配置 LVM

本文介绍如何在 Azure 虚拟机中配置逻辑卷管理器 (LVM)。尽管可以在任何连接到虚拟机的磁盘上配置 LVM，但默认情况下，大多数云映像不会在 OS 磁盘上配置 LVM。这是为了防止重复卷组相关的问题，因为 OS 磁盘可能曾经连接到相同分发版和类型的 VM（例如在执行恢复方案期间）。因此建议只在数据磁盘上使用 LVM。


## 线性与条带化逻辑卷

LVM 可用于将多个物理磁盘合并成单个存储卷。默认情况下，LVM 通常会创建线性逻辑卷，这意味着，物理存储是串连在一起的。在此情况下，读取/写入操作通常只发送到单个磁盘。相比之下，我们也可以创建条带化逻辑卷，其中的读取和写入将分布到卷组（类似于 RAID0）中的多个磁盘。出于性能考虑，你可能希望创建条带化逻辑卷，以便读取和写入操作利用所有附加的数据磁盘。

本文档介绍如何将多个数据磁盘合并成单个卷组，然后创建条带化逻辑卷。下面是通用化的步骤，适用于大多数分发版。在大多数情况下，Azure 上用于管理 LVM 的实用工具和工作流与其他环境中的基本上相同。像往常一样，另请咨询 Linux 供应商配合特定分发版使用 LVM 的文档和最佳实践。


## 附加数据磁盘
使用 LVM 时，通常一开始用二个或更多的空数据磁盘。根据 IO 需求，可以选择附加存储在标准存储且一个磁盘最多具有 500 IO/ps 的磁盘，或高级存储且一个磁盘最多具有 5000 IO/ps 的磁盘。本文将不详细介绍如何为 Linux 虚拟机预配和附加数据磁盘。请参阅 Azure 文章[附加磁盘](/documentation/articles/virtual-machines-linux-add-disk/)，以详细了解如何在 Azure 上为 Linux 虚拟机附加空数据磁盘。

## 安装 LVM 实用工具

- **Ubuntu**

		# sudo apt-get update
		# sudo apt-get install lvm2

- **RHEL、CentOS 和 Oracle Linux**

		# sudo yum install lvm2

- **SLES 12 和 openSUSE**

		# sudo zypper install lvm2

- **SLES 11**

		# sudo zypper install lvm2

	在 SLES11 上，还必须编辑 /etc/sysconfig/lvm 并将 `LVM_ACTIVATED_ON_DISCOVERED` 设置为 "enable"：

		LVM_ACTIVATED_ON_DISCOVERED="enable" 


## 配置 LVM
本指南假设已附加三个数据磁盘，分别为 `/dev/sdc`、`/dev/sdd` 和 `/dev/sde`。请注意，VM 中的路径名称不一定与上述相同。可以运行 `sudo fdisk -l` 或类似命令以列出可用的磁盘。

1. 准备物理卷：

		# sudo pvcreate /dev/sd[cde]
		  Physical volume "/dev/sdc" successfully created
		  Physical volume "/dev/sdd" successfully created
		  Physical volume "/dev/sde" successfully created


2.  创建卷组。在本例中，我们将调用卷组“data-vg01”：

		# sudo vgcreate data-vg01 /dev/sd[cde]
		  Volume group "data-vg01" successfully created


3. 创建一个或多个逻辑卷。以下命令将创建跨整个卷组的名为“data-lv01”的单个逻辑卷，但请注意，在卷组中创建多个逻辑卷也是可行的。

		# sudo lvcreate --extents 100%FREE --stripes 3 --name data-lv01 data-vg01
		  Logical volume "data-lv01" created.


4. 格式化逻辑卷

		# sudo mkfs -t ext4 /dev/data-vg01/data-lv01

  >[AZURE.NOTE] 在 SLES11 上，请使用 "-t ext3" 而不是 ext4。SLES11 仅支持对 ext4 文件系统进行只读访问。


## 将新文件系统添加到 /etc/fstab

**警告：**错误地编辑 /etc/fstab 文件可能会导致系统无法引导。如果没有把握，请参考分发的文档来获取有关如何正确编辑该文件的信息。另外，建议在编辑之前创建 /etc/fstab 文件的备份。

1. 为新文件系统创建需要的装入点，例如：

		# sudo mkdir /data


2. 查找逻辑卷路径

		# lvdisplay
		--- Logical volume ---
		LV Path                /dev/data-vg01/data-lv01
		....


3. 在文本编辑器中打开 /etc/fstab 并为新文件系统添加新条目，例如：

		/dev/data-vg01/data-lv01  /data  ext4  defaults  0  2

	然后，保存并关闭 /etc/fstab。


4. 测试该 /etc/fstab 条目是否正确：

		# sudo mount -a

	如果此命令导致错误消息，请检查 /etc/fstab 文件中的语法。

	接下来，运行 `mount` 命令以确保文件系统已装入：

		# mount
		......
		/dev/mapper/data--vg01-data--lv01 on /data type ext4 (rw)


5. （可选）/etc/fstab 中的防故障引导参数

	许多分发版包括 `nobootwait` 或 `nofail` 装入参数，这些参数可以添加到 /etc/fstab 文件中。这些参数允许装入某特定文件系统时失败，并且允许 Linux 系统继续引导，即使它无法正确装入 RAID 文件系统也无妨。请参阅你的分发的文档，以了解有关这些参数的详细信息。

	示例 (Ubuntu)：

		/dev/data-vg01/data-lv01  /data  ext4  defaults,nobootwait  0  2

<!---HONumber=Mooncake_0606_2016-->