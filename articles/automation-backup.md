<properties 
   pageTitle="备份 Azure 自动化"
   description="介绍如何备份自动化帐户的内容，以便在删除自动化帐户后可以保留这些内容。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.date="07/22/2015"
   wacn.date="09/15/2015" />

# 备份 Azure 自动化

当你删除 Windows Azure 中的某个自动化帐户时，该帐户中的所有对象都将被删除，包括 Runbook、模块、设置、作业和资产。在删除帐户后，这些对象不可恢复。在删除自动化帐户之前，你可以参考以下信息来备份该帐户的内容。

## Runbook

可以使用 Azure 管理门户或 Windows PowerShell 中的 [Get-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/zh-CN/library/dn690269.aspx) cmdlet 将 Runbook 导出到脚本文件。可以根据[创建或导入 Runbook](https://msdn.microsoft.com/zh-CN/library/dn643637.aspx) 中所述，将这些脚本文件导入另一个自动化帐户。


## 集成模块

你无法从 Azure 自动化导出集成模块。必须确保这些模块可在自动化帐户外部使用。

## 资产

你无法从 Azure 自动化导出[资产](https://msdn.microsoft.com/zh-CN/library/dn939988.aspx)。使用 Azure 管理门户时，必须记下变量、凭据、证书、连接和计划的详细信息。然后，必须手动创建你导入到另一个自动化中的 Runbook 使用的任何资产。

你可以使用 [Azure cmdlet](https://msdn.microsoft.com/zh-CN/library/dn690262.aspx) 来检索未加密资产的详细信息，然后保存这些资产供将来参考，或者在另一个自动化帐户中创建等效的资产。

无法使用 cmdlet 检索已加密变量或凭据密码字段的值。如果你不知道这些值，可以使用 [Get-AutomationVariable](https://msdn.microsoft.com/zh-CN/library/dn940012.aspx) 和 [Get-AutomationPSCredential](https://msdn.microsoft.com/zh-CN/library/dn940015.aspx) 活动从 Runbook 中检索这些值。

你无法从 Azure 自动化导出证书。必须确保所有证书在 Azure 外部可用。

## 相关文章

- [创建或导入 Runbook](https://msdn.microsoft.com/zh-CN/library/dn643637.aspx)
- [自动化资产](https://msdn.microsoft.com/zh-CN/library/dn939988.aspx)
- [Azure cmdlet](https://msdn.microsoft.com/zh-CN/library/dn690262.aspx)

<!---HONumber=69-->