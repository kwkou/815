<properties linkid="manage-linux-howto-attach-a-disk" urlDisplayName="Attach a disk" pageTitle="将磁盘附加到虚拟机 | Azure" metaKeywords="Azure Linux virtual machine, Azure Windows virtual machine, Azure attach disk, Azure initialize disk" description="了解如何将数据磁盘附加到 Azure 虚拟机。然后在 Windows Server 或 Linux 虚拟机中初始化该磁盘。" metaCanonical="" services="virtual-machines,storage" documentationCenter="" title="How to Attach a Data Disk to a Virtual Machine" authors="" solutions="" manager="" editor="" />


#如何将数据磁盘附加到 Windows 虚拟机

你可以附加空磁盘和包含数据的磁盘。在这两种情况下，这些磁盘实际上是驻留在 Azure 存储帐户中的 .vhd 文件。此外，也是在这两种情况下，在附加磁盘之后，你将需要对其进行初始化，然后才能使用。 

> [WACOM.NOTE] 最佳做法是使用一个或多个不同的磁盘来存储虚拟机的数据。当你创建 Azure 虚拟机时，它具有一个映射到 C 驱动器的操作系统磁盘和一个映射到 D 驱动器的临时磁盘。**不要使用 D 驱动器来存储数据。**顾名思义，它仅提供临时存储。它不提供冗余或备份，因为它不驻留在 Azure 存储空间中。

- [如何：附加空磁盘](#attachempty)
- [如何：附加现有磁盘](#attachexisting)
- [如何：在 Windows Server 中初始化新的数据磁盘](#initializeinWS)


[WACOM.INCLUDE [howto-attach-disk-windows-linux](../includes/howto-attach-disk-windows-linux.md)]

##<a id="initializeinWS"></a>如何：在 Windows Server 中初始化新的数据磁盘

1. 使用[如何登录到运行 Windows Server 的虚拟机][logon]中列出的步骤连接到虚拟机。



2. 在登录之后，打开"服务器管理器"，在左侧窗格中展开"存储"，然后单击"磁盘管理"。



	![Open Server Manager](./media/storage-windows-attach-disk/ServerManager.png)



3. 右键单击**"磁盘 2"**，单击**"初始化磁盘"**，然后单击**"确定"**。



	![Initialize the disk](./media/storage-windows-attach-disk/InitializeDisk.png)


4. 右键单击磁盘 2 的空间分配区域，单击"新建简单卷"，然后使用默认值完成该向导。
 

	![Initialize the volume](./media/storage-windows-attach-disk/InitializeDiskVolume.png)


[logon]: ../virtual-machines-log-on-windows-server/



	磁盘现在处于联机状态且可以使用新的驱动器号。



	![已成功初始化卷](./media/storage-windows-attach-disk/InitializeSuccess.png)
<!--HONumber=41-->
