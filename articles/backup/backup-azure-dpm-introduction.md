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
	wacn.date="09/05/2016"/>

# 使用 DPM 准备将工作负荷备份到 Azure

> [AZURE.SELECTOR]
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

1. **创建恢复服务保管库** -- 使用 Azure PowerShell 创建保管库。
2. **下载保管库凭据** -- 下载可用于将 DPM 服务器注册到恢复服务保管库的凭据。
3. **安装 Azure 备份代理** -- 将代理从 Azure 备份安装到每个 DPM 服务器上。
4. **注册服务器** -- 将 DPM 服务器注册到恢复服务保管库。

### 1\.创建恢复服务保管库

1. 首先使用以下命令行登录你的 Azure 订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

1. 如果你是首次使用 Azure 备份，则必须使用 **[Register-AzureRMResourceProvider](https://msdn.microsoft.com/zh-cn/library/mt603685.aspx)** cmdlet 注册用于订阅的 Azure 恢复服务提供程序。

        Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

2. 恢复服务保管库是一种 Resource Manager 资源，因此需要将它放在资源组中。你可以使用现有的资源组，也可以使用 **[New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt603739.aspx)** cmdlet 创建新的资源组。创建新的资源组时，请指定资源组的名称和位置。

        New-AzureRmResourceGroup –Name "test-rg" –Location "China North"

3. 使用 **[New-AzureRmRecoveryServicesVault](https://msdn.microsoft.com/zh-cn/library/mt643910.aspx)** cmdlet 创建新的保管库。确保为保管库指定的位置与用于资源组的位置是相同的。

        New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "West US"

4. 指定要使用的存储冗余类型；你可以使用[本地冗余存储 (LRS)](/documentation/articles/storage-redundancy/#locally-redundant-storage) 或[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage)。以下示例显示，testVault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。

        $vault1 = Get-AzureRmRecoveryServicesVault –Name "testVault"
        Set-AzureRmRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant

### 2\.下载保管库凭据

使用以下 PowerShell 脚本下载保管库凭据。

    $vault1 = Get-AzureRmRecoveryServicesVault –Name “testVault”
    $credspath = "C:\downloads"
    $credsfilename = Get-AzureRmRecoveryServicesVaultSettingsFile -Backup -Vault $vault1 -Path  $credspath

### 3\.安装备份代理

在创建 Azure 备份保管库后，应在每个 Windows 计算机（Windows Server、Windows 客户端、System Center Data Protection Manager 服务器或 Azure 备份服务器计算机）上安装代理，以便将数据和应用程序备份到 Azure。

1. 点击[这里](https://go.microsoft.com/fwLink/?LinkID=288905&clcid=0x0409)下载 Windows Server 或 Windows 客户端的代理。

4.	如果使用代理服务器连接到 Internet，请在“代理配置”屏幕中，输入代理服务器详细信息。如果使用已经过身份验证的代理，请在此屏幕中输入用户名和密码详细信息。

5.	Azure 备份代理将安装 .NET Framework 4.5 和 Windows PowerShell（如果没有）以完成安装。

6.	安装代理之后，**关闭**窗口。

    ![关闭](../../includes/media/backup-install-agent/dpm_FinishInstallation.png)

7. 若要**注册 DPM 服务器**到保管库，请在“管理”选项卡中，单击“联机”。然后，选择“注册”。此时会打开注册安装程序向导。

8. 如果使用代理服务器连接到 Internet，请在“代理配置”屏幕中，输入代理服务器详细信息。如果使用已经过身份验证的代理，请在此屏幕中输入用户名和密码详细信息。

	![代理配置](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Proxy.png)

9. 在保管库凭据屏幕中，浏览到并选择前面下载的保管库凭据文件。

    ![保管库凭据](../../includes/media/backup-install-agent/DPM_SetupOnlineBackup_Credentials.jpg)

    保管库凭据文件只能生效 48 小时。如果此屏幕中显示任何错误（例如“提供的保管库凭据文件已过期”），请再次下载保管库凭据文件。

    确保将保管库凭据文件放置在安装应用程序可访问的位置。如果你遇到访问相关的错误，请将保管库凭据文件复制到此计算机中的临时位置，然后重试操作。

    如果遇到无效的保管库凭据错误（例如“所提供的保管库凭据无效”），则该文件已损坏，或者没有与恢复服务关联的最新凭据。请在下载新的保管库凭据文件后重试该操作。

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
- 你需要一个启用了 Azure 备份功能的 Azure 帐户。如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。阅读 [Azure 备份定价](/pricing/details/back-up/)的相关信息。
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