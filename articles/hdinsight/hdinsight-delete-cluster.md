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
    ms.date="10/28/2016"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 如何删除 HDInsight 群集
HDInsight 群集在创建群集时开始计费，在删除群集后停止计费并且费用每分钟按比例分配，因此，应始终在不再使用群集时删除群集。本文档介绍如何使用 Azure 门户预览、Azure PowerShell 和 Azure CLI 删除群集。

> [AZURE.IMPORTANT]
删除 HDInsight 群集不会删除与群集关联的 Azure 存储帐户。这样你就可以保留和重复使用由群集存储的所有数据。
> 
> 

## Azure 门户预览
1. 登录到 [Azure 门户预览](https://portal.azure.cn)，并选择 HDInsight 群集。如果 HDInsight 群集未固定到仪表板，可以使用导航栏右侧的搜索字段（放大镜图标）按名称搜索它。
   
    ![门户搜索](./media/hdinsight-delete-cluster/navbar.png)  

2. 打开群集的边栏选项卡后，选择“删除”图标。出现提示时，选择“是”即可删除该群集。
   
    ![删除图标](./media/hdinsight-delete-cluster/deletecluster.png)  


## Azure PowerShell
在 PowerShell 提示符处，使用以下命令删除群集：

    Remove-AzureRmHDInsightCluster -ClusterName CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

## Azure CLI
在提示符处，使用以下命令删除群集：

    azure hdinsight cluster delete CLUSTERNAME

将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->