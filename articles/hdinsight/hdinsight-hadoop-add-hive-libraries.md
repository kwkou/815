<properties
    pageTitle="在 HDInsight 群集创建过程中添加 Hive 库 | Azure"
    description="了解如何在群集创建过程中将 Hive 库（jar 文件）添加到 HDInsight 群集。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    translationtype="Human Translation" />
<tags
    ms.assetid="2fd74b8d-c006-45c6-a9e2-72ff5d2d978a"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/14/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.custom="H1Hack27Feb2017,hdinsightactive"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="7ccf2a7e8e44e206ba61e69ae7290130f6910ea8"
    ms.lasthandoff="04/28/2017" />

# <a name="add-custom-hive-libraries-when-creating-your-hdinsight-cluster"></a>创建 HDInsight 群集时添加自定义 Hive 库

如果你有经常与 HDInsight 上的 Hive 配合使用的库，本文档包含有关在群集创建期间使用脚本操作预加载库的信息。 使用本文档中的步骤添加的库已在 Hive 中正式发布 - 无需使用 [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) 加载它们。

## <a name="how-it-works"></a>工作原理

创建群集时，可以选择指定脚本操作，以便在创建群集节点时在其上运行脚本。 本文档中的脚本接受单个参数，即包含要预加载的库（存储为 jar 文件）的 WASB 位置。

在群集创建过程中，该脚本将枚举文件、将这些文件复制到头节点和工作节点上的 `/usr/lib/customhivelibs/` 目录，然后将它们添加到 `core-site.xml` 文件中的 `hive.aux.jars.path` 属性。 在基于 Linux 的群集中，它还会针对文件位置更新 `hive-env.sh` 文件。

> [AZURE.NOTE]
> 使用本文中的脚本操作使库可用于以下方案：
><p> * **基于 Linux 的 HDInsight** - 使用 Hive 客户端、**WebHCat** 和 **HiveServer2** 时。
><p> * **基于 Windows 的 HDInsight** - 使用 Hive 客户端和 **WebHCat** 时。

## <a name="the-script"></a>脚本

**脚本位置**

对于 **基于 Linux 的群集**： [https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh)

对于 **基于 Windows 的群集**： [https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

**要求**

* 这些脚本必须同时应用于“头节点”和“辅助角色节点”。

* 要安装的 jar 必须存储在**单个容器**中的 Azure Blob 存储中。

* 在创建期间，包含 jar 文件的库的存储帐户 **必须** 链接到 HDInsight 群集。 它必须是默认的存储帐户，或通过 __可选配置__添加的帐户。

* 必须指定容器的 WASB 路径作为脚本操作的参数。 例如，如果 jar 存储在名为 **mystorage** 的存储帐户上名为 **libs** 的容器中，则该参数应为 **wasbs://libs@mystorage.blob.core.chinacloudapi.cn/**。

    > [AZURE.NOTE]
    > 本文档假定你已创建存储帐户、blob 容器，并已将文件上传到该容器。
    > <p>
    > 如果尚未创建存储帐户，可以通过 [Azure 门户预览](https://portal.azure.cn)创建该帐户。 然后可以使用实用工具（如 [Azure 存储资源管理器](http://storageexplorer.com/) ）在帐户中创建容器并将文件上载到该容器。

## <a name="create-a-cluster-using-the-script"></a>使用脚本创建群集。

> [AZURE.NOTE]
> 以下步骤创建基于 Linux 的 HDInsight 群集。 若要创建基于 Windows 的群集，创建群集时请选择 **Windows** 作为群集 OS，并使用 Windows (PowerShell) 脚本而不是 bash 脚本。
> <p>
> 也可以使用 Azure PowerShell 或 HDInsight .NET SDK 来使用此脚本创建群集。 有关使用这些方法的详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

1. 使用[预配 Linux 上的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)中的步骤开始预配群集，但不要完成预配。

2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供以下信息：

    * **名称**：输入脚本操作的友好名称。

    * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh

    * **标头**：选中此选项。

    * **工作节点**：选中此选项。

    * **ZOOKEEPER**：将此项留空。

    * **参数**：输入包含 jar 的容器和存储帐户的 WASB 地址。 例如，**wasbs://libs@mystorage.blob.core.chinacloudapi.cn/**。

3. 在“脚本操作”的底部，使用“选择”按钮保存配置。

4. 在“可选配置”边栏选项卡上，选择“链接存储帐户”，然后选择“添加存储密钥”链接。 选择包含 jar 的存储帐户，然后使用“选择”按钮保存设置并返回“可选配置”边栏选项卡。

5. 使用“可选配置”边栏选项卡底部的“选择”按钮以保存可选配置信息。

6. 继续按[预配 Linux 上的 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)中所述预配群集。

群集创建完成后，你将能够使用通过此脚本从 Hive 添加的 jar，而无需使用 `ADD JAR` 语句。

## <a name="next-steps"></a>后续步骤

有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)