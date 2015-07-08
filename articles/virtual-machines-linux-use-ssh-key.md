<properties urlDisplayName="Use SSH" pageTitle="在 Azure 中使用 SSH 连接到 Linux 虚拟机" metaKeywords="Azure SSH keys Linux, Linux vm SSH" description="了解如何在 Azure 上通过 Linux 虚拟机生成和使用 SSH 密钥。" metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Use SSH with Linux on Azure" authors="szarkos" solutions="" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.date="05/15/2015" wacn.date="06/19/2015"/>

# 如何在 Azure 上通过 Linux 使用 SSH

当前版本的 Azure 管理门户只接受封装在 X509 证书中的 SSH 公钥。按照下面的步骤操作可生成 SSH 密钥并将其与 Azure 配合使用。

## 在 Linux 中生成 Windows Azure 兼容密钥 ##

1. 根据需要安装 `openssl` 实用工具：

	**CentOS/Oracle Linux**

		# sudo yum install openssl

	**Ubuntu**

		# sudo apt-get install openssl

	**SLES 和 openSUSE**

		# sudo zypper install openssl


2. 使用 `openssl` 生成带 2048 位 RSA 密钥对的 X509 证书。请回答 `openssl` 提示输入的几个问题（也可以将其留空）。平台不使用这些字段中的内容：

		# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem

3.	更改私钥的权限可对其进行保护。

		# chmod 600 myPrivateKey.key

4.	创建 Linux 虚拟机时上载 `myCert.pem`。设置过程会自动将此证书中的公钥安装到虚拟机中指定用户的 `authorized_keys` 文件中。

5.	如果你要直接使用 API，而不使用管理门户，请使用以下命令将 `myCert.pem` 转换为 `myCert.cer`（DER 编码的 X509 证书）：

		# openssl  x509 -outform der -in myCert.pem -out myCert.cer


## 从现有 OpenSSH 兼容密钥生成密钥
前面的示例介绍了如何创建可用于 Windows Azure 的新密钥。在某些情况下，你可能已经拥有现有的 OpenSSH 兼容公钥和私钥对，并且希望将相同的密钥用于 Windows Azure。

OpenSSH 私钥可通过 `openssl` 实用工具直接读取。以下命令将采用现有的 SSH 私钥（在以下示例中为 id_rsa），并创建 Windows Azure 所需的 `.pem` 公钥：

	# openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem

**myCert.pem** 文件是公钥，它随后可用于在 Windows Azure 上设置 Linux 虚拟机。在设置期间，`.pem` 文件将转换为 `openssh` 兼容公钥并放置在 `~/.ssh/authorized_keys` 中。


## 从 Linux 连接到 Windows Azure 虚拟机

1. 在某些情况下，可以为默认端口 22 以外的其他端口配置用于 Linux 虚拟机的 SSH 终结点。可以在管理门户中的 VM 仪表板上（“SSH 详细信息”下）找到正确的端口号。

2.	使用 `ssh` 连接到 Linux 虚拟机。当您首次登录时，系统将提示您接受主机公钥的指纹。

		# ssh -i  myPrivateKey.key -p <port> username@servicename.chinacloudapp.cn

3.	（可选）你可以将 `myPrivateKey.key` 复制到 `~/.ssh/id_rsa`，以便你的 OpenSSH 客户端可自动选取它，而无需使用 `-i` 选项。

## 在 Windows 上获取 OpenSSL ##

有大量实用工具，包括 `openssl` for Windows。下面列出了几个示例 -

### 使用 Msysgit ###

1.	从以下位置下载并安装 msysgit：[http://msysgit.github.com/](http://msysgit.github.com/)
2.	从安装目录运行 `msys`（示例：c:\msysgit\msys.exe）
3.	通过键入 `cd bin` 更改为 `bin` 目录


### 使用针对 Windows 的 GitHub ###

1.	从以下位置下载并安装 GitHub for Windows：[http://windows.github.com/](http://windows.github.com/)
2.	从“开始”菜单 >“所有程序”>“GitHub, Inc”运行 Git Shell

	**注意：**在运行上述 `openssl` 命令时，可能会遇到以下错误：

		Unable to load config info from /usr/local/ssl/openssl.cnf

	解决此问题的最简单方法是设置 `OPENSSL_CONF` 环境变量。此变量的设置过程将因已在 Github 中配置的 shell 而异：

	**Powershell：**

		$Env:OPENSSL_CONF="$Env:GITHUB_GIT\ssl\openssl.cnf"

	**CMD：**

		set OPENSSL_CONF=%GITHUB_GIT%\ssl\openssl.cnf

	**Git Bash：**

		export OPENSSL_CONF=$GITHUB_GIT/ssl/openssl.cnf


### 使用 Cygwin ###

1.	从以下位置下载并安装 Cygwin：[http://cygwin.com/](http://cygwin.com/)
2.	确保安装了 OpenSSL 包及其所有依赖项。
3.	运行 `cygwin`


## 在 Windows 上创建私钥 ##

1.	按照上述某组说明进行操作，以便能够运行 `openssl.exe`
2.	键入以下命令：

		# openssl.exe req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem

3.	您的屏幕应与下图中所示类似：

	![linuxwelcomegit](./media/virtual-machines-linux-use-ssh-key/linuxwelcomegit.png)

4.	回答询问的问题。
5.	应已创建两个文件：`myPrivateKey.key` 和 `myCert.pem`。
6.	如果你要直接使用 API，而不使用管理门户，请使用以下命令将 `myCert.pem` 转换为 `myCert.cer`（DER 编码的 X509 证书）：

		# openssl.exe  x509 -outform der -in myCert.pem -out myCert.cer

## 为 Putty 创建 PPK ##

1. 从以下位置下载并安装 Puttygen：[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)

2. Puttygen 可能不能读取先前创建的私钥 (`myPrivateKey.key`)。运行以下命令以将其转换为 Puttygen 可以识别的 RSA 私钥：

		# openssl rsa -in ./myPrivateKey.key -out myPrivateKey_rsa
		# chmod 600 ./myPrivateKey_rsa

	上面的命令应生成名为 myPrivateKey_rsa 的新私钥。

3. 运行 `puttygen.exe`

4. 单击菜单：“文件”>“加载私钥”

5. 查找上述名为 `myPrivateKey_rsa` 的私钥。你将需要更改文件筛选器以显示**“所有文件 (\*.\*)”**

6. 单击**“打开”**。您将收到与如下所示的提示：

	![linuxgoodforeignkey](./media/virtual-machines-linux-use-ssh-key/linuxgoodforeignkey.png)

7. 单击**“确定”**

8. 单击在下面的屏幕截图中突出显示的**“保存私钥”**：

	![linuxputtyprivatekey](./media/virtual-machines-linux-use-ssh-key/linuxputtygenprivatekey.png)

9. 将文件另存为 PPK


## 使用 Putty 连接到 Linux 计算机 ##

1.	从以下位置下载并安装 putty：[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
2.	运行 putty.exe
3.	从管理门户使用 IP 填充主机名：

	![linuxputtyconfig](./media/virtual-machines-linux-use-ssh-key/linuxputtyconfig.png)

4.	在选择**“打开”**之前，依次单击“连接”>“SSH”>“身份验证”选项卡以选择你的密钥。在下面的屏幕截图中查看要填充的字段：

	![linuxputtyprivatekey](./media/virtual-machines-linux-use-ssh-key/linuxputtyprivatekey.png)

5.	单击**“打开”**以连接到你的虚拟机

<!---HONumber=60-->