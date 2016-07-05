
可按照以下步骤附加数据磁盘：

1. 在 [Azure 经典管理门户][AzurePreviewPortal]中，单击“虚拟机”，然后选择刚才创建的虚拟机 (testwinvm)。

2. 在命令栏上，单击“附加”，然后单击“附加空磁盘”。
	
	将显示“将空磁盘附加到虚拟机”对话框。


3. 已为您定义好了“虚拟机名称”、“存储位置”和“文件名”。您只需要输入所需的磁盘大小。在“大小”字段中键入 5。

	![附加空磁盘][Image2]

	**注意**：所有磁盘都是从 Azure 存储中的 VHD 文件创建的。您可以为添加到存储的 VHD 文件提供名称，但 Azure 会自动生成磁盘名称。

4. 单击复选标记以将数据磁盘附加到虚拟机。

5. 单击虚拟机的名称可显示仪表板；这样您可以验证数据磁盘是否已成功附加到虚拟机。

	现在虚拟机的磁盘数为 2 个。“磁盘”表中列出了您附加的磁盘。

	![附加空磁盘][Image3]

	在您将数据磁盘附加到虚拟机后，该磁盘会处于脱机和未初始化状态。您必须先登录虚拟机并初始化磁盘，才能使用该磁盘存储数据。

##使用远程桌面连接到虚拟机并完成安装
1. 配置虚拟机后，在经典管理门户中，单击“虚拟机”，然后单击新的虚拟机。将显示有关您的虚拟机的信息。	

2. 在页面底部，单击“连接”。使用 Windows 远程桌面程序 (*%windir%\system32\mstsc.exe*) 打开 .rpd 文件。	

3. 在“Windows 安全性”对话框中，提供 Administrator 帐户的密码。（系统可能会要求您验证虚拟机的凭据。）您首次登录该虚拟机时，可能需要完成几个过程，包括设置桌面、完成 Windows 更新和 Windows 初始配置任务。当您通过 Windows 远程桌面连接到虚拟机后，该虚拟机就会像任何其他计算机一样开始工作。

4. 在您登录虚拟机后，打开“服务器管理器”。在左侧窗格中，展开“存储”，然后单击“磁盘管理”。

	![服务器管理器][Image4]

5. 将显示“初始化磁盘”窗口。单击“确定”。

	![初始化磁盘][Image5.0]

6. 右键单击磁盘 2 的空间分配区域，单击“新建简单卷”，然后使用默认值完成该向导。

	![新建简单卷][Image6]

	磁盘现在处于联机状态且可以使用新的驱动器号。

	![初始化成功][Image7]


[AzurePreviewPortal]: https://manage.windowsazure.cn

[Image2]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/AttachDataDiskWinVM2.png
[Image3]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/AttachDataDiskWinVM3.png
[Image4]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/servermanager.png
[Image5.0]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/initializedisk0.png

[Image6]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/initializediskvolume.png
[Image7]: ./media/attach-data-disk-windows-server-2008-vm-in-portal/initializesuccess.png


