<properties
	pageTitle="如何从 Github 将 Azure Linux 代理更新到最新版本"
	description="了解如何从 Github 为 Azure 中的 Linux VM 更新 Azure Linux 代理。"
	services="virtual-machines"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/23/2015"
	wacn.date="05/15/2015"
	ms.author="mingzhan"/>


# 如何从 Github 将 Azure Linux 代理更新到最新版本

若要更新 [Azure Linux 代理](https://github.com/Azure/WALinuxAgent)，你必须已具备以下条件：

1. 在 Azure 中运行的 Linux VM
2. 已使用 SSH 连接到该 Linux VM

> [AZURE.NOTE] 如果你将从 Windows 计算机执行此任务，则可以使用 Putty 通过 SSH 登录到 Linux 计算机。有关详细信息，请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。

Azure 支持的 Linux 发行版已将 Azure Linux 代理包放入其存储库中，因此，请先从该发行版存储库中查找并安装最新版本（如果可能）。  

对于 Ubuntu，只需键入：
     
    #sudo apt-get install waagent

在 CentOS 和 Oracle Linux 上，键入：

    #sudo yum install waagent

通常，你只需要这样做，但是如果出于某种原因，你需要从 https://github.com directly, use the following steps 安装它。 


## 安装 wget

使用 SSH 登录到你的 VM。 

通过在命令行上键入 `#sudo yum install wget` 来安装 wget（有一些发行版在默认情况下未安装它，如 Redhat、CentOS 和 Oracle Linux 6.4 和 6.5 版）。


## 下载最新版本

在网页中打开 [Github 中的 Azure Linux 代理版本](https://github.com/Azure/WALinuxAgent/releases)，并找到最新的版本号，如下所示：2.0.12。（可以通过键入 `#waagent --version` 查明你的当前版本。）

    #wget https://raw.githubusercontent.com/Azure/WALinuxAgent/WALinuxAgent-[version]/waagent  

以下行使用版本 2.0.12 作为示例：

    #wget https://raw.githubusercontent.com/Azure/WALinuxAgent/WALinuxAgent-2.0.12/waagent  

## Make waagent executable

    #chmod +x waagent

## Copy new executable to /usr/sbin/
    
    #sudo cp waagent /usr/sbin

对于 CoreOS，使用：

    #sudo cp waagent /usr/share/oem/bin/
 
## Restart waagent service

对于大多数 linux 发行版：

    #sudo service waagent restart

对于 Ubuntu，使用：

    #sudo service walinuxagent restart

对于 CoreOS，使用：

    #sudo systemctl restart waagent 

## 确认 Azure Linux 代理版本
   
    #waagent -version

对于 CoreOS，上面的命令可能无法工作。

你将看到 Linux 代理版本已更新为新版本。

有关 Azure Linux 代理的详细信息，请参阅 [Azure Linux 代理自述文件](https://github.com/Azure/WALinuxAgent)。




<!--HONumber=53-->