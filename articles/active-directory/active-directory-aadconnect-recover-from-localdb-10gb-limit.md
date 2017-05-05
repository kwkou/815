<properties
    pageTitle="Azure AD Connect：如何从 LocalDB 10 GB 的限制问题恢复 | Azure"
    description="本主题介绍在遇到 LocalDB 10 GB 限制问题时，如何恢复 Azure AD Connect Synchronization Service。"
    services="active-directory"
    documentationcenter=""
    author="billmath"
    manager="femila"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="41d081af-ed89-4e17-be34-14f7e80ae358"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/23/2017"
    wacn.date="05/02/2017"
    ms.author="cychua"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="27516be74256b73e61a35f0eb83928d1e57901aa"
    ms.lasthandoff="04/22/2017" />

# <a name="azure-ad-connect-how-to-recover-from-localdb-10-gb-limit"></a>Azure AD Connect：如何从 LocalDB 10 GB 的限制恢复
Azure AD Connect 要求使用 SQL Server 数据库来存储标识数据。 可以使用随 Azure AD Connect 一起安装的默认 SQL Server 2012 Express LocalDB，也可以使用你自己的完整 SQL。 SQL Server Express 存在 10 GB 的大小限制。 使用 LocalDB 并达到此限制后，Azure AD Connect Synchronization Service 将无法正常启动或同步。 本文提供了恢复步骤。

## <a name="symptoms"></a>症状
有两种常见的症状：

- Azure AD Connect Synchronization Service **可以运行**但无法同步，并出现“stopped-database-disk-full”错误。

- Azure AD Connect Synchronization Service **无法启动**。 尝试启动该服务时失败且出现事件 6323 和错误消息“服务器遇到错误，因为 SQL Server 磁盘空间不足”。

## <a name="short-term-recovery-steps"></a>短期恢复步骤
本部分提供的步骤用于回收 DB 空间，该空间是 Azure AD Connect Synchronization Service 恢复运行所必需的。 步骤包括：

1. [确定 Synchronization Service 状态](#determine-the-synchronization-service-status)
2. [收缩数据库](#shrink-the-database)
3. [删除运行历史记录数据](#delete-run-history-data)
4. [缩短运行历史记录数据的保留期](#shorten-retention-period-for-run-history-data)

### <a name="determine-the-synchronization-service-status"></a>确定 Synchronization Service 状态
首先，确定 Synchronization Service 是否仍在运行：

1. 以管理员身份登录到 Azure AD Connect 服务器。

2. 转到“服务控制管理器”。

3. 检查 **Azure AD Sync** 的状态。


4. 请勿停止或重新启动正在运行的服务。 跳过[收缩数据库](#shrink-the-database)步骤，转到[删除运行历史记录数据](#delete-run-history-data)步骤。

5. 如果服务未运行，请尝试启动服务。 如果服务成功启动，请跳过[收缩数据库](#shrink-the-database)步骤，转到[删除运行历史记录数据](#delete-run-history-data)步骤。 否则，请继续执行[收缩数据库](#shrink-the-database)步骤。

### <a name="shrink-the-database"></a>收缩数据库
使用收缩操作可释放足够的 DB 空间，以便启动 Synchronization Service。 该操作释放 DB 空间的方式是删除数据库中的空格。 此步骤只需尽力操作即可，因为无法保证总能恢复空间。 若要详细了解收缩操作，请阅读 [Shrink a database](https://msdn.microsoft.com/zh-cn/library/ms189035.aspx)（收缩数据库）一文。

> [AZURE.IMPORTANT]
> 如果能够运行 Synchronization Service，请跳过此步骤。 建议不要收缩 SQL DB，因为随着碎片增加，可能会导致性能不佳。

为 Azure AD Connect 创建的数据库的名称为 **ADSync**。 若要执行收缩操作，必须以数据库的 sysadmin 或 DBO 身份登录。 在 Azure AD Connect 安装过程中，为以下帐户授予了 sysadmin 权限：
- 本地管理员
- 曾用于运行 Azure AD Connect 安装的用户帐户。
- 用作 Azure AD Connect Synchronization Service 操作上下文的 Sync Service 帐户。
- 安装期间创建的本地组 ADSyncAdmins。

1. 备份数据库，方法是将 `%ProgramFiles%\program files\Azure AD Sync\Data` 下的 **ADSync.mdf** 和 **ADSync_log.ldf** 文件复制到安全位置。

2. 启动新的 PowerShell 会话。

3. 导航到文件夹 `%ProgramFiles%\Program Files\Microsoft SQL Server\110\Tools\Binn`。

4. 启动 **sqlcmd** 实用程序，方法是运行 `./SQLCMD.EXE -S “(localdb)\.\ADSync” -U <Username> -P <Password>` 命令并使用 sysadmin 或数据库 DBO 的凭据。

5. 若要收缩数据库，请在 sqlcmd 提示符 (1>) 处输入 `DBCC Shrinkdatabase(ADSync,1);`，然后在下一行输入 `GO`。

6. 如果操作成功，请尝试再次启动 Synchronization Service。 如果可以启动 Synchronization Service，请转到[删除运行历史记录数据](#delete-run-history-data)步骤。 否则，请联系支持部门。

### <a name="delete-run-history-data"></a>删除运行历史记录数据
默认情况下，Azure AD Connect 最多保留 7 天的运行历史记录数据。 在此步骤中，我们会通过删除运行历史记录数据来回收 DB 空间，这样 Azure AD Connect Synchronization Service 就可以重新开始同步。

1.    转到“开始”→ Synchronization Service，以便启动 **Synchronization Service Manager**。

2.    转到“操作”选项卡。

3.    在“操作”下面，选择“清除运行…”

4.    可以选择“清除所有运行”或“清除 <date>之前的运行…”选项。 建议一开始清除超过两天的运行历史记录数据。 如果仍遇到 DB 大小问题，则选择“清除所有运行”选项。

### <a name="shorten-retention-period-for-run-history-data"></a>缩短运行历史记录数据的保留期
此步骤是为了在多次同步周期后降低遇到 10 GB 限制问题的可能性。

1. 打开新的 PowerShell 会话。

2. 运行 `Get-ADSyncScheduler` 并记下 PurgeRunHistoryInterval 属性，该属性指定当前的保留期。

3. 运行 `Set-ADSyncScheduler -PurgeRunHistoryInterval 2.00:00:00`，将保留期设置为两天。 根据需要调整保留期。

## <a name="long-term-solution---migrate-to-full-sql"></a>长期解决方案 - 迁移到完整的 SQL
通常情况下，此问题表示 10 GB 的数据库大小已经无法让 Azure AD Connect 将本地 Active Directory 同步到 Azure AD。 建议改用完整版 SQL Server。 不能直接将现有 Azure AD Connect 部署的 LocalDB 替换为完整版 SQL 的数据库， 而必须使用完整版 SQL 来部署新的 Azure AD Connect 服务器。 建议执行交叉迁移，将新的 Azure AD Connect 服务器（装有 SQL DB）部署为过渡服务器，与现有的 Azure AD Connect 服务器（装有 LocalDB）并存。 
- 有关如何使用 Azure AD Connect 配置远程 SQL 的说明，请参阅 [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/)一文。
- 有关如何通过交叉迁移进行 Azure AD Connect 升级的说明，请参阅 [Azure AD Connect：从旧版升级到最新版本](/documentation/articles/active-directory-aadconnect-upgrade-previous-version#swing-migration/)一文。

## <a name="next-steps"></a>后续步骤
了解有关 [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

