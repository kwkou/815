# 创建运行 Windows Server 的虚拟机 #

本教程将向您演示使用 Azure 在云中创建运行 Windows Server 的虚拟机的快速简单方法。我们将使用 Azure 经典管理门户中的映像库，它包含各种不同映像，例如基于 Windows 的操作系统、基于 Linux 的操作系统、应用程序映像等。您无需具备 Azure 使用经验即可使用本教程。 

您将了解到以下内容：

- [如何创建虚拟机](#createvirtualmachine)
- [虚拟机创建后如何进行登录](#logon)
- [如何将数据磁盘附加到新的虚拟机](#attachdisk)

如果您希望了解详细信息，请参阅[虚拟机](http://go.microsoft.com/fwlink/p/?LinkID=271224)。


##<a id="createvirtualmachine"></a>如何创建虚拟机##

本部分演示如何使用 Azure 经典管理门户中的"从库中"方法创建自定义虚拟机。此方法提供比"快速创建"方法更多的选项，在您创建虚拟机时对其进行配置。


1. 登录到 Azure [经典管理门户](http://manage.windowsazure.cn). 如果你还没有订阅，请查看[试用版](/pricing/free-trial)。

2. 在屏幕底部的命令栏上，单击"新建"。

	![从命令栏中选择"新建"](./media/CreateVirtualMachineWindowsTutorial/commandbarnew.png)

3. 在"计算"下，单击"虚拟机"，然后单击"从库中"。

	![在命令栏中导航到"从库中"](./media/CreateVirtualMachineWindowsTutorial/fromgallery.png)
	
4. 在第一个配置屏幕中，您可从映像库的某个列表中，为您的虚拟机"选择映像"。（可用映像可能各不相同，具体取决于您使用的订阅。）在本教程中，我们将选择 "Windows Server 2012 R2 Datacenter"。单击箭头以继续。

	![选择映像](./media/CreateVirtualMachineWindowsTutorial/chooseimage.png)

5. 在第二个配置屏幕中，您可以指定**虚拟机本身的属性**。在本教程中，请按下图所示填写字段。在本屏幕中填写完毕之后，请单击箭头继续。

	![配置虚拟机的属性](./media/CreateVirtualMachineWindowsTutorial/vmconfiguration.png)

	>[WACOM.NOTE]"新用户名"指将用于管理服务器的管理帐户。您需要为此帐户创建您自己的唯一密码。

6. 在第三个配置屏幕中，您可为**连接到虚拟机的资源**（例如云服务和存储帐户）指定属性。在本教程中，请按下图所示填写字段。在本屏幕中填写完毕之后，请单击箭头继续。 

	![配置虚拟机的连接资源](./media/CreateVirtualMachineWindowsTutorial/resourceconfiguration.png)

	请注意，**云服务 DNS 名称**是全局 DNS 名称，是用于联系虚拟机的 URI 的一部分。由于云服务名称必须是全局唯一的，因此您必须创建自己的云服务名称。在本教程中，我们将使用 MyTestService 作为服务名称。对于使用[多个虚拟机](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-linux-classic-connect-vms/)的更复杂方案而言，云服务非常重要。
 
	对于"区域/地缘组/虚拟网络"，我们将使用"美国东部"，但您可以使用更适合您所在位置的区域。您也可以选择指定一个虚拟网络。
 
	>[WACOM.NOTE]如果您希望虚拟机使用虚拟网络，则必须在创建虚拟机时指定虚拟网络。有关详细信息，请参阅 [Azure 虚拟网络概述](http://go.microsoft.com/fwlink/p/?LinkID=294063)。

7. 在第四个配置屏幕中，您可以配置 **VM 代理和终结点**。在本教程中，不要在此屏幕上进行任何更改。单击复选标记可创建虚拟机。


	![配置虚拟机的 VM 代理和终结点](./media/CreateVirtualMachineWindowsTutorial/endpointconfiguration.png)

	>[WACOM.NOTE] VM 代理为您提供安装扩展的环境，可帮助您与虚拟机交互。有关详细信息，请参见[使用扩展](http://msdn.microsoft.com/zh-cn/library/dn606311.aspx)。有关配置终结点的详细信息，请参阅[如何设置虚拟机的终结点](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。
    
8. 创建虚拟机之后，经典管理门户将在"虚拟机"下列出新虚拟机。相应的云服务和存储帐户也在相应部分创建。创建完成之后，虚拟机和云服务都会自动启动，其状态将显示为"正在运行"。 

	![配置虚拟机的 VM 代理和终结点](./media/CreateVirtualMachineWindowsTutorial/vmcreated.png)

## <a id="logon"> </a>虚拟机创建后如何进行登录 ##

本部分将演示如何登录到您创建的虚拟机以管理其设置和在其上运行的应用程序。

1. 登录到 Azure [经典管理门户](http://manage.windowsazure.cn)。

2. 单击"虚拟机"，然后选择 **MyTestVM** 虚拟机。

	![选择 MyTestVM](./media/CreateVirtualMachineWindowsTutorial/selectvm.png)

3. 在命令栏中，单击"连接"。

	![连接到 MyTestVM](./media/CreateVirtualMachineWindowsTutorial/commandbarconnect.png)
	
4. 单击"打开"以使用为虚拟机自动创建的远程桌面协议文件。

	![打开 rdp 文件](./media/CreateVirtualMachineWindowsTutorial/openrdp.png)
	
5. 单击"连接"。

	![继续连接](./media/CreateVirtualMachineWindowsTutorial/connectrdc.png)

6. 在密码框中，键入创建虚拟机时指定的用户名和密码，然后单击"确定"。

7. 单击"是"以验证虚拟机的标识。

	![验证计算机的标识](./media/CreateVirtualMachineWindowsTutorial/certificate.png)

	您现在可以像使用您办公室中的服务器一样使用虚拟机。

## <a id="attachdisk"> </a>如何将数据磁盘附加到新虚拟机 ##

本部分将演示如何将空数据磁盘附加到虚拟机。请参阅[附加数据磁盘教程](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-windows-classic-attach-disk/)。

1. 登录到 Azure [经典管理门户](http://manage.windowsazure.cn)。

2. 单击"虚拟机"，然后选择 **MyTestVM** 虚拟机。

	![选择 MyTestVM](./media/CreateVirtualMachineWindowsTutorial/selectvm.png)
	
3. 您可能首先进入"快速启动"页。如果是这样，请从顶部选择"仪表板"。

	![选择"仪表板"](./media/CreateVirtualMachineWindowsTutorial/dashboard.png)

4. 在命令栏上，单击"附加"，然后在弹出窗口时单击"附加空磁盘"。

	![从命令栏选择"附加"](./media/CreateVirtualMachineWindowsTutorial/commandbarattach.png)	

5. 其中已经为您定义好了"虚拟机名称"、"存储位置"、"文件名"和"主机缓存首选项"。您只需要输入所需的磁盘大小。在"大小"字段中键入 **5**。然后单击复选标记，以便将空磁盘附加到虚拟机。

	![指定空磁盘的大小](./media/CreateVirtualMachineWindowsTutorial/emptydisksize.png)	
	
	>[WACOM.NOTE] 所有磁盘都是从 Azure 存储中的 VHD 文件创建的。在"文件名"下，您可以为添加到存储的 VHD 文件提供名称，但是 Azure 会自动生成磁盘名称。

6. 返回仪表板以验证空数据磁盘是否已成功附加到虚拟机。该磁盘将在"磁盘"列表中，作为第二个磁盘与操作系统磁盘一同列出。

	![附加空磁盘](./media/CreateVirtualMachineWindowsTutorial/disklistwithdatadisk.png)

	在您将数据磁盘附加到虚拟机后，该磁盘会处于脱机和未初始化状态。您需要先登录虚拟机并初始化磁盘，然后才能使用它存储数据。

7. 使用在上一节[创建虚拟机后如何登录](#logon)中列出的步骤连接到虚拟机。

8. 在您登录虚拟机后，打开"服务器管理器"。在左窗格中，选择"文件和存储服务"。

	![在服务器管理器中展开"文件和存储服务"](./media/CreateVirtualMachineWindowsTutorial/fileandstorageservices.png)

9. 从展开的菜单中选择"磁盘"。

	![在服务器管理器中展开"文件和存储服务"](./media/CreateVirtualMachineWindowsTutorial/selectdisks.png)	
	
10. 在"磁盘"部分中，列表中存在三个磁盘：磁盘 0、磁盘 1 和磁盘 2。磁盘 0 是操作系统磁盘，磁盘 1 是临时资源磁盘（不应用于数据存储），磁盘 2 是已附加到虚拟机的数据磁盘。请注意，数据磁盘的容量是以前指定的 5 GB。右键单击磁盘 2，然后选择"初始化"。

	![开始初始化](./media/CreateVirtualMachineWindowsTutorial/initializedisk.png)

11. 单击"是"开始初始化过程。

	![继续初始化](./media/CreateVirtualMachineWindowsTutorial/yesinitialize.png)

12. 再次右键单击磁盘 2，然后选择"新建卷"。 

	![创建卷](./media/CreateVirtualMachineWindowsTutorial/initializediskvolume.png)

13. 使用提供的默认值完成向导操作。向导操作完成之后，将在"卷"部分列出一个新卷。 

	![创建卷](./media/CreateVirtualMachineWindowsTutorial/newvolumecreated.png)

	磁盘现在处于联机状态且可以使用新的驱动器号。 
	
##后续步骤 

若要了解有关 Azure 上 Windows 虚拟机的详细信息，请参阅以下文章：

[如何连接云服务中的虚拟机](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-linux-classic-connect-vms/)

[如何创建和上载您自己的包含 Windows Server 操作系统的虚拟硬盘](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-windows-classic-createupload-vhd/)

[将数据磁盘附加到虚拟机](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-windows-classic-attach-disk/)

[管理虚拟机的可用性](http://azure.microsoft.com/zh-cn/documentation/articles/virtual-machines-windows-manage-availability/)

[有关 Azure 中的虚拟机]: #virtualmachine
[如何创建虚拟机]: #custommachine
[虚拟机创建后如何进行登录]: #logon
[如何将数据磁盘附加到新的虚拟机]: #attachdisk
[如何设置与虚拟机的通信]: #endpoints


<!--HONumber=41-->
