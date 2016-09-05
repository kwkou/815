<!-- need to be customized -->

<properties
	pageTitle="Azure DPM 备份简介 | Azure"
	description="使用 Azure 备份服务备份 DPM 服务器的简介"
	services="backup"
	documentationCenter=""
	authors="Nkolli1"
	manager="shreeshd"
	editor=""
	keywords="System Center Data Protection Manager, Data Protection Manager, dpm 备份"/>

<tags
	ms.service="backup"
	ms.date="08/08/2016"
	wacn.date=""/>

# 使用 DPM 准备将工作负荷备份到 Azure

> [AZURE.SELECTOR]
- [Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup/)
- [SCDPM](/documentation/articles/backup-azure-dpm-introduction/)
- [Azure 备份服务器（经典）](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)
- [SCDPM（经典）](/documentation/articles/backup-azure-dpm-introduction-classic/)

本文介绍如何使用 Azure 备份来保护 System Center Data Protection Manager (DPM) 服务器和工作负荷。通过阅读本文，你将会了解：

- Azure DPM 服务器备份的工作原理
- 实现顺畅备份体验的先决条件
- 遇到的典型错误以及如何处理它们
- 支持的方案

> [AZURE.NOTE] Azure 有两种用于创建和使用资源的部署模型：[Resource Manager 部署模型和经典部署模型](/documentation/articles/resource-manager-deployment-model/)。本文提供有关还原使用 Resource Manager 模型部署的 VM 的信息和过程。

System Center DPM 备份文件和应用程序数据。备份到 DPM 的数据可以存储在磁带、磁盘上，或者使用 Azure Backup 备份到 Azure。DPM 可与 Azure 备份交互，如下所述：

- **部署为物理服务器或本地虚拟机的 DPM** -- 如果 DPM 部署为物理服务器或本地 Hyper-V 虚拟机，则除了磁盘和磁带备份外，还可以将数据备份到恢复服务保管库。
- **部署为 Azure 虚拟机的 DPM** -- 通过 System Center 2012 R2 Update 3，可以将 DPM 部署为 Azure 虚拟机。如果 DPM 部署为 Azure 虚拟机部署，则你可以将数据备份到附加到 DPM Azure 虚拟机的 Azure 磁盘，也可以通过将数据备份到恢复服务保管库来卸载数据存储。

## 为什么从 DPM 备份到 Azure？

使用 Azure 备份来备份 DPM 服务器所带来的业务好处包括：

- 对于本地 DPM 部署，你可以使用 Azure 来取代长期的磁带部署。
- 对于 Azure 中的 DPM 部署，Azure 备份可让你卸载 Azure 磁盘中的存储，从而通过将较旧的数据存储在恢复服务保管库中，并将较新的数据存储在磁盘上来实现扩展。

## 先决条件
按如下所述让 Azure 备份做好备份 DPM 数据的准备：

1. **创建恢复服务保管库** -- 在 Azure 门户中创建保管库。
2. **下载保管库凭据** -- 下载可用于将 DPM 服务器注册到恢复服务保管库的凭据。
3. **安装 Azure 备份代理** -- 将代理从 Azure 备份安装到每个 DPM 服务器上。
4. **注册服务器** -- 将 DPM 服务器注册到恢复服务保管库。

### 1\.创建恢复服务保管库
若要创建恢复服务保管库，请执行以下操作：

1. 登录到 [Azure 门户](https://portal.azure.cn/)。

2. 在“中心”菜单中，单击“浏览”，然后在资源列表中，键入“恢复服务”。当你开始键入时，会根据你的输入筛选该列表。单击“恢复服务保管库”。

    ![创建恢复服务保管库步骤 1](./media/backup-azure-dpm-introduction/open-recovery-services-vault.png)

    此时将显示恢复服务保管库列表。

3. 在“恢复服务保管库”菜单中，单击“添加”。

    ![创建恢复服务保管库步骤 2](./media/backup-azure-dpm-introduction/rs-vault-menu.png)

    此时将打开恢复服务保管库边栏选项卡，其中会提示你提供“名称”、“订阅”、“资源组”和“位置”。

    ![创建恢复服务保管库步骤 5](./media/backup-azure-dpm-introduction/rs-vault-attributes.png)

4. 对于“名称”，请输入一个友好名称来标识保管库。名称对于 Azure 订阅需要是唯一的。键入包含 2 到 50 个字符的名称。名称必须以字母开头，只能包含字母、数字和连字符。

5. 单击“订阅”查看可用订阅列表。如果不确定要使用哪个订阅，请使用默认的（或建议的）订阅。仅当组织帐户与多个 Azure 订阅关联时，才会有多个选项。

6. 单击“资源组”查看可用资源组列表，或单击“新建”以创建新资源组。有关资源组的完整信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)

7. 单击“位置”，为保管库选择地理区域。

8. 单击“创建”。创建恢复服务保管库可能需要一段时间。可以在门户右上区域监视状态通知。创建保管库后，它将在门户中打开。

### 设置存储复制

存储复制选项可让你在异地冗余存储与本地冗余存储之间进行选择。默认情况下，保管库具有异地冗余存储。如果这是你的主要备份，请将选项保持设置为异地冗余存储。如果你想要一个更便宜、但持久性不太高的选项，请选择本地冗余存储。请在 [Azure 存储空间复制概述](/documentation/articles/storage-redundancy/)中深入了解[异地冗余](/documentation/articles/storage-redundancy/#geo-redundant-storage)和[本地冗余](/documentation/articles/storage-redundancy/#locally-redundant-storage)存储选项。

若要编辑存储复制设置，请执行以下操作：

1. 选择你的保管库以打开保管库仪表板和“设置”边栏选项卡。如果“设置”边栏选项卡未打开，请在保管库仪表板中单击“所有设置”。

2. 在“设置”边栏选项卡中，单击“备份基础结构”>“备份配置”，以打开“备份配置”边栏选项卡。在“备份配置”边栏选项卡中，选择保管库的存储复制选项。

    ![备份保管库列表](./media/backup-azure-vms-first-look-arm/choose-storage-configuration-rs-vault.png)  


    选择好保管库的存储选项后，可以开始将 VM 与保管库相关联。若要开始关联，请发现及注册 Azure 虚拟机。


### 2\.下载保管库凭据

保管库凭据文件是门户为每个备份保管库生成的证书。然后，门户会将公钥上载到访问控制服务 (ACS)。证书的私钥将提供给用户作为工作流的一部分，它被指定为计算机注册工作流的输入。当计算机向 Azure 备份服务中标识的保管库发送备份数据时，该证书将对计算机进行身份验证。

保管库凭据仅在注册工作流的过程中使用。用户需负责确保保管库凭据文件不会泄露。如果该文件落入恶意用户之手，他们可以使用保管库凭据文件将其他计算机注册到同一个保管库。但是，但是备份数据已使用属于客户的通行短语加密，因此现有的备份数据不会泄露。为了消除这种忧虑，保管库凭据设置为 48 小时后过期。你可以下载恢复服务的保管库凭据任意次数 - 但是，在运行注册工作流期间，只有最新的保管库凭据文件才适用。

从 Azure 门户通过安全通道下载保管库凭据文件。Azure 备份服务并不知道证书的私钥，并且私钥不会保存在门户或服务中。使用以下步骤将保管库凭据文件下载到本地计算机。

1. 登录到 [Azure 门户](https://portal.azure.cn/)。

2. 打开要将 DPM 计算机注册到的恢复服务保管库。

3. 默认情况下会打开“设置”边栏选项卡。如果设置边栏选项卡已关闭，请在保管库仪表板中单击“设置”将它打开。在“设置”边栏选项卡中单击“属性”。

	![打开保管库边栏选项卡](./media/backup-azure-dpm-introduction/vault-settings-dpm.png)  


4. 在“属性”页上的“备份凭据”下面单击“下载”。门户将生成可供下载的保管库凭据文件。

    ![下载](./media/backup-azure-dpm-introduction/vault-credentials.png)  


门户将使用保管库名称和当前日期的组合生成保管库凭据。单击“保存”将保管库凭据下载到本地帐户的 downloads 文件夹，或者从“保存”菜单中选择“另存为”以指定保管库凭据保存到的位置。生成文件最长需要一分钟时间。

### 注意
- 确保将保管库凭据文件保存在可从计算机访问的位置。如果将它存储在文件共享/SMB 中，请检查访问权限。
- 保管库凭据文件仅在注册工作流中使用。
- 保管库凭据文件在 48 小时后过期，可从门户下载。



### 3\.安装备份代理

在创建 Azure 备份保管库后，应在每个 Windows 计算机（Windows Server、Windows 客户端、System Center Data Protection Manager 服务器或 Azure 备份服务器计算机）上安装代理，以便将数据和应用程序备份到 Azure。

1. 打开要将 DPM 计算机注册到的恢复服务保管库。

2. 默认情况下会打开“设置”边栏选项卡。如果“设置”边栏选项卡已关闭，请单击“设置”将它打开。在“设置”边栏选项卡中单击“属性”。

	![打开保管库边栏选项卡](./media/backup-azure-dpm-introduction/vault-settings-dpm.png)  


3. 在“设置”页上的“Azure 备份代理”下面单击“下载”。

    ![下载](./media/backup-azure-dpm-introduction/azure-backup-agent.png)

   下载代理后，双击 MARSAgentInstaller.exe 以启动 Azure 备份代理的安装。选择代理所需的安装文件夹和临时文件夹。指定的缓存位置必须至少有备份数据的 5% 的可用空间。

4.	如果使用代理服务器连接到 Internet，请在“代理配置”屏幕中，输入代理服务器详细信息。如果使用已经过身份验证的代理，请在此屏幕中输入用户名和密码详细信息。

5.	Azure 备份代理将安装 .NET Framework 4.5 和 Windows PowerShell（如果没有）以完成安装。

6.	安装代理之后，**关闭**窗口。

    ![关闭](../../includes/media/backup-install-agent/dpm_FinishInstallation.png)

7. 若要**注册 DPM 服务器**到保管库，请在“管理”选项卡中，单击“联机”。然后，选择“注册”。此时会打开注册安装程序向导。

8. 如果使用代理服务器连接到 Internet，请在“代理配置”屏幕中，输入代理服务器详细信息。如果使用已经过身份验证的代理，请在此屏幕中输入用户名和密码详细信息。

	![代理配置](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Proxy.png)

9. 在保管库凭据屏幕中，浏览到并选择前面下载的保管库凭据文件。

    ![保管库凭据](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Credentials.jpg)

    保管库凭据文件只能生效 48 小时（从门户下载后算起）。如果此屏幕中显示任何错误（例如“提供的保管库凭据文件已过期”），请登录到 Azure 门户，并再次下载保管库凭据文件。

    确保将保管库凭据文件放置在安装应用程序可访问的位置。如果你遇到访问相关的错误，请将保管库凭据文件复制到此计算机中的临时位置，然后重试操作。

    如果遇到无效的保管库凭据错误（例如“所提供的保管库凭据无效”），则该文件已损坏，或者没有与恢复服务关联的最新凭据。请在从门户下载新的保管库凭据文件后重试该操作。如果用户在 Azure 门户中快速连续单击“下载保管库凭据”选项，则通常会出现此错误。在这种情况下，只有第二个保管库凭据文件有效。

10. 若要控制工作时间和非工作时间网络带宽的使用情况，可在“限制设置”屏幕中设置带宽使用限制并定义工作时间和非工作时间。

    ![限制设置](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Throttling.png)  


11. 在“恢复文件夹设置”屏幕中，浏览到将在其中暂存从 Azure 下载的文件的文件夹。

    ![恢复文件夹设置](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_RecoveryFolder.png)  


12. 在“加密设置”屏幕中，你可以生成一个通行短语，或者提供一个通行短语（最少包含 16 个字符）。请记住将通行短语保存在安全位置。

    ![加密](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Encryption.png)  


    > [AZURE.WARNING] 如果您丢失或忘记了通行短语，Microsoft 无法帮助您恢复备份的数据。加密通行短语由最终用户拥有，Microsoft 看不到最终用户所用的通行短语。请将该文件保存在安全位置，因为在恢复操作期间需要用到它。

13. 单击“注册”按钮后，计算机即会成功注册到保管库。现在，你可以开始备份到 Azure。

14. 在使用 Data Protection Manager 时，可以通过在“管理”选项卡下选择“联机”，单击“配置”选项来修改在注册工作流期间指定的设置。

## 要求和限制

- DPM 可作为物理服务器或安装在 System Center 2012 SP1 或 System Center 2012 R2 上的 Hyper-V 虚拟机运行。它也可以作为 System Center 2012 R2（至少包含 DPM 2012 R2 Update Rollup 3）上运行的 Azure 虚拟机，或 System Center 2012 R2（至少包含 Update Rollup 5）上运行的 VMWare 中的 Windows 虚拟机运行。
- 如果你要运行 System Center 2012 SP1 中的 DPM，则应安装 System Center Data Protection Manager SP1 的 Update Rollup 2。只有满足此要求，你才能安装 Azure 备份代理。
- DPM 服务器上应已安装 Windows PowerShell 和 .NET Framework 4.5。
- DPM 可将大多数工作负载备份到 Azure 备份。有关所支持内容的完整列表，请参阅下面的 Azure 备份支持项。
- 使用“复制到磁带”选项无法恢复存储在 Azure 备份中的数据。
- 你需要一个启用了 Azure 备份功能的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。阅读 [Azure 备份定价](/pricing/details/backup/)的相关信息。
- 若要使用 Azure 备份，应在要备份的服务器上安装 Azure 备份代理。每台服务器上的可用本地存储空间必须至少为要备份的数据大小的 5%。例如，如果要备份 100 GB 的数据，则暂存位置至少需要 5 GB 的可用空间。
- 数据将存储在 Azure 保管库存储中。你可以备份到 Azure 备份保管库的数据量没有限制，但数据源（例如虚拟机或数据库）的大小不应超过 54400 GB。

支持将以下文件类型备份到 Azure：

- 加密（仅限完整备份）
- 压缩（支持增量备份）
- 稀疏（支持增量备份）
- 压缩和稀疏（视为稀疏）

以下内容不受支持：

- 不支持区分大小写的文件系统上的服务器。
- 硬链接（跳过）
- 重分析点（跳过）
- 加密和压缩（跳过）
- 加密和稀疏（跳过）
- 压缩流
- 稀疏流

>[AZURE.NOTE] 从 System Center 2012 DPM SP1 开始，你可以使用 Azure 备份将 DPM 保护的工作负荷备份到 Azure。

<!---HONumber=Mooncake_0829_2016-->