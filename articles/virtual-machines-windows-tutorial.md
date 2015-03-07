<properties linkid="manage-windows-tutorial-virtual-machine-gallery" urlDisplayName="Create a virtual machine" pageTitle="创建在 Azure 中运行 Windows Server 的虚拟机" metaKeywords="Azure capture image vm, capturing vm" description="了解如何捕获运行 Windows Server 2008 R2 的 Azure 虚拟机 (VM) 的映像。 " metaCanonical="" services="virtual-machines" documentationCenter="" title="" authors="kathydav" solutions="" manager="jeffg" editor="tysonn" />




# 创建运行 Windows 的虚拟机 #

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/documentation/articles/virtual-machines-windows-tutorial/" title="Azure Portal" class="current">Azure 门户</a></div>

本教程演示创建 Azure 虚拟机 (VM) 运行 Windows，例如使用 Windows Server 映像在 Azure 管理门户的映像库是多么容易。映像库提供了各种各样的映像，包括 Windows 操作系统、基于 Linux 的操作系统和应用程序映像。 

> [WACOM.NOTE] 您不需要使用 Azure 虚拟机，若要完成本教程中的任何经验。但是，您需要一个 Azure 帐户。只需几分钟即可创建一个免费的试用帐户。有关详细信息，请参阅[创建 Azure 帐户](/zh-cn/develop/php/tutorials/create-a-windows-azure-account/). 

本教程演示：

- [如何创建虚拟机](#createvirtualmachine)
- [如何登录到虚拟机创建表后](#logon)
- [如何将数据磁盘附加到新的虚拟机](#attachdisk)

如果您想要了解详细信息，请参阅[虚拟机](http://msdn.microsoft.com/library/azure/jj156003.aspx)。


##<a id="createvirtualmachine"> </a>如何创建虚拟机 # #

本部分演示如何使用**从库**在管理门户中的选项创建虚拟机。此选项提供更多的配置选项比**快速创建**选项。例如，如果您想要将虚拟机加入到虚拟网络，您将需要使用**从库**选项。

> [WACOM.NOTE] 多少和什么样的映像库中有取决于您所拥有的订阅的类型。本教程使用的 Windows Server 映像，但 MSDN 订阅可以具有可用的包括桌面映像的其他图像。 
<!--
> 您还可以尝试更丰富、 可自定义[Azure 预览版门户](https://portal.azure.com)若要创建虚拟机时，自动部署多台计算机的应用程序模板，使用增强型 VM 监视和诊断功能，以及更多内容。两个门户网站中可用的虚拟机配置选项大体上重叠，但并不完全相同。  
-->
[WACOM.INCLUDE [虚拟-机-创建-WindowsVM](../includes/virtual-machines-create-WindowsVM.md)]

## <a id="logon"> </a>如何登录到虚拟机创建表后 # #

本部分演示了如何登录到虚拟机以便您可以管理其设置和应用程序将在其上运行。

[WACOM.INCLUDE [virtual-machines-log-on-win-server](../includes/virtual-machines-log-on-win-server.md)]

## <a id="attachdisk"> </a>如何将数据磁盘附加到新的虚拟机 # #

本部分演示了如何将空数据磁盘附加到虚拟机。请参阅[附加数据磁盘教程](/zh-cn/documentation/articles/storage-windows-attach-disk/) 有关附加空磁盘以及如何附加现有磁盘的详细信息。

1. 在登录到 Azure 的[管理门户](http://manage.windowsazure.cn)。

2. 单击**虚拟机**，然后选择**MyTestVM**虚拟机。

	![Select MyTestVM](./media/virtual-machines-windows-tutorial/selectvm.png)
	
3. 您可能会转到快速启动页第一个。如果是这样，则选择**仪表板**从顶部。

	![Select Dashboard](./media/virtual-machines-windows-tutorial/dashboard.png)

4. 在命令栏中，单击**附加**，然后单击**附加空磁盘**时它会弹出。

	![Select Attach from the command bar](./media/virtual-machines-windows-tutorial/commandbarattach.png)	

5. **虚拟机名称**，**存储位置**，**文件名**，和**主机缓存首选项**已为您定义。您所要做的就是输入所需的磁盘的大小。类型**5**中**大小**字段。然后单击复选标记以将空磁盘附加到虚拟机。

	>[WACOM.NOTE] 值得指出的在 Azure 中的磁盘映像在 Azure 存储中作为页 blob storeed。在 Azure 中，外部虚拟硬盘可以使用 VHD 或 VHDX 格式。它们还可以被固定，动态扩展或差异。Azure 支持 VHD 格式的固定磁盘。固定的格式的逻辑磁盘以线性方式布局在文件中，因此，该磁盘偏移量 X 存储在 blob 偏移量 X。在 blob 末尾的一个小的页脚描述了 VHD 的属性。通常情况下，因为大多数磁盘中它们具有较大的未使用的区域固定的格式会浪费空间。但是，Azure 存储.vhd 文件以稀疏格式，以便在同一时间接收固定和动态磁盘的优点。您可以阅读更多有关在此[有关 Vhd 在 Azure 中](http://msdn.microsoft.com/zh-cn/library/azure/dn790344.aspx)  


	![Specify the size of the empty disk](./media/virtual-machines-windows-tutorial/emptydisksize.png)	
	
	>[WACOM.NOTE] 所有磁盘都是从 Windows Azure 存储中的 VHD 文件都创建的。在下**文件名**，您可以添加到存储的 VHD 文件的名称，但 Azure 会自动生成的磁盘的名称。 

6. 返回到仪表板以验证空数据磁盘已成功附加到虚拟机。它将列为中的第二个磁盘**磁盘**以及操作系统磁盘的列表。

	![Attach empty disk](./media/virtual-machines-windows-tutorial/disklistwithdatadisk.png)

	将数据磁盘附加到虚拟机后，该磁盘是脱机和未初始化状态。可以使用它来存储数据之前，您需要登录到虚拟机并初始化该磁盘。

7. 通过使用上一节中的步骤连接到虚拟机[如何登录到虚拟机创建表后] (#logon).

8. 在你登录到虚拟机后，打开**服务器管理器**。在左窗格中，选择**文件和存储服务**。

	![Expand File and Storage Services in Server Manager](./media/virtual-machines-windows-tutorial/fileandstorageservices.png)

9. 选择**磁盘**从展开的菜单。

	![Expand File and Storage Services in Server Manager](./media/virtual-machines-windows-tutorial/selectdisks.png)	
	
10. 在**磁盘**部分中，在列表中有三个磁盘： 磁盘 0、 磁盘 1，和磁盘 2。磁盘 0 是操作系统磁盘、 磁盘 1 是临时资源磁盘 （它不应使用的数据存储），和磁盘 2 是已附加到虚拟机的数据磁盘。请注意数据磁盘前面有指定的容量为 5 GB。右键单击磁盘 2，然后选择**初始化**。

	![Start initialization](./media/virtual-machines-windows-tutorial/initializedisk.png)

11. 单击**是**若要开始初始化过程。

	![Continue initialization](./media/virtual-machines-windows-tutorial/yesinitialize.png)

12. 再次右键单击磁盘 2 并选择**新卷**。 

	![Create the volume](./media/virtual-machines-windows-tutorial/initializediskvolume.png)

13. 完成该向导使用所提供的默认值。当向导完成后时，在中列出新的卷**卷**部分。 

	![Create the volume](./media/virtual-machines-windows-tutorial/newvolumecreated.png)

	磁盘现在处于联机状态并且准备使用新的驱动器号。 
	
##后续步骤 

若要了解有关在 Azure 上配置 Windows 虚拟机的详细信息，请参阅这些文章：

[如何连接云服务中的虚拟机](/zh-cn/documentation/articles/cloud-services-connect-virtual-machine/)

[如何创建和上载您自己包含 Windows Server 操作系统的虚拟硬盘](/zh-cn/documentation/articles/virtual-machines-create-upload-vhd-windows-server/)

[管理虚拟机的可用性](/zh-cn/documentation/articles/manage-availability-virtual-machines/)

[有关 Azure 虚拟机配置设置](http://msdn.microsoft.com/library/azure/dn763935.aspx)

[视频：Getting Started with Vhd-真实情况什么](/zh-cn/documentation/videos/getting-started-with-azure-virtual-machines)

[视频：常见问题与 Mark Russinovich-没有 Windows Azure 中运行 Windows 吗？](/zh-cn/documentation/videos/mark-russinovich-windows-on-azure)

[视频：将新的虚拟机添加到 Web 场中，通过创建可重复使用的映像](/zh-cn/documentation/videos/adding-virtual-machines-web-farm)

[视频：添加虚拟硬盘驱动器、 存储帐户和缩放的虚拟机](/zh-cn/documentation/videos/adding-drives-scaling-virtual-machines)

[视频：Scott Guthrie 开头的虚拟机](/zh-cn/documentation/videos/virtual-machines-scottgu)

[视频：存储和磁盘对于 Azure 虚拟机的基础知识](/zh-cn/documentation/videos/storage-and-disks-virtual-machines)



[有关在 Azure 中的虚拟机]: #virtualmachine
[如何创建虚拟机]: #custommachine
[如何登录到虚拟机创建表后]: #logon
[如何将数据磁盘附加到新的虚拟机]: #attachdisk
[如何设置与虚拟机的通信]: #endpoints
