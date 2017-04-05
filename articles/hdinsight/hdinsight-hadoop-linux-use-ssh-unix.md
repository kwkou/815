<properties
    pageTitle="在 Windows、Linux、Unix 或 OS X 上将 SSH 与 HDInsight (Hadoop) 配合使用 | Azure"
    description=" 使用安全外壳 (SSH) 访问 HDInsight。本文档提供有关在 Windows、Linux、Unix 或 OS X 客户端中将 SSH 与 HDInsight 配合使用的信息。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="a6a16405-a4a7-4151-9bbf-ab26972216c5"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/27/2017"
    wacn.date="03/31/2017"
    ms.author="larryfr"
    ms.custom="H1Hack27Feb2017" />  


# 在 Windows 10 上的 Bash、Linux、Unix 或 OS X 中将 SSH 与 HDInsight (Hadoop) 配合使用
> [AZURE.SELECTOR]
- [PuTTY (Windows)](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
- [SSH（Windows、Linux、Unix、OS X）](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)

在[安全外壳 (SSH)](https://zh.wikipedia.org/wiki/Secure_Shell) 中，可以使用命令行接口登录到基于 Linux 的 HDInsight 群集并运行命令。本文档提供有关 SSH 的基本信息，以及有关在 HDInsight 中使用 SSH 的具体信息。

## 什么是 SSH？

SSH 是一种加密网络协议，可用于通过不安全的网络来与远程服务器安全通信。使用 SSH 可以通过命令行安全登录到远程服务器。在本例中，远程服务器是 HDInsight 群集的头节点或边缘节点。

还可以使用 SSH 以隧道方式将网络流量从客户端传送到 HDInsight 群集。使用隧道可以访问 HDInsight 群集中不直接在 Internet 上公开的服务。有关在 HDInsight 中使用 SSH 隧道的详细信息，请参阅 [Use SSH tunneling with HDInsight](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)（在 HDInsight 中使用 SSH 隧道）。

## SSH 客户端

许多操作系统通过 `ssh` 和 `scp` 命令行实用工具提供 SSH 客户端功能。

* __ssh__：可用于建立远程命令行会话和创建隧道的常规 SSH 客户端。
* __scp__：可以使用 SSH 协议在本地系统与远程系统之间复制文件的实用工具。

Windows 10 周年纪念版提供 Bash 作为开发人员功能。它提供了 `ssh`、`scp` 和其他 Linux 命令。有关使用 Bash on Windows 10 的详细信息，请参阅 [Bash on Ubuntu on Windows](https://msdn.microsoft.com/commandline/wsl/about)（Windows 上的 Ubuntu Bash）。

如果你使用的是 Windows 但无法访问 Bash on Windows 10，我们建议使用以下 SSH 客户端：

* [适用于 Windows 的 Git](https://git-for-windows.github.io/)：提供 `ssh` 和 `scp` 命令行实用工具。
* [Cygwin](https://cygwin.com/)：提供 `ssh` 和 `scp` 命令行实用工具。

> [AZURE.NOTE]
本文档中的步骤假设你可以访问 `ssh` 命令。如果使用 puTTY 或 MobaXterm 等客户端，请查阅相应产品的文档，了解等效的命令和参数。

## SSH 身份验证

可以使用密码或[公钥加密 (https://en.wikipedia.org/wiki/Public-key\_cryptography)](https://en.wikipedia.org/wiki/Public-key_cryptography) 对 SSH 连接进行身份验证。最安全的做法是使用密钥，因为它不像密码那样容易受到多种攻击。但是，与使用密码相比，创建和管理密钥更为复杂。

使用公钥加密涉及到创建_公钥_和_私钥_对。

* **公钥**将载入 HDInsight 群集的节点中，或者载入要对其使用公钥加密的其他任何服务。

* **私钥**是使用 SSH 客户端登录时，为了验证自己的身份而提供给 HDInsight 群集的凭据。请保护好私钥，不要透露给其他人。

    为私钥创建通行短语可以进一步提高安全性。如果你使用密码，必须在使用 SSH 进行身份验证时输入它。

### 创建公钥和私钥

创建用于 HDInsight 的公钥和私钥对的最简单方法是使用 `ssh-keygen` 实用工具。从命令行使用以下命令即可创建用于 HDInsight 的新密钥对：

> [AZURE.NOTE]
如果使用 MobaXTerm 或 puTTY 等 GUI SSH 客户端，请查阅客户端的文档了解如何生成密钥。

    ssh-keygen -t rsa -b 2048

系统将提示输入以下信息：

* 文件位置：位置默认为 `~/.ssh/id_rsa`。

* 可选的通行短语：如果输入了一个通行短语，在 HDInsight 群集上身份验证时必须重新输入该通行短语。

> [AZURE.IMPORTANT]
通行短语是私钥的密码。每当使用私钥进行身份验证时，必须先提供通行短语，然后才能使用该密钥。如果有人获取了你的私钥，在不知道通行短语的情况下，他们无法使用该私钥。
><p>
> 但是，如果你忘记了通行短语，就没有办法重置或恢复它。

完成上述命令后，将获得两个新文件：

* __id\_rsa__：此文件包含私钥。

    > [AZURE.WARNING]
    需限制对此文件的访问，防止有人未经授权访问公钥保护的服务。

* __id\_rsa.pub__：此文件包含公钥。创建 HDInsght 群集时需要用到此文件。

    > [AZURE.NOTE]
    谁有权访问_公钥_并不重要。所有公钥的作用无非就是验证私钥。当你使用私钥进行身份验证时，SSH 服务器等服务使用公钥来验证你的身份。

## 在 HDInsight 上配置 SSH

创建基于 Linux 的 HDInsight 群集时，必须提供 _SSH 用户名_以及_密码_或_公钥_。在创建群集的过程中，将使用这些信息在 HDInsight 群集节点上创建登录名。密码或公钥用于保护用户帐户。

有关在创建群集期间配置 SSH 的详细信息，请参阅以下文档之一：

* [Create HDInsight using the Azure portal preview（使用 Azure 门户预览创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-portal/)
* [Create HDInsight using the Azure CLI（使用 Azure CLI 创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-cli/)
* [Create HDInsight using Azure PowerShell（使用 Azure PowerShell 创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-azure-powershell/)
* [Create HDInsight using Azure Resource Manager templates（使用 Azure Resource Manager 模板创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-arm-templates/)
* [Create HDInsight using the .NET SDK（使用 .NET SDK 创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-dotnet-sdk/)
* [Create HDInsight using REST（使用 REST 创建 HDInsight）](/documentation/articles/hdinsight-hadoop-create-linux-clusters-curl-rest/)

### 其他 SSH 用户

尽管创建群集后可将其他 SSH 用户添加到群集，但我们不建议这样做。

* 需要将新的 SSH 用户添加到群集中的每个节点。

* 新 SSH 用户对 HDInsight 的访问权限与默认用户相同。无法根据 SSH 用户帐户限制访问 HDInsight 中的数据或作业。

若要基于用户限制访问权限，必须使用已加入域的 HDInsight 群集。已加入域的 HDInsight 使用 Active Directory 来控制对群集资源的访问。

##<a id="connect" name="connect-to-a-linux-based-hdinsight-cluster"></a>连接到 HDInsight

尽管 HDInsight 群集中的所有节点都运行 SSH 服务器，但你只能通过公共 Internet 连接到头节点或边缘节点。

* 若要连接到_头节点_，请使用 `CLUSTERNAME-ssh.azurehdinsight.cn`，其中，__CLUSTERNAME__ 是 HDInsight 群集的名称。端口 22（SSH 的默认端口）连接到主头节点。端口 23 连接到辅助头节点。

* 若要连接到_边缘节点_，请使用 `EDGENAME.CLUSTERNAME-ssh.azurehdinsight.cn`，其中，__EDGENAME__ 是边缘节点的名称，__CLUSTERNAME__ 是 HDInsight 群集的名称。连接到边缘节点时请使用端口 22。

以下示例演示如何使用 SSH 用户名 __sshuser__ 连接到名为 __myhdi__ 的群集的头节点和边缘节点。边缘节点名为 __myedge__。

| 为此，请执行以下操作... | 使用此方法... |
| ----- | ----- |
| 连接到主头节点 | `ssh sshuser@myhdi-ssh.azurehdinsight.cn` |
| 连接到辅助头节点 | `ssh -p 23 sshuser@myhdi-ssh.azurehdinsight.cn` |
| 连接到边缘节点 | `ssh sshuser@edge.myhdi-ssh.azurehdinsight.cn` |

如果使用密码保护 SSH 帐户，系统将提示输入该密码。

如果使用公钥保护 SSH 帐户，可能需要使用 `-i` 开关指定匹配的私钥的路径。以下示例演示如何使用 `-i` 开关：

    ssh -i /path/to/public.key sshuser@myhdi-ssh.azurehdinsight.cn

### 连接到其他节点

辅助角色节点和 Zookeeper 节点不能直接从群集外部访问，但可以从群集头节点或边缘节点访问。下面是连接到其他节点的常规步骤：

1. 使用 SSH 连接到头节点或边缘节点：

        ssh sshuser@myhdi-ssh.azurehdinsight.cn

2. 通过 SSH 连接到头节点或边缘节点后，使用 `ssh` 命令连接到群集中的辅助角色节点：

        ssh sshuser@wn0-myhdi

    若要检索群集中辅助角色节点的列表，请参阅 [Manage HDInsight by using the Ambari REST API](/documentation/articles/hdinsight-hadoop-manage-ambari-rest-api/#example-get-the-fqdn-of-cluster-nodes)（使用 Ambari REST API 管理 HDInsight）文档中有关如何检索群集节点完全限定域名的示例。

如果使用密码保护 SSH 帐户，系统会要求输入该密码来建立连接。

如果使用 SSH 密钥对用户帐户进行身份验证，必须确保为本地环境配置 SSH 代理转发。

> [AZURE.IMPORTANT]
以下步骤假设在基于 Linux/UNIX 的系统上操作，并且能够使用 Bash on Windows 10。如果这些步骤不适用于你的系统，你可能需要查阅 SSH 客户端的文档。

1. 使用文本编辑器打开 `~/.ssh/config`。如果此文件不存在，可以在命令行中输入 `touch ~/.ssh/config` 来创建。

2. 将以下内容添加到该文件中。将 *CLUSTERNAME* 替换为 HDInsight 群集的名称。

        Host CLUSTERNAME-ssh.azurehdinsight.cn
          ForwardAgent yes

    此条目为 HDInsight 群集配置 SSH 代理转发。

3. 在终端中通过使用以下命令测试 SSH 代理转发：

        echo "$SSH_AUTH_SOCK"

    此命令返回类似于以下文本的信息：

        /tmp/ssh-rfSUL1ldCldQ/agent.1792

    如果未返回任何信息，则表示 `ssh-agent` 未运行。请参阅 [Using ssh-agent with ssh (http://mah.everybody.org/docs/ssh)](http://mah.everybody.org/docs/ssh)（将 ssh-agent 与 ssh 配合使用）中的代理启动脚本信息，或者查阅 SSH 客户端文档，了解安装和配置 `ssh-agent` 的具体步骤。

4. 验证了 **ssh-agent** 处于运行状态后，请使用以下方式将你的 SSH 私钥添加到代理：

        ssh-add ~/.ssh/id_rsa

    如果你的私钥存储在不同文件中，请将 `~/.ssh/id_rsa` 替换为该文件的路径。

## <a id="tunnel"></a>SSH 隧道

可以使用 SSH 来以隧道方式将本地请求（例如 Web 请求）传送到 HDInsight 群集。请求将转发到群集，然后在群集中对其进行解析。

> [AZURE.IMPORTANT]
访问某些 Hadoop 服务的 Web UI 需要使用 SSH 隧道。例如，作业历史记录 UI 或资源管理器 UI 只能使用 SSH 隧道访问。

有关创建和使用 SSH 隧道的详细信息，请参阅 [Use SSH Tunneling to access Ambari web UI, JobHistory, NameNode, Oozie, and other web UIs](/documentation/articles/hdinsight-linux-ambari-ssh-tunnel/)（使用 SSH 隧道访问 Ambari Web UI、JobHistory、NameNode、Oozie 和其他 Web UI）。

## 后续步骤

既然你了解了如何使用 SSH 密钥进行身份验证，就可以学习如何在 HDInsight 上将 MapReduce 与 Hadoop 配合使用。

* [将 Hive 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)
* [将 MapReduce 作业与 HDInsight 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

[preview-portal]: https://portal.azure.cn/

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description: wording update-->