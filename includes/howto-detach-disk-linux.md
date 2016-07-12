<properties writer="kathydav" editor="tysonn" manager="timlt" />


当你不再需要附加到虚拟机的数据磁盘时，你可以轻松地分离它。这将从虚拟机中删除磁盘，但不会从存储中删除它。若果你希望再次使用磁盘上的现有数据，可以将其重新附加到相同的虚拟机或另一个虚拟机。

> [AZURE.NOTE]Azure 中的虚拟机使用不同类型的磁盘 - 操作系统磁盘、本地临时磁盘和可选数据磁盘。数据磁盘是为虚拟机存储数据的推荐方法。有关详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-linux-about-disks-vhds/)。除非你同时也删除虚拟机，否则不能分离操作系统磁盘。

## 找到磁盘

在你可以分离虚拟机中的磁盘之前，你需要先确定 LUN 号（要分离的磁盘的标识符）。为此，请执行以下步骤：

1. 	打开适用于 Mac、Linux 和 Windows 的 Azure CLI 并连接到 Azure 订阅有关详细信息，请参阅[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect/)。

2.  确保处于 Azure 服务管理模式下，这是键入 `azure config
 	mode asm` 时的默认模式。

3. 	通过使用 `azure vm disk list
	<virtual-machine-name>` 找出哪些磁盘已附加到虚拟机，如下所示：

		$azure vm disk list ubuntuVMasm
		info:    Executing command vm disk list
		+ Fetching disk images
		+ Getting virtual machines
		+ Getting VM disks
		data:    Lun  Size(GB)  Blob-Name                         OS
		data:    ---  --------  --------------------------------  -----
		data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
		data:    1    10        test.VHD
		data:    0    30        ubuntuVMasm-76f7ee1ef0f6dddc.vhd
		info:    vm disk list command OK

4. 	请注意你想要分离的磁盘的 LUN 或**逻辑单元号** 。


## 分离磁盘

找到磁盘的 LUN 号后，你就可以分离该磁盘：

1. 	通过运行命令 `azure vm disk detach
 	<virtual-machine-name> <LUN>` 分离虚拟机中的所选磁盘，如下所示：

		$azure vm disk detach ubuntuVMasm 0
		info:    Executing command vm disk detach
		+ Getting virtual machines
		+ Removing Data-Disk
		info:    vm disk detach command OK

2. 	你可以通过运行此命令来检查是否该磁盘已分离：

		$azure vm disk list ubuntuVMasm
		info:    Executing command vm disk list
		+ Fetching disk images
		+ Getting virtual machines
		+ Getting VM disks
		data:    Lun  Size(GB)  Blob-Name                         OS
		data:    ---  --------  --------------------------------  -----
		data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
		data:    1    10        test.VHD
		info:    vm disk list command OK

分离的磁盘保留在存储中，但不再附加到虚拟机。

<!---HONumber=79-->