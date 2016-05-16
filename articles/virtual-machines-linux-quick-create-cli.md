<properties
   pageTitle="使用 CLI 在 Azure 上创建 Linux VM | Azure"
   description="使用 CLI 在 Azure 上创建 Linux VM。"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="vlivech"
   manager="timlt"
   editor=""/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/08/2016"
	wacn.date="05/16/2016"/>

# 创建 Linux 虚拟机

> [AZURE.SELECTOR]
- [Portal - Windows](/documentation/articles/virtual-machines-windows-classic-tutorial)
- [PowerShell](/documentation/articles/virtual-machines-windows-classic-create-powershell)
- [Portal - Linux](/documentation/articles/virtual-machines-linux-portal-create)
- [CLI](/documentation/articles/virtual-machines-linux-quick-create-cli)

通过命令行或门户创建运行 Linux 虚拟机 (VM) 是一项很简单的操作。本教程说明如何使用 Mac、Linux 和 Windows 的 Azure 命令行界面 (CLI) 来快速创建运行在 Azure 中的 Ubuntu Server VM，如何使用 **ssh** 连接到它，以及如何创建和装入新磁盘。本主题使用 Ubuntu Server VM，不过你也可以[将自己的映像作为模板](/documentation/articles/virtual-machines-linux-classic-create-upload-vhd)来创建 Linux 虚拟机。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

[AZURE.INCLUDE [free-trial-note](../includes/free-trial-note.md)]

## 安装 Azure CLI

第一步是[安装 Azure CLI](/documentation/articles/xplat-cli-install)。

很好。现在，请确保你是处于服务管理器模式下，可通过键入 `azure config mode asm` 来验证。

太好了。现在，通过键入 `azure login -e AzureChinaCloud -u <your account>` 并遵循提示进行 Azure 帐户的交互式登录体验，来[使用工作或学校 ID 登录](/documentation/articles/xplat-cli-connect#use-the-log-in-method)。

## 创建 Linux 虚拟机

可以使用 `azure vm create-from <your-vm-name> example.json` 来新建虚拟机。如果要为虚拟机创建新云服务，需要使用 `-l` 参数来指定云服务的位置。例如，`azure vm create-from <your-vm-name> example.json -l "China East"`。下面是用于创建虚拟机的示例 json 文件。

	{
	    "configurationSets": [
	        {
	            "inputEndpoints": [
	                {
	                    "localPort": 22,
	                    "name": "SSH",
	                    "port": 22,
	                    "protocol": "tcp",
	                    "virtualIPAddress": "",
	                    "enableDirectServerReturn": false
	                }
	            ],
	            "networkInterfaces": [],
	            "publicIPs": [],
	            "storedCertificateSettings": [],
	            "subnetNames": [],
	            "configurationSetType": "NetworkConfiguration"
	        },
	        {
	            "configurationSetType":"LinuxProvisioningConfiguration",
	            "hostName": "myubuntu",
	            "userName": "<adminName>",
	            "userPassword": "<adminPass>",
                "disableSshPasswordAuthentication": "false",
                "sSh":{
                    "publicKeys": [
                        {
                            "fingerprint": "",
                            "path": ""
                        }
                    ],
                    "keyPairs": [
                        {
                            "fingerprint": "",
                            "path": ""
                        }
                    ]
                },
                "customData": ""
	        }
	    ],
	    "dataVirtualHardDisks": [],
	    "resourceExtensionReferences": [],
	    "roleName": "myubuntu",
	    "oSVersion": "",
	    "roleType": "PersistentVMRole",
	    "oSVirtualHardDisk": {
	        "hostCaching": "ReadWrite",
	        "mediaLink": "https://<storageaccountname>.blob.core.chinacloudapi.cn/vhds/myubuntu-myubuntu-2016-02-19.vhd",
	        "sourceImageName":"b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140416.1-en-us-30GB",
	        "operatingSystem": "Linux"
	    },
	    "roleSize": "Basic_A0",
	    "provisionGuestAgent": true
	}

- configurationSets：包含配置集的集合，这些配置集定义系统和应用程序设置。

	* NetworkConfiguration：你可以选择指定 NetworkConfiguration 集，其中包含为虚拟机创建虚拟网络配置所需的元数据。
	* LinuxProvisioningConfiguration：可以配置 VM 主机名、用户名、密码和 SSH 公钥，等等。

- dataVirtualHardDisks：指定已附加到 VM 的 VHD。如果你使用高级存储，“roleSize”必须是 DS 系列。
- oSVirtualHardDisk：指定 VM 的 OS VHD。

	* mediaLink：指定创建 VHD 文件的位置。
	* sourceImageName：指定用于创建虚拟机的映像的名称。可以通过 `azure vm image list` 获取映像名称。有关映像搜索的详细信息，请参阅[导航和选择 VM 映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage)。

- roleSize：指定虚拟机的大小。可以使用 `azure vm location list --json` 查看可用的“virtualMachinesRoleSizes”。

有关每个字段的详细信息，请查看[创建虚拟机部署](https://msdn.microsoft.com/zh-cn/library/azure/jj157194.aspx) REST API 的请求正文。json 文件与请求正文在结构上完全相同，不过，请求正文采用 xml，并且每个字段名称以大写字母开头，而 `azure vm create-from` 使用 json 文件，并且每个字段名称以小写字母开头。

片刻之后，你将看到以下输出 ︰

	info:    Executing command vm create-from
	+ Looking up cloud service
	info:    cloud service myubuntu not found.
	+ Creating cloud service
	+ Creating VM
	info:    vm create-from command OK

你可以使用 `azure vm show <you-vm-name>` 来获取虚拟机的状态。你将获得如下所示的输出：

	info:    Executing command vm show
	+ Getting virtual machines
	data:    DNSName "myubuntu.chinacloudapp.cn"
	data:    Location "China East"
	data:    VMName "myubuntu"
	data:    IPAddress "<private ip>"
	data:    InstanceStatus "ReadyRole"
	data:    InstanceSize "Basic_A0"
	data:    Image "b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140416.1-en-us-30GB"
	data:    OSDisk hostCaching "ReadWrite"
	data:    OSDisk name "myubuntu-myubuntu-0-201601070618550344"
	data:    OSDisk mediaLink "https://<storageaccountname>.blob.core.chinacloudapi.cn/vhds/myubuntu-myubuntu-2016-02-19.vhd"
	data:    OSDisk sourceImageName "b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140416.1-en-us-30GB"
	data:    OSDisk operatingSystem "Linux"
	data:    OSDisk iOType "Standard"
	data:    ReservedIPName ""
	data:    VirtualIPAddresses 0 address "<public ip>"
	data:    VirtualIPAddresses 0 name "myubuntuContractContract"
	data:    VirtualIPAddresses 0 isDnsProgrammed true
	data:    Network Endpoints 0 localPort 22
	data:    Network Endpoints 0 name "SSH"
	data:    Network Endpoints 0 port 22
	data:    Network Endpoints 0 protocol "tcp"
	data:    Network Endpoints 0 virtualIPAddress "<public ip>"
	data:    Network Endpoints 0 enableDirectServerReturn false

如果看到了 `InstanceStatus "ReadyRole"`，则表示 VM 已启动并运行，正在等待你进行连接。

## 连接到 Linux 虚拟机

对于 Linux 虚拟机，通常使用 ssh 进行连接。

> [AZURE.NOTE] 本主题介绍如何使用用户名和密码连接到 VM；若要使用公钥和私钥对与 VM 通信，请参阅[如何在 Azure 上通过 Linux 使用 SSH](/documentation/articles/virtual-machines-linux-ssh-from-linux)。

如果你不熟悉如何使用 **ssh** 进行连接，请注意，该命令采用以下形式：`ssh <username>@<publicdnsaddress> -p <the ssh port>`。在本示例中，我们使用前一步的用户名和密码，并使用端口 22，该端口是默认的 **ssh** 端口。

	ssh ops@myuni-westu-1432328437727-pip.chinanorth.chinacloudapp.cn -p 22
	The authenticity of host 'myuni-westu-1432328437727-pip.chinanorth.chinacloudapp.cn (191.239.51.1)' can't be established.
	ECDSA key fingerprint is bx:xx:xx:xx:xx:xx:xx:xx:xx:x:x:x:x:x:x:xx.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'myuni-westu-1432328437727-pip.chinanorth.chinacloudapp.cn,191.239.51.1' (ECDSA) to the list of known hosts.
	ops@myuni-westu-1432328437727-pip.chinanorth.chinacloudapp.cn's password:
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

连接新的磁盘很快。键入 `vm disk attach-new [options] <vm-name> <size-in-gb> [blob-url]` 即可为 VM 创建和连接新的 GB 磁盘。你应该会看到类似下面的屏幕：

	azure vm disk attach-new <you-vm-name> 5 https://<storageAccoutName>.blob.core.chinacloudapi.cn/vhds/temp.vhd
	info:    Executing command vm disk attach-new
	+ Getting virtual machines
	+ Adding Data-Disk
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

> [AZURE.NOTE] 你还可以在连接到 Linux 虚拟机时，使用 SSH 密钥进行身份验证。有关详细信息，请参阅[如何在 Azure 上将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-ssh-from-linux)。

## 后续步骤

请记住，即使重新启动 VM，你的新磁盘通常也无法供 VM 使用，除非你将该信息写入 [fstab](http://en.wikipedia.org/wiki/Fstab) 文件。如果需要，你可以再添加几个磁盘并[配置 RAID](/documentation/articles/virtual-machines-linux-configure-raid)。

若要了解有关 Azure 上的 Linux 的详细信息，请参阅：

- [Azure 上的 Linux 和开源计算](/documentation/articles/virtual-machines-linux-opensource-links)

- [如何使用 Azure 命令行界面](/documentation/articles/virtual-machines-command-line-tools)

- [使用适用于 Linux 的 Azure CustomScript 扩展部署 LAMP 应用程序](/documentation/articles/virtual-machines-linux-classic-lamp-script)

- [Azure 上用于 Linux 的 Docker 虚拟机扩展](/documentation/articles/virtual-machines-linux-dockerextension)

<!---HONumber=Mooncake_0314_2016-->