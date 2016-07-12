<properties
	pageTitle="将 VMM 云中的 Hyper-V 虚拟机复制到 Azure | Azure"
	description="本文介绍如何将位于 System Center VMM 云中的 Hyper-V 主机上的 Hyper-V 虚拟机复制到 Azure。"
	services="site-recovery"
	documentationCenter=""
	authors="rayne-wiselman"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="site-recovery"
	ms.date="05/06/2016"
	wacn.date="06/06/2016"/>

#  将 VMM 云中的 Hyper-V 虚拟机复制到 Azure

> [AZURE.SELECTOR]

- [Azure 管理门户](/documentation/articles/site-recovery-vmm-to-azure-classic/)
- [PowerShell](/documentation/articles/site-recovery-deploy-with-powershell/)



Azure Site Recovery 服务有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。虚拟机可复制到 Azure 中，也可复制到本地数据中心中。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)。

## 概述

本文说明如何部署 Site Recovery，以便将位于 VMM 私有云中的 Hyper-V 主机服务器上的 Hyper-V 虚拟机复制到 Azure。

文中包括了方案的先决条件并展示了如何设置 Site Recovery 保管库，在源 VMM 服务器上安装 Azure Site Recovery 提供程序，在保管库中注册服务器，添加 Azure 存储帐户，在 Hyper-V 主机服务器上安装 Azure 恢复服务代理，为 VMM 云配置将应用于所有受保护虚拟机的保护设置，然后为这些虚拟机启用保护。最后将测试故障转移以确保一切都正常工作。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/forums/zh-cn/home?forum=hypervrecovmgr)。

## 体系结构

![体系结构](./media/site-recovery-vmm-to-azure-classic/topology.png)

- 于 Site Recovery 部署期间在 VMM 上安装 Azure Site Recovery 提供程序，并在 Site Recovery 保管库中注册 VMM 服务器。提供程序会与 Site Recovery 通信以处理复制业务流程。
- 于 Site Recovery 部署期间在 Hyper-V 主机服务器上安装 Azure 恢复服务代理。它会处理到 Azure 存储空间的数据复制。


## Azure 先决条件

以下是你在 Azure 中需要的内容。

**先决条件** | **详细信息**
--- | ---
**Azure 帐户**| 需要一个 [Azure](https://azure.cn/) 帐户。你可以从[试用版](/pricing/1rmb-trial/)开始。[详细了解](/home/features/site-recovery/pricing/) Site Recovery 定价。
**Azure 存储空间** | 你将需要使用 Azure 存储帐户来存储复制的数据。复制的数据存储在 Azure 空间，Azure VM 在发生故障转移时启动。<br/><br/>你需要[标准异地冗余存储帐户](/documentation/articles//storage-redundancy#geo-redundant-storage)。该帐户必须位于 Site Recovery 服务所在的同一区域，并与同一订阅相关联。请注意，目前不支持复制到高级存储帐户，因此不应使用该功能。<br/><br/>[了解](/documentation/articles/storage-introduction/) Azure 存储空间。
**Azure 网络** | 你需要一个 Azure 虚拟网络，以便发生故障转移时 Azure VM 能够连接到其中。Azure 虚拟网络必须位于与 Site Recovery 保管库相同的区域中。

## 本地先决条件

以下是你在本地需要的内容。

**先决条件** | **详细信息**
--- | ---
**VMM** | 你至少需要一台部署为单独的物理或虚拟服务器（或虚拟群集）的 VMM 服务器。<br/><br/>VMM 服务器应运行安装了最新累积更新的 System Center 2012 R2。<br/><br/>需要在 VMM 服务器上配置至少一个云。<br/><br/>要保护的源云必须包含一个或多个 VMM 主机组。<br/><br/>了解有关如何设置 VMM 云的详细信息，请参阅[演练：使用 System Center 2012 SP1 VMM 创建私有云](http://blogs.technet.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)（Keith Mayer 博客文章）。
**Hyper-V** | 你在 VMM 云中需要一或多个 Hyper-V 主机服务器或群集。主机服务器应该有一或多个 VM。<br/><br/>Hyper-V 服务器必须至少运行包含 Hyper-V 角色的 Windows Server 2012 R2，并安装了最新更新。<br/><br/>任何包含你所要保护的 VM 的 Hyper-V 服务器都必须位于 VMM 云中。<br/><br/>如果你要在群集中运行 Hyper-V，请注意，如果你的群集是静态的基于 IP 地址的群集，则不会自动创建群集代理。你需要手动配置群集代理。请阅读 Aidan Finn 的博客文章[了解详细信息](https://www.petri.com/use-hyper-v-replica-broker-prepare-host-clusters)。
**受保护的计算机** | 你想要保护的 VM 应该符合 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。


## 网络映射先决条件
当在 Azure 中保护虚拟机时，网络映射会在源 VMM 服务器上的 VM 网络与目标 Azure 网络之间进行映射以实现以下功能：

- 在同一网络上进行故障转移的所有计算机都可以彼此连接到对方，不管它们位于哪个恢复计划中。
- 如果在目标 Azure 网络上设置了网络网关，则虚拟机可以连接到其他本地虚拟机。
- 如果没有配置网络映射，则只有在同一恢复计划中进行故障转移的虚拟机能够在故障转移到 Azure 后彼此连接到对方。

如果希望部署网络映射，需要满足下列条件：

- 源 VMM 服务器上你要保护的虚拟机应当连接到某个 VM 网络。该网络应当该链接到与该云相关联的逻辑网络。
- 具有在故障转移后复制的虚拟机可以连接到的 Azure 网络。你将在故障转移时选择此网络。此网络应当与你的 Azure Site Recovery 订阅位于同一区域中。

准备网络映射，如下所示：

1. [了解](/documentation/articles/site-recovery-network-mapping/)网络映射要求。
2. 在 VMM 中准备 VM 网络：

	- [设置逻辑网络](https://technet.microsoft.com/zh-cn/library/jj721568.aspx)。
	- [设置 VM 网络](https://technet.microsoft.com/zh-cn/library/jj721575.aspx)。


## 步骤 1：创建站点恢复保管库

1. 从你要注册的 VMM 服务器登录到[管理门户](https://manage.windowsazure.cn)。
2. 单击“数据服务”>“恢复服务”>“Site Recovery 保管库”。
3. 单击“新建”>“快速创建”。
4. 在“名称”中，输入一个友好名称以标识此保管库。
5. 在“区域”中，为保管库选择地理区域。若要查看受支持的区域，请参阅 Azure Site Recovery 价格详细信息中的“地域可用性”[](/home/features/site-recovery/pricing/)。
6. 单击“创建保管库”。

	![新保管库](./media/site-recovery-vmm-to-azure-classic/create-vault.png)

检查状态栏以确认保管库已成功创建。保管库将以“活动”形式列在主要的“恢复服务”页上。

## 步骤 2：生成保管库注册密钥

在保管库中生成一个注册密钥。在下载 Azure Site Recovery 提供程序并将其安装到 VMM 服务器上后，你将使用此密钥在保管库中注册 VMM 服务器。

1. 在“恢复服务”页中，单击保管库以打开“快速启动”页。也可随时使用该图标打开“快速启动”。

	![“快速启动”图标](./media/site-recovery-vmm-to-azure-classic/qs-icon.png)

2. 在下拉列表中，选择“本地 VMM 站点与 Microsoft Azure 之间”。
3. 在“准备 VMM 服务器”中，单击“生成注册密钥文件”。密钥文件将自动生成并且自生成后在 5 天内有效。如果你不是从 VMM 服务器访问 Azure 门户，则需要将此文件复制到服务器。

	![注册密钥](./media/site-recovery-vmm-to-azure-classic/register-key.png)

## 步骤 3：安装 Azure Site Recovery 提供程序

1. 在“快速启动”>“准备 VMM 服务器”中，单击“下载用于在 VMM 服务器上安装的 Microsoft Azure Site Recovery 提供程序”来获取最新版本的提供程序安装文件。
2. 在源 VMM 服务器上运行此文件。

	>[AZURE.NOTE] 如果 VMM 部署到群集中并且你是首次安装该提供程序，请将其安装在一个活动节点上并完成安装以在保管库中注册 VMM 服务器。然后在其他节点上安装该提供程序。请注意，如果你是在升级提供程序，则需要在所有节点上进行升级，因为所有节点都应当运行相同的提供程序版本。
	
3. 安装程序将执行先决条件检查，并请求授权停止 VMM 服务以开始安装提供程序。VMM 服务将在安装程序完成时自动重新启动。如果你是在 VMM 群集上进行安装，则会提示你停止群集角色。

4. 在“Microsoft 更新”中，你可以选择获取更新。当启用了此设置时，将根据你的 Microsoft 更新策略自动安装提供程序更新。

	![Microsoft 更新](./media/site-recovery-vmm-to-azure-classic/updates.png)


5.  提供程序的安装位置设置为 **<SystemDrive>\\Program Files\\Microsoft System Center 2012 R2\\Virtual Machine Manager\\bin**。单击“安装”。

	![InstallLocation](./media/site-recovery-vmm-to-azure-classic/install-location.png)

6. 安装提供程序之后，请单击“注册”，以在保管库中注册服务器。

	![InstallComplete](./media/site-recovery-vmm-to-azure-classic/install-complete.png)

7. 在“Internet 连接”中，指定在 VMM 服务器上运行的提供程序如何连接到 Internet。选择“使用默认系统代理设置”以使用服务器上配置的默认 Internet 连接设置。

	![Internet 设置](./media/site-recovery-vmm-to-azure-classic/proxy.png)

	- 如果希望使用自定义代理，则应当在安装该提供程序之前设置它。当配置自定义代理设置时，会运行测试来检查代理连接。
	- 如果你确实使用自定义代理，或者你的默认代理要求进行身份验证，则需要输入代理详细信息，包括代理地址和端口。
	- 以下 URL 应可从 VMM 服务器和 Hyper-v 主机访问
		- *.hypervrecoverymanager.windowsazure.com
		- *.accesscontrol.chinacloudapi.cn
		- *.backup.windowsazure.cn
		- *.blob.core.chinacloudapi.cn
		- *.store.core.chinacloudapi.cn
	- 允许 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)中所述的 IP 地址，以及 HTTPS (443) 协议。

	- 如果你使用自定义代理，则将使用指定的代理凭据自动创建一个 VMM 运行身份帐户 (DRAProxyAccount)。对代理服务器进行配置以便该帐户可以成功通过身份验证。可以在 VMM 控制台中修改 VMM 运行身份帐户设置。若要执行此操作，请打开“设置”工作区，展开“安全性”，单击“运行身份帐户”，然后修改 DRAProxyAccount 的密码。你将需要重新启动 VMM 服务以使此设置生效。

8. 在“注册密钥”中，选择你从 Azure Site Recovery 下载并复制到 VMM 服务器的密钥。
9. 在“保管库名称”中，验证将要在其中注册服务器的保管库的名称。

	![服务器注册](./media/site-recovery-vmm-to-azure-classic/credentials.png)

10. 可以指定一个位置，用于保存为数据加密自动生成的 SSL 证书。如果你在 Site Recovery 部署期间为 VMM 云启用数据加密，则会使用此证书。请确保该证书安全。当你运行到 Azure 的故障转移时，将选择该证书以便对加密的数据进行解密。

	![服务器注册](./media/site-recovery-vmm-to-azure-classic/encryption.png)

11. 在“服务器名称”中，指定一个友好名称以在保管库中标识该 VMM 服务器。在群集配置中，请指定 VMM 群集角色名称。

12. 在“初始云元数据同步”中，选择是否要将 VMM 服务器上所有云的元数据与保管库进行同步。此操作在每个服务器上只需执行一次。如果你不希望同步所有云，可以将此设置保留为未选中状态并在 VMM 控制台中的云属性中分别同步各个云。

	![服务器注册](./media/site-recovery-vmm-to-azure-classic/friendly.png)

13. 单击“下一步”以完成此过程。注册后，Azure Site Recovery 将检索 VMM 服务器中的元数据。服务器显示在保管库中“服务器”页上的“VMM 服务器”选项卡中。

### 命令行安装

也可使用以下命令行来安装 Azure Site Recovery 提供程序。此方法可用来将提供程序安装在 Server Core for Windows Server 2012 R2 上。

1. 将提供程序安装文件和注册密钥下载到某个文件夹中。例如 C:\\ASR。
2. 停止 System Center Virtual Machine Manager 服务
3. 从提升的命令提示符处，使用下列命令提取提供程序安装程序：

    	C:\Windows\System32> CD C:\ASR
    	C:\ASR> AzureSiteRecoveryProvider.exe /x:. /q

4. 安装提供程序，如下所示：

		C:\ASR> setupdr.exe /i

5. 注册提供程序，如下所示：

    	CD C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin
    	C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin> DRConfigurator.exe /r  /Friendlyname <friendly name of the server> /Credentials <path of the credentials file> /EncryptionEnabled <full file name to save the encryption certificate>       

参数如下所示：

 - **/Credentials**：用于指定注册密钥文件所在位置的必需参数  
 - **/FriendlyName**：在 Azure Site Recovery 门户中显示的 Hyper-V 主机服务器名称的必需参数。
 - **/EncryptionEnabled**：可选参数，用于指定是否要在 Azure 中对虚拟机加密（静态加密）。文件名应该有 **.pfx** 扩展名。
 - **/proxyAddress**：可选参数，用于指定代理服务器的地址。
 - **/proxyport**：可选参数，用于指定代理服务器的端口。
 - **/proxyUsername**：可选参数，用于指定代理用户名。
 - **/proxyPassword**：可选参数，用于指定代理密码。  


## 步骤 4：创建 Azure 存储帐户

1. 如果你没有 Azure 存储帐户，请单击“添加 Azure 存储帐户”创建一个。
2. 创建帐户并启用异地复制。该帐户必须位于 Azure Site Recovery 服务所在的同一区域，并与同一订阅相关联。

	![存储帐户](./media/site-recovery-vmm-to-azure-classic/storage.png)

## 步骤 5：安装 Azure 恢复服务代理

在 VMM 云中的每个 Hyper-V 主机服务器上安装 Azure 恢复服务代理。

1. 单击“快速启动”>“下载 Azure Site Recovery 服务代理并安装在主机上”，以获取最新版本的代理安装文件。

	![安装恢复服务代理](./media/site-recovery-vmm-to-azure-classic/install-agent.png)

2. 在每个 Hyper-V 主机服务器上运行安装文件。
3. 在“先决条件检查”页上，单击“下一步”。将自动安装任何缺少的必备组件。

	![恢复服务代理必备组件](./media/site-recovery-vmm-to-azure-classic/agent-prereqs.png)

4. 在“安装设置”页上，指定要安装代理的位置，并选择将在其中安装备份元数据的缓存位置。然后单击“安装”。
5. 安装完成之后，单击“关闭”以完成向导。

	![注册 MARS 代理](./media/site-recovery-vmm-to-azure-classic/agent-register.png)

### 命令行安装

你也可以使用下列命令，从命令行安装 Azure 恢复服务代理：

    marsagentinstaller.exe /q /nu

## 步骤 6：配置云保护设置

在注册 VMM 服务器后，你可以配置云保护设置。你在安装提供程序时启用了“将云数据与保管库同步”选项，所以 VMM 服务器上的所有云都将出现在保管库中的“受保护的项”选项卡中。<b></b>

![已发布的云](./media/site-recovery-vmm-to-azure-classic/clouds-list.png)

1. 在“快速启动”页上，单击“为 VMM 云设置保护”。
2. 在“受保护的项”选项卡上，单击你要配置的云，然后转到“配置”选项卡。
3. 在“目标”中，选择“Azure”。
4. 在“存储帐户”中，选择用于复制的 Azure 存储帐户。
5. 将“加密存储的数据”设置为“关闭”。此设置指定应该加密在本地站点与 Azure 之间复制的数据。
6. 在“复制频率”中，保留默认设置。此值指定数据应在源和目标位置之间同步的频率。
7. 在“恢复点保留时长”中，保留默认设置。当默认值为零时，副本主机服务器上只存储主虚拟机的最新恢复点。
8. 在“应用程序一致性快照频率”中，保留默认设置。此值指定创建快照的频率。快照使用卷影复制服务 (VSS) 来确保应用程序在拍摄快照时处于一致状态。如果确实要设置一个值，请确保该值小于你配置的附加恢复点数。
9. 在“复制开始时间”中，指定应开始向 Azure 进行初始数据复制的时间。将使用 Hyper-V 主机服务器上的时区。我们建议你将初始复制安排在非高峰时段进行。

	![云复制设置](./media/site-recovery-vmm-to-azure-classic/cloud-settings.png)

在保存设置后，将创建一个作业，可以在“作业”选项卡上监视该作业。VMM 源云中的所有 Hyper-V 主机服务器将为复制进行配置。

保存后，可以在“配置”选项卡上修改云设置。若要修改目标位置或目标存储帐户，需要删除云配置，然后重新配置云。请注意，如果你更改存储帐户，在修改存储帐户后，只对已启用保护的虚拟机应用更改。现有虚拟机不会迁移到新的存储帐户。

## 步骤 7：配置网络映射
在开始网络映射之前，请验证源 VMM 服务器上的虚拟机是否已连接到一个 VM 网络。此外，请创建一个或多个 Azure 虚拟机。注意，可以将多个 VM 网络映射到单个 Azure 网络。

1. 在“快速启动”页上，单击“映射网络”。
2. 在“网络”选项卡上，在“源位置”中，选择源 VMM 服务器。在“目标位置”中，选择“Azure”。
3. 在“源网络”中，会显示与 VMM 服务器关联的 VM 网络的列表。在“目标网络”中，会显示与订阅关联的 Azure 网络。
4. 选择源 VM 网络并单击“映射”。
5. 在“选择目标网络”页上，选择要使用的目标 VM 网络。
6. 单击复选标记以完成映射过程。

	![云复制设置](./media/site-recovery-vmm-to-azure-classic/map-networks.png)

在保存设置后，将启动一个作业来跟踪映射进度，可以在“作业”选项卡上监视该作业。与源 VM 网络对应的任何现有副本虚拟机都将连接到目标 Azure 网络。在复制后，连接到源 VM 网络的新虚拟机将连接到映射的 Azure 网络。如果你修改了与新网络之间的映射，则会使用新设置来连接副本虚拟机。

请注意，如果目标网络具有多个子网，并且其中一个子网与源虚拟机所在的子网同名，则在故障转移后副本虚拟机将连接到该目标子网。如果没有具有匹配名称的目标子网，则虚拟机将连接到网络中的第一个子网。

## 步骤 8：为虚拟机启用保护

在正确配置服务器、云和网络后，可以在云中为虚拟机启用保护。注意以下事项：

- 虚拟机必须满足 [Azure 要求](/documentation/articles/site-recovery-best-practices/#azure-virtual-machine-requirements)。
- 若要启用保护，必须为虚拟机设置操作系统和操作系统磁盘属性。当你使用虚拟机模板在 VMM 中创建虚拟机时，可以设置属性。也可以在虚拟机属性的“常规”和“硬件配置”选项卡中为现有虚拟机设置这些属性。如果未在 VMM 中设置这些属性，可以在 Azure Site Recovery 门户中配置它们。

	![创建虚拟机](./media/site-recovery-vmm-to-azure-classic/enable-new.png)

	![修改虚拟机属性](./media/site-recovery-vmm-to-azure-classic/enable-existing.png)


1. 若要启用保护，请在虚拟机所在的云中的“虚拟机”选项卡上，单击“启用保护”>“添加虚拟机”。
2. 从云中的虚拟机列表中，选择要保护的虚拟机。

	![启用虚拟机保护](./media/site-recovery-vmm-to-azure-classic/select-vm.png)

	在“作业”选项卡中跟踪“启用保护”操作的进度，包括初始复制。在“完成保护”作业运行之后，虚拟机就可以进行故障转移了。在启用保护并复制虚拟机后，你将能够在 Azure 中查看它们。


	![虚拟机保护作业](./media/site-recovery-vmm-to-azure-classic/vm-jobs.png)

3. 验证虚拟机属性并根据需要进行修改。

	![验证虚拟机](./media/site-recovery-vmm-to-azure-classic/vm-properties.png)


4. 在虚拟机属性的“配置”选项卡上可以修改以下网络属性。





- **目标虚拟机的网络适配器数目** - 网络适配器数目根据你为目标虚拟机指定的大小来确定。查看[虚拟机大小规格](/documentation/articles//virtual-machines-linux-sizes#size-tables)，了解虚拟机大小所支持的适配器数目。修改虚拟机的大小并保存设置后，下一次打开“配置”页时，网络适配器的数量将会改变。目标虚拟机的网络适配器数目是源虚拟机上网络适配器的最小数目和所选虚拟机大小支持的网络适配器的最大数目，如下所示：

	- 如果源计算机上的网络适配器数小于或等于目标计算机大小允许的适配器数，则目标的适配器数将与源相同。
	- 如果源虚拟机的适配器数大于目标大小允许的数目，则使用目标大小允许的最大数目。
	- 例如，如果源计算机有两个网络适配器，而目标计算机大小支持四个，则目标计算机将有两个适配器。如果源计算机有两个适配器，但支持的目标大小只支持一个，则目标计算机只有一个适配器。 	

- **目标虚拟机的网络** - 虚拟机连接的网络取决于源虚拟机网络的网络映射。如果源虚拟机有多个网络适配器，并且源网络已映射到目标上的不同网络，则必须选择其中一个目标网络。
- **每个网络适配器的子网** - 对于每个网络适配器，你可以选择故障转移的虚拟机要连接到的子网。
- **目标 IP 地址** - 如果源虚拟机的网络适配器配置为使用静态 IP 地址，那么，你可以提供目标虚拟机的 IP 地址。借助此功能，可以在故障转移之后保留源虚拟机的 IP 地址。如果未提供任何 IP 地址，在故障转移时会将任何可用的 IP 地址提供给网络适配器。如果指定了目标 IP 地址，但该地址已被 Azure 中运行的其他虚拟机使用，则故障转移会失败。  

	![修改网络属性](./media/site-recovery-vmm-to-azure-classic/multi-nic.png)

>[AZURE.NOTE] 不支持具有静态 IP 地址的 Linux 虚拟机。

## 测试部署

若要测试你的部署，可针对一个虚拟机运行测试故障转移，或者创建一个包括多个虚拟机的恢复计划并针对该计划运行测试故障转移。

测试故障转移在隔离的网络中模拟你的故障转移和恢复机制。请注意：

- 如果想要在故障转移之后使用远程桌面连接到 Azure 中的虚拟机，请在虚拟机上启用远程桌面连接，然后运行测试故障转移。
- 在故障转移后，你将使用公共 IP 地址通过远程桌面连接到 Azure 中的虚拟机。如果要执行此操作，请确保没有任何域策略阻止你使用公共地址连接到虚拟机。

>[AZURE.NOTE] 在为 Azure 执行故障转移时，若要获得最佳性能，请确保已在受保护计算机中安装 Azure 代理。这有助于加快启动速度，并且也对出现问题时的诊断有所帮助。Linux 代理可在[此处](https://github.com/Azure/WALinuxAgent)找到 - Windows 代理可在[此处](http://go.microsoft.com/fwlink/?LinkID=394789)找到。

### 创建恢复计划

1. 在“恢复计划”选项卡上，添加一个新计划。指定一个名称，在“源类型”中指定“VMM”，在“源”中指定源 VMM 服务器。目标将是 Azure。

	![创建恢复计划](./media/site-recovery-vmm-to-azure-classic/recovery-plan1.png)

2. 在“选择虚拟机”页上，选择要添加到恢复计划的虚拟机。这些虚拟机将添加到恢复计划的默认组（组 1）中。最多将测试单个恢复计划中的 100 个虚拟机。

	- 如果希望在将虚拟机添加到计划之前验证虚拟机属性，请在虚拟机所在云的属性页上单击该虚拟机。你还可以在 VMM 控制台中配置虚拟机属性。
	- 显示的所有虚拟机都已启用了保护。此列表包括已启用了保护且已完成初始复制的虚拟机和已启用了保护但未完成初始复制的那些虚拟机。在执行恢复计划期间，只有已完成了初始复制的虚拟机可以进行故障转移。


	![创建恢复计划](./media/site-recovery-vmm-to-azure-classic/select-rp.png)

在创建恢复计划后，它将出现在“恢复计划”选项卡中。你还可以将 [Azure 自动化 Runbook](/documentation/articles/site-recovery-runbook-automation/) 添加到恢复计划，以自动执行故障转移期间的操作。

### 运行测试故障转移

可通过两种方式运行到 Azure 的测试故障转移。

- **没有 Azure 网络的测试故障转移** — 此类型的测试故障转移检查虚拟机是否可以在 Azure 中正常运行。在故障转移后，虚拟机不会连接到任何 Azure 网络。
- **具有 Azure 网络的测试故障转移** — 此类型的故障转移检查整个复制环境是否可以按预期运行，以及在故障转移后，虚拟机是否能连接到指定的目标 Azure 网络。对于测试故障转移的子网处理，将根据副本虚拟机的子网确定测试虚拟机的子网。这不同于常规复制，在常规复制中，副本虚拟机的子网是根据源虚拟机的子网确定的。

如果你希望在不指定 Azure 目标网络的情况下为启用了保护的虚拟机运行到 Azure 的测试故障转移，则不需要做任何准备。若要运行具有目标 Azure 网络的测试性故障转移，你将需要创建一个与你的 Azure 生产网络相隔离的新 Azure 网络（你在 Azure 中新建网络时的默认行为）。看看如何[运行测试性故障转移](/documentation/articles/site-recovery-failover/#run-a-test-failover)，以获取更多详细信息。


你还需要设置已复制虚拟机的基础结构，使之能够正常工作。使用，可以使用 Azure Site Recovery 将包含域控制器和 DNS 的虚拟机复制到 Azure，并可以使用测试故障故障转移在测试网络中创建此类虚拟机。如需更多详细信息，请参阅 [Active Directory 的测试性故障转移注意事项](/documentation/articles/site-recovery-active-directory/#considerations-for-test-failover)部分。

若要运行测试故障转移，请执行以下操作：

1. 在“恢复计划”选项卡上，选择该计划并单击“测试故障转移”。
2. 在“确认测试故障转移”页上，选择“无”或选择一个特定的 Azure 网络。请注意，如果你选择了“无”，则测试故障转移将检查虚拟机是否可以正确复制到 Azure，但不会检查你的复制网络配置。

	![无网络](./media/site-recovery-vmm-to-azure-classic/test-no-network.png)

3. 如果为云启用了数据加密，请在“加密密钥”中选择你在打开此选项为云启用数据加密时在 VMM 服务器上安装提供程序期间颁发的证书。
4. 在“作业”选项卡上，你可以跟踪故障转移进度。在 Azure 门户中，你应当也能够看到虚拟机测试副本。如果你已设置为从本地网络访问虚拟机，则可以启动与虚拟机的远程桌面连接。
5. 在故障转移到达“完成测试”阶段时，单击“完成测试”以结束测试故障转移。你可以向下钻取到“作业”选项卡来跟踪故障转移进度和状态以及执行所需的任何操作。
6. 在故障转移后，你将能够在 Azure 门户中看到虚拟机测试副本。如果你已设置为从本地网络访问虚拟机，则可以启动与虚拟机的远程桌面连接。请执行以下操作：

    1. 验证虚拟机成功启动。
    2. 如果想要在故障转移之后使用远程桌面连接到 Azure 中的虚拟机，请在虚拟机上启用远程桌面连接，然后运行测试故障转移。还需要在虚拟机上添加 RDP 终结点。你可以利用 [Azure 自动化 Runbook](/documentation/articles/site-recovery-runbook-automation/) 来执行此操作。
    3. 故障转移后，如果想要在远程桌面中使用公共 IP 地址连接到 Azure 中的虚拟机，请确保没有任何域策略阻止你使用公共地址连接到虚拟机。

7.  完成测试后，执行以下操作：
	- 单击“测试故障转移已完成”。清理测试环境以自动关闭电源，并删除测试虚拟机。
	- 单击“说明”以记录并保存与测试故障转移相关联的任何观测结果。

## 后续步骤

了解[设置恢复计划](/documentation/articles/site-recovery-create-recovery-plans/)和[故障转移](/documentation/articles/site-recovery-failover/)。

<!---HONumber=Mooncake_0530_2016-->