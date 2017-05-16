<properties
    pageTitle="Azure 备份服务器故障排除 | Azure"
    description="排查 Azure 备份服务器的安装和注册以及应用程序工作负荷的备份和还原中的问题"
    services="backup"
    documentationcenter=""
    author="pvrk"
    manager="shreeshd"
    editor="" />
<tags
    ms.assetid="2d73c349-0fc8-4ca8-afd8-8c9029cb8524"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="3/24/2017"
    ms.author="pullabhk;markgal;"
    wacn.date="05/15/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="f420e9d6cb8322fd5b51b33cc4cfe8a6e6d9d7e6"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="troubleshoot-azure-backup-server"></a>对 Azure 备份服务器进行故障排除

可参考下表中所列的信息，排查使用 Azure 备份服务器时遇到的错误。

## <a name="registration-and-agent-related-issues"></a>注册和代理相关问题
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 注册到保管库 | 提供的保管库凭据无效。 该文件已损坏，或者没有与恢复服务关联的最新凭据 | 若要修复此错误，请执行以下操作： <ol><li> 从保管库中下载最新的凭据文件，然后重试 <br>（或者）</li> <li> 如果上述操作不起作用，请尝试将凭据下载到其他本地目录，或创建新的保管库 <br>（或者）</li> <li> 根据[此博客](https://azure.microsoft.com/zh-cn/blog/troubleshooting-common-configuration-issues-with-azure-backup/)中的说明，尝试更新日期和时间设置 <br>（或者）</li> <li> 查看 C:\windows\temp 中的文件数是否超过 65000。 将过时文件移到另一位置，或者删除 Temp 文件夹中的项 <br>（或者）</li> <li> 检查证书的状态 <br> a. 打开“管理计算机证书”（位于控制面板中） <br> b.保留“数据库类型”设置，即设置为“共享”。 展开“个人”节点及其子节点“证书” <br> c.  删除证书“Azure Tools” <br> d. 重试在 Azure 备份客户端中注册 <br> （或者） </li> <li> 检查组策略是否已到位 </li></ol> |
| 将代理推送到受保护的服务器 | 代理操作失败，因为 \<ServerName> 上的 DPM 代理协调器服务出现通信错误 | **如果产品中显示的建议操作不起作用**： <ol><li> 如果是从不可信域附加计算机，请执行[这些](https://technet.microsoft.com/zh-cn/library/hh757801(v=sc.12).aspx)步骤 <br> （或者） </li><li> 如果是从可信域附加计算机，则可通过[此博客](https://blogs.technet.microsoft.com/dpm/2012/02/06/data-protection-manager-agent-network-troubleshooting/)中概述的步骤进行故障排除 <br>（或者）</li><li> 尝试将禁用防病毒作为故障排除步骤。 如果解决了问题，请根据[此文](https://technet.microsoft.com/zh-cn/library/hh757911.aspx)中的建议修改防病毒设置</li></ol> |
| 将代理推送到受保护的服务器 | 为服务器指定的凭据无效 | **如果产品中显示的建议操作不起作用**： <br> 尝试根据[此文](https://technet.microsoft.com/zh-cn/library/hh758186(v=sc.12).aspx#BKMK_Manual)中的说明，在生产服务器上手动安装保护代理|


## <a name="configuring-protection-group"></a>配置保护组
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 配置保护组 | DPM 无法枚举受保护计算机（受保护计算机名称）上的应用程序组件 | 在相关数据源/组件级别的“配置保护组”UI 屏幕上单击“刷新” |
| 配置保护组 | 无法配置保护 | 如果受保护的服务器是 SQL Server，则请根据[此文](https://technet.microsoft.com/zh-cn/library/hh757977(v=sc.12).aspx)中的说明，检查是否已在受保护计算机上将 sysadmin 角色权限提供给系统帐户 (NTAuthority\System)
| 配置保护组 | 此保护组的存储池中没有足够的可用空间 | 添加到存储池的磁盘[不应包含分区](https://technet.microsoft.com/zh-cn/library/hh758075(v=sc.12).aspx)。 删除磁盘上的任何现有卷，然后将其添加到存储池|

## <a name="backup"></a>备份
| 操作 | 错误详细信息 | 解决方法 |
| --- | --- | --- |
| 备份 | 副本不一致 | [此处](https://technet.microsoft.com/zh-cn/library/cc161593.aspx)提供有关副本不一致原因的更多详细信息以及相关建议 <br> <ol><li> 进行系统状态/BMR 备份时，请检查是否已在受保护服务器上安装 Windows Server 备份 </li><li> 检查 DPM/MABS 服务器上的 DPM 存储池是否存在空间相关问题，并根据需要分配存储 </li><li> 检查受保护服务器上卷影复制服务的状态。 如果该服务处于禁用状态，则将其设置为手动启动，然后在服务器上启动该服务。 然后返回到 DPM/MABS 控制台，开始通过一致性检查作业进行同步。</li></ol>|
| 备份 | 运行作业时发生意外错误，设备未就绪 | **如果产品中显示的建议操作不起作用**： <br> <ol><li>将卷影副本存储空间设置为不对保护组中的项进行限制，然后运行一致性检查 <br></li> （或者） <li>尝试删除现有的保护组，然后创建多个新组 - 每个新组中都有一个项</li></ol> |
| 备份 | 如果只备份系统状态，请验证受保护计算机上是否有足够的可用空间来存储系统状态备份 | <ol><li>验证是否已在受保护计算机上安装 WSB</li><li>验证受保护计算机上是否存在用于系统状态的足够空间：若要执行此操作，最简单的方式是转到受保护的计算机，打开 WSB 并单击相关选项，选中 BMR。 然后，UI 就会告知你需要多少空间。 打开 WSB ->“本地备份”->“备份计划”->“选择备份配置”->“完整服务器”（大小已显示）。 使用该大小进行验证。</li></ol>
| 备份 | 在线恢复点创建失败 | 如果错误消息指出“Azure 备份代理无法创建所选卷的快照”，请尝试增加副本和恢复点卷中的空间。
| 备份 | 在线恢复点创建失败 | 如果错误消息指出“未设置此服务器的加密通行短语， 请配置加密通行短语”，则请尝试配置加密通行短语。 如果该操作失败， <br> <ol><li>请检查是否存在暂存位置。 应存在注册表项 HKEY_LOCAL_MACHINE\Software\Microsoft\Azure Backup\Config 中提到的名为“ScratchLocation”的位置。</li><li> 如果暂存位置存在，请尝试使用旧的通行短语重新注册。 **配置加密通行短语时，请将其保存在安全的位置**</li><ol>
| 备份 | BMR 备份失败 | 如果 BMR 非常大，则可将某些应用程序文件移到 OS 驱动器，然后重试 |
| 备份 | 访问文件/共享文件夹时出错 | 尝试根据[此处](https://technet.microsoft.com/zh-cn/library/hh757911.aspx)提供的建议修改防病毒设置|

