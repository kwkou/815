<properties
    pageTitle="使用 Azure 备份代理备份文件和文件夹 | Microsoft 文档"
    description="使用 Azure 备份代理将 Windows 文件和文件夹备份到 Azure。 创建恢复服务保管库，安装备份代理，定义备份策略，以及对文件和文件夹运行初始备份。"
    services="backup"
    documentationcenter=""
    author="markgalioto"
    manager="carmonm"
    editor=""
    keywords="备份保管库; 备份 Windows 服务器; 备份 windows;"
    translationtype="Human Translation" />
<tags
    ms.assetid="7f5b1943-b3c1-4ddb-8fb7-3560533c68d5"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="2/23/2017"
    wacn.date="04/24/2017"
    ms.author="markgal;trinadhk;"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="31d14c08675a4fcf4bef04d49f4a4f4b4aec5819"
    ms.lasthandoff="04/14/2017" />

# <a name="back-up-a-windows-server-or-client-to-azure-using-the-resource-manager-deployment-model"></a>通过资源管理器部署模型将 Windows Server 或客户端备份到 Azure
> [AZURE.SELECTOR]
- [经典管理门户](/documentation/articles/backup-configure-vault-classic/)
- [资源管理器模型](/documentation/articles/backup-configure-vault/)

本文介绍如何通过资源管理器部署模型使用 Azure 备份将 Windows Server（或 Windows 客户端）文件和文件夹备份到 Azure。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/backup-deployment-models.md)]

![备份过程的步骤](./media/backup-configure-vault/initial-backup-process.png)

## <a name="before-you-start"></a>开始之前
若要将服务器或客户端备份到 Azure，你需要一个 Azure 帐户。 如果没有帐户，只需几分钟的时间就能创建一个 [试用帐户](/pricing/1rmb-trial/) 。

## <a name="create-a-recovery-services-vault"></a>创建恢复服务保管库
恢复服务保管库是存储所有按时间创建的备份和恢复点的实体。 恢复服务保管库还包含应用到受保护文件和文件夹的备份策略。 创建恢复服务保管库时，也应选择适当的存储冗余选项。

### <a name="to-create-a-recovery-services-vault"></a>创建恢复服务保管库
以下步骤将引导用户创建恢复服务保管库。 恢复服务保管库不同于备份保管库。

1. 使用以下命令通过订阅凭据登录。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 如果是首次使用 Azure 备份，则必须使用 **[Register-AzureRMResourceProvider](https://msdn.microsoft.com/zh-cn/library/mt679020.aspx)** cmdlet 将 Azure 恢复服务提供程序注册到订阅。

        PS C:\> Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

3. 恢复服务保管库是一种资源管理器资源，因此需要将它放在资源组中。 你可以使用现有的资源组，也可以使用 **[New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt678985.aspx)** cmdlet 创建新的资源组。 创建新的资源组时，请指定资源组的名称和位置。  

        PS C:\> New-AzureRmResourceGroup -Name "test-rg" -Location "China North"

4. 使用 **[New-AzureRmRecoveryServicesVault](https://msdn.microsoft.com/zh-cn/library/mt643910.aspx)** cmdlet 创建新的保管库。 确保为保管库指定的位置与用于资源组的位置是相同的。

        PS C:\> New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "China North"

5. 指定要使用的存储冗余类型；你可以使用[本地冗余存储 (LRS)](/documentation/articles/storage-redundancy/#locally-redundant-storage/) 或[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage/)。 以下示例显示，testVault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。

        PS C:\> $vault1 = Get-AzureRmRecoveryServicesVault -Name "testVault"
        PS C:\> Set-AzureRmRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant

   > [AZURE.TIP]
   > 许多 Azure 备份 cmdlet 要求使用恢复服务保管库对象作为输入。 因此，在变量中存储备份恢复服务保管库对象可提供方便。
   >
   >

## <a name="step-2-download-the-vault-credential-file"></a>步骤 2：下载保管库凭据文件
单击[此处](https://go.microsoft.com/fwLink/?LinkID=288905&clcid=0x0409)即可下载 Windows Server 或 Windows 客户端代理。

可以使用下面的 powershell 脚本下载保管库凭据文件：

    $vault1 = Get-AzureRmRecoveryServicesVault -Name “testVault”
    $credspath = "C:\downloads"
    $credsfilename = Get-AzureRmRecoveryServicesVaultSettingsFile -Backup -Vault $vault1 -Path  $credspath

## <a name="step-3-install-and-register-the-agent"></a>步骤 3 - 安装并注册代理
1. 在 Downloads 文件夹（或其他保存位置）中找到并双击 **MARSagentinstaller.exe** 。
2. 完成 Azure 恢复服务代理安装向导。 若要完成该向导，需完成以下操作：

   - 选择安装和缓存文件夹的位置。
   - 如果使用代理服务器来连接 Internet，请提供代理服务器信息。
   - 如果使用经过身份验证的代理，请提供用户名和密码详细信息。
   - 提供已下载的保管库凭据
   - 将加密通行短语保存在安全的位置。

     > [AZURE.NOTE]
     > 如果你丢失或忘记了通行短语，Microsoft 无法帮助你恢复备份数据。 请将文件保存在安全的位置。 还原备份时需要用到此文件。
     >
     >

现已安装代理，且已向保管库注册计算机。 接下来可以配置和计划备份。


## <a name="step-4-complete-the-initial-backup"></a>步骤 4：完成初始备份
初始备份包括两个关键任务：

- 计划备份
- 首次备份文件和文件夹

若要完成初始备份，请使用 Azure 备份代理。

### <a name="to-schedule-the-backup"></a>计划备份
1. 打开 Azure 备份代理。 可以通过在计算机中搜索“Azure 备份”找到该代理。

    ![启动 Azure 备份代理](./media/backup-configure-vault/snap-in-search.png)
2. 在“备份代理”中，单击“**计划备份**”。

    ![计划 Windows Server 备份](./media/backup-configure-vault/schedule-first-backup.png)
3. 在计划备份向导的“开始使用”页上，单击“**下一步**”。
4. 在“选择要备份的项”页上，单击“**添加项**”。
5. 选择要备份的文件和文件夹，然后单击“**确定**”。
6. 单击“下一步”。
7. 在“**指定备份计划**”页上指定**备份计划**，然后单击“**下一步**”。

    可以计划每日（频率为一天最多三次）或每周备份。

    ![Windows Server 备份项](./media/backup-configure-vault/specify-backup-schedule-close.png)

	> [AZURE.NOTE]
	> 有关如何指定备份计划的详细信息，请参阅 [使用 Azure 备份来取代磁带基础结构](/documentation/articles/backup-azure-backup-cloud-as-tape/)一文。
	>
	>
8. 在“**选择保留策略**”页上，为备份复制选择“**保留策略**”。

    保留策略指定备份可以存储的时间长短。 你可以根据备份的创建时间指定不同的保留策略，而不只是为所有备份点指定一个“通用的策略”。 你可以根据需要修改每日、每周、每月和每年保留策略。
9. 在“选择初始备份类型”页上，选择初始备份类型。 将“**自动通过网络**”选项保持选中状态，然后单击“**下一步**”。

    你可以通过网络自动备份，或者脱机备份。 本文的余下部分将介绍自动备份过程。 如果你想要执行脱机备份，请查看 [Azure 备份中的脱机备份工作流](/documentation/articles/backup-azure-backup-import-export/) 以了解更多信息。
10. 在“确认”页上复查信息，然后单击“**完成**”。
11. 在向导完成创建备份计划后，请单击“**关闭**”。

### <a name="enable-network-throttling"></a>启用网络限制
备份代理提供网络限制。 限制功能将控制数据传输期间的网络带宽使用方式。 如果需要在工作时间内备份数据，但不希望备份程序干扰其他 Internet 流量，此控制机制很有帮助。 限制适用于备份和还原活动。

> [AZURE.NOTE]
> 网络限制在 Windows Server 2008 R2 SP1、Windows Server 2008 SP2 或 Windows 7（带 Service Pack）上不可用。 Azure 备份网络限制功能需要在本地操作系统上使用服务质量 (QoS)。 虽然 Azure 备份可以保护这些操作系统，但这些平台上的可用 QoS 版本不兼容 Azure 备份网络限制。 网络限制可用于所有其他[支持的操作系统](/documentation/articles/backup-azure-backup-faq/)。
>
>

**启用网络限制**

1. 在“备份代理”中，单击“更改属性”。

    ![更改属性](./media/backup-configure-vault/change-properties.png)
2. 在“**限制**”选项卡上，选中“**为备份操作启用 Internet 带宽使用限制**”复选框。

    ![网络限制](./media/backup-configure-vault/throttling-dialog.png)
3. 启用限制后，指定在“**工作时间**”和“**非工作时间**”允许使用多少带宽进行备份数据传输。

    带宽值从每秒 512 千字节 (Kbps) 开始，最高可为每秒 1,023 兆字节 (MBps)。 你还可以指定“**工作时间**”的开始和结束时间，以及一周中有哪几天被视为工作日。 指定的工作时间以外的时间视为非工作时间。
4. 单击 **“确定”**。

### <a name="to-back-up-files-and-folders-for-the-first-time"></a>首次备份文件和文件夹
1. 在“备份代理”中单击“立即备份”，以通过网络完成初始种子设定。

    ![立即备份 Windows Server](./media/backup-configure-vault/backup-now.png)
2. 在“确认”页上复查“立即备份向导”用于备份计算机的设置。 然后单击“**备份**”。
3. 单击“**关闭**”以关闭向导。 如果你在备份过程完成之前执行此操作，向导将继续在后台运行。

完成初始备份后，备份控制台中将显示“**作业已完成**”状态。

![IR 完成](./media/backup-configure-vault/ircomplete.png)

## <a name="questions"></a>有疑问？
如果你有疑问，或者希望包含某种功能，请 [给我们反馈](http://aka.ms/azurebackup_feedback)。

## <a name="next-steps"></a>后续步骤
有关备份 VM 或其他工作负荷的详细信息，请参阅：

- 备份文件和文件夹后，可以 [管理保管库和服务器](/documentation/articles/backup-azure-manage-windows-server-classic/)。
- 如果需要还原备份，请参阅 [将文件还原到 Windows 计算机](/documentation/articles/backup-azure-restore-windows-server/)一文。

<!--Update_Description: wording update-->