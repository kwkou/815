<properties writer="kathydav" editor="tysonn" manager="jeffreyg" /> 

**注意**：本文创建的是不连接到虚拟网络的虚拟机。如果您希望您的虚拟机使用虚拟网络，以便可以按主机名直接连接到
它或设置跨界连接，请在创建虚拟机时使用“从库中”方法并
指定虚拟网络。有关虚拟网络的更多信息，请参见 [Windows Azure 虚拟网络概述](http://go.microsoft.com/fwlink/p/?LinkID=294063)。

1. 使用您的 Windows Azure 帐户登录 Windows Azure 管理门户。
2. 在管理门户中，在网页的左下角依次单击“+新建”、“虚拟机”，然后单击“从库中”。

	![新建虚拟机][Image1]

3. 从“平台映像”中选择一个 CentOS 虚拟机映像，然后单击页面右下角的下一步箭头。
	
4. 在“虚拟机配置”页上，提供下列信息：
	- 提供“虚拟机名称”，例如“testlinuxvm”。
	- 指定将添加到 Sudoers 列表文件中的“新用户名”，例如“newuser”。
	- 在“新密码”框中，键入一个[强密码](http://msdn.microsoft.com/zh-cn/library/ms161962.aspx)。
	- 在“确认密码”框中，再次键入该密码。
	- 从下拉列表中选择适当的“大小”。

	单击下一步箭头以继续。
	
5. 在“虚拟机模式”页上，提供下列信息：
	- 选择“独立虚拟机”。
	- 在“DNS 名称”框中，键入有效的 DNS 地址。例如，“testlinuxvm”
	- 在“存储帐户”框中，选择“使用自动生成的存储帐户”。
	- 在“区域/地缘组/虚拟网络”框中，选择将承载此虚拟映像的区域。

	单击下一步箭头以继续。

6. 在“虚拟机选项”页上，在“可用性集”框中选择“(无)”。

	单击复选标记以继续。
	
7. 请等候 Windows Azure 准备您的虚拟机。

##配置终结点
创建虚拟机后，请配置终结点以进行远程连接。

1. 在管理门户中，依次单击“虚拟机”、您的新虚拟机的名称和“终结点”。

2. 单击页面底部的“编辑终结点”，并编辑 SSH 终结点以使其“公用端口”为 22。

##连接到虚拟机
当设置虚拟机和配置终结点后，可以使用 SSH 或 PuTTY 连接到虚拟机。

###使用 SSH 进行连接
如果您使用的是 Linux，请使用 SSH 连接到虚拟机。在命令提示符处，运行：

	$ ssh newuser@testlinuxvm.cloudapp.net -o ServerAliveInterval=180

输入用户的密码。

###使用 PuTTY 进行连接
如果您使用的是 Windows 计算机，请使用 PuTTY 连接到 VM。可从 [PuTTY 下载页][PuTTYDownLoad]下载 PuTTY。

1. 将 putty.exe 下载并保存到您的计算机上的某个目录。打开命令提示符，导航到该文件夹，然后执行 putty.exe。

2. 为“主机名”输入“testlinuxvm.cloudapp.net”，为“端口”输入“22”。
![PuTTY 屏幕][Image6]

##更新虚拟机（可选）
在连接到虚拟机后，您可以选择安装更新。运行：

	$ sudo yum update

再次输入密码。安装更新时，请等待。


[PuTTYDownload]: http://www.puttyssh.org/download.html

[Image1]: ./media/create-and-configure-centos-vm-in-portal/CreateVM.png

[Image6]: ./media/create-and-configure-centos-vm-in-portal/putty.png

