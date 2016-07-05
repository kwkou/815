<properties
pageTitle="如何删除 HDInsight 群集 | Azure"
description="删除 HDInsight 群集的各种方式的相关信息。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/07/2016"
	wacn.date="04/11/2016"/>

#如何删除 HDInsight 群集

HDInsight 群集按小时收费，因此在不再需要使用群集时，就应将其删除。在本文档中，你将学习如何使用 Azure 经典管理门户、Azure PowerShell 和 Azure CLI 删除群集。

> [AZURE.IMPORTANT] 删除 HDInsight 群集不会删除与群集关联的 Azure 存储帐户。这样你就可以保留和重复使用由群集存储的任何数据。

##Azure 经典管理门户

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)，单击“HDInsight”，然后选择要删除的群集。

2. 单击底部的“删除”。出现提示时，选择“是”即可删除该群集。

##Azure PowerShell

在 PowerShell 提示符处，使用以下命令删除群集：

    Remove-AzureHDInsightCluster -Name CLUSTERNAME

将 __CLUSTERNAME__ 替换为 HDInsight 群集的名称。

##Azure CLI

在提示符处，使用以下命令删除群集：

    azure hdinsight cluster delete CLUSTERNAME
    
将 __CLUSTERNAME__ 替换为 HDInsight 群集的名称。

<!---HONumber=Mooncake_0405_2016-->