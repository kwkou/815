<properties
	pageTitle="优化 Linux VM 上的 MySQL 性能 | Azure"
	description="了解如何优化运行 Linux 的 Azure 虚拟机 (VM) 上运行的 MySQL。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="NingKuang"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="12/15/2015"
	wacn.date="01/29/2016"/>

#优化 Azure Linux VM 上的 MySQL 性能

影响 Azure 上 MySQL 性能的因素有很多，主要体现在虚拟硬件选择和软件配置两个方面。本文重点介绍如何通过存储、系统和数据库配置优化性能。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

##利用 Azure 虚拟机上的 RAID
存储是影响云环境中的数据库性能的关键因素。与单个磁盘相比，RAID 可以通过并发访问提供更快的访问速度。有关更多详细信息，请参阅[标准 RAID 级别](http://en.wikipedia.org/wiki/Standard_RAID_levels)。

通过 RAID 可大幅提升 Azure 中的磁盘 I/O 吞吐量和 I/O 响应时间。我们的检验测试表明，随着 RAID 磁盘数量的加倍（从 2 到 4，从 4 到 8，等等），磁盘 I/O 吞吐量平均增加一倍，I/O 响应时间平均缩短一半。有关详细信息，请参阅[附录 A](#AppendixA)。

增加 RAID 级别时，除了磁盘 I/O，MySQL 性能也得到改善。有关详细信息，请参阅[附录 B](#AppendixB)。

你可能还要考虑区块大小。通常，区块越大，获得的开销就越低，对于大型写入操作尤其如此。不过，区块太大时，它可能会添加额外的开销，并且你无法利用 RAID。当前的默认大小为 512KB，经证实，它是大多数常见生产环境的最佳大小。有关详细信息，请参阅[附录 C](#AppendixC)。

请注意，对于不同的虚拟机类型，可添加的磁盘数量是有限制的。[Azure 的虚拟机和云服务大小](/documentation/articles/cloud-services-sizes-specs/)中详细介绍了这些限制。你可以选择设置磁盘较少的 RAID，不过，在本文的 RAID 示例中，你需要附加 4 个数据磁盘。

本文假定你已经创建 Linux 虚拟机，并且安装和配置了 MYSQL。有关入门的详细信息，请参阅“如何在 Azure 上安装 MySQL”。

###在 Azure 上设置 RAID
以下步骤显示如何使用 Azure 经典管理门户在 Azure 上创建 RAID。你也可以使用 Windows PowerShell 脚本设置 RAID。 
在本示例中，我们将使用 4 个磁盘配置 RAID 0。

####步骤 1：向虚拟机添加数据磁盘  

在 Azure 经典管理门户的“虚拟机”页上，单击要向其添加数据磁盘的虚拟机。在本示例中，该虚拟机为 mysqlnode1。

![][1]

在该虚拟机的相应页面上，单击“仪表板”。

![][2]


在任务栏中，单击“附加”。

![][3]

然后单击“附加空磁盘”。

![][4]

对于数据磁盘，“主机缓存首选项”应设置为“无”。

这将向你的虚拟机添加一个空磁盘。再重复此步骤三次，以便为 RAID 附加 4 个数据磁盘。

通过查看内核消息日志，可以看到虚拟机中添加的驱动器。例如，若要在 Ubuntu 上查看，请使用以下命令：

	sudo grep SCSI /var/log/dmesg

####步骤 2：创建具有更多磁盘的 RAID
有关详细的 RAID 设置步骤，请参照本文：

[在 Linux 上配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/)

>[AZURE.NOTE]如果使用的是 XFS 文件系统，创建 RAID 后请按照下面的步骤操作。

若要在 Debian、Ubuntu 或 Linux Mint 上安装 XFS，请使用以下命令：

	apt-get -y install xfsprogs  

若要在 Fedora、CentOS 或 RHEL 上安装 XFS，请使用以下命令：

	yum -y install xfsprogs  xfsdump


####步骤 3：设置新的存储路径
请使用以下命令：

	root@mysqlnode1:~# mkdir -p /RAID0/mysql

####步骤 4：将原始数据复制到新的存储路径
请使用以下命令：

	root@mysqlnode1:~# cp -rp /var/lib/mysql/* /RAID0/mysql/

####步骤 5：修改权限以便 MySQL 访问（读取和写入）数据磁盘
请使用以下命令：

	root@mysqlnode1:~# chown -R mysql.mysql /RAID0/mysql && chmod -R 755 /RAID0/mysql


##调整磁盘 I/O 计划算法
Linux 实现了四种类型的 I/O 计划算法：

-	NOOP 算法（无操作）
-	截止时间算法（截止时间）
-	完全公平队列算法 (CFQ)
-	预算期算法（预测）  

在不同的情况下，你可以选择使用不同的 I/O 计划程序来优化性能。在完全随机的访问环境中，CFQ 算法和截止时间算法对性能的影响区别不大。为保持稳定性，一般建议将 MySQL 数据库环境设置为截止时间算法。如果有大量的顺序 I/O，CFQ 可能会降低磁盘 I/O 性能。

对于 SSD 和其他设备，使用 NOOP 或截止时间算法可以比默认计划程序实现更好的性能。

从内核 2.5 开始，默认 I/O 计划算法为“截止时间”。从内核 2.6.18 开始，CFQ 成为默认 I/O 计划算法。你可以在内核启动时指定此设置或者在系统运行时动态修改此设置。

下面的示例演示如何检查默认计划程序并将其设置为 NOOP 算法。

对于 Debian 分发系列：

###步骤 1.查看当前的 I/O 计划程序
请使用以下命令：

	root@mysqlnode1:~# cat /sys/block/sda/queue/scheduler

你将看到以下输出，指示当前的计划程序。

	noop [deadline] cfq


###步骤 2.更改当前设备 (/dev/sda) 的 I/O 计划算法
使用以下命令：

	azureuser@mysqlnode1:~$ sudo su -
	root@mysqlnode1:~# echo "noop" >/sys/block/sda/queue/scheduler
	root@mysqlnode1:~# sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"/g' /etc/default/grub
	root@mysqlnode1:~# update-grub

>[AZURE.NOTE]对 /dev/sda 单独进行此设置毫无用处。需对数据库所在的所有数据磁盘进行设置。

你应该会看到以下输出，指示已成功重新生成 grub.cfg 并且默认计划程序已更新为 NOOP。

	Generating grub configuration file ...
	Found linux image: /boot/vmlinuz-3.13.0-34-generic
	Found initrd image: /boot/initrd.img-3.13.0-34-generic
	Found linux image: /boot/vmlinuz-3.13.0-32-generic
	Found initrd image: /boot/initrd.img-3.13.0-32-generic
	Found memtest86+ image: /memtest86+.elf
	Found memtest86+ image: /memtest86+.bin
	done

对于 Redhat 分发系列，只需以下命令：

	echo 'echo noop >/sys/block/sda/queue/scheduler' >> /etc/rc.local

##配置系统文件操作设置
最佳做法之一是禁用文件系统上的 atime 日志记录功能。Atime 指最后一次文件访问的时间。无论何时访问文件，文件系统都会在日志中记录时间戳。但是，很少使用此信息。如果不需要，可以禁用它，这样将减少总体磁盘访问时间。

若要禁用 atime 日志记录，需修改文件系统配置文件 /etc/ fstab 并添加“noatime”选项。

例如，编辑 vim /etc/fstab 文件，并按下面所示添加 noatime。

	# CLOUD_IMG: This file was created/modified by the Cloud Image build process
	UUID=3cc98c06-d649-432d-81df-6dcd2a584d41       /        ext4   defaults,discard        0 0
	#Add the “noatime” option below to disable atime logging
	UUID="431b1e78-8226-43ec-9460-514a9adf060e"     /RAID0   xfs   defaults,nobootwait, noatime 0 0
	/dev/sdb1       /mnt    auto    defaults,nobootwait,comment=cloudconfig 0       2

然后，使用以下命令重新装载文件系统：

	mount -o remount /RAID0

测试修改后的结果。请注意，当你修改测试文件时，系统不会更新访问时间。

之前的示例：

![][5]

之后的示例：

![][6]

##增加系统句柄的最大数量以实现高并发
MySQL 是高并发数据库。对于 Linux，默认的并发句柄数量是 1024 个，这个数量有时候不够用。**通过执行以下步骤来增加系统的最大并发句柄数，以此支持 MySQL 的高并发**。

###步骤 1：修改 limits.conf 文件
在 /etc/security/limits.conf 文件中添加以下四行，以增加允许的最大并发句柄数。请注意，65536 是系统可以支持的最大数量。

	* soft nofile 65536
	* hard nofile 65536
	* soft nproc 65536
	* hard nproc 65536

###步骤 2：更新系统的新限制
运行以下命令：

	ulimit -SHn 65536
	ulimit -SHu 65536

###步骤 3：确保在启动时更新限制
在 /etc/rc.local 文件中放入以下启动命令，以便其在每次启动时生效。

	echo “ulimit -SHn 65536” >>/etc/rc.local
	echo “ulimit -SHu 65536” >>/etc/rc.local

##MySQL 数据库优化
可以在 Azure 上使用与在本地计算机上相同的性能优化策略来配置 MySQL。

主要 I/O 优化规则包括：

-	增加缓存大小。
-	减少 I/O 响应时间。  

若要优化 MySQL 服务器设置，可以更新 my.cnf 文件，它是服务器和客户端计算机的默认配置文件。

以下配置项是影响 MySQL 性能的主要因素：

-	**innodb\_buffer\_pool\_size**：缓冲池包含缓冲数据和索引。此值通常设置为物理内存的 70%。
-	**innodb\_log\_file\_size**：这是重做日志大小。重做日志用于确保写入操作快速、可靠并且可在出现故障后恢复。此值设置为 512MB，这将为记录写入操作提供大量空间。
-	**max\_connections**：应用程序有时候不会正确关闭连接。值越大，服务器就有越多时间回收空闲的连接。最大连接数为 10000，但建议的最大值为 5000。
-	**Innodb\_file\_per\_table**：此设置允许或禁止 InnoDB 将表存储在单独的文件中。启用该选项将确保可以有效地应用多项高级管理操作。从性能角度来看，它可以提高表空间传输的速度和优化碎片管理性能。因此，此选项的推荐设置是“打开”。</br> 从 MySQL 5.6 开始，默认设置为“打开”。因此，无需执行任何操作。对于早于 5.6 的其他版本，默认设置为“关闭”。必须启用此选项，并在加载数据之前应用此选项，因为只有新创建的表才会受影响。
-	**innodb\_flush\_log\_at\_trx\_commit**：默认值为 1，范围设置为 0~2。默认值是最适合独立 MySQL DB 的选项。设置 2 支持最大程度的数据完整性，适用于 MySQL 群集中的主节点。设置 0 允许数据丢失，这可能会影响可靠性，但在某些情况下能提供更好的性能，适用于 MySQL 群集中的从属节点。
-	**Innodb\_log\_buffer\_size**：借助日志缓冲区，即使在事务提交之前未将日志刷新到磁盘，事务也可以运行。但是，如果有大型的二进制对象或文本字段，将很快地耗尽缓存，并触发频繁的磁盘 I/O。如果 Innodb\_log\_waits 状态变量不为 0，最好增加缓冲区大小。
-	**query\_cache\_size**：最好是从一开始就禁用。将 query\_cache\_size 设置为 0（现在是 MySQL 5.6 中的默认设置）并使用其他方法来提高查询速度。  

有关优化后的性能比较，请参阅[附录 D](#AppendixD)。


##启用 MySQL 慢查询日志进行性能瓶颈分析
MySQL 慢查询日志有助于识别 MySQL 的慢查询。在启用 MySQL 慢查询日志后，可以使用 **mysqldumpslow** 等 MySQL 工具来识别性能瓶颈。

请注意，默认情况下不启用此选项。启用慢查询日志可能会占用一些 CPU 资源。因此，建议临时启用此选项来排除性能瓶颈。

###步骤 1：通过在末尾添加以下行来修改 my.cnf 文件   

	long_query_time = 2
	slow_query_log = 1
	slow_query_log_file = /RAID0/mysql/mysql-slow.log

###步骤 2：重新启动 mysql 服务器
	service  mysql  restart

###步骤 3：使用“show”命令检查该设置是否生效

![][7]

![][8]

在本示例中，可以看到慢查询功能已启用。然后可以使用 **mysqldumpslow** 工具来确定性能瓶颈和优化性能，比如添加索引。





##附录

以下是在目标实验室环境中生成的示例性能测试数据，它们提供了有关使用不同性能优化方法的性能数据趋势的常规背景信息，但是，不同环境或产品版本下的结果可能会有所不同。

<a name="AppendixA"></a>附录 A：  
**不同 RAID 级别的磁盘性能 (IOPS)**


![][9]

**测试命令：**

	fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=5G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite

>AZURE。请注意：此测试的工作负荷使用 64 个线程，并尝试达到 RAID 的上限。

<a name="AppendixB"></a>附录 B：  
**不同 RAID 级别的 MySQL 性能（吞吐量）比较**   
（XFS 文件系统）


![][10]  
![][11]

**测试命令：**

	mysqlslap -p0ps.123 --concurrency=2 --iterations=1 --number-int-cols=10 --number-char-cols=10 -a --auto-generate-sql-guid-primary --number-of-queries=10000 --auto-generate-sql-load-type=write –engine=innodb

**不同 RAID 级别的 MySQL 性能 (OLTP) 比较**  
![][12]

**测试命令：**

	time sysbench --test=oltp --db-driver=mysql --mysql-user=root --mysql-password=0ps.123  --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-socket=/var/run/mysqld/mysqld.sock --mysql-db=test --oltp-table-size=1000000 prepare

<a name="AppendixC"></a>附录 C：   
**不同区块大小的磁盘性能 (IOPS) 比较**  
（XFS 文件系统）


![][13]

**测试命令：**

	fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=30G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite
	fio -filename=/path/test -iodepth=64 -ioengine=libaio -direct=1 -rw=randwrite -bs=4k -size=1G -numjobs=64 -runtime=30 -group_reporting -name=test-randwrite  

请注意，用于此测试的文件大小分别为 30GB 和 1GB，并且使用的是 RAID 0（4 个磁盘）XFS 文件系统。


<a name="AppendixD"></a>附录 D：  
**优化前和优化后的 MySQL 性能（吞吐量）比较**  
（XFS 文件系统）


![][14]

**测试命令：**

	mysqlslap -p0ps.123 --concurrency=2 --iterations=1 --number-int-cols=10 --number-char-cols=10 -a --auto-generate-sql-guid-primary --number-of-queries=10000 --auto-generate-sql-load-type=write –engine=innodb,misam

**默认值和优化值的配置设置如下所示：**

|Parameters |默认 |优化值
|-----------|-----------|-----------
|**innodb\_buffer\_pool\_size** |无 |7G
|**innodb\_log\_file\_size** |5M |512M
|**max\_connections** |100 |5000
|**innodb\_file\_per\_table** |0 |1
|**innodb\_flush\_log\_at\_trx\_commit** |1 |2
|**innodb\_log\_buffer\_size** |8M |128M
|**query\_cache\_size** |16M |0


有关更多更详细的优化配置参数，请参阅 mysql 正式说明。

[http://dev.mysql.com/doc/refman/5.6/en/innodb-configuration.html](http://dev.mysql.com/doc/refman/5.6/en/innodb-configuration.html)

[http://dev.mysql.com/doc/refman/5.6/en/innodb-parameters.html#sysvar\_innodb\_flush\_method](http://dev.mysql.com/doc/refman/5.6/en/innodb-parameters.html#sysvar_innodb_flush_method)

**测试环境**

|硬件 |详细信息
|-----------|-------
|Cpu |AMD Opteron(tm) 处理器 4171 HE/4 核
|内存 |14G
|disk |10G/磁盘
|os |Ubuntu 14.04.1 LTS



[1]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-01.png
[2]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-02.png
[3]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-03.png
[4]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-04.png
[5]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-05.png
[6]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-06.png
[7]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-07.png
[8]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-08.png
[9]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-09.png
[10]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-10.png
[11]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-11.png
[12]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-12.png
[13]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-13.png
[14]: ./media/virtual-machines-linux-classic-optimize-mysql/virtual-machines-linux-optimize-mysql-perf-14.png

<!---HONumber=Mooncake_0118_2016-->