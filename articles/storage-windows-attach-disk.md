<properties
	pageTitle="将磁盘附加到虚拟机 | Microsoft Azure"
	description="了解如何将数据磁盘附加到 Azure 虚拟机并将其初始化，以便它可供使用。"
	services="virtual-machines, storage"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags 
	ms.service="virtual-machines"
	ms.date="07/16/2015"
	wacn.date="09/15/2015"/>

# 如何将数据磁盘附加到 Windows 虚拟机

你可以附加空磁盘和包含数据的磁盘。在这两种情况下，这些磁盘实际上是驻留在 Azure 存储帐户中的 .vhd 文件。此外，也是在这两种情况下，在附加磁盘之后，你将需要对其进行初始化，然后才能使用。

> [AZURE.NOTE]最佳做法是使用一个或多个不同的磁盘来存储虚拟机的数据。当你创建 Azure 虚拟机时，它具有一个映射到 C 驱动器的操作系统磁盘和一个映射到 D 驱动器的临时磁盘。**不要使用 D 驱动器来存储数据**。 顾名思义，它仅提供临时存储。它不提供冗余或备份，因为它不驻留在 Azure 存储空间中。




[AZURE.INCLUDE [howto-attach-disk-windows-linux](../includes/howto-attach-disk-windows-linux.md)]

## <a id="initializeinWS"></a>如何：在 Windows Server 中初始化新的数据磁盘

1. 连接到虚拟机。有关说明，请参阅[如何登录到运行 Windows Server 的虚拟机][logon]。

2. 在你登录虚拟机后，打开“服务器管理器”。在左窗格中，选择“文件和存储服务”。

	![打开服务器管理器](./media/storage-windows-attach-disk/fileandstorageservices.png)

3. 展开菜单并选择“磁盘”。

4. “磁盘”部分列出了磁盘 0、磁盘 1 和磁盘 2。磁盘 0 是操作系统磁盘，磁盘 1 是临时资源磁盘（不应该用于数据存储），磁盘 2 是已附加到虚拟机的数据磁盘。根据你附加磁盘时指定的设置，数据磁盘的容量为 5 GB。右键单击磁盘 2，然后选择“初始化”。

5.	在初始化磁盘时，系统会告知你将要擦除所有数据。单击“是”确认警告并初始化磁盘。然后，再次右键单击磁盘 2，然后选择“新建卷”。

6.	使用默认值完成向导操作。完成向导后，“卷”部分将列出新卷。现在，磁盘处于联机状态并已准备好存储数据。

	![已成功初始化卷](./media/storage-windows-attach-disk/newvolumecreated.png)

> [AZURE.NOTE]虚拟机的大小决定了可以在其上附加多少个磁盘。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs)。

## 其他资源

[如何从 Windows 虚拟机分离磁盘](/documentation/articles/storage-windows-detach-disk)

[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-disks-vhds)

[logon]: /documentation/articles/virtual-machines-log-on-windows-server
<!---HONumber=69-->