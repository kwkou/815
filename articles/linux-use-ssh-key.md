<properties linkid="article" urlDisplayName="Use SSH" pageTitle="在 Azure 中使用 SSH 连接到 Linux 虚拟机" metaKeywords="Azure SSH keys Linux, Linux vm SSH" description="了解如何在 Azure 上通过 Linux 虚拟机生成和使用 SSH 密钥。" metaCanonical="" services="virtual-machines" documentationCenter="" title="如何在 Azure 上通过 Linux 使用 SSH" authors="" solutions="" manager="" editor="" />

# 如何在 Azure 上通过 Linux 使用 SSH

本主题介绍了用于生成与 Azure 兼容的 SSH 密钥的步骤。

当前版本的 Azure 管理门户只接受封装在 X509 证书中的 SSH 公钥。按照下面的步骤操作可生成 SSH 密钥并将其与 Azure 配合使用。

## 在 Linux 中生成 Windows Azure 兼容密钥

1.  根据需要安装 `openssl` 实用程序：

    **CentOS / Oracle Linux**

        `sudo yum install openssl`

    **Ubuntu**

        `sudo apt-get install openssl`

    **SLES 和 openSUSE**

        `sudo zypper install openssl`

2.  使用 `openssl` 生成带有 2048 位 RSA 密钥对的 X509 证书。请回答 `openssl` 提示的几个问题（或者您可以将其保留为空）。平台不使用这些字段中的内容：

            openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem

3.  更改私钥的权限可对其进行保护。

            chmod 600 myPrivateKey.key

4.  创建 Linux 虚拟机时上载 `myCert.pem`。设置过程会自动将此证书中的公钥安装到虚拟机中指定用户的 `authorized_keys` 文件中。

5.  如果您将直接使用 API，且不使用管理门户，请使用以下命令将 `myCert.pem` 转换为 `myCert.cer`（DER 编码的 X509 证书）：

            openssl  x509 -outform der -in myCert.pem -out myCert.cer

## 从现有 OpenSSH 兼容密钥生成密钥

前面的示例介绍了如何创建可用于 Windows Azure 的新密钥。在某些情况下，用户可能已经拥有现有的 OpenSSH 兼容公钥和私钥对，并且希望将相同的密钥用于 Windows Azure。

OpenSSH 私钥可通过 `openssl` 实用程序直接读取。以下命令将采用现有的 SSH 私钥（在以下示例中为 id\_rsa），并创建 Windows Azure 所需的 `.pem` 公钥：

    # openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem

“myCert.pem”文件是公钥，它随后可用于在 Windows Azure 上设置 Linux 虚拟机。在设置期间，`.pem` 文件将转换为 `openssh` 兼容公钥并放置在 `~/.ssh/authorized_keys` 中。

## 从 Linux 连接到 Windows Azure 虚拟机

在特定端口（此端口可能与使用的标准端口不同）中使用 SSH 设置每个 Linux 虚拟机，以便您

1.  从管理门户中查找将用于连接到 Linux 虚拟机的端口。
2.  使用 `ssh` 连接到 Linux 虚拟机。当您首次登录时，系统将提示您接受主机公钥的指纹。

        ssh -i  myPrivateKey.key -p <port> username@servicename.chinacloudapp.cn

3.  （可选）您可以将 `myPrivateKey.key` 复制到 `~/.ssh/id_rsa`，以便您的 openssh 客户端可自动选取它，而无需使用 `-i` 选项。

## 在 Windows 上获取 OpenSSL

### 使用 msysgit

1.  从以下位置下载和安装 msysgit：<http://msysgit.github.com/>
2.  从安装的目录运行 `msys`（示例：c:\\msysgit\\msys.exe）
3.  通过键入 `cd bin` 更改为 `bin` 目录

### 使用针对 Windows 的 GitHub

1.  从以下位置下载和安装针对 Windows 的 GitHub：<http://windows.github.com/>
2.  从“开始”菜单 \>“所有程序”\>“GitHub, Inc”运行 Git Shell

### 使用 cygwin

1.  从以下位置下载和安装 Cygwin：<http://cygwin.com/>
2.  确保安装了 OpenSSL 包及其所有依赖项。
3.  运行 `cygwin`

## 在 Windows 上创建私钥

1.  按照上述某组说明进行操作，以便能够运行 `openssl.exe`
2.  键入以下命令：

        openssl.exe req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem

3.  您的屏幕应与下图中所示类似：

    ![linuxwelcomegit][linuxwelcomegit]

4.  回答询问的问题。
5.  应已创建两个文件：`myPrivateKey.key` 和 `myCert.pem`。
6.  如果您将直接使用 API，且不使用管理门户，请使用以下命令将 `myCert.pem` 转换为 `myCert.cer`（DER 编码的 X509 证书）：

        openssl.exe  x509 -outform der -in myCert.pem -out myCert.cer

## 为 Putty 创建 PPK

1.  从以下位置下载和安装 puttygen：<http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html>
2.  运行 `puttygen.exe`
3.  单击菜单：“文件”\>“加载私钥”
4.  查找名为 `myPrivateKey.key` 的私钥。您将需要更改文件筛选器以显示“所有文件 (\*.\*)”
5.  单击“打开”。您将收到与如下所示的提示：

    ![linuxgoodforeignkey][linuxgoodforeignkey]

6.  单击“确定”。
7.  单击在下面的屏幕快照中突出显示的“保存私钥”：

    ![linuxputtyprivatekey][linuxputtyprivatekey]

8.  将文件另存为 PPK。

## 使用 Putty 连接到 Linux 计算机

1.  从以下位置下载和安装 putty：<http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html>
2.  运行 putty.exe
3.  从管理门户使用 IP 填充主机名。

    ![linuxputtyconfig][linuxputtyconfig]

4.  在选择“打开”之前，依次单击“连接”\>“SSH”\>“Auth”选项卡以选择您的密钥。在下面的屏幕快照中查看要填充的字段。

    ![linuxputtyprivatekey][1]

5.  单击“打开”以连接到您的虚拟机。

  [linuxwelcomegit]: ./media/linux-use-ssh-key/linuxwelcomegit.png
  [linuxgoodforeignkey]: ./media/linux-use-ssh-key/linuxgoodforeignkey.png
  [linuxputtyprivatekey]: ./media/linux-use-ssh-key/linuxputtygenprivatekey.png
  [linuxputtyconfig]: ./media/linux-use-ssh-key/linuxputtyconfig.png
  [1]: ./media/linux-use-ssh-key/linuxputtyprivatekey.png
