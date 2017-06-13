<properties
    pageTitle="如何删除 HDInsight 群集 | Azure"
    description="删除 HDInsight 群集的各种方式的相关信息。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="55f7838b-9786-47ff-96db-1b64437bd0bb"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/03/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.custom="H1Hack27Feb2017,hdinsightactive"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="18bb1699d5a48d5e3146a4c85f3f6f2d2063bf91"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="delete-an-hdinsight-cluster-using-your-browser-powershell-or-the-azure-cli"></a>使用浏览器、PowerShell 或 Azure CLI 删除 HDInsight 群集

HDInsight 群集计费在创建群集之后便会开始，删除群集后才会停止。 HDInsight 群集按分钟收费，因此不再需要使用群集时，应将其删除。 本文档介绍如何使用 Azure 门户预览、Azure PowerShell 和 Azure CLI 1.0 删除群集。

> [AZURE.IMPORTANT]
> 删除 HDInsight 群集不会删除与该群集关联的 Azure 存储帐户。 这使你可以在将来重复使用数据。

## <a name="azure-portal-preview"></a>Azure 门户预览

1. 登录到 [Azure 门户预览](https://portal.azure.cn) ，并选择 HDInsight 群集。 如果 HDInsight 群集未固定到仪表板，可以使用搜索字段按名称搜索。

    ![门户搜索](./media/hdinsight-delete-cluster/navbar.png)

2. 打开群集的边栏选项卡后，选择“删除”  图标。 出现提示时，选择“是”  即可删除该群集。

    ![删除图标](./media/hdinsight-delete-cluster/deletecluster.png)

## <a name="azure-powershell"></a>Azure PowerShell

在 PowerShell 提示符处，使用以下命令删除群集：

    Remove-AzureRmHDInsightCluster -ClusterName CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

## <a name="azure-cli"></a>Azure CLI

在提示符处，使用以下命令删除群集：

    azure hdinsight cluster delete CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

<!--Update_Description: wording update-->