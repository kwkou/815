<properties 
   pageTitle="Azure Automation 数据保留"
   description="介绍 Azure Automation 的数据保留策略。"
   services="automation"
   documentationCenter=""
   authors="bwren"
   manager="stevenka"
   editor="tysonn" />
<tags 
wacn.date="05/15/2015"
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/13/2015"
   ms.author="bwren" />

# Azure Automation 数据保留

在删除 Azure Automation 中的某个资源时，该资源在被永久删除之前将保留 90 天以供审核。在此期间，你无法查看或使用该资源。此策略也适用于属于已删除的 Automation 帐户的资源。

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

## 相关文章
- [备份 Azure Automation](https://msdn.microsoft.com/zh-CN/library/dn643635.aspx)

<!--HONumber=53-->