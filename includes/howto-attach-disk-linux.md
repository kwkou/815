
有关磁盘的更多详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-linux-about-disks-vhds/)。

<a id="attachempty"></a>
## 如何：附加空磁盘
附加空磁盘是添加数据磁盘的更简单方法，因为 Azure 将为你创建 .vhd 文件并将其存储在存储帐户中。

1.  打开适用于 Mac、Linux 和 Windows 的 Azure CLI 并连接到 Azure 订阅有关详细信息，请参阅[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect/)。

2.  确保处于 Azure 服务管理模式下，这是键入 `azure config mode asm` 时的默认模式。

3.  使用命令 `azure vm disk attach-new` 创建规则并附加一个新磁盘，如下所示。请注意，_ubuntuVMasm_ 将被替换为你在你的订阅中已创建的 Linux 虚拟机的名称。在此示例中数字 30 是以 GB 为单位的磁盘大小。

        azure vm disk attach-new ubuntuVMasm 30

4.	创建并附加数据磁盘后，它列出在 `azure vm disk list
    <virtual-machine-name>` 的输出中，如下所示：

        $ azure vm disk list ubuntuVMasm
        info:    Executing command vm disk list
        + Fetching disk images
        + Getting virtual machines
        + Getting VM disks
        data:    Lun  Size(GB)  Blob-Name                         OS
        data:    ---  --------  --------------------------------  -----
        data:         30        ubuntuVMasm-2645b8030676c8f8.vhd  Linux
        data:    0    30        ubuntuVMasm-76f7ee1ef0f6dddc.vhd
        info:    vm disk list command OK

<a id="attachexisting"></a>
## 如何：附加现有磁盘

附加现有磁盘需要存储帐户中具有可用的 .vhd。

1. 	打开适用于 Mac、Linux 和 Windows 的 Azure CLI 并连接到 Azure 订阅有关详细信息，请参阅[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect/)。

2.  确保处于 Azure 服务管理模式下，这是默认模式。如果你已将模式更改为资源管理，只需通过键入 `azure config mode asm` 来还原。

3.	通过使用以下命令确定你想要附加的 VHD 是否已上载到你的 Azure 订阅：

        $azure vm disk list
    	info:    Executing command vm disk list
    	+ Fetching disk images
    	data:    Name                                          OS
    	data:    --------------------------------------------  -----
    	data:    myTestVhd                                     Linux
    	data:    ubuntuVMasm-ubuntuVMasm-0-201508060029150744  Linux
    	data:    ubuntuVMasm-ubuntuVMasm-0-201508060040530369
    	info:    vm disk list command OK

4.  如果没有找到你想要使用的磁盘，您可以使用 `azure vm disk create` 或 `azure vm disk upload` 将一个本地 VHD 上载到你的订阅。如以下示例所示：

        $azure vm disk create myTestVhd2 .\TempDisk\test.VHD -l "China East" -o Linux
		info:    Executing command vm disk create
		+ Retrieving storage accounts
		info:    VHD size : 10 GB
		info:    Uploading 10485760.5 KB
		Requested:100.0% Completed:100.0% Running:   0 Time:   25s Speed:    82 KB/s
		info:    Finishing computing MD5 hash, 16% is complete.
		info:    https://portalvhdsq1s6mc7mqf4gn.blob.core.chinacloudapi.cn/disks/test.VHD was
		uploaded successfully
		info:    vm disk create command OK

	你也可以使用 `azure vm disk upload` 命令来将 VHD 上载到特定的存储帐户。详细阅读命令，以管理你[在这里](/documentation/articles/virtual-machines-command-line-tools/#commands-to-manage-your-azure-virtual-machine-data-disks)的 Azure 虚拟机数据磁盘。

5.  键入以下命令以将所需上载的 VHD 附加到虚拟机：

		$azure vm disk attach ubuntuVMasm myTestVhd
		info:    Executing command vm disk attach
		+ Getting virtual machines
		+ Adding Data-Disk
		info:    vm disk attach command OK

	请确保将 _ubuntuVMasm_ 替换为您的虚拟机名称并将 _myTestVhd_ 替换为你所需的 VHD。

6.	你可以按照以下所示使用命令 `azure vm disk list <virtual-machine-name>` 验证磁盘是否已附加到虚拟机：

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


> [AZURE.NOTE]
在添加数据磁盘后，您需要登录到虚拟机并初始化磁盘，然后虚拟机才能使用磁盘来存储数据。

<!---HONumber=79-->