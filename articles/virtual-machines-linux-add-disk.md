<!-- ARM: tested -->

<properties
	pageTitle="将磁盘添加到 Linux VM | Azure"
	description="了解如何将持久性磁盘添加到 Linux VM"
	keywords="linux 虚拟机,添加资源磁盘"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="rickstercdn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/29/2016"
	wacn.date="06/27/2016"/>

# 将磁盘添加到 Linux VM

本文介绍如何将持久性磁盘附加到 VM 以保存数据 - 即使 VM 由于维护或调整大小而重新预配。若要添加磁盘，需要在 Resource Manager 模式下配置 [Azure CLI](/documentation/articles/xplat-cli-install/) (`azure config mode arm`)。

## 快速命令

在以下命令示例中，请将 &lt; 与 &gt; 之间的值替换为你自己环境中的值。
	
	azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>

## 附加磁盘

连接新的磁盘很快。键入 `azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>` 即可为 VM 创建和连接新的 GB 磁盘。如果你未显式标识存储帐户，则创建的任何磁盘将位于 OS 磁盘所在的同一个存储帐户中。你应该会看到类似下面的屏幕：

	azure vm disk attach-new myuniquegroupname myuniquevmname 5

输出

	info:    Executing command vm disk attach-new
	+ Looking up the VM "myuniquevmname"
	info:    New data disk location: https://cliexxx.blob.core.chinacloudapi.cn/vhds/myuniquevmname-20150526-0xxxxxxx43.vhd
	+ Updating VM "myuniquevmname"
	info:    vm disk attach-new command OK


## 连接到 Linux VM 以装入新磁盘

> [AZURE.NOTE] 本主题介绍如何使用用户名和密码连接到 VM；若要使用公钥和私钥对与 VM 通信，请参阅[如何在 Azure 上通过 Linux 使用 SSH](/documentation/articles/virtual-machines-linux-ssh-from-linux/)。你可以通过使用 `azure vm reset-access` 命令完全重置 **SSH** 访问权限、添加或删除用户，或者添加公钥文件以确保安全访问，来修改使用 `azure vm quick-create` 命令创建的 VM 的 **SSH** 连接。

需要使用 SSH 访问 Azure VM 才能分区、格式化和装入新磁盘以供 Linux VM 使用。如果你不熟悉如何使用 **ssh** 进行连接，请注意，该命令采用 `ssh <username>@<FQDNofAzureVM> -p <the ssh port>` 格式，如下所示：

	ssh ops@myuni-china-1432328437727-pip.chinanorth.chinacloudapp.cn -p 22

输出

	The authenticity of host 'myuni-china-1432328437727-pip.chinanorth.chinacloudapp.cn (191.239.51.1)' can't be established.
	ECDSA key fingerprint is bx:xx:xx:xx:xx:xx:xx:xx:xx:x:x:x:x:x:x:xx.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'myuni-china-1432328437727-pip.chinanorth.chinacloudapp.cn,191.239.51.1' (ECDSA) to the list of known hosts.
	ops@myuni-china-1432328437727-pip.chinanorth.chinacloudapp.cn's password:
	Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

	 * Documentation:  https://help.ubuntu.com/

	  System information as of Fri May 22 21:02:32 UTC 2015

	  System load: 0.37              Memory usage: 2%   Processes:       207
	  Usage of /:  41.4% of 1.94GB   Swap usage:   0%   Users logged in: 0

	  Graph this data and manage this system at:
	    https://landscape.canonical.com/

	  Get cloud support with Ubuntu Advantage Cloud Guest:
	    http://www.ubuntu.com/business/services/cloud

	0 packages can be updated.
	0 updates are security updates.

	The programs included with the Ubuntu system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.

	Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
	applicable law.

	ops@myuniquevmname:~$

现在，你已连接到 VM，可以连接磁盘了。首先，使用 `dmesg | grep SCSI` 来查找磁盘（用于发现新磁盘的方法可能各不相同）。在本示例中，键入的内容如下所示：

	dmesg | grep SCSI

输出

	[    0.294784] SCSI subsystem initialized
	[    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
	[    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
	[    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
	[ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk

而在本主题中，`sdc` 磁盘是我们所需要的。现在使用 `sudo fdisk /dev/sdc` 对磁盘进行分区 -- 假定在你的示例中，磁盘为 `sdc`，则应将其设置为分区 1 中的主磁盘，并接受其他默认设置值。

	sudo fdisk /dev/sdc

输出

	Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
	Building a new DOS disklabel with disk identifier 0x2a59b123.
	Changes will remain in memory only, until you decide to write them.
	After that, of course, the previous content won't be recoverable.

	Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

	Command (m for help): n
	Partition type:
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended
	Select (default p): p
	Partition number (1-4, default 1): 1
	First sector (2048-10485759, default 2048):
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
	Using default value 10485759

系统提示时，键入 `p` 即可创建分区：

	Command (m for help): p

	Disk /dev/sdc: 5368 MB, 5368709120 bytes
	255 heads, 63 sectors/track, 652 cylinders, total 10485760 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x2a59b123

	   Device Boot      Start         End      Blocks   Id  System
	/dev/sdc1            2048    10485759     5241856   83  Linux

	Command (m for help): w
	The partition table has been altered!

	Calling ioctl() to re-read partition table.
	Syncing disks.

同时，还需将文件系统写入分区，只需使用 **mkfs** 命令指定文件系统类型和设备名称即可。在本主题中，我们将使用上面提供的 `ext4` 和 `/dev/sdc1`：

	sudo mkfs -t ext4 /dev/sdc1

输出

	mke2fs 1.42.9 (4-Feb-2014)
	Discarding device blocks: done
	Filesystem label=
	OS type: Linux
	Block size=4096 (log=2)
	Fragment size=4096 (log=2)
	Stride=0 blocks, Stripe width=0 blocks
	327680 inodes, 1310464 blocks
	65523 blocks (5.00%) reserved for the super user
	First data block=0
	Maximum filesystem blocks=1342177280
	40 block groups
	32768 blocks per group, 32768 fragments per group
	8192 inodes per group
	Superblock backups stored on blocks:
		32768, 98304, 163840, 229376, 294912, 819200, 884736
	Allocating group tables: done
	Writing inode tables: done
	Creating journal (32768 blocks): done
	Writing superblocks and filesystem accounting information: done

现在，我们使用 `mkdir` 创建一个目录来装载文件系统：

	sudo mkdir /datadrive

可使用 `mount` 来装载目录：

	sudo mount /dev/sdc1 /datadrive

数据磁盘现在可以作为 `/datadrive` 使用。

	ls

输出

	bin   datadrive  etc   initrd.img  lib64       media  opt   root  sbin  sys  usr  vmlinuz
	boot  dev        home  lib         lost+found  mnt    proc  run   srv   tmp  var

> [AZURE.NOTE] 你还可以在连接到 Linux 虚拟机时，使用 SSH 密钥进行身份验证。有关详细信息，请参阅[如何在 Azure 上将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-ssh-from-linux/)。

## 后续步骤

- 请记住，即使重新启动 VM，你的新磁盘通常也无法供 VM 使用，除非你将该信息写入 [fstab](http://en.wikipedia.org/wiki/Fstab) 文件。
- 请查看[优化 Linux 计算机性能](/documentation/articles/virtual-machines-linux-optimization/)的建议，以确保 Linux VM 正确配置。
- 通过添加更多的磁盘来扩展存储容量，并[配置 RAID](/documentation/articles/virtual-machines-linux-configure-raid/) 以提高性能。

<!---HONumber=Mooncake_0620_2016-->