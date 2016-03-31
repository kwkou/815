<properties
	pageTitle="从 GitHub 更新 Azure Linux 代理 | Azure"
	description="了解如何从 Github 将 Azure 中 Linux VM 的 Azure Linux 代理更新到最新版本"
	services="virtual-machines"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="12/14/2015"
	wacn.date="01/29/2016"/>


# 如何从 GitHub 将 VM 上的 Azure Linux 代理更新到最新版本

若要更新 Azure 中 Linux VM 上的 [Azure Linux 代理](https://github.com/Azure/WALinuxAgent)，你必须已具备以下条件：

1. 在 Azure 中运行的 Linux VM
2. 已使用 SSH 连接到该 Linux VM

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


> [AZURE.NOTE]如果你将从 Windows 计算机执行此任务，则可以使用 Putty 通过 SSH 登录到 Linux 计算机。有关详细信息，请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。

Azure 支持的 Linux 发行版已将 Azure Linux 代理包放入其存储库中，因此，请先从该发行版存储库中查找并安装最新版本（如果可能）。

对于 Ubuntu，只需键入：

    #sudo apt-get install walinuxagent

在 CentOS 中，请键入：

    #sudo yum install waagent

对于 Oracle Linux，请确保加载项存储库在文件 `/etc/yum.repo.d/public-yum-ol6.repo` 或 `/etc/yum.repo.d/public-yum-ol7.repo` 中已启用，然后键入：

    #sudo yum install WALinuxAgent

通常只需要这样做，但如果因某种原因而需要直接从 https://github.com 安装它，请使用以下步骤。


## 安装 wget

使用 SSH 登录到你的 VM。

通过在命令行上键入 `#sudo yum install wget` 来安装 wget（有一些发行版在默认情况下未安装它，如 Redhat、CentOS 和 Oracle Linux 6.4 和 6.5 版）。


## 下载最新版本

在网页中打开 [GitHub 中的 Azure Linux 代理版本](https://github.com/Azure/WALinuxAgent/releases)，并找到最新的版本号。（可以通过键入 `#waagent --version` 查明你的当前版本。）

###对于版本 2.0.x，请键入：

    #wget https://raw.githubusercontent.com/Azure/WALinuxAgent/WALinuxAgent-[version]/waagent  

   以下行使用版本 2.0.14 作为示例：

    #wget https://raw.githubusercontent.com/Azure/WALinuxAgent/WALinuxAgent-2.0.14/waagent  

###对于 2.1.x 或更高版本，请键入：

    #wget https://github.com/Azure/WALinuxAgent/archive/WALinuxAgent-[version].zip
    #unzip WALinuxAgent-[version].zip
    #cd WALinuxAgent-[version]

   以下行使用版本 2.1.0 作为示例：

    #wget https://github.com/Azure/WALinuxAgent/archive/WALinuxAgent-2.1.0.zip
    #unzip WALinuxAgent-2.1.0.zip  
    #cd WALinuxAgent-2.1.0

##安装 Linux 代理

###对于版本 2.0.x，请使用：

 Make waagent executable

    #chmod +x waagent

 Copy new executable to /usr/sbin/

  对于大多数 Linux，请使用

    #sudo cp waagent /usr/sbin

  对于 CoreOS，使用：

    #sudo cp waagent /usr/share/oem/bin/

  如果这是新安装的 Azure Linux 代理，请运行以下内容：
 
    #sudo /usr/sbin/waagent -install -verbose

###对于版本 2.1.x，请使用：

你可能需要先安装程序包 `setuptools`，详情请参阅[此处](https://pypi.python.org/pypi/setuptools)。然后运行以下项目：

    #sudo python setup.py install

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

<!---HONumber=Mooncake_0118_2016-->