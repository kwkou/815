1. 在 [Windows Azure（预览版）管理门户][AzurePreviewPortal]中，单击“虚拟机”，然后选择刚才创建的虚拟机 (**testlinuxvm**)。

2. 在命令栏上，单击“附加”，然后单击“附加空磁盘”。

	将显示“附加空磁盘”对话框。


3. 已为您定义好了“虚拟机名称”、“存储位置”和“文件名”。您只需要输入所需的磁盘大小。在“大小”字段中键入 5。

	![附加空磁盘][Image2]

	**注意**：所有磁盘都是从 Windows Azure 存储中的 VHD 文件创建的。您可以为添加到存储的 VHD 文件提供名称，但是 Windows Azure 会自动生成磁盘名称。

4. 单击复选标记以将数据磁盘附加到虚拟机。

5. 单击虚拟机的名称可显示仪表板；这样您可以验证数据磁盘是否已成功附加到虚拟机。

	现在虚拟机的磁盘数为 2 个。“磁盘”表中列出了您附加的磁盘。

	![附加空磁盘][Image3]

	在您将数据磁盘附加到虚拟机后，该磁盘会处于脱机和未初始化状态。您必须先登录虚拟机并初始化磁盘，才能使用该磁盘存储数据。

##使用 SSH 或 PuTTY 连接到虚拟机并完成安装
您刚刚添加到虚拟机中的数据磁盘在您添加它后处于脱机和未初始化状态。您必须先登录到虚拟机并初始化磁盘，然后才能使用该磁盘存储数据。

1. 配置虚拟机后，使用 SSH 或 PuTTY 进行连接，并作为 newuser 进行登录（如上述步骤中所述）。	

2. 在 SSH 或 PuTTY 窗口中，键入以下命令，然后输入帐户密码：

	`$ sudo grep SCSI /var/log/messages`

	您可以在所示消息中找到上次添加的数据磁盘的标识符（在此示例中为 sdc）。

	![GREP][Image4]

3. 在 SSH 或 PuTTY 窗口中，输入以下命令，对磁盘 /dev/sdc 进行分区：

	`$ sudo fdisk /dev/sdc`

4. 输入 n 新建一个分区。

	![FDISK][Image5]

5. 键入 p 将该分区设置为主分区，键入 1 将其设置为第一分区，然后键入 Enter 以接受柱面的默认值 (1)。

	![FDISK][Image6]

6. 键入 p 以查看有关分区磁盘的详细信息。

	![FDISK][Image7]

7. 键入 w 以写入磁盘的设置。

	![FDISK][Image8]

8. 使用 mkfs.ext3 命令格式化新磁盘：

	`$ sudo mkfs.ext3 /dev/sdc1`

	![Format Disk][Image9]

9.创建目录以便为驱动器设置装入点：

	`$ sudo mkdir /mnt/datadrive`

10. 安装驱动器：

	`$ sudo mount /dev/sdc1 /mnt/datadrive`

11. 打开 /etc/fstab 文件并附加以下行：

	`/dev/sdc1        /mnt/datadrive      ext3    defaults   1 2`

12. 保存并关闭 /etc/fstab 文件。

13. 使用 e2label 标记分区：

	`$ sudo e2label /dev/sdc1 /mnt/datadrive`




[Image2]: ./media/attach-data-disk-centos-vm-in-portal/AttachDataDiskLinuxVM2.png
[Image3]: ./media/attach-data-disk-centos-vm-in-portal/AttachDataDiskLinuxVM3.png
[Image4]: ./media/attach-data-disk-centos-vm-in-portal/GrepScsiMessages.png
[Image5]: ./media/attach-data-disk-centos-vm-in-portal/fdisk1.png
[Image6]: ./media/attach-data-disk-centos-vm-in-portal/fdisk2.png
[Image7]: ./media/attach-data-disk-centos-vm-in-portal/fdisk3.png
[Image8]: ./media/attach-data-disk-centos-vm-in-portal/fdisk4.png
[Image9]: ./media/attach-data-disk-centos-vm-in-portal/mkfs.png



