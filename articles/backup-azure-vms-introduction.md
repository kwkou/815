<properties
	pageTitle="Azure 虚拟机备份简介"
	description="介绍如何使用 Azure 备份服务在 Azure 中备份虚拟机"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags ms.service="backup" ms.date="04/30/2015" wacn.date="06/16/2015"/>

# Azure 虚拟机备份 - 简介

本部分介绍如何使用 Windows Azure 备份来保护 Azure 虚拟机。通过阅读本文，你将会了解：

+ Azure 虚拟机备份原理

+ 备份 Azure 虚拟机的过程

+ 实现顺畅备份体验的先决条件

+ 遇到的典型错误以及如何处理它们

+ 不支持的方案的列表以及如何影响对产品所做的更改

若要快速了解有关 Azure 虚拟机的详细信息，请参阅[虚拟机文档][vm-doc]。

## 为何要备份 Azure 虚拟机？
云计算使应用程序能够在可缩放且高度可用的环境中执行，这正是 Microsoft 开发 Azure 虚拟机的原因。从这些 Azure 虚拟机生成的数据很重要，需要进行备份来保证安全。需要从备份恢复数据的典型场景包括：

+ 文件被意外删除或恶意删除

+ 虚拟机在修补程序更新期间遭到损坏

+ 整个虚拟机被意外删除或恶意删除

可以通过两种不同的方式在这些虚拟机中备份数据：

+ 在虚拟机中备份数据的各个源

+ 备份整个虚拟机


备份整个虚拟机更常用，因为它管理起来要简单得多，而且可以轻松地恢复整个应用程序和操作系统。Azure 备份可用于来宾内部数据的备份或完整虚拟机的备份。

企业使用 Azure 备份进行虚拟机备份的优势是：

+ 实现针对虚拟机的备份和恢复工作流的自动化

+ 实现应用程序一致性备份，确保恢复的数据从一致的状态开始。

+ 在虚拟机备份期间不需要停机。

+ 可以备份 Windows 虚拟机或 Linux 虚拟机。

+ 恢复点可用来在“Azure 备份”保管库中轻松进行恢复。

+ 对较旧的恢复点进行自动修剪和垃圾回收。 

## Azure 虚拟机备份原理
若要备份虚拟机，首先需要获得数据的时间点快照。Azure 备份服务在计划的时间启动备份作业，并触发要拍摄快照的备份扩展。备份扩展会与来宾内部的 VSS 服务进行协调以确保一致性，并会在达成一致后调用 Azure 存储服务的 Blob 快照 API。这样做可以获得虚拟机磁盘的一致性快照，不必关闭虚拟机。


拍摄快照后，数据将由 Azure 备份服务传输到备份保管库中。该服务负责确定并传输自上次备份以来进行了更改的块，使备份存储更高效。数据传输完成后，将会删除快照并创建恢复点。在 Azure 管理门户中，可以查看此恢复点。

![Azure 虚拟机备份体系结构][vm-backup-arch]

>  
>[AZURE.NOTE]Linux 虚拟机只能使用文件一致性备份。


## 计算受保护的实例
使用 Azure 备份进行备份的 Azure 虚拟机的收费依据 [Azure 备份定价][azure-backup-pricing]。受保护的实例计算基于虚拟机的*实际* 大小，即虚拟机中除“资源磁盘”外的所有数据之和。*不是*按连接到虚拟机的每个受支持数据磁盘的最大大小对你收费，而是按存储在数据磁盘中的实际数据收费。与此类似，备份存储空间的收费是根据通过 Azure 存储空间存储的数据容量，即每个恢复点中实际数据之和。

以 A2 标准大小的虚拟机为例，该虚拟机有两个额外的数据磁盘，总大小为 1TB。下表提供了每个这样的磁盘上存储的实际数据：

|磁盘类型|最大大小|实际存在的数据|
|---------|--------|------|
| 操作系统磁盘 | 1023GB | 17GB |
| 本地磁盘/资源磁盘 | 135GB | 5GB（不包括在备份中） |
| 数据磁盘 1 |	1023GB | 30GB |
| 数据磁盘 2 | 1023GB | 0GB |

此示例中，虚拟机的*实际* 大小为 17GB + 30GB + 0GB = 47GB。这就是按月收费所基于的受保护实例大小。随着虚拟机中数据量的增长，用于计费的受保护实例大小也会相应变化。


## 先决条件
### 1.备份保管库
若要开始备份 Azure 虚拟机，你需要首先创建备份保管库。保管库是存储所有按时间创建的备份和恢复点的实体。保管库还包含将应用到要备份的虚拟机的备份策略。

下图显示了各种 Azure 备份实体之间的关系：

![Azure 备份实体和关系][backup-entities]

### 创建备份保管库的步骤

1. 登录到[管理门户][mgmt-portal]。

2. 单击“新建”->“数据服务”->“恢复服务”->“备份保管库”，然后选择“快速创建”。如果有多个订阅与你的组织帐户相关联，请选择要与备份保管库关联的正确订阅。在每个 Azure 订阅中，你可以使用多个备份保管库来组织受保护的虚拟机。

3. 在“名称”中，输入一个友好名称以标识此保管库。这必须是每个订阅的唯一名称。

4. 在“区域”中，为保管库选择地理区域。请注意，保管库必须与你要保护的虚拟机位于同一区域中。如果你的虚拟机位于不同的区域中，请在每个区域中创建一个保管库。无需指定存储帐户即可存储备份数据 — 备份保管库和 Azure 备份服务将会自动处理这种情况。

  	![创建备份保管库][create-vault]

  	> [AZURE.NOTE]使用 Azure 备份服务的虚拟机备份仅在选定区域受支持。在创建保管库期间，如果你要寻找的区域目前不受支持，则不会在下拉列表中显示它。

5. 单击“创建保管库”。创建备份保管库可能需要一段时间。可以在门户底部监视状态通知。

	![创建保管库 toast 通知][create-vault-toast]

6. 将出现一条消息来确认保管库已成功创建，并且将在“恢复服务”页上将保管库列出为“活动”保管库。

	![备份保管库列表][vault-list]

7. 单击备份保管库将转到“快速启动”页，其中会显示 Azure 虚拟机的备份说明。

	![“仪表板”页中的虚拟机备份说明][vmbackup-instructions]

  	> [AZURE.NOTE]确保在创建保管库后立即选择适当的存储冗余选项。阅读有关[在备份保管库中设置存储冗余选项][vault-storage-redundancy]的更多内容。

### 2.VM 代理
在开始备份 Azure 虚拟机之前，请确保 Azure VM 代理已正确安装到虚拟机上。为了备份虚拟机，Azure 备份服务会将扩展安装到 VM 代理上。由于创建虚拟机时 VM 代理是可选组件，因此需确保选中 VM 代理的复选框，然后才能对虚拟机进行预配。

了解 [VM 代理][vmagent-doc]以及[如何安装它][vmagent-howtoinstall]。

> [AZURE.NOTE]如果你打算将虚拟机从本地数据中心迁移到 Azure，请确保在启动迁移过程之前，下载并安装 VM 代理 MSI。这也适用于受到使用 Azure Site Recovery 的 Azure 保护的虚拟机。

## 预览期间的限制

+ 不支持备份超过 6 个磁盘的虚拟机。

+ 不支持备份使用“高级”存储空间的虚拟机。

+ 不支持在恢复过程中替换现有虚拟机。首先删除现有虚拟机以及任何关联的磁盘，然后从备份恢复数据。

+ 使用 Azure Site Recovery 备份恢复的虚拟机。

+ 不支持跨区域备份和恢复。

+ 使用 Azure 备份服务的虚拟机备份仅在选定区域受支持。在创建保管库期间，如果你要寻找的区域目前不受支持，则不会在下拉列表中显示它。

+ 注册脱机虚拟机将失败。必须运行虚拟机才能成功完成注册过程。


## 后续步骤
若要开始虚拟机备份，请学习如何：

- [发现、注册和保护虚拟机](backup-azure-vms)

- [恢复虚拟机](backup-azure-restore-vms)

+ 监视备份作业




[mgmt-portal]: http://manage.windowsazure.cn/
[vm-doc]: /documentation/services/virtual-machines/
[azure-backup-pricing]: /home/features/back-up/#price
[vault-storage-redundancy]: /documentation/articles/backup-azure-backup-create-vault#azure-backup---storage-redundancy-options
[vmagent-doc]: https://msdn.microsoft.com/zh-cn/library/dn606311.aspx
[vmagent-howtoinstall]: http://azure.microsoft.com/blog/2014/04/15/vm-agent-and-extensions-part-2/

[vm-backup-arch]: ./media/backup-azure-vms-introduction/vmbackup-architecture.png
[backup-entities]: ./media/backup-azure-vms-introduction/vault-policy-vm.png
[create-vault]: ./media/backup-azure-vms-introduction/backup_vaultcreate.png
[create-vault-toast]: ./media/backup-azure-vms-introduction/creating-vault.png
[vault-list]: ./media/backup-azure-vms-introduction/backup_vaultslist.png
[vmbackup-instructions]: ./media/backup-azure-vms-introduction/vmbackup-instructions.png

<!---HONumber=60-->