<properties
	pageTitle="对删除 Azure 存储帐户、容器或 VHD 进行故障排除 | Azure"
	description="对删除 Azure 存储帐户、容器或 VHD 进行故障排除"
	services="storage"
	documentationCenter=""
	authors="genlin"
	manager="felixwu"
	editor=""
	tags="storage"/>

<tags
	ms.service="storage"
	ms.date="03/20/2016"
	wacn.date="04/11/2016"/>

# 对删除 Azure 存储帐户、容器或 VHD 进行故障排除

## 摘要
当你尝试在 [Azure 管理门户](https://manage.windowsazure.cn/)中删除 Azure 存储帐户、容器或的 VHD 时，可能会收到错误。这些问题可能由以下原因引起：

-	删除 VM 时没有自动删除磁盘和 VHD，这可能是存储帐户删除失败的原因。我们不删除磁盘，以便你可以使用该磁盘安装另一个 VM。

-	磁盘或磁盘上关联的 Blob 上仍有租约。



## 解决方法
若要解决最常见的问题，请尝试以下方法：

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)。
2. 选择“虚拟机”>“磁盘”。

	![disk.png](./media/storage-cannot-delete-storage-account-container-vhd/VMUI.png)

3. 找到与你想要删除的存储帐户、容器或 VHD 关联的磁盘。检查磁盘的位置，你将找到关联的存储帐户、容器和 VHD。

	![location](./media/storage-cannot-delete-storage-account-container-vhd/DiskLocation.png)

4. 确认没有任何 VM 列于磁盘的“附加到”字段，然后删除磁盘。

 	> [AZURE.NOTE] 如果磁盘附加到 VM，你将不能将其删除。磁盘从已删除的 VM 异步分离，在删除 VM 后，清除此字段可能需要几分钟。

5. 选择“虚拟机”>“映像”，然后删除与存储帐户、容器或 VHD 关联的映像。

之后，再次尝试删除存储帐户、容器或 VHD。

> [AZURE.WARNING] 请在删除帐户之前务必备份你想要保存的任何内容。无法恢复已删除的存储帐户，也无法检索删除之前该存储帐户包含的任何内容。对于帐户中的任务资源也是如此 - 一旦你删除了一个 VHD、Blob、表、队列或文件，则它将被永久删除。请确保该资源未使用。

## 症状

以下部分列出了在试图删除 Azure 存储帐户、容器或 VHD 时可能收到的常见错误，请参阅：

### 场景 1：无法删除存储帐户

在通过导航到 [Azure 管理门户](https://manage.windowsazure.cn/)中的存储帐户试图删除不再使用的存储帐户并选择“删除”时，你可能会看到以下错误消息：


**在 Azure 管理门户中**：

存储帐户<vm-storage-account-name>具有一些活动的映像和/或磁盘，例如 xxxxxxxxx-xxxxxxxxx-O-209490240936090599。删除此存储帐户前，请确保删除这些映像和/或磁盘。

你可能也会看到此错误：


**在 Azure 管理门户中**：

提交失败的存储帐户<vm-storage-account-name>拥有 1 个具有活动的映像和/或磁盘项目的容器。在删除此存储帐户前，请确保从映像存储库中删除这些项目。
当你试图删除某个存储帐户，但仍存在与之关联的活动磁盘时，你将看到一条告诉你有活动磁盘需要进行删除的消息。

### 场景 2：无法删除容器

当你试图删除存储容器时，可能会看到以下错误：

未能删除存储容器 <container name>。错误：目前容器上有租约，但请求中未指定任何租约 ID。

### 场景 3：无法删除 VHD

在删除 VM 后，你试图删除关联 VHD 的 Blob 并收到以下消息：

未能删除 Blob “路径/XXXXXX-XXXXXX os-1447379084699.vhd”。错误：目前 Blob 上有租约，但请求中未指定任何租约 ID。

## 详细信息

已保留的 V1 VM 将如 [Azure 管理门户](https://manage.windowsazure.cn/)上的已停止取消分配状态中所示。



**Azure 管理门户**：

![screenshot2](./media/storage-cannot-delete-storage-account-container-vhd/moreinfo2.png)

“已停止（取消分配）”状态释放了计算资源（如 CPU、内存和网络），但仍会保留磁盘以便用户在需要时能够快速重新创建 VM。这些磁盘都基于 Azure 存储空间支持的 VHD 创建。存储帐户拥有这些 VHD 并且磁盘在这些 VHD 上有租约。

## 参考

- [删除存储帐户](/documentation/articles/storage-create-storage-account/#delete-a-storage-account)
- [如何在 Azure（PowerShell）中中断 Blob 存储的锁定租约](https://gallery.technet.microsoft.com/scriptcenter/How-to-break-the-locked-c2cd6492)

<!---HONumber=Mooncake_0405_2016-->