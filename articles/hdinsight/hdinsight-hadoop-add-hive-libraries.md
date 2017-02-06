<properties
    pageTitle="在 HDInsight 群集创建过程中添加 Hive 库 | Azure"
    description="了解如何在群集创建过程中将 Hive 库（jar 文件）添加到 HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="2fd74b8d-c006-45c6-a9e2-72ff5d2d978a"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 在 HDInsight 群集创建过程中添加 Hive 库

如果你有经常与 HDInsight 上的 Hive 配合使用的库，本文档包含有关在群集创建期间使用脚本操作预加载库的信息。这使得库在 Hive 中全局可用（无需使用 [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) 加载它们。）

## 工作原理

创建群集时，可以选择指定脚本操作，以便在创建群集节点时在其上运行脚本。本文档中的脚本接受单个参数，即包含将预加载的库（存储为 jar 文件）的 WASB 位置。

在群集创建过程中，该脚本将枚举文件、将这些文件复制到头节点和工作节点上的 `/usr/lib/customhivelibs/` 目录，然后将它们添加到 `core-site.xml` 文件中的 `hive.aux.jars.path` 属性。在基于 Linux 的群集中，它还会使用这些文件的位置更新 `hive-env.sh` 文件。

> [AZURE.NOTE]
使用本文中的脚本操作使库可用于以下方案：
><p>
><p> *基于 Linux 的 HDInsight** - 当使用 **Hive 命令行**、**WebHCat** (Templeton) 和 **HiveServer2** 时。
><p> * **基于 Windows 的 HDInsight** - 当使用 **Hive 命令行**和 **WebHCat** (Templeton) 时。
>
>

## 脚本
**脚本位置**

对于**基于 Linux 的群集**：[https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh)

对于**基于 Windows 的群集**：[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

**要求**

* 这些脚本必须同时应用于**头节点**和**工作节点**。
* 要安装的 jar 必须存储在**单个容器**中的 Azure Blob 存储中。
* 在创建期间，包含 jar 文件的库的存储帐户**必须**链接到 HDInsight 群集。可以通过两种方式之一完成此操作：

    * 在群集的默认存储帐户上的容器中。
    * 在链接的存储容器上的容器中。例如，在门户中，可以使用“可选配置”、“链接存储帐户”添加额外存储。
* 必须指定容器的 WASB 路径作为脚本操作的参数。例如，如果 jar 存储在名为 **mystorage** 的存储帐户上名为 **libs** 的容器中，则该参数应为 **wasbs://libs@mystorage.blob.core.chinacloudapi.cn/**。

    > [AZURE.NOTE]
    本文档假定你已创建存储帐户、blob 容器，并已将文件上传到该容器。
    ><p>
    > 如果尚未创建存储帐户，可以通过 [Azure 门户预览](https://portal.azure.cn)创建该帐户。然后可以使用实用工具（如 [Azure 存储资源管理器](http://storageexplorer.com/)）在帐户中新建容器并将文件上传到该容器。
    >
    >

## 使用脚本创建群集。
> [AZURE.NOTE]
以下步骤新建基于 Linux 的 HDInsight 群集。若要新建基于 Windows 的群集，请在创建群集时选择“Windows”作为群集 OS，并使用 Windows (PowerShell) 脚本而不是 bash 脚本。
>
> 还可以使用此脚本通过 Azure PowerShell 或 HDInsight .NET SDK 创建群集。有关使用这些方法的详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。
>
>

1. 使用[预配 Linux 上的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)中的步骤开始预配群集，但不要完成预配。
2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供如下所示信息：

    * **名称**：输入脚本操作的友好名称。
    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh
    * **头节点**：选中此选项
    * **工作节点**：选中此选项。
    * **ZOOKEEPER**：将此项留空。
    * **参数**：输入包含 jar 的容器和存储帐户的 WASB 地址。例如 **wasbs://libs@mystorage.blob.core.chinacloudapi.cn/**。
3. 在“脚本操作”的底部，使用“选择”按钮保存配置。
4. 在“可选配置”边栏选项卡上，选择“链接存储帐户”，然后选择“添加存储密钥”链接。选择包含 jar 的存储帐户，然后使用“选择”按钮保存设置并返回“可选配置”边栏选项卡。
5. 使用“可选配置”边栏选项卡底部的“选择”按钮以保存可选配置信息。
6. 继续按[预配 Linux 上的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)中所述预配群集。

群集创建完成后，你将能够使用通过此脚本从 Hive 添加的 jar，而无需使用 `ADD JAR` 语句。

## 后续步骤
在本文档中，你已了解如何在群集创建过程中将 jar 文件中包含的 Hive 库添加到 HDInsight 群集中。有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)

<!---HONumber=Mooncake_0120_2017-->