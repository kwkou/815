<properties
	pageTitle="如何在 Linux 虚拟机上扩展根文件系统"
	description="如何在 Linux 虚拟机上扩展根文件系统"
	service=""
	resource="virtualmachines"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Virtual Machines, Linux, Root System File, GNOME, VNC"
    cloudEnvironments="MoonCake" />
<tags
	ms.service="virtual-machines-linux-aog"
	ms.date=""
	wacn.date="1/20/2017" />
# 如何在 Linux 虚拟机上扩展根文件系统

## **问题描述**

通过 Azure 平台部署的 Linux 虚拟机默认的根文件系统容量有限，需要进行扩展。

## **问题分析**

由于 Azure 平台部署的 Linux 虚拟机默认根文件系统容量比较小，客户在使用过程中，经常会出现根文件系统用满，导致虚拟机不可用的情况，需要进行手动对根文件系统进行扩容。

## **解决方案**

>[AZURE.NOTE]在执行如下操作前，一定要针对虚拟机的系统盘进行备份。以下步骤基于 CentOS 6.8，其他 Linux 版本，可能会略有区别。

1.	通过 Azure portal 关闭虚拟机。
2.	执行以下 Powershell 命令，对系统 盘进行扩展：

		Get-AzureVM -ServiceName "vfldev" -Name "vfldev" | get-AzureOSDisk 

		## 使用正确的 ServiceName 和 VM Name 取代上述参数。

		Update-AzureDisk –DiskName "vfldev-vfldev-0-201503091934500547" -Label "ResiZedOS" -ResizedSizeInGB 100

		## 用步骤一获取的 OSdisk 的名字取代上述的 DiskName，并输入想要扩容的磁盘大小。

3.	通过 Azure portal 启动虚拟机。
4.	登陆虚拟机，切换成 root 用户，查看当前的虚拟机的根文件系统容量。

		[root@resizeSDA chpaadmin]# df -h
		Filesystem      Size  Used Avail Use% Mounted on
		/dev/sda1        30G  1.1G   27G   4% /
		devtmpfs        832M     0  832M   0% /dev
		tmpfs           840M     0  840M   0% /dev/shm
		tmpfs           840M  8.3M  832M   1% /run
		tmpfs           840M     0  840M   0% /sys/fs/cgroup
		/dev/sdb1        69G   53M   66G   1% /mnt/resource

5.	打开分区表

		[root@resizeSDA chpaadmin]# fdisk /dev/sda
		Welcome to fdisk (util-linux 2.23.2).
		
		Changes will remain in memory only, until you decide to write them.
		Be careful before using the write command.
		
		
		Command (m for help): p
		
		Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
		Units = sectors of 1 * 512 = 512 bytes
		Sector size (logical/physical): 512 bytes / 512 bytes
		I/O size (minimum/optimal): 512 bytes / 512 bytes
		Disk label type: dos
		Disk identifier: 0x00093e4e
		## 请记录分区信息
		   Device Boot      Start         End      Blocks   Id  System
		/dev/sda1   *        2048    62914559    31456256   83  Linux
		
		## 删除分区
		Command (m for help): d
		Selected partition 1
		Partition 1 is deleted
		
		## 新建分区
		Command (m for help): n
		Partition type:
		   p   primary (0 primary, 0 extended, 4 free)
		   e   extended
		Select (default p): p
		Partition number (1-4, default 1):
		First sector (2048-209715199, default 2048):
		Using default value 2048
		Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199):
		Using default value 209715199
		Partition 1 of type Linux and of size 100 GiB is set
		
		## 此时修改分区结束，打印分区信息，确认信息无误
		Command (m for help): p
		
		Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
		Units = sectors of 1 * 512 = 512 bytes
		Sector size (logical/physical): 512 bytes / 512 bytes
		I/O size (minimum/optimal): 512 bytes / 512 bytes
		Disk label type: dos
		Disk identifier: 0x00093e4e
		
		## 注意，这里的start的值，必须和此前的分区表里的信息一致
		   Device Boot      Start         End      Blocks   Id  System
		/dev/sda1            2048   209715199   104856576   83  Linux
		
		##激活分区
		Command (m for help): a
		Selected partition 1
		
		## 再次打印分区，确认已激活
		Command (m for help): p
		
		Disk /dev/sda: 107.4 GB, 107374182400 bytes, 209715200 sectors
		Units = sectors of 1 * 512 = 512 bytes
		Sector size (logical/physical): 512 bytes / 512 bytes
		I/O size (minimum/optimal): 512 bytes / 512 bytes
		Disk label type: dos
		Disk identifier: 0x00093e4e
		
		   Device Boot      Start         End      Blocks   Id  System
		/dev/sda1   *        2048   209715199   104856576   83  Linux
		
		## 如果信息有误，或者不确定，请及时联系我们，如果信息确认无误，写入分区表
		Command (m for help): wr
		The partition table has been altered!
		
		Calling ioctl() to re-read partition table.
		
		WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
		The kernel still uses the old table. The new table will be used at
		the next reboot or after you run partprobe(8) or kpartx(8)
		Syncing disks.
		
	>[AZURE.NOTE]在 CentOS 7.x 中，可能会存在 /dev/sda1 和 /dev/sda2 分区，在进行上述步骤时，仅需要扩展 /dev/sda2 分区，且无需激活。

6.	分区表修改完毕，重启虚拟机。

		[root@resizeSDA chpaadmin]# init 6

7.	登陆虚拟机，切换到 root 用户，检查当前根文件系统的容量。
		
		[root@resizeSDA chpaadmin]# df -h
		Filesystem      Size  Used Avail Use% Mounted on
		/dev/sda1        30G  1.1G   27G   4% /
		devtmpfs        832M     0  832M   0% /dev
		tmpfs           840M     0  840M   0% /dev/shm
		tmpfs           840M  8.3M  832M   1% /run
		tmpfs           840M     0  840M   0% /sys/fs/cgroup
		/dev/sdb1        69G   53M   66G   1% /mnt/resource

8.	修改根文件系统的大小。

		[root@resizeSDA chpaadmin]# resize2fs /dev/sda1
		resize2fs 1.42.9 (28-Dec-2013)
		Filesystem at /dev/sda1 is mounted on /; on-line resizing required
		old_desc_blocks = 4, new_desc_blocks = 13
		The filesystem on /dev/sda1 is now 26214144 blocks long.

	>[AZURE.NOTE]在 CentOS 7.x 中，resize2fs 命令被 xfs 命令取代，请使用 xfs_growfs 命令扩展分区。

9.	检查根文件系统大小。

		[root@resizeSDA chpaadmin]# df -h
		Filesystem      Size  Used Avail Use% Mounted on
		/dev/sda1        99G  1.1G   93G   2% /
		devtmpfs        832M     0  832M   0% /dev
		tmpfs           840M     0  840M   0% /dev/shm
		tmpfs           840M  8.3M  832M   1% /run
		tmpfs           840M     0  840M   0% /sys/fs/cgroup
		/dev/sdb1        69G   53M   66G   1% /mnt/resource

10.	至此，根文件系统扩容完毕。

