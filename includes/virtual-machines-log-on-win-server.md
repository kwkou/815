<properties services="virtual-machines" title="How to Log on to a Virtual Machine Running Windows Server" authors="cynthn" solutions="" manager="timlt" editor="tysonn" />

4. 单击“连接”创建和下载远程桌面协议文件（.rdp 文件）。单击“打开”以使用此文件。

5. 在“远程桌面”窗口中，单击“连接”以继续。

	![继续连接](./media/virtual-machines-log-on-win-server/connectpublisher.png)

6. 在“Windows 安全性”窗口中，键入虚拟机上帐户的凭据，然后单击“确定”。

 	在大多数情况下，凭据是创建虚拟机时指定的本地帐户用户名和密码。在本示例中，域是虚拟机的名称，输入格式为 *vmname*&#92;*username*。  
	
	如果虚拟机属于你的组织的一个域，请确保用户名包含该域的名称，格式为“Domain&#92;Username”。该帐户还需要属于管理员组或已被授予虚拟机的远程访问权限。
	
	如果虚拟机是域控制器，则键入该域的域管理员帐户的用户名和密码。

7.	单击“是”以验证虚拟机的 ID 并完成登录。

	![验证计算机的标识](./media/virtual-machines-log-on-win-server/connectverify.png)

<!---HONumber=Mooncake_0314_2016-->