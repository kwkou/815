<properties pageTitle="如何更改 Windows 临时磁盘的驱动器号" description="介绍了如何重新映射 Azure 中 Windows VM 上的临时磁盘" metaKeywords="" services="virtual machines" documentationCenter="" authors="kathydav"  manager="timlt" />
<tags  
	ms.service="virtual-machines"
	ms.date="05/27/2015"
	wacn.date="08/29/2015"/>

#如何更改 Windows 临时磁盘的驱动器号

如果需要使用 D 盘存储数据，请遵循这些说明操作以使用其他驱动器作为临时磁盘。切勿使用临时磁盘来存储需要保存的数据。

在开始之前，需要有一个附加到虚拟机的数据磁盘，以便在此过程中存储 Windows 页面文件 (pagefile.sys)。如果没有，请参阅[如何将数据磁盘附加到 Windows 虚拟机][Attach]。有关如何查看已附加磁盘的说明，请参阅[如何从 Windows 虚拟机分离数据磁盘][Detach]中的“查找磁盘”。

如果要使用 D 盘上的现有数据磁盘，请确保将 VHD 也上载到存储帐户。有关说明，请参阅[创建 Windows Server VHD 并将其上载到 Azure][VHD] 中的步骤 3 和 4。

> [AZURE.WARNING]如果调整虚拟机大小并将虚拟机移动到其他主机，则临时磁盘将重新更改回 D 盘。

##更改驱动器号

1. 登录到虚拟机。有关详细信息，请参阅[如何登录到运行 Windows Server 的虚拟机][Logon]。

2. 将 pagefile.sys 从 D 盘移动到其他驱动器。

3. 重启虚拟机。

4. 	重新登录，并将驱动器号从 D 改为 E。

5.	从 [Azure 门户](http://manage.windowsazure.cn)中，附加现有数据磁盘或空的数据磁盘。

6.	重新登录到虚拟机，对磁盘进行初始化，并为刚才附加的磁盘指定驱动器号为 D。

7.	验证 E 是否映射到临时磁盘。

8.	将 pagefile.sys 从其他驱动器移动到 E 盘。

##其他资源
[如何登录到运行 Windows Server 的虚拟机][Logon]

[如何从 Windows 虚拟机分离数据磁盘][Detach]

[关于 Azure 存储帐户][Storage]

<!--Link references-->
[Attach]: /documentation/articles/storage-windows-attach-disk
[VHD]: /documentation/articles/virtual-machines-create-upload-vhd-windows-server
[Logon]: /documentation/articles/virtual-machines-log-on-windows-server
[Detach]: /documentation/articles/storage-windows-detach-disk
[Storage]: /documentation/articles/storage-whatis-account
<!---HONumber=67-->