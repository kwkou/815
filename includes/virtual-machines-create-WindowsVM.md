1. 登录到[经典管理门户](http://manage.windowsazure.cn)。如果你尚未获取订阅，请查看[试用](/pricing/1rmb-trial/)优惠产品。

2. 在窗口底部的命令栏上，单击“新建”。

3. 在“计算”下，单击“虚拟机”，然后单击“从库中”。

	![在命令栏中导航到“从库中”](./media/virtual-machines-create-WindowsVM/fromgallery.png)

4. 随后出现的第一个屏幕可让你从可用映像列表中为虚拟机**选择映像**。可以从库中选择映像，也可以从已上载的映像和磁盘中选择。可用映像可能各不相同，具体取决于你使用的订阅。

5. 第二个屏幕可让你选择计算机名称、大小和管理用户名与密码。使用所需的层和大小运行你的应用或工作负载。下面是一些提示：

	- **虚拟机名称**只能包含字母、数字和连字符。它还必须以字母开头，并以字母或数字结尾。
	- “新用户名”指用于管理服务器的管理帐户。密码的长度必须是 8 到 123 个字符，并且必须至少包含 3 个以下字符：1 个小写字符、1 个大写字符、1 个数字和 1 个特殊字符。**你需要使用用户名和密码登录到虚拟机**。
	- 虚拟机的大小会影响其使用成本，还会影响某些配置选项，例如，可以附加的数据磁盘数。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

6. 第三个屏幕可让你配置网络资源、存储和可用性。下面是一些提示：

	- “云服务 DNS 名称”是全局 DNS 名称，是用于联系虚拟机的 URI 的一部分。你需要指定自己的云服务名称，因为该名称在 Azure 中必须唯一。对于使用[多个虚拟机](/documentation/articles/virtual-machines-windows-classic-connect-vms/)的方案而言，云服务非常重要。

	- 对于“区域/地缘组/虚拟网络”，请使用适合你所在位置的区域。您也可以选择指定一个虚拟网络。

	>[AZURE.NOTE] 如果你希望虚拟机使用虚拟网络，则**必须**在创建虚拟机时指定虚拟网络。创建虚拟机后，不能将它加入虚拟网络。有关详细信息，请参阅 [Azure 虚拟网络概述](/documentation/articles/virtual-networks-overview/)。
	> <p>有关配置终结点的详细信息，请参阅[如何设置虚拟机的终结点](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。

7. 第四个配置屏幕可让你安装 VM 代理，以及配置某些可用的扩展。

	>[AZURE.NOTE] VM 代理为你提供安装扩展的环境，可帮助你与虚拟机交互或管理虚拟机。有关详细信息，请参阅[关于 VM 代理和扩展](/documentation/articles/virtual-machines-windows-classic-agents-and-extensions/)。

8. 创建虚拟机之后，经典管理门户将在“虚拟机”下列出新虚拟机。此外，还会创建相应的云服务和存储帐户，并将其列在这些部分中。虚拟机和云服务都会自动启动，其状态将显示为“正在运行”。

	![配置虚拟机的 VM 代理和终结点](./media/virtual-machines-create-WindowsVM/vmcreated.png)

<!---HONumber=Mooncake_0215_2016-->