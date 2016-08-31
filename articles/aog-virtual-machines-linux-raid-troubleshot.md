<properties 
	pageTitle="Azure Linux虚机上配置RAID的常见问题及解决方案" 
	description="独立硬盘冗余阵列（RAID, Redundant Array of Independent Disks），简称磁盘阵列。能增强数据集成度，增强容错功能，增加处理量或容量。" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machines-aog" ms.date="" wacn.date="08/31/2016"/>
#Azure Linux虚机上配置RAID的常见问题及解决方案

##简介

独立硬盘冗余阵列（RAID, Redundant Array of Independent Disks），简称磁盘阵列。能增强数据集成度，增强容错功能，增加处理量或容量。详情参见[这篇文章](https://zh.wikipedia.org/zh-cn/RAID)。

##配置方法

> 注: 以下范例均在CentOS平台运行,其他版本Linux略有差异, 请注意区别。

1.	在Azure平台的Linux虚拟机上添加至少2块空磁盘。
2.	以管理员身份登录Linux虚机并切换至root用户。
3.	安装mdadm工具。
	`# yum install mdadm`
4.	查看磁盘及分区。

		# fdisk  -l |grep -i "Disk /dev/"
		Disk /dev/sdb: 145.0 GB, 144955146240 bytes
		Disk /dev/sda: 32.2 GB, 32212254720 bytes
		Disk /dev/sdc: 1073 MB, 1073741824 bytes
		Disk /dev/sdd: 1073 MB, 1073741824 bytes

5.	创建RAID。

		# mdadm --create /dev/md0 --level 0 --raid-devices 2 /dev/sdc /dev/sdd
		mdadm: Defaulting to version 1.2 metadata
		mdadm: array /dev/md0 started.

6.	基于RAID,创建文件系统。

		# mkfs.ext4 /dev/md0

7.	添加新文件系统到/etc/fstab。
	
		# mkdir /data
		# blkid  |grep -i md0
		/dev/md0: UUID="21424152-440e-42f5-b8fc-07ded5a0bea4" TYPE="ext4"
		# echo "UUID=21424152-440e-42f5-b8fc-07ded5a0bea4 /data ext4 defaults 0 2 " >> /etc/fstab
		# mount -a
		# df -h |grep -i data
		/dev/md0        2.0G   35M  1.9G   2% /data


## 常见问题及解决

1.	**问题**:是否可以把临时盘(默认/dev/sdb)加入RAID中?
	
	**答**:不可以, 因为临时盘每次重启都会清空数据。

2.	**问题**:系统默认会启用RAID的每周自检,如何调整执行时间或者关闭自检?

	**答**:编辑定时任务脚本/etc/cron.d/raid-check, 修改执行时间。默认如下:

		# cat /etc/cron.d/raid-check
		# Run system wide raid-check once a week on Sunday at 1am by default
		0 1 * * Sun root /usr/sbin/raid-check

	编辑自检脚本/etc/sysconfig/raid-check 将ENABLED=yes行改成ENABLED=no 来关闭自检。


