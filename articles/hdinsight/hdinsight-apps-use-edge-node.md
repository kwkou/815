<!-- not suitable for Mooncake -->

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
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/15/2016"
    wacn.date="02/06/2017"
    ms.author="jgao" />

# 在 HDInsight 中使用空边缘节点
了解如何将空边缘节点添加到基于 Linux 的 HDInsight 群集。空边缘节点是安装并配置了与头节点中相同的客户端工具，但未运行 hadoop 服务的 Linux 虚拟机。可以使用该边缘节点来访问群集、测试客户端应用程序和托管客户端应用程序。

可以将空边缘节点添加到现有 HDInsight 群集，或者在创建群集时将此类节点添加到新群集。添加空边缘节点的操作是使用 Azure Resource Manager 模板完成的。以下示例演示如何使用模板执行此操作：

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

如示例中所示，可以选择性地调用[脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster/)来执行其他配置，例如，在边缘节点中安装 [Apache Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)。

创建边缘节点后，可以使用 SSH 连接到该节点，运行客户端工具访问 HDInsight 中的 Hadoop 群集。

## 将边缘节点添加到现有群集
本部分介绍如何使用 Resource Manager 模板将边缘节点添加到现有 HDInsight 群集。可以在 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-with-edge-node/)中找到 Resource Manager 模板。Resource Manager 模板调用位于 https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-with-edge-node/scripts/EmptyNodeSetup.sh 中的脚本操作脚本。该脚本不执行任何操作。此处只是演示如何从 Resource Manager 模板调用脚本操作。

**将空边缘节点添加到现有群集**

1. 创建一个 HDInsight 群集（如果没有）。请参阅 [Hadoop tutorial: Get started using Linux-based Hadoop in HDInsight](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/)（Hadoop 教程：开始使用 HDInsight 中基于 Linux 的 Hadoop）
2. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Azure Resource Manager 模板。
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>
3. 配置以下属性：
   
   * **订阅**：选择用于创建此群集的 Azure 订阅。
   * **资源组**：选择现有 HDInsight 群集所用的资源组。
   * **位置**：选择现有 HDInsight 群集的位置。
   * **群集名称**：输入现有 HDInsight 群集的名称。
   * **边缘节点大小**：选择一个 VM 大小。
   * **边缘节点前缀**：默认值为 **new**。如果使用默认值，边缘节点的名称为 **new-edgenode**。可以通过门户自定义前缀。也可以通过模板自定义完整名称。
4. 选中“我同意上述条款和条件”，然后单击“购买”创建边缘节点。

## 创建群集时添加边缘节点
本部分介绍如何使用 Resource Manager 模板创建包含边缘节点的 HDInsight 群集。可以在 [Azure 快速启动模板库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-linux-with-edge-node/)中找到 Resource Manager 模板。Resource Manager 模板调用位于 https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-with-edge-node/scripts/EmptyNodeSetup.sh 中的脚本操作脚本。该脚本不执行任何操作。此处只是演示如何从 Resource Manager 模板调用脚本操作。

**将空边缘节点添加到现有群集**

1. 创建一个 HDInsight 群集（如果没有）。请参阅 [Hadoop tutorial: Get started using Linux-based Hadoop in HDInsight](/documentation/articles/hdinsight-hadoop-tutorial-get-started-windows/)（Hadoop 教程：开始使用 HDInsight 中基于 Linux 的 Hadoop）
2. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Azure Resource Manager 模板。
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>
3. 配置以下属性：
   
   * **订阅**：选择用于创建此群集的 Azure 订阅。
   * **资源组**：创建用于群集的新资源组。
   * **位置**：选择资源组的位置。
   * **群集名称**：输入要创建的新群集的名称。
   * **群集登录用户名**：输入 Hadoop HTTP 用户名。默认名称为 **admin**。
   * **群集登录密码**：输入 Hadoop HTTP 用户密码。
   * **SSH 用户名**：输入 SSH 用户名。默认名称为 **sshuser**。
   * **SSH 密码**：输入 SSH 用户密码。
   * **安装脚本操作**：保留本教程自始至终使用的默认值。
     
     已在模板中硬编码的某些属性：群集类型、群集辅助角色节点计数、边缘节点大小，以及边缘节点名称。
4. 选中“我同意上述条款和条件”，然后单击“购买”创建带边缘节点的群集。

## 访问边缘节点
边缘节点 ssh 终结点为 <边缘节点名称>.<群集名称>-ssh.azurehdinsight.cn:22。例如，new-edgenode.myedgenode0914-ssh.azurehdinsight.cn:22。

在 Azure 门户预览上，边缘节点显示为应用程序。门户中提供了使用 SSH 访问边缘节点时所需的信息。

**验证边缘节点 SSH 终结点**

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 打开包含边缘节点的 HDInsight 群集。
3. 在群集边栏选项卡中单击“应用程序”。此时将显示该边缘节点。默认名称为 **new-edgenode**。
4. 单击该边缘节点。此时将显示 SSH 终结点。

**在边缘节点上使用 Hive**

1. 使用 SSH 连接到边缘节点。请参阅 [Use SSH with Linux-based Hadoop on HDInsight from Linux, Unix, or OS X](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)（在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用）或 [Use SSH with Linux-based Hadoop on HDInsight from Windows](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)（在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用）。
2. 使用 SSH 连接到边缘节点后，使用以下命令打开 Hive 控制台：
   
        hive
3. 运行以下命令显示群集中的 Hive 表：
   
        show tables;

## 删除边缘节点
可以在 Azure 门户预览中删除边缘节点。

**访问边缘节点**

1. 登录到 [Azure 门户预览](https://portal.azure.cn)。
2. 打开包含边缘节点的 HDInsight 群集。
3. 在群集边栏选项卡中单击“应用程序”。此时将显示边缘节点的列表。
4. 右键单击要删除的边缘节点，然后单击“删除”。
5. 单击“是”确认。

## 后续步骤
本文介绍了如何添加边缘节点以及如何访问边缘节点。若要了解更多信息，请参阅下列文章：

* [Install HDInsight applications](/documentation/articles/hdinsight-apps-install-applications/)（安装 HDInsight 应用程序）：了解如何将 HDInsight 应用程序安装到群集。
* [Install custom HDInsight applications](/documentation/articles/hdinsight-apps-install-custom-applications/)（安装自定义 HDInsight 应用程序）：了解如何将未发布的 HDInsight 应用程序部署到 HDInsight。
* [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）：了解如何定义 HDInsight 应用程序。
* [Customize Linux-based HDInsight clusters using Script Action](/documentation/articles/hdinsight-hadoop-customize-cluster/)（使用脚本操作自定义基于 Linux 的 HDInsight 群集）：了解如何使用脚本操作安装其他应用程序。
* [Create Linux-based Hadoop clusters in HDInsight using Resource Manager templates](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)（使用 Resource Manager 模板在 HDInsight 中创建基于 Linux 的 Hadoop 群集）：了解如何调用 Resource Manager 模板来创建 HDInsight 群集。

<!---HONumber=Mooncake_1205_2016-->