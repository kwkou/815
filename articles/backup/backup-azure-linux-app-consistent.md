<properties
    pageTitle="Azure 备份：Linux VM 的应用程序一致性备份 | Azure"
    description="对于 Linux 虚拟机，使用脚本以保证对 Azure 的应用程序一致性备份。 脚本仅适用于资源管理器部署中的 Linux VM；脚本不适用于 Windows VM 或服务管理器部署。 本文将指导完成包括故障排除在内的脚本配置步骤。"
    services="backup"
    documentationcenter="dev-center-name"
    author="anuragmehrotra"
    manager="shivamg"
    keywords="应用程序一致性备份; 应用程序一致性 Azure VM 备份; Linux VM 备份; Azure 备份" />
<tags
    ms.assetid="bbb99cf2-d8c7-4b3d-8b29-eadc0fed3bef"
    ms.service="backup"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage-backup-recovery"
    ms.date="4/12/2017"
    ms.author="anuragm;markgal"
    wacn.date="05/15/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="d82a7da4ace7a5fb0689698ec2a53c9dea4e259c"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="application-consistent-backup-of-azure-linux-vms-preview"></a>Azure Linux VM 的应用程序一致性备份（预览版）

本文介绍 Linux 操作前和操作后脚本框架，以及如何使用该框架来创建 Azure Linux VM 的应用程序一致性备份。

> [AZURE.NOTE]
> 操作前和操作后脚本框架仅受资源管理器部署的 Linux 虚拟机支持。 应用程序一致性脚本不受 Service Manager 部署的虚拟机或 Windows 虚拟机支持。
>

## <a name="how-the-framework-works"></a>框架工作原理

该框架提供一个选项，用于在创建 VM 快照时运行自定义的操作前和操作后脚本。 操作前脚本在创建 VM 快照之前的那一刻执行，操作后脚本在完成 VM 快照后立即执行。 这样，用户便可以在创建 VM 备份时灵活控制其应用程序和环境。

此框架的一个重要用途就是确保创建应用程序一致的 VM 备份。 操作前脚本可以调用应用程序的本机 API 来冻结 IO 并将内存中的内容刷新到磁盘，确保快照的应用程序一致性，即，在还原后启动 VM 时，应用程序可以启动。 可以通过操作后脚本使用应用程序的本机 API 来解冻 IO，使应用程序可以在完成 VM 快照后恢复正常操作。

## <a name="steps-to-configure-pre-script-and-post-script"></a>配置操作前脚本和操作后脚本的步骤

1. 以根用户身份登录到要备份的 Linux VM。

2. 从 [github](https://github.com/MicrosoftAzureBackup/VMSnapshotPluginConfig) 下载 VMSnapshotPluginConfig.json，并将它复制到要备份的所有 VM 上的 /etc/azure 文件夹。 创建 /etc/azure 目录（如果不存在）。

3. 在所有要备份的 VM 上复制应用程序的操作前脚本和操作后脚本。 可将脚本复制到 VM 中的任何位置；需要在 VMSnapshotPluginConfig.json 文件中更新脚本文件的完整路径

4. 确保分配对这些文件的以下权限：

   - VMSnapshotPluginConfig.json - 权限“600”，即，只有“root”用户才对此文件拥有“读取”和“写入”权限，任何用户都不应该拥有其“执行”权限。
   - 操作前脚本文件 - 权限“700”，即，只有“root”用户才对此文件拥有“读取”、“写入”和“执行”权限。
   - 操作后脚本文件 - 权限“700”，即，只有“root”用户才对此文件拥有“读取”、“写入”和“执行”权限。

    > [AZURE.NOTE]
    > 该框架为用户提供大量的强大功能，因此，必须确保它是完全安全，只有“root”用户才能访问关键的 json 和脚本文件。
    > 如果不满足上述要求，脚本将不会执行，导致生成文件系统/崩溃一致性备份。
    >

5. 根据以下详细信息配置 VMSnapshotPluginConfig.json
    - **pluginName** - 将此字段保留原样，否则脚本可能无法按预期工作。
    - **preScriptLocation** - 提供要备份的 VM 上的操作前脚本的完整路径。
    - **postScriptLocation** - 提供要备份的 VM 上的操作后脚本的完整路径。
    - **preScriptParams** - 需要传递给操作前脚本的任何可选参数，所有参数应括在引号中，如果有多个参数，请使用逗号分隔。
    - **postScriptParams** - 需要传递给操作后脚本的任何可选参数，所有参数应括在引号中，如果有多个参数，请使用逗号分隔。
    - **preScriptNoOfRetries** - 发生任何错误时，终止之前应该重试操作前脚本的次数。 0 表示只尝试一次，发生失败时不会重试。
    - **postScriptNoOfRetries** - 发生任何错误时，终止之前应该重试操作后脚本的次数。 0 表示只尝试一次，发生失败时不会重试。
    - **timeoutInSeconds** - 操作前脚本和操作后脚本的单次超时。
    - **continueBackupOnFailure** - 如果希望在操作前脚本或操作后脚本失败时 Azure 备份故障回复到文件系统/崩溃一致性备份，请将此值设置为 true。 如果将此值设置为 false，则脚本失败时，备份也会失败（除非在单磁盘 VM，在这种情况下，备份将故障回复到崩溃一致性备份，不管此项设置如何）。
    - **fsFreezeEnabled** - 指定为了确保文件系统一致性，在创建 VM 快照时是否应调用 Linux fsfreeze。 建议将此值保留为 true，除非必须禁用 fsfreeze 才能让应用程序正常工作。

6. 现已配置脚本框架，如果已配置 VM 备份，则下一次备份将调用这些脚本，并触发应用程序一致的备份。 如果未配置 VM 备份，请参考[将 Azure 虚拟机备份到恢复服务保管库](/documentation/articles/backup-azure-vms-first-look/)配置备份。

## <a name="troubleshooting"></a>故障排除

编写操作前脚本和操作后脚本时，请确保添加相应的日志记录，并查看脚本日志来解决任何脚本问题。 如果执行脚本时仍然遇到问题，请参阅下表中的详细信息。

| 错误 | 错误消息 | 建议的操作 |
| ------------------------ | -------------- | ------------------ |
| Pre-ScriptExecutionFailed |操作前脚本返回错误，因此备份可能不是应用程序一致的。    | 请查看脚本的失败日志来解决问题。|  
|    Post-ScriptExecutionFailed |    操作后脚本返回了可能影响应用程序状态的错误。 |    请查看脚本的失败日志来解决问题，并检查应用程序状态。 |
| Pre-ScriptNotFound |    在 VMSnapshotPluginConfig.json 配置文件中指定的位置找不到操作前脚本。 |    请确保操作前脚本在配置文件中指定的路径处存在，以确保创建应用程序一致的备份。|
| Post-ScriptNotFound |    在 VMSnapshotPluginConfig.json 配置文件中指定的位置找不到操作后脚本 |    请确保操作后脚本在配置文件中指定的路径处存在，以确保创建应用程序一致的备份。|
| IncorrectPluginhostFile |    VmSnapshotLinux 扩展随附的 Pluginhost 文件已损坏，因此操作前脚本和操作后脚本无法执行，将不会创建应用程序一致性的备份。    | 请卸载 VmSnapshotLinux 扩展，下一次备份时会自动重新安装它，这样就可以解决问题。 |
| IncorrectJSONConfigFile | VMSnapshotPluginConfig.json 文件不正确，因此操作前脚本和操作后脚本无法执行，将不会创建应用程序一致性的备份 | 请从 [github](https://github.com/MicrosoftAzureBackup/VMSnapshotPluginConfig) 下载副本并重新配置该文件 |
| InsufficientPermissionforPre-Script | 对于正在执行的脚本，root 用户应是该文件的所有者，并且应该对文件设置“700”权限，即，只有所有者才拥有“读取”、“写入”和“执行”权限 | 确保“root”用户是脚本文件的“所有者”，只有所有者才拥有“读取”、“写入”和“执行”权限。 |
| InsufficientPermissionforPost-Script | 对于正在执行的脚本，root 用户应是该文件的所有者，并且应该对文件设置“700”权限，即，只有所有者才拥有“读取”、“写入”和“执行”权限 | 确保“root”用户是脚本文件的“所有者”，只有所有者才拥有“读取”、“写入”和“执行”权限。 |
| Pre-ScriptTimeout | 执行应用程序一致的备份时操作前脚本超时。 | 请检查脚本，并在 /etc/azure 中的 VMSnapshotPluginConfig.json 文件中增大超时。 |
| Post-ScriptTimeout | 执行应用程序一致的备份时操作后脚本超时。 | 请检查脚本，并在 /etc/azure 中的 VMSnapshotPluginConfig.json 文件中增大超时。 |

## <a name="next-steps"></a>后续步骤
[配置 VM 到恢复服务保管库的备份](/documentation/articles/backup-azure-vms/)

