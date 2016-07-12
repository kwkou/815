<properties 
	pageTitle="将 Hyper-V 虚拟机（位于 VMM 云中）复制到辅助 VMM 站点 | Azure"
	description="本文介绍如何通过 Azure Site Recovery 将 VMM 云中的 Hyper-V VM 复制到辅助 VMM 站点。"
	services="site-recovery" 
	documentationCenter="" 
	authors="raynew" 
	manager="jwhit" 
	editor="tysonn"/>

<tags
	ms.service="site-recovery"
	ms.date="02/17/2016"
	wacn.date="04/05/2016"/>

# 将 VMM 云中的 Hyper-V 虚拟机复制到辅助 VMM 站点

Azure Site Recovery 服务有助于业务连续性和灾难恢复 (BCDR) 策略，因为它可以协调虚拟机和物理服务器的复制、故障转移和恢复。虚拟机可复制到 Azure 中，也可复制到本地数据中心中。如需快速概览，请阅读[什么是 Azure Site Recovery？](/documentation/articles/site-recovery-overview/)。

## 概述

本文介绍如何使用 Azure Site Recovery 将 VMM 云中管理的 Hyper-V 主机服务器上的 Hyper-V 虚拟机复制到辅助 VMM 站点。

本文包括了各种先决条件并展示了如何设置 Site Recovery 保管库，在源和目标 VMM 服务器上安装 Azure Site Recovery 提供程序，在保管库中注册服务器，为 VMM 云配置保护设置，然后为 Hyper-V VM 启用保护。最后将测试故障转移以确保一切都正常工作。

请将任何评论或问题发布到本文底部，或者发布到 [Azure 恢复服务论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=hypervrecovmgr)。

## 体系结构

以下图片显示了Azure Site Recovery 用来完成业务流程和复制的不同通信通道和端口

![E2E 拓扑](./media/site-recovery-vmm-to-vmm/e2e-topology.png)

## 开始之前

确保已满足以下先决条件：

**先决条件** | **详细信息** 
--- | ---
**Azure**| 需要一个 [Azure](https://azure.cn/) 帐户。你可以从 [1rmb 试用版](/pricing/1rmb-trial/)开始。[详细了解](/home/features/site-recovery/pricing/) Site Recovery 定价。 
**VMM** | 你将需要至少一个 VMM 服务器。<br/><br/>VMM 服务器应至少运行安装了最新累积更新的 System Center 2012 SP1。<br/><br/>如果你需要对单个 VMM 服务器设置保护，则需在服务器上至少配置两个云。<br/><br/>如果你需要为两个 VMM 服务器部署保护，则每个服务器都必须将至少一个云配置在所要保护的主 VMM 服务器上，一个云配置在需要用于保护和恢复的辅助 VMM 服务器上<br/><br/>所有 VMM 云都必须设置 Hyper-V 容量配置文件。<br/><br/>要保护的源云必须包含一个或多个 VMM 主机组。<br/><br/>若要详细了解如何设置 VMM 云，请参阅 Keith Mayer 的博客中的[演练：使用 System Center 2012 SP1 VMM 创建私有云](http://blogs.technet.com/b/keithmayer/archive/2013/04/18/walkthrough-creating-private-clouds-with-system-center-2012-sp1-virtual-machine-manager-build-your-private-cloud-in-a-month.aspx)。
**Hyper-V** | 你需要在主 VMM 主机组和辅助 VMM 主机组中有一个或多个 Hyper-V 主机服务器，在每个 Hyper-V 主机服务器中有一个或多个虚拟机。<br/><br/>主机和目标 Hyper-V 服务器必须至少运行包含 Hyper-V 角色的 Windows Server 2012，并安装了最新更新。<br/><br/>任何包含你所要保护的 VM 的 Hyper-V 服务器都必须位于 VMM 云中。<br/><br/>如果你要在群集中运行 Hyper-V，请注意，如果你的群集是静态的基于 IP 地址的群集，则不会自动创建群集中转站。你需要手动配置群集代理。在 Aidan Finn 的博客文章中[了解详细信息](https://www.petri.com/use-hyper-v-replica-broker-prepare-host-clusters)。
**网络映射** | 你可以配置网络映射，以确保在故障转移后以最佳方式将复制的虚拟机放置在辅助 Hyper-V 主机服务器上，并确保它们连接到适当的 VM 网络。如果不配置网络映射，则在故障转移后，副本 VM 将不会连接到任何网络。<br/><br/>若要在部署期间设置网络映射，请确保源 Hyper-V 主机服务器上的虚拟机连接到 VMM VM 网络。该网络应链接到与云关联的逻辑网络。<br/<br/>辅助 VMM 服务器上用于恢复的目标云应当配置了相应的 VM 网络，并且该网络应当链接到与目标云关联的相应逻辑网络。<br/><br/>[详细了解](/documentation/articles/site-recovery-network-mapping/)网络映射。
**存储映射** | 默认情况下，将源 Hyper-V 主机服务器上的虚拟机复制到目标 Hyper-V 主机服务器时，复制的数据存储到在 Hyper-V 管理器中为目标 Hyper-V 主机指定的默认位置。若要对存储已复制数据的位置进行更多的控制，可以配置存储映射<br/><br/>若要配置存储映射，需要在开始部署之前先在源和目标 VMM 服务器上设置存储分类。[了解详细信息](/documentation/articles/site-recovery-storage-mapping/)。

##<a id="step-1-create-a-site-recovery-vault"></a> 步骤 1：创建站点恢复保管库

1. 从你要注册的 VMM 服务器登录到[管理门户](https://manage.windowsazure.cn)。

2. 展开“数据服务”>“恢复服务”，并单击“Site Recovery 保管库”。

3. 单击“新建”>“快速创建”。
	
4. 在“名称”中，输入一个友好名称以标识此保管库。

5. 在“区域”中，为保管库选择地理区域。若要查看支持的区域，请参阅 [Azure Site Recovery 定价详细信息](/home/features/site-recovery/#home_rec_pri)中的“上市地区”。</a>

6. 单击“创建保管库”。

	![创建保管库](./media/site-recovery-vmm-to-vmm/create-vault.png)

在状态栏中检查该保管库是否已创建。保管库将以“活动”形式列在主要的“恢复服务”页上。

## 步骤 2：生成保管库注册密钥

在保管库中生成一个注册密钥。在下载 Azure Site Recovery 提供程序并将其安装到 VMM 服务器上后，你将使用此密钥在保管库中注册 VMM 服务器。

1. 在“恢复服务”页中，单击保管库以打开“快速启动”页。也可随时使用该图标打开“快速启动”。

	![“快速启动”图标](./media/site-recovery-vmm-to-vmm/quick-start-icon.png)

2. 在下拉列表中，选择“两个本地 Hyper-V 站点之间”。
3. 在“准备 VMM 服务器”中，单击“生成注册密钥文件”。密钥文件将自动生成并且自生成后在 5 天内有效。如果你不是从 VMM 服务器访问 Azure 门户，则需要将此文件复制到服务器。

	![注册密钥](./media/site-recovery-vmm-to-vmm/register-key.png)
	
##<a id="step-3-install-the-azure-site-recovery-provider"></a> 步骤 3：安装 Azure 站点恢复 提供程序	

4. 在“快速启动”页面上的“准备 VMM 服务器”中，单击“下载用于在 VMM 服务器上安装的 Azure Site Recovery 提供程序”来获取最新版本的提供程序安装文件。

2. 在源 VMM 服务器上运行此文件。如果 VMM 部署到群集中并且你是首次安装该提供程序，请将其安装在一个活动节点上并完成安装以在保管库中注册 VMM 服务器。然后在其他节点上安装该提供程序。请注意，如果你是在升级提供程序，则需要在所有节点上进行升级，因为所有节点都应当运行相同的提供程序版本。


3. 安装程序将执行简单的“先决条件检查”，并请求授权停止 VMM 服务以开始安装提供程序。VMM 服务将在安装程序完成时自动重新启动。如果你是在 VMM 群集上进行安装，则会提示你停止群集角色。

4. 在“Microsoft 更新”中，你可以选择获取更新。当启用了此设置时，将根据你的 Microsoft 更新策略自动安装提供程序更新。

	![Microsoft 更新](./media/site-recovery-vmm-to-vmm/ms-update.png)

5. 安装位置设置为 **<SystemDrive>\\Program Files\\Microsoft System Center 2012 R2\\Virtual Machine Manager\\bin**。单击“安装”按钮，开始安装提供程序。

	![InstallLocation](./media/site-recovery-vmm-to-vmm/install-location.png)

6. 安装提供程序之后，请单击“注册”，以在保管库中注册服务器。

	![InstallComplete](./media/site-recovery-vmm-to-vmm/install-complete.png)

7. 在“Internet 连接”中，指定在 VMM 服务器上运行的提供程序如何连接到 Internet。选择“使用现有代理设置进行连接”以使用服务器上配置的默认 Internet 连接设置。

	![Internet 设置](./media/site-recovery-vmm-to-vmm/proxy-details.png)

	- 如果希望使用自定义代理，则应当在安装该提供程序之前设置它。当配置自定义代理设置时，会运行测试来检查代理连接。
	- 如果你确实使用自定义代理，或者你的默认代理要求进行身份验证，则需要输入代理详细信息，包括代理地址和端口。
	- 以下 URL 应可从 VMM 服务器和 Hyper-v 主机访问
		- *.hypervrecoverymanager.windowsazure.cn
		- *.accesscontrol.chinacloudapi.cn
		- *.backup.windowsazure.cn
		- *.blob.core.chinacloudapi.cn
		- *.store.core.chinacloudapi.cn
	- 允许 [Azure 数据中心 IP 范围](http://go.microsoft.com/fwlink/?LinkId=511094)中所述的 IP 地址，以及 HTTPS (443) 协议。必须将你打算使用的 Azure 区域的 IP 范围以及中国东部的 IP 范围加入允许列表。 
	- 如果你使用自定义代理，则将使用指定的代理凭据自动创建一个 VMM 运行身份帐户 (DRAProxyAccount)。对代理服务器进行配置以便该帐户可以成功通过身份验证。可以在 VMM 控制台中修改 VMM 运行身份帐户设置。若要执行此操作，请打开“设置”工作区，展开“安全性”，单击“运行身份帐户”，然后修改 DRAProxyAccount 的密码。你将需要重新启动 VMM 服务以使此设置生效。

8. 在“注册密钥”中，选择你从 Azure Site Recovery 下载并复制到 VMM 服务器的密钥。
9. 在“保管库名称”中，验证将要在其中注册服务器的保管库的名称。单击*“下一步”*。

	![服务器注册](./media/site-recovery-vmm-to-vmm/vault-creds.png)

10.  仅当你要将 VMM 云中的 Hyper-V VM 复制到 Azure 时，才使用加密设置。如果要复制到辅助站点，则不使用加密设置。

	![服务器注册](./media/site-recovery-vmm-to-vmm/encrypt.png)

11.  在“服务器名称”中，指定一个友好名称以在保管库中标识该 VMM 服务器。在群集配置中，请指定 VMM 群集角色名称。
12.  在“同步云元数据”中，选择是否要将 VMM 服务器上所有云的元数据与保管库进行同步。此操作在每个服务器上只需执行一次。如果你不希望同步所有云，可以将此设置保留为未选中状态并在 VMM 控制台中的云属性中分别同步各个云。
 
	![服务器注册](./media/site-recovery-vmm-to-vmm/friendly-name.png)

13.  单击“下一步”以完成此过程。注册后，Azure Site Recovery 将检索 VMM 服务器中的元数据。服务器显示在保管库中“服务器”页上的“VMM 服务器”选项卡中。


### 命令行安装

也可通过以下命令行来安装 Azure Site Recovery 提供程序。此命令可用来将提供程序安装在 Server CORE for Windows Server 2012 R2 上。

1. 将提供程序安装文件和注册密钥下载到某个文件夹中。例如 C:\\ASR。
2. 停止 System Center Virtual Machine Manager 服务
3. 使用 **Administrator** 权限从命令提示符处运行以下命令，以便提取提供程序安装程序：

    	C:\Windows\System32> CD C:\ASR
    	C:\ASR> AzureSiteRecoveryProvider.exe /x:. /q

4. 通过运行以下命令来安装提供程序：

    	C:\ASR> setupdr.exe /i

5. 通过运行以下命令来注册提供程序：

    	CD C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin
    	C:\Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\bin> DRConfigurator.exe /r  /Friendlyname <friendly name of the server> /Credentials <path of the credentials file> /EncryptionEnabled <full file name to save the encryption certificate>     

其中的参数如下：

 - **/Credentials**：用于指定注册密钥文件所在位置的必需参数  
 - **/FriendlyName**：在 Azure Site Recovery 门户中显示的 Hyper-V 主机服务器名称的必需参数。
 - **/EncryptionEnabled**：仅当你需要在 Azure 中以静止方式为虚拟机加密时，才需要在 VMM 到 Azure 方案中使用这个可选参数。请确保提供的文件名具有 **.pfx** 扩展名。
 - **/proxyAddress**：可选参数，用于指定代理服务器的地址。
 - **/proxyport**：可选参数，用于指定代理服务器的端口。
 - **/proxyUsername**：可选参数，用于指定代理服务器用户名（如果代理服务器要求身份验证）。
 - **/proxyPassword**：可选参数，用于指定密码，以便通过代理服务器进行身份验证（如果代理服务器要求身份验证）。  

##<a id="step-4-configure-cloud-protection-settings"></a> 步骤 4：配置云保护设置

在注册 VMM 服务器后，你可以配置云保护设置。如果你在安装提供程序时启用了选项“与保管库同步云数据”，则 VMM 服务器上的所有云都将显示在保管库中的“受保护的项”选项卡上。如果没有启用该选项，则可以在 VMM 控制台中云属性页面的“常规”选项卡中将特定的云同步到 Azure Site Recovery。

![已发布的云](./media/site-recovery-vmm-to-vmm/clouds-list.png)

1. 在“快速启动”页上，单击“为 VMM 云设置保护”。
2. 在“VMM 云”选项卡上，选择你要配置的云并转到“配置”选项卡。
3. 在“目标”中，选择“VMM”。
4. 在“目标位置”中，选择管理着你要用于恢复的云的现场 VMM 服务器。
4. 在“目标云”中，选择要用于源云中虚拟机故障转移的目标云。请注意：

	- 我们建议你选择可满足你要保护的虚拟机的恢复要求的目标云。
	- 一个云只能属于一个云对 — 作为主云或目标云。

5. 在“复制频率”中，指定应在源位置与目标位置之间同步数据的频率。请注意，只有当 Hyper-V 主机运行 Windows Server 2012 R2 时，此设置才适用。对于其他服务器，将使用默认设置五分钟。
6. 在“其他恢复点”中，指定是否要创建其他恢复点。默认值零指定只将主虚拟机的最新恢复点存储在副本主机服务器上。请注意，启用多个恢复点需要为在每个恢复点上存储的快照提供额外的存储。默认情况下，每隔一小时会创建恢复点，因此每个恢复点包含一小时的有用数据。你在 VMM 控制台中为虚拟机分配的恢复点值不应小于你在 Azure Site Recovery 控制台中分配的值。
7. 在“与应用程序一致的快照的频率”中，指定以何频率创建与应用程序一致的快照。Hyper-V 使用两种类型的快照 — 标准快照，它提供整个虚拟机的增量快照；与应用程序一致的快照，它生成虚拟机内的应用程序数据的时间点快照。与应用程序一致的快照使用卷影复制服务 (VSS) 来确保应用程序在拍摄快照时处于一致状态。请注意，如果你启用了与应用程序一致的快照，它将影响在源虚拟机上运行的应用程序的性能。请确保你设置的值小于你配置的额外恢复点的数目。

	![配置保护设置](./media/site-recovery-vmm-to-vmm/cloud-settings.png)

8. 在“数据传输压缩”中，指定是否应压缩所传输的复制数据。
9. 在“身份验证”中，指定如何对主 Hyper-V 主机服务器和恢复 Hyper-V 主机服务器之间的流量进行身份验证。除非你配置了有效的 Kerberos 环境，否则，请选择 HTTPS。Azure Site Recovery 将为 HTTPS 身份验证自动配置证书。不需要手动配置。如果你选择了 Kerberos，则将使用 Kerberos 票证执行主机服务器的相互身份验证。默认情况下，端口 8083 和 8084（用于证书）在 Hyper-V 主机服务器上的 Windows 防火墙中将处于打开状态。请注意，此设置仅适用于在 Windows Server 2012 R2 上运行的 Hyper-V 主机服务器。
10. 在“端口”中，修改源和目标主机计算机用于侦听复制通信的端口号。例如，如果你希望对复制通信应用服务质量 (QoS) 网络带宽限制，可以修改此设置。确认该端口未被任何其他应用程序使用并且在防火墙设置中已打开。
11. 在“复制方法”中，指定在开始定期复制之前将如何处理从源到目标位置的初始数据复制。

	- **通过网络** - 通过网络复制数据会相当耗时且需消耗大量资源。如果云包含的虚拟机所具有的虚拟硬盘相对较小，并且主站点通过较宽的带宽连接到辅助站点，则我们建议你使用此选项。你可以指定复制应当立即启动，或者选择一个时间。如果你使用网络复制，建议你将其安排在非高峰时间进行。
	- **脱机** - 此方法指定使用外部介质执行初始复制。如果你要避免网络性能下降或在地理上处于远程位置，则这种方法很有用。要使用这种方法，请在源云中指定导出位置，并在目标云中指定导入位置。当你为虚拟机启用保护时，虚拟硬盘将复制到指定的导出位置。你将其发送到目标站点，并将其复制到导入位置。系统将导入的信息复制到副本虚拟机。 

12. 选择“删除副本虚拟机”可指定当通过在云属性的“虚拟机”选项卡上选择“删除对虚拟机的保护”选项停止保护虚拟机时应当删除副本虚拟机。启用此设置后，当你禁用保护时，会从 Azure Site Recovery 中删除该虚拟机，从 VMM 控制台中删除该虚拟机的 Site Recovery 设置，并且会删除副本。

	![配置保护设置](./media/site-recovery-vmm-to-vmm/cloud-settings-replica.png)

在保存设置后，将创建一个作业，可以在“作业”选项卡上监视该作业。VMM 源云中的所有 Hyper-V 主机服务器将为复制进行配置。可以在“配置”选项卡上修改云设置。如果希望修改目标位置或目标云，必须删除云配置，然后重新配置该云。

### 为脱机初始复制做准备

你需要执行以下操作来为脱机初始复制做好准备：

- 在源服务器上，你需要指定从中进行数据导出的路径位置。在导出路径上分配对 NTFS 的“完全控制”权限和对 VMM 服务的“共享”权限。在目标服务器上，你需要指定从中进行数据导入的路径位置。在此路径上分配同样的权限。
- 如果导入或导出路径是共享的，请为共享路径所在的远程计算机上的 VMM 服务帐户分配 Administrator、Power User、Print Operator 或 Server Operator 组成员资格。
- 如果你使用任何运行身份帐户来添加主机，请在 VMM 中在导入和导出路径上为运行身份帐户分配读取和写入权限。
- 导入和导出共享不应当位于用作 Hyper-V 主机服务器的任何计算机上，因为 Hyper-V 不支持环回配置。
- 在 Active Directory 中，在包含你要保护的虚拟机的每个 Hyper-V 主机服务器上，启用并配置约束委派以信任导入和导出路径所在的远程计算机，如下所述：
	1. 在域控制器上，打开“Active Directory 用户和计算机”。
	2. 在控制台树中，单击“域名”>“计算机”。
	3. 右键单击 Hyper-V 主机服务器名称 >“属性”。
	4. 在“委托”选项卡上，单击“仅信任此计算机来委派指定的服务”。
	5. 单击“使用任意身份验证协议”。
	6. 单击“添加”>“用户和计算机”。
	7. 键入承载着导出路径的主机的名称，单击“确定”。从可用服务的列表中，按住 CTRL 键并单击“cifs”>“确定”。针对承载着导入路径的主机的名称重复该步骤。根据需要针对其他 Hyper-V 主机服务器重复该步骤。

## 步骤 5：配置网络映射
1. 在“快速启动”页上，单击“映射网络”。
2. 选择你要映射其中的网络的源 VMM 服务器，然后选择要将网络映射到的目标 VMM 服务器。此时将显示源网络及其关联的目标网络的列表。对于当前未映射的网络将显示空值。
3. 在“源上的网络”中选择一个网络，然后选择“映射”。服务将检测目标服务器上的 VM 网络并显示它们。单击源和目标网络名称旁的信息图标以查看每个网络的子网。

	![配置网络映射](./media/site-recovery-vmm-to-vmm/network-mapping1.png)

4. 在对话框中，从目标 VMM 服务器上选择其中一个 VM 网络。

	![选择目标网络](./media/site-recovery-vmm-to-vmm/network-mapping2.png)

5. 当选择某个目标网络时，会显示使用源网络的受保护云。此时也会显示与用于保护的云关联的可用目标网络。建议你选择可供你用于保护的所有云使用的一个目标网络。或者，你也可以转到 VMM 服务器并修改云属性，以添加对应于你要选择的 VM 网络的逻辑网络。
6. 单击复选标记以完成映射过程。作业开始跟踪映射进度。你可以在“作业”选项卡上查看该作业。


## 步骤 6：配置存储映射
默认情况下，将源 Hyper-V 主机服务器上的虚拟机复制到目标 Hyper-V 主机服务器时，复制的数据存储到在 Hyper-V 管理器中为目标 Hyper-V 主机指定的默认位置。若要进一步控制将复制的数据存储在何处，可以配置存储映射，如下所述：



1. 在源和目标 VMM 服务器上定义存储分类。[了解详细信息](http://technet.microsoft.com/zh-cn/library/gg610685.aspx)。分类必须可供源和目标云中的 Hyper-V 主机服务器使用。分类不需要具有相同类型的存储。例如，你可以将包含 SMB 共享的源分类映射到包含 CSV 的目标分类。
2. 在分类就位后，你可以创建映射。为此，请在“快速启动”页上，单击“映射存储”。
3. 单击“存储”选项卡，然后单击“映射存储分类”。
4. 在“映射存储分类”选项卡上，在源和目标 VMM 服务器上选择分类。保存你的设置。

	![选择目标网络](./media/site-recovery-vmm-to-vmm/storage-mapping.png)


##<a id="step-7-enable-virtual-machine-protection"></a> 步骤 7：启用虚拟机保护
在正确配置服务器、云和网络后，可以在云中为虚拟机启用保护。

1. 在虚拟机所在的云中的“虚拟机”选项卡上，单击“启用保护”，然后单击“添加虚拟机”。
2. 从云中的虚拟机列表中，选择要保护的虚拟机。

	![启用虚拟机保护](./media/site-recovery-vmm-to-vmm/enable-protection.png)

3. 在“作业”选项卡中跟踪“启用保护”操作的进度，包括初始复制。在“完成保护”作业运行之后，虚拟机就可以进行故障转移了。在启用保护并复制虚拟机后，你将能够在 Azure 中查看它们。

	![虚拟机保护作业](./media/site-recovery-vmm-to-vmm/vm-jobs.png)
	
>[AZURE.NOTE] 你还可以在 VMM 控制台中为虚拟机启用保护。在虚拟机属性的“Azure Site Recovery”选项卡中，在工具栏上单击“启用保护”。

### 将现有虚拟机加入进来

如果你在 VMM 中已有使用 Hyper-V 副本进行复制的虚拟机，你需要将它们加入到 Azure Site Recovery 保护范围中，如下所述：

1. 验证你是否已具有主云和辅助云。确保承载着现有虚拟机的 Hyper-V 服务器位于主云中，并且承载着副本虚拟机的 Hyper-V 服务器位于辅助云中。确保你已经为云配置了保护设置。这些设置应当与当前为 Hyper-V 副本配置的设置相同。否则，虚拟机复制可能无法按预期方式工作。
2. 然后，为主虚拟机启用保护。Azure Site Recovery 和 VMM 将确保检测到相同的副本主机和虚拟机，并且 Azure Site Recovery 将使用在配置云期间配置的设置重用并重新建立复制。


## 测试你的部署

若要测试你的部署，可针对一台虚拟机运行测试故障转移，或者创建一个包括多个虚拟机的恢复计划并针对该计划运行测试故障转移。测试故障转移在隔离的网络中模拟你的故障转移和恢复机制。

### 创建恢复计划

1. 在“恢复计划”选项卡上，单击“创建恢复计划”。
2. 为恢复计划指定一个名称，并指定源和目标 VMM 服务器。源服务器必须具有启用了故障转移和恢复的虚拟机。选择“Hyper-V”仅查看针对 Hyper-V 复制进行了配置的云。

	![创建恢复计划](./media/site-recovery-vmm-to-vmm/recovery-plan1.png)

3. 在“选择虚拟机”中，选择复制组。将选择与复制组关联的所有虚拟机，并将其添加到恢复计划。这些虚拟机将添加到恢复计划的默认组—“组 1”中。如果需要，你可以添加更多的组。请注意，在复制后，各个虚拟机将根据恢复计划组的顺序启动。

	![添加虚拟机](./media/site-recovery-vmm-to-vmm/recovery-plan2.png)

在创建恢复计划后，它将出现在“恢复计划”选项卡上的列表中。

###运行测试故障转移

1. 在“恢复计划”选项卡上，选择该计划并单击“测试故障转移”。
2. 在“确认测试故障转移”页面上，选择“无”。请注意，当启用了此选项时，故障转移后的副本虚拟机不会连接到任何网络。这将测试虚拟机是否按预期进行故障转移，但是不会测试你的复制网络环境。有关如何使用不同网络选项的详细信息，请查看[如何运行测试性故障转移](/documentation/articles/site-recovery-failover/#run-a-test-failover)。
3. 将在副本虚拟机所在的同一主机上创建测试虚拟机。它将被添加到副本虚拟机所在的同一个云中。

### 运行恢复计划
在复制之后，副本虚拟机的 IP 地址可能不同于主虚拟机的 IP 地址。虚拟机在启动后将更新它们使用的 DNS 服务器。你也可以添加一个脚本来更新 DNS 服务器，以确保及时更新。

#### 用于检索 IP 地址的脚本
运行此示例脚本来检索 IP 地址。

    	$vm = Get-SCVirtualMachine -Name <VM_NAME>
		$na = $vm[0].VirtualNetworkAdapters>
		$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
		$ip.address  

#### 用于更新 DNS 的脚本
运行此示例脚本来更新 DNS 并指定你通过前一个示例脚本检索到的 IP 地址。

		string]$Zone,
		[string]$name,
		[string]$IP
		)
		$Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
		$newrecord = $record.clone()
		$newrecord.RecordData[0].IPv4Address  =  $IP
		Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord

## 后续步骤

运行测试性故障转移以确保环境功能正常以后，请[了解](/documentation/articles/site-recovery-failover/)不同类型的故障转移。


<a name="privacy"></a><h2>站点恢复的隐私信息</h2>

本部分提供了 Azure Site Recovery 服务（“服务”）的更多隐私信息。若要查看 Azure 服务的隐私声明，请参阅 [Azure 隐私声明](/support/legal/privacy-statement/)

**功能：注册**

- **作用**：向服务注册服务器以便可以保护虚拟机
- **收集的信息**：在注册后，服务将从指定的 VMM 服务器收集、处理和传输管理证书信息以使用 VMM 服务器的服务名称以及你的 VMM 服务器上的虚拟机云的名称提供灾难恢复。
- **信息的使用**：
	- 管理证书—这用于帮助识别已注册的 VMM 服务器和对其进行身份验证以便访问“服务”。“服务”使用证书的公钥部分来保护只有已注册的 VMM 服务器可以访问的一个令牌。服务器需要使用该令牌来获取对“服务”功能的访问权限。
	- VMM 服务器的名称—VMM 服务器名称是进行识别以及与云所在的相应 VMM 服务器进行通信所必需的。
	- VMM 服务器中的云名称—当使用下面所述的“服务”云配对/取消配对功能时，云名称是必需的。当决定将主数据中心内的云与恢复数据中心内的另一个云进行配对时，需要提供恢复数据中心内所有云的名称。

- **选择**：该信息是服务注册过程中必不可少的部分，因为它可以帮助你和服务识别你要为其提供 Azure Site Recovery 保护的 VMM 服务器，并且可以帮助识别正确的已注册 VMM 服务器。如果不希望向“服务”发送该信息，请不要使用本“服务”。如果你注册了你的服务器，以后希望取消注册它，可以通过从“服务”门户（即 Azure 门户）中删除 VMM 服务器信息来完成此操作。

**功能：启用 Azure Site Recovery 保护**

- **作用**：VMM 服务器上安装的 Azure Site Recovery 提供程序是用于与“服务”进行通信的通道。该提供程序是 VMM 进程中承载的一个动态链接库 (DLL)。在安装该提供程序后，会在 VMM 管理员控制台中启用“数据中心恢复”功能。云中任何新的或现有的虚拟机都可以启用一个名为“数据中心恢复”的属性来帮助保护虚拟机。在设置该属性后，该提供程序会将虚拟机的名称和 ID 发送到“服务”。虚拟保护是通过 Windows Server 2012 或 Windows Server 2012 R2 Hyper-V 复制技术启用。虚拟机数据将从一台 Hyper-V 主机复制到另一台主机（通常位于一个不同的“恢复”数据中心内）。

- **收集的信息**：服务将收集、处理和传输虚拟机的元数据，这包括虚拟机的名称、ID、虚拟网络以及它所属的云的名称。

- **信息的使用**：服务使用上面的信息来填充你的服务门户中的虚拟机信息。

- **选择**：这是服务必不可少的组成部分，无法关闭。如果不希望向“服务”发送该信息，请不要为任何虚拟机启用 Azure Site Recovery 保护。请注意，该提供程序发送到“服务”的所有数据都是通过 HTTPS 发送的。

**功能：恢复计划**

- **作用**：此功能可帮助你为“恢复”数据中心构建业务流程计划。你可以定义多个虚拟机或一组虚拟机在恢复站点上启动时应当遵循的顺序。你还可以指定在恢复每个虚拟机时要运行的任何自动脚本，或者要采取的任何手动操作。通常会在恢复计划级别触发故障转移（将在下一部分中介绍）以实现协调的恢复。

- **收集的信息**：服务将收集、处理和传输恢复计划的元数据，包括虚拟机元数据以及任何自动脚本和手动操作说明的元数据。

- **信息的使用**：上面所述的元数据用来在你的服务门户中构建恢复计划。

- **选择**：这是服务必不可少的组成部分，无法关闭。如果不希望向“服务”发送该信息，请不要在本“服务”中构建恢复计划。

**功能：网络映射**

- **作用**：此功能允许你将主数据中心内的网络信息映射到恢复数据中心。当在恢复站点上恢复虚拟机时，此映射可帮助它们建立网络连接。

- **收集的信息**：作为网络映射功能的一部分，服务将收集、处理和传输每个站点（主站点和数据中心）的逻辑网络的元数据。

- **信息的使用**：服务使用该元数据来填充你的服务门户，你可以在该门户中映射网络信息。

- **选择**：这是服务必不可少的组成部分，无法关闭。如果不希望向“服务”发送该信息，请不要使用网络映射功能。

**功能：故障转移 - 计划内的、计划外、测试**

- **作用**：此功能帮助执行从一个 VMM 托管数据中心到另一个 VMM 托管数据中心的虚拟机故障转移。故障转移操作是由用户在其“服务”门户上触发的。发生故障转移的可能原因包括计划外事件（例如发生自然灾害）；计划内事件（例如数据中心负载平衡）；测试故障转移（例如恢复计划演练）。

VMM 服务器上的提供程序将从“服务”那里收到事件通知，并通过 VMM 接口在 Hyper-V 主机上执行故障转移操作。从一台 Hyper-V 主机到另一台主机（通常位于一个不同的“恢复”数据中心内）执行的虚拟机的实际故障转移是通过 Windows Server 2012 或 Windows Server 2012 R2 Hyper-V 复制技术处理的。在故障转移完成后，“恢复”数据中心内的 VMM 服务器上安装的提供程序会向“服务”发送成功信息。

- **收集的信息**：服务使用上面的信息来填充你的服务门户中故障转移操作信息的状态。

- **信息的使用**：服务按如下所述使用上面的信息：

	- 管理证书—这用于帮助识别已注册的 VMM 服务器和对其进行身份验证以便访问“服务”。“服务”使用证书的公钥部分来保护只有已注册的 VMM 服务器可以访问的一个令牌。服务器需要使用该令牌来获取对“服务”功能的访问权限。
	- VMM 服务器的名称—VMM 服务器名称是进行识别以及与云所在的相应 VMM 服务器进行通信所必需的。
	- VMM 服务器中的云名称—当使用下面所述的“服务”云配对/取消配对功能时，云名称是必需的。当决定将主数据中心内的云与恢复数据中心内的另一个云进行配对时，需要提供恢复数据中心内所有云的名称。

- **选择**：这是服务必不可少的组成部分，无法关闭。如果不希望向“服务”发送该信息，请不要使用本“服务”。

<!---HONumber=Mooncake_0328_2016-->