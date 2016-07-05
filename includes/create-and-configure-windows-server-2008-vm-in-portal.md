<properties writer="kathydav" editor="tysonn" manager="jeffreyg" />
<tags ms.service=""
    ms.date="12/12/2014"
    wacn.date="04/11/2015"
    /> 

**注意**：本文创建的是不连接到虚拟网络的虚拟机。如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到它或设置跨界连接，请在创建虚拟机时使用“从库中”方法并指定虚拟网络。有关虚拟网络的更多信息，请参见 [Azure 虚拟网络概述](http://go.microsoft.com/fwlink/p/?LinkID=294063)。


可按照以下步骤创建虚拟机：

1. 使用您的 Azure 帐户登录 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在经典管理门户中，在网页的左下角依次单击“+新建”、“虚拟机”，然后单击“从库中”。
	![新建虚拟机][Image1]

3. 选择一个 Windows Server 2008 R2 SP1 虚拟机映像，然后单击页面右下角的下一步箭头。
	
4. 在“虚拟机配置”页上，提供下列信息：

- 提供“虚拟机名称”，例如“testwinvm”。
- 在“新用户名”框中，键入“Administrator”。
- 在“新密码”框中，键入一个[强密码](http://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。
- 在“确认密码”框中，再次键入该密码。
- 从下拉列表中选择适当的“大小”。

	单击下一步箭头以继续。


5. 在“虚拟机模式”页上，提供下列信息：

- 选择“独立虚拟机”。
- 在“DNS 名称”框中，按照格式 testwinvm.chinacloudapp.cn 键入一个有效子域
- 在“区域/地缘组/虚拟网络”框中，选择将承载此虚拟映像的区域。

   单击下一步箭头以继续。

	
6. 在“虚拟机选项”页上，在“可用性集”框中选择“(无)”。单击复选标记以继续。
	

7. 请等候 Azure 准备您的虚拟机。


[Image1]: ./media/create-and-configure-windows-server-2008-vm-in-portal/CreateWinVM.png



