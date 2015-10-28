<properties
	pageTitle="创建在 Azure 中运行 Linux 的虚拟机"
	description="了解如何使用 Azure 中的映像创建运行 Linux 的 Azure 虚拟机 (VM)。"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-management" />

<tags
	ms.service="virtual-machines"
	ms.date="07/13/2015"
	wacn.date="09/18/2015"/>

# 创建运行 Linux 的虚拟机

> [AZURE.SELECTOR]
- [Azure Portal](/documentation/articles/virtual-machines-linux-tutorial-portal-rm)
- [Azure CLI](/documentation/articles/virtual-machines-linux-tutorial)

通过命令行或门户创建运行 Linux 的 Azure 虚拟机 \(VM\) 是一项很简单的操作。本教程演示如何使用 Mac、Linux 和 Windows 的 Azure 命令行界面 \(Azure CLI\) 来快速创建运行在 Azure 中的 Ubuntu Server VM，如何使用 **ssh** 连接到它，以及如何创建和装入新磁盘。（本主题使用 Ubuntu Server VM，不过你也可以[将自己的映像作为模板](/documentation/articles/virtual-machines-linux-create-upload-vhd)来创建 Linux VM。）

<!--[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]-->

## 安装 Azure CLI

第一步是[安装 Azure CLI](/documentation/articles/xplat-cli-install)。

很好。现在，请确保你是处于资源管理模式下，可通过键入 `azure config mode arm` 来验证。

太好了。现在，请使用你的工作或学校 ID 登录，先请键入 `azure login`，然后按提示进行操作。

> [AZURE.NOTE]如果你在登录时收到错误，则可能需要<!--[-->从个人 Microsoft 帐户创建工作或学校 ID<!--](/documentation/articles/resource-group-create-work-id-from-personal)-->。

## 创建 Azure VM

键入 `azure group create <my-group-name> chinanorth`，将 _&lt;my-group-name&gt;_ 替换为对你来说属于唯一名称的组名（如果你愿意，你可以使用其他区域）。你会看到下面这样的内容：

	azure group create myuniquegroupname chinanorth
	info:    Executing command group create
	+ Getting resource group myuniquegroupname
	+ Creating resource group myuniquegroupname
	info:    Created resource group myuniquegroupname
	data:    Id:                  /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myuniquegroupname
	data:    Name:                myuniquegroupname
	data:    Location:            chinanorth
	data:    Provisioning State:  Succeeded
	data:    Tags:
	data:
	info:    group create command OK

现在可以创建你的 VM 了，先键入 `azure vm quick-create`，然后系统会提示你输入余下的参数。使用你刚在上面创建的资源组的名称，而对于 **ImageURN** 值，请使用 `canonical:ubuntuserver:14.04.2-LTS:latest`，这样你的体验看起来将如下所示：

	azure vm quick-create
	info:    Executing command vm quick-create
	Resource group name: myuniquegroupname
	Virtual machine name: myuniquevmname
	Location name: chinanorth
	Operating system Type [Windows, Linux]: /documentation/articles/Linux
	ImageURN (format: "publisherName:offer:skus:version"): canonical:ubuntuserver:14.04.2-LTS:latest
	User name: ops
	Password: *********
	Confirm password: *********
	+ Looking up the VM "myuniquevmname"
	info:    Using the VM Size "Standard_D1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Retrieving storage accounts
	info:    Could not find any storage accounts in the region "chinanorth", trying to create new one
	+ Creating storage account "cli3c0464f24f1bf4f014323" in "chinanorth"
	+ Looking up the storage account cli3c0464f24f1bf4f014323
	+ Looking up the NIC "myuni-westu-1432328437727-nic"
	info:    An nic with given name "myuni-westu-1432328437727-nic" not found, creating a new one
	+ Looking up the virtual network "myuni-westu-1432328437727-vnet"
	info:    Preparing to create new virtual network and subnet
	/ Creating a new virtual network "myuni-westu-1432328437727-vnet" [address prefix: "10.0.0.0/16"] with subnet "myuni-westu-1432328437727-snet"+[address prefix: "10.0.1.0/24"]
	+ Looking up the virtual network "myuni-westu-1432328437727-vnet"
	+ Looking up the subnet "myuni-westu-1432328437727-snet" under the virtual network "myuni-westu-1432328437727-vnet"
	info:    Found public ip parameters, trying to setup PublicIP profile
	+ Looking up the public ip "myuni-westu-1432328437727-pip"
	info:    PublicIP with given name "myuni-westu-1432328437727-pip" not found, creating a new one
	+ Creating public ip "myuni-westu-1432328437727-pip"
	+ Looking up the public ip "myuni-westu-1432328437727-pip"
	+ Creating NIC "myuni-westu-1432328437727-nic"
	+ Looking up the NIC "myuni-westu-1432328437727-nic"
	+ Creating VM "myuniquevmname"
	+ Looking up the VM "myuniquevmname"
	+ Looking up the NIC "myuni-westu-1432328437727-nic"
	+ Looking up the public ip "myuni-westu-1432328437727-pip"
	data:    Id                              :/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myuniquegroupname/providers/Microsoft.Compute/virtualMachines/myuniquevmname
	data:    ProvisioningState               :Succeeded
	data:    Name                            :myuniquevmname
	data:    Location                        :chinanorth
	data:    FQDN                            :myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com
	data:    Type                            :Microsoft.Compute/virtualMachines
	data:
	data:    Hardware Profile:
	data:      Size                          :Standard_D1
	data:
	data:    Storage Profile:
	data:      Image reference:
	data:        Publisher                   :canonical
	data:        Offer                       :ubuntuserver
	data:        Sku                         :14.04.2-LTS
	data:        Version                     :latest
	data:
	data:      OS Disk:
	data:        OSType                      :Linux
	data:        Name                        :cli3c0464f24f1bf4f0-os-1432328438224
	data:        Caching                     :ReadWrite
	data:        CreateOption                :FromImage
	data:        Vhd:
	data:          Uri                       :https://cli3c0464f24f1bf4f014323.blob.core.chinacloudapi.cn/vhds/cli3c0464f24f1bf4f0-os-1432328438224.vhd
	data:
	data:    OS Profile:
	data:      Computer Name                 :myuniquevmname
	data:      User Name                     :ops
	data:      Linux Configuration:
	data:        Disable Password Auth       :false
	data:
	data:    Network Profile:
	data:      Network Interfaces:
	data:        Network Interface #1:
	data:          Id                        :/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myuniquegroupname/providers/Microsoft.Network/networkInterfaces/myuni-westu-1432328437727-nic
	data:          Primary                   :true
	data:          MAC Address               :00-0D-3A-31-55-31
	data:          Provisioning State        :Succeeded
	data:          Name                      :myuni-westu-1432328437727-nic
	data:          Location                  :chinanorth
	data:            Private IP alloc-method :Dynamic
	data:            Private IP address      :10.0.1.4
	data:            Public IP address       :191.239.51.1
	data:            FQDN                    :myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com
	info:    vm quick-create command OK

你的 VM 已启动并运行，正在等待你进行连接。

## 连接到 VM

对于 Linux VM，通常使用 **ssh** 进行连接。本主题介绍如何使用用户名和密码连接到 VM；若要使用公钥和私钥对与 VM 通信，请参阅[如何在 Azure 上通过 Linux 使用 SSH](/documentation/articles/virtual-machines-linux-use-ssh-key)。

如果你不熟悉如何使用 **ssh** 进行连接，请注意，该命令采用以下形式：`ssh <username>@<publicdnsaddress> -p <the ssh port>`。在本示例中，我们使用前一步的用户名和密码，并使用端口 22，该端口是默认的 **ssh** 端口。

	ssh ops@myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com -p 22
	The authenticity of host 'myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com (191.239.51.1)' can't be established.
	ECDSA key fingerprint is bx:xx:xx:xx:xx:xx:xx:xx:xx:x:x:x:x:x:x:xx.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com,191.239.51.1' (ECDSA) to the list of known hosts.
	ops@myuni-westu-1432328437727-pip.chinanorth.cloudapp.azure.com's password:
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

现在，你已连接到 VM，可以连接磁盘了。

## 连接和安装磁盘

连接新的磁盘很快。键入 `azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>` 即可为 VM 创建和连接新的 GB 磁盘。你应该会看到类似下面的屏幕：

	azure vm disk attach-new myuniquegroupname myuniquevmname 5
	info:    Executing command vm disk attach-new
	+ Looking up the VM "myuniquevmname"
	info:    New data disk location: https://cliexxx.blob.core.chinacloudapi.cn/vhds/myuniquevmname-20150526-0xxxxxxx43.vhd
	+ Updating VM "myuniquevmname"
	info:    vm disk attach-new command OK


现在，让我们使用 `dmesg | grep SCSI` 来查找磁盘（用于发现新磁盘的方法可能各不相同）。在本示例中，键入的内容如下所示：

	dmesg | grep SCSI
	[    0.294784] SCSI subsystem initialized
	[    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
	[    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
	[    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
	[ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk

而在本主题中，`sdc` 磁盘是我们所需要的。现在使用 `sudo fdisk /dev/sdc` 对磁盘进行分区 -- 假定在你的示例中，磁盘为 `sdc`，则应将其设置为分区 1 中的主磁盘，并接受其他默认设置值。

	sudo fdisk /dev/sdc
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
	bin   datadrive  etc   initrd.img  lib64       media  opt   root  sbin  sys  usr  vmlinuz
	boot  dev        home  lib         lost+found  mnt    proc  run   srv   tmp  var

> [AZURE.NOTE]你还可以在连接到 Linux 虚拟机时，使用 SSH 密钥进行身份验证。有关详细信息，请参阅[如何在 Azure 上将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-use-ssh-key)。

## 后续步骤

请记住，即使重新启动 VM，你的新磁盘通常也无法供 VM 使用，除非你将该信息写入 [fstab](http://en.wikipedia.org/wiki/Fstab) 文件。

若要了解有关 Azure 上的 Linux 的详细信息，请参阅：

- [Azure 上的 Linux 和开源计算](/documentation/articles/virtual-machines-linux-opensource)

- [如何使用 Azure 命令行界面](/documentation/articles/virtual-machines-command-line-tools)

- [使用适用于 Linux 的 Azure CustomScript 扩展部署 LAMP 应用程序](/documentation/articles/virtual-machines-linux-script-lamp)

- [关于 Azure VM 配置设置](http://msdn.microsoft.com/zh-cn/library/azure/dn763935.aspx)

- [Azure 上用于 Linux 的 Docker 虚拟机扩展](/documentation/articles/virtual-machines-docker-vm-extension)
 
