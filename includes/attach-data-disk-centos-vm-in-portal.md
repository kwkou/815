1. 在 Azure [经典管理门户](http://manage.windowsazure.cn)中，单击“虚拟机”，然后选择刚才创建的虚拟机 (**testlinuxvm**)。

2. 在命令栏上，单击“附加”，然后单击“附加空磁盘”。

	将显示“附加空磁盘”对话框。


3. 已经为你定义了“虚拟机名称”、“存储位置”和“文件名”。您只需要输入所需的磁盘大小。在“大小”字段中键入“5”。

	![附加空磁盘][Image2]

	**注意：**所有磁盘都是根据 Azure 存储中的 .vhd 文件创建的。你可以为添加到存储的 .vhd 文件提供名称，但是 Azure 会自动生成磁盘名称。

4. 单击复选标记以将数据磁盘附加到虚拟机。

5. 单击虚拟机的名称可显示仪表板，这样你就可以验证数据磁盘是否已成功附加到虚拟机。你附加的磁盘会列在“磁盘”表中。

	附加数据磁盘后，必须登录完成设置才能使用该磁盘。

##使用 SSH 或 PuTTY 连接到虚拟机并完成安装
登录虚拟机完成磁盘设置，以便用它来存储数据。

1. 预配虚拟机后，使用 SSH 或 PuTTY 进行连接，并作为 **newuser** 进行登录（如上述步骤中所述）。	


2. 在 SSH 或 PuTTY 窗口中，键入以下命令，然后输入帐户密码：

	`$ sudo grep SCSI /var/log/messages`

	你可以在所示消息中找到上次添加的数据磁盘的标识符（在此示例中为 **sdc**）。

	![GREP][Image4]


3. 在 SSH 或 PuTTY 窗口中，输入以下命令，对磁盘 **/dev/sdc** 进行分区：

	`$ sudo fdisk /dev/sdc`


4. 输入 **n** 新建一个分区。

	![FDISK][Image5]


5. 键入 **p** 将该分区设置为主分区，键入 **1** 将其设置为第一分区，然后按 Enter 以接受默认分区值 (1)。

	![FDISK][Image6]


6. 键入 **p** 以查看有关分区磁盘的详细信息。

	![FDISK][Image7]


7. 键入“w”以写入磁盘的设置。

	![FDISK][Image8]


8. 使用 **mkfs** 命令格式化新磁盘：

	`$ sudo mkfs -t ext4 /dev/sdc1`

9. 接下来，您必须有一个目录可用于装载新文件系统。例如，键入下面的命令来创建一个用于装载驱动器的新目录，然后输入帐户密码：

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

	如果还创建了其他数据驱动器或分区，您同样也需要分别将其输入到 /etc/fstab 中。

	现在，你可以通过简单地卸载并重新装载文件系统（即使用在之前的步骤中创建的示例装载点 `/datadrive`）来测试文件系统是否已正确装载：

		`sudo umount /datadrive`
		`sudo mount /datadrive`

	如果第二个命令产生错误，请检查 /etc/fstab 文件的语法是否正确。


	>[AZURE.NOTE]之后，在不编辑 fstab 的情况下删除数据磁盘可能会导致 VM 无法引导。如果这是一种常见情况，则请注意，大多数分发都提供了 `nofail` 和/或 `nobootwait` fstab 选项，这些选项使系统在磁盘不存在时也能引导。有关这些参数的详细信息，请查阅您的分发文档。


[Image2]: ./media/attach-data-disk-centos-vm-in-portal/AttachDataDiskLinuxVM2.png
[Image4]: ./media/attach-data-disk-centos-vm-in-portal/GrepScsiMessages.png
[Image5]: ./media/attach-data-disk-centos-vm-in-portal/fdisk1.png
[Image6]: ./media/attach-data-disk-centos-vm-in-portal/fdisk2.png
[Image7]: ./media/attach-data-disk-centos-vm-in-portal/fdisk3.png
[Image8]: ./media/attach-data-disk-centos-vm-in-portal/fdisk4.png
[Image9]: ./media/attach-data-disk-centos-vm-in-portal/mkfs.png

<!---HONumber=Mooncake_1207_2015-->