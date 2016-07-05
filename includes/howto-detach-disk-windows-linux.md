<properties writer="kathydav" editor="tysonn" manager="timlt" />

当你不再需要附加到虚拟机的数据磁盘时，你可以轻松地分离它。这将从虚拟机中删除磁盘，但不会从存储中删除它。

若果你希望再次使用磁盘上的现有数据，可以将其重新附加到相同的虚拟机或另一个虚拟机。

> [AZURE.NOTE]除非你同时也删除虚拟机，否则不能分离操作系统磁盘。


## 找到磁盘

如果你不知道磁盘名称或要在分离磁盘之前验证磁盘名称，请按照以下步骤进行操作。


1. 如果你尚未登录 [Azure 经典管理门户](http://manage.windowsazure.cn)，请先登录。

2. 依次单击“虚拟机”、虚拟机名称和“仪表板”。

3. 在“磁盘”下，表格将列出所有附加的磁盘的名称和类型。例如，此屏幕显示带有一个操作系统 (OS) 磁盘和一个数据磁盘的虚拟机：

	![查找数据磁盘](./media/howto-detach-disk-windows-linux/FindDataDisks.png)


## 分离磁盘

1. 单击“虚拟机”，单击其数据磁盘需要进行分离的虚拟机的名称，然后单击“仪表板”。

2. 从命令栏中，单击“分离磁盘”。

3. 选择数据磁盘，然后单击复选标记以分离该磁盘。

	![分离磁盘详细信息](./media/howto-detach-disk-windows-linux/DetachDiskDetails.png)

磁盘保留在存储中，但不再附加到虚拟机。

<!---HONumber=Mooncake_1207_2015-->