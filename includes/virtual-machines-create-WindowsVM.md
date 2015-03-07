1. 在登录到 Azure 的[管理门户](http://manage.windowsazure.cn)。签出[免费试用版](/pricing/1rmb-trial/) 提供了如果您不具有尚未订阅。

2. 在在窗口底部的命令栏中，单击**新建**。

3. 在下**计算**，单击**虚拟机**，然后单击**从库**。

	![Navigate to From Gallery in the Command Bar](./media/virtual-machines-create-WindowsVM/fromgallery.png)
	
4. 可以使用第一个屏幕**选择一个图像**为虚拟机从一个映像库中的列表。（可用的映像可能不同具体取决于正在使用的订阅。）单击箭头以继续。

	![Choose an image](./media/virtual-machines-create-WindowsVM/chooseimage.png)

5. 第二个屏幕允许您选择计算机名称、 大小和管理权限的用户名和密码。如果您只是想要尝试出 Azure 虚拟机，填写字段下图中所示。否则，选择层和运行您的应用程序或工作负荷所需的大小。下面是一些详细信息，以帮助您填写此： 
	
	- **新用户名**指的是用于管理服务器的管理帐户。创建此帐户的唯一密码，请务必记住它。**您将需要用户名和密码以登录到虚拟机**。 

	- 虚拟机的大小会影响使用它，也可以将附加的数据如磁盘您的配置选项的成本。有关详细信息，请参阅[虚拟机和 Azure 的云服务大小](http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx)。

	![Configure the properties of the virtual machine](./media/virtual-machines-create-WindowsVM/vmconfiguration.png)



6. 第三个屏幕允许您配置的网络、 存储和可用性的资源。下面是一些详细信息，以帮助您填写此： 
	

	- **云服务 DNS 名称**是将成为用于联系虚拟机的 URI 的一部分的全局 DNS 名称。您将需要使用您自己的云服务名称提出，因为它必须是唯一在 Azure 中。云服务非常重要的使用方案[多个虚拟机](/zh-cn/documentation/articles/cloud-services-connect-virtual-machine/)。
 
	- 对于**区域/地缘组/虚拟网络**，使用适合于您所在位置的区域。您还可以选择改为指定的虚拟网络。
 
	>[WACOM.NOTE] 如果您希望虚拟机使用虚拟网络，您**必须**在创建虚拟机时指定虚拟网络。在创建虚拟机后，不能向虚拟网络中加入虚拟机。有关详细信息，请参阅[Azure 虚拟网络概述](http://go.microsoft.com/fwlink/p/?LinkID=294063)。

	- 有关配置终结点的详细信息，请参阅[如何设置终结点到虚拟机](/zh-cn/documentation/articles/virtual-machines-set-up-endpoints/)。

	![Configure the connected resources of the virtual machine](./media/virtual-machines-create-WindowsVM/resourceconfiguration.png)

7. 第四个配置屏幕允许您配置 VM 代理和一些可用的扩展。单击复选标记以创建虚拟机。


	![Configure VM Agent and extensions for the virtual machine](./media/virtual-machines-create-WindowsVM/agent-and-extensions.png)

	>[WACOM.NOTE] VM 代理提供了为您安装扩展可帮助您管理虚拟机或与交互环境。有关详细信息，请参阅[使用扩展](http://msdn.microsoft.com/zh-cn/library/dn606311.aspx)。  
    
8. 管理门户创建虚拟机后，列出在新的虚拟机**虚拟机**。相应的云服务和存储帐户还创建，并在这些部分中列出。自动启动虚拟机和云服务和管理门户中显示其状态为都**运行**。 

	![Configure VM Agent and the endpoints of the virtual machine](./media/virtual-machines-create-WindowsVM/vmcreated.png)
