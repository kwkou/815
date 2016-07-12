<properties
pageTitle="在 HDInsight 群集创建过程中添加 Hive 库 | Azure"
description="了解如何在群集创建中将 Hive 库（jar 文件）添加到 HDInsight 群集中。"
services="hdinsight"
documentationCenter=""
authors="Blackmist"
manager="paulettm"
editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="04/20/2016"
	wacn.date="06/29/2016"/>

#在 HDInsight 群集创建过程中添加 Hive 库

如果你有经常与 HDInsight 上的 Hive 配合使用的库，本文档包含有关在群集创建期间使用脚本操作预加载库的信息。这使得库在 Hive 中全局可用（无需使用 [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) 加载它们。）

##工作原理

创建群集时，可以选择指定脚本操作，以便在创建群集节点时在其上运行脚本。本文档中的脚本接受单个参数，即包含将预加载的库 （存储为 jar 文件）的 WASB 位置。

在群集创建过程中，该脚本将枚举文件、将这些文件复制到头节点和工作节点上的 `/usr/lib/customhivelibs/` 目录，然后将它们添加到 `core-site.xml` 文件中的 `hive.aux.jars.path` 属性。

> [AZURE.NOTE] 使用本文中的脚本操作使库可用于以下方案：
> * __基于 Windows 的 HDInsight__ - 当使用 __Hive 命令行__ 和 __WebHCat__ (Templeton) 时。

##脚本

__脚本位置__

对于__基于 Windows 的群集__：[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

__要求__

* 这些脚本必须同时应用于__头节点__和__工作节点__。

* 要安装的 jar 必须存储在__单个容器__中的 Azure Blob 存储中。

* 在创建期间，包含 jar 文件的库的存储帐户__必须__链接到 HDInsight 群集。可以通过两种方式之一来完成此操作：

    * 在群集的默认存储帐户上的容器中。
    
    * 在链接的存储容器上的容器中。例如，在经典管理门户中可以使用“可选配置”、“链接存储帐户”添加额外的存储空间。

* 必须指定容器的 WASB 路径作为脚本操作的参数。例如，如果 jar 存储在名为 __mystorage__ 的存储帐户上名为 __libs__ 的容器中，则该参数应为 __wasb://libs@mystorage.blob.core.chinacloudapi.cn/__。

    > [AZURE.NOTE] 本文档假定你已创建存储帐户、blob 容器，并已将文件上载到该容器。
    >
    > 如果尚未创建存储帐户，可以通过 [Azure 经典管理门户](https://manage.windowsazure.cn)创建该帐户。然后可以使用实用程序（如 [Azure 存储资源管理器](http://storageexplorer.com/)）在帐户中创建一个新容器并将文件上载到该容器。

##使用脚本创建群集。

1. 使用[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters-v1/#portal)中的步骤开始预配群集，但不要完成预配。

2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供如下所示信息：

    * __名称__：输入脚本操作的友好名称。
    * __脚本 URI__：https://hdiconfigactions.blob.core.windows.net/windowssetupcustomhivelibsv01/setup-customhivelibs-v01.sh
    * __标头__：选中此选项
    * __辅助角色__：选中此选项。
    * __ZOOKEEPER__：将此项留空。
    * __参数__：输入包含 jar 的容器和存储帐户的 WASB 地址。例如，__wasb://libs@mystorage.blob.core.chinacloudapi.cn/__。

3. 在“脚本操作”的底部，使用“选择”按钮保存配置。

4. 在“可选配置”边栏选项卡上，选择“链接存储帐户”，然后选择“添加存储密钥”链接。选择包含 jar 的存储帐户，然后使用“选择”按钮保存设置并返回“可选配置”边栏选项卡。

5. 使用“可选配置”边栏选项卡底部的“选择”按钮以保存可选配置信息。

6. 按[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters-v1/#portal)中所述，继续预配群集。

群集创建完成后，你将能够使用通过此脚本从 Hive 添加的 jar，而无需使用 `ADD JAR` 语句。

##后续步骤

在本文档中，你已学习如何在群集创建过程中将 jar 文件中包含的 Hive 库添加到 HDInsight 群集中。有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
<!---HONumber=Mooncake_0405_2016-->