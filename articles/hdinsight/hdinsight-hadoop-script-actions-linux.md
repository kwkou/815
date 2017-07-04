<properties
    pageTitle="使用基于 Linux 的 HDInsight 进行脚本操作开发 | Azure"
    description="如何使用脚本操作自定义基于 Linux 的 HDInsight 群集。 使用脚本操作可以通过指定群集配置设置，或者在群集上安装额外的服务、工具或其他软件，来自定义 Azure HDInsight 群集。 "
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="cf4c89cd-f7da-4a10-857f-838004965d3e"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/02/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="77b8d28e83332709f53e3a5b1813f870c79b8513"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="script-action-development-with-hdinsight"></a>使用 HDInsight 进行脚本操作开发

了解如何使用 Bash 脚本自定义 HDInsight 群集。 在创建群集期间和之后，可以通过脚本操作自定义 HDInsight。

> [AZURE.IMPORTANT]
> 本文档中的步骤需要使用 Linux 的 HDInsight 群集。 Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a name="what-are-script-actions"></a>什么是脚本操作

脚本操作是 Azure 在群集节点上运行的以进行配置更改或安装软件的 Bash 脚本。 脚本操作以 root 身份执行，提供对群集节点的完全访问权限。

可通过以下方法应用脚本操作：

| 使用此方法来应用脚本... | 在创建群集期间... | 在运行中的群集上... |
| --- |:---:|:---:|
| Azure 门户 |✓ |✓ |
| Azure PowerShell |✓ |✓ |
| Azure CLI |&nbsp; |✓ |
| HDInsight .NET SDK |✓ |✓ |
| Azure Resource Manager 模板 |✓ |&nbsp; |

有关使用这些方法来应用脚本操作的详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

## <a name="bestPracticeScripting"></a>脚本开发最佳实践

在针对 HDInsight 群集开发自定义脚本时，有些最佳做法要铭记于心：

* [选择目标 Hadoop 版本](#bPS1)
* [选择目标 OS 版本](#bps10)
* [提供指向脚本资源的可靠链接](#bPS2)
* [使用预编译的资源](#bPS4)
* [确保群集自定义脚本是幂等的](#bPS3)
* [确保群集体系结构的高可用性](#bPS5)
* [配置自定义组件以使用 Azure Blob 存储](#bPS6)
* [将信息写入 STDOUT 和 STDERR](#bPS7)
* [将文件另存为包含 LF 行尾的 ASCII](#bps8)
* [使用重试逻辑从暂时性错误中恢复](#bps9)

> [AZURE.IMPORTANT]
> 脚本操作必须在 60 分钟内完成。 否则脚本将会失败。 在节点预配期间，脚本将与其他安装和配置进程一同运行。 争用 CPU 时间和网络带宽等资源可能导致完成脚本所需的时间要长于在开发环境中所需的时间。

### <a name="bPS1"></a>选择目标 Hadoop 版本

不同版本的 HDInsight 有不同版本的 Hadoop 服务和已安装的组件。 如果脚本需要特定版本的服务或组件，你应该只在包含所需组件的 HDInsight 版本中使用该脚本。 可以使用 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/)文档来查找 HDInsight 随附组件版本的相关信息。

### <a name="bps10"></a> 选择目标 OS 版本

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

基于 Linux 的 HDInsight 取决于 Ubuntu Linux 分发版。 不同版本的 HDInsight 依赖不同版本的 Ubuntu，这可能会改变脚本的行为方式。 例如，HDInsight 3.4 及更低版本基于使用 Upstart 的 Ubuntu 版本。 3.5 版本取决于使用 Systemd 的 Ubuntu 16.04。 Systemd 和 Upstart 采用不同的命令，因此你编写的脚本应能与这两者配合使用。

HDInsight 3.4 和 3.5 的另一个重要区别在于 `JAVA_HOME` 现在能够指向 Java 8。

可通过使用 `lsb_release`检查 OS 版本。 以下代码演示如何确定脚本是在 Ubuntu 14 还是 16 上运行：

    OS_VERSION=$(lsb_release -sr)
    if [[ $OS_VERSION == 14* ]]; then
        echo "OS verion is $OS_VERSION. Using hue-binaries-14-04."
        HUE_TARFILE=hue-binaries-14-04.tgz
    elif [[ $OS_VERSION == 16* ]]; then
        echo "OS verion is $OS_VERSION. Using hue-binaries-16-04."
        HUE_TARFILE=hue-binaries-16-04.tgz
    fi
    ...
    if [[ $OS_VERSION == 16* ]]; then
        echo "Using systemd configuration"
        systemctl daemon-reload
        systemctl stop webwasb.service    
        systemctl start webwasb.service
    else
        echo "Using upstart configuration"
        initctl reload-configuration
        stop webwasb
        start webwasb
    fi
    ...
    if [[ $OS_VERSION == 14* ]]; then
        export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    elif [[ $OS_VERSION == 16* ]]; then
        export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    fi

可以在 https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh 中找到包含这些代码片段的完整脚本。

有关 HDInsight 所使用的 Ubuntu 版本，请参阅 [HDInsight 组件版本](/documentation/articles/hdinsight-component-versioning/)文档。

若要了解 Systemd 和 Upstart 之间的差异，请参阅 [Upstart 用户的 Systemd](https://wiki.ubuntu.com/SystemdForUpstartUsers)。

### <a name="bPS2"></a>提供指向脚本资源的可靠链接

在群集的整个生存期内，脚本和关联的资源必须保持可用。 如果在缩放操作期间将新节点添加到了群集，将需要用到这些资源。

最佳做法是下载订阅上 Azure 存储帐户中的所有内容并将其存档。

> [AZURE.IMPORTANT]
> 使用的存储帐户必须是群集的默认存储帐户，或其他任何存储帐户的公共只读容器。

例如，Microsoft 提供的示例存储在 [https://hdiconfigactions.blob.core.windows.net/](https://hdiconfigactions.blob.core.windows.net/) 存储帐户中，这是 HDInsight 团队维护的公共只读容器。

### <a name="bPS4"></a>使用预编译的资源

若要减少运行脚本所花费的时间，请避免使用从源代码编译资源的操作。 请预编译这些资源，并将其存储在 Azure Blob 存储中，使之能够快速下载。

### <a name="bPS3"></a>确保群集自定义脚本是幂等的

脚本必须是幂等的。 如果脚本多次运行，则每次都应将群集返回到相同的状态。

例如，修改配置文件的脚本如果运行多次，不应添加重复的项。

### <a name="bPS5"></a>确保群集体系结构的高可用性

基于 Linux 的 HDInsight 群集提供在群集中保持活动状态的两个头节点，而脚本操作将针对这两个节点运行。 如果安装的组件只应使用一个头节点，请不要在两个头节点上安装组件。

> [AZURE.IMPORTANT]
> 作为 HDInsight 一部分提供的服务旨在根据需要在两个头节点之间进行故障转移。 此功能未扩展到通过脚本操作安装的自定义组件。 如果需要为自定义组件提供高可用性，必须实现自己的故障转移机制。

### <a name="bPS6"></a>配置自定义组件以使用 Azure Blob 存储

在群集上安装的组件可能具有使用 Hadoop 分布式文件系统 (HDFS) 存储的默认配置。 HDInsight 使用 Azure 存储作为默认存储。 两者可以提供与 HDFS 兼容的文件系统，即使删除了群集，也能保存数据。 可能需要将安装的组件配置为使用 WASB 或 ADL，而不是 HDFS。

对于大多数操作，不需要指定文件系统。 例如，以下脚本将 giraph-examples.jar 文件从本地文件系统复制到群集存储：

    hdfs dfs -put /usr/hdp/current/giraph/giraph-examples.jar /example/jars/

在此示例中，`hdfs` 命令以透明方式使用默认群集存储。 对于某些操作，可能需要指定 URI。 例如，`wasb:///example/jars` 适用于 Azure 存储。

### <a name="bPS7"></a>将信息写入 STDOUT 和 STDERR

HDInsight 会记录已写入 STDOUT 和 STDERR 的脚本输出。 可以使用 Ambari Web UI 查看这些信息。

> [AZURE.NOTE]
> 只有在成功创建群集之后，才能使用 Ambari。 如果在群集创建期间使用脚本操作但创建失败，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#troubleshooting)的故障排除部分，了解访问已记录信息的其他方式。

大多数实用工具和安装包已将信息写入 STDOUT 和 STDERR，不过你可能需要添加更多日志记录。 若要将文本发送到 STDOUT，请使用 `echo`。 例如：

    echo "Getting ready to install Foo"

默认情况下，`echo` 会将字符串发送到 STDOUT。 若要将它定向到 STDERR，请在 `echo` 的前面添加 `>&2`。 例如：

    >&2 echo "An error occurred installing Foo"

这会将写入 STDOUT 的信息改为重定向到 STDERR (2)。 有关 IO 重定向的详细信息，请参阅 [http://www.tldp.org/LDP/abs/html/io-redirection.html](http://www.tldp.org/LDP/abs/html/io-redirection.html)。

若要详细了解如何查看脚本操作记录的信息，请参阅[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/#troubleshooting)

### <a name="bps8"></a> 将文件另存为包含 LF 行尾的 ASCII

应将 Bash 脚本存储为 ASCII 格式，该格式以 LF 作为行尾。 存储为 UTF-8 的文件或使用 CRLF 作为行尾的文件可能发生故障，并返回以下错误：

    $'\r': command not found
    line 1: #!/usr/bin/env: No such file or directory

### <a name="bps9"></a> 使用重试逻辑从暂时性错误中恢复

下载文件、使用 apt-get 安装包或者执行通过 Internet 传输数据的其他操作时，可能会由于暂时性网络错误而失败。 例如，与你通信的远程资源可能正在故障转移到备份节点。

若要使脚本能够从暂时性错误中恢复，可以实现重试逻辑。 以下函数演示如何实现重试逻辑。 它在失败之前重试操作三次。

    #retry
    MAXATTEMPTS=3

    retry() {
        local -r CMD="$@"
        local -i ATTMEPTNUM=1
        local -i RETRYINTERVAL=2

        until $CMD
        do
            if (( ATTMEPTNUM == MAXATTEMPTS ))
            then
                    echo "Attempt $ATTMEPTNUM failed. no more attempts left."
                    return 1
            else
                    echo "Attempt $ATTMEPTNUM failed! Retrying in $RETRYINTERVAL seconds..."
                    sleep $(( RETRYINTERVAL ))
                    ATTMEPTNUM=$ATTMEPTNUM+1
            fi
        done
    }

以下示例演示如何使用此函数。

    retry ls -ltr foo

    retry wget -O ./tmpfile.sh https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh

## <a name="helpermethods"></a>自定义脚本的帮助器方法

脚本操作帮助器方法是可以在编写自定义脚本时使用的实用工具。 这些方法包含在 [https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh) 脚本中。 使用以下命令下载这些方法并在脚本中使用：

    # Import the helper method module.
    wget -O /tmp/HDInsightUtilities-v01.sh -q https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh && source /tmp/HDInsightUtilities-v01.sh && rm -f /tmp/HDInsightUtilities-v01.sh

可在脚本中使用以下帮助器：

| 帮助器用法 | 说明 |
| --- | --- |
| `download_file SOURCEURL DESTFILEPATH [OVERWRITE]` |将文件从源 URI 下载到指定的文件路径。 默认情况下，它不会覆盖现有的文件。 |
| `untar_file TARFILE DESTDIR` |将 tar 文件解压缩（使用 `-xf`）到目标目录。 |
| `test_is_headnode` |如果在群集头节点上运行，则返回 1；否则返回 0。 |
| `test_is_datanode` |如果当前节点是数据（辅助角色）节点，则返回 1；否则返回 0。 |
| `test_is_first_datanode` |如果当前节点是第一个数据（辅助角色）节点（名为 workernode0），则返回 1；否则返回 0。 |
| `get_headnodes` |返回群集中头节点的完全限定域名。 名称以逗号分隔。 出错时返回空字符串。 |
| `get_primary_headnode` |获取主头节点的完全限定域名。 出错时返回空字符串。 |
| `get_secondary_headnode` |获取辅助头节点的完全限定域名。 出错时返回空字符串。 |
| `get_primary_headnode_number` |获取主头节点的数字后缀。 出错时返回空字符串。 |
| `get_secondary_headnode_number` |获取辅助头节点的数字后缀。 出错时返回空字符串。 |

## <a name="commonusage"></a>常见使用模式

本部分提供有关实现你在编写自己的自定义脚本时可能遇到的一些常见使用模式的指导。

### <a name="passing-parameters-to-a-script"></a>将参数传递给脚本

在某些情况下，脚本可能需要参数。 例如，使用 Ambari REST API 时，可能需要群集的管理员密码。

传递给脚本的参数称为“位置参数”，将分配到 `$1` 作为第一个参数，分配到 `$2` 作为第二个参数，依此类推。 `$0` 包含脚本本身的名称。

作为参数传递给脚本的值应括在单引号 (') 中。 这样可以确保将传递的值视为文本。

### <a name="setting-environment-variables"></a>设置环境变量

以下语句设置环境变量：

    VARIABLENAME=value

其中，VARIABLENAME 是变量的名称。 若要访问变量，请使用 `$VARIABLENAME`。 例如，若要将位置参数提供的值指定为名为 PASSWORD 的环境变量，请使用以下语句：

    PASSWORD=$1

对信息进行后续访问时可以使用 `$PASSWORD`。

在脚本中设置的环境变量只在脚本范围内存在。 在某些情况下，可能需要添加系统范围的环境变量，这些变量在脚本完成之后仍会存在。 若要添加系统范围的环境变量，请将变量添加到 `/etc/environment`。 例如，以下语句添加 `HADOOP_CONF_DIR`：

    echo "HADOOP_CONF_DIR=/etc/hadoop/conf" | sudo tee -a /etc/environment

### <a name="access-to-locations-where-the-custom-scripts-are-stored"></a>访问存储自定义脚本的位置

用于自定义群集的脚本需要存储在以下位置之一：

* 与群集关联的 __Azure 存储帐户__。

* 与群集关联的 __其他存储帐户__ 。

* __可公开读取的 URI__。 例如，在 OneDrive、Dropbox 或其他文件托管服务中存储的数据的 URL。

脚本使用的资源也必须公开提供。

将文件存储在 Azure 存储帐户中可以快速访问，因为是在 Azure 网络中进行。

> [AZURE.NOTE]
> 用于引用脚本的 URI 格式因所使用的服务而异。 对于与 HDInsight 群集关联的存储帐户，请使用 `wasb://` 或 `wasbs://`。 对于可公开读取的 URI，请使用 `http://` 或 `https://`。

### <a name="checking-the-operating-system-version"></a>检查操作系统版本

不同版本的 HDInsight 依赖于特定版本的 Ubuntu。 你的脚本必须查看的 OS 版本之间可能存在差异。 例如，可能需要安装与 Ubuntu 版本相关的二进制文件。

若要检查 OS 版本，请使用 `lsb_release`。 例如，以下脚本演示如何根据 OS 版本引用特定的 tar 文件：

    OS_VERSION=$(lsb_release -sr)
    if [[ $OS_VERSION == 14* ]]; then
        echo "OS verion is $OS_VERSION. Using hue-binaries-14-04."
        HUE_TARFILE=hue-binaries-14-04.tgz
    elif [[ $OS_VERSION == 16* ]]; then
        echo "OS verion is $OS_VERSION. Using hue-binaries-16-04."
        HUE_TARFILE=hue-binaries-16-04.tgz
    fi

## <a name="deployScript"></a>有关部署脚本操作的清单

下面是我们在准备部署这些脚本时执行的步骤：

* 将包含自定义脚本的文件放置在群集节点在部署期间可访问的位置中。 例如，群集的默认存储。 还可以将文件存储在可公开读取的托管服务中。
* 验证脚本是否幂等。 这样，便可以在同一个节点上执行脚本多次。
* 使用临时文件目录 /tmp 来保存脚本使用的下载文件，并在执行脚本后将其清除。
* 如果更改了 OS 级别设置或 Hadoop 服务配置文件，可能需要重新启动 HDInsight 服务。

## <a name="runScriptAction"></a>如何运行脚本操作

可以使用以下方法通过脚本操作自定义 HDInsight 群集：

* Azure 门户
* Azure PowerShell
* Azure Resource Manager 模板
* HDInsight .NET SDK。

有关使用每种方法的详细信息，请参阅[如何使用脚本操作](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)。

## <a name="sampleScripts"></a>自定义脚本示例

Microsoft 提供了在 HDInsight 群集上安装组件的示例脚本。 参阅以下链接，了解更多示例脚本操作。

* [在 HDInsight 群集上安装并使用 Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)
* [在 HDInsight 群集上安装并使用 Solr](/documentation/articles/hdinsight-hadoop-solr-install-linux/)
* [在 HDInsight 群集上安装并使用 Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-linux/)
* [在 HDInsight 群集上安装或升级 Mono](/documentation/articles/hdinsight-hadoop-install-mono/)

## <a name="troubleshooting"></a>故障排除

使用开发的脚本时可能会遇到以下错误：

错误：`$'\r': command not found`。 有时后面会接着出现“ `syntax error: unexpected end of file`”。

*原因*：此错误的原因是脚本中以 CRLF 作为行尾。 Unix 系统只允许使用 LF 作为行尾。

此问题最常出现于 Windows 环境中编写的脚本，因为 CRLF 是 Windows 上许多文本编辑器中常见的行尾符号。

*解决方法*：如果文本编辑器提供了选项，请选择 Unix 格式或 LF 作为行尾。 也可以在 Unix 系统上使用以下命令，将 CRLF 更改为 LF：

> [AZURE.NOTE]
> 以下命令大致相当于将 CRLF 行尾更改为 LF。 根据系统中提供的实用工具选择一种解决方法。

| 命令 | 说明 |
| --- | --- |
| `unix2dos -b INFILE` |原始文件将以 .BAK 扩展名备份 |
| `tr -d '\r' < INFILE > OUTFILE` |OUTFILE 包含只带 LF 行尾的版本 |
| `perl -pi -e 's/\r\n/\n/g' INFILE` | 直接修改文件 |
| ```sed 's/$'"/`echo \\\r`/" INFILE > OUTFILE``` |OUTFILE 包含只带 LF 行尾的版本。 |

错误：`line 1: #!/usr/bin/env: No such file or directory`。

原因：将脚本另存为包含字节顺序标记 (BOM) 的 UTF-8 时会发生此错误。

*解决方法*：将文件另存为 ASCII，或者不带 BOM 的 UTF-8。 也可以在 Linux 或 Unix 系统上使用以下命令来创建不带 BOM 的文件：

    awk 'NR==1{sub(/^\xef\xbb\xbf/,"")}{print}' INFILE > OUTFILE

将 `INFILE` 替换为包含 BOM 的文件。 `OUTFILE` 应是新文件名，该文件包含不带 BOM 的脚本。

## <a name="seeAlso"></a>后续步骤

* 了解如何[使用脚本操作自定义 HDInsight 群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)
* 通过 [HDInsight.NET SDK 参考](https://msdn.microsoft.com/zh-cn/library/mt271028.aspx)详细了解如何创建用于管理 HDInsight 的 .NET 应用程序
* 使用 [HDInsight REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt622197.aspx) 了解如何通过 REST 在 HDInsight 群集上执行管理操作。

<!--Update_Description: wording update-->