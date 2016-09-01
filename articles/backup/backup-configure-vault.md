<!-- need to be customized -->

<properties
	pageTitle="通过 Resource Manager 部署模型使用 Azure 备份将 Windows Server 或客户端备份到 Azure | Azure"
	description="通过创建备份保管库、下载凭据、安装备份代理并完成文件和文件夹的初始备份，将 Windows 服务器或客户端备份到 Azure。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""
	keywords="备份保管库; 备份 Windows 服务器; 备份 windows;"/>

<tags
	ms.service="backup"
	ms.date="05/10/2016"
	wacn.date="07/26/2016"/>
	
# 通过 Resource Manager 部署模型使用 Azure 备份将 Windows Server 或客户端备份到 Azure

本文介绍如何通过 Resource Manager 部署模型使用 Azure 备份将 Windows Server（或 Windows 客户端）文件和文件夹备份到 Azure。

![备份过程的步骤](./media/backup-configure-vault/initial-backup-process.png)

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-rm-include.md)] 经典部署模型。

## 开始之前
若要将服务器或客户端备份到 Azure，你需要一个 Azure 帐户。如果你没有帐户，只需几分钟的时间就能创建一个[试用帐户](/pricing/1rmb-trial/)。

## 步骤 1：创建恢复服务保管库

恢复服务保管库是存储所有按时间创建的备份和恢复点的实体。恢复服务保管库还包含应用到受保护文件和文件夹的备份策略。在创建恢复服务保管库时，你还应选择适当的存储冗余选项。

### 创建恢复服务保管库

1. 如果你尚未登录 [Azure 新门户](https://portal.azure.cn/)，请使用你的 Azure 订阅登录。

2. 在“中心”菜单中，单击“浏览”，然后在资源列表中，键入“恢复服务”。当你开始键入时，会根据你的输入筛选该列表。单击“恢复服务保管库”。

    ![创建恢复服务保管库步骤 1](./media/backup-configure-vault/browse-to-rs-vaults.png) <br/>

    此时将显示恢复服务保管库列表。

3. 在“恢复服务保管库”菜单中，单击“添加”。

    ![创建恢复服务保管库步骤 2](./media/backup-configure-vault/rs-vault-menu.png)

    此时将打开恢复服务保管库边栏选项卡，其中会提示你提供“名称”、“订阅”、“资源组”和“位置”。

    ![创建恢复服务保管库步骤 5](./media/backup-configure-vault/rs-vault-attributes.png)

4. 对于“名称”，请输入一个友好名称以标识保管库。名称对于 Azure 订阅需要是唯一的。键入包含 2 到 50 个字符的名称。名称必须以字母开头，只能包含字母、数字和连字符。

5. 单击“订阅”查看可用订阅列表。如果不确定要使用哪个订阅，请使用默认（或建议）的订阅。仅当组织帐户与多个 Azure 订阅关联时，才会有多个选项。

6. 单击“资源组”查看可用资源组列表，或单击“新建”以创建新的资源组。有关资源组的完整信息，请参阅 [Using the Azure Portal to deploy and manage your Azure resources（使用 Azure 门户部署和管理 Azure 资源）](/documentation/articles/resource-group-portal/)

7. 单击“位置”，为保管库选择地理区域。此选项决定了备份数据要发送到的地理区域。选择靠近你所在位置的地理区域可以减少备份到 Azure 时的网络延迟。

8. 单击“创建”。创建恢复服务保管库可能需要一段时间。可以在门户右上区域监视状态通知。创建保管库后，它应在门户中打开。如果在完成后看不到你的保管库列出，请单击“刷新”。在列表刷新时，单击该保管库的名称。

### 确定存储冗余
在首次创建恢复服务保管库时，需确定复制存储的方式。

1. 在“设置”边栏选项卡（它将随保管库仪表板自动打开）中，单击“备份基础结构”。

2. 在“备份基础结构”边栏选项卡中，单击“备份配置”以查看“存储复制类型”。

    ![创建恢复服务保管库步骤 5](./media/backup-configure-vault/backup-infrastructure.png)

3. 为你的保管库选择存储复制选项。

    ![恢复服务保管库列表](./media/backup-configure-vault/choose-storage-configuration.png)

    默认情况下，保管库具有异地冗余存储。如果你使用 Azure 作为主要备份存储终结点，则继续使用异地冗余存储。如果使用 Azure 作为非主要备份存储终结点，则选择本地冗余存储，以减少在 Azure 中存储数据的成本。请在此[概述](/documentation/articles/storage-redundancy/)中深入了解[异地冗余](/documentation/articles/storage-redundancy/#geo-redundant-storage)和[本地冗余](/documentation/articles/storage-redundancy/#locally-redundant-storage)存储选项。

    选择好保管库的存储选项后，可以开始将文件和文件夹与保管库相关联。

既然你已创建保管库，就可以准备基础结构以备份文件和文件夹了，方法是下载并安装 Microsoft Azure 恢复服务代理、下载保管库凭据，然后使用这些凭据向保管库注册该代理。

## 步骤 2 - 下载文件

>[AZURE.NOTE] 通过 Azure 门户启用备份功能即将推出。目前，可以使用本地 Microsoft Azure 恢复服务代理备份文件和文件夹。

1. 单击恢复服务保管库仪表板上的“设置”。

    ![打开备份目标边栏选项卡](./media/backup-configure-vault/settings-button.png)

2. 单击“设置”边栏选项卡上的“入门”>“备份”。

    ![打开备份目标边栏选项卡](./media/backup-configure-vault/getting-started-backup.png)

3. 单击“备份”边栏选项卡上的“备份目标”。

    ![打开备份目标边栏选项卡](./media/backup-configure-vault/backup-goal.png)

4. 从“运行工作负荷的位置?”菜单选择“本地”。

5. 从“要备份的项?”菜单选择“文件和文件夹”，然后单击“确定”。

#### 下载恢复服务代理

1. 在“准备基础结构”边栏选项卡中，单击“下载 Windows Server 或 Windows 客户端的代理”。

    ![准备基础结构](./media/backup-configure-vault/prepare-infrastructure-short.png)

2. 在下载弹出窗口中单击“保存”。默认情况下，**MARSagentinstaller.exe** 文件将保存到 Downloads 文件夹。

#### 下载保管库凭据

1. 在“准备基础结构”边栏选项卡上单击“下载”>“保存”。

    ![准备基础结构](./media/backup-configure-vault/prepare-infrastructure-download.png)

## 步骤3 - 安装并注册代理

1. 在 Downloads 文件夹（或其他保存位置）中找到并双击 **MARSagentinstaller.exe**。

2. 完成 Microsoft Azure 恢复服务代理安装向导。若要完成该向导，你需要：

    - 选择安装和缓存文件夹的位置。
    - 如果使用代理服务器来连接 Internet，请提供代理服务器信息。
    - 如果使用经过身份验证的代理，请提供用户名和密码详细信息。
    - 提供已下载的保管库凭据
    - 将加密通行短语保存在安全的位置。

    >[AZURE.NOTE] 如果你丢失或忘记了通行短语，Azure 无法帮助你恢复备份数据。请将文件保存在安全的位置。还原备份时需要用到此文件。

现已安装代理，且已向保管库注册计算机。接下来可以配置和计划备份。

## 步骤 4：完成初始备份

初始备份包括两个关键任务：

- 计划备份
- 首次备份文件和文件夹

若要完成初始备份，请使用 Microsoft Azure 备份代理。

### 计划备份

1. 打开 Microsoft Azure 备份代理。可以通过在计算机中搜索“Microsoft Azure 备份”找到该代理。

    ![启动 Azure 备份代理](./media/backup-configure-vault/snap-in-search.png)

2. 在“备份代理”中，单击“计划备份”。

    ![计划 Windows Server 备份](./media/backup-configure-vault/schedule-first-backup.png)

3. 在计划备份向导的“开始使用”页上，单击“下一步”。

4. 在“选择要备份的项”页上，单击“添加项”。

5. 选择要备份的文件和文件夹，然后单击“确定”。

6. 单击**“下一步”**。

7. 在“指定备份计划”页上指定**备份计划**，然后单击“下一步”。

    可以计划每日（频率为一天最多三次）或每周备份。

    ![Windows Server 备份项](./media/backup-configure-vault/specify-backup-schedule-close.png)

    >[AZURE.NOTE] 有关如何指定备份计划的详细信息，请参阅 [Use Azure Backup to replace your tape infrastructure（使用 Azure 备份来取代磁带基础结构）](/documentation/articles/backup-azure-backup-cloud-as-tape/) 一文。

8. 在“选择保留策略”页上，为备份副本选择“保留策略”。

    保留策略指定备份可以存储的时间长短。你可以根据备份的创建时间指定不同的保留策略，而不只是为所有备份点指定一个“通用的策略”。你可以根据需要修改每日、每周、每月和每年保留策略。

9. 在“选择初始备份类型”页上，选择初始备份类型。将“自动通过网络”选项保持选中状态，然后单击“下一步”。

   

10. 在“确认”页上复查信息，然后单击“完成”。

11. 在向导完成创建备份计划后，请单击“关闭”。

### <a name="enable-network-throttling"></a>启用网络限制（可选）

备份代理提供网络限制。限制功能将控制数据传输期间的网络带宽使用方式。如果需要在上班时间内备份数据，但不希望备份程序干扰其他 Internet 流量，此控制机制很有帮助。限制适用于备份和还原活动。

**启用网络限制**

1. 在“备份代理”中，单击“更改属性”。

    ![更改属性](./media/backup-configure-vault/change-properties.png)

2. 在“限制”选项卡上，选中“为备份操作启用 Internet 带宽使用限制”复选框。

    ![网络限制](./media/backup-configure-vault/throttling-dialog.png)

3. 启用限制后，指定在“工作时间”和“非工作时间”允许使用多少带宽进行备份数据传输。

    带宽值从每秒 512 千字节 (Kbps) 开始，最高可为每秒 1,023 兆字节 (MBps)。你还可以指定“工作时间”的开始和结束时间，以及一周中有哪几天被视为工作日。指定的工作时间以外的时间视为非工作时间。

4. 单击**“确定”**。

### 首次备份文件和文件夹

1. 在“备份代理”中单击“立即备份”，以通过网络完成初始种子设定。

    ![立即备份 Windows Server](./media/backup-configure-vault/backup-now.png)

2. 在“确认”页上复查“立即备份向导”用于备份计算机的设置。然后单击“备份”。

3. 单击“关闭”以关闭向导。如果你在备份过程完成之前执行此操作，向导将继续在后台运行。

完成初始备份后，备份控制台中将显示“作业已完成”状态。

![IR 完成](./media/backup-configure-vault/ircomplete.png)

## 有疑问？
如果你有疑问，或者希望包含某种功能，请[给我们反馈](http://aka.ms/azurebackup_feedback)。

## 后续步骤
有关备份 VM 或其他工作负荷的详细信息，请参阅：

- 备份文件和文件夹后，可以[管理保管库和服务器](/documentation/articles/backup-azure-manage-windows-server/)。
- 如果需要还原备份，请使用本文[将文件还原到 Windows 计算机](/documentation/articles/backup-azure-restore-windows-server/)。

<!---HONumber=AcomDC_0718_2016-->