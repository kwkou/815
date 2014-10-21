<properties linkid="manage-linux-howto-logon-linux-vm" urlDisplayName="Log on to a VM" pageTitle="Log on to a virtual machine running Linux in Azure" metaKeywords="Azure Linux vm, Linux SSH" description="Learn how to log on to an Azure virtual machine running Linux by using a Secure Shell (SSH) client." metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Log on to a Virtual Machine Running Linux" authors="" solutions="" manager="" editor="" />

# 如何登录到运行 Linux 的虚拟机

对于运行 Linux 操作系统的虚拟机，你可以使用安全外壳 (SSH) 客户端进行登录。

需要在用于登录到虚拟机的计算机上安装 SSH 客户端。你可以选择很多 SSH 客户端程序。下面是可能的选项：

-   在运行 Windows 操作系统的计算机上，你可能希望使用 PuTTY 之类的 SSH 客户端。有关详细信息，请参阅 [PuTTY 下载页][PuTTY 下载页]。
-   在运行 Linux 操作系统的计算机上，你可能希望使用 OpenSSH 之类的 SSH 客户端。有关详细信息，请参阅 [OpenSSH][OpenSSH]。

此过程将向你演示如何使用 PuTTY 程序访问虚拟机。

1.  从[管理门户][管理门户]中找到“主机名”和“端口信息”。你可以从虚拟机的仪表板中找到所需信息。单击虚拟机名称并查看仪表板“速览”部分的“SSH 详细信息”。

    ![获取 SSH 详细信息][获取 SSH 详细信息]

2.  打开 PuTTY 程序。

3.  输入你从仪表板中收集到的主机名和端口信息，然后单击“打开”。

    ![打开 PuTTY][打开 PuTTY]

4.  使用你在创建虚拟机时指定的帐户登录到虚拟机。默认用户名为 azureuser。

    ![登录到虚拟机][登录到虚拟机]

    你现在可以像使用任何其他服务器一样使用该虚拟机。

[WACOM.NOTE] 有关要求和疑难解答提示，请参阅[使用 RDP 或 SSH 连接到 Windows Azure 虚拟机][使用 RDP 或 SSH 连接到 Windows Azure 虚拟机]。

  [PuTTY 下载页]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
  [OpenSSH]: http://www.openssh.org/
  [管理门户]: http://manage.windowsazure.cn
  [获取 SSH 详细信息]: ./media/virtual-machines-linux-how-to-log-on/sshdetails.png
  [打开 PuTTY]: ./media/virtual-machines-linux-how-to-log-on/putty.png
  [登录到虚拟机]: ./media/virtual-machines-linux-how-to-log-on/sshlogin.png
  [使用 RDP 或 SSH 连接到 Windows Azure 虚拟机]: http://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx
