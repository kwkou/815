<properties
	pageTitle="Linux VM 的常见问题 | Azure"
	description="回答了通过 Resource Manager 模型创建的 Linux 虚拟机的一些常见问题。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="05/16/2016"
	wacn.date="07/11/2016"/>

# 有关 Linux 虚拟机的常见问题 


本文讨论了在 Azure 中使用 Resource Manager 部署模型创建的 Linux 虚拟机的一些用户常见问题。有关本主题的 Windows 版本，请参阅[有关 Windows 虚拟机的常见问题](/documentation/articles/virtual-machines-windows-faq/)

## 我可以在 Azure VM 上运行什么程序？

所有订户都可以在 Azure 虚拟机上运行服务器软件。有关详细信息，请参阅 [Azure 认可分发版中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distros/)


## 使用虚拟机时，我可以使用多少存储？

每个数据磁盘的容量高达 1 TB。你可以使用的数据磁盘的数目取决于虚拟机的大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)。

Azure 存储帐户提供可用于操作系统磁盘和任意数据磁盘的存储。每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。有关定价详细信息，请参阅[存储定价详细信息](/home/features/storage/pricing/)。



## 如何访问我的虚拟机？

需使用安全外壳 (SSH) 建立远程连接，以登录到虚拟机。请参阅如何[从 Windows](/documentation/articles/virtual-machines-linux-ssh-from-windows/) 或[从 Linux 和 Mac](/documentation/articles/virtual-machines-linux-ssh-from-linux/) 进行连接的相关说明。默认情况下，SSH 允许的并发连接最多为 10 个。通过编辑配置文件，可以增大此数目。


如果遇到问题，请查阅[排除安全外壳 (SSH) 连接故障](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)。

## 我是否可以使用临时磁盘 (/dev/sdb1) 存储数据？

不应使用临时磁盘 (/dev/sdb1) 存储数据。这种磁盘只是临时存储空间，存在丢失数据且数据不能恢复的风险。

## 我是否可以复制或克隆现有的 Azure VM？

是的。相关说明请参阅[如何在 Resource Manager 部署模型中创建 Linux 虚拟机的副本](/documentation/articles/virtual-machines-linux-specialized-image/)。

<!---HONumber=Mooncake_0704_2016-->