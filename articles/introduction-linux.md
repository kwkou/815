<properties linkid="manage-linux-fundamentals-intro-to-linux" urlDisplayName="Intro to Linux" pageTitle="Azure 中的 Linux 简介 - Azure 教程" metaKeywords="Azure Linux vm, Linux vm" description="了解如何在 Azure 上使用 Linux 虚拟机。" metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Azure 上的 Linux 简介" authors="" solutions="" manager="" editor="" />

# Azure 上的 Linux 简介

本主题提供了在 Azure 云中使用 Linux 虚拟机的某些方面的概述。在使用库中预先存在的映像时，部署 Linux 虚拟机是一个简单的过程。

## 目录

-   [身份验证：用户名、密码和 SSH 密钥。][身份验证：用户名、密码和 SSH 密钥。]
-   [生成并使用 SSH 密钥以登录到 Linux 虚拟机。][生成并使用 SSH 密钥以登录到 Linux 虚拟机。]
-   [使用 sudo 获取超级用户特权][使用 sudo 获取超级用户特权]
-   [防火墙配置][防火墙配置]
-   [主机名更改][主机名更改]
-   [虚拟机映像捕获][虚拟机映像捕获]
-   [附加磁盘][附加磁盘]

## <span id="authentication"></span></a>身份验证：用户名、密码和 SSH 密钥

在使用 Azure 管理门户创建 Linux 虚拟机时，系统会要求您提供用户名、密码和（可选）SSH 公钥。对用于在 Azure 上部署 Linux 虚拟机的用户名的选择将受到以下限制：不允许虚拟机中已存在的系统帐户的名称 - 例如根。如果您未提供 SSH 公钥，则您将能够使用指定的用户名和密码登录到虚拟机。如果您选择在管理门户中上载 SSH 公钥，则将禁用基于密码的身份验证，并且您需要使用相应的 SSH 私钥对虚拟机进行身份验证并进行登录。

## <span id="keygeneration"></span></a>SSH 密钥生成

当前版本的管理门户只接受封装在 X509 证书中的 SSH 公钥。请按照下面的步骤操作以通过 Azure 生成并使用 SSH 密钥。

1.  使用 openssl 生成带 2048 位 RSA 密钥对的 X509 证书。

        openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout myPrivateKey.key -out myCert.pem

    请回答 openssl 提示的几个问题（您可以将这些问题保留为空）。平台不使用这些字段中的内容。

2.  更改私钥的权限可对其进行保护。

        chmod 600 myPrivateKey.key

3.  将 myCert.pem 转换为 myCert.cer（DER 编码的 X509 证书）

        openssl  x509 -outform der -in myCert.pem -out myCert.cer

4.  创建 Linux 虚拟机时上载 myCert.cer。设置过程会自动将此证书中的公钥安装到虚拟机中指定用户的 authorized\_keys 文件中。

5.  使用 SSH 连接到 Linux 虚拟机。

        ssh -i  myPrivateKey.key -p port  username@servicename.cloudapp.net

    当您首次登录时，系统将提示您接受主机公钥的指纹。

6.  您可以选择将 myPrivateKey.key 复制到 ~/.ssh/id\_rsa，以便您的 openssh 客户端能够自动选取它而不使用“-i”选项。

## <span id="superuserprivileges"></span></a>使用 Sudo 获取超级用户特权

在 Azure 上部署虚拟机实例的过程中指定的用户帐户是特权帐户。此帐户由 Azure Linux 代理配置为能够使用 sudo 工具提升根（超级用户帐户）的权限。在使用此用户帐户登录后，您将能够使用“sudo 命令”将命令作为根运行。可以选择使用“sudo -s”获取根外壳程序。

## <span id="firewallconfiguration"></span></a>防火墙配置

Azure 提供了一个入站数据包筛选器，用于限制与管理门户中指定的端口的连接。默认情况下，唯一允许的端口为 SSH。通过在管理门户中添加规则，可以启用对 Linux 虚拟机上的其他端口的访问。

库中的 Linux 映像不支持 Linux 虚拟机中的 iptables 防火墙。如果需要，可以配置 IPtables 防火墙以提供其他功能。

## <span id="hostnamechanges"></span></a>主机名更改

在初始部署 Linux 映像的实例时，您需要提供虚拟机的主机名。运行虚拟机后，此主机名将发布到平台 DNS 服务器，以便多个相互连接的虚拟机能够使用主机名执行 IP 地址查找。如果在部署虚拟机后需要更改主机名，请使用 **hostname newname** 命令。Azure Linux 代理包括自动检测此名称更改的功能，并会相应地配置虚拟机以保留此更改以及将此更改发布到平台 DNS 服务器。有关其他详细信息和与此功能相关的配置选项，请参考 Azure Linux 代理的自述文件。

## <span id="virtualmachine"></span></a>虚拟机映像捕获

利用 Azure，可以将现有虚拟机的状态捕获到映像中，该映像随后可用于部署其他虚拟机实例。Azure Linux 代理可用于回滚在设置过程中执行的某些自定义。您可以按照下面的步骤操作来将虚拟机作为映像捕获：

1.  运行“waagent -deprovision”以撤消设置自定义项。或（可选）运行“waagent -deprovision+user”以删除设置期间指定的用户帐户和所有关联的数据。

2.  使用管理门户/工具关闭虚拟机。

3.  在管理门户中单击“捕获”或使用这些工具将虚拟机作为映像捕获。

## <span id="attachingdisks"></span></a>附加磁盘

在有关如何[创建 Linux 虚拟机]的教程中对此进行了分步说明。

  [身份验证：用户名、密码和 SSH 密钥。]: #authentication
  [生成并使用 SSH 密钥以登录到 Linux 虚拟机。]: #keygeneration
  [使用 sudo 获取超级用户特权]: #superuserprivileges
  [防火墙配置]: #firewallconfiguration
  [主机名更改]: #hostnamechanges
  [虚拟机映像捕获]: #virtualmachine
  [附加磁盘]: #attachingdisks
