<properties linkid="manage-linux-common-tasks-user-root-privileges" urlDisplayName="Use root privileges" pageTitle="在 Azure 中的 Linux 虚拟机上使用根权限" metaKeywords="" description="了解如何在 Azure 中的 Linux 虚拟机上使用根权限。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Using root privileges on Linux virtual machines in Azure" authors="" solutions="" manager="" editor="" />





# 在 Azure 中使用 Linux 虚拟机的根权限

默认情况下，`root`用户已在 Azure 中的 Linux 虚拟机上被禁用。用户可以通过使用提升的权限运行命令`sudo`命令。但是，体验可能因系统如何设置。

1. **SSH 密钥和密码或密码仅**-与其中一个证书已设置虚拟机 (`.CER`文件） 或 SSH 密钥，以及密码，或只是用户名和密码。在这种情况下`sudo`在执行该命令之前将提示输入用户的密码。

2. **SSH 密钥仅**-虚拟机已使用证书设置 （`.cer`或`.pem`文件） 或 SSH 密钥，但没有密码。在这种情况下`sudo`**将不会**提示执行命令之前的用户的密码。


## SSH 密钥和密码，或者仅密码

登录到 Linux 虚拟机使用 SSH 密钥或密码身份验证，然后运行命令 （使用`sudo`，例如：

	# sudo <command>
	[sudo] password for azureuser:

在这种情况下用户将提示输入密码。在输入密码后`sudo`将运行中的命令`root`特权。


## 仅 SSH 密钥

登录到 Linux 虚拟机使用 SSH 密钥身份验证，然后运行命令 （使用`sudo`，例如：

	# sudo <command>

在这种情况下用户将**不**提示输入密码。按下之后`<enter>`， `sudo`将运行中的命令`root`特权。

