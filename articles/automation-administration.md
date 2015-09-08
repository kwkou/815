<properties 
   pageTitle="Azure Automation 管理"
   description="本文包含有关管理 Azure Automation 环境的多个主题。当前包括数据保留期和备份 Azure Automation。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="07/06/2015"
   wacn.date="08/29/2015" />

# Azure Automation 管理

本文包含有关管理 Azure Automation 环境的多个主题。

## 数据保留期

在删除 Azure Automation 中的某个资源时，该资源在被永久删除之前将保留 90 天以供审核。在此期间，您无法查看或使用该资源。此策略也适用于属于已删除的 Automation 帐户的资源。

Azure Automation 会自动删除并永久移除 90 天之前的作业。

下表汇总了各种资源的保留策略。

|数据|策略|
|:---|:---|
|帐户|在帐户被用户删除 90 天后将其永久移除。|
|资产|在资产被用户删除 90 天后或者在包含该资产的帐户被用户删除 90 天后将其永久移除。|
|模块|在模块被用户删除 90 天后或者在包含该模块的帐户被用户删除 90 天后将其永久移除。|
|Runbook|在资源被用户删除 90 天后或者在包含该资源的帐户被用户删除 90 天后将其永久移除。|
|作业|在上次修改 90 天后删除并永久移除。这可能发生在作业已完成、已停止或已暂停之后。|

保留策略应用于所有用户并且当前无法自定义。

## 备份 Azure Automation

当您删除 Windows Azure 中的某个 Automation 帐户时，该帐户中的所有对象都将被删除，包括 Runbook、模块、设置、作业和资产。在删除帐户后，这些对象不可恢复。在删除 Automation 帐户之前，您可以参考以下信息来备份该帐户的内容。

### Runbook

可以使用 Azure 管理门户或 Windows PowerShell 中的 [Get-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/zh-cn/library/dn690269.aspx) cmdlet 将 Runbook 导出到脚本文件。可以根据“[创建或导入 Runbook](https://msdn.microsoft.com/zh-cn/library/dn643637.aspx)”中所述，将这些脚本文件导入另一个 Automation 帐户。


### 集成模块

您无法从 Azure Automation 导出集成模块。必须确保这些模块可在 Automation 帐户外部使用。

### 资产

您无法从 Azure Automation 导出“[资产](https://msdn.microsoft.com/zh-cn/library/dn939988.aspx)”。使用 Azure 管理门户时，必须记下变量、凭据、证书、连接和计划的详细信息。然后，必须手动创建您导入到另一个 Automation 中的 Runbook 使用的任何资产。

您可以使用 [Azure cmdlet](https://msdn.microsoft.com/zh-cn/library/dn690262.aspx) 来检索未加密资产的详细信息，然后保存这些资产供将来参考，或者在另一个 Automation 帐户中创建等效的资产。

无法使用 cmdlet 检索已加密变量或凭据密码字段的值。如果您不知道这些值，可以使用 [Get-AutomationVariable](https://msdn.microsoft.com/zh-cn/library/dn940012.aspx) 和 [Get-AutomationPSCredential](https://msdn.microsoft.com/zh-cn/library/dn940015.aspx) 活动从 Runbook 中检索这些值。

您无法从 Azure Automation 导出证书。必须确保所有证书在 Azure 外部可用。

<!---HONumber=67-->