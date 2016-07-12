

## <a id="logon"> </a>虚拟机创建后如何进行登录 ##

若要管理虚拟机的设置以及在其上运行的应用程序，可以使用 SSH 客户端。为此，您必须在计算机上安装要用于访问虚拟机的 SSH 客户端。您可以选择很多 SSH 客户端程序。以下是可能的选择：

- 如果您要使用运行 Windows 操作系统的计算机，则可能希望使用 PuTTY 等 SSH 客户端。有关详细信息，请参阅 [PuTTY 下载](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。
- 如果您要使用运行 Linux 操作系统的计算机，则可能希望使用 OpenSSH 等 SSH 客户端。有关详细信息，请参阅 [OpenSSH](http://www.openssh.org/)。

此教程将向您演示如何使用 PuTTY 程序访问虚拟机。

1. 请从“经典管理门户”查找“主机名”和“端口信息”。您可以从虚拟机的仪表板中找到所需信息。单击虚拟机名称并查看仪表板“速览”部分的“SSH 详细信息”。

	![查找 SSH 详细信息](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/SSHdetails.png)

2. 打开 PuTTY 程序。

3. 输入你从仪表板收集到的“主机名”和“端口信息”，然后单击“打开”。

	![输入主机名和端口信息](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/putty.png)

4. 使用你在创建虚拟机时添加的 NewUser1 帐户登录到虚拟机。

	![登录到新虚拟机](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/sshlogin.png)

	您现在可以像使用任何其他服务器一样使用该虚拟机。


## <a id="attachdisk"> </a>如何将数据磁盘附加到新虚拟机 ##

您的应用程序可能需要存储数据。若要执行此操作，您可以将数据磁盘附加到之前创建的虚拟机。执行此操作的最简单方法是将空数据磁盘附加到计算机。

在 Linux 上，资源磁盘通常由 Azure Linux 代理管理并且自动装载到 /mnt/resource（或 Ubuntu 映像上的 /mnt）。另一方面，在 Linux 上，数据磁盘由内核命名为 `/dev/sdc`，并且用户将需要对该资源进行分区、格式化和安装。有关详细信息，请参阅 [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)。

>[AZURE.NOTE]不要在资源磁盘上存储数据。该磁盘为应用程序和进程提供临时存储空间，用于存储不需要保留的数据，比如交换文件。数据磁盘在 Azure 存储空间中以页 Blob 中的 .vhd 文件形式驻留，并且提供存储冗余，从而保护你的数据。有关详细信息，请参阅[关于 Azure 中的磁盘和映像](http://msdn.microsoft.com/zh-cn/library/jj672979.aspx)。

1. 如果您尚未这么做，请登录到 Azure 经典管理门户。

2. 单击“虚拟机”，然后选择你之前创建的 MyTestVM1 虚拟机。

3. 在命令栏上，单击“附加”，然后单击“附加空磁盘”。
	
	将显示“附加空磁盘”对话框。

	![定义磁盘详细信息](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/attachnewdisklinux.png)

4. 已经为你定义了“虚拟机名称”、“存储位置”和“文件名”。您只需要输入所需的磁盘大小。在“大小”字段中键入“5”。

	**注意：**所有磁盘都是从 Azure 存储中的 VHD 文件创建的。您可以为添加到存储的 VHD 文件提供名称，但会自动生成磁盘名称。

5. 单击复选标记以将数据磁盘附加到虚拟机。

6. 您可以通过查看仪表板验证数据磁盘是否已成功附加到虚拟机。单击虚拟机的名称以显示仪表板。

	现在虚拟机的磁盘数是 2 个，你附加的磁盘列在“磁盘”表中。

	![附加磁盘成功](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/attachemptysuccess.png)


您刚刚添加到虚拟机中的数据磁盘在您添加它后处于脱机和未初始化状态。您必须先登录到虚拟机并初始化磁盘，然后才能使用该磁盘存储数据。

1. 使用在上面的“创建虚拟机后如何进行登录”中列出的步骤连接到虚拟机。


2. 在 SSH 窗口中，键入以下命令，然后输入帐户密码：

	`sudo grep SCSI /var/log/messages`

	您可以在所示消息中找到最后添加的数据磁盘的标识符。

	![标识磁盘](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskmessages.png)


3. 在 SSH 窗口中，键入下面的命令以新建设备，然后输入帐户密码：

	`sudo fdisk /dev/sdc`

	>[AZURE.NOTE]此示例中，如果 /sbin 或 /usr/sbin 不在你的 `$PATH` 中，你在某些分发上可能需要使用 `sudo -i`。


4. 键入“n”以创建新分区。

	![新建设备](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskpartition.png)


5. 键入“p”将该分区设置为主分区，键入“1”将其设置为第一分区，然后键入 Enter 以接受柱面的默认值。

	![创建分区](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskcylinder.png)


6. 键入“p”以查看有关分区磁盘的详细信息。

	![列出磁盘信息](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskinfo.png)


7. 键入“w”以写入磁盘的设置。

	![写入磁盘更改](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskwrite.png)


8. 您必须在新分区上创建文件系统。例如，键入下面的命令以创建文件系统，然后输入帐户密码：

	`sudo mkfs -t ext4 /dev/sdc1`

	![创建文件系统](./media/virtual-machines-Linux-tutorial-log-on-attach-disk/diskfilesystem.png)

	>[AZURE.NOTE]请注意，SUSE Linux Enterprise 11 系统对于 ext4 文件系统仅提供了只读访问权限。对于这些系统，建议将新文件系统格式化为 ext3 而非 ext4。


9. 创建一个目录来装载新的文件系统。例如，可键入以下命令，然后键入帐户密码：

	`sudo mkdir /datadrive`


10. 键入下面的命令以安装驱动器：

	`sudo mount /dev/sdc1 /datadrive`

	数据磁盘现在可以作为 /datadrive 使用。


11. 将新驱动器添加到 /etc/fstab：

	若要确保在重新引导后自动重新装载驱动器，必须将其添加到 /etc/fstab 文件。此外，强烈建议在 /etc/fstab 中使用 UUID（全局唯一标识符）来引用驱动器而不是只使用设备名称（即 /dev/sdc1）。若要查找新驱动器的 UUID，可以使用 blkid 实用程序：
	
		`sudo -i blkid`

	输出与以下内容类似：

		`/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"`
		`/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"`
		`/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"`

	>[AZURE.NOTE]Blkid 可能不是在所有情况下都需要 sudo 访问权限，不过，如果 /sbin 或 /usr/sbin 不在你的 `$PATH` 中，则在某些分发上使用 `sudo -i` 运行可能更为容易。

	**警告：**错误地编辑 /etc/fstab 文件可能会导致系统无法引导。如果没有把握，请参考分发的文档来获取有关如何正确编辑该文件的信息。另外，建议在编辑之前创建 /etc/fstab 文件的备份。

	使用文本编辑器，在 /etc/fstab 文件的末尾输入有关新文件系统的信息。在此示例中，我们将使用在之前的步骤中创建的新 /dev/sdc1 设备的 UUID 值并使用装载点 /datadrive：

		`UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults   1   2`

	另外，在基于 SUSE Linux 的系统上，您可能需要使用稍微不同的格式：

		`/dev/disk/by-uuid/33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /   ext3   defaults   1   2`

	如果还创建了其他数据驱动器或分区，您同样也需要分别将其输入到 /etc/fstab 中。

	现在，你可以通过简单地卸载并重新装载文件系统（即使用在之前的步骤中创建的示例装载点 `/datadrive`）来测试文件系统是否已正确装载：

		`sudo umount /datadrive`
		`sudo mount /datadrive`

	如果第二个命令产生错误，请检查 /etc/fstab 文件的语法是否正确。


	>[AZURE.NOTE]之后，在不编辑 fstab 的情况下删除数据磁盘可能会导致 VM 无法引导。如果这是一种常见情况，则请注意，大多数分发都提供了 `nofail` 和/或 `nobootwait` fstab 选项，这些选项使系统在磁盘不存在时也能引导。有关这些参数的详细信息，请查阅您的分发文档。

<!---HONumber=70-->