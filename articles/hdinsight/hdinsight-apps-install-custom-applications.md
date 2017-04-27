<properties
    pageTitle="在 Azure HDInsight 上安装自定义 Hadoop 应用程序 | Azure"
    description="了解如何在 HDInsight 上安装 HDInsight 应用程序。"
    services="hdinsight"
    documentationcenter=""
    author="mumian"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="e556b29c-8176-4bc5-a90b-aa01abfd3aee"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/22/2017"
    wacn.date="04/27/2017"
    ms.author="jgao" />  


# 在 Azure HDInsight 上安装自定义 Hadoop 应用程序

本文介绍如何在 Azure HDInsight 上安装尚未发布到 Azure 门户预览的 Hadoop 应用程序。本文中要安装的应用程序是 [Hue](http://gethue.com/)。

HDInsight 应用程序是用户可以在基于 Linux 的 HDInsight 群集上安装的应用程序。这些应用程序可能是 Microsoft、独立软件供应商 (ISV) 或你自己开发的。

其他相关文章：

* [Install HDInsight applications](/documentation/articles/hdinsight-apps-install-applications/)（安装 HDInsight 应用程序）：了解如何将 HDInsight 应用程序安装到群集。
* [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）：了解如何定义 HDInsight 应用程序。

## 先决条件
如果想要在现有的 HDInsight 群集上安装 HDInsight 应用程序，必须有一个 HDInsight 群集。若要创建群集，请参阅[创建群集](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/#create-cluster)。也可以在创建 HDInsight 群集时安装 HDInsight 应用程序。

## 安装 HDInsight 应用程序
可以在创建群集时安装 HDInsight 应用程序，也可以将它安装到现有的 HDInsight 群集。有关如何定义 Azure Resource Manager 模板，请参阅 [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）。

部署此应用程序 (Hue) 时所需的文件：

* [azuredeploy.json](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/azuredeploy.json)：用于安装 HDInsight 应用程序的 Resource Manager 模板。有关如何开发自己的 Resource Manager 模板，请参阅 [MSDN：安装 HDInsight 应用程序](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)。
* [hue-install\_v0.sh](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/scripts/Hue-install_v0.sh)：Resource Manager 模板为配置边缘节点而调用的脚本操作。
* [hue-binaries.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz)：从 hui-install\_v0.sh 调用的 hue 二进制文件。
* [hue-binaries-14-04.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz)：从 hui-install\_v0.sh 调用的 hue 二进制文件。
* [webwasb-tomcat.tar.gz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/webwasb-tomcat.tar.gz)：从 hui-install\_v0.sh 调用的示例 Web 应用程序 (Tomcat)。

**将 Hue 安装到现有的 HDInsight 群集**

1. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Resource Manager 模板。
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FHue%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/deploy-to-azure.png" alt="Deploy to Azure"></a>
   
    此按钮可在 Azure 门户预览上打开 Resource Manager 模板。Resource Manager 模板位于 [https://github.com/hdinsight/Iaas-Applications/tree/master/Hue](https://github.com/hdinsight/Iaas-Applications/tree/master/Hue) 中。若要了解如何编写此 Resource Manager 模板，请参阅 [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）。
2. 在“参数”边栏选项卡中，输入以下内容：
   
    * “ClusterName”：输入要在其中安装应用程序的群集的名称。此群集必须是现有的群集。
3. 单击“确定”以保存参数。
4. 在“自定义部署”边栏选项卡中输入“资源组”。资源组是对群集、依赖存储帐户和其他资源进行分组的容器。必须使用与群集相同的资源组。
5. 单击“法律条款”，然后单击“创建”。
6. 确认已选中“固定到仪表板”复选框，然后单击“创建”。可以从固定到门户仪表板的磁贴和门户通知查看安装状态（单击门户顶部的铃铛图标）。安装此应用程序大约需要 10 分钟。

**若要创建群集的同时安装色调**

1. 单击以下图像登录到 Azure，然后在 Azure 门户预览中打开 Resource Manager 模板。
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhdinsightapps%2Fcreate-linux-based-hadoop-cluster-in-hdinsight.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/deploy-to-azure.png" alt="Deploy to Azure"></a>
   
    >[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，将一些终结点 -“blob.core.windows.net”替换为“blob.core.chinacloudapi.cn”，将“cloudapp.azure.com”替换为“chinacloudapp.cn”；将允许的位置更改为“China North”和“China East”；将 HDInsight Linux 版本更改为 Azure 中国区支持的版本 3.5。

    此按钮可在 Azure 门户预览上打开 Resource Manager 模板。Resource Manager 模板位于 [https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json](https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json) 中。若要了解如何编写此 Resource Manager 模板，请参阅 [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）。
2. 根据说明来创建群集并安装 Hue。有关创建 HDInsight 群集的详细信息，请参阅 [在 HDInsight 中创建基于 Linux 的 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)。

除了 Azure 门户预览，也可以使用 [Azure PowerShell](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/#deploy-with-powershell) 和 [Azure CLI](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/#deploy-with-azure-cli) 来调用 Resource Manager 模板。

## 验证安装
可以在 Azure 门户预览中检查应用程序状态，从而验证应用程序安装。此外，还可以验证所有 HTTP 终结点和网页（如果有）是否按预期出现：

**打开 Hue 门户**

1. 登录 [Azure 门户预览](https://portal.azure.cn)。
2. 在左侧菜单中单击“HDInsight 群集”。如果未看到，请单击“浏览”，然后单击“HDInsight 群集”。
3. 单击已安装应用程序的群集。
4. 在“设置”边栏选项卡中，单击“常规”类别下的“应用程序”。应会显示出“已安装的应用”边栏选项卡中列出了“hue”。
5. 单击列表中的“hue”以列出属性。
6. 单击网页链接以验证网站；在浏览器中打开 HTTP 终结点以验证 Hue Web UI，并使用 [PuTTY](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/) 或其他 [SSH 客户端](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)打开 SSH 终结点。

## <a name="troubleshoot-the-installation"></a>故障排除安装问题
可以通过门户通知查看应用程序安装状态（单击门户顶部的铃铛图标）。

如果应用程序安装失败，可以从 3 个位置查看错误消息和调试信息：

* HDInsight 应用程序：常规错误信息。
  
    从门户打开群集，然后在“设置”边栏选项卡中单击“应用程序”：
  
    ![hdinsight 应用程序安装错误](./media/hdinsight-apps-install-applications/hdinsight-apps-error.png)
* HDInsight 脚本操作：如果 HDInsight 应用程序的错误消息指出脚本操作失败，“脚本操作”窗格会显示有关脚本失败的详细信息。
  
    在“设置”边栏选项卡中单击“脚本操作”。脚本操作历史记录中显示了错误消息
  
    ![hdinsight 应用程序脚本操作错误](./media/hdinsight-apps-install-applications/hdinsight-apps-script-action-error.png)
* Ambari Web UI：如果安装脚本是失败的原因，请使用 Ambari Web UI 来检查有关安装脚本的完整日志。
  
    有关详细信息，请参阅[疑难解答](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#troubleshooting)。

## 删除 HDInsight 应用程序
可通过多种方式删除 HDInsight 应用程序。

### 使用门户
**使用门户删除应用程序**

1. 登录 [Azure 门户预览](https://portal.azure.cn)。
2. 在左侧菜单中单击“HDInsight 群集”。如果未看到，请单击“浏览”，然后单击“HDInsight 群集”。
3. 单击已安装应用程序的群集。
4. 在“设置”边栏选项卡中，单击“常规”类别下的“应用程序”。你将看到已安装的应用程序列表。在本教程中，“hue”列在“已安装的应用”边栏选项卡中。
5. 右键单击想要删除的应用程序，然后单击“删除”。
6. 单击“是”确认。

在门户中，你还可以删除群集，或删除包含应用程序的资源组。

### 使用 Azure PowerShell
使用 Azure PowerShell 可以删除群集或删除资源组。请参阅[使用 Azure PowerShell 删除群集](/documentation/articles/hdinsight-administer-use-powershell/#delete-clusters)。

### 使用 Azure CLI
使用 Azure CLI 可以删除群集或删除资源组。请参阅[使用 Azure CLI 删除群集](/documentation/articles/hdinsight-administer-use-command-line/#delete-clusters)。

## 后续步骤
* [MSDN: Install an HDInsight application](https://msdn.microsoft.com/zh-cn/library/mt706515.aspx)（MSDN：安装 HDInsight 应用程序）：了解如何开发用于部署 HDInsight 应用程序的 Resource Manager 模板。
* [Install HDInsight applications](/documentation/articles/hdinsight-apps-install-applications/)（安装 HDInsight 应用程序）：了解如何将 HDInsight 应用程序安装到群集。
* [Customize Linux-based HDInsight clusters using Script Action](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)（使用脚本操作自定义基于 Linux 的 HDInsight 群集）：了解如何使用脚本操作安装其他应用程序。
* [Create Linux-based Hadoop clusters in HDInsight using Resource Manager templates](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)（使用 Resource Manager 模板在 HDInsight 中创建基于 Linux 的 Hadoop 群集）：了解如何调用 Resource Manager 模板来创建 HDInsight 群集。
* [在 HDInsight 中使用空边缘节点](/documentation/articles/hdinsight-apps-use-edge-node/)：了解如何使用空边缘节点访问 HDInsight 群集、测试和托管 HDInsight 应用程序。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->