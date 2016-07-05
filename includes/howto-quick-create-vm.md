<properties writer="kathydav" editor="tysonn" manager="jeffreyg" />
<tags ms.service=""
    ms.date="12/31/2014"
    wacn.date="04/11/2015"
    /> 

#如何快速创建虚拟机

您可以使用经典管理门户中的“快速创建”方法快速创建虚拟机。在创建此虚拟机时，您将使用一个对话框来提供配置详细信息。

**注意**：本文创建的是不连接到虚拟网络的虚拟机。如果您希望虚拟机使用虚拟网络，请改用“从库中”方法并在创建虚拟机时指定虚拟网络。有关虚拟网络的更多信息，请参见 [Azure 虚拟网络概述](http://go.microsoft.com/fwlink/p/?LinkID=294063)。

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。

2. 在命令栏上，单击“新建”。

	![新建虚拟机](./media/howto-quick-create-vm/create.png)

3. 单击“虚拟机”，然后单击“快速创建”。

	![快速创建新的虚拟机](./media/howto-quick-create-vm/createquick.png)

	将显示“新建虚拟机”对话框。

4. 键入新虚拟机的以下信息：

	- **DNS 名称** - 同时用于所创建的虚拟机和包含虚拟机的云服务的名称。
	- **映像** - 用于创建虚拟机的平台映像。
	- **用户名** - 要用于管理虚拟机的帐户的名称。
	- **帐户密码** - 键入并确认帐户的强密码。
	- **位置** - 包含虚拟机的区域。

5. 单击复选标记可创建虚拟机。

	**注意**：将自动创建一个存储帐户以包含此虚拟机。

	“虚拟机”页中将列出新的虚拟机。

	![成功创建虚拟机](./media/howto-quick-create-vm/vmsuccesswindows.png)


