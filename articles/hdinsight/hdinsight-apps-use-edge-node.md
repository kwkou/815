<properties
    pageTitle="在 HDInsight 中使用空边缘节点 | Azure"
    description="如何将空边缘节点添加到可充当客户端的 HDInsight 群集，以及如何测试/托管 HDInsight 应用程序。"
    services="hdinsight"
    editor="cgronlun"
    manager="jhubbard"
    author="mumian"
    tags="azure-portal"
    documentationcenter="" />
<tags
    ms.assetid="cdc7d1b4-15d7-4d4d-a13f-c7d3a694b4fb"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/01/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="659378b38465d095c4edd9cea07a197db0bcfbd3"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-empty-edge-nodes-in-hdinsight"></a>在 HDInsight 中使用空边缘节点

了解如何将空边缘节点添加到基于 Linux 的 HDInsight 群集。 空边缘节点是安装并配置了与头节点中相同的客户端工具，但未运行 hadoop 服务的 Linux 虚拟机。 可以使用该边缘节点来访问群集、测试客户端应用程序和托管客户端应用程序。 

可以将空边缘节点添加到现有 HDInsight 群集，或者在创建群集时将此类节点添加到新群集。 添加空边缘节点的操作是使用 Azure Resource Manager 模板完成的。  以下示例演示如何使用模板执行此操作：

    "resources": [
        {
            "name": "[concat(parameters('clusterName'),'/', variables('applicationName'))]",
            "type": "Microsoft.HDInsight/clusters/applications",
            "apiVersion": "2015-03-01-preview",
            "dependsOn": [ "[concat('Microsoft.HDInsight/clusters/',parameters('clusterName'))]" ],
            "properties": {
                "marketPlaceIdentifier": "EmptyNode",
                "computeProfile": {
                    "roles": [{
                        "name": "edgenode",
                        "targetInstanceCount": 1,
                        "hardwareProfile": {
                            "vmSize": "Standard_D3"
                        }
                    }]
                },
                "installScriptActions": [{
                    "name": "[concat('emptynode','-' ,uniquestring(variables('applicationName')))]",
                    "uri": "[parameters('installScriptAction')]",
                    "roles": ["edgenode"]
                }],
                "uninstallScriptActions": [],
                "httpsEndpoints": [],
                "applicationType": "CustomApplication"
            }
        }
    ],

如示例中所示，可以选择性地调用[脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)来执行其他配置，例如，在边缘节点中安装 [Apache Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)。 脚本操作脚本必须可在 Web 上公开访问。  例如，如果该脚本存储在 Azure 存储中，请使用公共容器或公共 blob。

边缘节点虚拟机大小必须满足 HDInsight 群集工作节点 vm 的大小要求。 有关建议的工作节点 vm 的大小信息，请参阅[在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/#cluster-types)。

创建边缘节点后，可以使用 SSH 连接到该节点，运行客户端工具访问 HDInsight 中的 Hadoop 群集。

## <a name="add-an-edge-node-to-an-existing-cluster"></a>将边缘节点添加到现有群集
本部分介绍如何使用 Resource Manager 模板将边缘节点添加到现有 HDInsight 群集。  可以在 [GitHub](https://github.com/hdinsight/Iaas-Applications/tree/master/EmptyNode) 中找到 Resource Manager 模板。 Resource Manager 模板调用位于 https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/EmptyNode/scripts/EmptyNodeSetup.sh 的脚本操作脚本。 该脚本不执行任何操作。  它只是演示如何从 Resource Manager 模板调用脚本操作。

**将空边缘节点添加到现有群集**

1. 创建一个 HDInsight 群集（如果没有）。  请参阅 [Hadoop 教程：开始使用 HDInsight 中基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。
2. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Azure Resource Manager 模板。 

    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FEmptyNode%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>
3. 配置以下属性：

    * **订阅**：选择用于创建此群集的 Azure 订阅。
    * **资源组**：选择现有 HDInsight 群集所用的资源组。
    * **位置**：选择现有 HDInsight 群集的位置。
    * **群集名称**：输入现有 HDInsight 群集的名称。
    * **边缘节点大小**：选择一个 VM 大小。 VM 大小必须满足工作节点 VM 大小要求。 有关建议的工作节点 vm 的大小信息，请参阅[在 HDInsight 中创建 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/#cluster-types)。
    * **边缘节点前缀**：默认值为 **new**。  如果使用默认值，边缘节点的名称为 **new-edgenode**。  可以通过门户自定义前缀。 也可以通过模板自定义完整名称。
4. 单击“法律条款”，然后单击“购买”。 确认已选中“固定到仪表板”复选框，然后单击“创建”。

## <a name="add-an-edge-node-when-creating-a-cluster"></a>创建群集时添加边缘节点
本部分介绍如何使用 Resource Manager 模板创建包含边缘节点的 HDInsight 群集。  可以在 [Azure 快速启动模板库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-with-edge-node/)中找到 Resource Manager 模板。 Resource Manager 模板调用位于 https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-with-edge-node/scripts/EmptyNodeSetup.sh 的脚本操作脚本。 该脚本不执行任何操作。  它只是演示如何从 Resource Manager 模板调用脚本操作。

**将空边缘节点添加到现有群集**

1. 创建一个 HDInsight 群集（如果没有）。  请参阅 [Hadoop 教程：开始使用 HDInsight 中基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)。
2. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Azure Resource Manager 模板。 

    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>

    >[AZURE.NOTE]
    > 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。 例如，将一些终结点 -“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”；将允许的位置更改为“China North”和“China East”；将 HDInsight Linux 版本更改为 Azure 中国区支持的版本 3.5。

3. 配置以下属性：

    * **订阅**：选择用于创建此群集的 Azure 订阅。
    * **资源组**：创建用于群集的新资源组。
    * **位置**：选择资源组的位置。
    * **群集名称**：输入要创建的新群集的名称。
    * **群集登录用户名**：输入 Hadoop HTTP 用户名。  默认名称为 **admin**。
    * **群集登录密码**：输入 Hadoop HTTP 用户密码。
    * **SSH 用户名**：输入 SSH 用户名。 默认名称为 **sshuser**。
    * **SSH 密码**：输入 SSH 用户密码。
    * **安装脚本操作**：保留本教程自始至终使用的默认值。

    模板中已硬编码某些属性：群集类型、群集辅助角色节点计数、边缘节点大小和边缘节点名称。
4. 单击“法律条款”，然后单击“购买”。 确认已选中“固定到仪表板”复选框，然后单击“创建”。

## <a name="access-an-edge-node"></a>访问边缘节点
边缘节点 ssh 终结点为 &lt;边缘节点名称>.&lt;群集名称>-ssh.azurehdinsight.cn:22。  例如，new-edgenode.myedgenode0914-ssh.azurehdinsight.cn:22。

在 Azure 门户预览中，边缘节点显示为应用程序。  门户中提供了使用 SSH 访问边缘节点时所需的信息。

**验证边缘节点 SSH 终结点**

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 打开包含边缘节点的 HDInsight 群集。
3. 在群集边栏选项卡中单击“应用程序”。 此时将显示该边缘节点。  默认名称为 **new-edgenode**。
4. 单击该边缘节点。 此时将显示 SSH 终结点。

**在边缘节点上使用 Hive**

1. 使用 SSH 连接到边缘节点。 有关信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

2. 使用 SSH 连接到边缘节点后，使用以下命令打开 Hive 控制台：

        hive
3. 运行以下命令显示群集中的 Hive 表：

        show tables;

## <a name="delete-an-edge-node"></a>删除边缘节点
可以在 Azure 门户预览中删除边缘节点。

**访问边缘节点**

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 打开包含边缘节点的 HDInsight 群集。
3. 在群集边栏选项卡中单击“应用程序”。 此时将显示边缘节点的列表。  
4. 右键单击要删除的边缘节点，然后单击“删除”。
5. 单击“是”确认。

## <a name="next-steps"></a>后续步骤
本文介绍了如何添加边缘节点以及如何访问边缘节点。 若要了解更多信息，请参阅下列文章：

* [安装 HDInsight 应用程序](/documentation/articles/hdinsight-apps-install-applications/)：了解如何将 HDInsight 应用程序安装到群集。
* [安装自定义 HDInsight 应用程序](/documentation/articles/hdinsight-apps-install-custom-applications/)：了解如何将未发布的 HDInsight 应用程序部署到 HDInsight。
* [MSDN：安装 HDInsight 应用程序](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)：了解如何定义 HDInsight 应用程序。
* [使用脚本操作自定义基于 Linux 的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)：了解如何使用脚本操作安装其他应用程序。
* [使用 Resource Manager 模板在 HDInsight 中创建基于 Linux 的 Hadoop 群集](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)：了解如何调用 Resource Manager 模板创建 HDInsight 群集。

<!--Update_Description: wording update-->