<properties
	pageTitle="Azure 中的 Linux 简介 | Azure"
	description="了解如何在 Azure 上使用 Linux 虚拟机。"
	services="virtual-machines-linux"
	documentationCenter="python"
	authors="szarkos"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/01/2016"
	wacn.date="03/28/2016"/>

#Azure 上的 Linux 简介

本主题提供了在 Azure 云中使用 Linux 虚拟机的某些方面的概述。在使用库中的映像时，部署 Linux 虚拟机是一个非常简单的过程。

## 身份验证：用户名、密码和 SSH 密钥

在使用 Azure 经典管理门户创建 Linux 虚拟机时，系统会要求你提供用户名、密码或 SSH 公钥。在 Azure 上部署 Linux 虚拟机时，用户名的选择受到以下限制：不允许使用虚拟机中已经存在的系统帐户 (UID <100) 的名称，例如，“根”。


 - 请参阅[创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-quick-create-cli/)
 - 请参阅[如何在 Azure 上将 SSH 用于 Linux 和 Mac](/documentation/articles/virtual-machines-linux-ssh-from-linux/)

## 使用 `sudo` 获取超级用户特权

在 Azure 上部署虚拟机实例的过程中指定的用户帐户是特权帐户。此帐户由 Azure Linux 代理配置为能够使用 `sudo` 实用工具提升根（超级用户帐户）的特权。在使用此用户帐户登录后，你将能够使用命令语法以根用户身份运行命令。

	# sudo <COMMAND>

可以选择使用 **sudo -s** 获取根 shell。

- 请参阅[在 Azure 中对 Linux 虚拟机使用根特权](/documentation/articles/virtual-machines-linux-use-root-privileges/)

## 防火墙配置

Azure 提供了一个入站数据包筛选器，用于限制与经典管理门户中指定的端口的连接。默认情况下，唯一允许的端口为 SSH。通过在经典管理门户中配置终结点，可以启用对 Linux 虚拟机上的其他端口的访问：

 - 请参阅：[如何设置虚拟机的终结点](/documentation/articles/virtual-machines-linux-classic-setup-endpoints/)

默认情况下，Azure 库中的 Linux 映像不支持 *iptables* 防火墙。如果需要，可以将该防火墙配置为提供附加筛选。


## 主机名更改

在初始部署 Linux 映像的实例时，您需要提供虚拟机的主机名。运行虚拟机后，此主机名将发布到平台 DNS 服务器，以便多个相互连接的虚拟机能够使用主机名执行 IP 地址查找。

如果在部署虚拟机后需要更改主机名，请使用命令

	# sudo hostname <newname>

Azure Linux 代理包含自动检测此名称更改的功能，并会相应地配置虚拟机以保留此更改以及将此更改发布到平台 DNS 服务器。

 - [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)

### Cloud-Init
**Ubuntu** 和 **CoreOS** 映像利用 cloud-init pn Azure 为启动虚拟机提供附加功能。

 - [如何插入自定义数据](/documentation/articles/virtual-machines-linux-classic-inject-custom-data/)
 - [Azure 上的自定义数据和 Cloud-Init](http://azure.microsoft.com/blog/2014/04/21/custom-data-and-cloud-init-on-windows-azure/)
 - [使用 Cloud-Init 创建 Azure 交换分区](https://wiki.ubuntu.com/AzureSwapPartitions)
 - [如何在 Azure 上使用 CoreOS](https://coreos.com/os/docs/latest/booting-on-azure.html)


## 虚拟机映像捕获

利用 Azure，可以将现有虚拟机的状态捕获到映像中，该映像随后可用于部署其他虚拟机实例。Azure Linux 代理可用于回滚在设置过程中执行的某些自定义。您可以按照下面的步骤操作来将虚拟机作为映像捕获：

1. 运行 **waagent -deprovision** 以撤消预配自定义项。或（可选）运行 **waagent -deprovision+user** 以删除预配期间指定的用户帐户和所有关联的数据。

2. 关闭虚拟机。

3. 在经典管理门户中单击“捕获”或者使用 Powershell 或 CLI 工具将虚拟机作为映像捕获。

 - 请参阅：[如何捕获 Linux 虚拟机以用作模板](/documentation/articles/virtual-machines-linux-classic-capture-image/)


## 附加磁盘

每个虚拟机都附加有一个临时的本地*资源磁盘*。由于资源磁盘上的数据可能不能在重新启动后持久存在，因此它通常由在虚拟机中运行的应用程序和进程用于数据的短暂和**临时**存储。它还用来为操作系统存储页面文件或交换文件。

在 Linux 上，资源磁盘通常由 Azure Linux 代理管理并且自动装载到 **/mnt/resource**（或 Ubuntu 映像上的 **/mnt**）。


>[AZURE.NOTE]请注意，资源磁盘是**临时**磁盘，并可能在重新启动 VM 时被删除或重新格式化。

在 Linux 上，数据磁盘可能由内核命名为 `/dev/sdc`，并且用户需要对该资源进行分区、格式化和装载。在[如何将数据磁盘附加到虚拟机](/documentation/articles/virtual-machines-linux-classic-attach-disk/)的教程中对此进行了分步说明。

 - **另请参阅：**[在 Linux 上配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/)

<!---HONumber=Mooncake_1207_2015-->