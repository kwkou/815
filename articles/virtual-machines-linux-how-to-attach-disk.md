<properties
	pageTitle="在 Azure 中将磁盘附加到运行 Linux 的虚拟机"
	description="了解如何将数据磁盘附加到 Azure 虚拟机并将其初始化，以便它可供使用。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="08/11/2015"
	wacn.date="09/18/2015"/>

# 如何将数据磁盘附加到 Linux 虚拟机

你可以附加空磁盘和包含数据的磁盘。在这两种情况下，这些磁盘实际上是驻留在 Azure 存储帐户中的 .vhd 文件。此外，也是在这两种情况下，在附加磁盘之后，你将需要对其进行初始化，然后才能使用。本文所指的虚拟机是使用经典部署模型创建的虚拟机。

> [AZURE.NOTE]最佳做法是使用一个或多个不同的磁盘来存储虚拟机的数据。当你创建 Azure 虚拟机时，该虚拟机有一个操作系统磁盘和一个临时磁盘。**不要使用临时磁盘来存储数据。** 顾名思义，它仅提供临时存储。它不提供冗余或备份，因为它不驻留在 Azure 存储空间中。临时磁盘通常由 Azure Linux 代理管理并且自动装载到 **/mnt/resource**（或 Ubuntu 映像上的 **/mnt**）。另一方面，数据磁盘可以由 Linux 内核命名为 `/dev/sdc` 这样的形式，而用户则需对该资源进行分区、格式化和安装。有关详细信息，请参阅 [Azure Linux 代理用户指南][Agent]。

[AZURE.INCLUDE [howto-attach-disk-windows-linux](../includes/howto-attach-disk-linux.md)]

## 如何：在 Linux 中初始化新的数据磁盘

1. 连接到虚拟机。有关说明，请参阅[如何登录到运行 Linux 的虚拟机][Logon]。



2. 接下来，你需要查找可供数据磁盘初始化的设备标识符。可通过两种方式实现该目的：

	a) 在 SSH 窗口中，键入下面的命令，然后输入为管理虚拟机而创建的帐户的密码：

			$sudo grep SCSI /var/log/messages

	如果是最近的 Ubuntu 分发，你可能需要使用 `sudo grep SCSI /var/log/syslog`，因为默认情况下可能已禁止登录到 `/var/log/messages`。

	您可以在所示消息中找到最后添加的数据磁盘的标识符。

	![获取磁盘消息](./media/virtual-machines-linux-how-to-attach-disk/DiskMessages.png)

	或者

	b) 使用 `lsscsi` 命令找出设备 ID。`lsscsi` 的安装可以通过 `yum install lsscsi`（在基于 Red Hat 的分发上）或 `apt-get install lsscsi`（在基于 Debian 的分发上）来进行。你可以通过 _lun_（即**逻辑单元号**）找到所要的磁盘。例如，所附加磁盘的 _lun_ 可以轻松地通过 `azure vm disk list <virtual-machine>` 来查看，如下所示：

			~$ azure vm disk list ubuntuVMasm
			info:    Executing command vm disk list
			+ Fetching disk images
			+ Getting virtual machines
			+ Getting VM disks
			data:    Lun  Size(GB)  Blob-Name                         OS
			data:    ---  --------  --------------------------------  -----
			data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
			data:    1    10        test.VHD
			data:    2    30        ubuntuVMasm-76f7ee1ef0f6dddc.vhd
			info:    vm disk list command OK

	将此结果与同一示例性虚拟机的 `lsscsi` 的输出进行比较：

			adminuser@ubuntuVMasm:~$ lsscsi
			[1:0:0:0]    cd/dvd  Msft     Virtual CD/ROM   1.0   /dev/sr0
			[2:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sda
			[3:0:1:0]    disk    Msft     Virtual Disk     1.0   /dev/sdb
			[5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
			[5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd
			[5:0:0:2]    disk    Msft     Virtual Disk     1.0   /dev/sde

	每一行的元组中的最后一个数字就是 _lun_。有关详细信息，请参阅 `man lsscsi`。

3. 在 SSH 窗口中，键入下面的命令以新建设备，然后输入帐户密码：

		$sudo fdisk /dev/sdc

	>[AZURE.NOTE]此示例中，如果 /sbin 或 /usr/sbin 不在你的 `$PATH` 中，你在某些分发上可能需要使用 `sudo -i`。


4. 出现提示时，键入 **n** 以创建新分区。


	![新建设备](./media/virtual-machines-linux-how-to-attach-disk/DiskPartition.png)

5. 出现提示时，键入 **p** 将该分区设置为主分区，键入 **1** 将其设置为第一分区，然后键入 Enter 以接受柱面的默认值。


	![创建分区](./media/virtual-machines-linux-how-to-attach-disk/DiskCylinder.png)



6. 键入 **p** 以查看有关分区磁盘的详细信息。


	![列出磁盘信息](./media/virtual-machines-linux-how-to-attach-disk/DiskInfo.png)



7. 键入 **w** 以写入磁盘的设置。


	![写入磁盘更改](./media/virtual-machines-linux-how-to-attach-disk/DiskWrite.png)

8. 在新分区上创建文件系统。例如，可键入以下命令，然后输入帐户密码：

		# sudo mkfs -t ext4 /dev/sdc1

	![创建文件系统](./media/virtual-machines-linux-how-to-attach-disk/DiskFileSystem.png)

	>[AZURE.NOTE]请注意，对于 ext4 文件系统，SUSE Linux Enterprise 11 系统仅支持只读访问。对于这些系统，建议将新文件系统格式化为 ext3 而非 ext4。


9. 创建一个目录来装载新的文件系统。例如，可键入以下命令，然后输入帐户密码：

		# sudo mkdir /datadrive


10. 键入下面的命令以安装驱动器：

		# sudo mount /dev/sdc1 /datadrive

	数据磁盘现在可以作为 **/datadrive** 使用。


11. 将新驱动器添加到 /etc/fstab：

	若要确保在重新引导后自动重新装载驱动器，必须将其添加到 /etc/fstab 文件。此外，强烈建议在 /etc/fstab 中使用 UUID（全局唯一标识符）来引用驱动器而不是只使用设备名称（即 /dev/sdc1）。若要查找新驱动器的 UUID，可以使用 **blkid** 实用程序：
	
		# sudo -i blkid

	输出与以下内容类似：

		/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
		/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
		/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"


	>[AZURE.NOTE]错误地编辑 **/etc/fstab** 文件可能会导致系统无法引导。如果没有把握，请参考分发的文档来获取有关如何正确编辑该文件的信息。另外，建议在编辑之前创建 /etc/fstab 文件的备份。

	接下来，请在文本编辑器中打开 **/etc/fstab** 文件。请注意，/etc/fstab 是系统文件，因此需使用 `sudo` 来编辑该文件，例如：

		# sudo vi /etc/fstab

	在此示例中，我们将使用在之前的步骤中创建的新 **/dev/sdc1** 设备的 UUID 值并使用装载点 **/datadrive**。将以下行添加到 **/etc/fstab** 文件的末尾：

		UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults   1   2

	另外，在基于 SUSE Linux 的系统上，您可能需要使用稍微不同的格式：

		/dev/disk/by-uuid/33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /   ext3   defaults   1   2

	现在，你可以通过简单地卸载并重新装载文件系统（即使用在之前的步骤中创建的示例装载点 `/datadrive`）来测试文件系统是否已正确装载：

		# sudo umount /datadrive
		# sudo mount /datadrive

	如果 `mount` 命令产生错误，请检查 /etc/fstab 文件的语法是否正确。如果还创建了其他数据驱动器或分区，您同样也需要分别将其输入到 /etc/fstab 中。


>[AZURE.NOTE]之后，在不编辑 fstab 的情况下删除数据磁盘可能会导致 VM 无法引导。如果这是一种常见情况，则请注意，大多数分发都提供了 `nofail` 和/或 `nobootwait` fstab 选项，这些选项使系统在磁盘无法装载的情况下也能引导。有关这些参数的详细信息，请查阅您的分发文档。

## 其他资源
[如何登录到运行 Linux 的虚拟机][Logon]

[如何从 Linux 虚拟机分离磁盘](/documentation/articles/virtual-machines-linux-how-to-detach-disk)

[使用带服务管理 API 的 Azure CLI](/documentation/articles/virtual-machines-command-line-tools)

<!--Link references-->
[Agent]: /documentation/articles/virtual-machines-linux-agent-user-guide
[Logon]: /documentation/articles/virtual-machines-linux-how-to-log-on
<!---HONumber=70-->