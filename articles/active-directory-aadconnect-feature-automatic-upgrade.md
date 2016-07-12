<properties
   pageTitle="Azure AD Connect：自动升级 | Azure"
   description="本主题介绍 Azure AD Connect sync 中的内置自动升级功能。"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="StevenPo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="04/15/2016"
   wacn.date="06/03/2016"/>

# Azure AD Connect：自动升级
此功能是随内部版本 1.1.105.0（于 2016 年 2 月发布）一起推出的。

## 概述
使用**自动升级**功能是确保 Azure AD Connect 安装始终保持最新状态的最简单方法。快速安装默认启用此功能，可确保在新版本发布时自动升级安装。

默认情况下，将针对以下方案启用自动升级：

- 快速设置安装
- 使用 SQL Express LocalDB，这是“快速”设置始终使用的选项
- AD 帐户是“快速”设置创建的默认 MSOL\_ 帐户
- Metaverse 中的对象少于 100,000 个

可以使用 PowerShell cmdlet `Get-ADSyncAutoUpgrade` 来查看当前的自动升级状态。状态包括：

| 状态 | 注释 |
| ---- | ---- |
| Enabled | 自动升级已启用。 |
| 已挂起 | 只能由系统设置。系统不再能够接收自动升级。 |
| 已禁用 | 自动升级已禁用。 |

可以使用 `Set-ADSyncAutoUpgrade` 在“已启用”与“已禁用”之间切换。应该只有系统才能设置“暂停”状态。

自动升级使用 Azure AD Connect Health 作为升级基础结构。为使自动升级正常工作，请确保根据 [Office 365 URLs and IP address ranges（Office 365 URL 和 IP 地址范围）](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述，在代理中打开 **Azure AD Connect Health** 的 URL。

如果服务器上正在运行**同步服务管理器** UI，则会暂停升级，直到 UI 关闭为止。

## 故障排除
如果 Connect 安装未按预期自动升级，请遵循以下步骤来找出可能的错误。

首先，不建议在新版本发行的第一天就自动升级。由于升级前有刻意设计的随机性，因此你不用担心安装没有立即升级。

如果你认为有问题，请先运行 `Get-ADSyncAutoUpgrade` 确保已启用自动升级。

启动事件日志，并查看**应用程序**事件日志。针对源 **Azure AD Connect 升级**的事件日志以及事件 ID 范围 **300-399** 添加筛选器。  
![自动升级的事件日志筛选器](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogfilter.png)

这样，便可以获得已与自动升级状态关联的事件日志。  
![自动升级的事件日志筛选器](./media/active-directory-aadconnect-feature-automatic-upgrade/eventlogresult.png)

结果的前半部分提供状态的概述。

| 结果前缀 | 说明 |
| --- | --- |
| 成功 | 安装已成功升级。 |
| UpgradeAborted | 某种临时状态停止了升级。升级将会重试，并且预期会成功。 |
| UpgradeNotSupported | 系统中的某个配置阻止了系统自动升级。升级将会重试，以查看状态是否已变化，但预期只能手动升级系统。 |

下面是最常见的消息列表。该列表并不完整，但结果消息应会明确说明问题所在。

| 结果消息 | 说明 |
| --- | --- |
| **UpgradeAborted** | |
| UpgradeAbortedInsufficientDiskSpace | 没有足够的磁盘空间用于支持升级。 |
| UpgradeAbortedSyncOrConfigurationInProgress | 安装向导正在运行，或者在计划程序外部计划了同步。 |
| **UpgradeNotSupported** | |
| UpgradeNotSupportedCustomizedSyncRules | 你已将自己的自定义规则添加到配置中。 |
| UpgradeNotSupportedMetaverseSizeExceeeded | Metaverse 中的对象超过 100,000 个。 |
| UpgradeNotSupportedMultiForestSetup | 你正在连接到多个林。快速安装只会连接到一个林。 |
| UpgradeNotSupportedNonMsolAccount | [AD 连接器帐户](/documentation/articles/active-directory-aadconnect-accounts-permissions/#active-directory-account)不再是默认的 MSOL\_ 帐户。
| UpgradeNotSupportedStagingModeEnabled | 服务器已设置为[过渡模式](/documentation/articles/active-directory-aadconnectsync-operations/#staging-mode)。 |

## 后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0606_2016-->